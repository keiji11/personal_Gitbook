# Gestione stack storage

## Panoramica sul Logical Volume Manager

### Logical Volume Management (LVM)

LVM è un sistema che permette di creare volumi di archiviazione logici al di sopra dell'archiviazione fisica, offrendo maggiore flessibilità. Consente di modificare dimensioni dei volumi senza interrompere servizi.

#### Componenti Principali

* **Dispositivi Fisici**: Comprendono partizioni disco, interi dischi, RAID o dischi SAN. Devono essere inizializzati come volumi fisici LVM.
* **Volumi Fisici (PVs)**: Rappresentano il livello di base e sono segmentati in Estensioni Fisiche (PEs).
* **Gruppi di Volumi (VGs)**: Costituiti da uno o più PVs, equivalgono a un disco intero e possono contenere volumi logici e spazio inutilizzato.
* **Volumi Logici (LVs)**: Creati da Estensioni Logiche (LEs) che mappano a PEs, vengono usati da applicazioni e sistemi operativi per l'archiviazione.

### LVM Workflow

1. Determinare i dispositivi fisici utilizzati per creare volumi fisici e inizializzare questi dispositivi come volumi fisici LVM.&#x20;
2. Creare un gruppo di volumi da più volumi fisici.&#x20;
3. Creare volumi logici dallo spazio disponibile nel gruppo di volumi.&#x20;
4. Formattare il volume logico con un file system e montarlo, attivarlo come spazio di swap, o passare il volume grezzo a un database o un server di storage per strutture avanzate.

<figure><img src=".gitbook/assets/image (1) (1).png" alt=""><figcaption></figcaption></figure>

## Buildare Storage LVM

La creazione di un volume logico comporta la <mark style="color:orange;">creazione di partizioni di dispositivi fisici</mark>, <mark style="color:yellow;">volumi fisici</mark> e <mark style="color:green;">gruppi di volumi</mark>. \
Dopo aver creato un LV, formattare il volume e montarlo per accedervi come spazio di archiviazione.

### Preparazione dispositivi fisici (Partizionamento)

<pre class="language-bash"><code class="lang-bash"><strong>[root@host ~]\# parted /dev/vdb mklabel gpt mkpart primary 1MiB 769MiB
</strong>...output omitted...
<strong>[root@host ~]\# parted /dev/vdb mkpart primary 770MiB 1026MiB
</strong><strong>[root@host ~]\# parted /dev/vdb set 1 lvm on
</strong><strong>[root@host ~]\# parted /dev/vdb set 2 lvm on
</strong><strong>[root@host ~]\# udevadm settle
</strong></code></pre>

### Creazione volume fisico

<mark style="color:orange;">`pvcreate`</mark>&#x20;

<pre class="language-bash"><code class="lang-bash"><strong>[root@host ~]\# pvcreate /dev/vdb1 /dev/vdb2
</strong>  Physical volume "/dev/vdb1" successfully created.
  Physical volume "/dev/vdb2" successfully created.
  Creating devices file /etc/lvm/devices/system.devices
</code></pre>

### Creazione Gruppo di Volumi (VG)

<mark style="color:orange;">`vgcreate`</mark>&#x20;

<pre class="language-bash"><code class="lang-bash"><strong>#                    NOME_GRUPPO  vol1    vol2      vol3  vol...
</strong><strong>[root@host ~]\# vgcreate vg01 /dev/vdb1 /dev/vdb2
</strong>  Volume group "vg01" successfully created
</code></pre>

### Creazione Volume Logico (LV)

<mark style="color:orange;">`lvcreate`</mark>&#x20;

<pre class="language-bash"><code class="lang-bash"><strong>#                       NOME_LV   SIZE   NOME_GRUPPO
</strong><strong>[root@host ~]\# lvcreate -n lv01 -L 300M vg01
</strong>  Logical volume "lv01" created.
</code></pre>

### Creazione LV tramite Deduplicazione e Compressione

RHEL 9 utilizza un'implementazione <mark style="color:orange;">LVM Virtual Data Optimizer (VDO</mark>) per la gestione dei volumi VDO.

Il VDO offre deduplicazione a livello di blocco in linea, compressione e thin provisioning per lo storage.\
Un LVM VDO è composto da <mark style="color:yellow;">due volumi logici:</mark>

* _<mark style="background-color:orange;">**VDO pool LV**</mark>**&#x20;=**_ questo LV archivia, deduplica, comprime i dati e imposta la dimensione del volume VDO supportato dal dispositivo fisico. Il VDO viene deduplicato e comprime separatamente ogni LV VDO, poiché ogni LV del pool VDO può contenere solo un LV VDO.
* _<mark style="background-color:orange;">**VDO LV**</mark>_ = un dispositivo virtuale viene fornito sul pool LV VDO e imposta la dimensione logica del volume VDO per archiviare i dati prima che avvengano la deduplicazione e la compressione.

Per utilizzare la deduplicazione e la compressione VDO, installa i pacchetti `vdo` e `kmod-kvdo` :&#x20;

<mark style="color:orange;">`lvcreate --type vdo`</mark>&#x20;

<pre class="language-bash"><code class="lang-bash"><strong>[root@host ~]\# lvcreate --type vdo --name vdo-lv01 --size 5G vg01
</strong>    Logical blocks defaulted to 523108 blocks.
    The VDO volume can address 2 GB in 1 data slab.
    It can grow to address at most 16 TB of physical storage in 8192 slabs.
    If a larger maximum size might be needed, use bigger slabs.
  Logical volume "vdo-lv01" created.
</code></pre>

### Creazione di un filesystem su un volume logico (LV)

1.  creazione fs su un LV :&#x20;

    <pre class="language-bash"><code class="lang-bash"><strong>#                           /dev/vgname/lvname
    </strong><strong>#                           /dev/mapper/vgname-lvname
    </strong><strong>[root@host ~]\# mkfs -t xfs /dev/vg01/vdo-lv01
    </strong>...output omitted...
    </code></pre>
2.  creazione mount point :&#x20;

    <pre><code><strong>[root@host ~]# mkdir /mnt/data
    </strong></code></pre>
3.  rendere il _fs persistente :_&#x20;

    ```bash
    /dev/vg01/vdo-lv01 /mnt/data xfs defaults 0 0
    ```
4.  montare il volume :&#x20;

    <pre class="language-bash"><code class="lang-bash"><strong>[root@host ~]\# mount /mnt/data/
    </strong></code></pre>

## Visualizzare lo status dei componenti LVM

### Info Volumi Fisici `pvdisplay`&#x20;

<pre><code><strong>[root@host ~]# pvdisplay /dev/vdb1
</strong>  --- Physical volume ---
  PV Name               /dev/vdb1
  VG Name               vg01
  PV Size               731.98 MiB / not usable 3.98 MiB
  Allocatable           yes
  PE Size               4.00 MiB
  Total PE              182
  Free PE               107
  Allocated PE          75
  PV UUID               zP0gD9-NxTV-Qtoi-yfQD-TGpL-0Yj0-wExh2N
</code></pre>

### Info Gruppo dei Volumi `vgdisplay`&#x20;

<pre class="language-bash"><code class="lang-bash"><strong>[root@host ~]\# vgdisplay vg01
</strong>  --- Volume group ---
  VG Name               vg01
  System ID
  Format                lvm2
  Metadata Areas        2
  Metadata Sequence No  2
  VG Access             read/write
  VG Status             resizable
  MAX LV                0
  Cur LV                1
  Open LV               1
  Max PV                0
  Cur PV                2
  Act PV                2
  VG Size               1012.00 MiB
  PE Size               4.00 MiB
  Total PE              253
  Alloc PE / Size       75 / 300.00 MiB
  Free  PE / Size       178 / 712.00 MiB
  VG UUID               jK5M1M-Yvlk-kxU2-bxmS-dNjQ-Bs3L-DRlJNc
</code></pre>

### Info Volumi Logici `lvdisplay`&#x20;

<pre class="language-bash"><code class="lang-bash"><strong>[root@host ~]\ lvdisplay /dev/vg01/lv01
</strong>  --- Logical volume ---
  LV Path                /dev/vg01/lv01
  LV Name                lv01
  VG Name                vg01
  LV UUID                FVmNel-u25R-dt3p-C5L6-VP2w-QRNP-scqrbq
  LV Write Access        read/write
  LV Creation host, time servera.lab.example.com, 2022-04-07 10:45:34 -0400
  LV Status              available
  # open                 1
  LV Size                300.00 MiB
  Current LE             75
  Segments               1
  Allocation             inherit
  Read ahead sectors     auto
  - currently set to     8192
  Block device           253:0
</code></pre>

## Estendere e ridurre LVM Storage

### Estendere la dimensione dei VG (`parted` + `vgextend` )

<pre class="language-bash"><code class="lang-bash"><strong>[root@host ~]\# parted /dev/vdb mkpart primary 1072MiB 1648MiB
</strong>...output omitted...
<strong>[root@host ~]\# parted /dev/vdb set 3 lvm on
</strong>...output omitted...
<strong>[root@host ~]\# udevadm settle
</strong><strong>[root@host ~]\# pvcreate /dev/vdb3
</strong>  Physical volume "/dev/vdb3" successfully created.
</code></pre>

Il comando `vgextend` aggiunge il nuovo PV al VG.&#x20;

<pre class="language-bash"><code class="lang-bash"><strong># Estendiamo vg01 alla dimensione di /dev/vdb3
</strong><strong>[root@host ~]\# vgextend vg01 /dev/vdb3
</strong>  Volume group "vg01" successfully extended
</code></pre>

### Estendere la dimensione dei LV `lvextend`&#x20;

<pre class="language-bash"><code class="lang-bash"><strong>[root@host ~]\# lvextend -L +500M /dev/vg01/lv01
</strong>  Size of logical volume vg01/lv01 changed from 300.00 MiB (75 extents) to 800.00 MiB (200 extents).
  Logical volume vg01/lv01 successfully resized.
</code></pre>

### Estendere un fs di tipo XFS alla dimensione intera del volume LV

`xfs_growfs`&#x20;

<pre class="language-bash"><code class="lang-bash"><strong># /mnt/data deve essere giá montato
</strong><strong>[root@host ~]\# xfs_growfs /mnt/data/
</strong>...output omitted...
data blocks changed from 76800 to 204800
</code></pre>

### Estendere un fs di tipo EXT4 alla dimensione intera del volume LV

`resize2fs`&#x20;

<pre class="language-bash"><code class="lang-bash"><strong>[root@host ~]\# resize2fs /dev/vg01/lv01
</strong>resize2fs 1.46.5 (30-Dec-2021)
Resizing the filesystem on /dev/vg01/lv01 to 256000 (4k) blocks.
The filesystem on /dev/vg01/lv01 is now 256000 (4k) blocks long.
</code></pre>

Il comando `resize2fs` ridimensiona i file system ext2, ext3 o ext4 e prende il nome del dispositivo come argomento.&#x20;

Il comando `xfs_growfs` supporta solo il ridimensionamento online, mentre il comando `resize2fs` supporta sia il ridimensionamento online che offline.&#x20;

Sebbene si possa ridimensionare un file system `ext4` sia verso l'alto che verso il basso, un file system `xfs` può essere ridimensionato solo verso l'alto.

### Estendere lo spazio di SWAP LV

I volumi logici utilizzati come spazio di swap devono essere offline per essere estesi.

&#x20;`swapoff` disattivare lo spazio di swap su LV.

<pre class="language-bash"><code class="lang-bash"><strong>[root@host ~]\# swapoff -v /dev/vg01/swap
</strong>swapoff /dev/vg01/swap
</code></pre>

`lvextend`

<pre class="language-bash"><code class="lang-bash"><strong>[root@host ~]\# lvextend -L +300M /dev/vg01/swap
</strong>  Size of logical volume vg01/swap changed from 500.00 MiB (125 extents) to 800.00 MiB (200 extents).
  Logical volume vg01/swap successfully resized.
</code></pre>

`mkswap` formatta LV come spazio di swap.

<pre class="language-bash"><code class="lang-bash"><strong>[root@host ~]\# mkswap /dev/vg01/swap
</strong>mkswap: /dev/vg01/swap: warning: wiping old swap signature.
Setting up swapspace version 1, size = 800 MiB (838856704 bytes)
no label, UUID=25b4d602-6180-4b1c-974e-7f40634ad660
</code></pre>

`swapon` attivare lo spazio di swap su LV.

<pre class="language-bash"><code class="lang-bash"><strong>[root@host ~]\# swapon /dev/vg01/swap
</strong></code></pre>

### Riduzione Storage VG

Per ridurre un VG, è necessario rimuovere i PV non utilizzati.

&#x20;\
`pvmove` sposta i dati da un PV a un altro con extenti liberi nello stesso VG.&#x20;

<pre class="language-bash"><code class="lang-bash"><strong>[root@host ~]\# pvmove -A y /dev/vdb3
</strong></code></pre>

È possibile continuare a usare l'LV mentre si riduce il VG. \
L'opzione `-A` del comando `pvmove` esegue automaticamente il backup dei metadati del VG utilizzando `vgcfgbackup`.

Rimozione PV da un VG:

<pre class="language-bash"><code class="lang-bash"><strong>[root@host ~]\# vgreduce vg01 /dev/vdb3
</strong>  Removed "/dev/vdb3" from volume group "vg01"
</code></pre>

## Rimozione storage LVM

### Preparazione file-system

Fare un backup, perché la rimozione di un LV comporta la cancellazione di tutti i dati, e smontare il volume:

<pre class="language-bash"><code class="lang-bash"><strong>[root@host ~]\# umount /mnt/data
</strong></code></pre>

### Rimozione LV

<mark style="color:orange;">`lvremove`</mark>` `_`DEVICE-NAME`_&#x20;

<pre class="language-bash"><code class="lang-bash"><strong>[root@host ~]\# lvremove /dev/vg01/lv01
</strong><strong>Do you really want to remove active logical volume vg01/lv01? [y/n]: y
</strong>  Logical volume "lv01" successfully removed.
</code></pre>

### Rimozione VG

<mark style="color:orange;">`vgremove`</mark> _`VG-NAME`_&#x20;

<pre><code><strong>[root@host ~]# vgremove vg01
</strong>  Volume group "vg01" successfully removed
</code></pre>

### Rimozione PV

<mark style="color:orange;">`pvremove`</mark>` `_`PV-NAME`_&#x20;

<pre class="language-bash"><code class="lang-bash"><strong>[root@host ~]\# pvremove /dev/vdb1 /dev/vdb2
</strong>  Labels on physical volume "/dev/vdb1" successfully wiped.
  Labels on physical volume "/dev/vdb2" successfully wiped.
</code></pre>

