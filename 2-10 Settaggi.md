I settaggi di un plugin consistono in una serie di configurazioni che vengono salvate in una coppia chiave/valore nel database.

Moodle ha una serie di funzioni che permettono il salvataggio e caricamento dei settaggi, oltre ad un robusto meccanismo per la creazione di menu appositi.

Unica cosa da fare e' definire i settaggi in un apposito file denominato `settings.php` che va posizionato nella root della cartella. Si tratta di un file che viene incluso da moodle quando deve mostrare i settaggi.

I settaggi hanno una struttura simile all'albero della navigazione, e vanno ad incidere sulla variabile `$ADMIN`, alla quale aggiungeremo potenzialmente una pagina standard coi settaggi, con un oggetto di classe `admin_settingpage`.

Dentro questa classe aggiungeremo i vari settaggi usando il tipo di controllo che vogliamo fare usare all'utente per impostare il messaggio

Sembra complesso, ma una volta impostato una volta la struttura e' identica e si puo' prendere a modello per progetti successivi.

Ad esempio inseriamo un settaggio per decidere se fare visualizzare il nostro plugin nella navigazione o no.

L'esatta sintassi da utilizzare varia a seconda della tipologia  plugin, per un plugin di tipo local ad esempio dobbiamo modificare l'albero dei settaggi ed aggiungere il nodo con la pagina dei settaggi. 

Alcune tipologie di plugin, come ad esempio i plugin di iscrizione o autenticazione, ottengono automaticamente una pagina da moodle (che mostra o nasconde se il plugin e' attivo o no), in questi casi nel file `settings.php` avremo gia' valorizzata la variabile `$settings` con la pagina dei settaggi, e dovremo quindi aggiungere solo i singoli settaggi

I Settaggi sono particolari istanze di `admin_setting`, o meglio di una classe figlia (e' astratta), le varie classi che implementano `admin_setting` sono vari possibili tipi di settaggi caraterizzati in base al controllo che l'utente usera' per modificarli (es campo di testo, select, ecc).

Esempio per plugin che deve definire la propria pagina:

```php
defined('MOODLE_INTERNAL') || die();

if ($hassiteconfig) {
    $ADMIN->add('localplugins', new admin_category('local_helloworld_settings', new lang_string('pluginname', 'local_helloworld')));
    $settingspage = new admin_settingpage('managelocalhelloworld', new lang_string('manage', 'local_helloworld'));

    if ($ADMIN->fulltree) {
        $settingspage->add(new admin_setting_configcheckbox(
            'local_helloworld/showinnavigation',
            new lang_string('showinnavigation', 'local_helloworld'),
            new lang_string('showinnavigation_desc', 'local_helloworld'),
            1
        ));
    }

    $ADMIN->add('localplugins', $settingspage);
}
```

* `local_helloworld/showinnavigation` e' il nome del settaggio da usare per richiamarlo
* `$hassiteconfig` e' true se l'utente ha capacita' amministrative sul sito
* `$ADMIN->fulltree` serve a verificare se stiamo visualizzando l'intero albero di navigazione. In modo da evitare di processare codice che non serve
* Tutte le stringhe vanno definite nei file della lingua

mentre per un plugin di tipo enrol conviene invece solo aggiungere elementi alla variabile `$settings`

```php
 defined('MOODLE_INTERNAL') || die();

 if ($ADMIN->fulltree) {
    $settings->add(new admin_setting_configcheckbox(
        'enrol_esempio/showinnavigation',
        get_string('showinnavigation', 'enrol_esempio'),
        get_string('showinnavigation_desc', 'enrol_esempio'),
        1
    ));
}
```

E' anche possibile aggiungere all'albero di navigazione un oggetto di tipo `admin_externalpages` che reindirizza su una pagina. Potete usare questa opzione per avere un link ad una vostra pagina di settaggi che magari richiede il salvataggio di dati in una vostra tabella. Per uniformita' visiva potete utilizzare nella pagina creata il metodo `admin_externalpage_setup($pagename)`. Si raccomanda laddove possibile di utilizzare settaggi normali piuttosto che creare pagine ad hoc, perche' sono meglio integrati (ricerca e cache).

Per consultare un settaggio sara' sufficiente utilizzare il metodo `get_config('nome_plugin','settaggio')`

Ad esempio possiamo inserire un check nei metodi che aggiungono la navigazione nel nostro caso:

```php
public static function estendi_navigazione(\core\hook\navigation\primary_extend $hook): void {
    if(get_config('local_helloworld','showinnavigation') == '1'){
        $primaryview = $hook->get_primaryview();
        $primaryview->add('Sono un link',new \moodle_url('/local/helloworld/view.php', []));
    }    
}
```

I settaggi sono salvati come stringhe.

[Documentazione Ufficiale](https://moodledev.io/docs/apis/subsystems/admin)

Settaggi utente
---------------

I settaggi specifici di un utente sono salvati nel database ed accessibili in maniera similare ai settaggi del sito.

Sono a disposizione i metodi `get_user_preferences()`, `set_user_preference()` e `unset_user_preference()`

vedi il codice o la documentazione per altri dettagli ([link](https://moodledev.io/docs/apis/core/preference))