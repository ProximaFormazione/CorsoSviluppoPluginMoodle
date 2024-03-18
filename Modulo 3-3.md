Unit Test
=========

Moodle prevede un solido framework per gli unit test, con features di recupero dello stato iniziale per i test incorporate. In questo capitolo vedremo come utilizzarlo.

Su ogni piattaforma installata e' presente una voce nel menu' di amministrazione in Sviluppo -> PHP Unit test con una checklist e link alle documentazioni

Introduzione agli unit test
---------------------------

Gli unit test sono solo uno dei processi usati per il test di un applicativo, si tratta di test **automatici** che verificano unicamente un elemento di codice indipendentemente dagli elementi di codice che consumano quello testato, ed indipendentemente dagli elementi usati da quello testato.

Non si tratta di test eseguiti da una persona, si tratta di codice che esegue delle operazioni e verifica il risultato. Gli unit test aiutano a verificare la funzionalita' di una feature anche in fasi di sviluppo successive e/o di altre funzionalita' ed aiutano ad intercettare possibili regressioni inserite.

Gli unit test non sono sostitutivi al test manuale del codice.

Gli unit test vanno scritti nel codice, idealmente in fase di sviluppo. I test devono avere le seguenti caratteristiche:

* Devono essere ripetibili
* Non devono dipendere o influenzare altri test impostati
* Devono poter essere lanciati senza intervento

Generalmente, un unit test si compone di tre fasi:

* Inizializzazione dei dati
* Esecuzione dei test
* Assert, ovvero verifica dei risultati rispetto ai valori attesi

Esistono diversi framework per gli Unit test. Nel PHP Moodle utilizza PHPUnit([link](https://docs.phpunit.de/en/9.6/)).

Gli Unit test da definizione dovrebbero occuparsi di testare la minima unita' di codice possibile, ma per fare cio' e' necessario che l'architettura sia compatibile (ad esempio implementando un meccanismo di inversione delle dipendenze). Moodle non e' strutturato per aderire perfettamente a tale standard, per cui propiamente non sempre i test che si creano sono definibili come "unit test". (spesso sono piu' propiamente integration test) 

Le best practice e la teoria dei test automatici del codice non e' argomento di questo corso. Nel resto di questo modulo per "Unit test" si puo' intendere grossolanamente "Test automatico".

Installazione librerie PHPUnit
------------------------------

Per funzionare, il framework necessita di PHPUnit installato, possiamo installarlo manualmente, oppure possiamo utilizzare **Composer**.

Composer e' un gestore di librerie per il PHP (simile a nuGet sul .Net). Moodle ha nella root di ogni installazione un file con varie dipendenze per l'ambiente di sviluppo che possiamo utilizzare per configurare velocemente quanto ci serve.

La procedura di sotto e' per ambienti Windows.

Per prima cosa installiamo composer, ad esempio scaricando l'installer ([Link](https://getcomposer.org/Composer-Setup.exe)), oppure seguendo le istruzioni sul sito ufficiale ([Link](https://getcomposer.org/doc/00-intro.md)). Attenzione se usate l'installer vi e' un problema di compatibilita' con xDebug (potete disattivare l'estensione fino al termine dell'installazione nel php.ini)

Una volta installato composer, possiamo usarlo per installare tutte le varie dipendenze indicate da moodle eseguendo il seguente comando nella **cartella root di moodle**:

```
php composer.phar install
```

( a seconda del metodo di installazione di composer il comando potrebbe invece `composer install`)

A questo punto abbiamo installato PHPUnit (ed altre librerie) sulla piattaforma del nostro computer.

**ATTENZIONE**: Si tratta di librerie per l'ambiente di sviluppo, che presentano criticita' a livello di performance e sicurezza se usati su un sito pubblico, assicuratevi di non copiare per sbaglio tali librerie in produzione. Moodle comunque controlla e notifica se sono installate.

Inizializzazione ambiente di test
---------------------------------

[Documentazione ufficiale](https://moodledev.io/general/development/tools/phpunit)

Il framework di test di moodle richiede l'inizializzazione di tabelle e dati, di fatto installa una replica del sito che viene utilizzata in fase di test. Tale replica viene poi reinizializzata dal framework per garantire che sia sempre identica ad ogni run deio test.

Prima dell'installazione bisogna aggiungere delle informazioni al config.php

```php
$CFG->phpunit_prefix = 'phpu_';
$CFG->phpunit_dataroot = '/home/example/phpu_moodledata';
```

Si tratta delle stesse informazioni della piattaforma principale, il prefisso e' per le tabelle del DB e la dataroot e' una seconda cartella dati. E' possibile anche definire un database separato

```php
$CFG->phpunit_dbtype    = 'pgsql'; 
$CFG->phpunit_dblibrary = 'native'; 
$CFG->phpunit_dbhost    = '127.0.0.1';
$CFG->phpunit_dbname    = 'mytestdb';
$CFG->phpunit_dbuser    = 'postgres'; 
$CFG->phpunit_dbpass    = 'some_password'; 
```

Una volta definito dove il framework salva le informazioni, dobbiamo inizializzarlo con il comando

```
php admin/tool/phpunit/cli/init.php
```

questo script lancia l'installazione dell'ambiente di test, e' analoga alla procedura di installazione di moodle, con la differenza che ad ogni aggiornamento elimina e ricrea le tabelle (richiede un po' di tempo per essere eseguito). 

Lo script va rilanciato ogniqualvolta si esegue un aggiornamento di moodle, ovvero quando si aggiorna o installa uno o piu' plugin. 

Scrittura Unit Test
-------------------

[Documentazione ufficiale](https://docs.moodle.org/dev/Writing_PHPUnit_tests)

Gli unit test vanno inseriti in appositi files nella cartella `/tests` del plugin.

I nomi dei files devono coincidere con il nome della classe del test, e questa deve finire in `_test`. E' richiesto un namespace corretto per il plugin, ovvero il nome plugin (frankenstyle) con le cartelle usate come sottonamespaces, eccetto per la cartella `/tests`.

ESEMPIO: il test per la creazione di un link sara' messo nella classe `enrol_magiclink\create_link_test` nel file `enrol/magiclink/tests/create_link_test.php`.

Moodle mette a disposizione tre classi base da estendere per gli unit test:

* **basic_testcase**: Test semplici che non modificano ne il database, ne variabili globali. Questa classe va usata in questo caso d'uso perche' ha il vantaggio addizionale di verificare e dare un errore se invece vengono eseguite modifiche
* **advanced_testcase**: Per test generali che non ricadono nella categoria di sopra, include molti metodi utili.
* **provider_testcase**: Test per i provider privacy. Usato raramente

All'interno della classe inseriremo i metodi per i nostri casi di test. Ogni metodo deve iniziare con `test_`

ESEMPIO: nel file `enrol/magiclink/tests/create_link_test.php`:

```php
namespace enrol_magiclink;

defined('MOODLE_INTERNAL') || die();

global $CFG; //- $CFG non e' nello stesso scope degli unit test

class create_link_test extends \advanced_testcase {
 
    public function test_create link() {
        // [...] logica del test
    }
}
```

Generalmente i test hanno bisogno di parametri da usare e valori attesi. In PhpUnit e' possibile definire nei PhpDocs del metodo un secondo metodo **data provider** che fornira' un array con i dati, in modo da separare la definizione degli stessi dai passaggi di test ed assert.

Per utilizzare un data provider dobbiamo specificare il metodo nel PhpDocs con l'istruzione `@dataProvider`. Il metodo che provvisiona i dati restuira' un array di array, questi ultimi dovranno avere i parametri richiesti dal metodo

ES:
```php
/**
     * @dataProvider local_greetings_get_greeting_provider
     * @param string|null $country User country
     * @param string $expected Greetings message language string
     */
    public function test_local_greetings_get_greeting(?string $country, string $expected) {
        
        //[...]

    }

    /**
     * Data provider for {@see test_local_greetings_get_greeting()}.
     *
     * @return array List of data sets - (string) data set name => (array) data
     */
    public function local_greetings_get_greeting_provider() {
        return [
            'No user' => [ // Not logged in.
                'country' => null,
                'expected' => 'Hello'
            ],
            'AU user' => [
                'country' => 'AU',
                'expected' => 'Hello mate'
            ],
            'ES user' => [
                'country' => 'ES',
                'expected' => 'Hola'
            ],
            'IT user' => [ 
                'country' => 'IT',
                'expected' => 'Ciao'
            ],
        ];
    }
```

E' importante che il data provider non esegua logiche, ma si limiti a definire dati cablati. Questo metodo viene infatti eseguito spesso internamente.

All'interno del metodo di test abbiamo a disposizione una serie di metodi helper. Vedi [Documentazione ufficiale](https://docs.moodle.org/dev/Writing_PHPUnit_tests) per una lista piu' esaustiva

### Assert

Abbiamo una serie di metodi per eseguire l'Assert, come `assertEmpty` o `assertEquals`. Questi metodi verificano i risultati dell'elaborazione ed in caso di errore restituiscono un messaggio parlante sul problema. Si tratta di metodi di PhpUnit la lista completa e' presente sulla documentazione ufficiale ([link](https://docs.phpunit.de/en/9.6/assertions.html#assertequals)).

In generale il test si conclude con esito negativo se il codice restituisce un errore, non e' necessario catturare e rilanciare eccezioni. I metodi di Assert infatti restituiscono errori se non vanno a buon fine

### Reset stato iniziale

Di default dopo un test lo stato della cartella dati e del database vengono riportati allo stato iniziale.

E' necessario indicare nel test che e' sono previste delle modifiche, altrimenti se il reset esegue delle operazioni riportera' il fallimento del test.

E' possibile indicare tramite il metodo `$this->resetAfterTest(true)` se tali modifiche sono previste. Il messaggio di errore *"Warning: unexpected database modification, resetting DB state"* e' dovuto alla mancata presenza di tale istruzione (o se non prevista, da una modifica avvenuta).

### Generatori dati 

Nel suo stato iniziale l'ambiente degli unit test e' spoglio di dati. E' compito di ogni singolo caso di test impostare cio' che e' necessario al test. Sono presenti una serie di metodi per generare dati utili.

Una lista esaustiva e' presente nella documentazione ufficiale ([link](https://docs.moodle.org/dev/Writing_PHPUnit_tests#Generators)), alcuni esempi utili:

```php
$user = $this->getDataGenerator()->create_user();
$user1 = $this->getDataGenerator()->create_user(['email'=>'user1@example.com', 'username'=>'user1']);

$course1 = $this->getDataGenerator()->create_course(/*[dati]*/);

$this->getDataGenerator()->enrol_user($userid, $courseid);
```

per le attivita' ed i repository e' necessario definire un generatore apposito nel plugin, e poi si puo' utilizzare:

```php
 $course = $this->getDataGenerator()->create_course();
 $generator = $this->getDataGenerator()->get_plugin_generator('mod_page');
 $generator->create_instance(array('course'=>$course->id));

 $this->getDataGenerator()->create_repository($type, $record, $options);
```

Per gli utenti, di base sono presenti solo **Guest** ed **Administrator**, altri utenti vanno inseriti. Inoltre di base non vi e' un utente loggato e' necessario impostare l'utente attuale se necessario

```php
 $this->setGuestUser();
 $this->setAdminUser();
 $this->setUser(null);
```

### Messaggi e mail

E' possibile raccogleire le mail in un oggetto verificabile via codice tramite i seguenti metodi

```php
unset_config('noemailever');
$sink = $this->redirectEmails();
//... codice del test
$messages = $sink->get_messages();
$this->assertEquals(1, count($messages));
```

Esecuzione test
---------------

per eseguire dei test bisogna lanciare dei comandi a terminale. Se si usa VS Code si consiglia di utilizzare il terminale integrato. 

Il comando per lanciare un test e'

```
php vendor/bin/phpunit --filter {TEST_SPECIFICO}_testcase
```

ad esempio, lanciamo un test di un plugin gia' installato in moodle:

```
php vendor/bin/phpunit --filter 'tool_dataprivacy\\metadata_registry_test'
```

e' possibile anche lanciare direttamente il file con PhpUnit:

```
vendor/bin/phpunit admin/tool/dataprivacy/tests/metadata_registry_test.php
```

Possiamo poi omettere il test da eseguire per eseguire tutti i test 

```
php vendor/bin/phpunit
```

**ATTENZIONE**: questo lancera' tutti i test per tutti i plugin installati, l'operazione puo' essere molto lunga (ore).

Il comando puo' essere integrato in eventuali pipelines di rilascio in ambienti di staging o produzione.

Se si ha installato l'estensione MDLCode, e' presente un link nei metodi di test correttamente inseriti per lanciare direttamente il test.
