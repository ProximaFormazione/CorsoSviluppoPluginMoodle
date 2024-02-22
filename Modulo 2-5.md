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

Settaggi
========

I settaggi di un plugin consistono in una serie di configurazioni che vengono salvate in una coppia chiave/valore nel database.

Moodle ha una serie di funzioni che permettono il salvataggio e caricamento dei settaggi, oltre ad un robusto meccanismo per la creazione di menu appositi.

Unica cosa da fare e' definire i settaggi in un apposito file denominato `settings.php`.

I settaggi hanno una struttura simile all'albero della navigazione, e vanno ad incidere sulla variabile `$ADMIN`, alla quale aggiungeremo una pagina standard coi settaggi, con un oggetto di classe `admin_settingpage`.

Dentro questa classe aggiungeremo i vari settaggi usando il tipo di controllo che vogliamo fare usare all'utente per impostare il messaggio

Sembra complesso, ma una volta impostato una volta la struttura e' identica e si puo' prendere a modello per progetti successivi.

Ad esempio inseriamo un settaggio per decidere se fare visualizzare il nostro plugin nella navigazione o no.

L'esatta sintassi da utilizzare varia a seconda del plugin, per un plugin di tipo local ad esempio possiamo modificare l'albero dei settaggi ed aggiungere le voci

```php
defined('MOODLE_INTERNAL') || die();

if ($hassiteconfig) {
    $ADMIN->add('localplugins', new admin_category('local_helloworld_settings', new lang_string('pluginname', 'local_helloworld')));
    $settingspage = new admin_settingpage('managelocalhelloworld', new lang_string('manage', 'local_helloworld'));

    if ($ADMIN->fulltree) {
        $settingspage->add(new admin_setting_configcheckbox(
            'local_helloworld/showinnavigation',
            new lang_string('showinnavigation', 'local_helloworld'),
            new lang_string('showinnavigation_desc', 'local_helloworld'),
            1
        ));
    }

    $ADMIN->add('localplugins', $settingspage);
}
```

* `local_helloworld/showinnavigation` e' il nome del settaggio da usare per richiamarlo
* `$hassiteconfig` e' true se l'utente ha capacita' amministrative sul sito
* `$ADMIN->fulltree` serve a verificare se stiamo visualizzando l'intero albero di navigazione. In modo da evitare di processare codice che non serve
* Tutte le stringhe vanno definite nei file della lingua

mentre per un plugin di tipo enrol conviene invece solo aggiungere elementi alla variabile `$settings`

```php
 defined('MOODLE_INTERNAL') || die();

 if ($ADMIN->fulltree) {
    $settings->add(new admin_setting_configcheckbox(
        'enrol_magiclink/showinnavigation',
        get_string('showinnavigation', 'enrol_magiclink'),
        get_string('showinnavigation_desc', 'enrol_magiclink'),
        1
    ));
}
```

Per consultare un settaggio sara' sufficiente utilizzare il metodo `get_config('nome_plugin','settaggio')`

Ad esempio possiamo inserire un check nei metodi che aggiungono la navigazione nel nostro caso:

```php
function enrol_magiclink_extend_navigation_frontpage(navigation_node $frontpage) {
    if(get_config('enrol_magiclink','showinnavigation') == '1'){
        $frontpage->add(
            get_string('pluginname', 'enrol_magiclink'),
            new moodle_url('/enrol/magiclink/helloworld.php')
        );
    }
}
```

I settaggi sono salvati come stringhe.

[Documentazione Ufficiale](https://moodledev.io/docs/apis/subsystems/admin)