Hooks
=====

Gli hooks sono una nuova API in Moodle pensata per andare a sostituire i callbacks in modo da venire incontro ai principi del gruppo PHP-FIG([Link](https://www.php-fig.org/)), in particolare ai criteri per gli "Event dispatcher"([PSR-14](https://www.php-fig.org/psr/psr-14/)).

Un hook e ' un concetto assimilabile a quello di un *evento* in un linguaggio di programmazione, ovvero si tratta di un punto del codice dove altri componenti possono inserire dei metodi che soddisfino una certa firma.

In realta' in moodle esiste anche un meccanismo di **Eventi**, che e' descritto sotto. Le differenze saranno discusse in quel capitolo.

Come utilizzare un hook esistente
---------------------------------

per utilizzare un hook esistente dobbiamo per prima cosa definire un metodo statico che abbia come parametro l'hook in questione.

es:

```php
    public static function estendi_navigazione(\core\hook\navigation\primary_extend $hook): void {
        $primaryview = $hook->get_primaryview();
        $primaryview->add('Sono un link',new \moodle_url('/local/dispense/visiona.php', []));
    }
```

Una volta preparato il metodo, bisogna censirlo nell'hook. Per i plugin e' necessario definirlo in un apposito file `db/hooks.php` che deve contenere un array di nome `$callbacks` con le definizioni:

es:

```php
$callbacks = [

    [
        'hook' => core\hook\navigation\primary_extend::class,
        'callback' => 'local_dispense\local\hooks\navigation\classe_estensione::estendi_navigazione',
        'priority' => 0,
    ],
];
```

In questo caso abbiamo censito il metodo `estendi_navigazione` della classe `classe_estensione` inserendo l'informazione alla chiave `callback`

`hook` e' la chiave che indica l'hook al quale ci stiamo iscrivendo, `priority` e' invece un valore che entra in gioco per determinare l'ordine nel caso ci siano piu' metodi sottoscritti allo stesso hook: i metodi con un numero piu' alto vengono eseguiti prima.

Attenzione al nome del file (`hooks.php`) la posizione nella cartella corretta (dentro `db` nella root del plugin) ed il nome dell'array (`$callbacks`) perche' sono tutti elementi che se sbagliati non abilitano la sottoscrizione

Una lista degli hook disponibili e' visibile in Amministrazione del sito -> Sviluppo -> Hooks Overview. Per vedere eventuali propieta' o metodi esposti dall'hook bisogna consultarlo sul codice.

Dichiarare un nuovo hook
------------------------

Dichiarare un nuovo hook e' generalmente un'attivita' meno richiesta nello sviluppo di plugin allo scopo di personalizzare una specifica piattaforma, vediamo comunque come fare nel caso servisse.

### Definizione Hook

L'hook stesso e' una classe, che potete definire dove volete (ma si raccomanda di mettere in un namespace `nome_plugin\hook`). La classe con l'hook deve dichiarare una descrizione ed opzionalmente una lista di tag, entrambi usati nella maschera di riepilogo degli hook. Potete fare questa definizione tramite un attributo, o implementando l'interfaccia `\core\hook\described_hook`

```php

// Con attributo:
#[\core\attribute\label('Questa e la descrizione dell hook')]
#[\core\attribute\tags('course')]
final class hook_esempio {
    //...
}

// Con interfaccia:
final class hook_esempio implements \core\hook\described_hook {

    public static function get_hook_description(): string {
        return 'Questa e la descrizione dell hook';
    }

    public static function get_hook_tags(): array {
        return ['course'];
    }

    //...
}

```

I tag sono una lista di stringhe a piacere che poi verranno aggiunte nella maschera di riepilogo. esempi sono `output` o `course`.

L'hook e' responsabile della comunicazione dei dati utili con gli utilizzatori, quindi dovremo inserire relative propieta' e metodi a seconda di come vogliamo che l'utente interagisca con il nostro hook. Le modalita' esatte di come caricare ed utilizzare tali dati e' compito di chi eseguira' l'hook.

E' possibile poi definire l'hoo come "fermabile", ipotizzando che gli utilizzatori del nostro hook possano interrompere il lancio di futuri eventi. In tal caso l'hook deve implementare l'interfaccia `Psr\EventDispatcher\StoppableEventInterface` 

### Eseguire un hook

Una volta definita la classe dell'hook dovremo poi eseguirlo da qualche parte nel nostro codice.

Per eseguire gli hook dobbiamo utilizzare il servizio `\core\hook\manager`, e' sufficiente istanziare il nostro hook e lanciare il metodo `dispatch($hook)`. (come richiamare il servizio vedi capitolo su dependency injection) 

Esempi:

```php
class classe_esempio {

    public function __construct(
        protected readonly \core\hook\manager $hook_manager,
    ) {
    }

    //.....
    function metodo(){
        $hook = new \nome_plugin\hook\hook_esempio();

        $hook_manager->dispatch($hook);
    }
}
```

```php
$hook = new \nome_plugin\hook\hook_esempio();
\core\di::get(\core\hook\manager::class)->dispatch($hook);
```

```php
$hook = new \nome_plugin\hook\hook_esempio();
\core\hook\manager::get_instance()->dispatch($hook);
```

Callbacks
---------

I **Callbacks** ([Documentazione](https://docs.moodle.org/dev/Callbacks)) sono dei metodi particolari che possiamo dichiarare nel nostro plugin e che vengono chiamati da altri processi di moodle core. Sono una feature che sta venendo deprecata e sostituita dai nuovi "Hooks", ma essendo stata presente a lungo mi aspetto che molti plugin li utilizzeranno ancora per un po'.

I callbacks devono aderire a rigide regole di nomenclatura:

1. Devono essere definiti come funzioni nello scope globale di un file chiamato `lib.php` posizionato nella root del plugin
2. Devono avere un nome nel formato **nome_plugin**_**callback** e rispettare la firma del tipo di callback usato

non tutti i plugin possono utilizzare tutti i tipi di callback, ma per iniziare i seguenti sono disponibili a tutti:

* `extend_navigation_frontpage`
* `extend_navigation_course`
* `extend_navigation_user`

ad esempio se volessimo aggiungere un link al nostro plugin definiremmo la seguente funzione in `lib.php`

```php
function local_anagrafe_extend_navigation_frontpage(navigation_node $frontpage) {
    $frontpage->add(
        get_string('pluginname', 'local_anagrafe'),
        new moodle_url('/local/anagrafe/helloworld.php')
    );
}
```

E' raccomandato non utilizzare i callbacks per i nuovi plugin, ed utilizzare invece degli Hooks (vedi sopra). I callback hanno i seguenti problemi:

- La documentazione su quali callback esistono e su quali plugin possono utilizzarli non e' immediata da reperire.
- Non vi e' audit a livello di sito dei callback attivi, inclusi metodi per vedere se il vostro callback e' effettivamente riconosciuto o no da moodle