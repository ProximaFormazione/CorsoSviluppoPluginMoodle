Utilizzo database
=================

Finora abbiamo utilizzato plugin stateless, ma nella maggior parte dei casi e' necessario far persistere dei dati su una qualche forma di database.

Avere una struttura modulare, dove i plugin possono essere installati ed aggiornati, comporta una serie di complessita' nella gestione dello schema, che fortunatemente sono gestite in moodle in maniera sistematica.

Vediamo in questo capitolo le basi di quanto serve per utilizzare il database nei plugin in moodle.



Consumare il database
=====================

Per utilizzare il database, moodle mette a disposizione delle sue API interne, nella variabile globale `$DB`

```php
function funzione_che_usa_database() {
    global $DB;

    // [...]
}
```

`$DB` e' un istanza della classe `moodle_database` che si trova nel file `lib/dml/moodle_database.php`. La classe e' ben documentata ma dovete aprire il file per vedere i commenti. 

Aprendo il file in VS Code dovreste avere i metodi nell'intellisense, altrimenti nelle estensioni di VS Code per moodle ci sono degli snippet di codice appositi. Alternativamente molti dei metodi sono elencati nella documentazione ([Data manipulation API](https://moodledev.io/docs/apis/core/dml)).

`$DB` contiene sia metodi 'semplici' dove basta indicare la tabella e le condizioni, ma anche metodi che permettono l'esecuzione di query SQL per le operazioni piu' complesse.

Laddove si crea una query SQL custom, bisogna fare attenzione a non utilizzare sintassi specifiche di un particolare tipo di database se si vuole garantire la compatibilita' con i vari motori di database supportati da moodle

Generalmente i record sono restituiti cone `stdClass`. I metodi che restituiscono liste di record restituiscono un array di `stdClass` con il primo campo come chiave. Come prassi nelle tabelle di moodle il primo campo e' una primary key intera con identity chiamata `id`

Alcuni concetti comuni nei metodi delle API:

Nome
----

Il prefisso delle tabelle, inserito in fase di installazione e definito nel config.php, NON va mai usato nel codice. Laddove viene richiesto un nome di una tabella questo va inserito senza il prefisso. Questo sia nei metodi dove viene chiesta come parametro

```php
$user = $DB->get_record('user', ['id' => '1']);
```

Sia laddove viene usata una query SQL, dove la tabella va inserita tra parentesi graffe

```php
$DB->get_record_sql('SELECT * FROM {user} WHERE id = 1');
```

Condizioni
----------

Il parametro `$conditions` permette di definire delle condizioni per filtrare il risultato (WHERE). Si tratta di un array dove la chiave e' il nome del campo ed il valore e' il valore usato nel filtro.

Tutte le consizioni devono essere verificate contemporaneamente (and)

```php
$user = $DB->get_record('user', ['firstname'  => 'Mario', 'lastname'  => 'Rossi']);
```

Parametri query
---------------

Nelle query che dipendono da dei parametri, moodle mette a disposizione un sistema di sanificazione degli input.

Sono supportati placeholder nelle query sia sotto forma di `?`

```php
$sql1 = 'SELECT * FROM {user} WHERE firstname = ? AND lastname = ?';
```

che sotto forma di parametri con nome, indicati con i `:` seguiti dal nome del placeholder

```php
$sql2 = 'SELECT * FROM {user} WHERE firstname = :nome AND lastname = :cognome';
```

I metodi che usano le query hanno un parametro per passare i valori dei placeholder sotto forma di array, per i placeholder con nome bisogna usare il nome come chiave, mentre quelli anonimi dipendono dall'ordine in cui vengono forniti

```php
$DB->get_record_sql(
    $sql1 ,
    [
        'Mario',
        'Rossi',
    ]
);

$DB->get_record_sql(
    $sql2,
    [
        'nome'  => 'Mario',
        'cognome'  => 'Rossi',
    ]
);
```

per le clausole IN(....) esiste il metodo `get_in_or_equal` che restituisce sia il SQL da inserire che i parametri da utilizzare.

Si raccomanda di non inserire i parametri delle query direttamente come stringa per evitare attacchi tramite SQL injection

![Esempio, chiaramente reale, di SQL injection](https://imgs.xkcd.com/comics/exploits_of_a_mom.png  "Esempio, chiaramente reale, di SQL injection")

Strictness
----------

Il parametro `$strictness` e' una costante che determina il comportamento della query in caso di record multipli o mancanti

* `MUST_EXIST` se il record deve esistere e deve essere unico, altrimenti restituisce un errore
* `IGNORE_MISSING` se e' accettabile che il record non esista (nel cui caso ritorna `false`), restituisce comunque un errore se ci sono record multipli
* `IGNORE_MULTIPLE` in questa modalita' se ci sono multipli record restituisce il primo senza dare alcun errore (generalmente sconsigliato)

Il parametro non viene usato nei metodi che restituiscono liste di record.

Altri parametri
---------------

Altri parametri usati:

* `$fields` permette di specificare i campi da includere nell'estrazione, va fornito come stringa con i nomi dei campi separati da `,`. Di default e' `'*'`. ATTENZIONE: ricordatevi che in caso di liste di record il primo campo viene usato come chiave dell'array, quindi assiccuratevi che sia unico. Raccomando di includere sempre l'`id`.
* `sort` permette di indicare l'ordine dei record nell'array restituito.
* `limitfrom` e `$limitnum` sono per il paging del risultato.
* metodi specifici potrebbero avere altri parametri

Update
------

Per i metodi di UPDATE e' necessario avere valorizzata la propieta' `id` del record con la primary key. I metodi INSERT generano sempre un nuovo record a meno di non usare lo specifico metodo `insert_record_raw`

Esempi
------

Alcuni esempi di utilizzo

```php
global $DB;

//query con estrazione di piu' elementi
$extraction = $DB->get_records('local_linkfacilita_cm_field', ['coursemoduleid' => $cmid], '', 'facilitafield,value'); //nota: non estraiamo l'id, auindi l'array $extraction ha come chiave il campo 'facilitafield'

//query di estrazione con JOIN e clausola IN(....)
//(estrae alcuni campi custom di un utente)
$params = ['userid' => $userid];
list($in_clause_sql,$in_clause_params) = $DB->get_in_or_equal($array_valori_richiesti,SQL_PARAMS_NAMED);
$params += $in_clause_params;

$fields_data = $DB->get_records_sql("
    select 
    muid.fieldid,
    muif.shortname,
    muid.`data`,
    muif.datatype 
    from {user_info_field} muif 
    join {user_info_data} muid on muid.fieldid = muif.id 
    where muid.userid = :userid and muif.shortname {$in_clause_sql}", $params);


//inserimento record
$new_record = new stdClass();
$new_record->coursemoduleid = $cmid;
$new_record->facilitafield = $facilitafield;
$new_record->value = $value;
$DB->insert_record('local_linkfacilita_cm_field', $new_record);


//update record tramite query custom
$update_sql = 'UPDATE {local_linkfacilita_cm_field} set `value` = :value where coursemoduleid = :cmid and facilitafield = :facilitafield';
$DB->execute($update_sql, ['value' => $value, 'cmid' => $cmid, 'facilitafield' => $facilitafield]);

```

Transazioni
-----------

`$DB` possiede delle API per la gestione delel transazioni, che quindi non vanno gestite direttamente nel SQL.

```php
try {
    $transaction = $DB->start_delegated_transaction();
    // [...]
    $transaction->allow_commit();
} catch (Exception $e) {
    // Eventuali operazioni aggiuntive richieste / cattura o log eccezioni
    $transaction->rollback($e);
}
```

[Documentazione](https://moodledev.io/docs/apis/core/dml/delegated-transactions)

Definire le tabelle
===================

Se un plugin utilizza delle tabelle, queste devono essere dichiarate in modo che in fase di installazione moodle possa provvedere a crearle.

E' inoltre presente un meccanismo per gestire gli aggiornamenti dello schema da una versione del plugin alla successiva.

Per fare cio' dovremo definire due files, il cui contenuto vedremo in dettaglio:

* un file XML con lo schema `db/install.xml`
* un file PHP con le operazioni di aggiornamento `db/upgrade.php`

Prassi
------

Come per il codice, moodle individua delle prassi da utilizzare per la nomenclatura e struttura del database.

A differenza dello stile del codice, qui si raccomanda di attenersi il piu' possibile alle linee guida, per i seguenti motivi:

* Modificare dopo la nomenclatura in uno schema e' problematico
* Si tratta di regole semplici
* Alcune prassi sono richieste dal codice e/o dell'oggetto `$DB`

La lista completa e' disponibile [qui](https://docs.moodle.org/dev/Database). Elenco qui alcune delle piu' rilevanti:

* la primary key di ogni tabella deve essere un campo INT(10) con identity/sequence/auto-increment chiamato **id** 
* i nomi delle tabelle devono essere piu' corti di 28 caratteri (53 da moodle 4.3), devono essere lowercase ed includere il prefisso frankenstyle del plugin
* come corollario di sopra, il nome effettivo della tabella deve essere breve
* eventuali foreign keys devono essere chiamate `tabella` + `id` (es: `userid`), alcune vecchie tabelle non rispettano questa nomenclatura
* Usare indici univoci piuttosto che chiavi
* Per i moduli di attivita' e' richiesta una tabella chiamata come il plugin con le istanze dell'attivita'. Deve includere `id` (la PK), `course` (id del corso di appartenenza) e `name` (nome dell'istanza)

Date
----

Le date nel database di moodle sono salvati in [timestamp unix](https://it.wikipedia.org/wiki/Tempo_(Unix)), ovvero con un intero che indica il numero di secondi passati dall' 1 gennaio 1970.

Esistono vari metodi di aiuto per gestire i timestamp in moodle ([Documentazione completa](https://moodle.academy/mod/lesson/view.php?id=817&pageid=162)).

Interessante la funzione `userdate` e la funzione `usertime` che tengono presente il fuso orario indicato dall'utente.

php ha una funzione `time()` per ottenere il timestamp attuale.

sul database i campi sono INT(10)

Definizione XMLDB ed editor
---------------------------

la definizione corrente delle tabelle e' contenuta nel file `db/install.xml`.

Come suggerisce il nome, questo file viene usato al momento della prima installazione del plugin in moodle.

E' un file in formato XML, con uno schema chiamato XMLDB ([Documentazione dettagliata](https://docs.moodle.org/dev/XMLDB_Documentation)). La necessita' di utilizzare un linguaggio specifico e' per garantire la compatibilita' con diverse tecnologie di database.

E' possibile compilare manualmente il file, magari copiandone uno simile, o per eseguire semplici modifiche. Tuttavia moodle mette a disposizione un editor specifico dentro la piattaforma che e' in grado di creare, leggere e modificare tale file.

L'editor XMLDB e' accessibile dal menu' Amministrazione del sito -> Sviluppo -> Editor XMLDB. 

L'editor ha un'interfaccia spartana ma funziona bene, i file che legge e crea sono posizionati nella cartella corretta della piattaforma. Quindi per utilizzarlo si richiede di sviluppare direttamente in un ambiente installato (che comunque e' consigliabile).

Nell'editor inizialmente si ha una lista dei plugin installati, dove si puo' caricare o creare il file xml relativo. Una volta caricato si possono editare le tabelle ed i campi tramite interfaccia grafica.

Fare attenzione a salvare le modifiche usando gli appositi pulsanti.

L'editor possiede poi dei generatori di codice php ed xml, utilizzabili per trasportare le definizioni fatte su altre cartelle, ma se sviluppate direttamente sull'ambiente che state utilizzando userete solo quello per il PHP.

Nella definizione delle tabelle e' possibile anche definire chiavi ed indici, prestare attenzione al fatto che non vi e' un grande controllo degli input.

Definizione step aggiornamento
------------------------------

La definizione delle operazioni da fare in sede di aggiornamento del plugin e' nel file `db/upgrade.php`

il file deve contenere le varie funzioni da lanciare, ed ha delle regole abbastanza stringenti sulle nomenclature usate. 

deve essere presente una funzione `xmldb_nome_plugin_upgrade($oldversion)` che prende come parametro il numero di versione del plugin prima dell'aggiornamento. All'interno possiamo utilizzare il manager del database in `$DB` per eseguire operazioni di DDL.

```php
defined('MOODLE_INTERNAL') || die();    

function xmldb_nome_plugin_upgrade($oldversion) {
    global $DB;
 
    $dbman = $DB->get_manager();
 
    
    if ($oldversion < 2021110501) {
        
        //[...] operazioni di aggiornamento

        upgrade_plugin_savepoint(true, 2021110501, 'nome', 'plugin');
    }
}
```

Fortunatamente l'editor XMLDB ha un'opzione per la creazione automatica di tali funzioni, permettendovi di fare copia ed incolla. L'output di tale funzione ha tutto cio' che vi serve e dovete solo inserire il numero di versione al posto degli `XXXXXXXXXX`

esempio di output: aggiunta di un campo

```php
if ($oldversion < XXXXXXXXXX) {

    // Define field autoactivate to be added to enrol_linknroll_key.
    $table = new xmldb_table('enrol_linknroll_key');
    $field = new xmldb_field('autoactivate', XMLDB_TYPE_INTEGER, '1', null, XMLDB_NOTNULL, null, '0', 'company');

    // Conditionally launch add field autoactivate.
    if (!$dbman->field_exists($table, $field)) {
        $dbman->add_field($table, $field);
    }

    // Linknroll savepoint reached.
    upgrade_plugin_savepoint(true, XXXXXXXXXX, 'enrol', 'linknroll');
}
```

se eseguite piu' operazioni in un update sentitevi liberi di editare gli output per rimuovere parti ridondanti.

In fase di aggiornamento questo metodo viene sempre lanciato, quindi potete usarlo anche per eseguire operazioni non collegate al database se ne avete necessita'. Se lo fate ricordate di implementarle in modo che funzionino indipendentemente dallo stato effettivo (ad esempio controllando che la modifica non esista gia').

Aggiungere elementi a questo file e' necessario solo se entrambe le seguenti sono vere:

* Avete apportato modifiche allo schema
* Il plugin e' rilasciato da qualche parte oltre al vostro ambiente di sviluppo


