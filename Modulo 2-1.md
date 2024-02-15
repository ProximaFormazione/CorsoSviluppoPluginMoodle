Pillole di PHP
==============

Questo modulo si propone di dare una velocissima infarinatura di PHP, non e' assolutamente una guida al linguaggio, ma indica alcune delle sue peculiarita'. 

Si tratta di un introduzione mirata a chi non conosce il linguaggio, ma ha esperienza di c++, c# o java; chi ha gia' conoscenze di PHP puo' tranquillamente saltare il capitolo

> Una guida piu' dettaglia potete trovare a questo [link](https://www.w3schools.com/php/default.asp)

Il PHP (Hypertext Preprocessor) e' un linguaggio di scripting, il cui "scopo principale" e' la creazione di testo, generalmente HTML.

Consisite in files con estensione .php che vengono processati dall'inteprete PHP lato server, per produrre del testo che viene ricevuto dal client, quindi l'utente non ha mai diretto accesso al sorgente semplicemente consultando la pagina.

Un file php puo' contenere anche html, che viene processato normalmente, e/o puo' contenere dei blocchi di codice PHP, il codice PHP e' contenuto all'interno di tag che iniziano con `<?php` (o `<?=`) e finiscono con `?>`

```
<?php

// codice qui

?>

```

i blocchi di codice vengono eseguiti nell'ordine in cui vengono trovati nel documento.

```
<p> Il numero casuale e' <?php echo rand(1,6) ?> </p>
```

Per lo sviluppo di moodle, non si usano quasi mai files con un mix di HTML e PHP, ma si usano sempre files con solo codice PHP. In questo caso non e' necessario chiudere il blocco con `?>` alla fine, ed e' anzi sconsigliato farlo per evitare di outputtare per errore del codice. 


Sintassi
========

La sintassi del PHP appartiene alla famiglia delle "Sintassi del C", quindi avete i vostri blocchi di codice contenuti in parentesi graffe con istruzioni che terminano con ;

Gli elementi assimilabili a tale sintassi non verranno indicati in questa guida (ad esempio i blocchi `if(....) {....}`)

I nomi delle variabili sono case sensitive, mentre tutto il resto no.

tutte le variabili vanno precedute dal `$`, altrimenti non vengono riconosciute come tali. le costanti invece non usano il `$`

```
<?php

$nome = "Mario";

echo $nome; 
```

Nel PHP non e' necessario dichiarare il tipo di una variabile, ed anzi questo puo' mutare

```
$variabile = 1;
$variabile = new stdClass();
$variabile = 'stringa';
// nessun errore, adesso $variabile vale 'stringa'
```

dalla versione 7.1 e' stato introdotto il supporto per la tipizzazione forte, ma in molto codice di moodle non la troverete.

Funzioni
--------

le funzioni vengono dichiarate con la parola chiave "function"

```
function scriviMessaggio($x) {
  return $x;
}

echo scriviMessaggio("Hello world!");
```

e' possibile dichiarare il tipo dei parametri, ed il tipo di ritorno mettendolo dopo un `:`
```
function addiziona(int $a, int $b) : int {
    return $a + $b;
}
```

Tuttavia attenzione perche' se il tipo che fornite non corrisponde non sempre l'IDE  grado di accorgersene (lo fa se avete l'estensione giusta) e l'unica cosa che viene forzata e' il ritorno di un errore a runtime. 

Non c'e' un requisito di mettere la definizione delle funzioni prima della loro esecuzione

Funzioni utili
--------------

* `echo` stampa la stringa sul documento, utile per produrre testo senza dovere chiudere il blocco php.
* `var_dump($x)` stampa su schermo in maniera verbosa il contenuto della variabile `$x`, e' una funzione di debug

Stringhe
-------

le stringhe possono essere delimitate sia da `"  "` che da `'  '`. Con le virgolette doppie vengono considerati caratteri speciali mentre con quelle singole no. L'operatore per la concatenazione di stringhe e' il punto `.`.

```
$nome = "Mario";
$cognone = "Rossi";

echo 'Benvenuto '.$nome.' '.$cognome; 
```

Array
-----

Gli array sono uno dei tipi piu' usati nel PHP, e malgrado il nome sono in realta' delle liste indicizzate piu' simili a dizionari c#.

Nel PHP gli array sono di fatto liste di coppie chiave/valore. la chiave puo' essere numerica (array indicizzato) o una stringa (array associativo). Se non viene specificata il PHP assegnera' degli indici numerici

```
$macchine = array("Volvo", "BMW", "Toyota");
$macchine = ["Volvo", "BMW", "Toyota"]; // scritture equivalenti 

// contenuto: array(3) { [0]=> string(5) "Volvo" [1]=> string(3) "BMW" [2]=> string(6) "Toyota" } 
```

```
$macchina = array("marca"=>"Ford", "modello"=>"Mustang", "anno"=>1964);
$macchina = ["marca"=>"Ford", "modello"=>"Mustang", "anno"=>1964]; // scritture equivalenti 

// contenuto: array(3) { ["marca"]=> string(4) "Ford" ["modello"]=> string(7) "Mustang" ["anno"]=> int(1964) } 
```

per aggiungere una coppia chiave/valore ad un array lo si puo' semplicemente assegnare

```
$macchina['ruote'] = 4;
```

se la chiave era assente la coppia verra' aggiunta, se esisteva verra' modificato il valore.

Si puo' aggiungere un valore ad un array senza specificare la chiave:

```
$frutti[] = 'mela';
```

Ed in tal caso la chiave sara' l'indice numerico massimo gia' presente piu' 1. Gli indici partono da 0.

Alcune funzioni utili:

* `unset($lista[1])` rimuove l'elemento con chiave `1` da `$lista`
* `sort()`, `rsort()` per ordinare gli array. Ci sono altri sort piu' specializzati per gli array associativi (es `ksort()` ordina in base alla chiave)
* `count()` per il numero di elementi
* `array_key_exists()` controlla l'esistenza di una chiave
* `array_keys()` restituisce le chiavi dell'array

esiste un `foreach` nel PHP utilizzabile sugli array, la sintassi e' leggermente diversa

```
$annilist = array("Peter"=>"35", "Ben"=>"37", "Joe"=>"43");

foreach ($annilist as $x => $y) {
  echo "$x : $y <br>";
}
```

non e' necessario indicare la variabile su cui si sta iterando come chiave=>valore

```
$colori = array("rosso", "verde", "blu");

foreach ($colori as $x) {
  echo "$x <br>";
}
```

Oggetti
-------

Il PHP supporta le classi e gli oggetti, le dichiarazioni sono analoghe al C#

```
<?php
class Fruit {
  // Properties
  public $name;
  public $color;

  // Methods
  function set_name($name) {
    $this->name = $name;
  }
  function get_name() {
    return $this->name;
  }
}
?>
```

l'ereditarieta' e' su singola classe, e si indica con la parola chiave `extends`

```
class Strawberry extends Fruit {
  
  // ....
}
```

abbiamo le seguenti funzionalita' similarmente a java e c#:

* Modificatori di accesso: `private`, `protected` e `public`
* Classi e Membri statici
* Interfacce e classi astratte
* Namespaces

alcune osservazioni:

* Per accedere ai membri di un oggetto si usa il separatore `->` (il punto e' il concatenatore di stringhe). Per le propieta' statiche invece si usa `::`
* Per accedere ai membri della classe stessa e' necessario usare `$this->...` , altrimenti cerca le variabili solo nello scope locale
* e' possibile eseguire un `foreach` su un oggetto, in tal caso verra' eseguita l'iterazione sulle propieta' dello stesso
* L'argomento e' chiaramente estensivo, ma e' assimilabile ad altri linguaggi noti

StdClass
--------

E' possibile definire un oggetto senza definire prima la classe, utilizzando la cosidddetta `StdClass`, otteniamo cosi' un oggetto dove le propieta' non sono definite.

Provando ad assegnare un valore ad una propieta', qualora questa non sia mai stata definita per quell'oggetto, verra' definita al momento

```
$employee_object = new stdClass; // non ho definito alcuna propieta' per ora
$employee_object->name = "John Doe"; // adesso ho la propieta' "name"
$employee_object->position = "Software Engineer"; 
$employee_object->address = "53, nth street, city"; 
```

Il meccanismo e' simile a quello degli array, ed infatti gli StdClass sono castabili in array e viceversa

```
$employee_array = (array) $employee_object; 
```

Referenziare altri files
------------------------

Se si vuole riutilizzare del codice PHP presente su un altro file, o su una libreria, bisogna inserire manualmente il riferimento al file, usando un istruzione `include` o `require`

```
require('../../config.php');
```

La differenza tra i due e' che `require` produce un errore se il file non viene trovato mentre `include` no

queste istruzioni includono il file nella posizione in cui vengono chiamati, se all'interno di questi sono presenti delle istruzioni, verranno eseguite.

esempio:

```
<html>
<body>

<h1>Welcome to my home page!</h1>
<p>Some text.</p>
<p>Some more text.</p>
<?php include 'footer.php';?>

</body>
</html> 
```

Per evitare operazioni impreviste, si consiglia di utilizare un pattern ad oggetti ovunque possibile, includere files con solo definizioni di classi non eseguira' operazioni non richieste.

Esistono anche le varianti `include_once` e `require_once`, che evitano di inserire un file che e' gia' stato inserito

Superglobali
------------

Nel PHP esistono delle variabili che vengono fornite dall'ambiente

* $GLOBALS
* $_SERVER
* $_REQUEST
* $_POST
* $_GET
* $_FILES
* $_ENV
* $_COOKIE
* $_SESSION

Questi oggetti contengono informazioni relativi ad ambiente, sessione, chiamata http, ecc.
 
Per dettagli sulle variabili rimando ad una guida piu' dettagliata ([link](https://www.w3schools.com/php/php_superglobals.asp))

Costanti magiche
----------------

Esistono anche dei valori di sistema utilizzabili:

* `__CLASS__` nome della classe (se si e' in una classe)	
* `__DIR__` la directory del file
* `__FILE__` il nome del file con il fullpath
* `__FUNCTION__` nome della funzione
* `__LINE__` numero della riga
* `__METHOD__` restituisce nome funzione e nome classe (se applicabili)
* `__NAMESPACE__` restituisce il nome del namespace
* `__TRAIT__` restituisce il nome del trait (che non abbiamo visto)
* `ClassName::class` restituisce nome di classe con il namespace
