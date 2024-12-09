Interfaccia grafica
===================

Finora abbiamo realizzato delle pagine utilizzando la variabile `$OUTPUT` base e lanciando direttamente comandi con la classe `html_writer` nel codice della pagina.

Per plugin semplici questo approccio e' assolutamente valido, ma puo' valere la pena avere una struttura diversa, dove la logica della grafica e' separata dalla logica di backend.

Vediamo ora una veloce carellata dei sistemi proposti da moodle. Da notare che non e' necessario aderire a questo meccanismo, ed infatti e' pratica diffusa non farlo, ma e' utile conoscerlo in modo da poter interagire con i plugin che lo utilizzano.

abbiamo gia' visto l'uso della variabile `$PAGE` per caratterizzare alcuni "metadati" della pagina. Vediamo un esempio piu' completo suggerito dalla documentazione ufficiale.



```php
<?php
require_once(__DIR__ . '/../../../config.php');
require_once($CFG->libdir.'/adminlib.php');

require_login();

// Set up the page.
$title = get_string('pluginname', 'nome_plugin');
$pagetitle = $title;
$url = new moodle_url("/nome/plugin/demo/index.php");
$PAGE->set_url($url);
$PAGE->set_title($title);
$PAGE->set_heading($title);

$output = $PAGE->get_renderer('nome_plugin');

$dati = //[...] recupero dati necessari per la visualizzazione

echo $output->header();
echo $output->heading($pagetitle);

$renderable = new \nome_plugin\output\index_page($dati);
echo $output->render($renderable);

echo $output->footer();

```

Si tratta delle stesse istruzioni usate in precedenza per la nostra prima pagina del plugin, con l'aggiunta di due elementi:

* L'utilizzo di un **renderer** specifico invece che `$OUTPUT` (maiuscolo)
* L'utilizzo di una classe **renderable** per il contenuto della pagina

Renderer
--------

Il renderer e' l'elemento che provvede a fornire l'html della pagina. Richiede che gli venga fornito una classe con i dati da visualizzare sotto forma di renderable.

Storicamente il renderer era responsabile della produzione dell'html tramite uso della classe `html_writer` o simili. La prassi suggerita adesso e' l'utilizzo di un **template** scritto in mustache ([Link](https://mustache.github.io/mustache.5.html)).

Il vantaggio di avere un renderer e' che e' possibile eseguire un override di un qualsiasi renderer in un tema, permettendo quindi di alterare il meccanismo di presentazione.

un renderer deve ereditare dalla classe `plugin_renderer_base` e fornire i vari metodi per disegnare le pagine del plugin nella forma `render_<page>` dove al posto di page abbiamo un identificativo per il contenuto.

```php
<?php


namespace nome_plugin\output;

use plugin_renderer_base;

class renderer extends plugin_renderer_base {
    /**
     * Defer to template.
     *
     * @param index_page $renderable
     *
     * @return string html for the page
     */
    public function render_index_page($renderable): string {
        $data = $renderable->export_for_template($this);
        return parent::render_from_template('nome_plugin/index_page', $data);
    }
}
```

In questo caso il renderer utilizza il renderable nella variabile `$renderable` per farsi dare i dati, e poi usa un template per la visualizzazione. 

e' possibile omettere il renderer nel processo: avendo un renderable fatto a dovere con il nome della classe identico al nome del template si puo' utilizzare il renderer di default, ovvero `$OUTPUT`.


Renderable
----------

il Renderable e' semplicemente una classe che contiene i dati necessari per le logiche di visualizzazione, nella sua forma base e' una classe che implementa le interfacce `renderable` e `templatable`, che si porta il metodo `export_for_template`. Quest'ultimo metodo deve restituire un tipo non complesso (ovvero array, stdClass, bool, int, float o string) in quanto produce un JSON.

```php
<?php


namespace nome_plugin\output;

use renderable;
use renderer_base;
use templatable;
use stdClass;

class index_page implements renderable, templatable {
    /** @var string $sometext Some text to show how to pass data to a template. */
    private $dati = null;

    public function __construct($dati): void {
        $this->dati = $dati;
    }

    /**
     * Export this data so it can be used as the context for a mustache template.
     *
     * @return stdClass
     */
    public function export_for_template(renderer_base $output): stdClass {
        $data = new stdClass();
        $data->dati = $this->dati; // qui possiamo eseguire operazioni sui dati necessarie per le logiche di presentazione
        return $data;
    }
}
```

scopo del renderable e' eseguire tutte quelle operazioni di raffinamento dei dati per l'inrterfaccia grafica, in modo da non avere tali logiche nei metodi business.

Si puo' omettere il renderable e fornire al metodo `render` direttamente i dati, qualora fossero gia' formattati correttamente

Templates
---------

[Documentazione completa](https://moodledev.io/docs/guides/templates#template-files)

I template sono dei file scritti in mustache che contengono l'html della pagina. i templates devono essere contenuti nella cartella `templates` nella root del plugin e verranno richiamati per nome file.

la prassi suggerita da moodle e' chiamare il template all'interno del renderer tramite il metodo `parent::render_from_template('nome_plugin/index_page', $data)`. In alternativa e' possibile utilizzare il renderer di defult:

```php
$data = [
    'name' => 'Lorem ipsum',
    'description' => 'blablabla',
];

echo $OUTPUT->render_from_template($templatename, $data);
```

Per la sintassi completa del mustache si rimanda alla sua documentazione ufficiale, diciamo che si tratta di files in html con dei placeholder che vengono sostituiti dai dati passati dal renderable.

I placeholder sono contenuti da doppie parentesi graffe (i baffetti appunto)

```html
<p>{{name}}</p>
<p>{{description}}</p>
```

e' possibile avere delle sezioni per le liste di oggetti:

```html
<ul>
    {{#grades}}
    <li>
        <em>{{course}}</em> - {{grade}}
    </li>
    {{/grades}}
</ul>
```

```json
{
    "grades": [
        {
            "course": "Arithmetic",
            "grade": "8/10"
        },
        {
            "course": "Geometry",
            "grade": "10/10"
        }
    ]
}
```

il testo nella sezione verra' ripetuto per ogni elemento della lista, per false o liste vuote la sezione viene saltata. In questo modo e' possibile avere delle sezioni la cui presenza e' condizionale

Moodle ha una serie di metodi helper per il moustache, che usano la stessa sintassi delle sezioni.

### str

`{{#str}}` e' utilizzabile per avere le stringhe tradotte, esattamente come il metodo `get_string`

```html
{{#str}} helloworld, mod_greeting {{/str}}
```

### pix

equivalente alle clasi delle icone presenti in moodle ([link](https://docs.moodle.org/dev/Moodle_icons))

```
{{#pix}} t/edit, core, Edit this section {{/pix}}
```

### userdate

formatta un timestamp UNIX nel fuso orario dell'utente, accetta parametri per il formato 

```html
{{#userdate}} {{time}}, {{#str}} strftimedate, core_langconfig {{/str}} {{/userdate}}
```

in questo caso usiamo `strftimedate` estratta in base alla lingua (es: `"%d %B %Y"`)

### shortentext

Riduce un testo se eccede un certo numero di caratteri mettendo i puntini

```html
{{#shortentext}} 15, {{{description}}} {{/shortentext}}
```

### javascript

Il javascript puo' essere inserito in un'apposita sezione `{{#js}}`

``` html
<div id="block-recentlyaccesseditems-{{uniqid}}" class="block-recentlyaccesseditems block-cards" data-region="recentlyaccesseditems">
    <div class="container-fluid p-0">
        {{> block_recentlyaccesseditems/recentlyaccesseditems-view }}
    </div>
</div>
{{#js}}
require(
[
    'jquery',
    'block_recentlyaccesseditems/main',
],
function(
    $,
    Main
) {
    var root = $('#block-recentlyaccesseditems-{{uniqid}}');

    Main.init(root);
});
{{/js}}
```

Questo metodo fa si che il javascript venga inserito nel footer della pagina dopo il caricamento di requirejs.

L'uso del javascript in moodle non e' oggetto di questo corso, documentazione completa la potete trovare [qui](https://moodledev.io/docs/guides/javascript)

