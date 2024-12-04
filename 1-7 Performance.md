
Analisi
=======

Per una corretto perfezionamento delle performance del sistema e' fondamentale basarsi su misurazioni accurate e frequenti. In assenza di dati effettivi non avete alcun feedback sull'efficacia delle vostre operazioni.

Plugin di benchmark
-------------------

Un primissimo strumento che potete utilizzare, su un moodle gia' esistente, e' il plugin di benchmark ([Link](https://moodle.org/plugins/report_benchmark)), questo plugin esegue dei test standard sulla piattaforma e fornisce i risultati rispetto a valori di riferimento. Questo plugin non e' assolutamente un sostituto a benchmark effettivi del sistema, ma e' utile come primo passo per identificare aree che sono grossolanamente inadeguate. 

Una volta installato, il test delle prestazioni e' accessibile in Amministrazione del sito -> report -> benchmark.

Questo plugin e' utile per identificare un area da migliorare nel nostro sistema, se tutti i valori hanno risultati sotto la media eccetto ad esempio la lettura da disco, allora concentreremo i nostri sforzi li.

Ci e' capitato di dover usare tale plugin su piattaforme cosi' problematiche da mandare il test stesso in timeout, in tal caso potete modificare il sorgente del plugin per ridurre il numero di test effettuati nel file `testlib.php` del plugin riducendo gli indici usati negli iteratori e i valori nell'array restituito di un fattore uguale.

ad esempio questo test

```php
$i = 0;
$pass = 500;
while ($i < $pass) {
    ++$i;
    $DB->get_record('course', array('id' => SITEID));
}

return array('limit' => .75, 'over' => 1, 'fail' => BENCHFAIL_SLOWDATABASE, 'url' => '');
```

diventa

```php
$i = 0;
$pass = 50;  // Ridotto di 10 volte
while ($i < $pass) {
    ++$i;
    $DB->get_record('course', array('id' => SITEID));
}

return array('limit' => .075, 'over' => 0.1, 'fail' => BENCHFAIL_SLOWDATABASE, 'url' => '');   //- ridotto limit ed over di 10 volte
```

Nella testata del file e' presente della documentazione su come aggiungere nuovi test.

JMeter
------

Apache JMeter ([Link](https://jmeter.apache.org/)) e' un software open source largamente usato per eseguire benchmark di applicazioni, tipicamente web. Puo' essere utilizzato per verificare i tempi di generazione delle pagine della piattaforma e quindi verificare le performance recepite dall'utente finale.

Potete utilizzare Jmeter per eseguire test comparativi sulla piattaforma: ripetendo lo stesso test ogniqualvolta eseguite una modifica alla configurazione del sistema avete un feedback immediato sull'efficacia dell'intervento effettuato e potete utilizzare questi dati per prendere decisioni informati su eventuali investimenti da fare sull'infrastruttura.

JMeter puo'essere configurato per effettuare piu' o meno richieste in un test, permettendo ad esempio di ripetere un test di caricamento dei contenuti del corso per 10, 100 o 10000 utenti in contemporanea e quindi di verificare un limite sperimentale per il funzionamento del vostro sito. Questo e' utile anche per verificare che la vostra piattaforma venga incontro ai requisiti richiesti dal cliente.

Moodle e' una MPA (multi page application), quindi un interazione dell'utente equivale al caricamento di una pagina con un particolare set di parametri. Questo rende molto semplice la redazione dei test in JMeter, ad esempio utilizzando il registratore di azioni integrato in JMeter

Un esempio di piano di test e' scaricabile a questo [Link](https://developerck.com/wp-content/uploads/2020/06/load-test-moodle.zip?x91701) ([]())

Sysbench
--------

Sysbench ([Link](https://github.com/akopytov/sysbench)) e'un tool utilizzabile per verificare un particolare elemento dello stack sistemistico usato, in primis del database, che di fatto e' l'elemento maggiormente configurabile.

Esiste notevole documentazione online su come utilizzare sysbench per analizzare qualsiasi aspetto del sistema come il processore, la memoria o i dischi, vediamo qui alcuni esempi di utilizzo utili per configurazioni comuni di moodle.

> ATTENZIONE: Questi test mettono il sistema sotto stress e non vanno utilizzati in un ambiente di produzione

[Link a manuale](https://imysql.com/wp-content/uploads/2014/10/sysbench-manual.pdf)

### MariaDb

Possiamo utilizzare sysbench per avere una metrica della performance del nostro database, il che e'utile per verificare l'effetto delle configurazioni effettuate.

Possiamo utilizzare i test preconfigurati per avere una misura base del numero di query al secondo eseguibili:

1. creiamo un database per i test chiamato `sysbench`

```
mysql -u root -pXXXX -e 'CREATE DATABASE sysbench'
```

2. Prepariamo il database di prova per il test 

```
sysbench oltp_common --db-driver=mysql --table-size=100000 --mysql-user=root --mysql-password=XXXX --mysql-port=3306 --mysql-host=localhost --mysql-db=sysbench prepare
```

questo crea una tabella di test di circa 20-30 MB

3. Eseguiamo il test di lettura

```
sysbench oltp_read_write --db-driver=mysql --table-size=100000 --mysql-user=root --mysql-password=XXXX --mysql-port=3306 --mysql-host=localhost --mysql-db=sysbench --threads=12 --report-interval=1 run
```

questo ci fornisce un report delle prestazioni rilevate sul database, con un valore medio separato per le operazioni di lettura e scrittura. Possiamo utilizzare questo test per verificare gli effetti delle nostre configurazioni.

### Filesystem

Possiamo anche utilizzare sysbench per avere dei benchmark sulle operazioni di lettura e scrittura su disco, utile per avere dei dati oggettivi in caso di valutazione di diverse soluzioni di storage.

Vediamo un esempio di configurazione del test:

1. Prepariamo il test generando die file di prova

```
sysbench fileio --file-total-size=15G prepare
```

Potete selezionare valori diversi per `--file-total-size` a seconda dello spazio a disposizione, e' consigliato scegliere un valore superiore alla memoria a disposizione. `--file-test-mode=rndrw` indica scrittura e lettura casuale da file. Per un elenco completo dei settaggi potete consultarli direttamente da sysbench con il comando `sysbench fileio help` o nel manuale di sopra

2. Eseguite il test con il comando 

```
sysbench fileio --file-total-size=15G --file-test-mode=rndrw --time=300 --max-requests=0 run
```

3. (OPZIONALE) Per rimuovere i files di prova utilizzare il comando

```
sysbench fileio cleanup
```

### Altro

Sono presenti altri test preimpostati, come ad esempio:

```
sysbench --test=cpu --cpu-max-prime=20000 --num-threads=2 run
sysbench --test=memory --num-threads=4 run
sysbench --test=threads --thread-locks=1 --max-time=20s run
```

Ed e' possibile importare test creati da terze parti, o sviluppare lo script. I test sono conservati nella cartella di sysbench (su ubuntu: `/usr/share/sysbench/`).

Tendenzialmente si tratta di test meno rilevanti, in quanto processore e RAM generalmente hanno una sola via per essere migliorati, ovvero metterne di piu' a disposizione.

Tuning
======

Una volta stabilito un solido framework di benchmark, utilizzando ad esempio gli strumenti di sopra, diventa possibile iniziare a lavorare sul miglioramento delle performance del sito.

L'argomento e' senza dubbio complesso, con tante variabili che dipendono dai sistemi utilizzati, qui ci soffermeremo su alcune considerazioni di carattere generale.

Per ulteriori informazioni, o per tecnologie non affrontate si rimanda alla documentazione ufficiale ([Link](https://docs.moodle.org/405/en/Performance_recommendations))

Selezionare l'architettura
--------------------------

Per prima cosa e' necessario avere in mente se l'installazione richiede molteplici istanze o no.

Impostare un architettura che puo' scalare orizzontalmente ha i suoi vantaggi, come il fatto di poter gestire carichi molto alti ed un utilizzo piu' efficiente delle risorse (se previsto un meccanismo di orchestratura delle istanze), tuttavia la separazione tra i sistemi comporta un overhead a livello di prestazioni, per cui e' sconsigliato optare per una infrastruttura di questo genere per installazioni che potrebbero benissimo funzionare su una singola macchina.

E' importante avere un idea dell'uso previsto della piattaforma. Il vero discrimine e' il numero di **utenti attivi in contemporanea**, ovvero utenti che stanno richiedendo pagine o comunque impegnando risorse del server, non e' un numero facilmente stimabile perche' dipende da diversi fattori:

- La platea possibile di utenti
- La tipologia di attivita': pagine statiche impegnano il server per l'apertura e consulta i materiali a lungo senza interagire piu' con il server, un quiz invece richiede continue operazioni sul database
- La Reattivita' del server. Se un operazione impegna 0.1 secondi allora 10 utenti corrispondono ad un carico di 10 utenti/sec. Se la stessa operazione impiega 1 secondo allora il carico effettivo e' di 10 volte tanto

Una stima grossolana, ma tipicamente usata per le installazioni e' predisporre almeno 50MB di memoria per utente in contemporanea che va servito, quindi una piattaforma con 4GB dovrebbe andare bene per un carico massimo di 4000/50 = 80 utenti. In realta' Moodle e' particolarmente vorace di RAM e non e'escluso che un singolo utente (specie un manager di corsi) possa utilizzarne di piu', ma questa stima e' utile per avere un idea di come provisionare le macchine.

Altro elemento utile da ottenere a livello di requisiti, e' il tipo di contenuto che si intende caricare. Se un requisito e' l'avere contenuti multimediali caricati sulla piattaforma (piuttosto che sistemi terzi tipo youtube) allora dovrete prevedere uno storage per la cartella moodledata dimensionato in base al numero di corsi previsti.

In caso di piu' istanze sara' opportuno provisionare tutte le macchine necessarie:

- I web servers, statici o gestiti da un orchestratore (es kubernetes), con eventuale reverse proxy
- Una macchina per il database
- Una macchina per il database delle cache (es Redis)
- Un filesystem accessibile ai web server (es:drive di rete)

Selezionare le tecnologie 
-------------------------

Spesso la decisione delle tecnologie da usare non e'in mano nostra, ma laddove possibile e' meglio utilizzare delle configurazioni raccomandate:

Il **sistema operativo** consigliato e' Linux, non vi e' una distribuzione raccomandata ma lo sviluppo avviene principalmente in ambienti debian quindi ad ad esempio Ubuntu 22.04 e' una buona scelta. Linux e' raccomandato non solo per la sua performance, ma per il fatto che buona parte dello stack utilizzato (come il PHP) e' sviluppato prevalentemente in Linux e su Windows sono disponibili dei port. 

Per il **Web server** non vi e' un consenso generale. Nginx e' la scelta piu'usata, ma Apache e' una scelta altrettanto valida. In ambienti Windows e' meglio utilizzare IIS

Per il database la scelta nativa e' **MariaDb**, sulla quale e' disponibile notevole documentazione online

per la **cartella dati** sara' opportuno identificare uno storage almeno SSD, preferibilmente un nvme. Altra opzione migliorativa sono una struttura RAID sui dischi, specie se non si dispone di drive nvme. E essendo la preoccupazione principale la velocita' di lettura qualsiasi livello di RAID comunemente usato va bene. In caso di drive condivisi via rete sara' opportuno usare un protocollo nativo per il sistema operativo usato: NFS per linux ed SMB per Windows (Questa scelta e' rilevante in caso di architetture su cloud)

Un motore per le **caches in RAM** e' necessario con piu' istanze, qui la scelta di mercato piu'comune e' Redis, ma vi e' ampia documentazione anche per Memcached

Configurazioni di moodle
------------------------

Alcuni accorgimenti possono essere presi all'interno di moodle per ridurre eventuali problemi di performance

### Caches

La prima cosa da fare in caso di architettura a piu'istanze e' selezionare un adeguato motore per le caches. Nella sua configurazione di default moodle salva le caches nella cartella dati, che pero' se e' un drive di rete ha performance assolutamente non adeguate.

Assumendo di avere a disposizione un sistema pensato allo scopo, come redis, dobbiamo andare a configurare moodle per utilizzarlo come storage delle caches (da non confondere con lo storage delle sessioni, che anch'esso andra' su redis). Nel resto della guida assumiamo redis come gestore delle cache ma gli stessi passaggi vanno bene anche per altri gestori scelti

La configurazione delle caches avviene in Amministrazione del sito -> Plugin -> caching -> Configurazione.

La maschera e' sicuramente intimidatoria, ma per configurare un nuovo motore per le caches e' sufficiente:

1. nella prima tabella, sulla riga redis cliccare su aggiungi istanza per configurare i dettagli di connessione. Se il pulsante non compare dovete installare l'estensione del php per redis `php-redis`

![immagine](https://docs.moodle.org/405/en/images_en/9/9d/Redis_cache_ready.png)

2. dopo aver configurato l'istanza, scorrete oltre il listone delle caches fino all'ultima tabella in fondo alla pagina e cliccate su "modifica mappature"

3. Qui potete indicare lo store di default utilizzato, selezionate lo store redis sia per application che per session

![Immagine](https://docs.moodle.org/405/en/images_en/3/30/Set_Redis_as_default_Application_and_Session_cache.png)

E'disponibile una pagina specifica in moodle per il benchmark delle caches, sempre in Amministrazione del sito -> Plugin -> caching. Specificatamente per redis dovete indicare il server usato per il test nella configurazione del plugin 

### Altro

Altri consigli utili:

Vale la pena inserire dei limiti per la conservazione dei log ed i dati delle valutazioni. Questi possono essere impostati in Server -> Impostazioni di pulizia ed in Plugin -> Logging -> Log Standard. Noi generalmente impostiamo 1000 gg.

Controllare che il backup automatico dei corsi sia disattivato. Siccome avrete una soluzione di backup per il sistema questa feature e' ridondante

Disattivate la raccolta delle statistiche in Amministrazione del sito -> Funzionalita' avanzate. Si tratta di una funzionalita' per la generazione di report sull'uso generale della piattaforma. I report sono interessanti ma la feature puo' risultare impegnativa per il sito

In Plugin -> Filtri, disattivare i filtri che non sono richiesti o di cui si puo' fare a meno. I Filtri vengono applicati ovunque ci sia del testo quindi sono molto impattanti 

PHP
---

Assicuratevi che OpCache sia abilitato

Valutare se aumentare il valore di `memory_limit` nel PHP. Moodle puo' richiedere un gran quantitativo di memoria per alcune operazioni di amministrazione, come la duplicazione dei corsi. Nella guida precedente abbiamo gia' alzato il valore a 256M ma potete valutare di andare oltre

MariaDb
-------

Il settaggio che puo' avere un enorme influenza sulle prestazioni del database MariaDb (e MySql) e' il settaggio della dimensione del buffer di memoria di InnoDb ([Documentazione ufficiale](https://dev.mysql.com/doc/refman/5.7/en/innodb-buffer-pool.html)). Essenzialmente e' spazio che in memoria che il database utiliza come cache per le query eseguite, piu' e' ampio piu' e' probabile che risulti utile alla piattaforma

L'argomento e' sofisticato e richiede attenta valutazione per trovare il valore ottimale, quello che e' sicuro e' che il valore di default di 128MB non e' adeguato. Se il server e' dedicato unicamente al database e' raccomandazione comune dedicare a questo settaggio almeno il 70% della memoria totale del sistema. Altrimenti sicuramente assegnare almeno 1-2 GB dovrebbe comportare un notevole miglioramento

Il settaggio va inserito nel file di configurazione di mariaDb (es `/etc/mysql/mariadb.conf.d/50-server.cnf`) e si chiama `innodb_buffer_pool_size`

```
innodb_buffer_pool_size=2000M
innodb_log_file_size=500M
skip-name-resolve=1
```

`innodb_log_file_size` e' un settaggio correlato che si raccomanda impostare al 25% della dimensione del pool

`skip-name-resolve` salta il controllo dell'hostname, impedendo quindi di differenziare gli utenti in base all'hostname. Siccome questo non ci interessa possiamo disattivarlo per evitare uno step inutile ad ogni query 

Questo e' l'unico consiglio facile riguardo MariaDb. Ci sono altre considerazioni da fare che dipendono dal vostro sistema. Segnalo solo che il settaggio `query_cache_type` e' meglio lasciare disattivato in quanto comporta riduzioni di performance per tabelle con modifiche frequenti.

Per un'ulteriore analisi potete utilizzare MySQLTuner ([Link](https://github.com/major/MySQLTuner-perl)) che e' uno script di controllo automatico delle prestazioni che si porta dietro alcuni suggerimenti preimpostati, per scaricarlo

```
wget http://mysqltuner.pl/ -O mysqltuner.pl
wget https://raw.githubusercontent.com/major/MySQLTuner-perl/master/basic_passwords.txt -O basic_passwords.txt
wget https://raw.githubusercontent.com/major/MySQLTuner-perl/master/vulnerabilities.csv -O vulnerabilities.csv
```

e per eseguirlo

```
perl mysqltuner.pl --host 127.0.0.1
```
