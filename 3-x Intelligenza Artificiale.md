Intelligenza artificiale
========================

In Moodle esistono due meccanismi che possono essere definite di "intelligenza artificiale": 

- Implementazioni di servizi terzi di LLM e generazione di immagini
- **Analytics** che e' il backend di machine learning per analisi sui dati

Implementazioni servizi terzi
=============================

Moodle prevede la possibilita' di usare modelli di "intelligenza artificiale generativa" esterni per i seguenti task:

- Generazione di testo
- Generazione di immagini
- Riassunto di testi

In Moodle base sono previsti collegamenti con OpenAI e Azure AI, questi vanno abilitati in Amministrazione del sito -> IA -> Provider IA.

Successivamente bisogna abilitare le varie funzionalita' in Amministrazione del sito -> IA -> Posizionamento IA.

Sono disponibili due posizionamenti di base in moodle:

- all'interno dell'editor di testo (solo tinyMCE), dove puo generare teso e/o immagini
- un "riassunto della pagina" che compare abbastanza ovunque

Si puo' regolare l'accesso usando appropriate capacita':

- `aiplacement/editor:generate_text` per generare testo negli editori di testo
- `aiplacement/editor:generate_image` per generare immagini negli editori di testo
- `aiplacement/courseassist:summarise_text` per riassumere le pagine

Come i lettori piu' attenti potranno immaginare, queste sono una nuova tipologia di plugin aggiunta. Nella fattispecie le tipologie di plugin sono due:

I **Provider** di servizi IA sono i plugin `aiprovider` e permettono il collegamento con API, o comunque producono i risultati dei prompt utente

I **Piazzamenti** di servizi IA sono i plugin `Placements` e determinano come l'utente richiede il prompt e come ottiene il risultato

TODO: Entrare nelle specifiche dei plugins, e valutare come fa il riassuntore ad infilarsi nelle pagine (e' un'implementazione core?)

Analitica
=========

[Documentazione ufficiale](https://moodledev.io/docs/apis/subsystems/analytics)

Moodle ha integrato un meccanismo di analisi dati basato sul machine learning, che puo' essere utilizzato su dataset creati a partire da dati di qualsiasi tabella del database.

Un esempio puo' essere il modello preimpostato che prevede la probabilita' che uno studente abbandoni un particolare corso.

L'utilizzo del sistema di analitica richiede una minima conoscenza relativa al machine learning: la documentazione a riguardo e' ben fatta ed introduce quello che e' necessario sapere, ma verificare la bonta' di un modello richiede di sapere come muoversi. 

Sono implementati in moodle core sono modelli basati sulla classificazione binaria, ma sono presenti interfacce per implementare algoritmi di classificazione con piu' categorie ed algoritmi di regressione.

Moodle utilizza le seguenti terminologie:

* i **Target**s sono le label, ovvero i vaolri che vogliamo prevedere
* gli **Indicator**s sono le features normalizzate, ovvero i dati usati per eseguire la predizione
* gli **Analyser**s sono i preprocessori delle labels, ovvero raccolgono i dati grezzi e li preparano per l'analisi o il training dei modelli
* gli **Intervalli di analisi** stabiliscono gli intervalli temporali da utilizzare per le analisi, ad esempio per un modello sui corsi andranno definiti intervalli basati sulle dati dei corsi
* i **Backend di machine learning** sono le implementazioni degli algoritmi di ML
* gli **Insights** sono notifiche (output) generate dai vari processi di ML 

![Schema funzionamento](https://moodledev.io/assets/images/Inspire_API_components-e456d58a4230fca6d50f7450ceeb6ae2.png)

l'affinamento dei modelli e la loro esecuzione e' gestita dai task schedulati `\tool_analytics\task\predict_models` e `\tool_analytics\task\train_models`

Per utilizzare all'interno del nostro plugin dobbiamo definire i vari elementi utilizzando nomenclature e prassi corrette, vedi la documentazione per i dettagli

Backend ML
----------

[Documentazione ufficiale](https://moodledev.io/docs/apis/plugintypes/mlbackend)

I modelli di ML utilizzati in moodle sono definiti dai plugin di classe `mlbackend`.

L'implementazione di tali plugin richiede l'utilizzo di una o entrambe le interfacce `classifier` per gli algoritmi di classificazione e `regressor` per gli algoritmi di regressione.

Di base in moodle sono disponibili due plugin di questo tipo: 

* un backend PHP, utilizzabile out of the box, che usa la regressione logistica
* un backend Python che usa una rete neurale basata sulla libreria [tensorflow](https://www.tensorflow.org/). Questo plugin richiede python installato sulla macchina (raccomandato 3.7) ed utilizza un [package](https://pypi.org/project/moodlemlbackend/0.0.5/) propietario di moodle