Date
----

Le date nel database di moodle sono salvati in [timestamp unix](https://it.wikipedia.org/wiki/Tempo_(Unix)), ovvero con un intero che indica il numero di secondi passati dall' 1 gennaio 1970.

Esistono vari metodi di aiuto per gestire i timestamp in moodle ([Documentazione completa](https://moodle.academy/mod/lesson/view.php?id=817&pageid=162)).

Interessatne la funzione `userdate` e la funzione `usertime` che tengono presente il fuso orario indicato dall'utente.

php ha una funzione `time()` per ottenere il timestamp attuale.

URL
---

Le url della pagina, e per il rendirizzamento del sito, devono includere la root del web server. Per semplificarsi la vita si puo' utilizzare la classe `moodle_url` che gestisce un sacco di cose per noi

```php
$PAGE->set_url(new moodle_url('/enrol/magiclink/helloworld.php'));
```

la classe offre una serie di vantaggi: e' convertibile in stringa implicitamente, e permette l'aggiunta dei parametri GET tramite array:

```php
$courseviewurl = new moodle_url('/course/view.php', ['id' => $courseid]);
```