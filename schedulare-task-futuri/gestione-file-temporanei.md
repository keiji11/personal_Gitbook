# Gestione file temporanei

## Gestione dei File Temporanei in Red Hat Enterprise Linux

Le applicazioni critiche spesso usano file e directory temporanei. Mentre alcune utilizzano `/tmp` per dati transitori, altre usano directory volatili specifiche sotto `/run` che risiedono solo in memoria e si puliscono al riavvio. La gestione corretta dei file temporanei è essenziale per evitare problemi di spazio su disco.

### `systemd-tmpfiles`

In Red Hat Enterprise Linux, lo strumento `systemd-tmpfiles` gestisce file e directory temporanee in modo strutturato. Il servizio `systemd-tmpfiles-setup` si avvia all'avvio del sistema e utilizza il comando `systemd-tmpfiles --create --remove`&#x20;

per leggere le istruzioni dai file di configurazione in `/usr/lib/tmpfiles.d/*.conf`, `/run/tmpfiles.d/*.conf`, e `/etc/tmpfiles.d/*.conf`.&#x20;

Questi file specificano quali file e directory creare, cancellare o proteggere con permessi adeguati.

## Pulizia file temporanei con Systemd Timer

Per evitare che i sistemi a lungo termine riempiano i loro dischi con dati obsoleti, un'unità timer di `systemd` chiamata `systemd-tmpfiles-clean.timer` attiva a intervalli regolari l'unità `systemd-tmpfiles-clean.service`, che esegue il comando `systemd-tmpfiles --clean`. Un'unità timer di `systemd` ha una sezione `[Timer]` per indicare come avviare il servizio con lo stesso nome del timer. Utilizza il seguente comando `systemctl` per visualizzare il contenuto del file di configurazione dell'unità `systemd-tmpfiles-clean.timer`.

```bash
[user@host ~]$ systemctl cat systemd-tmpfiles-clean.timer
# /usr/lib/systemd/system/systemd-tmpfiles-clean.timer
# SPDX-License-Identifier: LGPL-2.1-or-later
#
# Questo file fa parte di systemd.
#
# systemd è software libero; puoi ridistribuirlo e/o modificarlo
# secondo i termini della GNU Lesser General Public License come pubblicato da
# la Free Software Foundation; o la versione 2.1 della Licenza, o
# (a tua scelta) qualsiasi versione successiva.

[Unit]
Description=Pulizia Quotidiana delle Directory Temporanee
Documentation=man:tmpfiles.d(5) man:systemd-tmpfiles(8)
ConditionPathExists=!/etc/initrd-release

[Timer]
OnBootSec=15min
OnUnitActiveSec=1d
```

Nella configurazione sopra, il parametro `OnBootSec=15min` indica che l'unità `systemd-tmpfiles-clean.service` viene attivata 15 minuti dopo l'avvio del sistema. Il parametro `OnUnitActiveSec=1d` indica che ogni ulteriore attivazione dell'unità `systemd-tmpfiles-clean.service` avviene 24 ore dopo l'ultima attivazione dell'unità servizio. Puoi modificare i parametri nel file di configurazione dell'unità `systemd-tmpfiles-clean.timer` per soddisfare le tue esigenze.&#x20;

Ad esempio, un valore `30min` per il parametro `OnUnitActiveSec` attiva l'unità `systemd-tmpfiles-clean.service` 30 minuti dopo l'ultima attivazione dell'unità servizio. Di conseguenza, l'unità `systemd-tmpfiles-clean.service` viene attivata ogni 30 minuti dopo che le modifiche sono riconosciute. Dopo aver modificato il file di configurazione del timer, utilizza il comando `systemctl daemon-reload` per assicurarti che il demone `systemd` carichi la nuova configurazione.

```bash
[root@host ~]# systemctl daemon-reload
```

## Pulizia file temporanei manualmente

Il comando `systemd-tmpfiles --clean` analizza gli stessi file di configurazione del comando `systemd-tmpfiles --create`, ma invece di creare file e directory, elimina tutti i file che non sono stati aperti, modificati, o cambiati più recentemente dell'età massima definita nel file di configurazione.&#x20;

La sintassi consiste nelle seguenti colonne: Tipo, Percorso, Modalità, UID, GID, Età e Argomento. Tipo si riferisce all'azione che il servizio `systemd-tmpfiles` deve eseguire; per esempio, `d` per creare una directory se non esiste, o `Z` per ripristinare ricorsivamente i contesti SELinux, le autorizzazioni dei file, e la proprietà.

Il seguente comando elimina una configurazione con spiegazioni:

<pre class="language-bash"><code class="lang-bash"><strong>#Tipo Path Perm UID GID Age Arg
</strong><strong>d /run/systemd/seats 0755 root root - 
</strong></code></pre>

Crea la directory `/run/systemd/seats` se non esiste, con l'utente `root` e il gruppo `root` come proprietari, e con permessi `rwxr-xr-x`. Se questa directory esiste, non eseguire alcuna azione. Il servizio `systemd-tmpfiles` non elimina automaticamente questa directory.

```bash
D /home/student 0700 student student 1d
```

Crea la directory `/home/student` se non esiste. Se esiste, rimuovi tutto il suo contenuto. Quando il sistema esegue il comando `systemd-tmpfiles --clean`, rimuove dalla directory tutti i file che non sono stati aperti, cambiati, o modificati per più di un giorno.

```bash
L /run/fstablink - root root - /etc/fstab
```

Crea il link simbolico `/run/fstablink`, per puntare alla directory `/etc/fstab`. Non eliminare automaticamente questa riga.

## Precedenza dei File di coconfigurazione&#x20;

#### Configurazione del servizio `systemd-tmpfiles-clean`

I file di configurazione del servizio `systemd-tmpfiles-clean` possono esistere in tre posizioni con ordine di priioritá indicato:

* `/etc/tmpfiles.d/*.conf` # 1
* `/run/tmpfiles.d/*.conf` # 2
* `/usr/lib/tmpfiles.d/*.conf` # 3 NON MODIFICARE QUI

Date queste regole di precedenza, puoi sovrascrivere le impostazioni fornite dal fornitore copiando il file rilevante nella directory `/etc/tmpfiles.d/` e poi modificarlo. Utilizzando correttamente queste posizioni di configurazione, puoi gestire le impostazioni configurate dall'amministratore da un sistema di gestione centralizzato e gli aggiornamenti del pacchetto non sovrascriveranno le tue impostazioni configurate.

**Nota:**\
Quando si testano nuove configurazioni o configurazioni modificate, applica solo i comandi di un singolo file di configurazione alla volta. Specifica il nome del singolo file di configurazione sulla riga di comando di `systemd-tmpfiles`.

## Esempi:

* Configurare il servizio `systemd-tmpfiles` affinché pulisca la directory `/tmp` dai file inutilizzati degli ultimi 5 giorni.

1. Copia il file `/usr/lib/tmpfiles.d/tmp.conf`  in `/etc/tmpfiles.d/tmp.conf` :&#x20;

```bash
cp /usr/lib/tmpfiles.d/tmp.conf /etc/tmpfiles.d/tmp.conf
```

2. Modifica questa riga in `/etc/tmpfiles.d/tmp.conf` :&#x20;

```vim
q /tmp 1777 root root 5d
```

3. Verifica che la nuova configurazione funzioni:

```bash
[root@servera ~]# systemd-tmpfiles --clean /etc/tmpfiles.d/tmp.conf
[root@servera ~]# echo $?
0
```



* Aggiungere una nuova configurazione che garantisca l'esistenza della directory `/run/momentary` e che la proprietà utente e gruppo sia impostata sull'utente `root`. I permessi ottali per la directory devono essere `0700`. La configurazione deve eliminare da questa directory tutti i file che rimangono inutilizzati negli ultimi `30 secondi` :&#x20;

1. Creare la configurazione:

```bash
[root@servera ~]# vim /etc/tmpfiles.d/momentary.conf
d /run/momentary 0700 root root 30s
```

2. Verifica che la configurazione funazioni:

```bash
[root@servera ~]# systemd-tmpfiles --create /etc/tmpfiles.d/momentary.conf
[root@servera ~]# ls -ld /run/momentary
drwx------. 2 root root 40 Apr  4 06:35 /run/momentary
```
