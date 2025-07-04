# Rivedere le voci del registro di sistema

## Cercare eventi nel journal di sistema

### Cos'è `systemd-journald`?

* Memorizza log in un file _journal_
* File binario strutturato e indicizzato

### Contenuto del Journal

* Include informazioni extra sugli eventi
* Dettagli di syslog:
  * Priorità del messaggio
  * Facility

### Directory di Log

* Default: `/run/log` (Memoria volatile)
* I contenuti vanno persi allo spegnimento della macchina

## Uso di `journalctl`

* Recupera e visualizza messaggi di log
* Accesso completo per root
* Accesso limitato per utenti normali

#### Note

* Directory persistente configurabile
* Opzioni di ricerca avanzata disponibile tramite `journalctl`

<pre class="language-bash"><code class="lang-bash"><strong>[root@host ~]# journalctl
</strong>...output omitted...
Mar 15 04:42:16 host.lab.example.com systemd[2127]: Listening on PipeWire Multimedia System Socket.
Mar 15 04:42:16 host.lab.example.com systemd[2127]: Starting Create User's Volatile Files and Directories...
Mar 15 04:42:16 host.lab.example.com systemd[2127]: Listening on D-Bus User Message Bus Socket.
Mar 15 04:42:16 host.lab.example.com systemd[2127]: Reached target Sockets.
Mar 15 04:42:16 host.lab.example.com systemd[2127]: Finished Create User's Volatile Files and Directories.
Mar 15 04:42:16 host.lab.example.com systemd[2127]: Reached target Basic System.
Mar 15 04:42:16 host.lab.example.com systemd[1]: Started User Manager for UID 0.
Mar 15 04:42:16 host.lab.example.com systemd[2127]: Reached target Main User Target.
Mar 15 04:42:16 host.lab.example.com systemd[2127]: Startup finished in 90ms.
Mar 15 04:42:16 host.lab.example.com systemd[1]: Started Session 6 of User root.
Mar 15 04:42:16 host.lab.example.com sshd[2110]: pam_unix(sshd:session): session opened for user root(uid=0) by (uid=0)
Mar 15 04:42:17 host.lab.example.com systemd[1]: Starting Hostname Service...
Mar 15 04:42:17 host.lab.example.com systemd[1]: Started Hostname Service.
<strong>lines 1951-2000/2000 (END) q 
</strong></code></pre>

**`journalctl`**: Comando utilizzato per visualizzare i log di sistema.

* **Priorità messaggi**:
  * `notice` o `warning`: Testo **grassetto**.
  * `error` o superiore: Testo <mark style="color:red;">rosso</mark>.
* **Uso efficace**:
  * Limitare le ricerche per visualizzare solo le uscite pertinenti.

### Opzione `-n`

* Mostra ultime 10 voci di log per default.
* Personalizzabile con un argomento per specificare il numero di voci.
  * Esempio: Ultime 5 voci: `journalctl -n 5`.

```bash
[root@host ~]# journalctl -n 5
Mar 15 04:42:17 host.lab.example.com systemd[1]: Started Hostname Service.
Mar 15 04:42:47 host.lab.example.com systemd[1]: systemd-hostnamed.service: Deactivated successfully.
Mar 15 04:47:33 host.lab.example.com systemd[2127]: Created slice User Background Tasks Slice.
Mar 15 04:47:33 host.lab.example.com systemd[2127]: Starting Cleanup of User's Temporary Files and Directories...
Mar 15 04:47:33 host.lab.example.com systemd[2127]: Finished Cleanup of User's Temporary Files and Directories.
```

### Opzione `-f`

Mostra le ultime 10 righe del system journal ed attende in fg, `CTRL+C` per uscire:

```bash
[root@host ~]# journalctl -f
Mar 15 04:47:33 host.lab.example.com systemd[2127]: Finished Cleanup of User's Temporary Files and Directories.
Mar 15 05:01:01 host.lab.example.com CROND[2197]: (root) CMD (run-parts /etc/cron.hourly)
Mar 15 05:01:01 host.lab.example.com run-parts[2200]: (/etc/cron.hourly) starting 0anacron
Mar 15 05:01:01 host.lab.example.com anacron[2208]: Anacron started on 2022-03-15
Mar 15 05:01:01 host.lab.example.com anacron[2208]: Will run job `cron.daily' in 29 min.
Mar 15 05:01:01 host.lab.example.com anacron[2208]: Will run job `cron.weekly' in 49 min.
Mar 15 05:01:01 host.lab.example.com anacron[2208]: Will run job `cron.monthly' in 69 min.
Mar 15 05:01:01 host.lab.example.com anacron[2208]: Jobs will be executed sequentially
Mar 15 05:01:01 host.lab.example.com run-parts[2210]: (/etc/cron.hourly) finished 0anacron
Mar 15 05:01:01 host.lab.example.com CROND[2196]: (root) CMDEND (run-parts /etc/cron.hourly)
​^C
[root@host ~]#
```

### Opzione `-p`&#x20;

Visualizza le voci di registro con un livello di priorità specificato (per nome o numero) o superiore.\
Il comando `journalctl` gestisce i livelli di priorità `debug`, `info`, `notice`, `warning`, `err`, `crit`, `alert` ed `emerg`, in ordine di priorità crescente:

```bash
[root@host ~]# journalctl -p err
Mar 15 04:22:00 host.lab.example.com pipewire-pulse[1640]: pw.conf: execvp error 'pactl': No such file or direct
Mar 15 04:22:17 host.lab.example.com kernel: Detected CPU family 6 model 13 stepping 3
Mar 15 04:22:17 host.lab.example.com kernel: Warning: Intel Processor - this hardware has not undergone testing by Red Hat and might not be certif>
Mar 15 04:22:20 host.lab.example.com smartd[669]: DEVICESCAN failed: glob(3) aborted matching pattern /dev/discs/disc*
Mar 15 04:22:20 host.lab.example.com smartd[669]: In the system's table of devices NO devices found to scan
```

### Opzione `-u`&#x20;

Specifica l'unitá `systemd` specifica:

```bash
[root@host ~]# journalctl -u sshd.service
May 15 04:30:18 host.lab.example.com systemd[1]: Starting OpenSSH server daemon...
May 15 04:30:18 host.lab.example.com sshd[1142]: Server listening on 0.0.0.0 port 22.
May 15 04:30:18 host.lab.example.com sshd[1142]: Server listening on :: port 22.
May 15 04:30:18 host.lab.example.com systemd[1]: Started OpenSSH server daemon.
May 15 04:32:03 host.lab.example.com sshd[1796]: Accepted publickey for user1 from 172.25.250.254 port 43876 ssh2: RSA SHA256:1UGy...>
May 15 04:32:03 host.lab.example.com sshd[1796]: pam_unix(sshd:session): session opened for user user1(uid=1000) by (uid=0)
May 15 04:32:26 host.lab.example.com sshd[1866]: Accepted publickey for user2 from ::1 port 36088 ssh2: RSA SHA256:M8ik...
May 15 04:32:26 host.lab.example.com sshd[1866]: pam_unix(sshd:session): session opened for user user2(uid=1001) by (uid=0)
lines 1-8/8 (END) q
```

### `--since` e `--until`

```bash
[root@host ~]# journalctl --since "2022-03-11 20:30" --until "2022-03-14 10:00"
...output omitted...
```

### Note

Consultare la pagina man `systemd.journal-fields`(7) per altri campi journal.
