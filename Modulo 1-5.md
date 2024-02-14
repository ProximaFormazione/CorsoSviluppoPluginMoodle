Installazione ambiente di sviluppo
==================================

Ora vedremo come configurare l'ambiente di sviluppo per moodle. L'installazione effettiva di Moodle e' indicata nel capitolo precedetne, qui ci dedicheremo invece agli strumenti necessari per sviluppare plugin aggiuntivi.

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

Su Windows consiglio di utilizzare XAMPP 

> https://www.apachefriends.org/

Per l'ambiente di sviluppo, potrebbe valere la pena di installare la versione "portable" di XAMPP, in modo da poter installare successivamente altre versioni, ad esempio se si deve lavorare con versioni diverse del PHP, i files si trovano qui

> https://sourceforge.net/projects/xampp/files/

IDE
===

Ovviamente per sviluppare vi servira' un'applicazione adeguata.

Siete liberi di utilizzare Notepad++ o Vim se ci tenete (o altro ambiente a cui siete gia' abituati), altrimenti la scelta consigliata e' [Visual Studio Code](https://code.visualstudio.com/).

VS Code e' pratico per le seguenti ragioni:
* E' gratuito
* E' molto usato
* E' altamente espandibile, ed esistono diverse estensioni per PHP e Moodle. incluse estensioni per eseguire il debug
* Funziona su Windows e Linux

Una volta che avete installato VSCode, dovrete poi installare una serie di estensioni utili per lo sviluppo:





