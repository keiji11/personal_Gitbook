# Aggiungere partizioni,file system e mounts persistenti

## Dischi partizione

Puoi usare le partizioni per dividere l'archiviazione in base a diverse esigenze, e questa divisione offre molti vantaggi:

* Limitare lo spazio disponibile per applicazioni o utenti.
* Separare file di sistema operativo e programmi dai file utente.
* Creare un'area separata per lo scambio di memoria.
* Limitare l'uso dello spazio su disco per migliorare le prestazioni degli strumenti diagnostici e delle immagini di backup.

## Schema di partizione MBR

Il _<mark style="color:orange;">Master Boot Record</mark>_ <mark style="color:orange;"></mark><mark style="color:orange;">(MBR)</mark> è lo schema di partizionamento standard per i sistemi che utilizzano il firmware BIOS. \
Questo schema supporta un _massimo di 4 partizioni primarie_. \
Nei sistemi Linux, utilizzando partizioni estese e logiche, è possibile creare fino a 15 partizioni. Con una dimensione di partizione a 32 bit, i dischi partizionati con MBR possono raggiungere una dimensione massima di 2 TiB.

_<mark style="color:yellow;">Il limite di 2 TiB per le dimensioni del disco e delle partizioni è ormai una restrizione comune e limitante.</mark>_ Di conseguenza, _<mark style="color:red;">**lo schema MBR legacy è stato sostituito dallo schema di partizionamento GUID Partition Table (GPT).**</mark>_

<figure><img src="../.gitbook/assets/image (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

## Schema di partizione GPT

Una tabella GPT fornisce un massimo di 128 partizioni. Lo schema GPT assegna 64 bit agli indirizzi dei blocchi logici, per supportare partizioni e dischi fino a otto zebibyte (ZiB) o otto miliardi di tebibyte (TiB).

GPT utilizza un _<mark style="color:yellow;">identificatore univoco globale</mark>_ <mark style="color:yellow;"></mark><mark style="color:yellow;">(GUID)</mark> per identificare ciascun disco e partizione. \
GPT rende la tabella delle partizioni ridondante, con il GPT principale all'inizio del disco e un backup secondario alla fine del disco. \
GPT utilizza un checksum per rilevare errori nell'intestazione del GPT e nella tabella delle partizioni.

<figure><img src="../.gitbook/assets/image (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

## Gestione partizioni

Il programma standard per la gestione delle partizioni da riga di comando in Red Hat Enterprise Linux è <mark style="color:orange;">`parted`</mark>. \
Puoi utilizzare l'editor di partizioni `parted` con unità di archiviazione che utilizzano sia lo schema di partizionamento MBR che quello GPT. \
Il comando `parted` richiede come primo argomento il nome del dispositivo che rappresenta l'intero dispositivo di archiviazione o disco da modificare, seguito dai sottocomandi.

<pre class="language-bash"><code class="lang-bash"><strong># Visualizza partizione /dev/vda
</strong><strong>[root@host ~]\# parted /dev/vda print
</strong>Model: Virtio Block Device (virtblk)
Disk /dev/vda: 53.7GB
Sector size (logical/physical): 512B/512B
Partition Table: msdos
Disk Flags:

Number  Start   End     Size    Type     File system  Flags
 1      1049kB  10.7GB  10.7GB  primary  xfs          boot
 2      10.7GB  53.7GB  42.9GB  primary  xfs
</code></pre>

Sessione interattiva `parted` :&#x20;

<pre class="language-bash"><code class="lang-bash"><strong>[root@host ~]\# parted /dev/vda
</strong>GNU Parted 3.4
Using /dev/vda
Welcome to GNU Parted! Type 'help' to view a list of commands.
<strong>(parted) print
</strong>Model: Virtio Block Device (virtblk)
Disk /dev/vda: 53.7GB
Sector size (logical/physical): 512B/512B
Partition Table: msdos
Disk Flags:

Number  Start   End     Size    Type     File system  Flags
 1      1049kB  10.7GB  10.7GB  primary  xfs          boot
 2      10.7GB  53.7GB  42.9GB  primary  xfs

<strong>(parted) quit
</strong></code></pre>

Per impostazione predefinita, il comando `parted` mostra le dimensioni in potenze di 10 (KB, MB, GB). Puoi modificare l'unità di misura con il parametro `unit`, che accetta i seguenti valori:&#x20;

**`s`** per settori, **`B`** per byte, **`MiB`**, **`GiB`**, o **`TiB`** (potenze di 2); **`MB`**, **`GB`**, o **`TB`** (potenze di 10).

<pre class="language-bash"><code class="lang-bash"><strong>[root@host ~]\# parted /dev/vda unit s print
</strong>Model: Virtio Block Device (virtblk)
Disk /dev/vda: 104857600s
Sector size (logical/physical): 512B/512B
Partition Table: msdos
Disk Flags:

Number  Start      End         Size       Type     File system  Flags
 1      2048s      20971486s   20969439s  primary  xfs          boot
 2      20971520s  104857535s  83886016s  primary  xfs
</code></pre>

### Scrivi la tabella di partizione su un nuovo disco

#### <mark style="color:red;">Avvertenza</mark>

L'uso del sottocomando <mark style="color:orange;">`mklabel`</mark> comporta la <mark style="color:red;">cancellazione della tabella delle partizioni esistente</mark>. \
È consigliato utilizzare questo comando solo quando si ha intenzione di riutilizzare il disco ignorando i dati attuali. \
Inoltre, se una nuova etichetta altera i confini delle partizioni, tutti i dati presenti nei file system esistenti diventeranno inaccessibili.

<pre class="language-bash"><code class="lang-bash"><strong># Va creata un'etichetta (label)
</strong><strong>[root@host ~]\# parted /dev/vdb mklabel msdos
</strong>
<strong>[root@host ~]\# parted /dev/vdb mklabel gpt
</strong></code></pre>

## Creazione partizioni MBR

<pre class="language-bash"><code class="lang-bash"><strong># Entriamo in moodalitá interattiva
</strong><strong>[root@host ~]\# parted /dev/vdb
</strong>GNU Parted 3.4
Using /dev/vdb
Welcome to GNU Parted! Type 'help' to view a list of commands.
(parted)
</code></pre>

<pre class="language-bash"><code class="lang-bash"><strong># Creazione partizione primaria o estesa (primaria in questo caso)
</strong><strong>(parted) mkpart
</strong><strong>Partition type?  primary/extended? primary
</strong></code></pre>

<mark style="color:orange;">NOTA</mark>\
Se sono necessarie più di quattro partizioni su un disco con partizione MBR, creare 3 partizioni primarie e 1 partizione estesa. \
La partizione estesa funge da contenitore all'interno del quale è possibile creare più partizioni logiche.

<pre class="language-bash"><code class="lang-bash"><strong>File system type?  [ext2]? xfs
</strong></code></pre>

per mostrare i tipi di file-system supportati, usare il seguente comando:

<pre class="language-bash"><code class="lang-bash"><strong>[root@host ~]\# parted /dev/vdb help mkpart
</strong>  ...output omitted...
  mkpart PART-TYPE [FS-TYPE] START END     make a partition

    PART-TYPE is one of: primary, logical, extended
    FS-TYPE is one of: udf, btrfs, nilfs2, ext4, ext3, ext2, f2fs, fat32, fat16,
    hfsx, hfs+, hfs, jfs, swsusp, linux-swap(v1), linux-swap(v0), ntfs,
    reiserfs, hp-ufs, sun-ufs, xfs, apfs2, apfs1, asfs, amufs5, amufs4, amufs3,
    amufs2, amufs1, amufs0, amufs, affs7, affs6, affs5, affs4, affs3, affs2,
    affs1, affs0, linux-swap, linux-swap(new), linux-swap(old)

   'mkpart' makes a partition without creating a new file system on the
    partition.  FS-TYPE may be specified to set an appropriate partition
    ID.
</code></pre>

Specificare il settore del disco da cui iniziare la nuova partizione:

<pre class="language-bash"><code class="lang-bash"><strong>Start? 2048s
</strong></code></pre>

Quando viene avviato il comando `parted`, viene recuperata la topologia del disco dal dispositivo, come la dimensione del blocco fisico del disco.&#x20;

Il comando `parted` si assicura che la posizione iniziale fornita allinei correttamente la partizione con la struttura del disco per ottimizzare le prestazioni. Se la posizione iniziale risulta in una partizione non allineata, il comando `parted` visualizza un avviso. \
Con la maggior parte dei dischi, un settore iniziale che è un _<mark style="color:orange;">multiplo di 2048</mark>_ è sicuro. \
Specifica il settore del disco dove la nuova partizione dovrebbe terminare ed esci dal comando `parted`. Puoi specificare la fine come una dimensione o come una posizione finale.

<pre class="language-bash"><code class="lang-bash"><strong>End? 1000MB
</strong><strong>(parted) quit
</strong>Information: You may need to update /etc/fstab.

[root@host ~]#
</code></pre>

Esegui il comando <mark style="color:orange;">`udevadm settle`</mark>. \
Questo comando attende che il sistema rilevi la nuova partizione e crei il file di dispositivo associato nella directory `/dev`. Il prompt ritorna una volta completato il processo:&#x20;

<pre class="language-bash"><code class="lang-bash"><strong>[root@host ~]\# udevadm settle
</strong></code></pre>

### Creazione partizione con singolo comando (metodo non interattivo):

<pre class="language-bash"><code class="lang-bash"><strong>[root@host ~]\# parted /dev/vdb mkpart primary xfs 2048s 1000MB
</strong></code></pre>

## Creazione partizioni GPT

<pre class="language-bash"><code class="lang-bash"><strong>[root@host ~]\# parted /dev/vdb
</strong>GNU Parted 3.4
Using /dev/vdb
Welcome to GNU Parted! Type 'help' to view a list of commands.
(parted)
</code></pre>

<pre class="language-bash"><code class="lang-bash"><strong>(parted) mkpart
</strong><strong>Partition name?  []? userdata
</strong></code></pre>

<pre class="language-bash"><code class="lang-bash"><strong>File system type?  [ext2]? xfs
</strong></code></pre>

<pre class="language-bash"><code class="lang-bash"><strong>Start? 2048s
</strong></code></pre>

<pre class="language-bash"><code class="lang-bash"><strong>End? 1000MB
</strong><strong>(parted) quit
</strong>Information: You may need to update /etc/fstab.

[root@host ~]#
</code></pre>

<pre class="language-bash"><code class="lang-bash"><strong>[root@host ~]\# udevadm settle
</strong></code></pre>

### Creazione partizione con singolo comando

<pre class="language-bash"><code class="lang-bash"><strong>[root@host ~]\# parted /dev/vdb mkpart userdata xfs 2048s 1000MB
</strong></code></pre>

### Cancellare partizioni

Vale sia per partizioni MBR che GPT

<pre class="language-bash"><code class="lang-bash"><strong>[root@host ~]\# parted /dev/vdb
</strong>GNU Parted 3.4
Using /dev/vdb
Welcome to GNU Parted! Type 'help' to view a list of commands.
(parted)
</code></pre>

Identificare il numero della partizione da eliminare:

<pre class="language-bash"><code class="lang-bash"><strong>(parted) print
</strong>Model: Virtio Block Device (virtblk)
Disk /dev/vdb: 5369MB
Sector size (logical/physical): 512B/512B
Partition Table: gpt
Disk Flags:

Number  Start   End     Size   File system  Name       Flags
 1      1049kB  1000MB  999MB  xfs          usersdata
</code></pre>

Elimina la partizione ed esci da `parted`. \
Il sottocomando `rm` elimina immediatamente la partizione dalla tabella delle partizioni sul disco:

<pre class="language-bash"><code class="lang-bash"><strong>(parted) rm 1
</strong><strong>(parted) quit
</strong>Information: You may need to update /etc/fstab.

[root@host ~]\#
</code></pre>

### Cancellare partizione con singolo comando:

<pre class="language-bash"><code class="lang-bash"><strong>[root@host ~]\# parted /dev/vdb rm 1
</strong></code></pre>

## Creazione file-system

Dopo aver creato un dispositivo a blocchi, il passo successivo è aggiungere un file system. \
Red Hat Enterprise Linux supporta diversi tipi di file system, e _<mark style="color:orange;">XFS è il predefinito consigliato</mark>_.&#x20;

Come utente `root`, usa il comando <mark style="color:orange;">`mkfs.xfs`</mark> per applicare un file system XFS a un dispositivo a blocchi. Per un file system ext4, usa il comando `mkfs.ext4`.

<pre class="language-bash"><code class="lang-bash"><strong>[root@host ~]\# mkfs.xfs /dev/vdb1
</strong>meta-data=/dev/vdb1              isize=512    agcount=4, agsize=60992 blks
         =                       sectsz=512   attr=2, projid32bit=1
         =                       crc=1        finobt=1, sparse=1, rmapbt=0
         =                       reflink=1    bigtime=1 inobtcount=1
data     =                       bsize=4096   blocks=243968, imaxpct=25
         =                       sunit=0      swidth=0 blks
naming   =version 2              bsize=4096   ascii-ci=0, ftype=1
log      =internal log           bsize=4096   blocks=1566, version=2
         =                       sectsz=512   sunit=0 blks, lazy-count=1
realtime =none                   extsz=4096   blocks=0, rtextents=0
</code></pre>

## Montare file-system

Dopo aver aggiunto il file system, l'ultimo passaggio consiste nel montarlo su una directory nella struttura delle directory. \
Quando si monta un file system sulla gerarchia delle directory, le utility dello spazio utente possono accedere o scrivere file sul dispositivo.

### Montare manualmente file-system

Usa il comando <mark style="color:orange;">`mount`</mark> per collegare manualmente un dispositivo a una directory di _mount point_. \
Il comando `mount` richiede un _<mark style="color:yellow;">dispositivo</mark>_ e un _<mark style="color:yellow;">punto di mount</mark>_, e può includere <mark style="color:yellow;">opzioni di montaggio</mark> del file system.\
&#x20;Le opzioni del file system personalizzano il comportamento del file system.

<pre class="language-bash"><code class="lang-bash"><strong>[root@host ~]\# mount /dev/vdb1 /mnt
</strong></code></pre>

Puoi anche usare il comando `mount` per visualizzare i file system attualmente montati, i punti di mount e le loro opzioni.

<pre class="language-bash"><code class="lang-bash"><strong>[root@host ~]\# mount | grep vdb1
</strong>/dev/vdb1 on /mnt type xfs (rw,relatime,seclabel,attr2,inode64,noquota)
</code></pre>

### Montare file-system in modo persistente

Per configurare il sistema in modo che monti automaticamente il file system durante l'avvio, aggiungi una voce nel file `/etc/fstab`. \
Questo file di configurazione elenca i file system da montare all'avvio del sistema. \
Il file `/etc/fstab` è delimitato da spazi bianchi e ogni linea contiene sei campi.

<pre class="language-bash"><code class="lang-bash"><strong>[root@host ~]\# cat /etc/fstab
</strong># DEVICE    MOUNTING POINT    File-System_TYPE    OPTIONS    DUMP    fsck_ORDER

#
# /etc/fstab
# Created by anaconda on Thu Apr 5 12:05:19 2022
#
# Accessible filesystems, by reference, are maintained under '/dev/disk/'.
# See man pages fstab(5), findfs(8), mount(8) and/or blkid(8) for more info.
#
# After editing this file, run 'systemctl daemon-reload' to update systemd
# units generated from this file.
#
UUID=a8063676-44dd-409a-b584-68be2c9f5570   /        xfs   defaults   0 0
UUID=7a20315d-ed8b-4e75-a5b6-24ff9e1f9838   /dbdata  xfs   defaults   0 0
</code></pre>

#### <mark style="color:orange;">Nota</mark>

#### <mark style="color:red;">Un'errore in</mark> <mark style="color:red;"></mark><mark style="color:red;">`/etc/fstab`</mark> <mark style="color:red;"></mark><mark style="color:red;">potrebbe rendere la macchina non avviabile</mark>.&#x20;

* Verifica la validità di una voce nel file system smontandolo manualmente.
* Usare `mount <mountpoint>` per leggere `/etc/fstab`.
* Rimontare il file system con le opzioni specificate.
* Risolvere eventuali errori di `mount` prima del riavvio.
* In alternativa, utilizzare `findmnt --verify` per controllare la usabilità delle partizioni nel file `/etc/fstab`.

<pre class="language-bash"><code class="lang-bash"><strong># Riavvio dopo la modifica
</strong><strong>[root@host ~]\# systemctl daemon-reload
</strong></code></pre>

Red Hat consiglia l'utilizzo degli UUID per montare i file system in modo persistente.\
Il nome del file del dispositivo di blocco potrebbe cambiare, ma l'UUID rimane costante nel superblocco del file system.

Utilizza il comando <mark style="color:orange;">`lsblk --fs`</mark> per eseguire la scansione dei dispositivi a blocchi collegati a una macchina e recuperare gli UUID del file system:

<pre class="language-bash"><code class="lang-bash"><strong>[root@host ~]\# lsblk --fs
</strong>NAME   FSTYPE  FSVER  LABEL    UUID            FSAVAIL FSUSE% MOUNTPOINTS
vda
├─vda1
├─vda2 xfs            boot     49dd...75fdf    312M    37%    /boot
└─vda3 xfs            root     8a90...ce0da    4.8G    48%    /
</code></pre>
