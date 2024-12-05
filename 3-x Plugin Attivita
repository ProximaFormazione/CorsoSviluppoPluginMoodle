Plugin attivita
===============

[Documentazione ufficiale](https://moodledev.io/docs/apis/plugintypes/mod)

I plugin di attivita', o "modules" (`mod`) sono la prima tipologia di plugin storicamente introdotta su moodle, e di fatto una delle piu' espanse.

Per ragioni storiche, alcune regole e prassi di moodle sono leggermente diverse per i plugin mod, in particolare nella nomenclatura "Frankenstyle" il `mod_` va omesso nei nome delle tabelle del database.

Elementi obbligatori
--------------------

i plugin di tipo attivita' hanno esigenze diverse particolari rispetto ai plugin base, molte delle quali da implementare con prassi che moodle sta cercando di deprecare altrove.

Questi requisiti sono in aggiunta ai requisiti dei plugin normali (file version.php e language pack)

### Tabella DB

I plugin mod richiedono la presenza di una tabella dal nome identico al nome plugin, con i seguenti campi almeno:

* **id** *INT(10) auto sequence* 
* **course** *INT(10) foreign key course(id)*
* **name** *CHAR(255)* il nome indicato dall'utente per l'istanza dell'attivita' 
* **timemodified** *INT(10)* data ultima modifica
* **intro** *TEXT* la descrizione dell'istanza dell'attivita'
* **introformat** *INT(4)* formato del testo di sopra

### Form

Deve essere incluso un file `mod_form.php` che deve contenere un form dal nome `mod_[modname]_mod_form` usato nella creazione di una nuova istanza

### Callbacks

I moduli attivita' devono implementare le seguenti funzioni in `lib.php:

```php
function [modname]_add_instance($instancedata, $mform = null): int;
function [modname]_update_instance($instancedata, $mform): bool;
function [modname]_delete_instance($id): bool;
```

Queste funzioni vengono chiamate quando viene aggiunta, modificata o eliminata una istanza della nostra attivita', e vanno usate per gestire i dati nelle nostre tabelle. Il `$mform` usato e' quello di sopra.

Nella documentazione Moodle mette in guardia sul fatto che questo elemento e' bersaglio di possibile futura modifica

### Capacita'

Oltre a qualsiasi capacita' prevista dal vostro plugin dovete includere le seguenti capacita':

- `mod/[modname]:addinstance` che determina se un utente puo' aggiungere un istanza di questa attivita'
- `mod/[modname]:view` che determina se l'utente vede le attivita' sul corso

### Indice

Viene richiesta la presenza di una pagina `index.php` che dovrebbe essere usata per elencare tutte le istanze della nostra attivita' alla quale l'utente puo' accedere nel corso. L'utilita' ed effettiva necessita' di quest'ultimo requisito e' discutibile.

Meccanismi opzionali
--------------------

I plugin di attivita' hanno una serie di sistemi che possono implementare opzionalmente se necessario

### Condizioni completamento

[link](https://moodledev.io/docs/apis/core/activitycompletion)

Di base i nuovi plugin mod supportano solo il completamento manuale, ovvero la possibilita' per l'utente di marcare l'attivita' come completata premendo un pulsante.

Per abilitare altre modalita' e' necessario implementare in `lib.php` una funzione `[nomeplugin]_supports`

```php
function forum_supports(string $feature): bool {
    switch($feature) {
        case FEATURE_COMPLETION_TRACKS_VIEWS:
            return true;
        case FEATURE_COMPLETION_HAS_RULES:
            return true;
        default:
            return null;
    }
}
```

Sono disponibili molte modalita' di completamento del corso, per istruzioni dettagliate su come implementarle si rimanda alla documentazione

### Condizioni accesso


[link](https://moodledev.io/docs/apis/core/conditionalactivities)

Altro elemento simile e' quello delle condizioni di accesso all'attivita', ovvero come permettere all'utente di vedere l'attivita'.

A differenza del completamento queste sono configurate dal sito. Sono gia' presenti modalita' come accesso da particolare data, accesso condizionato in base al completamento di attivita' precedenti, ecc..

Esiste una classe di plugin `availability` per definire condizioni addizionali, ad esempio accesso dopo pagamento. Vedi [qui](https://moodledev.io/docs/apis/plugintypes/availability) per la documentazione 

### Backup e restore

i moduli attivita' ed i blocchi supportano la feature di backup, usata per copiare il corso con tutto il contenuto. Per utilizzarla e' necessario che il plugin supporti le API corrispondenti ([link](https://moodledev.io/docs/apis/subsystems/backup)).

Sostanzialmente il backup consiste nella definizione di uno o piu' step che verranno eseguiti.

In breve bisogna implementare due files:

- `backup/moodle2/backup_NOMEPLUGIN_activity_task.class.php`
- `backup/moodle2/backup_NOMEPLUGIN_stepslib.php`
- (OPZIONALE) `backup_NOMEPLUGIN_settingslib.php` con eventuali settaggi relativi al backup della vostra attivita'

I files devono definire gli step di backup, per un esempio vedi la documentazione o l'implementazione di `mod_book`

Per il restore bisogna invece implementare 

- `backup/moodle2/restore_NOMEPLUGIN_activity_task.class.php`
- `backup/moodle2/restore_NOMEPLUGIN_stepslib.php`

in maniera analoga