Override di un Tema
===================

I plugin di tipo tema sono tra i piu' complessi e laboriosi da creare, tuttavia abbiamo a volte problemi che possono essere risolti unicamente con una modifica ai template. Fortunatamente esiste la possibilita' di applicare modifiche minime eseguendo un **override** di alcuni elementi di un tema esistente

Prima di procedere con questa guida, e' buona cosa familiarizzarsi con questa opzione che possiamo mettere nel file `config.php` del sito:

```php
$CFG->theme = 'boost';
```

Questo obbliga moodle ad utilizzare il tema indicato, ad esempio nel caso in cui il tema corrente non funzioni e non permetta di accedere ad alcuna pagina

Tema di override minimale
-------------------------

Senza entrare nei dettagli dei plugin di topo tema, possiamo creare un override di un tema esistente, come ad esempio boost, in modo da ereditare tutti i renderer ed i template.

L'argomento e' complesso e lo affronteremo in maniera semplificata, ma una guid della documentazione ufficiale e' disponibile a questo [LINK](https://docs.moodle.org/dev/Themes)

Per prima cosa creiamo un nuovo tema, creando il file `version.php` e la stringa del language pack. Nella guida il tema verra chiamato `theme_nuovotema`, ma chiaramente potete scegliere voi il nome.

`theme/nuovotema/version.php`:

```php
<?php
defined('MOODLE_INTERNAL') || die();

$plugin->component    = 'theme_nuovotema';
$plugin->release      = '1.0';
$plugin->version      = 2024120601;
```

`theme/nuovotema/lang/en/theme_nuovotema.php`:

```php
<?php

defined('MOODLE_INTERNAL') || die();

$string['pluginname'] = 'nuovo login';
```

Poi aggiungiamo, nella root del plugin, un file `config.php` con le seguenti istruzioni:

`theme/nuovotema/version.php`:

```php
<?php

$THEME->name = 'nuovotema';

$THEME->parents = ['boost'];

$THEME->rendererfactory = theme_overridden_renderer_factory::class;

$THEME->yuicssmodules = [];  
```

La prima riga e' semplicemente il nome del plugin.

La seconda riga indica che il nostro plugin eredita dal tema **boost**, quindi laddove non e' presente un render verra' utilizzato quello del tema boost. Qui e' possibile indicare piu' temi, ed in tal caso gli ultimi inseriti hanno precedenza su quelli precedenti.

La terza riga indica di utilizzare una classe particolare per recuperare i renderer, che ricerca prima nel nostro plugin, e poi nei plugin da cui ereditiamo per poi finire con i renderer di moodle core. (Da notare che tutti i temi preinstallati usano la stessa classe).

La quarta riga e' specifica per i temi che ereditano da boost, e serve per la rappresentazione corretta di alcuni elementi grafici

A questo punto abbiamo un nostro tema, che possiamo selezionare per la nostra piattaforma, che e' identico a boost. A questo punti possiamo aggiungere elementi che vogliamo alterare.

Un sistema e' utilizzare l'auto discover dei template di moodle per caricare nostri template che modifichino elementi core. Ci basta quindi copiare i files dei template da modificare mettendoli nel percorso corretto nella cartella `templates`

Ad esempio, se volessimo creare la nostra maschera di login personalizzata, potremmo copiare il template del form di login da `lib/templates/loginform.mustache` e metterlo in `theme/nuovologin/templates/core/loginform.mustache`. A questo punto possiamo eseguire modifiche sul nuovo template poiche' sara' quello effettivamente selezionato a runtime.

Quando eseguiamo modifiche sul tema, e' buona pratica attivare l'opzione "Theme designer mode" in Amministrazione dle sito -> aspetto -> opzioni avanzate tema. Oltre che disattivare la cache dei template in Aspetto -> Template. Questo fa si che le vostre modifiche vengano recepite alla modifica della pagina senza dover svuotare le caches.

### Settaggi

Un problema dell'approccio precedente e' che il nuovo tema, malgrado abbia l'aspetto di boost, non e' configurabile. Questo perche' i settaggi sono legati a `theme_boost`, mentre noi cerchiamo settaggi per `theme_nuovotema`.

Una possibile soluzione e' clonare gli interi settaggi di boost, assegnandoci nuovi identificatori.

Un modo semplice consiste nel copiare il file `settings.php` da `theme/boost` in `theme/nuovotema`, e poi eseguire un trova e sostituisci da `theme_boost` a `theme_nuovotema`.

> A questo punto conviene eseguire la stessa operazione per il language pack

Purtroppo questo passaggio puo' richiedere la copia di files addizionali a seconda di come il tema abbia implementato i settaggi. Ad esempio per boost dovremo copiare anche i files `theme/boost/classes/admin_settingspage_tabs.php` e `theme/boost/templates/admin_setting_tabs.mustache`. Questo lo scoprite solo osservando il sorgente e/o procedendo per tentativi. (Si raccomanda di abilitare i messaggi di debug).

Specificatamente per boost, la documentazione fornisce un esempio di file modificato, che permette di evitare di dover copiare i files di sopra

Una volta copiati i settaggi correttamente, dovreste poter impostarli separatamente da quelli del tema boost. Questo stratagemma puo' essere utile anche per avere diverse configurazioni di boost da utilizzare ad esempio in base alla categoria.

### CSS

Per il tema boost, possiamo appoggiarci ai file SCSS di boost stesso, per farlo dobbiamo aggiungere questa riga nel file `config.php`:

```php
$THEME->scss = function($theme) {
    return theme_nuovotema_get_main_scss_content($theme);
};
```

e poi aggiungiamo la funzione in un file `lib.php`, sembre nella root del tema:

```php
function theme_nuovotema_get_main_scss_content($theme) {
    global $CFG;                                                                                                                    
                                                                                                                                    
    $scss = '';                                                                                                                     
    $filename = !empty($theme->settings->preset) ? $theme->settings->preset : null;                                                 
    $fs = get_file_storage();                                                                                                       
                                                                                                                                    
    $context = context_system::instance();                                                                                          
    if ($filename == 'default.scss') {                                                                                              
        $scss .= file_get_contents($CFG->dirroot . '/theme/boost/scss/preset/default.scss');                                        
    } else if ($filename == 'plain.scss') {                                                                                         
        $scss .= file_get_contents($CFG->dirroot . '/theme/boost/scss/preset/plain.scss');                                          
    } else if ($filename && ($presetfile = $fs->get_file($context->id, 'theme_nuovotema', 'preset', 0, '/', $filename))) {              
        $scss .= $presetfile->get_content();                                                                                        
    } else {                                                                                                                        
        $scss .= file_get_contents($CFG->dirroot . '/theme/boost/scss/preset/default.scss');                                        
    }                                                                                                                                       
                                                                                                                                    
    return $scss;       
}
```

La funzione e' basata sull'omologa di boost. Se vogliamo utilizzare le funzioni di aggiunta in fase pre e post del CSS dovremo implementare le relative funzioni (prendere a modello boost).

### Altre controindicazioni

E' possibile che ci siano altri requisiti da soddisfare, a seconda del tema usato.

### Alternative

Per quanto minimale, questa procedura potrebbe non essere applicabile al vostro caso d'uso. Potete procedere anche in altri modi:

- Potete modificare direttamente i template originali, che si trovino in un tema o in moodle core. Nel caso del tema conviene alterare il numero di versione rendendolo assurdamente alto (es modificando il primo 2 in un 3) in modo da impedire aggiornamenti che cancellerebbero le vostre modifiche. Alterare un template in moodle core richiede di mantenere la modifica negli aggiornamenti, ma dovrebbe essere fattibile.
- Potete generare una pagina nuova che svolga le funzioni richieste, meno API e prassi di Moodle implementate, piu' difficolta' avrete poi nel farla interagire con la piattaforma.
- Molti temi aggiuntivi fanno un largo uso di aree dove e' possibile inserire blocchi, molte personalizzazioni possono essere ottenute con i blocchi di testo (che sono blocchi con html generico)