Architettura 2
==============

Qui elencheremo la struttura dei plugins di moodle.


I plugins sono suddivisi in tipi diversi a seconda della loro funzionalita'. Questa tipizzazione non e' unicamente a scopo organizzativo, ma stabilisce le funzionalita' accessibili al plugin stesso.

I plugin possono essere pubblicati sulla cosiddetta [Moodle plugins Directory](https://moodle.org/plugins/?q=) che e' un sito dove e' possibile ricercarli per tipo o parole chiave. Ed e' la prima fermata nel processo di sviluppo di una nuova funzionalita' di moodle (probabilmente qualcuno ci ha gia' pensato).

Esistono poi plugins in vari repository in GitHub non pubblicati sulla Moodle plugins Directory, possono comunque essere installati.

Il processo di installazione di un plugin puo' essere anche eseguito da un utente del sito (amministratore) tramite il caricamento dei files zippati del plugin su un'apposita interfaccia dentro moodle. Attenzione a valutare se consentire questa funzionalita' o no, anche perche' in architetture con piu' istanze non puo' essere usata

Vedremo qui i plugin principali, una lista completa e' disponibile a questo 
[link](https://moodledev.io/docs/apis/plugintypes)

Mod - Activity Modules
----------------------

Cartella: mod

I moduli attivita', o moduli corso, sono le attivita' che e' possibile inserire all'interno di un corso.

Si tratta degli elementi che presentano i contenuti didattici quali testi, video o questionari.

Alcuni dei moduli attivita' preinstallati comprendono

* Label per l'inserimento di html generico
* Book, ovvero pagine di testo
* Files caricati sulla piattaforma
* Quiz sui contenuti
* Feedback di gradimento
* Pacchetti SCORM

*esempio di aggiunta attivita' al corso*

Questi plugin definiscono dei moduli che possono essere inseriti in uno o piu' corsi. Si hanno quindi molteplici istanze diverse del modulo sulla piattaforma.

I plugin di tipo mod hanno delle eccezioni alle regole sul codice dovute al fatto di essere la prima categoria di plugin storicamente implementata.

Auth - Plugin di autenticazione
-------------------------------

Sono plugin che gestiscono il metodo di autenticazione sul sito, come ad esempio: login tramite password, OAuth, LDAP, ecc.

Ogni utente in moodle e' legato ad uno specifico metodo di autenticazione, sul sito possono essercene diversi abilitati.

Sono presenti gia' un certo numero di plugin di autenticazione per i principali protocollli in voga. Di default sono tutti disattivati ed e' attivo solo la login "manuale", ovvero tramite username e password.

Blocks - Blocchi
----------------

I blocchi sono elementi che e' possibile inserire, tramite interfaccia, in vari punti di altre pagine moodle.

I blocchi possono avere le caratteristiche piu' disparate, ma potendo essere ovunque spesso sono funzionalita' generiche.

enrol - Iscrizione a corsi
--------------------------

Sono plugin che permettono e definiscono le modalita' di iscrizione, e se previsa disiscrizione, dai corsi.

Esempi sono l'autoiscrizione, l'iscrizione manuale, l'iscrizione tramite acquisto del corso, iscrizione tramite link mail, ecc...

Come per i plugin di autenticazione, piu' di uno puo' essere disponibile ed ogni utente avra' un solo metodo associato per l'iscirizione ad uno specifico corso.

I plugin di enrol vanno abilitati sia a livello di sito, sia a livello di specifico corso

theme - temi
------------

I plugin del tema decidono come le pagine rappresentano i loro contenuti. Questi plugin hanno il maggior impatto sull'aspetto generale della piattaforma.

Sono plugin generalmente estensivi e complessi, che generalmente sono realizzati da aziende e disponibili a pagamento. 

Moodle ha comunque dei suoi temi predefiniti: **Boost**, che e' un tema che utilizza bootstrap 4 con ampie possibilita' di customizzazione; e **Classic**, che malgrado il nome e' posteriore a boost.

Molti temi prevedono la possibilita' di alterare o creare le pagine, come la dashboard dell'utente, ma anche altri elementi.

Data la complessita' ed estensivita', non e' impossibile che i temi creino conflitti o problemi con altri plugin o con i sistemi core di moodle, anettodicamente ci e' capitato di dover correggere alcune linee di codice in temi acquistati.

### course/format - formato corsi

plugin che determinano il formato con cui le attivita' vengono presentate all'interno di un corso, una sorta di tema per i corsi.

local
-----

i plugin locali sono plugin che non rientrano nelle altre categorie. Si tratta di plugin che agiscono a livello di sito

report
------

Plugin per la realizzaione di report vari, course/report invece e' per i report di corso

repository
----------

I repository sono i plugin che determinano dove sono conservati i files caricati dagli utenti (studenti e/o amministratori). Esempi sono share di rete, bucket S3, ecc...

Similmente ad autenticazione ed enrol ci possono essere molteplici plugin installati, e per i repository addizionalmente possono esserci piu' istanze per ogni plugin (es: tre diversi drive di rete).

I repository si distinguono nelle capacita' che offrono, in base a queste altri plugin daranno possibilita' o meno di utilizzare il repository, ad esempio l'attivita' "url" che presenta un link funzionera' solo con repository che supportano questa modalita' di accesso.

### Strategie recupero files

Per i files sono possibili diverse modalita' di accesso, che andranno definite in un apposita funzione in fase di sviluppo:

* `FILE_INTERNAL` se il file e' nel file system di Mooddle
* `FILE_REFERENCE` se il file viene fornito dal repository esterno ma salvato localmente in cache
* `FILE_EXTERNAL` se il file viene fornito dal repository esterno
* `FILE_CONTROLLED_LINK` il file rimane sul repository esterno sul quale l'utente viene reindirizzato per accedere

### Contesto

il repository puo' essere unicamentea livello di sito, oppure e' possibile ammettere istanze separate a livello di corso e di utente.

/cache/stores - caches
----------------------

Questi plugin sono utilizzati per salvare i dati della cache, i plugin definiscono unicamente lo storage da utilizzare, la strategia di implementazione e' parte di moodle core, in un componente definito MUC (Moodle Universal Cache).

i plugin di cache hanno il compito di stabilire come salvare e recuperare i dati, e definiscono che tipologie di cache supportano. In un'apposita maschera di amministrazione e' possibile indicare che tipo di cache utilizzare per ogni scopo.

vi sono tre diversi scopes per le cache:

* application che viene usato trasversalmente nel sito
* session
* request

vi sono gia' diversi plugin di cache preinstallati, tra cui memchached e redis, che dovrebbero essere sufficienti per qualsiasi scenario, generalmente qui il discorso e' piu' configurarli in quanto di default sono abilitati il file system ed il database.

La performance della cache e' molto rilevante nelle performance generali del sito. Per le installazioni ad istanza singola vi e' una differenza sostanziale passando ad un SSD, mentre per le installazioni con multiple istanze e' praticamente richiesto usare redis o simile.

[Documentazione ufficiale](https://docs.moodle.org/403/en/Caching)





