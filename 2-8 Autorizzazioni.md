Permessi
========

Documentazione ufficiale [Qui](https://moodledev.io/docs/apis/subsystems/access) e [Qui](https://moodledev.io/docs/apis/subsystems/roles)

Moodle prevede un meccanismo autorizzativo per gli utenti, si tratta di un meccanismo con alcune particolarita' rispetto a simili altri sistemi, ma per quanto possa risultare complesso permette di avere notevoli funzionalita'.

Vi sono una serie di concetti che concorrono nel meccanismo autorizzativo:


* **Ruoli**
* **Capacita**'
* **Contesti**

Da notare che esistono anche due altri concetti in Moodle che, malgrado il loro nome possa fare pensare diversamente, non sono collegati al meccanismo autorizzazione, ovvero i **gruppi** ed i **gruppi globali** (o **cohorts** in inglese)

Contesti
========

Moodle prevede diversi livelli al quale si applicano le autorizzazioni, lo stesso utente puo' ad esempio essere un docente in un corso, ma essere uno studente in un altro, e potrebbe ad esempio anche avere accesso alla reportistica del sito.

Questi livelli vengono definiti **contesti**. Le autorizzazioni saranno possibili e/o avranno effetto a seconda del contesto in cui ci troviamo.

I contesti sono organizzati in una gerarchia e sono:

* `context_system` ovvero il contesto di sistema, ce ne e' solo uno ed ha le autorizzazioni generali del sito
* `context_user` il contesto dell'utente, ve ne e' uno per ogni utente.
* `context_coursecat` un contesto per categoria di corsi
* `context_course` un contesto per corso
* `context_module` un contesto per attivita' del corso
* `context_block` un contesto per ogni blocco

I contesti utente sono contenuti in quello di sistema.

I contesti di categorie sono anche loro contenuti in quello di sistema, a loro volta possono contenere altri contesti di categoria, o contesti di corsi. Questi ultimi contengono i contesti di attivita'.

I contesti di blocco possono essere contenuti in qualsiasi altro contesto.

All'interno del codice bisogna spesso recuperare il contesto in uso, per fare cio' esistono diversi metodi di aiuto:

```php
$systemcontext = context_system::instance();
$usercontext = context_user::instance($user->id);
$categorycontext = context_coursecat::instance($category->id);
$coursecontext = context_course::instance($course->id);
$contextmodule = context_module::instance($cm->id);
$contextblock = context_block::instance($this->instance->id);
```

ed e' possibile recuperare il contesto per id:

```php
$context = context::instance_by_id($contextid);
```

Il contesto viene ad esempio consumeto dai metodi per verificare le effettive autorizzazioni dell'utente. E' abbastanza pratico semplicemente assegnarlo ad una variabile nelle prime righe della pagina.

Ruoli
=====

I **Ruoli** sono dei raggruppamenti di utenti che condividono le stesse autorizzazioni, come in molti altri meccanismi autorizzativi simili e' raccomandato assegnare le autorizzazioni ad un gruppo invece che assegnarle ai singoli utenti.

> ATTENZIONE: i **gruppi** in moodle sono un altro concetto non direttamente legato a questo meccanismo

I ruoli vanno sempre letti assieme al contesto: un utente puo' avere ruoli diversi in contesti diversi, oppure lo stesso ruolo puo' avere autorizzazioni diverse a seconda del contesto. Non tutti i ruoli sono disponibili in tutti i contesti.

Un concetto molto importante da tenere a mente e' che i ruoli sono **interamente customizzabili dall'utente** e che quindi non si ha nessuna garanzia che un particolare ruolo sia definito, per cui nel codice noi i ruoli non li useremo mai per verificare un'autorizzazione, ma useremo invece le capacita' (vedi sotto).

Per esempio: su moodle core sono preimpostati ruoli come *studente*, *docente*, *manager*, eccetera, ma un utente potrebbe rimuovere tali ruoli, inserire un nuovo ruolo "supervisore", o passare ad un sistema completamente diverso con ruoli tipo *amministratore delegato* ,*direttore*, *responsabile di ufficio*, *dipendente*, ecc.

I ruoli vengono definiti nel menu' di amministrazione Utenti -> Autorizzazioni. Nel menu' di amministrazione del sito e' poi possibile assegnare utenti a ruoli nel contesto di sistema, mentre per gli altri contesti bisogna andare nelle configurazioni specifiche di quell'oggetto (es: settaggi del corso).

Archetipi
---------

Siccome i ruoli in se non sono fissi in moodle, esiste il concetto di **archetipi di ruoli** per ovviare a quelle situazioni dove e' necessario interagire con essi.

Il caso d'uso principale e' il settaggio delle autorizzazioni di default di un plugin installato: se non potessimo assegnarle costringeremmo l'utente ad eseguire manualmente tutte le autorizzazioni nel momento in cui installa il plugin.

Moodle definisce quindi questi archetipi, al quale appartengono i ruoli. In fase di assegnazione delle autorizzazioni di defult noi assegniamo tutto agli archetipi, e poi le autorizzazioni andranno al ruolo corrispondente.

Di fatto i ruoli di default alla prima installazione di moodle corrispondono agli archetipi, e nella maggior parte delle installazioni non vi e' bisogno di ritoccare i ruoli cosicche' ruoli ed archetipi sono esattamente sovrapponibili.

Gli archetipi sono:

* `manager`
* `coursecreator`
* `editingteacher`
* `teacher`
* `student`
* `guest`
* `user`
* `frontpage`

Che corrispondono ai ruoli

* **Manager** I manager possono accedere ai corsi e modificarli ma in  genere non vi partecipano.
* **Creatore di corsi** I creatori di corsi possono creare nuovi  corsi.
* **Docente** I docenti gestiscono interamente un corso e possono  modificare le attività e valutare gli studenti.
* **Docente non editor** I docenti non editor possono insegnare nei corsi e  valutare gli studenti, ma non possono modificare le attività.
* **Studente** 	Gli studenti all'interno di un corso di norma hanno  privilegi limitati.
* **Ospite** Gli ospiti hanno privilegi minimi e normalmente non possono partecipare alle attività.
* **Utente autenticato** Tutti gli utenti autenticati.
* **Utente autenticato nella pagina home** Tutti gli utenti autenticati nel corso pagina home.

> ATTENZIONE: vi e' una differenza di nomenclatura infelice tra archetipi e ruoli: nell'identificatore dell'archetipo abbiamo **editingteacher** e **teacher**, mentre nella descrizione per l'utente abbiamo **Docente** e **Docente non editor**, ovvero con logica contraria. Questo puo' essere causa di errori.

Nell'uso degli archetipi sara' necesssario usare l'identificatore sotto forma di stringa.

Capacita'
=========

L'elemento che conclude il cerchio autorizzativo con l'utente e' la **Capacita'**. Si tratta di un elemento cablato nel codice che viene usato per identificare un azione che richiede un qualche tipo di autorizzazione.

Le capacita' definiscono il contesto nel quale sono applicabili.

Le capacita' vanno poi assegnate ai ruoli nei vari contesti

Ad esempio potremmo definire una capacita' "Puo' consultare la pagina" e l'amministratore del sito potra' poi assegnare tale capacita' solo ai ruoli teacher in quel specifico corso. 

Le capacita' si assegnano ai ruoli nel menu' amministrazione del sito -> Utenti -> Gestione ruoli. Qui e' possibile assegnare, o negare, una particolare capacita' per un ruolo

Dato l'alto numero di capacita' a disposizione non e' semplice eseguire la configurazione, e generalmente e' meglio attenersi a quanto gia' presente. E' consigliabile aggiungere ruoli con poche capacita' invece che avere un unico ruolo omnicomprensivo.

E' possibile esportare le configurazioni dei ruoli ed importarle successivamente o su altre piattaforme.

Definizione
-----------

Le capacita' vengono unicamente definite dagli sviluppatori e sono cablate nel codice. La lista delle capacita' va definita in un file particolare ovvero `db/access.php` dove definiremo un array `$capabilities` che ha come chiavi il nome della capacita' e come valori degli array che dettagliano la capacita' in questione.

```php
$capabilities = [
    'mod/folder:managefiles' => [
        'riskbitmask' => RISK_SPAM,
        'captype' => 'write',
        'contextlevel' => CONTEXT_MODULE,
        'archetypes' => [
            'editingteacher' => CAP_ALLOW,
        ],
    ],
];
```

Il **nome** della capacita' da mostrare all'utente va definito nei files con le stringhe nella cartalla `lang` usando come chiave `nomeplugin:capacita`

```php
$string['folder:managefiles'] = 'Manage files in folder module';
```

le caratteristiche da attribuire alle capacita' sono:

`contextlevel` : il contesto della capacita'. E' possibile verificare la capacita' anche nei contesti inferiori a quello indicato.

`archetypes` : i permessi da assegnare di default ai ruoli in base al loro archetipo. Di prassi qui ha senso usare solo `CAP_ALLOW`.

`clonepermissionsfrom` : un'alternativa a definire la voce degli archetipi, fa si che i permessi verranno copiati da un'altra capacita' esistente.

`captype` : puo' essere `read` o `write`, le capacita' `write` saranno forzatamente negate ad utenti guest o non autenticati.

`riskbitmask` : permette di definire delle categorie di rischio legate alla capacita', a scopo di audit. Gli utenti vedranno nel menu' di amministrazione dei warning relativi alle categorie di rischio che associate. Queste sono:

* `RISK_SPAM` Se puo' far visualizzare contenuto sul sito o in notifiche ad utenti
* `RISK_PERSONAL` Se ha accesso a dati personali
* `RISK_XSS` Se da possibilita' di fornire input non sanificato
* `RISK_CONFIG` Se puo' modificare configurazioni globali
* `RISK_MANAGETRUST` Se puo' modificare i permessi degli utenti
* `RISK_DATALOSS` Se puo' causare, intenzionalmente o per errore, perdite di dati non facilmente recuperabili

Moodle consiglia di non dare ad utenti guest capacita' con rischi associati, agli studenti al massimo dare capacita' con RISK_SPAM, ed ai docenti dare anche al massimo RISK_PERSONAL e RISK_XSS.

Utilizzo
--------

Per controllare che un utente disponga di una particolare capacita' e' sufficiente utilizzare la funzione `has_capability()` alla quale e' necessario almeno passare il nome della capacita' ed i contesto dove va verificata

```php
if(has_capability('mod/folder:managefiles', $context)){
    // [...]
}
```

esiste anche un altro metodo `require_capability()` che fa la stessa cosa ma restituisce un'eccezione quando fallisce, utile come controllo di sicurezza in punti dove non si dovrebbe accedere altrimenti.

Altri concetti
==============

Amministratori del sito
-----------------------

Gli amministratori del sito sono definiti separatamente da ruoli e capacita'. Si tratta di una lista di utenti che non hanno restrizioni di accesso.

Si definiscono da un'apposita voce nel menu di amministrazione.

Si consiglia di evitare di utilizzare questa assegnazione se non per un account ed utilizzare invece il ruolo "manager".

Gruppi
------

[Documentazione](https://moodledev.io/docs/apis/subsystems/group)

I Gruppi sono un concetto non direttamente collegato alla sfera autorizzativa.

I gruppi sono unicamente definiti all'interno di un corso, e sono insiemi di partecipanti da usare per organizzare il lavoro del corso, ad esempio se si vuole riutilizzare lo stesso corso per piu' edizioni.

I gruppi vanno abilitati a livello di corso, e se abilitati possono funzionare come "visibili" o "segregati".

Se si usa l'opzione "segregati" allora i membri di un gruppo non dovrebbero poter interagire con i membri di altri gruppi, ne vedere la lista dei membri di altri gruppi, a meno di non possedere la capacita' `moodle/site:accessallgroups`.

Se i gruppi sono abilitati, indipendentemente dal settaggio, allora i report ed altre funzionalita' ne terranno conto nelle visualizzazioni.

Qui non vediamo come rendere un'attivita' conforme ai gruppi, per ora si rimanda alla documentazione.

Gruppi Globali
--------------

I **gruppi globali**, o **cohorts** nella versione inglese, sono degli insiemi di utenti a livello di sito.

I gruppi globali sono solo di carattere organizzativo, vi sono diverse operazioni, in primis l'iscrizione ad un corso, che puo' essere fatta a tutto un gruppo globale in un unica operazione.

E' possibile fare delle assegnazioni massive di ruoli nel contesto user ai gruppi globali, ma di certo esistono plugins che eseguono anche altre assegnazioni.

I gruppi globali, essendo trans-corso, sono in effetti usati abbastanza spesso per la loro semplicita'.





