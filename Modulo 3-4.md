Caches
======

Moodle ha un sistema standardizzato per le caches, progettato per essere relativamente semplice da implementare nei propri plugin, ma allo stesso tempo permettendo diverse opzioni di customizzazione per casi d'uso specifici, o per ottimizare al meglio la performance della cache.

Molti di questi strumenti richiedono una conoscenza avanzata per valutarne l'opportunita', e nel caso e' fondamentale avere un meccanismo di benchmark del codice adatto alla valutazione dei risultati. Per contro l'utilizzo base delle cache e' molto semplice.

Valutare o meno la necessita' di una cache non e' oggetto di questo modulo, vedremo semplicemente come utilizzarle.

Le tipologie di caches disponibili sono veri e propri plugin da configurare. Il meccanismo interno di moodle serve ad astrarre l'utilizzo delle cache nei plugin che le consumano in modo che sia la piattaforma stessa a gestire gli store veri e propri

Utilizzo caches in breve
------------------------

Vediamo quanto serve da sapere per utilizzare le caches nella loro configurazione standard.

La cache va definita in un file `db/caches.php` in un array `$definitions`, dove la chiave e' il nome della cache. 

I parametri delle cache sono molteplici, ma di obbligatorio ve ne e' solo uno:

```php
$definitions = [
    'cache1' => [
        'mode' => cache_store::MODE_APPLICATION,
    ]
];
```

`'mode'` puo' assumere tre valori:

* `MODE_APPLICATION` e' per le cache applicative, che sono condivise tra tutti gli utenti. Di fatto e' il tipo di cache piu' usato.
* `MODE_SESSION` e' una cache specifica per una sessione utente. Da notare che di default queste cache sono impostate per essere servite dal database, quindi sono utili solo se la piattaforma e' stata configurata diversamente
* `MODE_REQUEST` mantenuta in vita solo fino alla fine della attuale richiesta (raramente usata)

il nome per l'utente della cache andra' definito nelle stringhe del linguaggio con una stringa dal nome `cachedef_` + nome cache

```php
$string['cachedef_cache1'] = 'Cache dei dati del plugin XXXXX';
```

Per utilizzare una cache definita all'interno del plugin dobbiamo per prima cosa istanziarla 

```php
$cache = cache::make('enrol_magiclink', 'cache1');
```

Questo metodo ci restituira' un istanza dello store delle cache impostato nella piattaforma in base ai requisiti di `cache1` definiti da noi.

Per utilizzare la cache possiamo salvare coppie chiave-valore con il metodo

```php
$result = $cache->set('key', 'value'); // $result e' true se l'operazione e' andata a buon fine e false altrimenti
```

la chiave puo' essere un intero o una stringa, mentre il valore deve essere [serializzabile](https://www.php.net/manual/en/function.serialize.php) in php. Di fatto e' un meccanismo molto simile a Redis (che di fatto e' uno store cache possibile).

abbiamo poi metodi per richiedere chiavi

```php
$data = $cache->get('key'); // false se la chiave non e' impostata
```

e per eliminare le chiavi

```php
$result = $cache->delete('key'); // $result e' true se l'operazione e' andata a buon fine e false altrimenti
```

esistono poi dei metodi per operazioni multiple:

```php
$result = $cache->set_many([
    'key1' => 'data1',
    'key3' => 'data3'
]);
// $result e' il numero di chiavi salvate.

$result = $cache->get_many(['key1', 'key2', 'key3']);
// $result: array(
//     'key1' => 'data1',
//     'key2' => false,
//     'key3' => 'data3'
// )

$result = $cache->delete_many(['key1', 'key3']);
// $result e' il numero di chiavi eliminate.
```

per un uso base delle cahces non serve sapere altro

[Link a reference rapida](https://docs.moodle.org/dev/Cache_API_-_Quick_reference)

(NOTA: e' possibile usare una cache senza definirla nel file `db/caches.php` con un metodo ad-hoc, ma siccome la definizione sul file e' semplice, evidenzia il fatto che avete una cache nel codice e non definirla non ha particolari vantaggi si tratta di una pratica sconsigliata).

L'implementazione di moodle e' abbastanza basilare, rimane allo sviluppatore il compito di decidere come popolare le cache, alcune modalita':

* Inserire le caches in una classe che, laddove il valore non e' presente, provveda a recuperarlo e metterlo in cache. (esiste una configurazione per dare alle cache nativamente tale comportamento)
* Popolare la cache con un task schedulato che carica valori statici e/o provvede a fare pulizia
* Popolare la cache in risposta ad un evento di creazione di un entita'.

Configurazione avanzata
-----------------------

L'elenco di tutti i parametri possibili e' il seguente:

```php
$definitions = [
    'cache2' => [
        'mode' => cache_store::MODE_*,
        'simplekeys' => false,
        'simpledata' => false,
        'requireidentifiers' => ['ident1', 'ident2'],
        'requiredataguarantee' => false,
        'requiremultipleidentifiers' => false,
        'requirelockingread' => false,
        'requirelockingwrite' => false,
        'requirelockingbeforewrite' => false,
        'maxsize' => null,
        'overrideclass' => null,
        'overrideclassfile' => null,
        'datasource' => null,
        'datasourcefile' => null,
        'staticacceleration' => false,
        'staticaccelerationsize' => null,
        'ttl' => 0,
        'mappingsonly' => false,
        'invalidationevents' => ['event1', 'event2'],
        'canuselocalstore' => false
        'sharingoptions' => cache_definition::SHARING_DEFAULT,
        'defaultsharing' => cache_definition::SHARING_DEFAULT,
    ],
];
```

Si tratta di una lista di settaggi estensiva. Una descrizione per tutti i parametri la trovate sulla [Documentazione ufficiale](https://moodledev.io/docs/apis/subsystems/muc). Elenco alcuni settaggi chiave:

* `simplekeys` permette di usare come chiavi solo numeri lettere o underscore e se attivo salta l'hash delle chiavi.
* `simpledata` se i valori sono solo scalari o array di scalari (bool, int, float o string), se attivo salta la serializzazione.
* `staticacceleration` crea un unica istanza dell'oggetto cache, velocizza notevolmente la cache ma al costo di tenere i dati nella memoria. il parametro successivo `staticaccelerationsize` permette di dare una dimesnione massima per la parte statica in modo da contenere i rischi sulla memoria
* `ttl` e' il tempo in secondi di durata dei dati nella cache. La documentazione ufficiale sconsiglia di utilizzare questo parametro e di implementare una soluzione alternativa basata su eventi. Non tutti i plugin supportano questa opzione e apparentemente ci sono problemi di performance se e' richiesto ad un plugin che ne e' sprovvisto
* `canuselocalstore` indica se si puo' utilizzare lo store locale per la cache, lo store locale non e' condiviso in caso di piu' istanze dietro load balancer quindi la cache va implementata con attenzione a questo aspetto
* `overrideclass` e `overrideclassfile` sono per utilizzare una propria implementazione del metodo factory per la creazione della classe di cache. Inutile dire che e' una feature avanzata, cito la documentazione ufficiale a riguardo : *This is a super advanced feature and should not be done. Ever. Unless you have a very good reason to do so.*
* `datasource` e `datasourcefile` servono per implementare un meccanismo di caricamento dei dati se viene richiesto una chiave mancante. `datasource` e' la classe da usare, che deve ereditare da `cache_data_source`.
* `sharingoptions` e `defaultsharing` indicano se la cache e' da condividere con altri siti, di default e' impostato per nessuna condivisione. Chiaramente lo storage ed il caso d'uso devono essere compatibili.


Configurazione
--------------

La configurazione delle caches viene fatta in Plugins > Caching > Configuration. In questa schermata e' presente tutto cio' che e' necessario per la configurazione delle caches.

La prima tabella presenta la lista dei plugin di store delle cache correntemente installati, con la lista delle loro capacita'. Queste vengono confrontati con i parametri richiesti nel file `db/caches.php` per stabilire quale cache viene fornita ad un plugin dal framework.

La seconda tabella e' la lista delle istanze di cache store configurate al momento sulla piattaforma, ad esempio se abbiamo impostato uno (o piu') server Redis lo troviamo elencato qui, con il numero di plugin correntemente serviti dallo store, e la possibilita di svuotarlo o configurarlo.

La terza, lunghissima, tabella presenta la lista delle cache definite nei plugin, ed e' possibile mappare ciascuna di esse ad uno store specifico. E' inoltre possibile svuotarle singolarmente o decidere il livello di condivisione (con altri siti). Normalmente non e' richiesto ne opportuno indicare manualmente le mappature per tutte le cache, ma potrebbe essere necessario per uno specifico plugin.

La quarta tabella ha le istanze di plugin di lock delle cache, una tipologia di plugin che la stessa documentazione di moodle ammette non essere mai servita a memoria d'uomo.

La quinta tabella e' per indicare lo store di default da utilizzare in assenza di mappature. Se si vmodificare l'impostazione della piattaforma questo generalmente e' il settaggio da impostare.

La configurazione corretta delle cache e' sicuramente un argomento complesso, per motivi trasversali ad altre tecnologie. E' necessario prestare attenzione e pianificare questo aspetto laddove necessario. Alcuni punti chiave:

* per un installazione piccola su una singola macchina modificare le impostazioni di default non e' necessario ne porta grandi benefici. In tali condizione e' invece opportuno ragionare sulla tipologia di hardware, in primis il tipo di hard disk usato (HDD o SSD)
* e' opportuno avere una certa dimestichezza con il provider usato se diverso da quello standard
* e' importante avere metriche di test per verificare l'efficacia delle caches, ad esempio usando JMeter
* Le caches locali vanno abilitate nel file `config.php`, si tratta di cache usate in instalazioni con piu' macchine per quei dati che possono essere salvati in locale (duplicati su ogni macchina). In tali installazioni e' assolutamente sconsigliato usare un drive di rete condiviso per le caches

