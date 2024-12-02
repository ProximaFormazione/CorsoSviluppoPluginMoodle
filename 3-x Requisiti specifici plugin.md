Requisiti in base alla tipologia plugin 
=======================================

Ogni tipo di plugin puo' presentare una serie di requisiti addizionali nella struttura di files e cartelle.  

Per sicurezza e' sempre meglio consultare la documentazione ufficiale a riguardo ([link](https://moodledev.io/docs/apis/plugintypes))

Ad esempio per i plugin di tipo enrol, dobbiamo inserire una classe all'interno di `lib.php` che erediti da `enrol_plugin` che dovra' definire le caratteristiche del nostro plugin.

```php
class enrol_pluginname_plugin extends enrol_plugin {

    //...
}
```

Eseguendo l'override dei metodi della classe base andremo a definire diverse caratteristiche del nostro plugin.

Il nostro plugin di iscrizione agira' attraverso un link apposito, quindi non interagiremo molto con il processo usuale, aggiungeremo giusto la possibilita' di usare l'interfaccia esistente per gestire il plugin

```php
class enrol_pluginname_plugin extends enrol_plugin {

    public function use_standard_editing_ui() {
        return true;
    }

    public function edit_instance_form($instance, MoodleQuickForm $mform, $context) {
        // Do nothing by default.
    }


    public function edit_instance_validation($data, $files, $instance, $context) {
        // Niente validazione per ora
        return array();
    }

    public function can_add_instance($courseid) {
        return true;
    }
}
```

Se eseguiamo l'update della versione del plugin, dovremo ora vedere il nostro plugin nella lista presente nel menu' di amministrazione Plugin -> Enrollments -> manage enrollments.

Di base la modalita' e' disattivata, ma noi possiamo attivarla premendo sull'occhiello nella tabella.

> ATTENZIONE: Questo abilita solo il plugin a livello di sito, bisogna poi abilitare il plugin sui corsi dove si intende usarlo. E' possibile automatizzare il processo in modo che un nuovo corso abbia questa abilitazione di default, tuttavia nel nostro caso il requisito e' diverso.

Ritorneremo su questa classe quando aggiungeremo altre funzionalita', ad esempio per stabilire i permessi per disiscrivere gli utenti.
