Navigazione
===========

[Documentazione Ufficiale](https://moodledev.io/docs/apis/core/navigation)

Moodle ha un sistema condiviso per gestire i link di navigazione all'interno delle pagine.

Come poi il menu di navigazione e' presentato dipende dal tema in uso.

Vi sono tre livelli di navigazione: Primario, secondario e terziario. Questi livelli vengono rappresentati con consecutivo ordine di importanza.  

Esiste poi un secondo sistema di navigazione per i settaggi.

la struttura di navigazione si raggiunge nelle propieta':

* `$PAGE->navigation` per la struttura principale
* `$PAGE->settingsnav` per la navigazione dei settaggi
* `$PAGE->navbar` per le breadcrumb di navigazione, che e' il "path" logico della pagina, permette di risalire di un livello (es: da un attivita' al corso)

La navigazione 'e contestuale ad altri elementi contenuti in `$PAGE`:

* vengono letti `context` , il `course` ed eventualmente il `cm` (id del'attivita' specifica)
* il match per identificare la posizione attuale e' tramite `$PAGE->url`

Per alterare la struttura della navigazione un modo possibile sarebbe usare il metodo `$PAGE->navigation->add()`, tuttavia cio' e' possibile solo all'interno del nostro codice, cioe' dove l'utente dovrebbe arrivare tramite la navigazione che dobbiamo aggiungere.

Per ovviare a questo sono a disposizione dei **Callbacks** ([Documentazione](https://docs.moodle.org/dev/Callbacks)), ovvero funzioni che vengono chiamate dal codice core di moodle.

I callbacks devono aderire a rigide regole di nomenclatura:

1. Devono essere definiti come funzioni nello scope globale di un file chiamato `lib.php` posizionato nella root del plugin
2. Devono avere un nome nel formato **nome_plugin**_**callback** e rispettare la firma del tipo di callback usato

non tutti i plugin possono utilizzare tutti i tipi di callback, ma per iniziare i seguenti sono disponibili a tutti:

* `extend_navigation_frontpage`
* `extend_navigation_course`
* `extend_navigation_user`

ad esempio se volessimo aggiungere un link al nostro plugin definiremmo la seguente funzione in `lib.php`

```
function enrol_magiclink_extend_navigation_frontpage(navigation_node $frontpage) {
    $frontpage->add(
        get_string('pluginname', 'enrol_magiclink'),
        new moodle_url('/enrol/magiclink/helloworld.php')
    );
}
```

Blocco di navigazione
---------------------

Il blocco "navigazione" e' un blocco che visualizza l'albero della navigazione leggendolo da `$PAGE->navigation`.

> Il blocco navigazione non e' visualizzato nel tema di default di moodle 4, ma lo e' su versioni piu' vecchie o altri temi

per estendere il contenuto del blocco di navigazione dobbiamo usare il callback `extend_navigation`. Questo hook pero' non e' disponibile ai plugin di tipo `enrol`, ma puo' ad esempio essere usato in un plugin `local`

```
function local_greetings_extend_navigation(global_navigation $root) {
    $node = navigation_node::create(
        get_string('greetings', 'local_greetings'),
        new moodle_url('/local/greetings/index.php'),
        navigation_node::TYPE_CUSTOM,
        null,
        null,
        new pix_icon('t/message', '')
    );

    $root->add_node($node);
}
```

Potete eventualmente introdurre un plugin local solo per avere questo pulsante (o fare un plugin generico che controlla la presenza di vostri altri plugin e nel caso aggiunge le navigazioni richieste)

Prima di moodle 4
-----------------

Il sistema di navigazione 'e stato cambiato con il passaggio a moodle 4.

Nelle versioni precedenti, come la 3.11, si avevano tre elementi di navigazione

* **Navigation** che era il menu generico visualizzato
* **Breadcrumb** 
* **Settings** per le pagine che prevedevano un menu di settaggi

Gestire l'input utente
======================

Nel PHP per recuperare parametri dalla richiesta http si possono usare le variabili superglobali come `$_GET` o `$_POST`, tuttavia in moodle e' presente una serie di metodi per semplificare e garantire la sicurezza, per cui e' sconsigliato richiamare i superglobali.

Ricordate che questi parametri possono essere creati a piacimento in una richiesta http ad hoc ad opera di untenti malintenzionati, per cui garantire la sicurezza degli input e' un dovere a carico dello sviluppatore.

Essendo una soluzione open source, in moodle vi e' una grande attenzione all'aspetto sicurezza, ed eventuali falle sono impedite. 

La lettura e caricamento dei parametri e' eseguita richiamando il file `config.php`. Per ottenere un parametro in una pagina si consiglia quindi di utilizzare uno dei metodi inseriti.

`required_param()` restituisce il valore del parametro con il nome indicato, restituendo un errore qualora mancasse, mentre `optional_param()` vi permette invece di specificare un valore di default in caso il parametro mancasse.

Questi metodi gestiscono parametri in GET e POST.

Per utilizzare questi metodi e' necessario fornire un valore addizionale che caratterizza la tipologia di parametro atteso. Queste costanti sono definite in [`lib/moodlelib.php`](https://github.com/moodle/moodle/blob/master/lib/moodlelib.php) ma alcuni esempi sono:

* `PARAM_INT` per valori interi
* `PARAM_ALPHA` per valori [a-z, A-Z]
* `PARAM_BOOL` per valori booleani (converte roba tipo "true", "yes", "off" in 0 o 1)
* `PARAM_NOTAGS` rimuove i tag html dall'input

Cercate di usare la definizione piu' stringente per il vostro caso, questo per evitare comportamenti imprevisti o malevoli (es: iniezione di html)

Forms
-----

Per la creazione di form html moodle mette a disposizione un meccanismo ad hoc che semplifica l'implementazione, e che permette di delegare al tema i dettagli della rappresentazione, creando quindi omogeneita' con il resto del sito.

Tutto ruota intorno alla classe `moodleform` che fornisce il framework di base, voi dovete solo ereditare da questa classe e definire i vostri elementi facendo l'override del metodo `definition() : void` ed aggiungendo elementi alla propieta' `$_form` della classe.

La classe e' definita nel file di moodle core `lib/formslib.php` che va incluso nel file con la classe del nostro form

Esempio:

```php
require_once("$CFG->libdir/formslib.php");

class myform extends moodleform {
    public function definition() {

        // A reference to the form is stored in $this->form.
        // A common convention is to store it in a variable, such as `$mform`.
        $mform = $this->_form; // Don't forget the underscore!


        // Add elements to your form.
        $mform->addElement('text', 'email', get_string('email'));
        // Set type of element.
        $mform->setType('email', PARAM_NOTAGS);
        // Default value.
        $mform->setDefault('email', 'Please enter email');
    }
}
```

per ogni elemento definiamo il nome del parametro, ed il tipo di controllo da visualizzare sul form. La lista completa dei controlli disponibili e' visibile sul codice o sulla [documentazione ufficiale](https://moodledev.io/docs/apis/subsystems/form). Ci sono molti cintrolli a disposizione.

Alla fine del metodo dobbiamo ricordarci di aggiungere i pulsanti per submit e cancel se previsto. Il metodo `$this->add_action_buttons();` e' una scorciatoia per fare cio'.

Utilizzare il form sulla pagina e' molto semplice, e' sufficiente instanziare la classe, e questa va a verificare da sola sulla pagina se esistono dei set di valori gia' postati, o se l'utente ha magari deciso di annullare.

```php
$mform = new \plugintype_pluginname\form\myform();
```

A questo punto di solito bisogna processare eventuale contenuto del form.

Il metodo `get_data()` restituisce i valori forniti dall'utente (sotto forma di stdClass) in caso di submit, oppure NULL se non e' questo il caso. Il metodo `set_data()` permette invece di valorizzare il form (ad esempio con un elemento caricato dal DB).

Il metodo `is_cancelled()` restituisce true se l'utente ha premuto il pulsante di cancellazione (se presente).

Per visualizzare il form e' sufficiente chiamare il metodo `$mform->display();` nella zona di OUTPUT della pagina. oppure si puo' ottentere l'html con il metodo `$mform->render();`

Un esempio di gestione del form quindi e' il seguente

```php

// [...]

// Istanzio il form
$mform = new \plugintype_pluginname\form\myform();

// Gestisco eventuali dati submittati
if ($mform->is_cancelled()) {
    // Eseguo le operazioni di annullamento
    // ad esempio redirect .....
    
} else if ($fromform = $mform->get_data()) {
    // qui abbiamo in $fromform un stdClass con tutti i valori del form
    // eseguiremo qualcosa .....

} else {
    // questo e' il caso dove non abbiamo valori nel form (prima apertura) oppure i valori non hanno passato la validazione.
    
    // Inserisco valori nel form, ad esempio i default
    $mform->set_data($toform);
}

// [...]


echo $OUTPUT->header();

// mostro il form
$mform->display();

echo $OUTPUT->footer();
```