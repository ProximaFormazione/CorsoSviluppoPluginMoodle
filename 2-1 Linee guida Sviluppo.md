Linee Guida
===========

Moodle e' un progetto open source, se si desidera partecipare all'interno della comunita' di sviluppatori e' necessario aderire a delle precise regole di codice.

> Documentazione completa [Qui](https://moodledev.io/general/development/policies/codingstyle)

Ovviamente queste sono solo linee guida, siete liberi di seguire lo stile di codice che preferite.

Alcuni elementi interessanti:

* viene richiesto di non chiudere i tag php in files  solo codice (la stragrande maggioranza)
* viene suggerito di usare solo nomi minuscoli separati da underscore (Snake case)
* viene suggerito di avere un massimo di caratteri per riga di non oltre 180 caratteri
* viene suggerito di non inserire chiamate a funzioni come parametri, ma di usare variabili intermedie 

Sono regole utili ad aumentare la leggibilita' del codice. Se provate a fare delle pull request su repository di moodle senza seguire queste regole preparatevi a dover correggere tutte queste minuzie.

Le uniche prassi che hanno conseguenze operative sono:

* nome del plugin
* nome dei files base di un plugin, e delle cartelle

Nomi plugin e namespaces
------------------------

La nomenclatura di un plugin deve rispettare la forma:

> `tipo_nome`

Dove `tipo` e' la tipologia di plugin (es: local, report, auth) e `nome` e' il nome del plugin, dove sono permessi solo lettere ed underscore. Consiglio di scegliere nomi brevi perche' finiscono in tutti i namespace e nelle tabelle (fino alla 4.2 c'e' un limite di caratteri stringente)

Questa nomenclatura del dominio e' riferite nelle documentazioni di moodle come nomenclatura [FrankenStyle](https://moodledev.io/general/development/policies/codingstyle/frankenstyle) e va rispettata perche' alcuni elementi di moodle dipendono da essa.

le attivita' dei corsi hanno tipo `mod`, che e' l'unico tipo ad avere delle eccezioni nelle regole di uso del nome del plugin (ovvero in certi casi *mod* andra' omesso)

il nome completo del plugin va usato nelle seguenti situazioni:

* come prefisso nelle funzioni che non sono sotto namespace
* come prefisso nei namespaces
* come prefisso per le tabelle del database

nel resto della guida indicheremo questo prefisso con `tipo_nome`

Files e cartelle richieste
==========================

tutti i files di un plugin sono contenuti in una cartella con il nome del plugin stesso, la quale va sistemata nella cartella corrispondente al tipo di plugin (quindi il plugin di iscrizione "self" andra' al percorso `/enrol/self`).

All'interno della cartella devono poi essere presenti alcuni files necessari. 

Come minimo indispensabile vi serve:

* un file `version.php` nella root del plugin con la versione attuale ed il nome del plugin
* un file con le stringhe di testo in inglese, al percorso `lang/en/tipo_nome.php` con almeno la stringa con il nome del plugin

Inoltre, generalmente vi serviranno anche:

* Se avete tabelle sul database, un file `db/install.xml` con le definizioni delle stesse
* Una cartella `classes` per abilitare l'autocaricamento delle classi in base ai namespaces usati
* In file con i settaggi del plugin `settings.php`
* Un file per definire i tipi di permessi possibili `db/access.php`
* Un file `lib.php` che contiene funzioni particolari richieste o volute per collegarsi a specifici sistemi di moodle
* Un file per definire come aggiornare il database tra due versioni del plugin, se richiesto `db/upgrade.php`
* altri files a seconda della tipologia di plugin

Inoltre varie funzionalita' di moodle richiederanno di censire degli appositi array in files specifici, ad esempio:

* consumare gli eventi di moodle: `db/events.php`
* operazioni schedulate: `db/tasks.php`
* web services: `db/events.php`
* caches: `db/caches.php`