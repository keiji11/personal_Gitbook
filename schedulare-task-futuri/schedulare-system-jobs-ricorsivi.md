# Schedulare system jobs ricorsivi

## _System jobs_ ricorsivi

Vanno inseriti nel file `/etc/crontab` :&#x20;

```vim
SHELL=/bin/bash
PATH=/sbin:/bin:/usr/sbin:/usr/bin
MAILTO=root

# For details see man 4 crontabs

# Example of job definition:
# .---------------- minute (0 - 59)
# |  .------------- hour (0 - 23)
# |  |  .---------- day of month (1 - 31)
# |  |  |  .------- month (1 - 12) OR jan,feb,mar,apr ...
# |  |  |  |  .---- day of week (0 - 6) (Sunday=0 or 7) OR sun,mon,tue,wed,thu,fri,sat
# |  |  |  |  |
# *  *  *  *  * user-name  command to be executed
```

Il file `/etc/crontab` e gli altri file nella directory `/etc/cron.d/` definiscono i _system jobs_ ricorrenti.&#x20;

Crea sempre file _crontab_ personalizzati nella directory `/etc/cron.d/` per pianificare _system jobs_ ricorrenti.  Colloca il file crontab personalizzato nella directory `/etc/cron.d` per evitare che un aggiornamento del pacchetto sovrascriva il file `/etc/crontab`.&#x20;

Il sistema _crontab_ include anche _**repository**_ per script da eseguire ogni ora, giorno, settimana e mese. Questi repository sono collocati nelle directory `/etc/cron.hourly/`, `/etc/cron.daily/`, `/etc/cron.weekly/` e `/etc/cron.monthly/`. Queste directory contengono script shell eseguibili, non file crontab.&#x20;

## Lancia comandi periodici con _Anacron_

Il comando `anacron` utilizza lo script `run-parts` per eseguire _jobs_ giornalieri, settimanali e mensili dal file di configurazione `/etc/anacrontab`. Assicura che i _jobs_ pianificati vengano eseguiti anche se il sistema è stato spento o ibernato.

**Componenti principali**

1. **Periodo in giorni**:\
   Intervallo di ripetizione (es: `@daily` = 1 giorno, `@weekly` = 7 giorni).
2. **Ritardo in minuti**:\
   Tempo di attesa prima dell'avvio del _job_.
3. **Identificatore del lavoro**:\
   Nome unico del lavoro nei log.
4. **Comando**:\
   Comando da eseguire.

**File e Directory**

* **Configurazione**:\
  `/etc/anacrontab`
* **Timestamp files**:\
  `/var/spool/anacron/` (registra l'ultima esecuzione)

**Variabili di ambiente**

* **`START_HOURS_RANGE`**:\
  Intervallo orario consentito per l'esecuzione dei _jobs_. Se non eseguito, il _job_ attende il giorno successivo.

I _jobs_ non vengono eseguiti fuori dall'intervallo specificato.

## Timer systemd

L'unità timer di `systemd` attiva un'altra unità di un tipo diverso (come un servizio), il cui nome coincide con quello dell'unità timer. L'unità timer consente l'attivazione temporizzata di altre unità. L'unità timer di `systemd` registra gli eventi timer nei _**journal di sistema**_ per facilitare il debug.

### Semplice Unit Timer

Il pacchetto `sysstat` fornisce il servizio timer `sysstat-collect.timer` per raccogliere statistiche di sistema ogni 10 minuti.

### Configurazione del file

Il file `/usr/lib/systemd/system/sysstat-collect.timer` contiene:

* `[Unit]`: Descrizione del servizio.
* `[Timer]`: Specifica `OnCalendar=*:*:00/10` per attivare ogni 10 minuti.
* `[Install]`: `WantedBy=sysstat.service`.

### Opzioni del Timer

* **OnCalendar**: Formati complessi es. `2022-04-* 12:35,37,39:16` per attivazione a tempi specifici.
* **OnUnitActiveSec**: Esempio: `15min` per attivare 15 minuti dopo l'ultima esecuzione.

### Modifiche alla Configurazione

* <mark style="color:red;">**Non modificare file in**</mark><mark style="color:red;">**&#x20;**</mark><mark style="color:red;">**`/usr/lib/systemd/system`**</mark><mark style="color:red;">**.**</mark>
* **Copia e modifica** in `/etc/systemd/system`.
*   Esegui `systemctl daemon-reload` per applicare le modifiche:

    ```bash
    [root@host ~]# systemctl daemon-reload
    ```

### Attivazione del Timer

Abilita il timer con:

```bash
[root@host ~]# systemctl enable --now <unitname>.timer
```

## Esempi:

*   Programmare un job di sistema ricorrente che genera un messaggio di log per indicare il numero di utenti attivi nel sistema.\


    Creare il file di script `/etc/cron.daily/usercount` con il seguente contenuto:

    ```bash
    #!/bin/bash
    USERCOUNT=$(w -h | wc -l)
    logger "Attualmente ci sono ${USERCOUNT} utenti attivi"
    ```

    #### Impostazioni dei Permessi

    Rendere eseguibile il file di script:

    ```bash
    [root@servera ~]# chmod +x /etc/cron.daily/usercount
    ```

    Questo job verrà eseguito quotidianamente grazie al servizio `cron`, registrando il numero di utenti attivi nel sistema.
* Installa il pacchetto `sysstat` . Il timer unit deve triggerare l'unitá di servizio ogni 10 minuti per ricavare dati da collezionare con lo script `/usr/lib64/sa/sa1` . Cambia il timer unit del file di configurazioneper collezionare i dati delle attivitá di sistema ogni 2 minuti.
  1.  Installazione  `sysstat` :

      <pre><code><strong>[root@servera ~]# dnf install sysstat
      </strong>...output omitted...
      <strong>Is this ok [y/N]: y
      </strong>...output omitted...
      Complete!
      </code></pre>
  2.  Copia il file `/usr/lib/systemd/system/sysstat-collect.timer` nel file`/﻿etc/systemd/system/sysstat-collect.timer` :

      <pre><code><strong>[root@servera ~]# cp /usr/lib/systemd/system/sysstat-collect.timer \
      </strong><strong>/etc/systemd/system/sysstat-collect.timer
      </strong></code></pre>
  3.  Modifica il file `/etc/systemd/system/sysstat-collect.timer`  Usa il comando `vim /etc/systemd/system/sysstat-collect.timer` :

      ```
      ...output omitted...
      # Activates activity collector every 2 minutes

      [Unit]
      Description=Run system activity accounting tool every 2 minutes

      [Timer]
      OnCalendar=*:00/2

      [Install]
      WantedBy=sysstat.service
      ```
  4.  Aggiorna il daemon `systemd` per le modiche al servizio:

      <pre><code><strong>[root@servera ~]# systemctl daemon-reload
      </strong></code></pre>
  5.  Attiva l'unitá `sysstat-collect.timer` :

      <pre><code><strong>[root@servera ~]# systemctl enable --now sysstat-collect.timer
      </strong>...output omitted...
      </code></pre>
  6.  Attendi fino a che non venga creato il filee `/var/log/sa` .

      Il comando `while` , `ls /var/log/sa | wc -l` ritorna `0` se il file non esiste, o ritorna `1` se il file esiste. Il comando `while` si ferma per un secondo se il file non esiste ed esce se il file é presente:

      <pre><code><strong>[root@servera ~]# while [ $(ls /var/log/sa | wc -l) -eq 0 ];  \
      </strong><strong>do sleep 1s; done
      </strong></code></pre>
