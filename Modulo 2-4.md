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
