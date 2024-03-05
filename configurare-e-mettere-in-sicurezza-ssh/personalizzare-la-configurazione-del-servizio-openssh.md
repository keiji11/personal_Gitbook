# Personalizzare la configurazione del servizio OpenSSH

### Personalizzazione della Configurazione del Servizio OpenSSH

Obiettivi**:**&#x20;

Disabilitare i login diretti come utente root e l'autenticazione basata su password per il servizio OpenSSH.

### Configurazione del Server OpenSSH:

Puoi configurare il demone sshd, che fornisce il servizio OpenSSH, modificando il file `/etc/ssh/sshd_config`.

```bash
# Comando utile per visualizzare righe non commentate
grep ^[^#] /etc/ssh/sshd_config
```

Per rafforzare la sicurezza del sistema, potresti voler impedire il login remoto diretto all'account root e proibire l'autenticazione basata su password, preferendo quella con chiave privata SSH.

### Proibire il Login come Superuser:

È buona pratica proibire il login diretto come utente root da sistemi remoti. I rischi includono la semplicità per un attaccante di dover indovinare solo la password e il potenziale danno massimo in caso di compromissione dell'utente root. Da un punto di vista di auditing, è più difficile tracciare chi ha effettuato modifiche come utente root.

**IMPORTANTE:**&#x20;

Dalla versione 9 di Red Hat Enterprise Linux, il parametro `PermitRootLogin` è impostato di default su `prohibit-password`, che impone l'uso dell'autenticazione basata su chiavi anziché su password per l'accesso come utente root.

Per impedire il login come utente root, impostare `PermitRootLogin` su `no` nel file `/etc/ssh/sshd_config`, o su `without-password` per permettere l'autenticazione con chiave privata ma non con password.

Dopo ogni modifica, è necessario ricaricare il servizio sshd:

```sh
[root@host ~] systemctl reload sshd
```

### Proibire l'Autenticazione basata su Password per SSH

L'autenticazione solo con chiave privata presenta vantaggi, come la protezione da attacchi di forza bruta e la maggiore sicurezza fornita dalle passphrase protette. Utilizzando `ssh-agent`, la passphrase viene esposta meno spesso, rendendo più comodo il login.

Il server OpenSSH utilizza il parametro `PasswordAuthentication` nel file `/etc/ssh/sshd_config` per controllare se gli utenti possono usare l'autenticazione basata su password.

Impostare `PasswordAuthentication` su `no` per prevenire l'uso delle password.

### IMPORTANTE:

Se si disabilita l'autenticazione basata su password per ssh, assicurarsi che il file `~/.ssh/authorized_keys` dell'utente sul server remoto contenga la sua chiave pubblica, permettendogli così di effettuare l'accesso.

### EXTRA

Corso per gestione atuomatica SSH di RedHat - IDM - `RH362`.
