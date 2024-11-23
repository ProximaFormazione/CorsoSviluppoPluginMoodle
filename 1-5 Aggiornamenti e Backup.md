Backup
======

Gli elementi da mantenere della piattaforma sono tre:

- Il sorgente
- Il database
- La cartella moodledata

Database
--------

Per il database, e' necessario salvare il  database nella sua interezza. Tutti i motori di database implementano funzionalita' adatte e l'argomento non e' certo nuovo

Sorgente
--------

Per il sorgente, si raccomanda di utilizzare Git.

In questo modo potete mettere tutto il sorgente nel repository, inclusi i plugin sviluppati localmente che magari hanno il loro repository di sviluppo, ma che qui sono inclusi come files e basta. Questo repository verra' poi usato per caricare i files in produzione.

Con questo approccio e' necessario considerare la possibilita' di aggiornare il repository dalla produzione. Questo puo' essere necessario se gli utilizzatori del moodle hanno la possibilita' di caricare plugin addizionali, in questo caso la presenza o meno di modifiche da committare denota un caricamento di nuovo plugin. Possiamo schedulare un task che esegua il commit in caso di modifiche con il seguente script bash:

```
#!/bin/bash

cd /var/www/html/moodle

if [[ `git status --porcelain` ]]; then
  git add .
  git commit -m "Allineamento $(date +"%Y %m %d %H%M")"
  git push
else
  echo "Nessuna modifica rilevata"
fi

```

Chiaramente questa attenzione e' da avere solo per i moodle a singola istanza (con piu'web server il caricamento di plugin va impedito).

Se il sorgente e' sotto repository allora e' sotto backup per definizione. Sara' ovviamente opportuno avere backup del repository stesso.

Cartella moodledata
-------------------

La cartella moodledata contiene tutti i dati che non sono consoni alla conservazione nel database , fa parte integrante dell'installazione e va conservata.

Non tutte le cartelle hanno la stessa importanza, alcune contengono dati temporanei che non richiedono di essere conservati. 

Una spiegazione esaustiva dei contenuti di tutte le sottocartelle e' stata vista in un capitolo precedente, in breve qui elenchiamo le sottocartelle che richiedono di essere messe sotto backup:

- **filedir** contiene i files caricati tramite upload dagli utenti
- **trashdir** contiene i files recentemente eliminati, ma ancora visibili in moodle
- **lang** contiene i language pack, e soprattutto eventuali modifiche apportate dagli utenti alle stringhe degli stessi 
- **muc** (se presente) contiene dei settaggi relativi alle caches (non contiene le caches stesse)
- **repository** (se presente) contiene files caricati dagli utenti direttamente sul filesystem
- **models** (se presente) contiene files relativi ai modelli di machine learning allenati

Le seguenti cartelle invece **NON** richiedono di essere messe sotto backup:

- *cache* (se manca viene ricreata coi contenuti)
- *localcache* (come sopra)
- *temp* (files temporanei dei plugin)
- *sessions*
- *lock*
- *antivirus_quarantine* viene usata dai plugin di antivirus per i files sospetti, malgrado siano effettivamente informazioni rilevanti probabilmente e' meglio non portarsi il contenuto nei backup 

Altro
-----

Generalmente non e'richiesto mettere altri elementi sotto backup. Il contenuto di sistemi di caching come ad esempio redis puo'essere ricreato automaticamente dalla piattaforma se mancante. Puo'comunque essere utile avere copie della configurazione dei server e di eventuali ambienti virtuali utilizzati.

Aggiornamenti
=============

Moodle e' un software open source, il che porta sia benefici che problemi dal punto di vista della vulnerabilita' del prodotto.

Da un lato eventuali vulnerabilita' di sicurezza vengono identificate prima che diventino note per via di grandi incidenti, e le vulnerabilita' vengono regolarmente chiuse in patch di sicurezza. Dall'altro, essendo le vulnerabilita' pubbliche, se la vostra piattaforma non e' aggiornata allora effettivamente esiste un manuale su quali vulnerabilita' esistono.

E' necessario quindi mantenere sempre aggiornata la piattaforma all'ultima versione disponibile.

Non e' previsto alcun meccanismo di aggiornamento automatico all'interno di moodle per il sito (e' presente un simile meccanismo per i singoli plugins), quindi il processo di aggiornamento deve essere gestito.

Se si utilizza un repository git per il sito in produzione (altamente consigliato), allora un possibile processo di aggiornamento, facilmente automatizzabile in molte sue parti, e' il seguente:

1. Per i siti ad istanza singola dove gli amministratori possono caricare plugin bisogna committare eventuali modifiche presenti in produzione (= nuovi plugins installati), opzionalmente si puo' mettere il sito in modalita' manutenzione per impedire agli utenti di collegarsi (Amministrazione dle sito ->server -> modalita' manutenzione)

```
cd $(path_per_moodle); if [[ `git status --porcelain` ]]; then git add . && git commit -m "Allineamento $(date +"%Y %m %d %H%M")"; else echo "Nessuna modifica da pushare"; fi; git push
```

2. In un ambiente di sviluppo scaricare il repository ed effettuare il merge dal branch della versione utilizzata di moodle (es `MOODLE_405_STABLE`) sul repository ufficiale ([Link](https://github.com/moodle/moodle))
    - Risolvere eventuali conflitti se presenti. Data l'architettura a plugins tali conflitti sono rari, ma potrebbero avvenire se avete applicato customizzazioni a moodle core
    - A seconda di quanti ambienti di staging prevedete, il merge andra' fatto nel branch relativo (es: `prerelease`)

3. Su un ambiente di staging (eventualmente anche gestito localmente da chi sta curando l'aggiornamento) eseguire un pull con il merge appena effettuato. Negli ambienti di staging potete eseguire l'installazione a mano collegandovi direttamente con un browser, oppure usare uno script (vedi sotto)

4. Verificate il corretto funzionamento della piattaforma.

5. Una volta verificato che non ci siano problemi, portare le modifiche sul branch di produzione quando volete iniziare il rilascio effettivo
    - Se nella vostra installazione e' possibile per gli utenti installare plugin, dovete prima verificare che non siano state fatte aggiunte mentre eseguivate i punti 1-4, potete automatizzare questo controllo con il seguente script: `cd $(path_per_moodle); if [[ `git status --porcelain` ]]; then exit 1; else echo ok; fi` che produce un errore in caso di modifiche locali

6. In produzione eseguite il pull del branch utilizzato. In presenza di piu'istanze questo step puo' assumere forme diverse a seconda della tecnologia usata per creare le istanze.

7. Se non lo avete ancora fatto, questo e' il momento di mettere il sito in modalita' manutenzione (Amministrazione dle sito ->server -> modalita' manutenzione)

8. Eseguire l'installazione dell'aggiornamento per aggiornare il database. E' sufficiente collegarsi ad una singola istanza per procedere, ma deve essere consistentemente la stessa (la procedura prevede il passaggio su almeno tre pagine distinte)

9. Togliere il sito dalla modalita' manutenzione

Gli step 7-9 possono essere eseguiti in maniera automatica utilizzando i comandi CLI previsti da moodle ([Link](https://docs.moodle.org/405/en/Administration_via_command_line)), questo e' necessario in presenza di moodle dietro reverse proxy (es con Kubernetes), e puo' essere utile per ridurre il downtime, evitare errori o se si desidera schedulare l'aggiornamento fuori orario di lavoro (a vostro rischio e pericolo se lo fate in produzione)

un esempio di script che mette il sito in modalita' manutenzione, esegue l'aggiornamento e poi rimuove la modalita' manutenzione:

```
cd $(path_per_moodle); git pull; php admin/cli/maintenance.php --enable; php admin/cli/upgrade.php --non-interactive; php admin/cli/maintenance.php --disable
```

Con questo tipo di aggiornamento eventuali nuovi settaggi dei plugin verranno inseriti con i valori di default previsti dai loro sviluppatori.

> NOTA: Nella configurazione tipica su ambienti Linux,il PHP ha due diverse installazioni per il servizio FPM (usato dal web server) e per il PHP a linea di comando, ciascuna con il suo file di configurazione (es: `/etc/php/8.1/cli/php.ini` e `/etc/php/8.1/fpm/php.ini`). Quando eseguite l'aggiornamento a linea di comando i requisiti che vengono verificati sono quelli della versione a linea di comando, che non e' quella generalmente usata dal sito. Dovrete quindi avere cura che la versione a linea di comando abbia le stesse estensioni e sia configurata analogamente a quella FPM (o almeno abbia i requisiti minimi di moodle)  altrimenti l'installazione non partira'. 