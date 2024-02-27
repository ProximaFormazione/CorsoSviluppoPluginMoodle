

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