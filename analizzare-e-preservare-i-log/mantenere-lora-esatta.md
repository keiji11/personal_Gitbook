# Mantenere l'ora esatta

## Gestire l'orologio locale e i fusi orari

La sincronizzazione dell'orario di sistema è fondamentale per l'analisi dei file di log su più sistemi.&#x20;

Inoltre, alcuni servizi potrebbero richiedere la sincronizzazione dell'orario per funzionare correttamente.&#x20;

Le macchine utilizzano il _**Network Time Protocol (NTP)**_ per fornire e ottenere informazioni temporali corrette su internet. \
Un'altra opzione è sincronizzarsi con un orologio hardware di alta qualità per fornire orario preciso ai client locali.

## Comando `timedatectl`&#x20;

Mostra una panoramica delle impostazioni di sistema relative all'ora corrente, tra cui l'ora corrente, il fuso orario e le impostazioni di sincronizzazione NTP del sistema:

```bash
[user@host ~]$ timedatectl
               Local time: Wed 2022-03-16 05:53:05 EDT
           Universal time: Wed 2022-03-16 09:53:05 UTC
                 RTC time: Wed 2022-03-16 09:53:05
                Time zone: America/New_York (EDT, -0400)
System clock synchronized: yes
              NTP service: active
          RTC in local TZ: no
```

### Lista DB Time-Zone con l'opzione `list-timezones`&#x20;

```
[user@host ~]$ timedatectl list-timezones
Africa/Abidjan
Africa/Accra
...
```

### Modifica Time-Zone

L'utente `root` può modificare le impostazioni di sistema per aggiornare il fuso orario corrente utilizzando l'opzione `set-timezone` del comando `timedatectl` :&#x20;

```bash
[root@host ~]# timedatectl set-timezone America/Phoenix
[root@host ~]# timedatectl
               Local time: Wed 2022-03-16 03:05:55 MST
           Universal time: Wed 2022-03-16 10:05:55 UTC
                 RTC time: Wed 2022-03-16 10:05:55
                Time zone: America/Phoenix (MST, -0700)
System clock synchronized: yes
              NTP service: active
          RTC in local TZ: no
# set UTC (Coordinated Universal Time),ossia il timezone del sistema corrente
[root@host ~]# timedatectl set-timezone UTC
...
```

### Opzione `set-time`&#x20;

```bash
# Modifica il tempo di sistema corrente
[root@host ~]# timedatectl set-time 9:00:00
[root@host ~]# timedatectl
               Local time: Fri 2019-04-05 09:00:27 MST
               ...
```

### Opzione `set-ntp`&#x20;

```bash
# Abilita o disabilita la sincronizzazione NTP per la regolazione automatica dell'ora.
[root@host ~]# timedatectl set-ntp false
```

## Configurazione e Monitoraggio servizio `chronyd`&#x20;

Il servizio `chronyd` sincronizza l'orologio _Real-Time Clock (RTC)_ locale, solitamente impreciso, con i server NTP configurati. In assenza di connessione di rete, calcola la deriva dell'RTC e la registra nel file specificato dal valore `driftfile` nel file di configurazione `/etc/chrony.conf`.

**Caratteristiche Principali**

* **Server Predefiniti:** Per impostazione predefinita, utilizza i server dal progetto _**NTP Pool.**_
* **Rete Isolata:** È possibile modificare i server NTP per reti isolate.

**Stratum NTP**

* **Riferimento Stratum 0:** Risorsa di tempo di riferimento di alta qualità.
* **Stratum 1:** Server direttamente collegati alla risorsa di riferimento.
* **Stratum 2 e Oltre:** Macchine che sincronizzano il tempo dai server NTP precedenti.

**Configurazione `chronyd`**

* **Categorie:** Server e peer.
* **Server:** Un livello sopra rispetto al server NTP locale.
* **Peer:** Stesso livello stratum.
* **Opzione Raccomandata:** Utilizzare `iburst` per misure rapide iniziali.

**Comando di Riferimento**

* Per maggiori dettagli sulle opzioni: `man 5 chrony.conf`.

### Esempio impostazione del server NTP

Nel file `/etc/chrony.conf` inserisco :&#x20;

```vim
# Use public servers from the pool.ntp.org project.
...output omitted...
server classroom.example.com iburst
...output omitted...
```

Restartare il servizio :&#x20;

```bash
[root@host ~]# systemctl restart chronyd
```

### Comando `chronyc sources -v` :&#x20;

```bash
[root@host ~]# chronyc sources -v

  .-- Source mode  '^' = server, '=' = peer, '#' = local clock.
 / .- Source state '*' = current best, '+' = combined, '-' = not combined,
| /             'x' = may be in error, '~' = too variable, '?' = unusable.
||                                                 .- xxxx [ yyyy ] +/- zzzz
||      Reachability register (octal) -.           |  xxxx = adjusted offset,
||      Log2(Polling interval) --.      |          |  yyyy = measured offset,
||                                \     |          |  zzzz = estimated error.
||                                 |    |           \
MS Name/IP address         Stratum Poll Reach LastRx Last sample
===============================================================================
^* 172.25.254.254                3   6    17    26  +2957ns[+2244ns] +/-   25ms
```
