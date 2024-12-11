Competenze
===========

- [Link a documentazione ufficiale](https://docs.moodle.org/405/en/Competencies)
- [Link a FAQ](https://docs.moodle.org/405/en/Competencies_FAQ)

Le competenze in moodle sono una raccolta di qualifiche che uno studente puo' ottenere, le competenze possono essere assegnate a piu' corsi, permettendo di raccogliere corsi simili, o diverse edizioni dello stesso corso, nello stesso obiettivo per lo studente.

Il meccanismo consiste nella definizione di un **framework di competenze** (Quadro di competenze in italiano), ovvero una collezione di competenze. La singola competenza e' una capacita/skill che lo studente ha appreso. Le competenze vengono poi legate al completamento dei corsi ed eventualmente di singole attivita, con la stessa competenze che puo' apparire in piu' posti.

Le competenze sono posizionate in una struttura ad albero nel framework, con competenze superiori che raccolgono piu' competenze inferiori. E' possibile dare una nomenclatura diversa alla "competenza" in base al livello. 

Nel framework di competenze e' possibile stabilire le possibili valutazioni ottenibili, di default e' "Completata" e "non completata", ma e' possibile modificare (ad esempio in "Insufficiente", "sufficiente", "Buona", "Eccellente")

Una volta creato il framework di competenze e' possibile assegnarle ad un utente tramite la creazione di un learning plan (Piano di formazione in italiano) che puo' raccogliere una o piu' competenze di un framework, eventualmente fissare una scadenza, ed assegnarlo a studenti o gruppi globali.

Le competenze possono essere impostate per essere completate automaticamente cl completamente del corso/attivita', oppure possono richiedere una revisione manuale. In tal caso e' necessario indicare un ruolo "revisore" dotato della capacita' `moodle/competency:usercompetencyreview` che completera' la richiesta. In questo scenario il completamento del corso non dovra' causare il completamento dell'attivita', ma dovra' essere impostato per "contare come evidenza" in modo che il revisore possa vedere il risultato ottenuto.

E' lo studente in questo caso a richiedere la review, e lo deve fare accedendo alla voce dei piani di formazione enl proprio profilo utente. Qui puo' eventualmente inserire evidenze delle competenze non legate a moodle, indicando un url ed una descrizione della stessa. Sara' poi il revisore a valutare.

Le competenze sono gestite nel menu' di amministrazione nel tab Generale alla sezione "Competenze". In questi menu' e' possibile:

- Attivare o disattivare la funzionalita'
- Migrare i corsi da un framework di competenze ad un altro
- Importare/Esportare i framework di competenze
- Creare un framework di competenze (disponibile su tutto il sito)
- Creare un learning plan 

I framework di competenze specifici per categoria vanno invece create nel menu' della categoria accessibile ad esempio da "Gestione corsi e categorie"

Capacita' autorizzative collegate:

- `moodle/competency:userevidenceview` permette di vedere le evidenze di una competenza di un utente. Nota che il docente non la ha di default.
- `moodle/competency:usercompetencyreview` permette di eseguire la review di una richiesta di completamento di una competenza

