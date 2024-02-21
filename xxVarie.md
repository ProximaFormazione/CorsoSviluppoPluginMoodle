Date
====

Le date nel database di moodle sono salvati in [timestamp unix](https://it.wikipedia.org/wiki/Tempo_(Unix)), ovvero con un intero che indica il numero di secondi passati dall' 1 gennaio 1970.

Esistono vari metodi di aiuto per gestire i timestamp in moodle ([Documentazione completa](https://moodle.academy/mod/lesson/view.php?id=817&pageid=162)).

Interessatne la funzione `userdate` e la funzione `usertime` che tengono presente il fuso orario indicato dall'utente.

php ha una funzione `time()` per ottenere il timestamp attuale.

