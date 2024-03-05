# Sommario

* Con il comando ssh, gli utenti possono accedere a sistemi remoti in modo sicuro con il protocollo SSH.
* Un sistema client memorizza le identità dei server remoti nei file `~/.ssh/known_hosts` e `/etc/ssh/ssh_known_hosts`.
* SSH supporta sia l'autenticazione basata su password che quella basata su chiave.
* Il comando `ssh-keygen` genera una coppia di chiavi SSH per l'autenticazione. Il comando `ssh-copy-id` esporta la chiave pubblica nei sistemi remoti.
* Il servizio sshd implementa il protocollo SSH sui sistemi Red Hat Enterprise Linux.
* Configurare le impostazioni avanzate di SSH nel file di configurazione `/etc/ssh/sshd_config`.
* È consigliabile configurare sshd per disabilitare i login remoti come root e per richiedere l'autenticazione con chiave pubblica piuttosto che con password.

