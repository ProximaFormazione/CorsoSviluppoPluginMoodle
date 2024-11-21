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

La configurazione delle singole istanze non richiede particolari attenzioni, il puntamento per database e dati dovrebbe essere identico, e laddove avete un reverse proxy anche l'hostname risultera' lo stesso (quello del proxy). Il reverse proxy non e' obbligatorio potete anche avere diverse istanze indipendenti raggiungibili con indirizzi diversi (magari per platee diversificate), in tal caso ogni istanza avra' il proprio parametro `wwwroot` indicato nel file `config.php`

Vi sono pero' delle feature che non funzioneranno piu' correttamente: principalmente l'installazione di plugin via interfaccia, in quanto agirebbe unicamente sull'istanza raggiunta con il browser web. Nei moodle a piu' istanze le installazioni di plugin vanno gestite esternamente caricando direttamente la cartella  su tutte le istanze (di solito modificando il repository dal quale vengono create). 

Non mi sovvengono altre attenzioni particolari. In caso di plugin terzi bisogna verificare che non applichino modifiche ai files del sito affinche siano compatibili con questa configurazione

Dati
----

In casi con piu' istanze web bisogna trovare un modo per garantire che i dati siano gli stessi per tutte.

Per il database si tratta semplicemente di installarlo su una macchina diversa, questo permette anche di avere diverse configurazioni hardware e a livello di OS specifiche per il database. Siccome questo e' un caso d'uso normale per i database non dovrebbero esserci complessita' e' sufficiente inserire i dettagli di connessione nei file `config.php` delle singole istanze coi web server.

Per installazioni molto grandi puo' rivelarsi necessaria anche la moltiplicazione delle istanze di database, ma la loro integrazione e sincronizzazione e' argomento estraneo a questo corso.

Per la cartella moodledata  e' necessario utilizzare un file system raggiungibile da tutte le istanze, come ad esempio un drive di rete.

Siccome generalmente i drive di rete hanno performance notevolmente inferiori ad un filesystem locale, si rivelano necessari vari accorgimenti per farlo utilizzare da meno sistemi possibili.

Il settaggio principale da modificare in questo caso sono le caches, senza entrare in dettaglio qui (le vedremo meglio piu' avanti) nella sua configurazione di default moodle usa come deposito per le caches la cartella moodledata stessa, ma e' possibile indicare altri meccanismi possibili, come ad esempio un server Redis (da prevedere quindi nell'impianto), che forniscono prestazioni decisamente migliori.

Per esperienza, tipicamente le prestazioni del disco sono generalmente il collo di bottiglia piu' comune (sia si piu istanze ch su una singola), con tempi di caricamento della maschera amministrazione dell'ordine di 10 secondi per una configurazione su un drive a piatti condiviso usato per le caches.

Non tutto il contenuto della cartella moodledata puo' essere messo in sistemi alternativi. e' previsto un ulteriore settaggio che va inserito nel file config.php: `$CFG->localcachedir` che permette di utilizzare una cartella sul filesystem dell'istanza del web server come cache di elementi non modificabili, come il javascript o  le librerie. Questo velocizza la visualizzazione al costo di avere delle cache non facilmente svuotabili.

Non mi risulta sia pratica comune moltiplicare la cartella moodledata, anche perche' sarebbe difficoltoso visto il fatto che e' spesso di dimensioni considerevoli.