BigBlueButton
=============

[BigBlueButton](https://bigbluebutton.org/) (BBB) e' un software open source di video conferenza con enfasi particolare sul caso d'uso della formazione. E' distribuito con licenza GNU che permette l'utilizzo libero per attivita' commerciali e non.

BBB ha una serie di funzionalita' interessanti: ha integrato un sistema di presentazione dove e' possibile caricare documenti ed eseguire annotazioni durante la presentazione, con la possibilita' di permettere ad uno o piu' utenti collegati di partecipare (modalita' lavagna condivisa), ed inoltre:

- Possibilita' di registrare le riunioni
- Possibilita' di eseguire test o sondaggi al volo con i partecipanti
- Possibilita' di condividere video esterni
- Possibilita' di caricare files come nuove presentazioni 
- Funzionalita' tipiche di altri sistemi di video conferenza: condivisione schermo, alzata di mano, chat (scaricabile)

La qualita video ed audio e' adeguata, generalmente ritenuta al pari se non migliore di altre alternative dagli utenti, ed a seconda del server puo' gestire numeri alti di partecipanti (abbiamo ad esempio usato BB in una conferenza con 200 partecipanti in contemporanea senza intoppi).

Il problema di BBB risiede nel fatto che dovete fornire un server dove installarlo: non esiste un ente o fondazione centrale che lo fornisce come servizio cloud (ma ci sono rivenditori hosting terzi). Configurare correttamente un server con BBB non e' immediato e ha una certa curva di apprendimento.

Installazione
-------------

La procedura di installazione e' abbastanza semplice in quanto e' disponibile uno script ufficiale che esegue tutte le configurazioni necessarie.

Lo script e' visibile su [Github](https://github.com/bigbluebutton/bbb-install) e contiene tutta la documentazione dettagliata.

Senza ripetere l'intera documentazione, riportiamo alcuni punti salienti.

Per iniziare avrete bisogno di:

- Un server dedicato per l'installazione, BBB ha requisiti significativi:
    - almeno 8 core dedicati
    - almeno 16 GB di RAM
    - almeno 50 GB di spazio disco, se si vuole usare la registrazione ne servira' molto di piu' (almeno 500 GB)
    - Accesso alle porte TCP 80, 443 ed UDP 16384 - 32768
    - Un Indirizzo IPV4 ed IPV6 (opzionalmente IPV6 si puo' disattivare manualmente)
    
BBB e' garantito solo su Ubuntu 20.04. Se il server e' accessibile pubblicamente servira' anche un hostname valido per il certificato SSL.

Una volta scaricato lo script di installazione possiamo eseguirlo specificando vari settaggi. Ad esempio:

```
wget -qO- https://raw.githubusercontent.com/bigbluebutton/bbb-install/v2.7.x-release/bbb-install.sh | bash -s -- -w -v focal-270 -s www.sito.it -e nome.cognome@mail.it -g
```

scarica lo script ed esegue l'installazione in un colpo solo

- `-w` Configura il firewall di ubuntu in automatico
- `-v focal-270` Indica la versione di BBB da installare
- `-s www.sito.it` e' l'hostname del server (ovviamente da cambiare)
- `-e nome.cognome@mail.it` installa un certificato con Let's Encrypt
- `-g` installa Greenlight, che e' un'interfaccia grafica per configurare BBB senza usare API

NOTA: Lo script configura NGINX per utilizzare l'IPV6, se non e' disponibile bisogna eseguire modifiche allo script (vedi [Qui](https://github.com/bigbluebutton/bbb-install/issues/747) per i dettagli), e sara' necessario eseguire altre modifiche dopo l'installazione (documentate).

Una volta terminato lo script si puo' verificare lo stato con il comando `bbb-conf --check`. Molti problemi tipici vengono identificati da BBB con suggerimenti come procedere nell'output del comando precedente.

Il secret per le API e' accessibile con il comando `bbb-conf --secret`.

Si puo' creare un account amministratore con il comando `docker exec -it greenlight-v3 bundle exec rake admin:create['name','email','password']` inserendo nome mail (che e' l'username) e password.

Utilizzo
--------

Da solo BBB puo' essere usato autonomamente come sistema di video conferenze tramite l'interfaccia grafica "Greenlight"
(se installata).

Alternativamente sono disponibili API per eseguire l'integrazione con propri programmi.

In moodle e' presente un plugin di default che ha tale integrazione. 

BBB richiede la creazione di una stanza per riunione, e' poi possibile condividere il link per fare accedere gli utenti alla stanza. Il plugin di integrazione con moodle esegue tutte queste operazioni dietro alle quinte e non richiede di accedere al server di BBB separatamente.

Il plugin in moodle e' preinstallato di default dalla versione 4 in avanti, tuttavia e' necessario attivarlo nel menu di Amministrazione -> Plugin -> Moduli Attivita' -> Gestione attivita'. Bisogna poi configurare globalmente il plugin inserendo url e secret per collegarsi alle API (lanciare sul server il comando `bbb-conf --secret` per averle).

Una volta configurato il plugin, e' possibile aggiungere l'attivita ai corsi, con varie impostazioni disponibili come abilita/disabilita registrazioni, webcam ecc, o stabilire chi sara' il moderatore della riunione.

Possibili problemi
------------------

Il plugin di integrazione in moodle e' abbastanza lineare e se configurato correttamente generalmente non da problemi.

La configurazione del server e di BigBlueButton stesso e' invece piu' complessa, per le varie complessita che comportano i software di videoconferenza.

La categoria di problemi piu' comune e' l'utente che non riesce a connettersi (video e/o audio) per porte bloccate o impostazioni di sicurezza (sue) troppo stringenti.

Per ovviare al problema del firewall e' opportuno configurare un server TURN. Nelle ultime versioni coturn (server TURN su ubuntu) e' gia' preinstallato, altrimenti [Qui](https://docs.bigbluebutton.org/administration/turn-server/) e' disponibile la documentazione.

Altre problematiche dipendono dai casi e non sono oggetto di questa documentazione. 