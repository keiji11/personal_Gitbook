# Gestione Storage a strati

## Stack Storage

Questa sezione mostra lo stack di storage RHEL dal basso verso l'alto e introduce ogni livello.

Questa sezione tratta anche _<mark style="color:orange;">**Stratis**</mark>_, il demone che unifica, configura e monitora i componenti dello stack di storage RHEL sottostante e fornisce una gestione automatizzata dello storage locale dalla CLI o dalla console web RHEL.

<div data-full-width="true"><figure><img src="../.gitbook/assets/image (9).png" alt=""><figcaption></figcaption></figure></div>

### Dispositivo a blocchi

I dispositivi a blocchi sono alla base dello stack di storage e offrono un protocollo stabile e coerente. Questi dispositivi includono vari tipi come i dischi ATA, SSD, e gli adattatori host bus (HBA) aziendali.

RHEL supporta anche protocolli come iSCSI, FCoE, `virtio`, SAS e NVMe. \
Un target iSCSI può essere un dispositivo fisico o logico configurato via software, consentendo l'accesso a <mark style="color:yellow;">LUN(Logical Unit Numbers)</mark>. \
FCoE consente di trasmettere frame Fibre Channel su reti Ethernet, riducendo i costi tramite il consolidamento delle reti dati e SAN (Storage Area Network).

### Multipath

Un path è una connessione tra un server e lo storage sottostante. \
<mark style="color:orange;">Device Mapper multipath (</mark><mark style="color:orange;">`dm-multipath`</mark>) è uno strumento multipath nativo di RHEL per configurare percorsi I/O ridondanti in un singolo dispositivo logico con percorsi aggregati. \
Un dispositivo logico creato utilizzando il device mapper (`dm`) appare come un dispositivo a blocchi unico nella directory `/dev/mapper/` per ogni LUN associata al sistema.

### Partizioni

Le partizioni possono occupare l'intera dimensione del dispositivo a blocchi oppure suddividere il dispositivo a blocchi per creare più partizioni. È possibile utilizzare queste partizioni per creare un file system, dispositivi LVM o direttamente per strutture di database o altro storage raw.

### RAID

Un _<mark style="color:orange;">**Redundant Array of Inexpensive Disks (RAID)**</mark>_ è una tecnologia di virtualizzazione dello storage che crea volumi logici di grandi dimensioni a partire da più componenti di dispositivi a blocchi fisici o virtuali.&#x20;

I diversi tipi di volumi RAID offrono ridondanza, miglioramento delle prestazioni o entrambi, tramite layout di mirroring o striping. LVM supporta i livelli RAID 0, 1, 4, 5, 6 e 10, utilizzando i driver del kernel Multiple Devices (`mdadm`). Senza LVM, Device Mapper RAID (`dm-raid`) fornisce un'interfaccia a `mdadm`.

### Logical Volume Manager

LVM è in grado di gestire vari tipi di dispositivi di memoria fisici o virtuali formando nuovi volumi logici. Nasconde la configurazione fisica alle applicazioni e supporta funzionalità avanzate come la crittografia e la compressione. \
L'<mark style="color:yellow;">encryption LUKS</mark> offre sicurezza extra rendendo il dispositivo invisibile senza accesso al filesystem. LVM integra deduplica e comprime VDO per ottimizzare l'uso dello spazio, consentendo di combinare LUKS con VDO sui volumi logici.

### File System o altri usi

Il livello superiore dello stack è solitamente un file system e può essere utilizzato come spazio grezzo per database o applicazioni personalizzate. \
RHEL supporta diversi tipi di file system e raccomanda XFS per la maggior parte dei casi moderni. \
XFS è necessario quando l'LVM è implementato tramite Red Hat Ceph Storage o Stratis. \


Applicazioni server di database utilizzano lo storage in modi differenti. \
Database più piccoli memorizzano le strutture in file regolari, mentre quelli più grandi utilizzano storage grezzo ignorando il caching del file system. \
Ceph Storage utilizza LVM per inizializzare i dispositivi disco.

## Stratis Storage Manager

_<mark style="color:orange;">**Stratis**</mark>_ è uno strumento di gestione dello storage locale sviluppato da Red Hat e dalla comunità Fedora. Stratis gestisce la configurazione iniziale, modifica le configurazioni di Storage e utilizza funzionalità avanzate. \
Il servizio gestisce pool di dispositivi di archiviazione fisici e crea e gestisce volumi per nuovi file system tramite _<mark style="color:yellow;">thin provisioning</mark>_. Lo spazio di archiviazione viene allocato dinamicamente dal pool man mano che il file system archivia più dati, consentendo al file system di sembrare più grande della sua allocazione fisica reale. \
Puoi creare più pool da diversi dispositivi di archiviazione e fino a <mark style="color:yellow;">2²⁴ file system per pool.</mark> \
Stratis utilizza componenti standard Linux e si basa su XFS per formattare i file system gestiti..

<div data-full-width="true"><figure><img src="../.gitbook/assets/image (10).png" alt=""><figcaption></figcaption></figure></div>

## Metodi di amministrazione _Stratis_

Per gestire i file system utilizzando Stratis, è necessario installare i pacchetti `stratis-cli` e `stratisd`. \
Il pacchetto <mark style="color:orange;">`stratis-cli`</mark> offre il comando <mark style="color:yellow;">`stratis`</mark> per inviare richieste al demone <mark style="color:yellow;">`stratisd`</mark>. \
Il pacchetto <mark style="color:orange;">`stratisd`</mark> gestisce i dispositivi, pool e file system Stratis. \
L'amministrazione di Stratis è inclusa nella console web RHEL.

### Installazione ed abilitazione _STRATIS_

<pre class="language-bash"><code class="lang-bash"><strong>[root@host ~]\# dnf install stratis-cli stratisd
</strong>...output omitted...
<strong>Is this ok [y/N]: y
</strong>...output omitted...
Complete!
<strong>[root@host ~]\# systemctl enable --now stratisd
</strong></code></pre>

### Creazione _stratis pool_ con `stratis pool create`

<pre class="language-bash"><code class="lang-bash"><strong>[root@host ~]\# stratis pool create pool1 /dev/vdb
</strong><strong>[root@host ~]\# stratis pool list
</strong>Name                  Total Physical   Properties            UUID
pool1   5 GiB / 37.63 MiB / 4.96 GiB      ~Ca,~Cr   11f6f3c5-5...
</code></pre>

### `stratis pool add-data` e `stratis blockdev list`&#x20;

<pre class="language-bash"><code class="lang-bash"><strong>[root@host ~]\# stratis pool add-data pool1 /dev/vdc
</strong><strong>[root@host ~]\# stratis blockdev list pool1
</strong>Pool Name   Device Node   Physical Size   Tier
pool1       /dev/vdb              5 GiB   Data
pool1       /dev/vdc              5 GiB   Data
</code></pre>

### Gestione _Stratis_ file system con `stratis filesystem ...`&#x20;

<pre class="language-bash"><code class="lang-bash"><strong>[root@host ~]\# stratis filesystem create pool1 fs1
</strong><strong>[root@host ~]\# stratis filesystem list
</strong>Pool Name   Name   Used      Created             Device                   UUID
pool1       fs1    546 MiB   Apr 08 2022 04:05   /dev/stratis/pool1/fs1   c7b5719...
</code></pre>

<pre class="language-bash"><code class="lang-bash"><strong>[root@host ~]\# stratis filesystem snapshot pool1 fs1 snapshot1
</strong></code></pre>

### Montare il filesystem Stratis in modo permanente

ricaviamo l'UUID che ci serve:

<pre class="language-bash"><code class="lang-bash"><strong>[root@host ~]\# lsblk --output=UUID /dev/stratis/pool1/fs1
</strong>UUID
c7b57190-8fba-463e-8ec8-29c80703d45e
</code></pre>

<pre class="language-bash"><code class="lang-bash"><strong>[root@host ~]\# vi /etc/fstab
</strong><strong>
</strong>UUID=c7b57190-8fba-..-..  /dir1  xfs  defaults,x-systemd.requires=stratisd.service  0  0
</code></pre>

<mark style="color:orange;">`x-systemd.requires=stratisd.service`</mark> serve a dire di montare il volume quando il servizio é attivo ed abilitato!

<mark style="color:red;">ATTENZIONE</mark>

<mark style="color:red;">Non utilizzare il comando</mark> <mark style="color:red;"></mark><mark style="color:red;">`df`</mark> per verificare lo spazio del file system Stratis.\
Usa invece il comando <mark style="color:green;">`stratis pool list`</mark> per monitorare accuratamente lo spazio disponibile nel pool.

