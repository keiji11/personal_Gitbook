# Review log di sistema

## Eventi log di sistema

Tabella Facilities:

<table><thead><tr><th valign="top">Code</th><th valign="top">Facility</th><th width="368" valign="top">Facility description</th></tr></thead><tbody><tr><td valign="top">0</td><td valign="top">kern</td><td valign="top">Kernel messages</td></tr><tr><td valign="top">1</td><td valign="top">user</td><td valign="top">User-level messages</td></tr><tr><td valign="top">2</td><td valign="top">mail</td><td valign="top">Mail system messages</td></tr><tr><td valign="top">3</td><td valign="top">daemon</td><td valign="top">System daemon messages</td></tr><tr><td valign="top">4</td><td valign="top">auth</td><td valign="top">Authentication and security messages</td></tr><tr><td valign="top">5</td><td valign="top">syslog</td><td valign="top">Internal syslog messages</td></tr><tr><td valign="top">6</td><td valign="top">lpr</td><td valign="top">Printer messages</td></tr><tr><td valign="top">7</td><td valign="top">news</td><td valign="top">Network news messages</td></tr><tr><td valign="top">8</td><td valign="top">uucp</td><td valign="top">UUCP protocol messages</td></tr><tr><td valign="top">9</td><td valign="top">cron</td><td valign="top">Clock daemon messages</td></tr><tr><td valign="top">10</td><td valign="top">authpriv</td><td valign="top">Non-system authorization messages</td></tr><tr><td valign="top">11</td><td valign="top">ftp</td><td valign="top">FTP protocol messages</td></tr><tr><td valign="top">16-23</td><td valign="top">local0 to local7</td><td valign="top">Custom local messages</td></tr></tbody></table>

Tabella prioritá syslog&#x20;

<table><thead><tr><th valign="top">Code</th><th valign="top">Priority</th><th width="467" valign="top">Priority description</th></tr></thead><tbody><tr><td valign="top">0</td><td valign="top">emerg</td><td valign="top">System is unusable</td></tr><tr><td valign="top">1</td><td valign="top">alert</td><td valign="top">Action must be taken immediately</td></tr><tr><td valign="top">2</td><td valign="top">crit</td><td valign="top">Critical condition</td></tr><tr><td valign="top">3</td><td valign="top">err</td><td valign="top">Non-critical error condition</td></tr><tr><td valign="top">4</td><td valign="top">warning</td><td valign="top">Warning condition</td></tr><tr><td valign="top">5</td><td valign="top">notice</td><td valign="top">Normal but significant event</td></tr><tr><td valign="top">6</td><td valign="top">info</td><td valign="top">Informational event</td></tr><tr><td valign="top">7</td><td valign="top">debug</td><td valign="top">Debugging-level message</td></tr></tbody></table>

\


Il servizio `rsyslog` utilizza la facility e la priorità dei messaggi di log per determinare come gestirli. Le regole configurano questa facility e priorità nel file `/etc/rsyslog.conf` e in qualsiasi file nella directory `/etc/rsyslog.d` con estensione `.conf`. I pacchetti software possono aggiungere regole installando un file appropriati nella directory `/etc/rsyslog.d`.&#x20;

Il lato sinistro di ogni linea indica la facility e la priorità dei messaggi _syslog_ a cui la regola corrisponde. Il lato destro di ogni linea indica il path. \
Un asterisco (`*`) è un carattere jolly che corrisponde a tutti i valori. \
Per esempio, la seguente linea nel file `/etc/rsyslog.conf` registrerebbe i messaggi inviati alla facility `authpriv` a qualsiasi priorità nel file `/var/log/secure`:

```
authpriv.*                  /var/log/secure
```

A volte, i messaggi di log corrispondono a più di una regola nel file `rsyslog.conf`. In tali casi, un messaggio viene memorizzato in più di un file di log. \
La parola chiave `none` nel campo della priorità indica che nessun messaggio per la facility indicata viene memorizzato nel file specificato, per limitare i messaggi memorizzati. \
Il file `rsyslog.conf` ha un'impostazione per stampare tutti i messaggi syslog con priorità `emerg` sui terminali di tutti gli utenti connessi.

### Esempio di regole del servizio rsyslog

```vim
#### RULES ####

# Logga messaggi log del kernel sulla console.
#kern.*                                                 /dev/console

# Logga qualsiasi messaggio di livello info o piú alto eccetto mail.
# NON LOGGARE MESSAGGI authpriv!
*.info;mail.none;authpriv.none;cron.none                /var/log/messages

# Il file authpriv ha accesso limitato.
authpriv.*                                              /var/log/secure

# Logga tutti i messaggi mail in un posto solo.
mail.*                                                  -/var/log/maillog


# Logga roba cron
cron.*                                                  /var/log/cron

# Tutti leggono messaggi emerg 
*.emerg                                                 :omusrmsg:*

# Salva errori news di levello crit e superiori in un file speciale.
uucp,news.crit                                          /var/log/spooler

# Salva messaggi boot anche su boot.log
local7.*                                                /var/log/boot.log
```

#### Note

Consulta la pagina man `rsyslog.conf`(5) e l'ampia documentazione HTML disponibile in `/usr/share/doc/rsyslog/html/index.html`, fornita dal pacchetto `rsyslog-doc`.

### Rotazione file di Log

Il comando `logrotate` ruota i file di registro per evitare che occupino troppo spazio nella directory `/var/log`. \
Viene rinominato in `/var/log/messages-20220320` quando viene ruotato il 20 marzo 2022. Dopo la rotazione, viene creato un nuovo file di registro e notificato il servizio che ha scritto il file di log. Generalmente, dopo quattro settimane, il file di log più antico viene scartato per liberare spazio su disco.&#x20;

### Analizzare la voce Syslog

I messaggi di log sono in ordine di data ascendente. Il servizio `rsyslog` utilizza un formato standard per registrare le voci nei file di log. Il seguente esempio spiega l'anatomia di un messaggio di log nel file `/var/log/secure:`

```log
Mar 20 20:11:48 localhost sshd[1433]: Failed password for student from 172.25.0.10 port 59344 ssh2
```

**`Mar 20 20:11:48`** : Registra il timestamp della voce di log.\
&#xNAN;**`localhost`** : L'host che invia il messaggio di log.\
&#xNAN;**`sshd[1433]`** : Il nome del programma o processo e il numero PID che ha inviato il messaggio di log.\
&#xNAN;**`Failed password for …​`**: Il messaggio che è stato inviato.

### Monitorare eventi Log

Utilizzare il comando `tail -f /percorso/del/file`. Questo comando mostra le ultime dieci righe del file specificato e continua a mostrare le nuove righe scritte nel file. \
Ad esempio, per monitorare i tentativi di accesso falliti, esegui il comando `tail` in un terminale, e poi, in un altro terminale, esegui il comando `ssh` come utente `root` mentre un utente tenta di accedere al sistema:

Nel primo terminale, esegui il comando `tail`:

```bash
[root@host ~]# tail -f /var/log/secure
```

Nel secondo terminale, esegui il comando `ssh`:

```bash
[root@host ~]# ssh root@hosta
root@hosta's password: redhat
...output omesso...
[root@hostA ~]#
```

I messaggi del log sono visibili nel primo terminale.

```log
...output omesso...
Mar 20 09:01:13 host sshd[2712]: Accepted password for root from 172.25.254.254 port 56801 ssh2
Mar 20 09:01:13 host sshd[2712]: pam_unix(sshd:session): session opened for user root by (uid=0)
```

### Mandare messaggi syslog manualmente con `logger`

Il comando `logger` invia messaggi al servizio `rsyslog`. \
Per impostazione predefinita, il comando `logger` invia il messaggio al tipo utente con priorità `notice` (`user.notice`) a meno che non venga indicato diversamente con l'opzione `-p`.&#x20;

È utile testare qualsiasi modifica alla configurazione del servizio `rsyslog`.&#x20;

Per inviare un messaggio al servizio `rsyslog` da registrare nel file di log `/var/log/boot.log`, eseguire il seguente comando `logger`:

```sh
[root@host~]# logger -p local7.notice "Voce di log creata su host"
```

## Esempi:

* Configurare il servizio `rsyslog` su `servera` per loggare tutti i messaggi `debug` o superiori, per ogni servizio al nuovo file di log `/var/log/messages-debug` cambiando il file di configurazione `/etc/rsyslog.d/debug.conf` :&#x20;

1. &#x20;Creare il file `/etc/rsyslog.d/debug.conf` :&#x20;

```log
*.debug   /var/log/messages-debug
```

2. Riavviare il servizio `rsyslog` :&#x20;

```bash
[root@servera ~]# systemctl restart rsyslog
```

* Verifica che tutti i messaggi di log con priorità `debug` compaiano nel file di log `/var/log/messages-debug` :&#x20;

1. Scrivi:

```bash
[root@servera ~]# logger -p user.debug "Debug Message Test"
```

2. Visualizziamo fgli ultimi 10 messaggi di log su `/var/log/messages-debug` e verifichiamoche appaia il messaggio: `Debug Message Test` :&#x20;

```bash
[root@servera ~]# tail /var/log/messages-debug
Feb 13 18:22:38 servera systemd[1]: Stopping System Logging Service...
Feb 13 18:22:38 servera rsyslogd[25176]: [origin software="rsyslogd" swVersion="8.37.0-9.el8" x-pid="25176" x-info="http://www.rsyslog.com"] exiting on signal 15.
Feb 13 18:22:38 servera systemd[1]: Stopped System Logging Service.
Feb 13 18:22:38 servera systemd[1]: Starting System Logging Service...
Feb 13 18:22:38 servera rsyslogd[25410]: environment variable TZ is not set, auto correcting this to TZ=/etc/localtime  [v8.37.0-9.el8 try http://www.rsyslog.com/e/2442 ]
Feb 13 18:22:38 servera systemd[1]: Started System Logging Service.
Feb 13 18:22:38 servera rsyslogd[25410]: [origin software="rsyslogd" swVersion="8.37.0-9.el8" x-pid="25410" x-info="http://www.rsyslog.com"] start
Feb 13 18:27:58 servera root[25416]: Debug Message Test
```
