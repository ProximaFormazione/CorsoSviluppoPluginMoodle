Architettura 1
==============

Una installazione di moodle si compone di tre elementi principali su disco:

* La cartella con il sito esposto
* La cartella *moodledata*
* Il database

Questi tre elementi sono sufficienti per l'interita' della piattaforma base, e sono quanto serve per replicare la piattaforma in un altro ambiente.

A seconda dell'infrastruttura potrebbero poi esserci altri elementi, come ad esempio un Redis per gestire le sessioni in caso di piattaforma su piu' istanze dietro ad un load balancer, o servizi di autenticazione esterni.

Cartella sito
=============

[Documentazione Ufficiale](https://docs.moodle.org/403/en/Moodle_site_moodle_directory)

La cartella del sito contiene tutti i file (php ed altro) necessari alla esposizione delle pagine del sito. 

Questa cartella non contiene dati che non siano statici, nel funzionamento normale di un'istanza moodle non dovrebbero avvenire modifiche all'interno di questa cartella. Questo rende la cartella un ottimo candidato per un repository git.

Affinche' un nuovo plugin venga riconosciuto dalla piattaforma, sara' necessario seguire le prassi richieste da moodle, il che comporta posizionare i files nelle cartelle corrette.

Nella root vi sono alcuni script utilizzate in fase di installazione, e per la configurazione sistemistica base della piattaforma.

version.php
-----------

Il file **version.php** e' il file che definisce la versione attuale della piattaforma, non e' un file da modificare ed e' utile per capire la versione usata.

Il file ha solo una serie di propieta':

`$version`
:  E' il numero di versione effettivo, scritto nel formato YYYYMMDDXX.YY , dove YYYY MM e DD sono la data di commit, XX e' un progressivo per avere piu' versioni nella stessa data, ed YY e' un'ulteriore progressivo usato nella versione del sito. La modifica di questo valore scatena la procedura di aggiornamento del sito

`$release`
: e' un numero di versione leggibile per l'utente

`$branch`
: il branch originale di moodle utilizzato

`$maturity`
: una costante che identifica lo stato di sviluppo di questa versione. Valori usati sono `MATURITY_ALPHA` `MATURITY_BETA` `MATURITY_RC` e `MATURITY_STABLE`

La riga `defined('MOODLE_INTERNAL') || die();` e' una riga usata molto comunemente in moodle per i file ad uso interno e serve ad impedire che lo script venga fatto girare se richiesto direttamente tramite il web server.

La struttura di questo file e' la stessa usata nelle cartelle di plugin. Vedremo piu' in dettaglio in seguito le opzioni aggiuntive (non modificheremo il file del sito)

config.php
----------

Il file **config.php** e' il file di configurazione da impostare per il funzionamento base della piattaforma.

Definisce il global `$CFG` che poi viene utilizzato in altri punti del codice di moodle core.

Questo file dovrebbe a buona norma essere ignorato nel repository git (nel gitignore del repository ufficiale e' gia' cosi')

Contiene quantomeno le impostazioni per la connessione al database, la posizione della cartella dei dati, e l'url di root del sito.

Un esempio di un file di configurazione:

```
<?php  // Moodle configuration file

unset($CFG);
global $CFG;
$CFG = new stdClass();

$CFG->dbtype    = 'mariadb';
$CFG->dblibrary = 'native';
$CFG->dbhost    = 'localhost';
$CFG->dbname    = 'moodle';
$CFG->dbuser    = 'db_admin';
$CFG->dbpass    = '$#%; drop table scraped_passwords ;--';
$CFG->prefix    = 'mdl_';
$CFG->dboptions = array (
  'dbpersist' => 0,
  'dbport' => '',
  'dbsocket' => '',
  'dbcollation' => 'utf8mb4_unicode_ci',
);

$CFG->wwwroot   = 'https://moodlesito.it';

$CFG->dataroot  = '/var/moodledata';
$CFG->directorypermissions = 0777;

$CFG->admin     = 'admin';

//$CFG->debug = (E_ALL | E_STRICT);

require_once(__DIR__ . '/lib/setup.php');
```

Nel primo blocco abbiamo le informazioni di base del database, con il tipo di driver utilizzato, host utente e password. Qui e' anche definito il prefisso usato per le tabelle del database, che potete scegliere in fase di installazione e serve per evitare di avere i nomi standard per le tabelle

Successivamente viene definita la root per le url del sito, in modo da poter usare indrizzi relativi nei link e semplificare la portabilita' della piattaforma.

Segue la posizione della cartella con i dati e l'impostazione dei permessi per le nuove cartelle da creare.

la riga $CFG->admin va modificata se l'url "admin" e' gia' usata sul web server.

E' possibile impostare congigurazioni aggiuntive in questo file, incluse impostazioni che normalmente sono accessibili nel pannello di amministrazione dentro la piattaforma. Questo e' utile in fase di debug se la piattaforma non funziona (qui commentata avete l'impostazione per mostrare messaggi di errore dettagliati).

Nelle distribuzioni normali di moodle e' presente un file template completamente commentato: `config-dist.php` che spiega in dettaglio ogni opzione disponibile.

Questo file viene creato in fase di installazione di moodle dallo script di instalazione per le piattaforme nuove.

Cartelle
--------

Le cartelle di moodle contengono i vari files con pagine e librerie di supporto.

i plugin installati devono trovarsi nella cartella corretta per la classe di plugin affinche' vengano correttamente identificati dal sistema.

Alcune cartelle salienti:

lib
: Contiene le librerie core utilizzate praticamente ovunque

lang
: Contiene le stringhe usate nel sito, una cartella per lingua installata

pix, login, course, user, files, calendar, admin
: Cartelle con funzionalita' core di moodle

mod
: Cartella con i plugin di attivita' possibili per i corsi (course modues)

altre cartelle ospitano i plugin di quella classe, di cui generalmente moodle core ha uno o piu' esemplari, tra queste:

auth
: Modalita' di autenticazione

blocks
: Blocchi di html inseribili ovunque

enrol
: Modalita' di iscrizione ai corsi

local
: Plguin generici non riconducibili a classi esistenti

repository
: Gestione files salvati

theme
: Tema da usare per le pagine, comanda molti aspetti della visualizzazione

Cartella dati (moodledata)
==========================

La cartella dati di moodle e' il repository primo dei files caricati o generati sulla piattaforma.

Per le installazioni semplici (1 macchina), la cartella moodledata puo' tranquillamente stare nello stesso file system del sito e questo garantisce performance buone.

Per le installazioni con piu' istanze, la cartella deve essere in una posizione condivisa tra le istanze.

Sessioni
--------

la cartella **sessions** ospita di default le sessioni utente per il sito. 

Per le installazioni con piu' istanze utilizzare il salvataggio delle sessioni in questa cartella puo' essere un grosso problema per le performance, nel cui caso e' possibile indicare nel file config.php del sito un driver di sessioni diverso, ad esempio Redis 

```
$CFG->session_handler_class = '\core\session\redis';
$CFG->session_redis_host = '127.0.0.1';
$CFG->session_redis_acquire_lock_timeout = 120;
$CFG->session_redis_lock_expire = 7200;
```

driver alternativi includono il database (anch'esso poco performante) e memcached.

[Documentazione dettagliata su sessioni](https://docs.moodle.org/403/en/Session_handling)

Cache
-----

[Documentazione](https://docs.moodle.org/403/en/Caching)

Moodle fa largo uso di caching nel suo funzionamento normale.

L'implementazione delle cache e' mediata da un sistema chiamato MUC (Moodle Universal chaching) che ha il compito di gestire i vari depositi possibili per le cache.

Vi sono diversi plugin possibili per il salvataggio delle cache, alcuni preinstallati, tra cui il salvataggio su filesystem, che avviene appunto nella cartella moodledata nella sottocartella **cache**

le cache vanno condivise tra le istanze della piattaforma, tuttavia e' possibile definire una cartella di cache locale tramite il parametro `$CFG->localcachedir` per caches di elementi grafici o comunque immutabili che possono essere replicati sulle varie istanze. Questo approccio e' utile se le cache utilizzate non sono all'altezza delle aspettative.

E' possibile ripulire le caches del sito tramite apposito comando all'interno della piattaforma nel menu' di amministrazione -> sviluppo

filedir
-------

la cartella **filedir** e' il deposito di tutti i files caricati sulla piattaforma.

Al caricamento di un file, moodle esegue l'hash del file (sha1sum) e immagazzina il file usando l'hash come nome, questo fa si che non ci sian files duplicati.

I files sono poi conservati in cartelle basate sui primi due bytes dell'hash, immagino per evitare cartelle con troppi files.

La mappatura tra file ed hash e' nel database nella tabella `files`.

I plugin che determinano modalita' alternative di conservazione dei files sono i plugin di tipo **repository**, vi sono una vasta gamma di plugins (es: FTP, google drive, S3...).

Un plugin notabile, e preinstallato, e' **File System**, che permette di designare delle cartelle sul file system dove accedere ai files in chiaro. Questa modalita' e' utile per permettere agli utenti di caricare files di grosse dimensioni (gigabytes) tramite FTP o altro metodo
 
lang
----

la cartella **lang** contiene le stringhe del language pack che vengono modificate manualmente dagli utenti autorizzati a farlo. la carrella ha una struttura similare a quella nella cartella del sito

Altre cartelle
--------------

Altre cartelle utilizzate dalla piattaforma includono le cartelle **temp**, **trashdir**, **lock**. Queste cartelle possono essere escluse dai backup (eccetto trashdir) e dal controllo di versione se usato nella cartella moodledata (in tal caso escludere anche cache e sessions ).

I files in temp e trashdir vengono regolarmente cancellati da moodle in base ad impostazioni sul sito.






