Moodle Network
==============

[Documentazione](https://docs.moodle.org/405/en/MNet)

Moodle Network, o MNET (da non confondere con Moodle Net, che e' la rete di condivisione di corsi) e' una feature di moodle che permette di collegare Moodle diversi e permettere agli utenti di passare da una piattaforma all'altra liberamente.

Nella sua forma piu' base, e' possibile collegare due Moodle in modo che gli utenti di uno possano utilizzare l'altro con le stesse credenziali, di fatto i moodle diventano identity provider per l'altro in un sistema di Single Sign On. Opzionalmente e' disponibile un plugin per iscrivere remotamente gli utenti sui corsi di un altra piattaforma.

> ATTENZIONE: La funzionalita' di MNet e' marcata come deprecata dalla versione 3 e quindi candidata per essere rimossa dalle versioni future di moodle. Questo chiaramente comporta un rischio, anche se non essendoci un rimpiazzo non e' chiaro se effettivamente verra' rimossa o no, si tratta comunque di una feature non piu' seguita

Configurazione
--------------

Per abilitare la funzionalita' ci sono alcuni prerequisiti da soddisfare:

- sul server devono essere installate le estensioni del **PHP** curl ed **openssl** (generalmente incluse nelle distribuzioni del PHP)
- bisogna abilitare la funzionalita' in Amministrazione del sito > Funzionalita' avanzate alla voce **networking**

Una volta attivato il networking e' possibile accedere al tab corrispondente nell'amministrazione del sito dove e' possibile eseguire le impostazioni.

La maschera piu' importante e' *Manage Peers* (*Gestione Nodi*) dove vanno registrati i siti collegati. Per ogni sito e' poi possibile stabilire quali servizi consumare e/o quali servizi fornire.

I servizi includono l'identity provider, sia in un verso che nell'altro, o la possibilta' di eseguire iscrizioni remote. Ogni servizio ha un opzione per fornire il servizio al sito in questione ed un opzione per consumare il servizio dall'altro sito. Se l'altro sito sta richiedendo/fornendo il servizio appare un check verde a fianco dell'opzione.

Per una configurazione corretta e' necessario registrare i rispettivi indirizzi su entrambe le piattaforme usando l'url base del sito. 

Opzionalmente e' possibile impostare una delle due piattaforme in modalita' *promiscua* che fa in modo che il sito registri come peer qualsiasi sito cerchi di collegarsi con questa funzionalita'. Bisogna poi comunque abilitare i servizi una volta che il sito compare nella lista, oppure abilitare i servizi in maniera indiscriminata alla voce "all hosts".

Identity Provider
-----------------

La funzionalita' di IdP permette agli utenti di navigare da un moodle all'altro.

Per abilitare questa funzionalita e' necessario:

- Fornire e consumare i servizi di IdP tra i due server
- Attivare il plugin di autenticazione "MNet"
- Fornire la capacita' `moodle/site:mnetlogintoremote` "Roam to a remote Moodle" agli utenti cui si vuole fornire tale possibilita' (assegnarlo ad "Authenticated user" per abilitarlo per tutti)

Una volta eseguito cio' un utente puo' navigare sul secondo moodle. Potete inserire link all'altra piattaforma e l'utente sara' in grado di utilizzare le sue credenziali.

Di fatto gli utenti vengono importati sulla seconda piattaforma al primo login su di essa. E' possibile alterare la mappatura dei campi utente se necessario in una relativa opzione del menu' Amministrazione > Networking

Iscrizione remota
-----------------

L'iscrizione remota e' un altro servizio attivabile in MNet. Permette ad un amministratore su una piattaforma di iscrivere utenti a corsi che si trovano sull'altra piattaforma.

Per abilitare un corso sulla propria piattaforma per l'iscrizione remota, e' necessario abilitare il relativo metodo di iscrizione ai corsi.

Il menu' che permette l'iscrizione remota e' nel tab Amministrazione > Networking


Possibili problemi
------------------

Alcune distribuzioni linux (es: Ubuntu 22.04) non hanno openssl configurato con l'algoritmo RC4 per la cifratura, una possibile soluzione e' modificare la configurazione di openssl (es: in /etc/ssl/openssl.cnf)

```
[provider_sect]
default = default_sect
legacy = legacy_sect

...


[default_sect]
activate = 1

[legacy_sect]
activate = 1
```

occorre poi riavviare il php con `sudo service php8.1-fpm restart`