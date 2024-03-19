Web services
============

[Documentazione Ufficiale](https://moodledev.io/docs/apis/subsystems/external)

Moodle prevede la possibilita' di inserire dei **web service** nei plugin sviluppati. Si tratta di una funzionalita' core che poi potra' essere configurata assieme ai web services degli altri plugin nel menu di configurazione.

I web service sono anche usati internamente per alcune funzionalita' asincrone tramite AJAX e nell'app mobile di moodle.

Alternative
-----------

Siccome moodle e' scritto in php, nulla vieta di avere meccanismo di interazione con vostri sistemi scritto a mano con le vostre logiche, magari usando le API interne di moodle in uno script.

Questo approccio ha una serie di svantaggi:

- I web service in moodle ricevono una documentazione automatica visibile nel menu di amministrazione
- I web service di moodle utilizzano gli utenti gia' presenti e permettono ad un utente amministratore di gestire l'accesso, una soluzuione artigianale richiede che l'utente sia al corrente delle vostre logiche
- La definizione lato sviluppo e' relativamente semplice rispetto a dover implementare a mano autenticazione ed autorizzazione.
- Il protocollo di comunicazione viene lasciato a moodle, permettendo quindi ad esempio sia web service soap che rest in base alla configurazione del sito

L'unica vera controindicazione del sistema di web services di moodle e' che non e' semplicissimo da abilitare sul sito per un utente non informato, ma a questo ovvieremo con questo capitolo

Creare un web service in un plugin
----------------------------------

La creazione di un web service segue un po lo stesso approccio di eventi o task schedulati:

* Dobbiamo censire il nostro web service in array di un file specifico della cartella `db`
* Dobbiamo inserire una classe che discende da un tipo base, con una serie di metodi da implementare

la definizione va fatta nel file `db/services.php`, dove andremo a popolare l'array `$functions`. In questo array inseriremo come chiave il nome tecnico della funzione, che deve seguire alcune linee guida:

- deve iniziare con il nome(frankenstyle) del plugin (es: `enrol_magiclink_....`) per evitare collisioni con altri plugin
- deve essere seguito dal nome del metodo, nella forma `{verbo}_{nome}` (es: `enrol_magiclink_create_link`)

Vanno poi inserite le altre caratteristiche del servizio

```php
defined('MOODLE_INTERNAL') || die();

$functions = [
    // Nome tecnico dell'endpoint.
    'enrol_magiclink_create_link' => [
        / Il nome della classe (con namespace) con il codice
        'classname'   => 'enrol_magiclink\external\create_link',

        // Una breve descrizione per l'utente di cosa fa il servizio
        'description' => 'Crea un nuovo link',

        // read o write
        'type'        => 'write',

        // Se il servizio e' disponibile in chiamate AJAX (vengono fatte tramite apposito plugin)
        'ajax'        => true,

        // Lista di servizi opzionali in cui la funzione sara' inclusa
        'services'    => [
            // Una installazione base ha solo il servizio del mobile
            // - MOODLE_OFFICIAL_MOBILE_SERVICE.
            // specificare questa voce permette di utilizzare l'endpoint sull'app
            MOODLE_OFFICIAL_MOBILE_SERVICE,
        ],

        // la lista delle capacita' usate dall'endpoint. E' solo informativo per l'amministratore, non applica il controllo su quelle capacita' in automatico
        'capabilities' => 'moodle/course:creategroups,moodle/course:managegroups',

    ],
];
```

Queste informazioni verranno poi usate dall'interfaccia del menu' di amministrazione nel mostrare il servizio.

La classe con il servizio andra' definita correttamente nella posizione indicata da moodle:

- deve essere nel sottonamespace `external` del plugin
- deve essere messa in un file nella cartella `/classes/external`

Ad esempio, per la funzione di sopra `enrol_magiclink_create_link` creeremo una classe chiamata `enrol_magiclink\external\create_link` nel file `enrol/magiclink/classes/external/create_link.php`

La classe in questione deve soddisfare alcune caratteristiche:

- Deve estendere la classe `external_api` che si trova in `lib/externallib.php` (`require_once("$CFG->libdir/externallib.php");`) 
- Deve avere il metodo `execute_parameters` che indica i parametri in ingresso dell'endpoint
- Deve avere il metodo `execute_returns` che indica i valori di ritorno dell'endpoint
- Deve avere il metodo `execute` che e' il metodo lanciato quando viene chiamato l'endpoint.

E' poi possibile dichiarare un metodo `execute_is_deprecated` nel caso si volesse indicare che l'endpoint e' deprecato.

*NOTA: Moodle sta rivedendo la definizione dei web services, dalla versione 4.2 questi potranno essere definiti da classi contenute in namespace e si dora' estendere la classe `\core_external\external_api` invece che richiedere di includere i files in un `require_once`. Dalla versione 4.6 si prevede poi di inserire dei warning per le classi non in namespace per indicarne la deprecazione*

Un esempio preso dalla documentazione ufficiale:

```php
<?php

namespace local_groupmanager\external;

use external_function_parameters;
use external_multiple_structure;
use external_single_structure;
use external_value;

class create_groups extends \core_external\external_api {
    public static function execute_parameters(): external_function_parameters {
        return new external_function_parameters([
            'groups' => new external_multiple_structure(
                new external_single_structure([
                    'courseid'  => new external_value(PARAM_INT, 'The course to create the group for'),
                    'idnumber'    => new external_value(
                        PARAM_RAW,
                        'An arbitrary ID code number perhaps from the institution',
                        VALUE_DEFAULT,
                        null
                    ),
                    'name' => new external_value(
                        PARAM_RAW,
                        'The name of the group'
                    ),
                    'description' => new external_value(
                        PARAM_TEXT,
                        'A description',
                        VALUE_OPTIONAL
                    ),
                ]),
                'A list of groups to create'
            ),
        ]);
    }

    public static function execute(array $groups): array {
        // Validate all of the parameters.
        [
            'groups' => $groups,
        ] = self::validate_parameters(self::execute_parameters(), [
            'groups' => $groups,
        ]);

        // Perform security checks, for example:
        $coursecontext = \context_course::instance($courseid);
        self::validate_context($coursecontext);
        require_capability('moodle/course:creategroups', $coursecontext);

        // Create the group using existing Moodle APIs.
        $createdgroups = \local_groupmanager\util::create_groups($groups);

        // Return a value as described in the returns function.
        return [
            'groups' => $createdgroups,
        ];
    }

    public static function execute_returns(): external_single_structure {
        return new external_single_structure([
            'groups' => new external_multiple_structure([
                'id' => new external_value(PARAM_INT, 'Id of the created user'),
                'name' => new external_value(PARAM_RAW, 'The name of the group'),
            ])
        ]);
    }
}
```

I parametri sono definiti tramite delle classi apposite.

`external_function_parameters` e' per i parametri (valori in ingresso), che possono essere:

* Valori: `external_value`
* Oggetti: `external_single_structure`
* Liste di oggetti: `external_multiple_structure`

`external_value` ha un costruttore che chiede il tipo di parametro per la validazione/sanificazione dell'input (es: `PARAM_INT`) ed una stringa per la documentazione. (Da notare che persino la documentazione ufficiale qui usa stringhe secche invece che `get_string`).

Si puo' indicare se un parametro e' opzionale con i flag aggiuntivi nel costruttore: come terzo parametro possiamo indicare

* VALUE_REQUIRED (usato se non specificato diversamente)
* VALUE_OPTIONAL (non utilizzabile nel primo livello di alberatura)
* VALUE_DEFAULT (bisogna poi specificare il default)

ci sono poi altri parametri come `allownull`, si consiglia di vedere il codice a riguardo.

`external_single_structure` ed `external_multiple_structure` hanno un costruttore che vuole un array associativo con nella chiave il nome del parametro e come valore una delle classi di sopra. (anche `external_function_parameters`).

Il metodo di esecuzione avra' i parametri in un formato piu' consono al PHP, array per le liste ed oggetti/valori altrimenti

Nel metodo di esecuzione vero e proprio, dobbiamo eseguire noi validazione e controllo sull'autorizzazione. Per fare questo moodle mette a disposizione dei metodi nella classe base.

per la validazione abbiamo a disposizione `validate_parameters()`, che vuole come parametri la lista dei valori attesi (`self::execute_parameters()`) ed i valori proposti nel web service.

Per l'autorizzazione abbiamo a disposizione `validate_context()` per verificare che l'utente abbia accesso al contesto indicato (che va fornito alla funzione). 

ATTENZIONE: Nei web service NON va utilizzato `require_login` e NON va impostato il contesto in `$PAGE->set_context()` manualmente

Per verificare le capacita' dell'utente possiamo usare i metodi standard di moodle (es: `require_capability()`). Da notare che non dobbiamo inserire capacita' apposite per il webservice (la lista utenti abilitati e' gestita diversamente) ma dobbiamo solo indicare capacita' business.

Dobbiamo poi inserire la logica business del nostro servizio, eventuali eccezioni vengono raccolte e gestite automaticamente.

La validazione del tipo di ritorno e' fatta automaticamente da moodle. Da notare che gli oggetti andranno castati in array.

Abilitare i web service su moodle
---------------------------------

Abilitare l'uso dei web service in una piattaforma e' un'operazione da eseguire nel menu' amministrativo.

tutte le voci si trovano nel menu' amministrativo -> Server -> Web services.

La prima voce ha un'overview con due checklist delle operazioni da fare per abilitare i web services. Una checklist e' per l'accesso tramite token, la seconda e' per l'accesso tramite login utente. Le checklist sono copiate alla fine di questo documento

Moodle non ha abilitati tutti gli endpoint di default, ma prevede la possibilta' di definire dei **Servizi** che includono una o piu' funzioni abilitate, ed una lista opzionale di utenti che potranno ricevere il token per consumare il servizio. Esiste un apposita voce di menu' per gestire i servizi. Rilevante e' lo shortname che verra' poi utilizzata nelle chiamate al ws.

### Autenticazione

Per utilizzare il web service e' necessario fornire un token, questo token identifica un utente specifico per il servizio.

Ci sono due approcci possibili:

- Definire un utente dedicato al servizio in questione e distribuire il token a chi deve consumare il servizio
- Permettere agli utenti di crearsi loro i token

Il primo approccio richiede alcuni passaggi aggiuntivi in fase di configurazione del servizio, ma e' piu' semplice da mantenere una volta a regime. Con questo sistema non e' necessario che chi utilizzi il web service abbia un utente in moodle

Il secondo approccio e' piu' veloce da configurare, ma richiede che i client abbiano un utente moodle con le capacita' necessarie:

* `/webservice:createtoken` per creare i token
* accesso al protocollo/i usati dal servizio (`webservice/rest:use`, `webservice/soap:use`)
* tutte le capacita' richieste dalle funzioni da usare

Dovremo poi abilitare l'utente dedicato, o gli utenti specifici nel servizio. E' anche possibile lasciare il servizio aperto a tutti, in tal caso saranno le capacita' dell'utente (incluse quelle di sopra) a stabilire se hanno accesso o no

Consumare i web service
-----------------------

NOTA: nel menu' di amministrazione e' presente una voce con la documentazione delle funzioni attive sulla piattaforma, che include un [link alla documentazione ufficiale](https://docs.moodle.org/dev/Creating_a_web_service_client) su come creare un client, con tanto di esempi in vari linguaggi.

Per utilizzare un web service ci rivolgeremo all'endpoint

```
{url base}/webservice/{protocollo}/server.php
```

passando i vari parametri richiesti, tra cui:

* `wstoken` il token
* `wsfunction` la funzione da chiamare
* parametri specifici per il protocollo, ad esempio `moodlewsrestformat=json` o `wsdl=1`
* parametri per la funzione

Si consiglia di guardare gli esempi proposti sul repo di esempi ufficiale ([link](https://github.com/moodlehq/sample-ws-clients))

Il token puo essere creato in vari modi

* Puo' essere crerato da un amministratore dall'apposita voce dei web server, nota che per gli utenti amministratori e' l'unico modo abilitato
* Se l'utente ha la capacita' `/webservice:createtoken` puo generarsi i token da un apposita schermata nel profilo utente
* Se l'utente ha la capacita' `/webservice:createtoken` puo' ottenerla dallo script `/login/token.php` che richiede:
    * username
    * password
    * shortname del servizio

```
https://www.sitomoodle.it/login/token.php?username=USERNAME&password=PASSWORD&service=SERVICESHORTNAME
```

Checklist
---------

Queste checklist sono presenti nell'overview dei web service internamente alla piattaforma.

Per l'autenticazione tramite token:

1. **Abilita web service** I Web service si abilitano nelle Funzionalità avanzate.
2. **Abilita protocolli** Bisogna abilitare almeno un protocollo. Per motivi di sicurezza, dovresti abilitare solo i protocolli realmente necessari.
3. **Crea un utente specifico** È necessario creare un utente di sistema per il Servizio.
4. **Verifica i privilegi dell'utente** Per usare i web service, l'utente deve avere il privilegio corrispondente al protocollo in uso, per esempio 'webservice/rest:use', 'webservice/soap:use'. È possibile creare un ruolo globale 'Web service' con i privilegi relativi ai protocolli in uso, poi assegnare questo ruolo all'utente web service come ruolo globale.
5. **Seleziona un servizio** Un servizio è un insieme di funzioni web service. Devi abilitare l'utente ad accedere al servizio creato. Nella pagina Aggiungi servizio seleziona le opzioni 'Abilitato' e 'Solo utenti autorizzati'. Seleziona 'Nessun privilegio richiesto'.
6. **Aggiungi funzione** Scegli le funzioni per il web service creato
7. **Seleziona un utente specifico** Aggiungi l'utente web service tra gli utenti autorizzati.
8. **Crea un token per l'utente** Crea un token per l'utente web service
9. **Abilita documentazione per sviluppatori** Opzionale. È disponile la documentazione dettagliata dei web service relativa ai protocolli abilitati.
10. **Prova il funzionamento del servizio** Simula l'accesso al web service dall'esterno tramite il client di test. È possibile utilizzare un protocollo abilitato con l'autenticazione via token.Attenzione: la funzione SARA' REALMENTE ESEGUITA, fare molta attenzione su cosa si sceglie di provare.

Mentre per l'accesso tramite login utente


1. **Abilita web service** I Web service si abilitano nelle Funzionalità avanzate.
2. **Abilita protocolli** Bisogna abilitare almeno un protocollo. Per motivi di sicurezza, dovresti abilitare solo i protocolli realmente necessari.
3. **Seleziona un servizio** Un servizio è un insieme di funzioni web service. Devi abilitare gli utenti ad accedere al servizio creato. Nella pagina Aggiungi servizio seleziona l'opzione 'Abilitato' e disabilita l'opzione 'Solo utenti autorizzati'. Seleziona 'Nessun privilegio richiesto'.
4. **Aggiungi funzione** Scegli le funzioni per il web service creato
5. **Verifica i privilegi degli utenti** Per usare i web service, gli utenti devono avere i privilegi: '/webservice:createtoken' ed il privilegio corrispondente al protocollo in uso, ad esempio webservice/rest:use, 'webservice/soap:use'. È possibile creare un ruolo 'Web service' con gli opportuni privilegi ed assegnare questo ruolo all'utente web service a livello di sistema.
6. **Prova il funzionamento del servizio** Simula l'accesso al web service dall'esterno tramite il client di test. Prima di effettuare il test, autenticarsi con un account che abbia il privilegio "moodle/webservice:createtoken" e ottenere il token dell'utente, reperibile nelle impostazioni del profilo. Sarà possibile usare questo token nel client di test, dove si potrà selezionare anche il protocollo abilitato. Attenzione: la funzione SARA' REALMENTE ESEGUITA, fare molta attenzione su cosa si sceglie di provare.







 
