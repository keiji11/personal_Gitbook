# Conserva il journal di sistema

## Configurazione dello Storage Journal di Sistema

### Directory di Memorizzazione

* **Predefinita:** `/run/log` (volatile)
* **Persistente:** `/var/log/journal` (creata se non esiste)

### Parametro `Storage` nel file `/etc/systemd/journald.conf`

* **persistent:** Memorizza in `/var/log/journal`.
* **volatile:** Memorizza in `/run/log/journal`.
* **auto:** Usa `/var/log/journal` se esiste; altrimenti `/run/log/journal`.
* **none:** Non memorizza log, solo inoltro.

### Vantaggi e Limiti

* **Persistenti:** Accesso immediato ai log storici.
* **Meccanismo di Rotazione:** Mensile
* **Limiti:**
  * Max 10% del file system
  * Min 15% spazio libero

### Configurazione Limiti

Modificabile in `/etc/systemd/journald.conf`.

### Note

* I limiti correnti vengono loggati all’avvio di `systemd-journald`.

## Configurare Journal di sistema persistenti al riavvio

* Creare la directory `/var/log/journal` :&#x20;

```
[root@host ~]# mkdir /var/log/journal
```

* Settare il parametro `Storage` a `persistent` nel file `/etc/systemd/journald.conf` :&#x20;

```vim
[Journal]
Storage=persistent
...output omitted...
```

* Riavviare il servizio `systemd-journald` :&#x20;

```
[root@host ~]# systemctl restart systemd-journald
```

* Si creerá una sottodirectory in `/var/log/journal`  :&#x20;

```bash
[root@host ~]# ls /var/log/journal
4ec03abd2f7b40118b1b357f479b3112
[root@host ~]# ls /var/log/journal/4ec03abd2f7b40118b1b357f479b3112
system.journal  user-1000.journal
```

### Elenco eventi nei riavvii di sistema

```bash
[root@host ~]# journalctl --list--boots
  -6 27de... Wed 2022-04-13 20:04:32 EDT—Wed 2022-04-13 21:09:36 EDT
  -5 6a18... Tue 2022-04-26 08:32:22 EDT—Thu 2022-04-28 16:02:33 EDT
  -4 e2d7... Thu 2022-04-28 16:02:46 EDT—Fri 2022-05-06 20:59:29 EDT
  -3 45c3... Sat 2022-05-07 11:19:47 EDT—Sat 2022-05-07 11:53:32 EDT
  -2 dfae... Sat 2022-05-07 13:11:13 EDT—Sat 2022-05-07 13:27:26 EDT
  -1 e754... Sat 2022-05-07 13:58:08 EDT—Sat 2022-05-07 14:10:53 EDT
   0 ee2c... Mon 2022-05-09 09:56:45 EDT—Mon 2022-05-09 12:57:21 EDT
```

### Visualizza evento di riavvio sistema attuale e passati

```bash
# Evento avvio attuale
[root@host ~]# journalctl -b
...output omitted...

# Evento avvio precedente:
[root@host ~]# journalctl -b -1
...output omitted...
```
