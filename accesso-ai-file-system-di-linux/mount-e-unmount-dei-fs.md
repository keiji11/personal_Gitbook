# ⚙️ Mount e Unmount dei FS

### Montare manualmente i file system

Per accedere al file system di un dispositivo di archiviazione rimovibile, è necessario montarlo. Con il comando `mount` , l'utente root può montare un file system manualmente.

* Il primo argomento del comando `mount` specifica il file system da montare.&#x20;
* Il secondo argomento specifica la directory come punto di montaggio nella gerarchia del file system.

Con il comando `mount` è possibile montare il file system in uno dei seguenti modi:

* Con il _**nome del file del dispositivo**_ nella directory `/dev`.
* Con l'_**UUID**_, un identificatore universalmente unico del dispositivo.

Quindi, identificare il dispositivo da montare, assicurarsi che il punto di montaggio esista e montare il dispositivo sul punto di montaggio.

**NOTA** Se si monta un file system con il comando `mount` e poi si riavvia il sistema, il file system non viene rimontato automaticamente. Il corso Red Hat System Administration II (RH134) spiega come montare in modo persistente i file system con il file `/etc/fstab`.

### Identificare un dispositivo di blocco

Un dispositivo di archiviazione collegabile a caldo, sia esso un disco rigido (HDD) o un dispositivo a stato solido (SSD) in un server, o in alternativa un dispositivo di archiviazione USB, potrebbe essere collegato ogni volta a una porta diversa di un sistema. Usare il comando `lsblk` per elencare i dettagli di un dispositivo a blocchi specificato o di tutti i dispositivi disponibili.

```bash
[root@host ~] lsblk
NAME   MAJ:MIN RM  SIZE RO TYPE MOUNTPOINTS
vda    252:0    0   10G  0 disk
├─vda1 252:1    0    1M  0 part
├─vda2 252:2    0  200M  0 part /boot/efi
├─vda3 252:3    0  500M  0 part /boot
└─vda4 252:4    0  9.3G  0 part /
vdb    252:16   0    5G  0 disk
vdc    252:32   0    5G  0 disk
vdd    252:48   0    5G  0 disk
```

La dimensione della partizione aiuta a identificare il dispositivo quando il nome della partizione è sconosciuto. Ad esempio, considerando l'output precedente, se la dimensione della partizione identificata è di 9,3 GB, montare la partizione `/dev/vda4`.

### Montare il file system con il nome della partizione

Un identificatore stabile associato a un file system è il suo identificatore univoco universale (UUID). Questo UUID è memorizzato nel superblocco del file system e rimane lo stesso finché il file system non viene ricreato.

Il comando `lsblk -fp` elenca il percorso completo del dispositivo, gli UUID e i punti di montaggio e il tipo di file system della partizione. Il punto di montaggio è vuoto quando il file system non è montato.

```bash
[root@host ~] lsblk -fp
NAME        FSTYPE FSVER LABEL UUID                   FSAVAIL FSUSE% MOUNTPOINTS
/dev/vda
├─/dev/vda1
├─/dev/vda2 vfat   FAT16       7B77-95E7              192.3M     4% /boot/efi
├─/dev/vda3 xfs          boot  2d67e6d0-...-1f091bf1  334.9M    32% /boot
└─/dev/vda4 xfs          root  efd314d0-...-ae98f652    7.7G    18% /
/dev/vdb
/dev/vdc
/dev/vdd
```

Montare il FS tramite UUID:

```bash
[root@host ~] mount UUID="efd314d0-b56e-45db-bbb3-3f32ae98f652" /mnt/data
```

### Unmounting File Systems

Le procedure di spegnimento e riavvio del sistema smontano automaticamente tutti i file system. Tutti i dati del file system vengono scaricati sul dispositivo di archiviazione, per garantire l'integrità dei dati del file system.

ATTENZIONE

I dati del file system utilizzano la memoria cache durante il normale funzionamento. È necessario smontare i file system di un'unità rimovibile prima di scollegarla. La procedura di smontaggio esegue il lavaggio dei dati sul disco prima di rilasciare l'unità.

Il comando `umount` utilizza il punto di montaggio come argomento per smontare un file system.

```bash
[root@host ~] umount /mnt/data
```

Lo smontaggio non è possibile quando il file system montato è in uso. Affinché il comando `umount` abbia successo, tutti i processi devono interrompere l'accesso ai dati sotto il punto di montaggio.

Nell'esempio seguente, il comando `umount` fallisce perché la shell utilizza la directory `/mnt/data` come directory di lavoro corrente, generando così un messaggio di errore.

```bash
[root@host ~] cd /mnt/data
[root@host data] umount /mnt/data
umount: /mnt/data: target is busy.
```

Il comando `lsof` elenca tutti i file aperti e i processi che stanno accedendo al file system. L'elenco aiuta a identificare i processi che impediscono lo smontaggio del file system.

```bash
[root@host data] lsof /mnt/data
COMMAND  PID USER   FD   TYPE DEVICE SIZE/OFF NODE NAME
bash    1593 root  cwd    DIR 253,17        6  128 /mnt/data
lsof    2532 root  cwd    DIR 253,17       19  128 /mnt/data
lsof    2533 root  cwd    DIR 253,17       19  128 /mnt/data
```

Identificare e attendere il completamento dei processi, oppure inviare il segnale <mark style="background-color:green;">**SIGTERM**</mark> o <mark style="background-color:green;">**SIGKILL**</mark> per terminarli. In questo caso, <mark style="background-color:yellow;">è sufficiente passare a una directory di lavoro corrente esterna al punto di montaggio</mark>  :

```bash
[root@host data] cd
[root@host ~] umount /mnt/data
```

#### Comandi utili

1.  Query UUID del dispositivo `/dev/vdb1`.

    <pre><code><strong>[root@servera ~]# lsblk -fp /dev/vdb
    </strong>NAME        FSTYPE LABEL UUID                                 MOUNTPOINT
    /dev/vdb
    └─/dev/vdb1 xfs          a04c511a-b805-4ec2-981f-42d190fc9a65
    </code></pre>
2.  Montare il file system usando l' UUID sulla dir `/mnt/part1` . Usa l'UUID del `/dev/vdb1` dall' output del precedente comando.

    <pre class="language-bash"><code class="lang-bash"><strong>[root@servera ~] mount \
    </strong><strong>UUID="a04c511a-b805-4ec2-981f-42d190fc9a65" /mnt/part1
    </strong></code></pre>
3.  Verifica che il dispositivo `/dev/vdb1` é montato sulla dir  `/mnt/part1`.

    <pre class="language-bash"><code class="lang-bash"><strong>[root@servera ~] lsblk -fp /dev/vdb
    </strong>NAME        FSTYPE LABEL UUID                                 MOUNTPOINT
    /dev/vdb
    └─/dev/vdb1 xfs          a04c511a-b805-4ec2-981f-42d190fc9a65 /mnt/part1
    </code></pre>
4.  Smontare la dir `/mnt/part1` quando siamo nella dir `/mnt/part1`. Il comando `umount` fallisce nello smontare il dispositivo.

    <pre class="language-bash"><code class="lang-bash"><strong>[root@servera part1] umount /mnt/part1
    </strong>umount: /mnt/part1: target is busy.
    </code></pre>
5.  Per risolvere cambiare la dir corrente in `/root` directory.

    <pre class="language-bash"><code class="lang-bash"><strong>[root@servera part1] cd
    </strong>[root@servera ~]
    </code></pre>
6.  Ora si puó smontare la dir `/mnt/part1`.

    <pre class="language-bash"><code class="lang-bash"><strong>[root@servera ~] umount /mnt/part1
    </strong></code></pre>



### Localizzare i file nel sistema

#### Identificare i File tramite nome

Il comando `locate` cerca i file in base al nome o al percorso del file. Il comando è veloce perché cerca queste informazioni nel database <mark style="background-color:orange;">mlocate</mark>. Tuttavia, questo database non viene aggiornato in tempo reale e richiede aggiornamenti frequenti per ottenere risultati accurati. Questa caratteristica significa anche che il comando `locate` non cerca i file creati dopo l'ultimo aggiornamento del database.

Il database di localizzazione si aggiorna automaticamente ogni giorno. Tuttavia, l'utente root può lanciare il comando `updatedb` per forzare un aggiornamento immediato.

```bash
[root@host ~] updatedb 
```

Il comando `locate` limita i risultati per gli utenti non privilegiati. Per vedere il nome del file risultante, l'utente deve avere i permessi di ricerca sulla directory in cui risiede il file.&#x20;

Ad esempio, individua i file che l'utente sviluppatore può leggere e che corrispondono alla parola chiave `passwd` nel nome o nel percorso:

```bash
[developer@host ~]$ locate passwd
/etc/passwd
/etc/passwd-
/etc/pam.d/passwd
...output omitted...
```

L'esempio seguente mostra il nome del file o il percorso per una corrispondenza parziale con la query di ricerca:

```bash
[root@host ~] locate image
/etc/selinux/targeted/contexts/virtual\_image_context
/usr/bin/grub2-mkimage
/usr/lib/sysimage
...output omitted...
```

L'opzione `-i` (_case Insensitive_) del comando locate esegue una ricerca senza distinzione tra maiuscole e minuscole. Questa opzione restituisce tutte le possibili combinazioni di lettere maiuscole e minuscole corrispondenti:

```basic
[developer@host ~]$ locate -i messages
...output omitted...
/usr/share/locale/zza/LC_MESSAGES
/usr/share/makedumpfile/eppic_scripts/ap_messages_3_10_to_4_8.c
/usr/share/vim/vim82/ftplugin/msmessages.vim
...output omitted...
```

L'opzione `-n` del comando locate <mark style="background-color:orange;">limita il numero di risultati di ricerca restituiti</mark>. L'esempio seguente limita i risultati della ricerca del comando locate alle prime cinque corrispondenze:

```bash
[developer@host ~]$ locate -n 5 passwd
/etc/passwd
/etc/passwd-
/etc/pam.d/passwd
...output omitted...
```

#### Ricerca per file in real-time

Il comando `find` individua i file cercando in tempo reale nella gerarchia del file system. Questo comando è più lento ma <mark style="background-color:blue;">più preciso del comando locate</mark>. Il comando `find` cerca i file anche in base a criteri diversi dal nome del file, come i permessi del file, il tipo di file, la dimensione o l'ora di modifica.

Il comando `find` esamina i file nel file system con l'account utente che ha eseguito la ricerca. L'utente che esegue il comando `find` deve avere i permessi di lettura ed esecuzione su una directory per poterne esaminare il contenuto.

Il <mark style="background-color:yellow;">primo argomento</mark> del comando find è la <mark style="background-color:yellow;">directory da cercare</mark>. Se il comando `find` omette l'argomento della directory, inizia la ricerca nella directory corrente e cerca le corrispondenze in qualsiasi sottodirectory.

Per cercare i file in base al nome del file, utilizzare l'opzione `-name FILENAME` del comando `find` per ottenere il percorso dei file che corrispondono esattamente a FILENAME.&#x20;

Ad esempio, per cercare i file `sshd_config` nella directory root /, eseguire il seguente comando:

```bash
[root@host ~] find / -name sshd_config
/etc/ssh/sshd_config
```

<mark style="background-color:purple;">NOTA</mark> = Nel comando `find`, la parola completa opzioni utilizza un trattino singolo per le opzioni, a differenza del doppio trattino per la maggior parte degli altri comandi Linux.

I caratteri jolly sono disponibili per cercare un nome di file e per restituire tutti i risultati in caso di corrispondenza parziale. Con i caratteri jolly, è essenziale citare il nome del file, per evitare che il terminale interpreti erroneamente il carattere jolly.

Nell'esempio seguente, partendo dalla directory /, si cercano i file che terminano con l'estensione .txt:

```bash
[root@host ~] find / -name '*.txt'
...output omitted...
/usr/share/libgpg-error/errorref.txt
/usr/share/licenses/audit-libs/lgpl-2.1.txt
/usr/share/licenses/pam/gpl-2.0.txt
...output omitted...
```

Per cercare i file nella directory `/etc/` che contengono la stringa `pass`, eseguire il seguente comando:

```bash
[root@host ~] find /etc -name '*pass*'
/etc/passwd-
/etc/passwd
/etc/security/opasswd
...output omitted...
```

Per eseguire una ricerca senza distinzione tra maiuscole e minuscole di un nome di file, utilizzare il comando `find -opzione nome`, seguito dal nome del file da cercare. Per cercare i file con testo senza distinzione tra maiuscole e minuscole che corrisponde alla stringa dei messaggi nei loro nomi nella directory root /, eseguire il seguente comando:

```bash
[root@host ~] find / -iname '*messages*'
/sys/power/pm_debug_messages
/usr/lib/locale/C.utf8/LC_MESSAGES
/usr/lib/locale/C.utf8/LC_MESSAGES/SYS_LC_MESSAGES
...output omitted...
```

#### Ricerca per file basata su proprietario e/o permessi

Il comando `find` cerca i file in base alla proprietà o ai permessi. Le opzioni `-user` e `-group` del comando find effettuano la ricerca in base al nome di un utente e di un gruppo, oppure in base all'ID utente e all'ID gruppo.

Per cercare i file nella directory `/home/developer` di cui è proprietario l'utente `developer`:

```bash
[developer@host ~]$ find -user developer
.
./.bash_logout
./.bash_profile
...output omitted...
```

Per cercare i file nella directory `/home/developer` di cui è proprietario il gruppo di `developer`:

```bash
[developer@host ~]$ find -group developer
.
./.bash_logout
./.bash_profile
...output omitted...
```

Per cercare i file nella directory `/home/developer` di cui è proprietario l'ID utente (uid) `1000`:

```bash
[developer@host ~]$ find -uid 1000
.
./.bash_logout
./.bash_profile
...output omitted...
```

Per cercare i file nella directory `/home/developer` di cui è proprietario l'ID del gruppo (gid) `1000`:

```bash
[developer@host ~]$ find -gid 1000
.
./.bash_logout
./.bash_profile
...output omitted...
```

Le opzioni `-user` e `-group` del comando `find` cercano i file in cui il proprietario del file e il proprietario del gruppo sono diversi. L'esempio seguente elenca i file di cui è proprietario l'utente `root` e con il gruppo `mail`:

```bash
[root@host ~] find / -user root -group mail
/var/spool/mail
...output omitted...
```

L'opzione `-perm` del comando `find` cerca i file con un particolare set di permessi. I valori ottali definiscono i permessi con 4, 2 e 1 per lettura, scrittura ed esecuzione. I permessi sono preceduti dal segno / o - per controllare i risultati della ricerca.

(I permessi ottali preceduti dal segno / corrispondono a file in cui almeno un permesso è impostato per utente, gruppo o altro per quel set di permessi. Un file con i permessi r--r--r-- non corrisponde al permesso /222 ma al permesso rw-r--r--. Il segno - prima del permesso significa che tutte e tre le parti dei permessi devono corrispondere. Nell'esempio precedente, i file con i permessi rw-rw-rw- corrispondono. È anche possibile utilizzare l'opzione -perm del comando find con il metodo simbolico per i permessi.)

Ad esempio, i seguenti comandi corrispondono a qualsiasi file nella directory `/home` per il quale l'utente proprietario ha i permessi di lettura, scrittura ed esecuzione e i membri del gruppo proprietario hanno i permessi di lettura e scrittura, mentre altri hanno accesso in sola lettura. I due comandi sono equivalenti; il primo utilizza il metodo ottale per i permessi, mentre il secondo utilizza il metodo simbolico.

```bash
[root@host ~] find /home -perm 764
...output omitted...
[root@host ~] find /home -perm u=rwx,g=rw,o=r
...output omitted...
```

L'opzione `-ls` del comando find è comoda per la ricerca dei file in base ai permessi, perché fornisce informazioni sui file che includono i loro permessi.

```bash
[root@host ~] find /home -perm 764 -ls
 26207447   0 -rwxrw-r--   1 user  user   0 May 10 04:29 /home/user/file1
```

Per cercare i file per i quali l'utente ha almeno i permessi di scrittura e di esecuzione, il gruppo ha almeno i permessi di scrittura e gli altri hanno almeno i permessi di lettura, eseguire il seguente comando:

```bash
[root@host ~] find /home -perm -324
...output omitted...
[root@host ~] find /home -perm -u=wx,g=w,o=r
...output omitted...
```

Per cercare i file per i quali l'utente ha i permessi di lettura, o il gruppo ha almeno i permessi di lettura, o altri hanno almeno i permessi di scrittura, eseguire il seguente comando:

```bash
[root@host ~] find /home -perm /442
...output omitted...
[root@host ~] find /home -perm /u=r,g=r,o=w
...output omitted...
```

Quando viene utilizzato con i segni / o -, il valore 0 funziona come un carattere jolly, poiché indica qualsiasi permesso.

Per cercare qualsiasi file nella directory `/home/developer` per il quale altri hanno almeno l'accesso in lettura sul computer host, eseguire il seguente comando:

```bash
[developer@host ~]$ find -perm -004
...output omitted...
[developer@host ~]$ find -perm -o=r
...output omitted...
```

Per cercare tutti i file nella directory `/home/developer` in cui altri hanno il permesso di scrittura, eseguire il seguente comando:

```bash
[developer@host ~]$ find -perm -002
...output omitted...
[developer@host ~]$ find -perm -o=w
...output omitted...
```

#### Cercare file in base alla dimensione

L'opzione `-size` è seguita da un valore numerico e l'unità cerca i file che corrispondono alla dimensione specificata. Utilizzare il seguente elenco per le unità con l'opzione `-size`:

* Per i kilobyte, utilizzare l'unità k con k sempre in minuscolo.
* Per i megabyte, utilizzare l'unità M con M sempre in maiuscolo.
* Per i gigabyte, utilizzare l'unità G con G sempre in maiuscolo.

È possibile utilizzare i caratteri più + e meno - per includere i file rispettivamente più grandi e più piccoli della dimensione indicata. L'esempio seguente mostra una ricerca di file con una dimensione esatta di 10 megabyte:

```bash
[developer@host ~]$ find -size 10M
...output omitted...
```

Per cercare file di dimensioni superiori a 10 gigabyte:

```bash
[developer@host ~]$ find -size +10G
...output omitted...
```

<mark style="background-color:purple;">IMPORTANTE</mark> = L'opzione `-size` del comando find arrotonda tutto a singole unità. Ad esempio, il comando `find -size 1M` mostra i file più piccoli di 1 MB, perché arrotonda tutti i file a 1 MB.

#### Ricerca File basata sul Tempo di Modifica

L'opzione `-mmin` del comando find, seguita dall'ora in minuti, cerca tutti i file con contenuto cambiato `n` minuti fa. L'orario del file viene arrotondato per difetto e supporta valori frazionari con l'intervallo +n e -n.

Per cercare tutti i file con contenuti modificati 120 minuti fa:

```bash
[root@host ~] find / -mmin 120
...output omitted...
```

#### Ricerca File basata sul Tipo di File

L'opzione `-type` del comando find limita l'ambito di ricerca a un determinato tipo di file. Utilizzate i seguenti flag per limitare l'ambito di ricerca:

* Per i file regolari, utilizzare il flag `f`.
* Per le directory, utilizzare il flag `d`.
* Per i collegamenti soft, usare il flag `l`.
* Per i dispositivi a blocchi, usare il flag `b`.

Cerca tutte le directory nella directory `/etc`:

```bash
[root@host ~] find /etc -type d
/etc
/etc/tmpfiles.d
/etc/systemd
/etc/systemd/system
/etc/systemd/system/getty.target.wants
...output omitted...
```

L'opzione `-links` del comando find seguita da un numero cerca tutti i file con un determinato numero di collegamenti. Il numero preceduto da un modificatore + cerca i file con un numero superiore a quello indicato. Se il numero precede un modificatore -, la ricerca è limitata ai file con un numero di collegamenti più basso del numero indicato.

Cerca tutti i file regolari con più di un collegamento:

```bash
[root@host ~] find / -type f -links +1
...output omitted...
```
