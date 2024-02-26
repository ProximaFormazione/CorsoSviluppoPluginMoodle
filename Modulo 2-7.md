Utilizzo database
=================

Finora abbiamo utilizzato plugin stateless, ma nella maggior parte dei casi e' necessario far persistere dei dati su una qualche forma di database.

Avere una struttura modulare, dove i plugin possono essere installati ed aggiornati, composrta una serie di complessita' nella gestione dello schema, che fortunatemente sono gestite in moodle in maniera sistematica.

Vediamo in questo capitolo le basi di quanto serve per utilizzare il database nei plugin in moodle.



Consumare il database
=====================

Per utilizzare il database, moodle mette a disposizione delle sue API interne, nella variabile globale `$DB`

`$DB` e' un istanza della classe `moodle_database` che si trova nel file `lib/dml/moodle_database.php`. La classe e' ben documentata ma dovete aprire il file per vedere i commenti.

