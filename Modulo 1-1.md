Introduzione + Tecnologie Utilizzate
============

*breve presentazione personale del docente.*

Moodle (acronimo di Modular Object-Oriented Dynamic Learning Environment, ambiente per l'apprendimento modulare, dinamico, orientato ad oggetti) è un ambiente informatico per la gestione di corsi, ispirato al costruzionismo, teoria secondo la quale ogni apprendimento sarebbe facilitato dalla produzione di oggetti tangibili.

Il suo software è scritto in PHP e JavaScript; è open source e modulare, permettendo quindi a qualunque gruppo di utenti di sviluppare funzionalità aggiuntive personalizzate.

Unico requisito di sistema per funzionare e' un web server che supporti il php ed un database.

Moodle e' una piattaforma LMS (Learning management system) che permette agli utenti di partecipare in "Corsi" nelle quali partecipano ad una serie di "Attivita'" preparate dal creatore del corso, idealmente imparando qualcosa nel processo.

![Esempio corso](https://docs.moodle.org/403/en/images_en/b/bb/Boost40course.png)

Essendo un progetto open source esistente da oltre 20 anni vi e' un'ampia comunita' di utilizzatori e contributori online, e sono disponibili ampie documentazioni online.

Moodle e' progettato specificatamente per essere esteso tramite l'installazione di plugins. Nel corso degli anni diversi plugin popolari sono stati inclusi nell'installazione base di moodle. Questo causa anche una certa differenza di stili e strategie nel codice.

Esempio pratico
---------------

Vediamo un esempio di corso online, collegandoci al seguente url abbiamo il materiale di questo corso:

> [link](https://www.e-prox.it/course/view.php?id=1079)

Da notare che in questo caso e' necessaria una login per accedere, il che e' preferibile laddove si vuole identificare l'attivita' dello studente, ma e' possibile fare accedere gli utenti in maniera anonima se richiesto. Chiaramente il corso in questo caso deve essere "stateless"

Qui possiamo vedere la struttura di un corso: lo studente si trova all'interno una serie di **Attivita** eventualmente divisi in gruppi/argomenti. Le attivita' possono essere le piu' disparate e vanno da semplici blocchi di testo, a contenuti quali video, audio o pacchetti SCORM; fino a plugin con comportamenti complessi come ad esempio questionari o scaricamento attestati o qualsiasi altra cosa.

![Lista attivita'](https://docs.moodle.org/403/en/images_en/c/cc/actchooser4%2B.png)

L'accesso alle attivita' puo' essere libero o determinato da condizioni di vario tipo. Ad esempio richiedendo il completamento dell'attivita' precedente. Il completamento dell'attivita' e' anch'esso configurabile nei limiti dell'attivita' stessa. Ad esempio considerando l'attivita' completata al primo accesso, o per i test richiedendo il completamento con un certo voto e cosi' via...

La strategia di presentazione delle attivita' e' demandata, come quasi tutto su moodle, ad un plugin che puo' essere configurato o sostituito con un altro.

La struttura dei corsi e' molto versatile: Oltre all'uso della piattaforma per la presentazione di corsi fruibili interamente in E-Learning, e' possibile utilizzare la piattaforma come repositorio dei materiali didattici per un corso in presenza (come adesso) o come punto di partenza per una lezione tramite webinar: in questo caso bastera' inserire attivita' che permettono allo studente di accedere alla riunione sulla piattaforma di videoconferenza usata

Tecnologie
==========

Moodle e' una piattaforma scritta in PHP, vi sono diverse versioni disponibili. attualmente siamo alla versione 4.5 . e' possibile usare versioni piu' vecchie e, siccome fare l'upgrade di una versione major  non e' banale (es da 4.1 a 4.2) generalmente si trovano diverse piattaforme in essere con versioni piu' vecchie.

A dicembre 2023 e' terminato il supporto per le ultime versioni 3 (ovvero la 3.9 e la 3.11) cosi' come per la versione 4.0. La versione 4.1 e' considerata una versione LTS ed attualmente si progetta di mantenerla fino a dicembre 2025, mentre l'ultima verione LTS e' la 4.5.

La prossima versione, prevista per il 2025 e' la 5.0. Da questa versione la logica della numerazione cambia e vi saranno passaggi di serie (es da moodle 4 a moodle 5) ogni due anni, con l'ultima major release pensata per essere una release LTS

moodle non prevede un meccanismo integrato per la gestione degli aggiornamenti, si richiede quindi l'utilizzo di una procedura accorta da parte del gestore del sito (si raccomanda CALDAMENTE l'uso del Git per questo)


In ambiente windows, se si vuole procedere nativamente in maniera semplice, e' disponibile un pacchetto di installazione di uno stack LAMP semplificato nel pacchetto [XAMPP](https://www.apachefriends.org/it/index.html), tuttavia sara' comunque richiesto un certo grado di configurazione manuale, inoltre il lato MariaDb di XAMPP e' notoriamente capriccioso, per cui si raccomanda di utilizzare invece un database installato separatamente (come mariaDb stesso, che si installa facilmente su Windows con un installer)

database
--------

Moodle supporta una vasta gamma di databases:

* MySQL/MariaDB (il piu' comune)
* PostgreSQL
* MS SQL
* Oracle

la versione minima richiesta per ogni versione e' indicata nella [Documentazione](https://moodledev.io/general/releases/4.1). Generalmente l'ultima versione dovrebbe andare bene. Nella guida verra' usato MariaDB

Assicuratevi che ci sia un adeguado supporto unicode (es: `utf8mb4_unicode_ci` su MariaDB), e se possibile impostate il locale corretto (es: `lc_time_names = it_IT` su MariaDB) in modo da evitare problemi sui print delle date nei plugins

PHP
-----

[Documentazione Ufficiale](https://docs.moodle.org/403/en/PHP)

La versione di PHP supportata dipende dalla versione major utilizzata. dalla versione di moodle 4.5 si richiede almeno PHP 8.1 

Ci sono alcuni parametri consigliati per il PHP.

* `memory_limit = 256M` (96M minimo richiesto)
* `file_uploads = On` (Necessario)
* `upload_max_filesize = 100M `
* `post_max_size = 100M`
* `max_execution_time = 360`
* `cgi.fix_pathinfo = 0`
* `date.timezone = Europe/Rome`
* `intl.default_locale = it_IT.UTF-8`

Siccome i docenti potrebbero voler caricare immagini, video, o altri file di grosse dimensioni e' necessario avere un limite di caricamento consono nei limiti del buon senso. In questa guida useremo 100MB, anche perche' abbiamo constatato che il caricamento di files dell'ordine di grandezza di GB non e' gestito in maniera ottima da moodle, per files di queste dimensioni si raccomanda di usare un trasferimento SFTP o simile direttamente sul server o nello share utilizzato per i files.

Si consiglia di impostare la lingua in italiano in tutti i punti che lo permettono, per garantire formati corretti indipendentemente da come gli sviluppatori di moodle hanno deciso di gestire date e formati.

Web server
----------

Qualsiasi web server in grado di far girare il PHP e' in grado di fare funzionare moodle. In questa guida utilizzeremo NGINX, ma e' possibile configurar anche Apache o IIS in tal senso.

Bisogna avere premura a configurare i limiti di caricamento dei files almeno allo stesso livello impostato nel PHP, generalmente il default sara' piu' basso dei requisiti per un LMS. Altresi' sara' necessario configurare eventuali altri elementi dell'infrastruttura (es: interfaccia ingress su Kubernetes) in tal senso.

Esempio di file di configurazione del sito su NGINX:

```
server {
   listen 80;
   listen [::]:80;
   root /var/www/html/moodle;
   index  index.php index.html index.htm;
   server_name  ###.###.###.### www.sito;
   client_max_body_size 100M;

   location / {
   try_files $uri $uri/ =404;        
   }

   location /dataroot/ {
   internal;
   alias /var/www/html/moodledata/;
   }

   location ~ [^/]\.php(/|$) {
	   include snippets/fastcgi-php.conf;
	   fastcgi_pass unix:/var/run/php/php8.0-fpm.sock;
	   fastcgi_param SCRIPT_FILENAME $document_root$fastcgi_script_name;
	   include fastcgi_params;
   }

}
```

