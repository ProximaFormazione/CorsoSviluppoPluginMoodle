Installazione ambiente di sviluppo
==================================

Ora vedremo come configurare l'ambiente di sviluppo per moodle. L'installazione effettiva di Moodle e' indicata nel capitolo precedente, qui ci dedicheremo invece agli strumenti necessari per sviluppare plugin aggiuntivi.

Questa guida e' per l'installazione in un ambiente windows, ma passaggi analoghi possono essere eseguiti anche su ambienti linux.


Preliminari
===========

Queste operazioni vanno eseguite, se ancora non sono state fatte in fase di installazione di moodle

Installare Git per Windows
--------------------------

Git e' gia' fondamentale per la gestione degli aggiornamenti delle piattaforme, ma ancora piu' importante in fase di sviluppo.

Installare dal link:

> [https://git-scm.com/download/win](https://git-scm.com/download/win)

Su linux installare tramite apt o simili

Installare moodle e lo stack
----------------------------

Se non lo si ha gia' fatto, procedere ad installare un web server, un database, e soprattutto il PHP necessari per moodle. Ci sono due opzioni: potete installare lo stack direttamente sul vostro computer o utilizzare un container/VM

### In locale

Su Windows, a meno di non avere gia' il PHP funzionante (ad esempio se si e' seguita la guida del modulo precedente), consiglio di utilizzare XAMPP per il web server, in quanto si porta un ambiente PHP gia' funzionante

> https://www.apachefriends.org/

Per l'ambiente di sviluppo, potrebbe valere la pena di installare la versione "portable" di XAMPP, in modo da poter installare successivamente altre versioni, ad esempio se si deve lavorare con versioni diverse del PHP, i files si trovano qui

> https://sourceforge.net/projects/xampp/files/

Sebbene XAMPP includa un database MariaDB, sconsiglio di utilizzarlo in quanto l'implementazione fatta su XAMPP e' prona a rompersi in caso di arresto improvviso, richiedendo correzioni manuali sui files e facendo perdere tempo inutilmente. Invece raccomando di installare mariaDB a parte, ([Link](https://mariadb.org/download/?t=mariadb&p=mariadb&r=11.4.4&os=windows&cpu=x86_64&pkg=msi&mirror=mva)) operazione abbastanza semplice visto che e' disponibile un installer ufficiale.

Su Linux potete seguire la guida di installazione del modulo precedente.

### Virtualizzato

Un altra opzione e' installare una macchina virtuale o un container con l'installazione eseguita.

Per le VM, potete seguire la procedura guidata del modulo precedente. Raccomando di eseguire una copia del disco prima di eseguire l'installazione da interfaccia di moodle (ovvero prima di avere il config.php) in modo da poter utilizzare tale copia per replicare velocemente altre piattaforme. In questo modo potete clonare l'immagine, collegarvi al moodle e lasciare che la procedura guidata inserisca l'host corretto.

In locale Moodle non richiede troppe risorse. Come requisiti minimi moodle propone 1 core con 512MB di RAM. Consiglio di fornire almeno 5-20 GB di spazio per il disco, e se possibile di fornire 1-2 GB di RAM.

Se utilizzate docker sono disponibili immagini preconfezionate, come ad esempio quelle di [BitNami](https://hub.docker.com/r/bitnami/moodle), o i [file di configurazione forniti direttamente dalla Moodle](https://github.com/moodlehq/moodle-docker) (da non confondere con moodleapp che ha un hub su docker, ma che e' l'app mobile). Per l'installazione si rimanda alla documentazione delle stesse.

Per l'immagine di bitnami, dopo averla scaricata dovete aggiungere un istanza di database, come mariadb di bitnami, ed attivare la persistenza dei dati. Se volete eseguire l'installazione nel modo piu' semplice possibile potete usare il file di docker compose proposto da bitnami ([Link](https://raw.githubusercontent.com/bitnami/containers/main/bitnami/moodle/docker-compose.yml)):

```
wget https://raw.githubusercontent.com/bitnami/containers/main/bitnami/moodle/docker-compose.yml
docker compose up -d
```

Al primo avvio l'immagine esegue l'installazione, quindi potrebbero volerci alcuni minuti prima che la piattaforma sia raggiungibile, potete controllare lo stato nel log.

XDebug
======

Per eseguire sessioni di debug, fermando l'esecuzione del codice a comando per ispezionare le variabili, e' necessario attivare l'estensione **XDebug** del PHP.

Per installarlo, sul sito ufficiale di XDebug e' presente una [pagina con la procedura guidata](https://xdebug.org/wizard) che e' in grado di indirizzarvi alla versione corretta da scaricare, e vi fornisce pure informazioni personalizzate.

Per funzionare vi richiede l'output generato dalla funzione `phpinfo()`. Potete ottenere questo output creando un file con estensione .php, diciamo `phpinfo.php` con all'interno il seguente codice:

```
<?php phpinfo( ); ?>
```

copiate il file nella root del webserver (su xampp: `XAMPP\htdocs`) e navigate con il browser a `localhost/phpinfo.php`.

Alternativamente nella versione installata di moodle potete trovare una voce "PHP info" nel menu' "Server" in amministrazione

Su windows l'installazione consiste nello scaricare una dll precompilata, copiarla nella cartella delle estensioni del PHP (es: `C:\xampp\php\ext`) ed abilitarla in php.ini

Su linux la procedura e' piu' complessa perche' bisogna compilare l'estensione, fortunatamente la guida che vi fornisce il wizard di XDebug e' molto precisa (con i comandi specifici per il vostro server), seguitela con attenzione e dovreste farcela senza problemi

All'interno del file php.ini dovrete eseguire la configurazione del blocco [XDebug], ad esempio con:

```
[XDebug]
zend_extension= "{...}/php/ext/php_xdebug.dll"
xdebug.client_port = 9090
xdebug.mode = debug
xdebug.start_with_request = yes
```

questa configurazione funziona per le varie versioni di xdebug in locale. Tenete presente il numero della porta che inserite (o lasciate vuoto per il default 9003). Per eseguire debug in remoto potremmo dover ritornare su questo punto.

IDE
===

Ovviamente per sviluppare vi servira' un'applicazione adeguata.

Siete liberi di utilizzare Notepad++ o Vim se ci tenete (o altro ambiente a cui siete gia' abituati), altrimenti la scelta consigliata e' [Visual Studio Code](https://code.visualstudio.com/).

VS Code e' pratico per le seguenti ragioni:
* E' gratuito
* E' molto usato
* Ha un'integrazione con Git
* E' altamente espandibile, ed esistono diverse estensioni per PHP e Moodle. incluse estensioni per eseguire il debug
* Funziona su Windows e Linux

Una volta che avete installato VSCode, dovrete poi installare una serie di estensioni utili per lo sviluppo:

* [PHP Intelephense](https://marketplace.visualstudio.com/items?itemName=bmewburn.vscode-intelephense-client) per avere alcune funzionalita' di intellisense
* [PHP Debug](https://marketplace.visualstudio.com/items?itemName=xdebug.php-debug) per collegarsi all'estensione xDebug del PHP (debugger PHP)

Sono poi presenti diverse estensioni con snippet gia' pronti come [Moodle Pack](https://marketplace.visualstudio.com/items?itemName=imgildev.vscode-moodle-snippets) ed esiste una estensione specializzata per lo sviluppo, [MDLCode](https://marketplace.visualstudio.com/items?itemName=LMSCloud.mdlcode) (a pagamento), che ha moltissime utili funzionalita'. Questa ultima estensione e' estremamente utile nel lavoro quotidiano e vale l'investimento se si sviluppa regolarmente. In questa guida si raccomanda pero' di NON installarla nello svolgimento del corso in modo da prendere miglior dimestichezza con i sistemi e di installarla solo successivamente.

Se si lavora su macchine remote o con container sono caldamente consigliate le seguenti estensioni:

* [Remote - SSH](https://marketplace.visualstudio.com/items?itemName=ms-vscode-remote.remote-ssh) Per collegarsi in SSH
* [Dev Containers](https://marketplace.visualstudio.com/items?itemName=ms-vscode-remote.remote-containers) Per i container docker

Maggiori informazioni su come utilizzarle sono presenti nella sezione "Debug" sotto.

Debug
-----

Per eseguire un debug con VS Code e' necessario creare un profilo per il debugger, potete farlo dal tab "Run and debug". se cliccate sul link "create a launch.json file" con un file php aperto (e l'estensione di sopra installata) vi crea un template adatto, voi dovrete poi entrare nel file launch.json ed editare la porta usata nella configurazione di xdebug

```
"configurations": [
        {
            "name": "Listen for Xdebug",
            "type": "php",
            "request": "launch",
            "port": 9090
        },

        ...
```

Potete rimuovere gli altri profili a meno che non vogliate farne uso.

Debug in remoto
---------------

Per eseguire il debug in remoto, ad esempio se abbiamo installato moodle su un docker, abbiamo due possibili approcci:

Il primo e' configurare il server che fa girare moodle per contattare il server dove si trova l'IDE nell'esecuzione dei debug. La seconda e' configurare l'IDE per lavorare remotamente in SSH sulla macchina con moodle.

Quale usare dipende dal vostro caso d'uso, e a seconda dell'IDE che utilizzate una o entrambe potrebbero non essere disponibili, o potrebbero esistere alternative. In questa sezione ci concentreremo su **VS Code** come IDE.

La prima opzione ha la controindicazione di richiedere l'allineamento tra i files del server e quelli dell'IDE, che in assenza di meccanismi di CI/CD si preannuncia una grande scomodita'. La seconda ha come controindicazione che esegue delle installazioni sul server (per VS Code, potenzialmente altri IDE hanno approcci meno invasivi), inoltre richiede di potersi collegare in SSH (non scontato su macchine Windows).

Per lo sviluppo su docker raccomando la seconda opzione in quanto funziona e basta, e non richiede duplicazione di sorgente. La prima opzione e' interessante per eseguire debug in ambienti non di sviluppo (ricordate che xDebug riduce drasticamente le prestazioni e non va lasciato attivo in produzione)

### opzione 1

Per fare contattare la macchina dove si trova l'IDE da xdebug dovete aggiungere questa riga al php.ini:

```
xdebug.discover_client_host = 1
```

Questo fa si che XDebug riconosca automaticamente l'host chiamante ed inizi con lui la sessione di debug (da notare che e' XDebug a contattare l'IDE), questa configurazione e' la piu' semplice e non richiede attenzioni. Altrimenti si puo' specificare un ip con `xdebug.client_host = XXX.XXX.XXX.XXX`. Per altri casi particolari si rimanda alla documentazione di XDebug([link](https://xdebug.org/docs/step_debug)) che e' molto esaustiva.

Bisogna poi configurare l'IDE in modo che esegua la mappatura tra i files del server con i files in locale (che devono coincidere o rischiate di mancare i breakpoint), in VS code bisogna aggiungere questa configurazione al file launch.json nel profilo che volete utilizzare

```
{
    "name": "Listen for Xdebug",
    ...
    "pathMappings": {
        "/var/www/html/moodle": "${workspaceFolder}"
    }
},

```
ovviamente dovrete cambiare i path per la vostra situazione, a sinistra sono quelli sul server a destra quelli sull'IDE. `${workspaceFolder}` e' una variabile utilizzabile.

### opzione 2

Per lavorare in remoto con VS code dovete installare l'estensione **Remote - SSH** in VS Code. Questa estensione vi permette di collegarvi in ssh ad un server remoto ed aprire un workspace con i files li presenti.

Una volta installata, potete collegarvi ad un server digitando ssh-remote, o usando l'icona dell'ssh in basso a sinistra e selezionando la voce corretta dalla command palette (la barra di ricerca in alto al centro).

Da notare che questa estensione installa sulla macchina remota un miniserver per visual studio code ([link](https://code.visualstudio.com/docs/remote/vscode-server)), necessario per il funzionamento del meccanismo. Questo fa si che alcune estensioni, in particolare il debugger PHP, vadano reinstallate nell'ambiente remoto.

Per fare cio' e' sufficiente aprire il tab delle estensioni, dove vedrete le estensioni che vanno reinstallate, ed installarle. Questo le installa nel miniserver sulla macchina remota. Trattandosi di un altro ambiente potreste dover modificare configurazioni o ottenere comportamenti diversi per il nuovo ambiente.

Una volta connessi alla macchina remota possiamo cercare la cartella con il sorgente semplicemente facendo "open folder" (la navigazione avviene nella command palette) ed aprire la cartella con la nostra installazione. Qui possiamo configurare XDebug usando la stessa procedura per l'installazione su una macchina locale.

#### Con Docker

Entrambe le opzioni sono utilizzabili con docker, ma la loro implementazione richiede la modifica delle immagini per abilitare XDebug e/o altri requisiti (es abilitare connessioni SSH).

VSCode prevede un'altra estensione: "Dev Containers" che semplifica il collegamento con i container di docker. Una volta installata e' possibile collegarsi ai container attivi cliccando sul menu' in basso a sinistra e/o selezionando la voce dalla command palette **Attach to running container...**, di fatto questa e' identica all'opzione 2 di sopra ma utilizzando un altro plugin di VSCode.

Sull'immagine di bitnami, XDebug e' installata ma va attivata nel file di configurazione del PHP (in `/opt/bitnami/php/etc/php.ini`). Raccomando di utilizzare la configurazione di sopra per le macchine di sviluppo piuttosto che semplicemente decommentare le opzioni presenti

In breve, se lavorate con l'immagine di bitnami:

1. Installare l'estensione di VSCode **Dev Containers**
2. Cliccare sull'icona di connessione remota in basso a sinistra e selezionare **Attach to running container...**
3. Indicare il container con moodle
4. File -> Open folder e selezionare la cartella con moodle (es `/bitnami/moodle`)

In Aggiunta, per attivare il debug:

5. modificare il file di configurazione del PHP (`/opt/bitnami/php/etc/php.ini`) per attivare XDebug. (Su bitnami XDebug e' inclusa, altrimenti dovete eventualmente installarla). Dopo aver modificato il file riavviate il servizio fpm (o il container)

```
[XDebug]
zend_extension = xdebug
xdebug.mode = debug
xdebug.start_with_request = yes
```

6. Installare su VSCode aperto sulla cartella remota l'estensione **PHP Debug**, va installato in remoto indipendentemente dal fatto che lo abbiate in locale
7. Generare un file launch.json dal tab "Debug" di visual studio, indicare *PHP* per avere una configurazione funzionante









