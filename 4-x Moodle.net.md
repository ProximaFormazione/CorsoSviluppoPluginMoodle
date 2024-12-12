Moodle.net
===========

[Link a guida](https://docs.moodle.org/35/it/Moodle.net)

Moodle.net e' una piattaforma dove e' possibile pubblicare i corsi della propria piattaforma. Vi e' una istanza pubblica (all'indirizzo moodle.net) o alternativamente e' possibile utilizzare una propria istanza.

I corsi possono essere riportati direttamente sul vostro moodle da moodle.net, oppure scaricati in formato caricabile da restore corso. Chiunque puo' eseguire questa operazione, inoltre e' possibile inserire attivita' da moodle.net dall'apposita voce in calce alla lista della attivita'

![E' sempre stato li](https://docs.moodle.org/404/en/images_en/c/c5/browswforcontent.png)

Questa funzionalita' e' abilitata di default,. E' possibile disattivarla nel menu di amministrazione del sito in "funzionalita' avanzate", nella voce di menu relativa invece e' possibile specificare una diversa istanza a cui connettersi.

Da notare che e' possibile scaricare i corsi come files da moodle net, quindi di fatto non e' possibile impedire ad un utente motivato di caricare un corso da li (se ha la possibilita' di caricare corsi).

Pubblicare su Moodle.Net
------------------------

[Link a guida](https://docs.moodle.org/404/en/Share_to_MoodleNet)

Per permettere la pubblicazione di proprio materiale verso moodle.net bisogna attivare la funzionalita' in Amministrazione del sito -> Sviluppo -> Impostazioni sperimentali.

Per collegarsi serve un servizio OAuth2 verso moodle.net. La documentazione sembra suggerire che non sembra possibile collegarsi ad una istanza in-house, ma potrebbe riferirsi unicamente al fatto che il preset e' impostato per quello.

Una volta configurata la connessione, gli utenti con la capacita ` moodlenet:shareactivity` e/o ` moodlenet:sharecourse` potranno eseguire i push di attivita' e/o corsi.

Senza questo meccanismo l'unico modo per caricare i corsi e' caricare i backup manualmente su Moodle.net.

Gestire il proprio moodle.net
-----------------------------

E' possibile eseguire la propria istanza di moodle.net. 

Per una guida di installazione si rimanda alla documentazione ([Link](https://docs.moodle.org/dev/MoodleNet)). In breve lo stack tecnologico e':

- Node.js 
- npm
- ArangoDb

Da notare che lo sviluppo di moodle.net e' degli ultimi 4 anni, ed e' stato rilasciato da 2 quindi si tratta di un prodotto meno scafato rispetto a Moodle

