Prima interfaccia grafica in moodle
===================================

In questo modulo realizzeremo una semplice interfaccia grafica per prendere dimestichezza con le API di moodle relative.

Continueremo ad espandere il plugin `local_anagrafe` fornendogli una pagina con contenuto banale. I plugin locali non devono avere tale interfaccia necessariamente, ma spesso ce l'hanno.

Variabili Globali in Moodle
===========================

In moodle sono definite alcune [variabili globali](https://www.w3schools.com/php/php_superglobals_globals.asp) utilizzabili all'interno dei plugin. Le linee guida raccomandano di non definire altre variabili globali (e non ve n'e' bisogno).

Le variabili sono:

* `$CFG` Contiene i parametri di configurazione, include sia gli elementi presenti nel file config.php che quelli definiti nell'amministrazione del sito
* `$SESSION` Contiene informazioni sulla sessione, tra cui token del login ed il parametro `wantsurl` per il reindirizzamento tra plugins
* `$USER` Il record dell'utente attuale, se non si e' loggati qui avete il guest
* `$PAGE` snodo per le informazioni sulla pagina attualmente visualizzata, possiamo e dobbiamo valorizzare alcune propieta' quando scriviamo una pagina
* `$OUTPUT` classe per la generazione degli elementi della pagina
* `$DB` classe per accedere al database (la vedremo piu' avanti)
* `$COURSE` dettagli del corso visualizzato
* `$SITE` dettagli della front page, che e' il corso con id = 1

(per utilizzare una variabile globale all'interno di una funzione va dichiarata con la parola chiave `global`)

```
public static function get_data(){

        global $DB;

        // ......
}
```

per l'interfaccia grafica utilizzeremo `$PAGE` per dichiarare alcuni elementi come il titolo, e `$OUTPUT` per renderizzare il contenuto

Creazione della pagina
======================

Iniziamo creando il file con la pagina da mostrare, creiamo nella root il file `helloworld.php`. Nel resto della guida lavoreremo su quello.

> Il nome del file fara' parte dell'url della pagina (per noi www.sito.it/local/anagrafe/helloworld.php), scegliete quindi nomi consoni

All'interno del file apriamo il tag `<?php`, ed eventualmente inseriamo il commento generale al file (copiandolo da version.php)

All'interno del file inseriamo subito eventuali dipendenze, iniziando dal file di configurazione

```php
<?php

// .....

/**
 * @package    local_anagrafe
 * @author     Mario Rossi <mario.rossi@mailinesistente.it>
 * @copyright  2024 Azienda S.r.l. (https://www.sitoazienda.it/)
 * @license    http://www.gnu.org/copyleft/gpl.html GNU GPL v3 or later
 */

require_once('../../config.php');

```

(nei prossimi esempi ometteremo i commenti iniziali)

Il file di configurazione si importa a sua volta altri files, che contengono tutte le API e le variabili globali che useremo.

In files che gia' hanno importato tale file, potremo usare `$CFG->dirroot` per formare i percorsi degli altri files, ma qui siccome ancora non abbiamo nulla dobbiamo usare il path relativo

> Di fatto la presenza di una riga per importare il file config come `require_once('../../config.php');` caratterizza un file come un entry point (pagina) a differenza dei files che iniziano con `defined('MOODLE_INTERNAL') || die();` che invece caratterizzano il file come un file interno da includere in altri files

La prossima riga da aggiungere e' `require_login()`, che ci assicura che l'utente sia loggato. Se l'utente non e' loggato gestisce il redirect sulla pagina di login, al termine del quale l'utente verra' reindirizzato nuovamente sulla nostra pagina.

> ATTENZIONE: laddove e' abilitato l'account guest `require_login()` di default esegue da solo la login come guest, ci sono parametri opzionali al metodo per modificare il comportamento

Ovviamente se la pagina deve essere accessibile senza autenticazione (pagina statica) potete omettere tale riga.

Definizione caratteristiche pagina
==================================

All'interno di una pagina di moodle vi sono alcuni elementi da definire nella variabile globale `$PAGE`.

Per prima cosa definiamo titolo ed header, che andranno a definire il tag `<title>` ed il titolo del contenuto della pagina.

per farlo usiamo i metodi `set_title(string)` e `set_heading(string)` della variabile `$PAGE`

```php
<?php

// .....

/**
 * @package    local_anagrafe
 * @author     Mario Rossi <mario.rossi@mailinesistente.it>
 * @copyright  2024 Azienda S.r.l. (https://www.sitoazienda.it/)
 * @license    http://www.gnu.org/copyleft/gpl.html GNU GPL v3 or later
 */

require_once('../../config.php');
require_login();

$PAGE->set_title(get_string('pluginname', 'local_anagrafe'));
$PAGE->set_heading(get_string('pluginname', 'local_anagrafe'));

```

Per la stringa da inserire, abbiamo utilizzato il metodo `get_string(string, string)` che serve a recuperare la stringa con la localizzazione corretta. Qui abbiamo usato la stringa con chiave 'pluginname', ma potevamo inserirne un'altra (e generalmente va fatto)

Inseriamo poi l'url della pagina con il metodo `set_url`

```php
$PAGE->set_url(new moodle_url('/local/anagrafe/helloworld.php'));
```

la classe `moodle_url` e' un helper per costruire url con parametri, che include automaticamente la root quando poi la stringa viene usata.

definiamo poi il contesto utilizzato dalla pagina. Per ora soprassediamo su cosa sono i contesti. Qui utilizzeremo il contesto piu' alto, ovvero quello di sistema

```php
$PAGE->set_context(context_system::instance());
```

infine decidiamo il layout da utilizzare per la pagina.

Il layout e' la modalita' del tema per la presentazione del contenuto della pagina, decide come appare il titolo, i menu' di navigazione o i blocchi.

I layout disponibili dipendono dal tema, ma dovrebbero essere presenti almeno `base` (che e' usato di default), `standard`, `course`, `frontpage`, `mydashboard` e `login`

per la nostra pagina usiamo il layout standard

```php
$PAGE->set_pagelayout('standard');
```

Disegnare la pagina
===================

Una volta terminata la configurazione del DOM possiamo iniziare a stampare l'html della pagina.

Per fare cio' utilizzeremo `echo` ed i metodi della classe `$OUTPUT`.

E' importante che tutte le operazioni di backend siano gia' state eseguite prima di iniziare ad utilizzare la classe `$OUTPUT`, in modo da non presentare pagine  parziali o malformate in caso di errore.

usando il comando 

```php
echo $OUTPUT->header();
```

viene prodotto tutto il necessario html in base ai parametri della pagina, fino all'apertura del tag del `<body>`.

Usando il comando

```php
echo $OUTPUT->footer();
```

vengono chiusi i tag e finalizzata la pagina.

Tutto il contenuto della pagina deve quindi essere definito tra queste due istruzioni. 

Per questa semplicissima pagina stamperemo manualmente l'html usando la classe `html_writer` che ha una serie di metodi per aiutare la creazione di html secco

```php
echo html_writer::tag('p', get_string('helloworld', 'local_anagrafe'));
```

Esistono metodi piu' sofisticati che separano la logica di rappresentazione da quella business, ma per pagine semplicissime questo approccio va benissimo.

Da notare che abbiamo usato una stringa nuova, che quindi andra' definita all'interno del file `lang/en/local_anagrafe.php`

```php
$string['helloworld'] = 'Hello world!';
```

ed eventualmente in italiano in `lang/it/local_anagrafe.php`

```php
$string['helloworld'] = 'Ciao Mondo!';
```

Conclusione
===========

Ora possiamo controllare l'effettiva presenza della pagina, prima pero' aggiorniamo il numero di versione del plugin nel file `version.php` in modo da segnalare che questa e' una nuova versione.

Lo abbiamo fatto unicamente a scopo didattico non e' necessario per il funzionamento. Con le modifiche che abbiamo fatto sarebbe bastato svuotare le cache per popolare la nuova stringa che abbiamo aggiunto.

La pagina e' accessibile all'url `localhost/nomesito/local/anagrafe/helloworld.php`

Se avete fatto tutto correttamente, dovrebbe essere visualizzata con lo stesso stile delle altre pagine e dovrebbe essere presente in basso a destra un punto interrogativo con il footer.

la struttura finale della pagina e' la seguente:

```php
<?php

// Copyright (C) 2024 Azienda S.r.l. (https://www.sitoazienda.it/)
//
// This file is part of the Magic Link plugin for Moodle - http://moodle.org/
//
// This program is free software; you can redistribute it and/or modify
// it under the terms of the GNU General Public License as published by
// the Free Software Foundation; either version 3 of the License, or
// (at your option) any later version.
//
// This program is distributed in the hope that it will be useful,
// but WITHOUT ANY WARRANTY; without even the implied warranty of
// MERCHANTABILITY or FITNESS FOR A PARTICULAR PURPOSE.  See the
// GNU General Public License for more details: http://www.gnu.org/copyleft/gpl.html
/**
 * @package    local_anagrafe
 * @author     Mario Rossi <mario.rossi@mailinesistente.it>
 * @copyright  2024 Azienda S.r.l. (https://www.sitoazienda.it/)
 * @license    http://www.gnu.org/copyleft/gpl.html GNU GPL v3 or later
 */

require_once('../../config.php');
require_login();

$PAGE->set_title(get_string('pluginname', 'local_anagrafe'));
$PAGE->set_heading(get_string('pluginname', 'local_anagrafe'));
$PAGE->set_url(new moodle_url('/local/anagrafe/helloworld.php'));
$PAGE->set_context(context_system::instance());
$PAGE->set_pagelayout('standard');

//----------------------------------------------

echo $OUTPUT->header();

echo html_writer::tag('p', get_string('helloworld', 'local_anagrafe'));

echo $OUTPUT->footer();
```

