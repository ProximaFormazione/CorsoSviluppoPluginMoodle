Privacy dei dati
=======

[>> LINK A DOCUMENTAZIONE UFFICIALE <<](https://moodledev.io/docs/4.5/apis/subsystems/privacy)

L'avvento del GDPR ha richiesto diverse attenzioni ai produttori di software affinche' fossero compliant con i vari diritti dei consumatori ivi sanciti, in primis il "diritto all'oblio", ovvero la cancellazione di tutti i dati personali inerenti all'individuo.

La struttura modulare di Moodle ha richiesto l'implementazione di una serie di interfacce per i plugin in modo che possano ottemperare al meccanismo centrale impostato in moodle.

Da notare che questo meccanismo e' per l'applicazione dei principi del GDPR. Non si tratta della visualizzazione della policy del sito (anche se e' certamente argomento adiacente)

A cosa serve
------------

L'implementazione delle API della privacy permette all'utente le seguenti possibilita':

- Vedere le varie categorie di dati trattati, le loro finalita' e la durata del trattamento degli stessi
- Richiedere lo scaricamento dei propri dati
- Richiedere la cancellazione dei propri dati

> NOTA: Generalmente l'eliminazione dei dati elimina i record dal database. Tuttavia l'utente viene eliminato logicamente secondo le normali pratiche di moodle, il record rimane quindi nel database, con i dati anagrafici e la mail ancora recuperabile. Valutare se inserire un sistema ulteriore per completare l'eliminazione.

Impostazione
------------

L'impostazione avviene nel tab Amministrazione del sito -> Utenti -> Privacy. In tale gruppo sono presenti anche le impostazioni per la policy (che non sono oggetto di questo capitolo)

La configurazione master delle funzionalita' e' alla voce "Impostazione privacy", qui e' possibile attivare la possibilita' di richiesta cancellazione o esportazione dati (di default non e' attiva) ed eventualmente impostare l'accettazione automatica delle richieste.

> ATTENZIONE: Le cancellazioni di dati sono irreversibili ed includono tutti i dati collegati all'utente. Valutare attentamente se approvarle automaticamente.

Il secondo step necessario alla configurazione e' il registro dei dati, (che piu' correttamente sarebbe dovuto chiamarsi registro dei trattamenti) Dove potete definire le categorie di dati trattati e le finalita'. Le categorie hanno una funziona puramente descrittiva, mentre le finalita' possono decidere elementi funzionali come la durata del trattamento, e la possibilita' di flaggare i dati come non eliminabili.

Le categorie e finalita' sono impostabili a livello di singoli contesti, con opzioni per settare i default o permettere di ereditare dal contesto superiore.

Implementazione
---------------

In teoria ogni singolo componente di Moodle dovrebbe implementare questa API, questo include eventuali nostri plugin custom. Non vi e' pero' un blocco per cui capita di trovare plugins che non implementano l'API Privacy

L'implementazione consiste nella creazione di un apposita classe nel nostro plugin, la classe deve trovarsi in `/classes/privacy/provider.php` ed avere come namespace `nome_plugin\privacy`. La classe (che si chiama `provider` dal nome del file) dovra' implementare un'interfaccia adatta.

### Nessun dato conservato

Se il nostro plugin non conserva alcun dato personale, ovvero non abbiamo nessun riferimento ad un user id nelle nostre tabelle (o non abbiamo tabelle), possiamo eseguire un implementazione dell'interfaccia `\core_privacy\local\metadata\null_provider`, di fatto implementando un solo metodo che restituisce l'identificativo della stringa del language pack con la giustificazione del fatto che non ci sono dati sensibili

```php
class provider implements  \core_privacy\local\metadata\null_provider {
    public static function get_reason(): string {
        return 'privacy_scusa';
    }
}
```

language pack:

```php
$string['privacy_scusa'] = 'Questo plugin non fa nulla, e non facendo nulla non ha bisogno di dati, che pertanto non vengono conservati';
```

Questa implementazione fa si che il nostro plugin non venga flaggato come problematico nella lista dei trattamenti fatti dai singoli plugin (accessibile agli amministratori), di fatto attestando che avete valutato esplicitamente che non avete bisogno di implementare l'API.

### In caso si conservino dati personali

Se invece trattiamo dei dati personali dovremo implementare l'interfaccia ` \core_privacy\local\metadata\provider`, dobbiamo implementare tre metodi: uno per elencare le tipologie di dati conservati, uno per esportarli ed uno per eliminarli.

#### Dichiarazione

Il metodo che esegue la dichiarazione e' `get_metadata(collection $collection): collection` che richiede di aggiungere alla variabile collection tutte le tipologie di dati conservati, da notare che questo include:

- tabelle del nostro plugin
- settaggi dell'utente
- Eventuali contenuti dell'utente in altri sottosistemi di moodle, come ad esempio:
    - deposito domande
    - tags
    - files caricati
- eventuali dati salvati su sistemi esterni

Esistono metodi appositi per molti di questi casi da usare sulla variabile collection

Esempio:

```php
class provider implements
    \core_privacy\local\metadata\provider {

    public static function get_metadata(\core_privacy\local\metadata\collection $collection): \core_privacy\local\metadata\collection {

        $collection->add_subsystem_link(
            'core_files', // il sottosistema usato
            [], // opzionale: lista dei campi usati nella forma fieldname => descrizione
            'core_files_scusa' // identificativo della stringa nel nostro language pack
        );

        $collection->add_database_table(
            'local_dispense_creditcardnumber', // nome della tabella
             [
                'userid' => 'privacy_creditcardnumber_userid_scusa',  // giustificazioni per singolo campo
                'cardnumber' => 'privacy_creditcardnumber_cardnumber_scusa',
             ],
            'privacy_creditcardnumber_scusa' // giustificazione per la tabella
        );

        $collection->add_user_preference(
            'local_dispense_enableloginby_neuralink', // nome del settaggio
            'privacy_setting_scusa'
        );

        return $collection;
    }
}
```

#### Esportazione

Per la maggior pare dei plugin, l'interfaccia da implementare e' `\core_privacy\local\request\plugin\provider`. Vi sono eccezioni da valutare nei casi di sottoplugin (sia se siamo noi il sottoplugin che se li gestiamo) ed accorgimenti per versioni vecchie di moodle (< 3.6) per le quali si rimanda alla documentazione.

In breve per l'esportazione dovremo implementare i metodi 

- `get_contexts_for_userid(int $userid): \core_privacy\local\request\contextlist` per identificare i contesti che contengono dati personali dell'utente
- `export_user_data`

la funzione get_contexts_for_userid restituisce di fatto la lista degli id dei contesti in cui compare l'utente, ma la vuole dentro una classe `contextlist` che di fatto ammette come metodo di inserimento unicamente una query. Sono presenti dei wrapper per alcuni tipi di contesti come quello di sistema e quello utente

```php
public static function get_contexts_for_userid(int $userid): \core_privacy\local\request\contextlist {
    $contextlist = new \core_privacy\local\request\contextlist();

    $sql = "SELECT c.id
                FROM {context} c
        [...]
        WHERE x.userid = :userid
    ";

    $params = [
        'userid'=> $userid,
    ];

    $contextlist->add_from_sql($sql, $params); // aggiungo i contesti dalla query sopra, che sara' abbastanza convoluta
    $contextlist->add_user_context($userid); // aggiungo il contesto utente (se usato)
    $contextlist->add_system_context(); // aggiungo il contesto di sistema (se usato)

    return $contextlist;
}

```

L'esportazione effettiva produce un json articolato per contesto, che includendo tutti i dati dell'utente puo' essere molto complesso, per cui viene fornita una classe helper per la creazione dei dati

```php
use \core_privacy\local\request\approved_contextlist;
use \core_privacy\local\request\writer;

public static function export_user_data(approved_contextlist $contextlist){

    // ....recupero dati

    foreach($contextlist as $context){

         writer::with_context($context)
            ->export_data([], $record)
    }
}
```

L'esempio sopra e' nel caso di tabelle, per gli altri casi si rimanda alla documentazione ufficiale, questa api e' particolarmente complessa

#### Eliminazione

I metodi da implementare per l'eliminazione sono:

- `delete_data_for_all_users_in_context(\context $context) : void`
- `delete_data_for_user(\core_privacy\local\request\approved_contextlist $contextlist) : void`

Questi metodi non hanno requisiti particolari, al loro interno dovete semplicemente eseguire le operazioni per l'eliminazione dei dati, nel primo metodo in maniera indiscriminata su un contesto e nel secondo per un particolare utente nei contesti indicati. Per recuperare l'utente possiamo usare un apposito metodo di `approved_contextlist`

esempio:

```php
public static function delete_data_for_user(approved_contextlist $contextlist) {
    global $DB;

    if (empty($contextlist->count())) {
        return;
    }
    $userid = $contextlist->get_user()->id;
    foreach ($contextlist->get_contexts() as $context) {
        $DB->delete_records('local_dispense_offspring_address', ['contextid' => $context->id, 'userid' => $userid]);
    }
}
```







