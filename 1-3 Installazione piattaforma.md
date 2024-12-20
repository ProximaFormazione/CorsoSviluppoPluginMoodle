Installazione Moodle
==================================

In questo modulo eseguiremo l'installazione di un ambiente di sviluppo utilizzato per il resto del corso.

Da notare che per lo sviluppo e' caldamente consigliato installare lo stack su un computer locale dove si ha accesso ad un IDE con capacita' di debug. Il rilascio su macchine di test o produzione puo' essere eseguito in fasi successive.

Una guida ufficiale con altre casistiche e' disponibile al seguente [link](https://docs.moodle.org/403/en/Installing_Moodle).

E' chiaramente possibile installare un'immagine con un container gia' preconfigurato, vedi ad esempio il [Docker Hub di Bitnami](https://hub.docker.com/r/bitnami/moodle). Tuttavia in questo corso verra' eseguita invece l'installazione manualmente in modo da prendere dimestichezza con i vari elementi e saper risolvere eventuali problemi

Installazione tecnologie necessarie
===================================

Per il funzionamento di moodle sono richiesti tre elementi tecnologici:

* PHP
* Database
* Web Server
* (file system)

Fintanto vengono fatte delle scelte compatibili e' possibile utilizzare le specifiche tecnologie che si preferiscono. In questo modulo si provvedera a fornire le istruzioni per un'installazione su macchina ubuntu, ma su altri sistemi si puo' procedere eseguendo gli step analoghi.

Inoltre e' necessario avere per lo sviluppo:

* Git 
* IDE per sviluppo PHP



Linux
-----

Qui seguiremo una procedura di installazione su macchina vergine.

La procedura e' copiata da quelle utilizzate generalmente su Ubuntu, ovviamente non e' necessariamente il migliore modo di configurare un server allo scopo ma e' utile a livello esemplificativo

### Preliminari

Installare il locale IT:

$ `sudo apt-get install language-pack-it`

### Installare MariaDb

$ `sudo apt install mariadb-server mariadb-client`

attenzione alla versione utilizzata: serve almeno la versione 10.6, se vi ritrovate con una versione antecedente assicuratevi di accedere al repository apt corretto (selezionate [qui](https://mariadb.org/download/?t=repo-config) la versione richiesta).

Si consiglia di procedere con lo script preimpostato per la rimozione degli elementi di default che vengono installati, con il comando
$ `sudo mysql_secure_installation` e selezionando le opzioni richieste (Y a tutto). Questo e' un buon momento per **SALVARE LA PASSWORD DI ROOT DEL DATABASE** che avete appena scelto nelle modalita' a voi preferite per evitarvi di penare in futuro.

A questo punto possiamo gia' configurare il database che useremo per moodle:

1. $ `sudo mysql -u root -p`
1. MariaDB [(none)]> `CREATE DATABASE moodle DEFAULT CHARACTER SET utf8mb4 COLLATE utf8mb4_unicode_ci;` 
1. MariaDB [(none)]> `GRANT ALL ON moodle.* TO 'moodle'@'localhost' IDENTIFIED BY '@PASSWORD';` al posto di @PASSWORD scegliete voi una password (**Tenetela da parte** servira' dopo)
1. MariaDB [(none)]> `FLUSH PRIVILEGES`;
1. MariaDB [(none)]> `EXIT`;

Attenzione al set di caratteri che deve supportare UTF-8.

Terminare riavviando il servizio per buona fortuna:

$ `sudo systemctl restart mysql.service`

### Installare PHP

moodle richiede una serie di estensioni per il corretto funzionamento ([lista dettagliata](https://docs.moodle.org/403/en/PHP)).

Installeremo PHP 8.1 per una buona compatibilta' con le versioni 4.1 ed oltre di moodle attuali.

se non presente aggiungere il seguente repository apt:

* $ `sudo apt-get install software-properties-common`
* $ `sudo add-apt-repository ppa:ondrej/php`

Per l'ambiente di sviluppo (ma anche per la produzione), consiglio un approccio abbondante:

$ `sudo apt install php8.1-fpm php8.1-common php8.1-mbstring php8.1-xmlrpc php8.1-soap php8.1-gd php8.1-xml php8.1-intl php8.1-mysql php8.1-cli php8.1-mcrypt php8.1-ldap php8.1-zip php8.1-curl`

Per l'ambiente di sviluppo sara' necessaria l'estensione $ `sudo apt-get install php8.1-xdebug`

Modificare poi il file di confiurazione `/etc/php/8.1/fpm/php.ini` inserendo i seguenti valori:

```
  file_uploads = On
  allow_url_fopen = On
  memory_limit = 256M
  upload_max_filesize = 100M
  post_max_size = 100M
  max_execution_time = 360
  cgi.fix_pathinfo = 0
  date.timezone = Europe/Rome
  intl.default_locale = it_IT.UTF-8
  max_input_vars = 6000
```

ed attivando le seguenti estensioni

```
extension=curl
extension=gd
extension=intl
extension=mbstring
extension=soap
extension=sodium
```

a seconda di altre funzionalita' richieste potrebbe essere necessario ritornare in seguito ed aggiungere eventuali estensioni.

per XDebug controllate di avere i settaggi impostati

```
[XDebug]
zend_extension= "{...}/php/ext/php_xdebug.dll"
xdebug.client_port = 9090
xdebug.remote_enable = 1
xdebug.remote_autostart = 1
xdebug.mode = debug
xdebug.start_with_request = yes
```

indicando il percorso corretto e la porta a vostro piacimento

Dopo aver terminato le modifiche rilanciare il servizio fpm

$ `sudo service php8.1-fpm restart`

### NGINX

installare nginx

$ `sudo apt install nginx`

E poi procedere alla configurazione del sito: da notare che non abbiamo ancora caricato il sito quindi questo step andrebbe normalmente fatto alla fine

creare il file `/etc/nginx/sites-available/moodle` ed inserire il seguente incantesimo di magia nera:

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

       location ~ [^/]\.php(/|$) {
           include snippets/fastcgi-php.conf;
           fastcgi_pass unix:/var/run/php/php8.1-fpm.sock;
           fastcgi_param SCRIPT_FILENAME $document_root$fastcgi_script_name;
           include fastcgi_params;
       }

   }
```

alcune osservazioni:

* Il sito e' configurato unicamente per l'http (porta 80) in quanto e' un sito di sviluppo, altrimenti dovreste aggiungere anche la configurazione per l''https:

```
    listen 443;
    listen [::]:443;
    ssl on;
    ssl_certificate /etc/ssl/certs/SITO/certificate.crt;
    ssl_certificate_key /etc/ssl/certs/SITO/certificate.key;
```

* Il parametro root indica la posizione del sito nel filesystem
* per l'ambiente di sviluppo potete omettere il nime del sito ed usare unicamente l'ip, che va inserito alla riga `server_name` al posto dei cancelletti
* ebbene si, *www.sito* e' un placeholder

Una volta completato il file mettere un link nell'elenco dei siti abilitati con $ `sudo ln -s /etc/nginx/sites-available/moodle /etc/nginx/sites-enabled/`

infine scacciate gli spiriti maligni con la formula  $ `sudo systemctl restart nginx.service`

Windows
-------

Se si sta usando un dispositivo windows, dove non e' installato uno stack LAMP, un metodo rapido di installare i prerequisiti e' installare il pacchetto XAMPP ([Link](https://www.apachefriends.org/)). Che contiene il php, apache e mariaDb.

E' comunque possibile installare moodle usando unicamente tecnologie native microsoft, in questo capitolo accenniamo alla procedura di configurazione usando IIS come web server e Microsoft SQL (MSSQL) come database.

L'installazione su uno stack Microsoft presenta una serie di controindicazioni:

- Le performance sono peggiori rispetto ad una equivalente installazione su Linux e Nginx, anche se comunque il sito funziona in maniera accettabile. Nei nostri test su macchine equivalenti il benchmark impiega quasi il doppio del tempo su Windows
- Il PHP e' sviluppato prevalentemente per ambienti Linux, alcune estensioni potrebbero non essere disponibili o avere problemi imprevisti di compatibilita'

Detto cio', se non avete scelta e' comunque possibile procedere

### Installare PHP su IIS

Per installare il PHP possiamo semplicemente scaricarlo dal sito ufficiale ([Link](https://windows.php.net/download/)), selezionate voi la versione del PHP richiesta (es: 8.1) per l'architettura del vostro server (x86 o x64). Utilizzando fastCGI su IIS non dovrebbe essere richiesta la versione thread safe ed anzi converrebbe scaricare la versione non thread safe per migliorare la performance (da verificare sul campo).

Possiamo semplicemente copiare il contenuto della cartella in un folder a piacere sul server (diciamo c:\PHP\8.1).

Nel folder del PHP dobbiamo rinominare uno dei files `php.ini-development` o `php.ini-production` in `php.ini`, a seconda dell'ambiente che stiamo configurando in modo da avere una configurazione di partenza adatta (che poi possiamo modificare)

Una volta installato si raccomanda di aggiungere il folder alla variabile PATH

- nella casella di ricerca digitiamo "env" per accedere alla maschera di configurazione, ed in baso "variabili di ambiente" (o similare)
- cerchiamo PATH ed aggiungiamo la cartella (es c:\PHP\8.1) alla lista

### Configurare IIS

Per prima cosa dobbiamo accertarci di avere IIS installato, e che sia abilitato il modulo CGI.

- Aprire il pannello per modificare le funzionalita' di windows
  - In windows home/pro in Pannello di controllo -> Programmi e Funzionalita -> Attiva o disattiva funzionalita' di windows
  - In windows Server in Server Manager -> Manage(alto a dx) -> Add Roles adn Features -> Roles
- Selezionare CGI sotto Internet Information Services -> World Wide Web services -> Application development features -> CGI

![Immagine](https://stackify.com/wp-content/uploads/2019/03/image-6.png)

In IIS CGI include FastCGI di default senza bisogno di configurazioni aggiuntive. Una volta abilitato il CGI riavviare IIS Manager.

Bisogna poi abilitare la mappatura con l'interprete PHP.

- L'impostazione delle mappature e' accessibile sul IIS manager selezionando il nodo base del server stesso IIS-> Handler mappings -> doppio click

![Immagine](https://stackify.com/wp-content/uploads/2019/03/image-16.png)

- Aggiungiamo una mappatura indicando
  - Come path `*.php`
  - Come modulo "FastCIModule"
  - Come eseguibile l'eseguibile "php-cgi.exe" contenuto nella cartella del PHP (es: `c:\PHP\8.1\php-cgi.exe`)
  - Un nome friendly a piacere, es `PHP-CGI`

Per concludere dobbiamo andare nell'impostazione dei documenti di default (sempre nel nodo del server) ed aggiungere `index.php` alla lista

Per testare il corretto funzionamento, andate nella root dell'IIS (normalmente \inetpub\wwwroot) e create un file `test.php`, poi editate il contenuto con notepad ed inserite il seguente testo:

```
<?php phpinfo(); ?>
```

navigando su http://localhost/test.php dovreste vedere una schermata del php viola con la lista delle configurazioni, se la vedete il PHP sta girando correttamente. (Quest presume un IIS con configurazione di default, se necessario inserite il file in posizione esposta dall'IIS)

### MSSQL

Se dovete utilizzare Microsoft SQL server avete bisogno di installare driver aggiuntivi per permettere alle applicazione PHP (come moodle) di accedere al server.

I dirver sono necessari anche nel caso di MSSQL installato su ambiente Linux, dove si puo' facilmente procedere con apt o simili (vedi [Documentazione Microsoft](https://learn.microsoft.com/en-us/sql/connect/php/installation-tutorial-linux-mac?view=sql-server-ver16)).

I driver per Windows sono delle estensioni per il PHP da aggiungere. Possono essere scaricate dalle fonti Microsoft: [>Link<](https://go.microsoft.com/fwlink/?linkid=2258816) e [Documentazione](https://learn.microsoft.com/en-us/sql/connect/php/download-drivers-php-sql-server?view=sql-server-ver16).

Il pacchetto contiene tutte le varie versioni possibili, vi e' un file di istruzioni incorporato, ma la documentazione effettiva la trovate a [Questo link](https://learn.microsoft.com/en-us/sql/connect/php/loading-the-php-sql-driver?view=sql-server-ver16), in breve:

- verificate sul file `SQLSRV_Readme.htm` le dll richieste in base all PHP installato, sia per numero di versione che per il caso thread safe(ts) o non thread safe(nts) 
- spacchettate le due dll necessarie e copiatele nella cartella delle estensioni del PHP (es: `c:\PHP\8.1\ext`), se non siete sicuri di dove si trovi la cartella potete consultare il phpinfo() di sopra
- modificate il file php.ini aggiungendo le seguenti righe (con i nomi delle vostre dll) nella sezione delle estensioni:

```
extension=php_pdo_sqlsrv_81_nts_x64.dll
extension=php_sqlsrv_81_nts_x64.dll
```

riavviate il web server se necessario.

Ottenere i files di Moodle
==========================

Ora che l'ambiente e' stato preparato a puntino, e' il momento di scaricare i files di moodle.

Per l'ambiente di sviluppo installeremo moodle da 0 tutto sulla stessa macchina.

Cartella Dati
-------------

Servira' una cartella per i dati (moodledata). Createla dove vi viene piu' comodo, ma non esposta dal web server.

Se siete in un sistema Unix evitatevi tutta una serie di problemi con un chmod 777 su tutta la cartella

Nel resto della guida questa cartella verra' chiamata **moodledata**, ma voi siate liberi di chiamarla come preferite. Ricordatevi il percorso servira' dopo.

1. `sudo mkdir /var/moodledata` per creare la cartella
2. `sudo chown -R www-data /var/moodledata` per assegnare la propieta' della cartella all'utente NGINX
3. `sudo chmod -R 0777 /var/moodledata` per dare pieni diritti, valutare se necessario usare invece `0770` per limitare altri utenti del server

Cartella Moodle
---------------

Per i files di moodle e' assolutamente consigliato l'utilizzo di Git per l'installazione.

Chiaramente in quanto sviluppatori avrete necessita' di tracciare eventuali modifiche che apporterete al codice core di moodle, ma anche per installazioni semplici il Git e' fondamentale per gestire gli aggiornamenti.

Non e' presente in moodle un sistema di aggiornamento automatico, i files vanno scaricati nuovi per ogni versione. 

Sebbene si possa caricare i files a manina, tutti i vari plugin aggiuntivi o personalizzazioni al codice andrebbero riportate sulla nuova versione.
Quando si inizia a gestire un certo numero di piattaforme diventa difficile tenere traccia di tutte le modifiche fatte e si rischia di lasciare indietro qualcosa.

Utilizzare il git permette di tenere traccia delle personalizzazioni ed eseguire l'aggiornamento alla versione piu' recente (minor) semplicemente eseguendo un `git merge`.

Se non eseguite modifiche sul codice core, non dovrebbe mai dare conflitti e filare liscio, se invece avete conflitti allora questo metodo vi mette in risalto una modifica che e' stata sovrascritta dal team di moodle permettendovi di decidere il da farsi.

per iniziare clonate il repository ufficiale di moodle:

* cd `/var/www/html/moodle` per NGINX, oppure altra location in base al vostro web server 

* `git clone git://git.moodle.org/moodle.git .` occhio al punto ;)
* `git config --global --add safe.directory /var/www/html/moodle` se richiesto dall'OS

Bisogna poi assegnare i diritti corretti alla cartella 

* `sudo chown -R www-data:www-data /var/www/html/moodle/` 
* `sudo chmod -R 777 /var/www/html/moodle/` o anche 755 per essere piu' stringenti

i branch sul repository ufficiale hanno una nomenclatura basata sul numero di versione usato, quindi per la 4.5 cercheremo i branch con scritto 405

* `sudo git branch --track MOODLE_405_STABLE origin/MOODLE_405_STABLE`
* `sudo git checkout MOODLE_405_STABLE`

a questo punto converra' creare un branch separato per la nostra versione

* `sudo git checkout -b develop` o altro nome di branch a vostra discrezione

ora il sito e' pronto all'installazione

Installazione
=============

Configurazione iniziale
-----------------------

Moodle ha una procedura di installazione integrata, nel file **install.php**, che viene lanciata se si accede al sito senza avere il file config.php presente, la procedura provvede a creare il file inserendo i dati base riguardanti:

* Percorsi delle cartelle
* Driver del database da usare
* Credenziali del database

E' possibile creare a mano il file config.php ed inserire talli valori, ma non vi e' un vero vantaggio (lo potete editare anche dopo), la procedura e' poi comoda per i casi dove si ha un accesso ridotto al server, ad esempio se abbiamo solo accesso http/s, o per utenti piu' base.

Nel caso la piattaforma sia una copia di una piattaforma esistente, il file config.php sara' gia' presente quindi non serve nessuna procedura.

Allineamento Database
---------------------

Moodle ha robusto sistema per verificare che il database sia aggiornato alla versione richiesta dal sorgente, ogniqualvolta viene individuata una modifica, ovvero quando si aggiorna la piattaforma o anche solo un plugin. 

Questo secondo processo esegue le operazioni sul database per aggiungere effettivamente le tabelle, quindi da un certo punto di vista e' la vera installazione.

La procedura parte automaticamente se si logga un utente amministratore al sito, e per prima cosa esegue un controllo sui requisiti prima di procedere, con una maschera riassuntiva con tutti i problemi ed i warning del caso.

Segue poi una maschera con il riassunto dei plugin che verranno installati ed aggiornati. Per la prima installazione sara' una lista oscenamente lunga in quanto include tutti i plugin nel pacchetto di moodle base.

Se si procede, moodle provvedera' ad aggiornare le tabelle del db, secondo le istruzioni fornite da ogni singolo plugin. Questo meccanismo lo vedremo in dettaglio piu' avanti.

Se tutto va bene vi verra' mostrata una schermata dove dovrete inserire i dettagli dell'utente amministratore, questo utente sara' necessario per accedere alla piattaforma quindi segnatevi le credenziali. Successivamente vi verranno chieste alcune informazioni di base per il sito come il nome e la mail di contatto. Infine dovreste arrivare sul sito funzionante

Cron
----

Per il corretto funzionamento di moodle, e' necessario impostare il cron, ovvero un comando schedulato da lanciare ripetutamente.

c'e' uno script da chiamare nella posizione

> ....../admin/cli/cron.php

che va lanciato il piu' spesso possibile, idealmente una volta al minuto.

Questo job esegue tutte quelle operazioni asincrone richieste nel funzionamento di moodle.

### Su linux

Su una macchina linux e' sufficiente utilizzare il cron di linux, modificatelo con il comando `crontab -e` ed inserita la seguente riga

> */1 * * * * /usr/bin/php /var/www/html/moodle/admin/cli/cron.php >/dev/null

Non vi e' necessita' di conservare i log dei cron, anche perche' questi sono comunque conservati per i task schedulati ed altri eventi dal meccanismo di log interno di moodle, e salvare queste informazioni puo' occupare spazio inutilmente. 

### Su Windows

Su windows e' presente un tool adatto allo scopo

1. cercare l'applicazione "Task Scheduler"
2. inserire un nuovo task con frequenza maggiore possibile (dovrebbe essere 5 minuti, ma piu' frequente e' meglio)
3. Inserire un azione "start a program" con l'eseguibile del php (es: `C:\xampp\php\php.exe` o `c:\PHP\8.1\php.exe`) con argomento il file con lo script (es: `-f "C:\xampp\htdocs\moodle30\admin\cli\cron.php"` o `-f "C:\inetpub\wwwroot\moodle\admin\cli\cron.php"`)

### Via Web Server

Se non si ha accesso alla macchina dove e' installato moodle, esiste un opzione per lanciare il cron di moodle via una chiamata ad una url specifica.

E' un'opzione sconsigliata per motivi di sicurezza, ma in certe installazioni non vi e' altra scelta. 

Di default questa opzione e' disattivata, e' necessario abilitarla all'interno del sito in *Sicurezza -> Impostazioni di sicurezza del sito -> Esecuzione cron solamente a linea di comando* , dove e' necessario impostare la password per l'api.

Una volta abilitata l'api, dovete schedulare la chiamata su una vostra macchina, ad esempio lanciando:

> curl https://nomesito.it/admin/cron.php?password=12345

Si tratta di una chiamata in get con la password in chiaro, quindi evitate di usare una password che un idiota userebbe per la sua valigia.

![Attenzione a non scegliere password semplici](https://i.imgflip.com/3jcn63.png)

Impostazioni utili
==================

Vedremo in seguito come impostare moodle, ma intanto inserisco alcune impostazioni utili da controllare al momento della creazione di una nuova piattaforma.

Tutte queste voci si trovano nel menu' di amministrazione

* **Impostazioni di pulizia**: lasciare i default tranne l'eliminazione dello storico delle modifiche delle valutazioni da mai a dopo 365 giorni
* **log Standard**: Conserva il log per: 1000 gg
* **Sicurezza/Impostazioni di sicurezza del sito**: controllare limite caricamento file
* **Amministrazione del sito/Localizzazione/impostazioni**: controllare il fuso orario
* Impostare settaggi per le mail in uscita in **Server/EMail**. Per l'ambiente di sviluppo consiglio di usare una mail personale invece che configurare sendmail sulla macchina locale

Ora rimane solo da verificare che tutto funzioni.












