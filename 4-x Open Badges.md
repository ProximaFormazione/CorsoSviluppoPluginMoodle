Open Badges
===========

Gli Open Badge ([LINK](https://openbadges.org/)), (a volte indicato come "standard 1EDTECH"), Sono un formato di metadati associabili a files (tipicamente immagini) standardizzato che racchiude dentro di se informazioni relative ad una certificazione di competenze, alla persona che le ha conseguite, ed a l'ente che l'ha rilasciato. Il formato prevede anche una procedura di verifica dei badges.

![Schema riepilogativo](https://openbadges.org/sites/default/files/assets/Open%20Badges/Build%20Page/OB_Best_Practices.svg)

In questo capitolo non ci occuperemo delle specifiche dei metadati conservati nel badge, ne citeremo giusto alcuni laddove vengono impostati da moodle. La documentazione ufficiale del formato e' disponibile online ([LINK](https://www.imsglobal.org/sites/default/files/Badges/OBv2p0Final/index.html)).

Moodle rilascia badge usando la versione 2.0 dello standard, ora siamo alla 3.

Alcune terminologie utili:

- i **backpack** sono siti che ospitano i badge di un utente, generalmente forniscono servizi di condivisione e verifica integrati. Gli open badge sono files indipendenti e non necessitano di un backpack obbligatoriamente.
- il **recipient** e' il destinatario del badge. La mail e' crittata nei metadati per eseguire la validazione 'e necessario indicare l'indirizzo mail del destinatario

I badge si gestiscono nel menu di amministrazione all voce "Badges" nel primo tab.

Per creare un badge si procede dalla voce di menu omonima nel menu' di amministrazione. E' possibile stabilire qui i criteri di completamento per il badge, tra cui:

- Assegnazione manuale
- Completamento di corso
- Completamento di altri badge
- Completamento di competenze
- Appartenenza a gruppi globali o campi sul profilo utente

Se non viene indicato altrimenti, la piattaforma Moodle diventa il validatore del badge. E' possibile indicare url di validatori terzi.

Possiamo aggiungere al badge una serie di competenze esterne nel campo "Alignment" (in italiano "Equivalenza"), con un url di riferimento, es `https://www.thecorestandards.org/ELA-Literacy/RST/11-12/3/`.

Una volta terminata la creazione del badge, questo va attivato in modo che gli studenti possano ottenerlo. L'attivazione consegna il badge agli studenti che verificano i criteri, anche retroattivamente (es: consegna il badge a chi hha gia' completato il corso)

> ATTENZIONE: Se il badge viene erogato, alcune sue caratteristiche non sono piu' modificabili, questo perche' i dati devono corrispondere a quelli sui badge gia' rilasciati per la validazione. E' possibile creare versioni successive in sostituzione.

Moodle supporta il collegamento con backpack esterni, l'utente puo' mostrare i badge presenti sul suo backpack se esegue l'autenticazione da Menu in alto a dx -> Opzioni -> Impostazioni backpack. Lo standard open badge 2.0 definisce anche il protocollo di collegamento con i backpack, quindi per aggiungerne uno e' sufficiente inserire l'url del suo endpoint.

I badge possono essere creati anche dal docente di un corso, in questo caso i badge sono legati unicamente al corso. Un amministratore puo' disattivare questa funzionalita' a livello di sito

