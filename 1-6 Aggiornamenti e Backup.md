Backup
======

Gli elementi da mantenere della piattaforma sono tre:

- Il sorgente
- Il database
- La cartella moodledata

Per il database, e' necessario salvare il  database nella sua interezza. Tutti i motori di database implementano funzionalita' adatte e l'argomento non e' certo nuovo

Per il sorgente, si raccomanda di utilizzare Git.

In questo modo potete mettere tutto il sorgente nel repository, inclusi i plugin sviluppati localmente che magari hanno il loro repository di sviluppo, ma che qui sono inclusi come files e basta. Questo repository verra' poi usato per caricare i files in produzione.

Con questo approccio e' necessario considerare la possibilita' di aggiornare il repository dalla produzione. Questo puo' essere necessario se gli utilizzatori del moodle hanno la possibilita' di caricare plugin addizionali, in questo caso la presenza o meno di modifiche da committare denota un caricamento di nuovo plugin.

Chiaramente questa considerazione e' valida solo per i moodle a singola istanza (con piu'web server il caricamento di plugin va impedito).

Se il sorgente e'sotto repository e' sotto backup per definizione. Sara' ovviamente opportuno avere backup del repository stesso.


