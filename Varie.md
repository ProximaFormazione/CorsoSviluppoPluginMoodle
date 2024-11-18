
Altre API
=========

Altri meccanismi di moodle che non abbiamo visto

Settaggi utente
---------------

I settaggi specifici di un utente sono salvati nel database ed accessibili in maniera similare ai settaggi del sito.

Sono a disposizione i metodi `get_user_preferences()`, `set_user_preference()` e `unset_user_preference()`

vedi il codice o la documentazione per altri dettagli ([link](https://moodledev.io/docs/apis/core/preference))

Report Builder
--------------

Da moodle 4.0 e' disponibile un generatore di reportistica dove gli utenti possono costruire le loro dashboard basate su cubi di dati preimpostati in moodle. 

vedi [qui](https://moodledev.io/docs/apis/core/reportbuilder) per come aggiungere tabelle alla scelta dell'utente

install/uninstall.php
---------------------

sono files che vengono lanciati alla prima installazione del plugin, ed alla disinstallazione. Non vengono chiamati in fase di aggiornamento. Il file `install.php` viene eseguto subito dopo l'installazione dello schema (da `install.xml`). `uninstall.php` viene invece lanciato quando un amministratore disinstalla il plugin dall'apposita interfaccia amministrativa. Questi files sono utili se e' necessario eseguire delle configurazioni iniziali, ad esempio caricare dati nelle tabelle del plugin.

Messaggi e notifiche
--------------------

[link](https://moodledev.io/docs/apis/core/message)

La messaggistica e le notifiche in moodle sono configurabili lato utente, permettendo di decidere come e se ricevere notifiche in base alla loro provenienza.

Per fare funzionare cio' e' necessario che un plugin si censisca come produttore di messaggi attraverso il file `/db/messages.php`

```php
defined('MOODLE_INTERNAL') || die();
$messageproviders = [
    // Notify teacher that a student has submitted a quiz attempt
    'submission' => [
        'capability' => 'mod/quiz:emailnotifysubmission'
    ],
    // Confirm a student's quiz attempt
    'confirmation' => [
        'capability' => 'mod/quiz:emailconfirmsubmission',
        'defaults' => [
            'pop-up' => MESSAGE_PERMITTED + MESSAGE_DEFAULT_LOGGEDIN + MESSAGE_DEFAULT_LOGGEDOFF,
            'email' => MESSAGE_PERMITTED,
        ],
    ],
    // Ordinary single forum posts, available to all
    'posts' => [],
];
```

ogni possibile fonte di messaggi filtrabile va indiata nell'array. La capacita' e' necessaria per ricevere il messaggio, quindi omettendola il messaggio sara' disponibile a tutti.

la stringa descrittiva per l'utente va indicata con la formula `messageprovider:xxxxx`

```php
$string['messageprovider:confirmation'] = 'Confirmation of your own quiz submissions';
$string['messageprovider:submission'] = 'Notification of quiz submissions';
```

il campo *defaults* indica le impostazioni di default per i nuovi utenti/installazione plugin.

Per inviare un messaggio abbiamo a disposizione una classe apposita ed il metodo `message_send()`, di seguito un esempio esasustivo deipi a disposizione:

```php
$message = new \core\message\message();
$message->component = 'mod_yourmodule'; // Your plugin's name
$message->name = 'mynotification'; // Your notification name from message.php
$message->userfrom = core_user::get_noreply_user(); // If the message is 'from' a specific user you can set them here
$message->userto = $user;
$message->subject = 'message subject 1';
$message->fullmessage = 'message body';
$message->fullmessageformat = FORMAT_MARKDOWN;
$message->fullmessagehtml = '<p>message body</p>';
$message->smallmessage = 'small message';
$message->notification = 1; // Because this is a notification generated from Moodle, not a user-to-user message
$message->contexturl = (new \moodle_url('/course/'))->out(false); // A relevant URL for the notification
$message->contexturlname = 'Course list'; // Link title explaining where users get to for the contexturl
// Extra content for specific processor
$content = [
    '*' => [
        'header' => ' test ',
        'footer' => ' test ',
    ],
];
$message->set_additional_content('email', $content);

// You probably don't need attachments but if you do, here is how to add one
$usercontext = context_user::instance($user->id);
$file = new stdClass();
$file->contextid = $usercontext->id;
$file->component = 'user';
$file->filearea = 'private';
$file->itemid = 0;
$file->filepath = '/';
$file->filename = '1.txt';
$file->source = 'test';

$fs = get_file_storage();
$file = $fs->create_file_from_string($file, 'file1 content');
$message->attachment = $file;

// Actually send the message
$messageid = message_send($message);
```

Le notifiche vanno abilitate da un amministratore del sito. E' poi possibile per un utente disabilitare la ricezione o modificare i parametri.

Le notifiche possono essere impostate per singola voce ed essere attivate o disattivate, impostate per essere sul sito (icona campanella) e/o via mail.

Calendario
----------

[Documentazione](https://moodledev.io/docs/apis/core/calendar)

Moodle ha un sistema di eventi trasversale sul sito, dove e' piossibile inserire date visibili agli utenti, come ad esempio la scadenza di un corso.

Il calendario e' visualizzabile nell'apposito blocco calendario, oppure nella dashboard. Chiaramente esisteranno plugin alternativi per la visualizzazione.

Si puo' interagire con il calendario utilizzando la classe `calendar_event`

```php
$event = new stdClass();
$event->type = CALENDAR_EVENT_TYPE_STANDARD;
// ... vedi documentazione per la lista competa dei campi 
calendar_event::create($event); 

$eventid = required_param('id', PARAM_INT);
$event = calendar_event::load($eventid);

$data = $mform->get_data();
$event->update($data);

$event = calendar_event::load($eventid);
$event->delete($repeats);
```

E' possibile inserire un azione nell'evento definendo come tipo `CALENDAR_EVENT_TYPE_ACTION`, il callback dell'azione andra' nel file `lib.php` con il nome `mod_xyz_core_calendar_provide_event_action()`

La visibilita' degli eventi di un plugin mod e' determinata da metodi appositi in `lib.php`. E' possibile stabilire quando un evento e' visibile sul sito con il metodo `mod_xyz_core_calendar_is_event_visible()`, e' possibile stabilire anche un contatore nell'evento, se applicabile, con il metodo `mod_xyz_core_calendar_event_action_shows_item_count()`.

Se si intende usare gli eventi bisogna prestare attenzione che la loro visualizzazione dipende anche da come il calendario e' visualizzato. Ad esempio la documentazione indica che gli eventi `CALENDAR_EVENT_TYPE_STANDARD` non sono indicati nella overview, cosi' come eventi con action nulla. Altri plugin avranno altre loro logiche.

Valutazione avanzata
--------------------

[Documentazione](https://moodledev.io/docs/apis/core/grading)

Le attivita' di un corso possono supportare modalita' di valutazione avanzata nel metodo 

```php
// mod/[activityname]/lib.php

function [activityname]_supports(string $feature): ?bool {
    switch ($feature) {
        // ...
        case FEATURE_ADVANCED_GRADING:
            return true;
        // ...
    }
}
```

La valutazione avanzata inserisce opzioni di valutazione custom (es: *insufficiente, sufficiente, buono, distinto, ottimo*), con possibilita di definire un rendere apposito. Per maggiori informazioni consultare la documantazione.

Sottoplugin
-----------

E' possibile prevedere la presenza di sottoplugin per i nostri plugin che sviluppiamo.

Un sottoplugin di fatto non e' diverso da un plugin normale, eccetto che il suo tipo e' definito dal plugin padre.

La definizione dei tipi di sottoplugin viene fatta nel plugin padre nel file `db/subplugins.json`. Qui definiremo il folder in cui andranno messi i plugin di quel particolare tipo

```json
{
    "plugintypes": {
        "workshopform": "mod/workshop/form",
        "workshopallocation": "mod/workshop/allocation",
        "workshopeval": "mod/workshop/eval"
    }
}
```

Bisogna prestare attenzione a non scegliere nomi per i tipi di sottoplugin che collidano con tipi gia' esistenti, inclusi altri possibili sottoplugin, con la complicazione aggiuntiva di non poter usare nomi lunghi per non aggravare il problema dei nomi delle tabelle.

Gli unici tipi di plugin che supportano i sottoplugin sono i tipi `mod`, `local`, `admin` e gli editor html.

Moodle ha un supporto limitato out of the box per i sottoplugin, sono solo presenti nella lista generale dei plugin e viene dato acccesso al pulsante di disinstallazione, ma ad esempio non viene creata la maschera dei settaggi automaticamente. La aspettativa e' che sia il plugin padre a prendersi cura di gestire i sottoplugin. Incluse eventuali API a disposizione.

I nomi per le tipologie di sottoplugin vanno inserite nel plugin padre

```php
$string['subplugintype_workshopallocation'] = 'Submissions allocation method';
$string['subplugintype_workshopallocation_plural'] = 'Submissions allocation methods';
$string['subplugintype_workshopeval'] = 'Grading evaluation method';
$string['subplugintype_workshopeval_plural'] = 'Grading evaluation methods';
$string['subplugintype_workshopform'] = 'Grading strategy';
$string['subplugintype_workshopform_plural'] = 'Grading strategies';
```

I sottoplugin hanno senso se si prevede la possibilita' di dare ad altri sviluppatori la possibilita' di estendere o provvedere canali alternativi per i nostri plugin, o se si vuole dare una serie di opzioni installabili separatamente agli utenti delle piattaforme.

Moodle prevede gia' alcune classi di sottoplugin, alcune di queste con documentazione ufficiale: il modulo di "compito" ha una classe di plugin `assignsubmission`([link](https://moodledev.io/docs/apis/plugintypes/assign/submission)) per le tipologie di compito e `assignfeedback`([link](https://moodledev.io/docs/apis/plugintypes/assign/feedback)) per assegnare i voti

Tag
----

I Tag sono trasversali nella piattaforma e possono essere applicati a vari tipi di entita', come corsi, utenti, post, attivita' ecc...

E' possibile indicare delle entita' che si intende rendere disponibili ai tag indicandole nel file `db/tag.php`, per riferimento qui di seguito presento il file con i tag di moodle core che contiene la documentazione

```php
/**
 * Tag area definitions
 *
 * File db/tag.php lists all available tag areas in core or a plugin.
 *
 * Each tag area may have the following attributes:
 *   - itemtype (required) - what is tagged. Must be name of the existing DB table
 *   - component - component responsible for tagging, if the tag area is inside a
 *     plugin the component must be the full frankenstyle name of the plugin
 *   - collection - name of the custom tag collection that will be used to store
 *     tags in this area. If specified aministrator will be able to neither add
 *     any other tag areas to this collection nor move this tag area elsewhere
 *   - searchable (only if collection is specified) - wether the tag collection
 *     should be searchable on /tag/search.php
 *   - showstandard - default value for the "Standard tags" attribute of the area,
 *     this is only respected when new tag area is added and ignored during upgrade
 *   - customurl (only if collection is specified) - custom url to use instead of
 *     /tag/search.php to display information about one tag
 *   - callback - name of the function that returns items tagged with this tag,
 *     see core_tag_tag::get_tag_index() and existing callbacks for more details,
 *     callback should return instance of core_tag\output\tagindex
 *   - callbackfile - file where callback is located (if not an autoloaded location)
 *
 * Language file must contain the human-readable names of the tag areas and
 * collections (either in plugin language file or in component language file or
 * lang/en/tag.php in case of core):
 * - for item type "user":
 *     $string['tagarea_user'] = 'Users';
 * - for tag collection "mycollection":
 *     $string['tagcollection_mycollection'] = 'My tag collection';
 *
 * @package   core
 * @copyright 2015 Marina Glancy
 * @license   http://www.gnu.org/copyleft/gpl.html GNU GPL v3 or later
 */

defined('MOODLE_INTERNAL') || die();

$tagareas = [
    [
        'itemtype' => 'user', // Users.
        'component' => 'core',
        'callback' => 'user_get_tagged_users',
        'callbackfile' => '/user/lib.php',
        'showstandard' => core_tag_tag::HIDE_STANDARD,
    ],
    [
        'itemtype' => 'course', // Courses.
        'component' => 'core',
        'callback' => 'course_get_tagged_courses',
        'callbackfile' => '/course/lib.php',
    ],
    [
        'itemtype' => 'question', // Questions.
        'component' => 'core_question',
        'multiplecontexts' => true,
    ],
    [
        'itemtype' => 'post', // Blog posts.
        'component' => 'core',
        'callback' => 'blog_get_tagged_posts',
        'callbackfile' => '/blog/lib.php',
    ],
    [
        'itemtype' => 'blog_external', // External blogs.
        'component' => 'core',
    ],
    [
        'itemtype' => 'course_modules', // Course modules.
        'component' => 'core',
        'callback' => 'course_get_tagged_course_modules',
        'callbackfile' => '/course/lib.php',
    ],
];
```

I tag vengono gestiti in amministrazione del sito -> aspetto -> manage tags

Campi custom
------------

I campi custom sono aggiungibili a corsi ed utenti, ma e' possibile indicare nei nostri plugin altre entita' che vogliamo utilizzino i campi custom, come delineato in questo [link](https://moodledev.io/docs/apis/core/customfields).

Il vantaggio di cio' e' permetterci di utilizzare il lvaro gia' fatto da moodle a riguardo, inclusa la possibilita' di definire nuove tipologie di campi custom tramite plugin aggiuntivi.

------------------

Plugin attivita
===============

[Documentazione ufficiale](https://moodledev.io/docs/apis/plugintypes/mod)

I plugin di attivita', (`mod`) sono la prima tipologia di plugin storicamente introdotta su moodle, e di fatto una delle piu' espanse.

Per ragioni storiche, alcune regole e prassi di moodle sono leggermente diverse per i plugin mod, in particolare nella nomenclatura "Frankenstyle" il `mod_` va omesso nei nome delle tabelle del database.

i plugin di tipo attivita' hanno esigenze diverse particolari rispetto ai plugin base, ad esempio richiedono la presenza di una tabella dal nome identico al nome plugin, con i seguenti campi almeno:

* **id** *INT(10) auto sequence* 
* **course** *INT(10) foreign key course(id)*
* **name** *CHAR(255)* il nome indicato dall'utente per l'istanza dell'attivita' 
* **timemodified** *INT(10)* data ultima modifica
* **intro** *TEXT* la descrizione dell'istanza dell'attivita'
* **introformat** *INT(4)* formato del testo di sopra

inoltre richiedono i seguenti files:

* `mod_form.php` deve contenere un form dal nome `mod_[modname]_mod_form` usato nella creazione di una nuova istanza
* `index.php` che dovrebbe essere usata per elencare tutte le istanze della nostra attivita' alla quale l'utente puo' accedere nel corso

e le seguenti funzioni in `lib.php`:

```php
function [modname]_add_instance($instancedata, $mform = null): int;
function [modname]_update_instance($instancedata, $mform): bool;
function [modname]_delete_instance($id): bool;
```

Queste funzioni vengono chiamate quando viene aggiunta, modificata o eliminata una istanza della nostra attivita', e vanno usate per gestire i dati nelle nostre tabelle. Il form usato e' quello di sopra.

Condizioni completamento
------------------------

[link](https://moodledev.io/docs/apis/core/activitycompletion)

Di base i nuovi plugin mod supportano solo il completamento manuale, ovvero la possibilita' per l'utente di marcare l'attivita' come completata premendo un pulsante.

Per abilitare altre modalita' e' necessario implementare in `lib.php` una funzione `[nomeplugin]_supports`

```php
function forum_supports(string $feature): bool {
    switch($feature) {
        case FEATURE_COMPLETION_TRACKS_VIEWS:
            return true;
        case FEATURE_COMPLETION_HAS_RULES:
            return true;
        default:
            return null;
    }
}
```

Sono disponibili molte modalita' di completamento del corso, per istruzioni dettagliate su come implementarle si rimanda alla documentazione

Condizioni accesso
------------------

[link](https://moodledev.io/docs/apis/core/conditionalactivities)

Altro elemento simile e' quello delle condizioni di accesso all'attivita', ovveroc come permettere all'utente di vedere l'attivita'.

A differenza del completamento queste sono configurate dal sito. Sono gia' presenti modalita' come accesso da particolare data, accesso condizionato in base al completamento di attivita' precedenti, ecc..

Esiste una classe di plugin `availability` per definire condizioni addizionali, ad esempio accesso dopo pagamento. Vedi [qui](https://moodledev.io/docs/apis/plugintypes/availability) per la documentazione 

Backup
------

i moduli attivita' ed i blocchi supportano la feature di backup, usata per copiare il corso con tutto il contenuto. Per utilizzarla e' necessario che il plugin supporti le API corrispondenti ([link](https://docs.moodle.org/dev/Backup_2.0_for_developers))

------------------

Altri Tipi plugin
===========

Carellata di altre tipologie di plugin

Blocchi
-------

[Documentazione](https://moodledev.io/docs/apis/plugintypes/blocks)

I blocchi sono elementi di html autocontenuti che possono essere aggiunti in varie pagine.

Dove i blocchi possono essere aggiunti dipende dal tema. In Boost i blocchi sono aggiungibili solo in un drawer apposito o nella dashboard, ma ci sono temi aggiuntivi con altri meccanismi, che ad esempio permettono di aggiungere blocchi in qualsiasi pagina.

![blocchi](https://teaching.nmc.edu/wp-content/uploads/2023/07/01_AddingaBlock.jpg)

I blocchi hanno un loro livello di contesto specifico, ogni istanza ha quindi il suo contesto diverso per le autorizzazioni, ed e' in grado di ereditarle dal contesto in cui e' contenuto (esempio da sito o corso).

I blocchi sono molto utili nella creazione di plugin per funzionalita' specifiche del sito rivolto ad un utente medio, soprattutto se si vuole avere una interfaccia customizzata diversa da quella del sito.

Notabile e' il blocco di tipo "testo" che permette di renderizzare html, e di fatto puo' essere usato per eseguire customizzazioni in maniera semplice. Attenzione che questo blocco non salva i dati in chiaro sul database ma esegue delle serializzazioni (base64), questo puo' essere un problema se sono stati fatti blocchi con dei link statici non relativi perche' non vi permette di identificare facilmente sul database cosa modificare in caso di cambio di dominio.

Atto
----

*Atto* e' l'editor di testo usato nei campi "textarea" di moodle, e' di fatto un editor di html con alcuni pulsanti per alcune stilizzazioni comuni (es grassetto e corsivo).

Esiste tutta una classe di plugin che estendono Atto con nuove funzionalita', trattandosi di un editor di html vi sono moltissime potenzialita'. Ad esempio esiste un plugin per generare riunioni teams che e' un plugin di tipo atto (che genera degli iframe)

Course format
------------

I plugin di course format sono una sepcie di tema per la pagina del corso, permettono di stabilire come le varie attivita' siano visualizzate

![esempio di plugin tiles](https://www.elearningworld.org/wp-content/uploads/2019/07/tiles_boost_theme.jpg)

------------------

Moodle Mobile
=============

Esiste una versione di moodle sotto forma di app per android ed ios.

![alt text](https://moodledev.io/assets/images/MoodleApp40Store-77e90a1a541782a8e9b23afc7f2f10f4.webp)

L'app e' open source, ma realisticamente dovrete utilizzare quella pubblicata dalla Moodle stessa a meno di censirvi sugli store android ed apple o fornire apk custom. Quando un utente scarica l'app "Moodle" da uno di questi store sta scaricando quella ufficiale.

L'app puo' connettersi ad una installazione esistente di moodle, e permettere agli utenti di scaricare i corsi e consultarli offline. E' possibile consumare piu' piattaforme da una singola app.

Moodle offre diversi piani di utilizzo dell'app

-	Il piano FREE e’ gratuito e permette l’accesso ai contenuti della piattaforma. Gli utenti possono tenere offline al massimo due corsi alla volta e vi e’ un limite di 50 utenti notificabili al mese
-	Il piano PRO costa 200€ all’anno e aumenta i limiti a 4 corsi e 500 utenti notificabili, oltre che sbloccare altre funzionalita’
-	Il piano PREMIUM costa 500€ all’anno non ha limiti su corsi, notifiche, ecc. , e permette di customizzare il colore presentato nell’app
-	La versione BRANDIZZATA consiste in un app apposita, separata da quella standard, con possibilita’ di modificare il logo, il nome sullo store, ed altro. Viene sviluppata su richiesta dalla Moodle e bisogna contattare il loro reparto vendite per un preventivo

Nota che il costo e’ solo per il propietario del sito, l’app Moodle per gli utenti e’ gratuita.

La versione free e' abbastanza limitata nelle capacita', in particolare richiede che l'utente inserisca l'url del sito a mano per collegarsi la prima volta, mentre con la versione pro e' possibile inserire il sito in una lista filtrabile visibile sull'app, o permettere la login scansionando un qr code generabile su moodle.

Per abilitare la comunicazione con l'app e' necessario attivare i web services sul sito seguendo un apposita procedura.

L'aspetto dei contenuti non e' customizzabile ma e' dettato dall'app, avete quindi una visualizzazione di default. Bisogna verificare poi quali attivita' sono compatibili con l'app e quali no. Mentre possiamo aspettarci che quelle core di Moodle siano tutte ben compatibili, non e’ garantito per plugin aggiuntivi. Le attivita’ non compatibili lo dicono e riportano un pulsante che apre la pagina corrispondente nel browser. Per rendere un plugin compatibile con il mobile vedi [qui](https://moodledev.io/general/app/development/plugins-development-guide)

Chiaramente nulla vieta ad un utente di collegarsi alla piattaforma via il browser del porprio dispositivo mobile, senza bisogno di usare l'app. E' necessario che il tema sia responsive e fruibile su taglie xs.

Un'altra considerazione per l'app e' la dimensione complessiva dei contenuti dei corsi, magari evitando video 8k con un docente che mostra un powerpoint

------------------

Altro
=====

Upload utenti massivo
---------------------

[Documentazione](https://docs.moodle.org/403/en/Upload_users)

Nel menu di amministrazione di moodle e' presente un tool di caricamento massivo degli utenti tramite file csv

Il tool e' molto potente, in quanto potete specificare quanti pochi o molti campi dell'utente volete modificare, inclusi quelli personalizzati.

Il tool permette inoltre l'iscrizione di tali utenti ai corsi, e puo' essere usato per iscrivere utenti gia' esistenti ai corsi. Puo' inoltre assegnare ruoli di sistema, categoria o corso, ed inserire utenti in gruppi globali.

In fase di caricamento sono disponibili diverse opzioni, ad esempio per stabilire come comportarsi con utenti esistenti, ed e' disponibile una preview prima di persistere l'importazione.

XHProf
------

[link](https://www.php.net/manual/en/intro.xhprof.php)

Si tratta di un estensione per eseguire la profilazione delle risorse usate dal php a livello di singole funzioni. Vi e' un'alternativa chiamata tideways ([link](https://tideways.com/profiler/features)) a pagamento con sorgenti open source.

Moosh
-----

[link](https://moosh-online.com/)

Moosh e' un tool di amministrazione di moodle a linea di comando, va installato a parte sul server dove avete la piattaforma. Ha una serie di comandi utili che eseguono operazioni a volte non accessibili altrove. Ad esempio il comando

```
moosh delete-missingplugins
```

Elimina tutti i riferimenti a plugin la cui cartella non e' piu' presente su disco (ovvero non sono stati disinstallati correttamente)

Ambiente demo
-------------

Moodle mette a disposizione un ambiente di demo ([link](https://moodle.org/demo)), si tratta di una versione con alcuni corsi di esempio con il tema boost. E' un buon esempio per vedere le funzionalita' complete del pacchetto in quanto trovate tutto configurato.

E' possibile loggarsi con vari ruoli (amministratore, docente, studente, ecc). Le modifiche effettuate vengono revertate periodicamente. E' un sito pubblico non avete garanzia che qualche altro utente non lo abbia reso inutilizzabile o vandalizzato non usatelo per presentazioni importanti.

------------------
 


Features non incluse in moodle 4.1
==================================

Una lista di alcune delle features che non sono incluse nella versione 4.1:

Dalla versione 4.4 ([link](https://moodledev.io/docs/devupdate)):

* Dependency injection ([link](https://moodledev.io/docs/apis/core/di)) 
* Attributi PHP
* Possibilita' di iscrivere utenti ai corsi con piu' metodi nell'upload massivo
* Pagine dedicate per sezione di corso

Dalla versione 4.3 ([link](https://moodledev.io/docs/4.3/devupdate)):

* Raddoppio dei caratteri utilizzabili per nome tabelle e campi
* Hooks, rimpiazzo delle funzioni in `lib.php` ([link](https://moodledev.io/docs/apis/core/hooks)), ulteriormente migliorati nella 4.4
* possibilita di avere un footer "sticky" nei form

Dalla versione 4.2 ([link](https://moodledev.io/docs/4.2/devupdate))

* Modifica sostanziale dell'attivita' **Quiz**
* spostamento delle API core dei web service, non serve piu' includere il file `/lib/externallib.php` 

