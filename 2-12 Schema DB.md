Schema Database
===============

Per utilizzare moodle, a volte puo' essere richiesto eseguire query su tabelle non generate da noi, in tal caso puo' essere utile avere un idea generale di come e' fatto lo schema.

Una documentazione autogenerata dello schema, cone le relazioni tra le tabelle ed i commenti preservati, e' disponibile a guesto [>>LINK<<](https://www.examulator.com/er/output/index.html)

In questa guida, i nomi delle tabelle verranno inseriti con il prefisso `mdl_` , ma questo potrebbe essere diverso in base al valore che avete impostato in fase di installazione

Principi generali
-----------------

Alcune indicazioni di carattere generale per orientarsi meglio nel database:

Come gia' visto, le tabelle specifiche dei plugin sono prefissate dal nome Frankenstyle del plugin, come ad esempio `mdl_enrol_lti_users`, `mdl_auth_oauth2_linked_login`. I plugin di attivita' fanno leggera eccezione nel senso che omettono il prefisso `mod_` e mettono unicamente il nome del plugin, con tabelle tipo `mdl_book`, `mdl_book_chapters`.

Ogni tabella dovrebbe avere una primary key chiamata `id`, eventualy foreign keys dovrebbero essere chiamate `nometabellaid`, quindi un campo courseid dovrebbe essere una FK per la tabella course.

Tabelle Moodle Core
-------------------

Vediamo alcune delle tabelle principali usate da moodle core.

### Utenti

La tebella `mdl_user` e' la tabella principale di riferimento dell'utente, oltre ai dati anagrafici inseriti dallo stesso contiene:

- Il metodo di autenticazione legato all'utente (nome plugin)
- Indicazione se l'utente e' confermato/sospeso/eliminato.
- Indicazione sul fatto che l'utente ha accettato o no la policy privacy

I record degli utenti non vengono mail eliminati fisicamente dalla tabella, in caso di utenti eliminati moodle modifica la mail con un hash e nell'username mette la mail con la data di eliminazione in epoch (username ha un indice univoco)

I campi personalizzati dell'utente sono memorizzati in altre due tabelle: `mdl_user_info_field` ha le definizioni dei campi mentre i valori sono in `mdl_user_info_data`, relazionati a utente e campo.

Sono presenti tabelle con informazioni sulle credenziali e gli accessi: `mdl_user_lastaccess`, `mdl_user_password_history`, `mdl_user_password_resets`

### Settaggi

i Settaggi della piattaforma sono memorizzati in un formato nome/valore come singoli record. La tebella `mdl_config` ha i settaggi di moodle core, mentre i settaggi dei plugin sono memorizzati nella tabella `mdl_config_plugins`. Questa tabella contiene la versione installata del plugin al nome "version"

I Settaggi degli utenti sono conservati nella tabella `mdl_user_preferences` nello stesso formato name/value

### Contesti

I Contesti autorizzativi sono memorizzati come record nel database nella tabella `mdl_context`, alcune particolarita' di questa tabella:

`path` e `depth` sono colonne che contengono la struttura ad albero dei contesti, path ha la genealogia dei contesti con gli id separati da slash, ed e' una colonna ricorrente in altre entita' con struttura ad albero (come le categorie)

`contextlevel` indica la tipologia di contesto:

- CONTEXT_SYSTEM = 10;
- USER = 30;
- COURSECAT = 40;
- COURSE = 50;
- MODULE = 70;
- BLOCK = 80;

`instanceid` e' la chiave primaria della tabella dell'entita' associata al contesto.

- `mdl_user` per gli utenti
- `mdl_course_categories` per le categorie
- `mdl_course` per i corsi
- `mdl_course_modules` per le attivita' del corso
- `mdl_block_instances` per i blocchi

### Corsi ed iscrizioni

La tabella master dei corsi e' `mdl_course`, e tutte le tabelle `mdl_course_XXX` sono generalmente relative a vari aspetti dei corsi, come ad esempio i criteri di completamento.

Le categorie sono in `mdl_course_categories`.

Le iscrizioni hanno una serie di tabelle coinvolte:

Per iniziare la tabella `mdl_enrol` contiene la definizione delle modalita' di iscrizione impostate per ciascun corso. Questa tabella mette a disposizione valori per le istanze qualora ne dovessero fare uso (es: prezzo corso per iscrizioni e-commerce). La lista dei plugin attivabili a livello di sito e' un settaggio di `mdl_config` (enrol_plugins_enabled).

Le iscrizioni degli utenti ai corsi sono salvate nella tabella `mdl_user_enrolments` con riferimento alla modalita' nel campo `enrolid`

### Attivita' del corso

Le attivita' hanno l'implementazione penso meno intuitiva presente sul database.

Per iniziare la tabella `mdl_modules` ha la lista dei plugin attivita' presenti in piattaforma, con indicazione di quali sono attivi o no.

Ogni plugin di attivita' deve poi definire una tabella master con il nome del plugin i cui record sono le singole istanze dell'attivita.

Ad esempio `mod_label` (area di testo immagini) ha la tabella `mdl_label`. Questa tabella ha dei campi obbligatori (`id`, `course`, `name`, `timemodified`, `intro`, `introformat`) ma al di la di quelli ogni plugin puo' utilizzare campi specifici per le proprie funzionalita' (se non utilizzare direttamente altre tabelle).

Sebbene la tabella master del modulo abbia il campo `course`, la tabella che fa testo nello stabilire che atttvita' sono visibili in un corso e' la tabella `mdl_course_modules`. L'id di questa tabella e' ampiamente usato nel codice al punto che viene generalmente usata l'abbreviazione `cmid` per indicarla.

Questa tabella collega il corso (colonna `course`) al record dell'attivita'. Per farlo e' necessario vedere prima di che attivita' si tratta(colonna `module`) e poi possiamo fare la join con la tabella specifica per beccare l'istanza esatta (colonna instance). Questa tabella contiene poi le configurazioni standard delle attivita'

### Blocchi

La tabella `mdl_block` contiene le tipologie di plugin di tipo blocco installate, mentre la tabella `mdl_block_instances` contiene i vari blocchi che compaiono nel sito. Notabile e' il caso dei plugin di tipo "testo" che immagazzinano il testo in questa tabella nel campo `configdata` in formato base64

### AI

Moodle 4.5 salva informazioni dettagliate sulle richieste effettuate ad API di IA generativa. Le tabelle relative alle chiamate sono `mdl_ai_action_XXXX` con `mdl_ai_action_register` come master e le tabelle specifiche con i dettagli delle richieste, tra cui i token utilizzati per richiesta e risposta, utili se si vuole fare un analisi dei costi di utilizzo.

Le tabelle `mdl_analytics_XXX` sono invece relative ai meccanismi di Machine learning

### Altro

- `mdl_capabilities` ha la lista delle capacita'
- `mdl_cohort` e `mdl_cohort_members` riguardano i gruppi globali
- `mdl_groups` e `mdl_groups_members` riguarda i gruppi interni ai corsi
- la tabella dei log e' `mdl_logstore_standard_log`, mentre la tabella `mdl_log` e' deprecata
- Molte funzionalita' tipicamente assimilabili a moodle base sono nelle tabelle `mdl_tool_XXXX`, ad esempio `mdl_tool_policy` ha le policy di privacy.


