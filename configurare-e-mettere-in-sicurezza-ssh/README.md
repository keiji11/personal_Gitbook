# 🔐 Configurare e mettere in sicurezza SSH

## Accesso alla Linea di Comando Remota con SSH

SSH (Secure Shell) è un protocollo per comunicazioni sicure fra due sistemi, permettendo l'accesso remoto, l'esecuzione di comandi e l'automazione su reti non sicure. Utilizzato comunemente con il comando `ssh utente@host_remoto`, richiede autenticazione, spesso via password. OpenSSH, presente in Red Hat Enterprise Linux, consente sia sessioni interattive che l'esecuzione remota di specifici comandi senza avviare una shell interattiva.

### Obiettivi

* Imparare ad accedere alla linea di comando di un sistema remoto tramite SSH.
* Eseguire comandi in un sistema remoto utilizzando SSH.

Con l'impiego di SSH, gli utenti possono garantire che le loro comunicazioni con sistemi remoti siano protette e criptate, riducendo il rischio di intercettazioni o manipolazioni da parte di terzi.

Esempi di Shell Sicura (SSH)

Questo comando mostra l'output nel terminale del sistema locale.

```
[developer1@host ~]$ ssh developer2@hosta hostname
```

Il comando `ssh` sopra esegue il comando `hostname` sul sistema remoto `hosta` come l'utente `developer2`, senza accedere alla shell interattiva remota.

Per accedere al server remoto `hosta` come l'utente `developer2`, usa:

```
[developer1@host ~]$ ssh developer2@hosta
```

Questo comando ti logga sul server remoto `hosta`, richiedendo l'autenticazione con la password dell'utente `developer2`.

Per disconnettersi dal sistema remoto, usa il comando `exit`:

```
[developer1@hosta ~]$ exit
```

Per effettuare il login sul server remoto `hosta` utilizzando lo stesso nome utente del sistema locale, si può utilizzare:

```
[developer1@host ~]$ ssh hosta
```

In questo esempio, il sistema remoto chiederà l'autenticazione con la password dell'utente

#### Identificazione degli Utenti Remoti

Il comando `w` è uno strumento che fornisce un elenco degli utenti attualmente connessi al sistema. Oltre a mostrare l'ubicazione del sistema remoto, il comando rivela anche i comandi eseguiti dall'utente. Questa funzionalità è particolarmente utile per monitorare l'attività degli utenti sul sistema e identificare chi sta accedendo da remoto.

<pre><code><strong>[developer1@host ~]$ ssh developer1@hosta
</strong><strong>developer1@hosta's password: redhat
</strong><strong>[developer1@hosta ~]$ w
</strong> 16:13:38 up 36 min,  1 user,  load average: 0.00, 0.00, 0.00
USER     TTY      FROM             LOGIN@   IDLE   JCPU   PCPU WHAT
developer2   pts/0    172.25.250.10    16:13    7:30   0.01s  0.01s -bash
developer1   pts/1    172.25.250.10    16:24    3.00s  0.01s  0.00s w
</code></pre>

L'output indica che l'utente `developer2` ha effettuato l'accesso al sistema sul pseudo-terminale `0` alle `16:13` di oggi dall'host con l'indirizzo IP `172.25.250.10` ed è rimasto inattivo su un prompt dei comandi per sette minuti e trenta secondi. Inoltre, mostra che l'utente `developer1` ha effettuato l'accesso al sistema sul pseudo-terminale `1` ed è stato inattivo per tre secondi dopo aver eseguito il comando.

### Chiavi SSH degli Host

Quando un cliente utilizza il comando `ssh` per connettersi a un server SSH, il sistema verifica la presenza della chiave pubblica del server nei file locali degli host conosciuti, che possono essere situati in `/etc/ssh/ssh_known_hosts` o in `~/.ssh/known_hosts` nella directory home dell'utente. Se ha una copia della chiave, confronta questa con quella ricevuta dal server. In caso di discrepanza tra le chiavi, considera il traffico di rete compromesso e chiede all'utente di confermare se procedere con la connessione.

SSH assicura la comunicazione attraverso la crittografia a chiave pubblica. Durante la connessione di un client SSH a un server, il server invia al client una copia della sua chiave pubblica, che è cruciale per stabilire un canale di comunicazione sicuro e autenticare il sistema del client.

### Controllo Rigoroso Della Chiave Host

Accettare una nuova chiave comporta il salvataggio di una copia della chiave pubblica nel file `~/.ssh/known_hosts`, consentendo così conferme automatiche dell'identità del server nelle connessioni future. Nel caso in cui la chiave SSH del server sia cambiata rispetto a una connessione precedente, il comando `ssh` richiede una conferma per accettare la nuova chiave e procedere con il login.

Se il parametro `StrictHostKeyChecking` è impostato su `no`, `ssh` consente la connessione e aggiunge la nuova chiave al file `~/.ssh/known_hosts`. Al contrario, se è impostato su `yes`, `ssh` interrompe ogni volta la connessione SSH se le chiavi pubbliche non corrispondono.

La configurazione del parametro `StrictHostKeyChecking` può avvenire a livello utente nel file `~/.ssh/config`, a livello di sistema nel file `/etc/ssh/ssh_config`, o direttamente via comando `ssh` usando l'opzione \`-o StrictHostKeyChecking

#### NOTE

Quando si tenta di connettersi via SSH a un host remoto (`hostb`), si ottiene un messaggio che invita a verificare l'autenticità dell'host. Questo è evidente quando viene richiesto di accettare la fingerprint del key del server:

```bash
developer1@hostb ~]$ ssh hostb
[...]
The authenticity of host 'hostb (172.25.250.12)' can't be established.
ECDSA key fingerprint is SHA256:qaS0PToLrqlCO2XGklA0iY7CaP7aPKimerDoaUkv720.
Are you sure you want to continue connecting (yes/no)?
```

Per assicurare la sicurezza della connessione, si deve verificare manualmente se la fingerprint della chiave pubblica del server `hostb` (ottenuto via SSH) corrisponde a quella reale ritrovata sul server stesso. Questo può essere fatto accedendo localmente al server `hostb` e utilizzando il comando:

```bash
ssh-keygen -lf /etc/ssh/ssh_host_ecdsa_key.pub
```

Questo comando mostra la fingerprint della chiave pubblica del server, che deve corrispondere a quella mostrata durante il tentativo di connessione SSH. Se le fingerprint corrispondono, si può procedere con fiducia alla connessione.

Inoltre, è consigliato configurare il parametro `StrictHostKeyChecking` come `yes` nel file di configurazione di SSH (`~/.ssh/config` o `/etc/ssh/ssh_config`). Questo garantisce che la connessione SSH venga interrotta se le chiavi pubbliche non corrispondono, aumentando la sicurezza delle connessioni SSH.

**Gestione delle Chiavi degli Host SSH Conosciuti**

Quando ti connetti a un sistema remoto e la chiave pubblica di quel sistema non è presente nel file `/etc/ssh/ssh_known_hosts`, il client SSH cerca la chiave nel file `~/.ssh/known_hosts` dell'utente.

Una chiave pubblica del server potrebbe essere stata cambiata a causa della perdita per un guasto al disco fisso o per una sostituzione legittima. In tal caso, per effettuare l'accesso con successo a quel sistema, è necessario modificare il file `/etc/ssh/ssh_known_hosts` sostituendo la vecchia chiave pubblica con quella nuova.

Il file `/etc/ssh/ssh_known_hosts` è un file di sistema che archivia le chiavi pubbliche degli host conosciuti dal sistema. Devi creare e gestire questo file, manualmente o attraverso metodi automatizzati come l'uso di Ansible o uno script che utilizza l'utilità `ssh-keyscan`.

Le informazioni riguardo i sistemi remoti conosciuti e le loro chiavi sono conservate in uno dei seguenti luoghi:

* Il file `~/.ssh/known_hosts` nella directory home di ogni utente.
* Il file di sistema `/etc/ssh/ssh_known_hosts`.

Ogni voce della chiave host conosciuta consiste in una linea che contiene tre campi: l'elenco di nomi host e indirizzi IP che condividono la chiave pubblica, l'algoritmo di crittografia utilizzato per la chiave, e infine la chiave stessa.

<pre><code><strong>[developer1@host ~]$ cat ~/.ssh/known_hosts
</strong>server1 ssh-ed25519 AAAAC3NzaC1lZDI1NTE5AAAAIOmiLKMExRnsS1g7OTxMsOmgHuUSGQBUxHhuUGcv19uT
server1 ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAABgQC8WDOooY+rh6NPa9yhLsNQXBqcQknTL/WSd3zPvHLLd7KaC4IiEUxnwbfLBit8tRcirbQFxO20Am
...output omitted...
</code></pre>

### Risoluzione Problemi con la Chiave Host

Quando l'indirizzo IP o la chiave pubblica del sistema remoto cambia, e si tenta di connettersi nuovamente a tale sistema tramite SSH, il client SSH rileva che la voce della chiave per quel sistema nel file `~/.ssh/known_hosts` non è più valida. Un messaggio di avviso indica che l'identificazione dell'host remoto è cambiata e che è necessario modificare la voce della chiave.

```
@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@
@    WARNING: REMOTE HOST IDENTIFICATION HAS CHANGED!     @
@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@
IT IS POSSIBLE THAT SOMEONE IS DOING SOMETHING NASTY!
Someone could be eavesdropping on you right now (man-in-the-middle attack)!
It is also possible that a host key has just been changed.
The fingerprint for the ECDSA key sent by the remote host is
SHA256:hxttxb/qVi1/ycUU2wXF6mfGH++Ya7WYZv0r+tIkg4I.
Please contact your system administrator.
Add correct host key in /home/user/.ssh/known_hosts to get rid of this message.
Offending ECDSA key in /home/user/.ssh/known_hosts:12
ECDSA host key for server1.example.com has changed and you have requested strict checking.
Host key verification failed.
```

**Risolvere gli Avvertimenti delle Chiavi SSH**

Se incontri un messaggio di avvertimento relativo a una chiave SSH cambiata, è fondamentale procedere con cautela per assicurarti che la sicurezza della tua rete non sia messa a rischio. Segui questi passaggi a seconda della tua situazione.

**Chiave Modificata per Motivi Conosciuti (es. Cambio dell'Indirizzo IP)**

In sintesi, la modifica delle chiavi per motivi specifici e conosciuti è una pratica consigliata che contribuisce a rafforzare la sicurezza e l'affida

1. **Rimuovi la Vecchia Chiave**: Localizza ed elimina la vecchia chiave dal tuo file `~/.ssh/known_hosts`. Il messaggio di avviso dovrebbe specificare il numero di riga della chiave interessata.
2. **Riconnetti**: Prova a connetterti di nuovo al sistema. SSH ti chiederà di accettare la nuova chiave, aggiungendola al tuo file `known_hosts`.&#x20;

**Chiave Modificata per Motivi Sconosciuti**

* **Verifica la Chiave**: Se non sei sicuro del motivo per cui la chiave è cambiata, conferma che l'impronta digitale della nuova chiave sia legittima. Questo di solito significa contattare l'amministratore del sistema di destinazione tramite un metodo sicuro diverso da SSH (spesso definito verifica "out-of-band") per assicurarsi che il cambio di chiave sia genuino e non indicativo di una minaccia alla sicurezza.

<pre><code><strong>[developer1@host ~]$ ssh-keygen -R remotesystemname -f ~/.ssh/known_hosts
</strong># Host remotesystemname found: line 12
/home/user/.ssh/known_hosts updated.
Original contents retained as /home/user/.ssh/known_hosts.old
</code></pre>
