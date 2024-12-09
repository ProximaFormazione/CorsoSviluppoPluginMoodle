Eventi
======

Moodle definisce al suo interno un meccanismo di **eventi** al quale i vari plugin possono sottoscrivere dei loro metodi da eseguire.

Dettagli e la lista degli eventi in moodle core e' disponibile sulla [Documentazione ufficiale](https://docs.moodle.org/dev/Events_API) oppure dal menu di amministrazione in Site administration > Reports > Event list

Osservatori
-----------

I metodi sottoscritti agli eventi sono chiamati *osservatori* nella terminologia di moodle.

Per sottoscrivere un metodo ad un evento e' necessario definire il metodo e l'evento all'interno del file `db/events.php`. Qui va definito un array `$observers` i cui membri sono a loro volta array con le informazioni necessarie alla sottoscrizione

esempio:

```php
$observers = array(

    array(
        'eventname'   => '\core\event\course_viewed',
        'callback'    => 'local_anagrafe_observer::course_viewed',
    ),

    array(
        'eventname'   => '\core\event\user_loggedin',
        'callback'    => 'local_anagrafe_observer::user_loggedin',
    ),

    array(
        'eventname'   => '\core\event\course_module_viewed',
        'callback'    => 'local_anagrafe_observer::course_module_viewed',
    ),

);
```

I parametri minimi sono:

* **eventname** ovvero l'evento al quale ci stiamo sottoscrivendo
* **callback** ovvero il metodo da chiamare, e' importante seguire la nomenclatura frankenstyle per il namesapce o la classe per farlo localizzare.

E' poi possibile specificare dei parametri aggiuntivi:

* **includefile** per includere dei files aggiuntivi oltre al metodo chiamato
* **priority** un numero, i metodi con priorita' piu' alta vengono invocati per primi in un evento.
* **internal** di default e' true, se e' false il metodo non viene chiamato prima che la transazione del database sia stata completata.

Nel metodo che consuma l'evento avremo l'evento stesso, che e' una classe, come parametro

```php
public static function course_viewed(\core\event\course_viewed $event) {

        try{
            local_anagrafe_observer::_handle_event($event->get_data()['userid'],$event->get_data()['courseid'],null);
        } catch (Error $ex){
            local_anagrafe_observer::_handle_error($ex);
        }
    }
```

Per i dettagli dei parametri dell'evento si puo' controllare la documentazione, ma il metodo piu' rapido e' ispezionare direttamente l'evento nel codice, dove spesso vi e' documentazione sotto forma di commenti (come in molte altre parti di moodle)

Definire eventi
---------------

Se invece volete definire un nuovo evento, potete farlo definendo una classe per ogni evento nella cartella `classes/event`, gli eventi devono ereditare dalla classe `\core\event\base`

gli eventi devono avere una serie di propieta' standardizzate di moodle, la lista completa e' disponibile nella documentazione ufficiale ([link](https://docs.moodle.org/dev/Events_API))

l'evento va poi lanciato creando un oggetto della classe specifica ed usando il metodo `trigger()`

es:

```php
$event = \mod_myplugin\event\something_happened::create(array('context' => $context, 'objectid' => YYY, 'other' => ZZZ));
$event->trigger();
```

Log
---

Il log di moodle, dalla versione 2.5/2.7, e' stato modificato per essere basato unicamente sugli eventi. 

Di fatto il log e' un plugin che consuma tutti gli eventi e salva le informazioni ivi contenute nel log store.

E' comunque presente un plugin di "log legacy" che salva su file o simili.

[Documentazione completa](https://docs.moodle.org/dev/Logging_2)

Differenze con gli hooks
------------------------

Gli eventi e gli hooks sono concetti simili tra loro, tuttavia presentano alcune differenze:

- L'evento avviene in seguito ad un avvenimento "rilevante" sulla piattaforma, generalmente in seguito ad una qualche azione utente. Gli hooks possono essere lanciati anche banalmente al caricamento della pagina
- L'evento e' immutabile, i suoi dati sono gli stessi per ogni consumatore, i dati dell'hook possono essere modificati
- Gli eventi vengono loggati, gli hook no

Generalmente gli hook sono uno strumento piu' "tecnico", mentre gli eventi sono generalmente spiegabili ad amministratori che non hanno conoscenze di sviluppo

