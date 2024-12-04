Navigazione
===========

[Documentazione Ufficiale](https://moodledev.io/docs/apis/core/navigation)

Moodle ha un sistema condiviso per gestire i link di navigazione all'interno delle pagine.

Come poi il menu di navigazione e' presentato dipende dal tema in uso. La struttura qui descritta e' solo la struttura interna di moodle, non prevede automaticamente un metodo di visualizzazione (Poi di fatto tutti i temi la usano in qualche modo)

Vi sono tre livelli di navigazione: Primario, secondario e terziario. Questi livelli vengono rappresentati con consecutivo ordine di importanza.  

Nel tema Boost la navigazione primaria e' la tabview in testata con il link per l'amministrazione del sito, la navigazione secondaria e' la tabview sotto il titolo del corso, mentre la navigazione terziaria, quando presente, e' una combobox

Esiste poi un secondo sistema di navigazione per i settaggi.

la struttura di navigazione si raggiunge nelle propieta':

* `$PAGE->navigation` per la struttura principale
* `$PAGE->settingsnav` per la navigazione dei settaggi
* `$PAGE->navbar` per le breadcrumb di navigazione, che e' il "path" logico della pagina, permette di risalire di un livello (es: da un attivita' al corso)

La navigazione 'e contestuale ad altri elementi contenuti in `$PAGE`:

* vengono letti `context` , il `course` ed eventualmente il `cm` (id del'attivita' specifica)
* il match per identificare la posizione attuale e' tramite `$PAGE->url`

Per alterare la struttura della navigazione un modo possibile sarebbe usare il metodo `$PAGE->navigation->add()`, tuttavia cio' e' possibile solo all'interno del nostro codice, rendendolo quindi un metodo poco adatto per aggiungere elementi di navigazione che puntino alle nostre pagine (l'utente vedrebbe la navigazione solo una volta arrivato).

Per ovviare a questo dovremo implementare degli **Hooks** (dalla 4.3 in poi) oppure dei **Callbacks** (fino alla 4.2), oppure entrambi per avere compatibilita' maggiore. Indipendentemente dal metodo scelto dovremo utilizzare la funzione `add()` del nodo di navigazione ed aggiungere il nostro nodo

```php
$nodo->add(
    get_string('pluginname', 'local_anagrafe'),
    new moodle_url('/local/anagrafe/helloworld.php')
);
```

Blocco di navigazione
---------------------

Il blocco "navigazione" e' un blocco che visualizza l'albero della navigazione leggendolo da `$PAGE->navigation`.

> Il blocco navigazione non e' visualizzato nel tema di default di moodle 4, ma lo e' su versioni piu' vecchie o altri temi

per estendere il contenuto del blocco di navigazione dobbiamo usare il callback `extend_navigation`. Questo hook pero' non e' disponibile a tutti i plugin, ma puo' ad esempio essere usato in un plugin `local`

```php
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
