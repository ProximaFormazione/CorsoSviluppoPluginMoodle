Classi e Dipendenze
===================

Se dovete sviluppare un plugin che vada al di la' di una singola pagina con funzionalita' semplici, dovrete sicuramente produrre diversi files di codice.

Nel PHP, potete integrare i files utilizzando varie istruzioni `include(xxxxx)` per inserire le vostre dipendenze. Tuttavia questo approccio puo' portare a diversi problemi, sia di organizzazione vostra, che di esecuzione del programma. Vediamo qui alcuni suggerimenti e strumenti offerti da moodle.

Per prima cosa si raccomanda di utilizzare ovunque sia possibile un architettura ad oggetti. Il PHP supporta sia la programmazione procedurale che quella orientata agli oggetti e si raccomanda di inserire la maggioranza del codice all'interno di classi. Questo per evitare ad esempio che il vostro codice venga eseguito semplicemente perche', magari per errore, avete incluso il file nel posto sbagliato.

Questo e' un buon inizio e vi permette di avere sicurezza nell'isolare i vostri moduli assicurandovi che vengano lanciati solo quando richiesto.

Moodle mette poi a disposizione una serie di strumenti che vi permettono di lavorare senza dover includere manualmente le vostre dipendenze nel codice, a patto che seguiate certe linee guida.

Elemento centrale e' il motore di auto caricamento delle classi ([Link](https://docs.moodle.org/dev/Automatic_class_loading)), che e' un meccanismo che crea un indice delle classi create in modo da recuperare il file solo se e quando la classe viene utilizzata. Affinche' questo funzioni bisogna rispettare alcune convenzioni:

- La classe deve essere opsitata in un **file con lo stesso nome** della classe 
- La classe deve avere come namespace il nome frankenstyle del plugin, piu' eventuali altri sottonamespace a piacere
- Il file della classe deve risiedere nella cartella **classes** del vostro plugin. Se avete usato dei sottonamespace il file deve essere in una sottocartella con lo stesso percorso.

Ad esempio questa classe:

```php
namespace local_anagrafe\biz;

class helloworld {

    public static function helloworld(){
        return get_string('helloworld','local_dispense');
    }
}

```

deve risiedere in un file chiamato `helloworld.php` che deve trovarsi nella cartella `local/anagrafe/classes/biz`.

Questo sistema e' particolarmente esigente nei suoi requisiti, nell'esempio di prima una qualsiasi delle seguenti fa si che la classe **NON** venga caricata e che il codice andra' semplicemente in errore:

- Se il nome del file ee `lib_stringhe.php`
- Se il namespace e' `local_anagrafe` e basta
- Se il file e' nella root di `classes` o nella sottocartella sbagliata

Altrimenti se la classe e' stata inserita correttamente, potremmo richiamarla direttamente usando il suo namespace completo senza bisogno di includere manualmente il file dove si trova. Nelle versioni dalla 4.4 questo si fa utilizzando la **Dependency injection** (vedi sotto), mentre nelle versioni precedenti la pratica era istanziare direttamente le classi.

In versioni precedenti di moodle, invece che usare il namespace la pratica era prefiggere il nome della classe con il nome frankenstyle del plugin (es: `local_anagrafe_helloworld`), ma cio' e' stato deprecato.

Dependency Injection
--------------------

[Link a documentazione ufficiale](https://moodledev.io/docs/4.5/apis/core/di)

La serie 4 di moodle ha effettuato un cambio di requisiti imponendo l'uso del PHP 8, il quale porta con se una serie di novita' che lentamente sono state incorporate nell'architettura di moodle, in particolare l'uso di PHP-DI([Link](https://php-di.org/)) come framework per la dependency injection (DI).

Il container per la dependency injection e' stato introdotto nella versione 4.4 di moodle, in tali versioni e' possibile passare una dipendenza direttamente nel costruttore di una classe. Questo include le classi di moodle core.

Esempio:

```php
class classe_esempio {

    public function __construct(
        protected readonly \local_anagrafe\biz\helloworld $altra_classe,
    ) {
    }

    //.....
}
```

Questa sintassi in particolare utilizza l'autopromozione dei parametri a propieta' ([Link](https://stitcher.io/blog/constructor-promotion-in-php-8)) che e' una feature del PHP 8 altamente raccomandata nell'utilizzo della DI.

Laddove non e' possibile utilizzare la DI, ad esempio laddove non abbiamo una classe (tipo nel codice di una pagina) e' comunque disponibile un wrapper del container (di fatto un [Service locator](https://en.wikipedia.org/wiki/Service_locator_pattern)) nella classe `\core\di`

```php
$client = \core\di::get(\core\http_client::class);
$helloworld = \core\di::get(\local_anagrafe\biz\helloworld ::class);
```

(\core\di viene usata tramite il processo di autocaricamento di sopra)

L'implementazione del container DI fatta da moodle include automaticamente le classi compliant con le regole del paragrafo precedente, non e' quindi richiesto definire i contenuti in un file centrale e si puo' usare la DI semplicemente in sostituzione al processo di sopra.

Ovviamente e' possibile definire come i servizi all'interno del container, per fare cio' possiamo inserirci nell'hook `\core\hook\di_configuration`, ad esempio:

```php

public static function inject_dependencies(di_configuration $hook): void {
    $hook->add_definition(
        id: complex_client::class,
        definition: function (
            \moodle_database $db,
        ): complex_client {

            return new complex_client($parametri);
        }
    )
}

```

oppure in modalita' just in time registrando un nuovo servizio nel container con il metodo `\core\di::set()`

```php

\core\di::set(
    complex_client::class,
    new complex_client($parametri),
);

```

Il secondo approccio e' sconsigliato per il codice effettivo, in quanto la modifica al container DI e' valida anche per il resto dell'applicazione. E' invece molto pratico per gestire gli unit test.