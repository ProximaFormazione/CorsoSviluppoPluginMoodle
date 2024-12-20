Sviluppo di primo plugin
========================

Eseguiamo ora gli step necessari per la creazione di un plugin semplice, in modo da vedere la procedura.

> Esistono vari sistemi per creare lo scheletro di un plugin in maniera automatica, come il [Plugin Skeleton Generator](https://docs.moodle.org/403/en/Plugin_skeleton_generator), in questa guida pero' eseguiremo manualmente le operazioni

Come primo progetto creeremo un plugin semplice, ma che sara' un prodotto utilizzabile

Il plugin da creare si chiamera' **anagrafe** e premettera' di eseguire operazioni automatiche in risposta alla prima registrazione di un utente. Nel proseguire del corso aggiungeremo funzionalita' nuove al plugin.

Trattandosi di un plugin che esegue operazioni a livello di sistema sara' un plugin di tipo *local*, quindi il nome completo sara' `local_anagrafe`.

Inizialmente creeremo il plugin privo di funzionalita', che aggiungeremo dopo aver installato il plugin inizialmente. 

Per prima cosa creiamo una cartella di nome `anagrafe` all'interno della cartella `local`, questa cartella sara' l'unica cartella in cui lavoreremo, e la convertiremo in un repository git.

version.php
-----------

All'interno della cartella del plugin, nella root creiamo il file `version.php`.

All'inizio del file apriamo il tag php, ed inseriamo un cartiglio con commenti sul file. A seconda di quanto volete fare circolare il vostro file puo' essere richiesto inserire una licenza GNU o similare

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
```

All'interno di questo commento, che e' completamente opzionale, valutate voi cosa inserire in base alle vostre prassi aziendali. Inserire un autore, o una descrizione del plugin, sicuramente e' utile. Puo' essere utile spendere un minuto a compilarlo correttamente per questo primo file, perche' poi lo copierete su tutti gli altri da qui.

Nello stesso file, inserite poi le seguenti variabili:

```php
defined('MOODLE_INTERNAL') || die();         

$plugin->component = 'local_anagrafe';
$plugin->version = 2024022001;

$plugin->maturity = MATURITY_ALPHA;
$plugin->release   = "0.1";
```

La prima riga, `defined('MOODLE_INTERNAL') || die();` e' uno stratagemma, utilizzato ovunque in moodle e di fatto prassi, per evitare che il file venga servito dal web server quando l'utente naviga su `http://moodle/local/anagrafe/version.php`. 

Di fatto i files php dentro moodle sono divisibili in due categorie: le pagine web pensate per essere mostrate agli utenti, ed i files di "libreria" che non vanno fornite direttamente all'utente. La riga `defined('MOODLE_INTERNAL') || die();` categorizza un file in questa seconda tipologia. Ovunque vedete questa riga quindi non state vedendo una pagina accessibile all'utente. Questa riga va all'inizio per evitare di eseguire operazioni di alcun tipo.

Il significato delle variabili e':

* `$plugin->component = 'local_anagrafe';` e' OBBLIGATORIO ed e' il nome del plugin, seguendo la nomenclatura "Frankenstyle"
* `$plugin->version = 2024022001;` e' OBBLIGATORIO ed e' il numero di versione del plugin, se viene modificato allora viene scatenata la procedura di aggiornamento. Si tratta di una data in formato YYYYMMDDXX dove XX e' un progressivo per permettervi di avere piu' versioni in un giorno
* `$plugin->maturity = MATURITY_ALPHA;` e' utile da aggiungere in modo che venga visualizzato un warning qualora qualcuno provi ad installare il plugin, in modo che sia chiaro che stanno installando un prodotto non finito. Al termine dei lavori sostituirete con la costante `MATURITY_STABLE` (altri valori sono `MATURITY_BETA` e `MATURITY_RC`). Se non inserite un valore non verra' mostrato alcun warning
* `$plugin->release   = "0.1";` e' un numero di versione unicamente visivo per l'utente, da modificare ad ogni rilascio. Se manca verra' mostrato il version sopra

Questi sono i valori minimi da inserire. Vi sono poi tutta una serie di valori aggiuntivi che potete valorizzare, come

* `$plugin->requires = 2018120300.00;` Indica la versione minima di moodle da utilizzare 
* `$plugin->dependencies = ['local_altroplugin' => 2022042100, .... ]` per inserire dipendenze da altri plugin.

I numeri di versione possono essere inseriti anche prevedendo una versione massima, che puo' essere utile per garantire al 100% la funzionalita' indipendentemente da sviluppi futuri, con la controindicazione che dovete mantenere questo settaggio

Cartella stringhe
-----------------

Moodle supporta la traduzione in piu' lingue. Per permettere cio' tutte le stringhe usate devono essere definite separatamente dal punto di codice dove vengono usate, in una o piu' lingue.

La cartella dove vengono conservate le stringe e' la cartella `lang` all'interno della cartella del plugin. All'interno di questa cartella avremo poi una cartella per ogni pacchetto linguistico previsto. `en` per l'inglese ed ad esempio `it` per l'italiano.

Il language pack inglese e' obbligatorio, quindi almeno quello va inserito.

All'interno di ogni cartella della lingua, dovrete mettere un file con il nome "Frankensytle" del plugin dove dovrete associare valori all'array associativo `$string`.

```php
$string['pluginname'] = 'anagrafe';
$string['pluginname_localized'] = 'Anagrafe';
$string['hello_world'] = 'Hello World';
```

> I plugin di tipo mod fanno eccezione e richiedono un nome file con solo il nome del plugin senza "mod_"

La chiave usata sara' poi utilizzata per richiamare la stringa tramite il metodo `get_string`:

`get_string('hello_world', 'local_anagrafe');`

Alcune chiavi hanno ruoli di sistema, come ad esempio 'pluginname_localized' sopra, o descrizioni nei plugin mod (la lista e' disponibile nella documentazione del plugin). Queste chiavi verranno richiamate in vari punti di moodle core 

E' possibile avere dei placeholder all'interno delle stringhe

```php
$string['greeting'] = 'Dear {$a}';
$string['info'] = 'There are {$a->count} new messages from {$a->from}.';
```

e poi passare i parametri al momento di chiamarle

```php
echo get_string('greeting', 'tool_example', 'Mr. Anderson');
echo get_string('info', 'tool_example', ['count' => 42, 'from' => 'Mr. Smith']);
```

La documentazione completa e' disponibile a questo [link](https://docs.moodle.org/dev/String_API).

L'unica stringa obbligatoria e' `pluginname` nel pack `en`. Se volete inserire il plugin nel sistema di localizzazione allora dovrete poi aggiungere le altre stringhe su questo file, e molto probabilmente creare anche il pack `it` dove duplicherete le stringhe traducendole.

Potete ovviamente cablare invece le stringhe direttamente nel codice, che e' indubbiamente piu' veloce. Facendo cosi' non avrete le seguenti funzionalita':

* Supporto in altre lingue, in primis inglese
* Possibilita' per gli utenti di modificare le stringhe

Valutate voi come preferite agire. Chiaramente la funzionalita' multilingua e' interessante, ma non sottovalutate il tempo che dedicherete a compilare queste stringhe. Un buon compromesso potrebbe essere utilizzare unicamente il language pack "en" eventualmente con stringhe in italiano. In questo modo potete facilmente in futuro inserire il language pack inglese e conservare il lavoro fatto su quello italiano spostando il file

Repository Git
--------------

Una volta aggiunti i due files, non dimentichiamoci di creare un repository git con la cartella del plugin e committare i files.

Per l'ambiente di sviluppo, non e' fondamentale utilizzare un sottomodulo, potete anche semplicemente inserire nel .gitignore del repository padre la cartella con il plugin. (Ovviamente potete anche usare un sottomodulo).

L'uso di git e' il metodo raccomandato da Moodle per la gestione degli aggiornamenti.

Finalizzazione installazione
============================

Per completare l'installazione, e' sufficiente loggarsi all'interno della piattaforma dove si trova il plugin con un account amministratore, questo lancera' la solita procedura di aggiornamento del database che finalizza l'installazione del plugin.

Una volta terminata potete verificare la presenza del plugin nel menu' di amministrazione alla voce Plugins -> panoramica plugins.

Al momento il plugin non ha nessuna funzionalita', nel prossimo capitolo provvederemo a fornire il plugin di un interfaccia grafica.

