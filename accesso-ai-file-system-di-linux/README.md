# 📂 Accesso ai file system di Linux

Identificare File System e Dispositivi

Concetti di Gestione della Memoria (storage)

Red Hat Enterprise Linux (RHEL) utilizza il **file system Extents (XFS)** come file system locale predefinito. RHEL supporta il file system Extended File System (ext4) per la gestione dei file locali. A partire da RHEL 9, il file system **Extensible File Allocation Table (exFAT)** è supportato per l'uso di supporti rimovibili. In un cluster di server aziendali, i dischi condivisi utilizzano il file system **Global File System 2 (GFS2)** per gestire l'accesso simultaneo a più nodi.

**File system e punti di montaggio**

È possibile accedere al contenuto di un file system montandolo su una directory vuota. Questa directory è chiamata **punto di montaggio**. Quando la directory è montata, si può usare il comando `ls` per elencarne il contenuto. Molti file system vengono montati automaticamente all'avvio del sistema.

Un punto di montaggio è leggermente diverso da una lettera di unità di Microsoft Windows, dove ogni file system è un'entità separata. I punti di montaggio consentono di disporre di più dispositivi di file system in un'unica struttura ad albero. Questo punto di montaggio è simile alle cartelle montate su NTFS in Microsoft Windows.

**File system, archiviazione e dispositivi a blocchi**

Un dispositivo a blocchi è un file che fornisce un accesso di basso livello ai dispositivi di archiviazione. Un dispositivo a blocchi deve essere opzionalmente partizionato e un file system creato prima che il dispositivo possa essere montato.

La directory `/dev` contiene i file dei dispositivi a blocchi, che RHEL crea automaticamente per tutti i dispositivi. In RHEL 9, il primo disco rigido **SATA**, **SAS**, **SCSI** o **USB** rilevato è chiamato dispositivo `/dev/sda`; il secondo è il dispositivo`/dev/sdb` e così via. Questi nomi rappresentano l'intero disco rigido.

| tipo di dispositivo                         | nomenclatura dei dispositivi           |
| ------------------------------------------- | -------------------------------------- |
| SATA/SAS/USB-attached storage (SCSI driver) | `/dev/sda`, `/dev/sdb`, `/dev/sdc`, …​ |
| `virtio-blk` paravirtualized storage (VMs)  | `/dev/vda`, `/dev/vdb`, `/dev/vdc`,…​  |
| `virtio-scsi` paravirtualized storage (VMs) | `/dev/sda`, `/dev/sdb`, `/dev/sdc`, …​ |
| NVMe-attached storage (SSDs)                | `/dev/nvme0`, `/dev/nvme1`, …​         |
| SD/MMC/eMMC storage (SD cards)              | `/dev/mmcblk0`, `/dev/mmcblk1`, …​     |

**Esaminare i file system**

Usate il comando `df` per visualizzare una panoramica dei dispositivi del file system locale e remoto, che include lo spazio totale del disco, lo spazio utilizzato, lo spazio libero e la percentuale dell'intero spazio del disco.

L'esempio seguente mostra i file system e i punti di montaggio sul computer host:

```bash
[user@host ~]$ df -h
Filesystem      Size  Used Avail Use% Mounted on
devtmpfs        892M     0  892M   0% /dev
tmpfs           915M     0  915M   0% /dev/shm
tmpfs           915M   17M  899M   2% /run
tmpfs           915M     0  915M   0% /sys/fs/cgroup
/dev/vda3       8.0G  1.4G  6.7G  17% /
/dev/vda1      1014M  166M  849M  17% /boot
tmpfs           183M     0  183M   0% /run/user/1000
```

Utilizzate il comando `du` per ottenere informazioni più dettagliate su uno specifico spazio di directory. Le opzioni `-h` e `-H` del comando `du` convertono l'output in un formato leggibile dall'uomo. Il comando `du` mostra la dimensione di tutti i file della directory corrente in modo ricorsivo.

Visualizzare il rapporto di utilizzo del disco per la directory `/usr/share` sul computer host:

```bash
[root@host ~]# du /usr/share
...output omitted...
176 /usr/share/smartmontools
184 /usr/share/nano
8 /usr/share/cmake/bash-completion
8 /usr/share/cmake
356676  /usr/share
```

Visualizza il rapporto sull'utilizzo del disco in formato leggibile per la directory `/usr/share`:

```bash
[root@host ~]# du -h /usr/share
...output omitted...
176K  /usr/share/smartmontools
184K  /usr/share/nano
8.0K  /usr/share/cmake/bash-completion
8.0K  /usr/share/cmake
369M  /usr/share
```
