Installazione ambiente di sviluppo
==================================

Ora vedremo come configurare l'ambiente di sviluppo per moodle. L'installazione effettiva di Moodle e' indicata nel capitolo precedente, qui ci dedicheremo invece agli strumenti necessari per sviluppare plugin aggiuntivi.

Questa guida e' per l'installazione in un ambiente windows, ma passaggi analoghi possono essere eseguiti anche su ambienti linux

Preliminari
===========

Queste operazioni vanno eseguite, se ancora non sono state fatte in fase di installazione di moodle

Installare Git per Windows
--------------------------

Git e' gia' fondamentale per la gestione degli aggiornamenti delle piattaforme, ma ancora piu' importante in fase di sviluppo.

Installare dal link:

> [https://git-scm.com/download/win](https://git-scm.com/download/win)

Su linux installare tramite apt o simili

Installare lo stack LAMP
------------------------

Se non lo si ha gia' fatto, procedere ad installare un web server, un database, e soprattutto il PHP necessari per moodle.

Su Windows, a meno di non avere gia' il PHP funzionante, consiglio di utilizzare XAMPP 

> https://www.apachefriends.org/

Per l'ambiente di sviluppo, potrebbe valere la pena di installare la versione "portable" di XAMPP, in modo da poter installare successivamente altre versioni, ad esempio se si deve lavorare con versioni diverse del PHP, i files si trovano qui

> https://sourceforge.net/projects/xampp/files/

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

All'interno del file php.ini dovrete eseguire la configurazione del blocco [XDebug], ad esempio con:

```
[XDebug]
zend_extension= "{...}/php/ext/php_xdebug.dll"
xdebug.client_port = 9090
xdebug.remote_enable = 1
xdebug.remote_autostart = 1
xdebug.mode = debug
xdebug.start_with_request = yes
```

questa configurazione funziona per le varie versioni di xdebug. Tenete presente il numero della porta che inserite

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
* [PHP Debug](https://marketplace.visualstudio.com/items?itemName=xdebug.php-debug) per collegarsi all'estensioe xDebug del PHP (debugger PHP)

Sono poi presenti diverse estensioni con snippet gia' pronti come [Moodle Pack](https://marketplace.visualstudio.com/items?itemName=imgildev.vscode-moodle-snippets) e [MDLCode](https://marketplace.visualstudio.com/items?itemName=LMSCloud.mdlcode) (potente, ma a pagamento)

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






