Grandi installazioni
====================

La guida finora ha trattato installazioni A seconda del vostro caso d'uso, potreste dover valutare soluzioni con istanze multiple e load balancing. Le ragioni possono essere molteplici:

- Fornire scalabilita' in situazioni di carico intensivo senza sovradimensionare un server h24
- Fornire ridondanza in caso di guasti
- Il cliente lo richiede senza altre spiegazioni

Moodle puo' essere usato in installazioni dove sono previste piu' istanze, tuttavia saranno richiesti degli step aggiuntivi di configurazione, e soprattutto un analisi accurata delle prestazioni in modo da verificare i risultati.

Fintanto si rimane nel reame delle migliaia di utenti (non contemporanei) una singola istanza con risorse medie e' in grado di gestire il carico senza esplodere.

In questo capitolo non ci soffermeremo sui dettagli di una particolare tecnologia, come ad esempio il Kubernetes, ma discuteremo degli accorgimenti da seguire in linea generale in modo che possiate applicarli alle tecnologie usate. Ci sono guide per vari stack sulla documentazione ufficiale ([Link](https://docs.moodle.org/405/en/Large_installations))

Overview
--------

Una installazione di moodle di base e' composta da tre elementi:

- Il **sorgente**, ovvero i files esposti dal web server
- La cartella **moodledata**, che contiene vari elementi non conservati nel database come ad esempio i files caricati
- Il **Database**

In un installazione base questi tre elementi sono gestiti dalla stessa macchina, ma e' possibile separarli in server diversi in modo da poter gestire i requisiti di ognuno separatamente.

Generalmente il caso d'uso piu' comune e' la moltiplicazione del web server dietro ad un load balancer, con una istanza di database ed una cartella moodledata condivisa.

Web Server
----------

Come detto moltiplicare il web server e' generalmente l'opzione piu' praticata. 

Di fatto le uniche informazioni salvate sono nella cartella moodledata o sul database, Moodle non salva mai (nel suo uso quotidiano) direttamente dentro la cartella del sorgente, che contiene solo il php eseguito quando viene richiesta una pagina.

Detto questo, fintanto tutte le istanze hanno accesso agli stessi dati del database e della cartella moodledata, possono girare un qualsiasi numero di istanze senza problemi.

Le configurazioni gia' viste delle singole istanze non richiedono particolari attenzioni, il puntamento per database e dati dovrebbe essere identico, e laddove avete un reverse proxy anche l'hostname risultera' lo stesso (quello del proxy). Qualora non ci fosse un reverse proxy potete anche avere diverse istanze indipendenti raggiungibili con indirizzi diversi (magari per platee diversificate), in tal caso ogni istanza avra' il proprio parametro `wwwroot` indicato nel file `config.php`. Nel resto della guida ci soffermeremo sul caso che prevede il reverse proxy

Vi sono delle feature che non funzioneranno piu' correttamente con piu' istanze: principalmente l'installazione di plugin via interfaccia, in quanto agirebbe unicamente sull'istanza raggiunta con il browser web. Nei moodle a piu' istanze le installazioni di plugin vanno gestite esternamente caricando direttamente la cartella su tutte le istanze (di solito modificando il repository dal quale vengono create). 

In caso di plugin terzi bisogna verificare che non applichino modifiche ai files del sito affinche siano compatibili con questa configurazione

### Gestione sessioni

Moodle gestisce le sessioni utente lato server, con una serie di informazioni accessorie legate all'utente. I dati di sessione sono array PHP serializzati dell'ordine di grandezza di 4-8 kb.

In un installazione base di moodle le sessioni vengono salvate nella cartella moodledata nella sottocartella *sessions*. Vi sono a disposizione opzioni differenti, ma a livello di pagina di amministrazione del sito (Server -> Session handling) e' possibile unicamente optare per utilizzare invece il database, che ha performance anche peggiori. (nella pagina di amministrazione de sito possono invece essere settate impostazioni come la durata delle sessioni).

Per configurare uno dei restanti motori di sessioni e' necessario inserire i settaggi adeguati direttamente nel file `config.php`. La lista completa degli store di sessioni attualmente supportata e' la seguente:

- File
- Database
- Memcahced
- Redis

File e database sono indicate unicamente per installazioni singole, Memchaced e Redis sono due storage chiave-valore che utilizzano unicamente la RAM, rendendoli molto performanti ma meno indicati per conservare dati a lungo termine. Entrambi i sistemi sono invece ottimali per dati di vita breve come le cache o le sessioni, entrambi sono utilizzabili anche come storage per le **caches** in moodle (caches e sessioni sono ambiti diversi). Per le installazioni con piu' istanze e' caldamente raccomandato utilizzare uno di questi due.

Quale utilizzare dipende dalla vostra esperienza con le tecnologie, entrambi forniscono per questo scopo funzionalita' equivalenti, generalmente la scelta piu' comune che si vede in giro e' Redis,tuttavia Memcahced ha una storia piu' lunga e sicuramente si trova ancora in giro.

Per la configurazione di redis dobbiamo aggiungere queste righe al file `config.php`

```
$CFG->session_handler_class = '\core\session\redis';
$CFG->session_redis_host = '127.0.0.1';
$CFG->session_redis_port = 6379;
$CFG->session_redis_database = 0;
$CFG->session_redis_auth = '';
$CFG->session_redis_prefix = '';
$CFG->session_redis_acquire_lock_timeout = 120;
$CFG->session_redis_acquire_lock_warn = 0;
$CFG->session_redis_lock_expire = 7200;
$CFG->session_redis_lock_retry = 100;  
$CFG->session_redis_serializer_use_igbinary = false; // Optional, default is PHP builtin serializer.
$CFG->session_redis_compressor = 'none';
```

I settaggi obbligatori sono solo due:

- `$CFG->session_handler_class` indica di utilizzare il driver Redis per le sessioni, come molti altri settaggi impostati direttamente in questo file, questa impostazione ha la precedenza su qualsiasi altra impostazione effettuata via interfaccia
- `$CFG->session_redis_host` sono gli estremi dell'istanza redis, che generalmente e' una macchina a se.  

Gli altri settaggi sono opzionali, sopra sono assegnati i valori di default:

- `$CFG->session_redis_port` la porta, di default e' la 6379
- `$CFG->session_redis_database` indica quali dei 16 database dell'istanza redis verra' utilizzato, di default viene usato il db 0
- `$CFG->session_redis_auth` sono eventuali dettagli di autenticazione, se abilitati su redis. Stringa vuota se non impostati, altrimenti usate `[username] [password]` o `[password]` a seconda della configurazione e versione di redis usata
- `$CFG->session_redis_prefix` permette di aggiungere un prefisso per le chiavi usate, utile se volete utilizzare la stessa istanza di redis per piattaforme diverse (alternativa di utilizzare uno degli altri 16 databases)
- `$CFG->session_redis_acquire_lock_timeout` e' il valore in secondi dopo cui moodle si arrende se non riesce ad ottenere il lock (redis) su una risorsa, restituendo errore all'utente. Per le sessioni questo puo' avvenire se l'utente e' loggato su piu' browser o schede
- `$CFG->session_redis_acquire_lock_warn` se diverso da 0, e' il valore in secondi dopo cui moodle scrive un record nel log se non ha ancora ottenuto un lock 
- `CFG->session_redis_lock_expire` cancella automaticamente la chiave generata su redis dopo l'intervallo specificato in secondi, se non specificato viene popolato con la durata massima della sessione
- `$CFG->session_redis_lock_retry` e' l'intervallo in millisecondi tra un tentativo e l'altro di ottenere un lock, dopo 5 secondi viene ignorato e moodle riprova ogni seondo
- `$CFG->session_redis_serializer_use_igbinary` determina se utilizzare igbinary ([link](https://github.com/igbinary/igbinary)) come serializzatore al posto del serializzatore di default del php. utilizzare igbinary fa occupare meno spazio in memoria (fino al 50% secondo i suoi creatori.), ma trattandosi di un meccanismo di compressione questo e' a discapito della velocita'. Per essere abilitato richiede la relativa estensione sul php, e richiede che l'estensione di redis del php sia compilata con il flag `--enable-redis-igbinary`. Questo cambia il formato dei dati salvati quindi se modificato bisogna svuotare il db su redis
- `$CFG->session_redis_compressor` e' l'algoritmo usato per comprimere i valori salvati. Opzioni possibili sono `none` `gzip` e `zstd`. La compressione viene gestita lato moodle

Tipicamente la configurazione di default dovrebbe andare bene.

E' possibile poi attivare un'altra opzione inerente le **sessioni in sola lettura** che permette di ovviare all'acquisizione del lock sulla sessione laddove possibile. Nel file di configurazione tale opzione e' marcata come "sperimentale, la documentazione la dichiara "stabile dalla 3.11", ma sulla pull request rimane qualche sviluppatore incerto ([Link](https://tracker.moodle.org/browse/MDL-58018)).

Questa feature produce un aumento delle prestazioni sulle pagine che esplicitamente supportano tale feature, per abilitarla bisogna aggiungere questa riga al file `config.php`:

```
$CFG->enable_read_only_sessions = true;
```

E' disponibile un opzione alternativa che salva eventuali problemi sul log, `$CFG->enable_read_only_sessions_debug = true;` che puo' essere utile per verificare la funzionalita' nei primi tempi, ma che va sostituita una volta a regime.

Per attivare il supporto in una pagina di un nostro plugin andremo ad aggiungere la direttiva `define('READ_ONLY_SESSION', true);`, questo solo se siamo sicuri di non applicare modifiche alla sessione nella nostra pagina.

Dati
----

In casi con piu' istanze web bisogna trovare un modo per garantire che i dati siano gli stessi per tutte.

Per il database si tratta semplicemente di installarlo su una macchina diversa, questo permette anche di avere diverse configurazioni hardware e a livello di OS specifiche per il database. Siccome questo e' un caso d'uso normale per i database non dovrebbero esserci complessita' e' sufficiente inserire i dettagli di connessione nei file `config.php` delle singole istanze coi web server.

Per installazioni molto grandi puo' rivelarsi necessaria anche la moltiplicazione delle istanze di database, ma la loro integrazione e sincronizzazione e' argomento estraneo a questo corso.

Per la cartella moodledata  e' necessario utilizzare un file system raggiungibile da tutte le istanze, come ad esempio un drive di rete.

Siccome generalmente i drive di rete hanno performance notevolmente inferiori ad un filesystem locale, si rivelano necessari vari accorgimenti per farlo utilizzare da meno sistemi possibili.

Il settaggio principale da modificare in questo caso sono le caches, senza entrare in dettaglio qui (le vedremo meglio piu' avanti) nella sua configurazione di default moodle usa come deposito per le caches la cartella moodledata stessa, ma e' possibile indicare altri meccanismi possibili, come ad esempio un server Redis (da prevedere quindi nell'impianto), che forniscono prestazioni decisamente migliori.

Per esperienza, tipicamente le prestazioni del disco sono generalmente il collo di bottiglia piu' comune (sia su piu istanze che su una singola), con tempi di caricamento della maschera amministrazione dell'ordine di 10 secondi per una configurazione su un drive a piatti condiviso usato per le caches.

Non tutto il contenuto della cartella moodledata puo' essere messo in sistemi alternativi. e' previsto un ulteriore settaggio che va inserito nel file config.php: `$CFG->localcachedir` che permette di utilizzare una cartella sul filesystem dell'istanza del web server come cache di elementi non modificabili, come il javascript o  le librerie. Questo velocizza la visualizzazione al costo di avere delle cache non facilmente svuotabili.

Non mi risulta sia pratica comune moltiplicare la cartella moodledata, anche perche' sarebbe difficoltoso visto il fatto che e' spesso di dimensioni considerevoli.