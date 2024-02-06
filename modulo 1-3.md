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

