Ricerca Globale
===============

[Link Documentazione](https://docs.moodle.org/405/en/Global_search) (nota: le informazioni sulla documentazione ufficiale sembrano essere non aggiornate)

La ricerca globale e' una funzionalita' di ricerca a livello di sito che permette agli utenti di eseguire delle ricerche su tutta la piattaforma, prevalentemente sui contenuti dei corsi.

Questo meccanismo richiede un indicizzazione dei contenuti tramite dei meccanismi schedulati affinche' funzioni. 

Moodle include un meccanismo integrato che esegue la ricerca tramite query del database, ma come altri aspetti esiste la possibilita' di usare un altro servizio esterno tramite un plugin. In moodle core e' gia' installato il supporto per Solr ([link](https://solr.apache.org/)), ma sono disponibili plugin per [Elasticsearch](https://moodle.org/plugins/search_elastic), [Algolia](https://moodle.org/plugins/search_algolia), [Azure Search](https://moodle.org/plugins/search_azure) e [Postgres + Tika](https://moodle.org/plugins/search_postgresfulltext).

Configurazione in moodle
------------------------

Vediamo come configurare la funzionalita' lato moodle, lasciando per ora da parte l'aspetto di integrazione del motore di ricerca.

La ricerca globale non e' attiva in un installazione base, ma e' una "funzionalita' avanzata" da attivare marcando l'apposito check nel menu Amministrazione del sito -> Funzionalita' avanzate.

Una volta attivata, possiamo trovare le relative configurazione nel menu' Amministrazione del sito -> Plugin -> Search engine. La prima voce (Gestione ricerca globale) presenta, oltre alle impostazioni, una checklist verificata con gli step da intraprendere.

Molti dei settaggi sono ben spiegati nella maschera stessa. Notabile e' il settaggio "Includi tutti i corsi visibili" che determina se la pagina home viene inclusa o no. 

Altra attenzione e' decidere se limitare i risultati ai soli corsi in cui l'utente 'e iscritto o tutti i corsi a cui ha accesso, in questo secondo caso verranno inclusi corsi a cui si puo' accedere come ospite (se impostati), ed e' significativa per gli utenti manager. Non include corsi al quale l'utente puo' iscriversi (ad esempio tramite iscrizione manuale) se non puo' vederli evitando l'iscrizione.

Se la ricerca e' abilitata, l'utente puo' accedervici tramite un icona di ricerca in alto a destra (sul tema boost almeno) e ricercare tra i contenuti a cui ha accesso. E' disponibile anche un blocco con la stessa funzionalita'

### Cosa e' incluso

La ricerca globale include unicamente tipologie di contenuti che moodle decide di fornire per l'indicizzazione, non si tratta di un meccanismo di raccolta indiscriminato delle pagine.

La lista degli elementi soggetti ad indicizzazione (e quindi ricercabili) e' visibile nella maschera Amministrazione del sito -> Ricerca -> Aree di ricerca. Qui e' possibile escludere una o piu' tipologie di contenuti

In breve, i contenuti inclusi sono Divisi in tre categorie
- Utenti: Profili di utenti e messaggi
- Corsi: Titolo, descrizione e campi custom
- Contenuti di corsi: Ogni attivita determina quali, e se, contenuti fornire per la ricerca. 

Tutti gli elementi sono filtrati in base all'accesso dell'utente che fa la ricerca, quindi uno studente non trovera' tutti gli utenti del sito, mentre un manager del sito si.

Indicizzazione
--------------

Affinche le pagine possano comparire come risultati di una ricerca devono prima essere indicizzate dal motore di ricerca.

L'operazione di indicizzazione e' gestita da un task schedulato `\core\task\search_index_task`, che di default viene eseguito ogni 30 minuti. La natura esatta dell'indicizzazione dipende dal motore di ricerca usato ma parte dal contenuto che viene fornito dal gestore del contenuto.

Questo task puo' essere impegnativo per il server se vi e' una grande quantita' di contenuti. Vi e' un settaggio nella configurazione della ricerca apposito che mette un limite massimo di tempo per questo task. Modificando tale settaggio e/o la frequenza del task (in Amministrazione del sito -> server -> task schedulati) possiamo ridurre l'impatto a costo di una ricerca meno accurata per i nuovissimi contenuti.

Motore di ricerca
-----------------

Nella configurazione della ricerca globale si puo' stabilire che motore di ricerca utilizzare.

Il motore di ricerca di default e' denominato **Ricerca semplice** e consiste sostanzialmente nell eseguire query su contenuto del database usando le normali funzioni di ricerca del db usato come `CONTAINS()` (MsSql) o `MATCH () AGAINST ()` (MySql). Notabilmente nel database Postgre utilizza delle feature di vettorizzazione del testo.

Questo motore di ricerca ha il vantaggio di non richiedere altre configurazioni, ma se i contenuti del sito sono molti, potrebbe non essere performante. In tal caso e' necessario utilizzare un altro motore di ricerca.

### Solr

Solr ([Link](https://solr.apache.org/)) e' una soluzione di ricerca basata sulla libreria Apache Lucene ([Link](https://lucene.apache.org/)). Moodle viene gia' installato cin il plugin di integrazione con Solr (Probabilmente perche' e' interamente open source). 

Utilizzando Solr le query includono sintassi avanzate, come la possibilita' di usare wildcard, operatori logici (AND, OR, NOT) o risultati simili nello spazio delle parole usando il carattere "~": "bluebutton~3" trova anche "bigbluebutton". (C'e' da chiedersi se gli utenti saranno a conoscenza di questa feature) 

Solr include anche la funzionalita' di indicizzare i contenuti di files caricati, che va abilitata in moodle nella configurazione

Solr va installato separatamente, ed il PHP va fornito di tutte le estensioni necessarie.

Una guida su come installare Solr e' presente a questo [Link](https://solr.apache.org/guide/solr/latest/deployment-guide/installing-solr.html). In breve dovete scaricarvi i files dal sito, poi nella cartella `bin/` potete utilizzare lo script `install_solr_services.sh` per utilizzare solr come servizio, oppure lanciarlo manualmente con il comando `solr start`.

Il comando `solr` nella cartella `bin` e' utile anche per creare l'indice che verra' poi usato in moodle (core in solr). Se avete installato solr come servizio di default lo trovate in `/opt/solr-9.7.0/bin`

> ATTENZIONE: Solr e' molto sensibile ai permessi sulle cartelle, per evitare problemi impersonate l'utente corretto quando usate la linea di comando. (nell'installazione come servizio tale utente e' `solr`)

```
udo -u solr /opt/solr-9.7.0/bin/solr create -c moodle
```

Questo comando crea un core di nome moodle utilizzabile. I settaggi dei core di Solr non sono argomento di questo corso, ma con questa configurazione di default potrete almeno partire

Alcune considerazioni:

* Di default l'interfaccia web di Solr non e' raggiungibile se non con localhost, nel caso cambiare il settaggio `SOLR_JETTY_HOST` con 0.0.0.0 o 192.168.0.0 o quant'altro
* In caso di errori del tipo `Too many boolean clauses error` dovete modificare il settaggio `maxBooleanClauses` nel settaggio del core. questo si trova nel file `/conf/solrconfig.xml` nella cartella del core, che di default e' `/var/solr/data/NOME_CORE`
* In produzione Solr avra' bisogno di una buona quantita' di memoria, moodle raccomanda 10-20 GB rdi ram (il valore di default e' 512MB)

[Link su dimensionamento Solr](https://lucidworks.com/post/solr-sizing-guide-estimating-solr-sizing-hardware/)

Per sviluppatori
-----------------

E' chiaramente possibile implementare il proprio motore di ricerca, a livello di moodle e' sufficiente implementare un unica classe con i metodi richiesti ([Vedi documentazione](https://docs.moodle.org/dev/Search_engines)), si puo' anche vedere l'implementazione di `search_simpledb` in `/search/engine` per un esempio.

Ovviamente implementare un proprio motore di ricerca e' argomento assai piu' complesso e non oggetto di questo corso.

Per fare si invece che il contenuto di un proprio plugin sia indicizzabile bisogna censire una nuova area di ricerca nel plugin.

Per farlo bisogna creare una classe `\tipo_nome\search\NOMEAREA` che deve ereditare da una della classi in moodle core

- `\core_search\base`
- `\core_search\base_mod` per le attivita
- `\core_search\base_activity` per le informazioni base dell'attivita'
- `\core_search\base_block` per i blocchi

Per un esempio, si puo' consultare l'implementazione in `mod_book`, ed eventualmente prenderla come esempio.

Per i dettagli si rimanda alla documentazione ufficiale: [Link](https://docs.moodle.org/dev/Search_API)

