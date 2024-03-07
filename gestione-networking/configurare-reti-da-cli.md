# Configurare reti da CLI

Gestione delle impostazioni e dei dispositivi di rete con il comando `nmcli` su Red Hat Enterprise Linux:

* Il servizio NetworkManager monitora e gestisce le impostazioni di rete del sistema. Si può interagire con esso tramite riga di comando o strumenti grafici.
* Un dispositivo è un'interfaccia di rete fisica o virtuale. Una connessione ha impostazioni di configurazione per un singolo dispositivo di rete (anche detta profilo di rete).

`nmcli device status`&#x20;

`nmcli d s`

* mostra lo stato di tutti i dispositivi di rete.

`nmcli connection show`

`nmcli c s`&#x20;

* elenca tutte le connessioni. L'opzione `--active` mostra solo quelle attive.

`nmcli connection add`&#x20;

`nmcli c a`

* aggiunge nuove connessioni, i cui dati sono salvati in `/etc/NetworkManager/system-connections/`.

`nmcli connection up`&#x20;

* attiva una connessione sul suo dispositivo. `nmcli device disconnect` disconnette il dispositivo e disattiva la connessione.

`nmcli connection show <NAME>`&#x20;

* mostra le impostazioni correnti di una connessione.

`nmcli connection modify`&#x20;

* modifica le impostazioni, salvandole nel file di configurazione.

Dopo modifiche manuali ai file, `nmcli con reload`&#x20;

* ricarica le configurazioni.

`nmcli connection delete`&#x20;

* elimina una connessione e il suo file di configurazione.

#### Visualizzare connessioni su RHEL9

* `ls /etc/sysconfig/network-scripts/`
* `ls /etc/NetworkManager/system-connections/`

#### Configurazione interattiva delle reti

* `nmtui`

#### Settare l'hostname

* `hostnamectl set-hostname <NAME>`



