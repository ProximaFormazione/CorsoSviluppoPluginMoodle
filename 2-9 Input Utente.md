Gestire l'input utente
======================

Nel PHP per recuperare parametri dalla richiesta http si possono usare le variabili superglobali come `$_GET` o `$_POST`, tuttavia in moodle e' presente una serie di metodi per semplificare e garantire la sicurezza, per cui e' sconsigliato richiamare i superglobali.

Ricordate che questi parametri possono essere creati a piacimento in una richiesta http ad hoc ad opera di utenti malintenzionati, per cui garantire la sicurezza degli input e' un dovere a carico dello sviluppatore.

Essendo una soluzione open source, in moodle vi e' una grande attenzione all'aspetto sicurezza, ed eventuali falle sono impedite. 

La lettura e caricamento dei parametri e' eseguita richiamando il file `config.php`. Per ottenere un parametro in una pagina si consiglia quindi di utilizzare uno dei metodi delle API di moodle.

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

La classe e' definita nel file di moodle core `lib/formslib.php` che va incluso nel file con la classe del nostro form (non e' gestito dall'autocaricamento delle classi)

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

per ogni elemento definiamo il nome del parametro, ed il tipo di controllo da visualizzare sul form. La lista completa dei controlli disponibili e' visibile sul codice o sulla [documentazione ufficiale](https://moodledev.io/docs/apis/subsystems/form). Ci sono molti controlli a disposizione.

Alla fine del metodo dobbiamo ricordarci di aggiungere i pulsanti per submit e cancel se previsto. Il metodo `$this->add_action_buttons();` e' una scorciatoia per fare cio'. Dalla versione 4.3 e' disponibile anche la funzione `add_sticky_action_buttons` per avere i pulsanti sempre visibili anche in caso di scorrimento verticale.

E' possibile inserire la validazione degli input lato client inserendo la funzione `validation($data, $files)`.

```php
class myform extends moodleform {
    public function definition() {

        // [...]

        function validation($data, $files) {

            // [...]

            return [];
            // oppure, in caso di errori:
            return ['element_name' => 'stringa errore', ...]
        }
    }
}
```

Utilizzare il form sulla pagina e' molto semplice, e' sufficiente instanziare la classe, e questa va a verificare da sola sulla pagina se esistono dei set di valori gia' postati, o se l'utente ha magari deciso di annullare.

```php
$mform = new \nome_plugin\form\myform();
```

A questo punto di solito bisogna processare eventuale contenuto del form.

Il metodo `get_data()` restituisce i valori forniti dall'utente (sotto forma di stdClass) in caso di submit, oppure NULL se non e' questo il caso. Il metodo `set_data()` permette invece di valorizzare il form (ad esempio con un elemento caricato dal DB).

Il metodo `is_cancelled()` restituisce true se l'utente ha premuto il pulsante di cancellazione (se presente).

Per visualizzare il form e' sufficiente chiamare il metodo `$mform->display();` nella zona di OUTPUT della pagina. oppure si puo' ottentere l'html con il metodo `$mform->render();`

Un esempio di gestione del form quindi e' il seguente

```php

// [...]

// Istanzio il form
$mform = new \nome_plugin\form\myform();

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