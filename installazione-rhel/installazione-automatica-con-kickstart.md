# Installazione automatica con Kickstart

## Introduzione a _Kickstart_

La funzionalità _Kickstart_ di Red Hat Enterprise Linux automatizza le installazioni di sistema. \
È possibile utilizzare file di testo Kickstart per configurare partizionamento disco, interfacce di rete, selezione dei pacchetti e personalizzare l’installazione. \
I file Kickstart sono simili ai file di risposta usati per installazioni non interattive di Microsoft Windows.

#### Sintassi dei file Kickstart

* **Comandi Iniziali**: definiscono come installare la macchina target.
* **Sezioni**:
  * Ogni sezione usa direttive che iniziano con `%`.
  * La direttiva `%end` segna la fine di una sezione.
  * La sezione `%packages` specifica i software da includere:
    * Pacchetti singoli (senza versione).
    * `@` per gruppi di pacchetti.
    * `@^` per gruppi di ambiente.
    * Syntax `@module:stream/profile` per flussi di moduli.
  * Precedere con `-` per escludere pacchetti o gruppi, se non sono dipendenze obbligatorie.

#### Script Pre/Post

* **Sezioni `%pre` e `%post`**: per script di configurazione.
  * `%pre`: eseguiti prima del partizionamento, utili per inizializzare dispositivi richiesti dall'installazione.
  * `%post`: eseguiti dopo l’installazione, usando interpreti disponibili come Bash o Python.
  * Si possono avere più sezioni, interpretate nell’ordine di apparizione.

#### Altra Opzione: RHEL Image Builder

L'Image Builder di RHEL crea un'immagine con tutte le modifiche necessarie, compatibile con cloud pubblici e privati.

### Comandi di installazione

*   <mark style="color:orange;">`url`</mark>: Specifica l'URL che punta al supporto di installazione:

    ```bash
    url --url="http://classroom.example.com/content/rhel9.0/x86_64/dvd/"
    ```
*   <mark style="color:orange;">`repo`</mark>: Specifica dove trovare pacchetti aggiuntivi per l'installazione. \
    Questa opzione deve puntare ad una repository `DNF` valida:

    ```bash
    repo --name="appstream" --baseurl=http://classroom.example.com/content/rhel9.0/x86_64/dvd/AppStream/
    ```
* <mark style="color:yellow;">`text`</mark>: Forza un'installazione testuale.
*   <mark style="color:orange;">`vnc`</mark>: Abilita il _VNC viewer_ cosí da accdere all'installazione grafica da remoto _**over VNC:**_

    ```bash
    vnc --password=redhat
    ```

### Comandi per partizioni e dispositivi

*   <mark style="color:blue;">`clearpart`</mark>: Rimuove partizioni dal sistema prima della creazione delle partizioni:

    ```bash
    clearpart --all --drives=vda,vdb
    ```
*   <mark style="color:blue;">`part`</mark>: Specifica grandezza, formato e nome di una partizione. \
    Richiede almeno la presenza dei comandi `autopart` o `mount` :

    ```bash
    part /home --fstype=ext4 --label=homes --size=4096 --maxsize=8192 --grow
    ```
* <mark style="color:blue;">`autopart`</mark>: Crea automaticamente una _partizione root_, una _partizione di swap_ e una _partizione di avvio_ appropriata per l'architettura. \
  Su unità abbastanza grandi (50 GB+), questo comando crea anche una partizione `/home`.
*   <mark style="color:blue;">`ignoredisk`</mark>: Impedisce ad Anaconda di modificare i dischi ed è utile insieme al comando `autopart` :&#x20;

    ```bash
    ignoredisk --drives=sdc
    ```
*   <mark style="color:blue;">`bootloader`</mark>: Definisce dove installare il bootloader. Obbligatorio :&#x20;

    ```bash
    bootloader --location=mbr --boot-drive=sda
    ```
*   <mark style="color:blue;">`volgroup`</mark>, <mark style="color:blue;">`logvol`</mark>: Creata gruppi volume e volumi logici LVM :&#x20;

    ```bash
    part pv.01 --size=8192
    volgroup myvg pv.01
    logvol / --vgname=myvg --fstype=xfs --size=2048 --name=rootvol --grow
    logvol /var --vgname=myvg --fstype=xfs --size=4096 --name=varvol
    ```
* <mark style="color:blue;">`zerombr`</mark>: Inizializza i dischi la cui formattazione non é riconosciuta.

### Comandi di rete

*   <mark style="color:purple;">`network`</mark>: Configura info di rete per il sistema target. \
    Attiva i dispositivi di rete nell'ambiente d'installazione:&#x20;

    ```bash
    network --device=eth0 --bootproto=dhcp
    ```
*   <mark style="color:purple;">`firewall`</mark>: Definisce la configurazione firewall per il sistema target:

    ```bash
    firewall --enabled --service=ssh,http
    ```

### Comandi di Locazione e Sicurezza

*   <mark style="color:green;">`lang`</mark>: Impoosta la lingua di sistema. Richiesto:

    ```bash
    lang en_US
    ```
*   <mark style="color:green;">`keyboard`</mark>: Impoosta la tipologia di tastiera del sistema. Richiesto:

    ```bash
    keyboard --vckeymap=us
    ```
*   <mark style="color:green;">`timezone`</mark>: Definisce la time zone e se l'orologio hardware utilizza UTC. Richiesto:

    ```bash
    timezone --utc Europe/Amsterdam
    ```
*   <mark style="color:green;">`timesource`</mark>: Abilita o disabilita NTP. Se abiliti NTP, devi specificare server o pool NTP:

    ```bash
    timesource --ntp-server classroom.example.com
    ```
* <mark style="color:green;">`authselect`</mark>: Imposta le opzioni di autenticazione. \
  Per questo comando sono valide le opzioni che il comando `authselect` riconosce.
*   <mark style="color:green;">`rootpw`</mark>: Definisce la password inziale di `root` dell'user. Richiesto:

    ```bash
    rootpw --plaintext redhat
    or
    rootpw --iscrypted $6$KUnFfrTzO8jv.PiH$YlBbOtXBkWzoMuRfb0.SpbQ....XDR1UuchoMG1
    ```
*   <mark style="color:green;">`selinux`</mark>: Imposta la modalitá SELinux per il sistema installato:

    ```bash
    selinux --enforcing
    ```
*   <mark style="color:green;">`services`</mark>: Modifica il set predefinito di servizi da eseguire sotto il target predefinito di `systemd` :&#x20;

    ```bash
    services --disabled=network,iptables,ip6tables --enabled=NetworkManager,firewalld
    ```
*   <mark style="color:green;">`group`</mark>, <mark style="color:green;">`user`</mark>: Crea un gruppo o user locale sul sistema:

    ```bash
    group --name=admins --gid=10001
    user --name=jdoe --gecos="John Doe" --groups=admins
    ```

### Comandi misti

*   <mark style="color:red;">`logging`</mark>: definisce come Anaconda gestisce il logging durante l'installazione:

    ```bash
    logging --host=loghost.example.com
    ```
*   <mark style="color:red;">`firstboot`</mark>: All'avvio del sistema per la prima volta, se abilitato, si avvia il _Setup Agent._ \
    Questo comando richiede il pacchetto `initial-setup` :&#x20;

    ```bash
    firstboot --disabled
    ```
* <mark style="color:red;">`reboot`</mark>, <mark style="color:red;">`poweroff`</mark>, <mark style="color:red;">`halt`</mark>: Specifica l'azione finale ad installazione completata. \
  L'opzione di default é`halt`.

### Esempio del file _Kickstart_

```vim
# PRIMA PARTE 1/3
#version=RHEL9

# Define system bootloader options
bootloader --append="console=ttyS0 console=ttyS0,115200n8 no_timer_check net.ifnames=0  crashkernel=auto" --location=mbr --timeout=1 --boot-drive=vda

# Clear and partition disks
clearpart --all --initlabel
ignoredisk --only-use=vda
zerombr
part / --fstype="xfs" --ondisk=vda --size=10000

# Define installation options
text
repo --name="appstream" --baseurl="http://classroom.example.com/content/rhel9.0/x86_64/dvd/AppStream/"
url --url="http://classroom.example.com/content/rhel9.0/x86_64/dvd/"

# Configure keyboard and language settings
keyboard --vckeymap=us
lang en_US

# Set a root password, authselect profile, and selinux policy
rootpw --plaintext redhat
authselect select sssd
selinux --enforcing
firstboot --disable

# Enable and disable system services
services --disabled="kdump,rhsmcertd" --enabled="sshd,rngd,chronyd"

# Configure the system timezone and NTP server
timezone America/New_York --utc
timesource --ntp-server classroom.example.com
```

```vim
# SECONDA PARTE 2/3
%packages

@core
chrony
cloud-init
dracut-config-generic
dracut-norescue
firewalld
grub2
kernel
rsync
tar
-plymouth

%end
```

```vim
# ULTIMA PARTE 3/3
%post

echo "This system was deployed using Kickstart on $(date)" > /etc/motd

%end
```

```vim
# PARTE EXTRA
%post --interpreter="/usr/libexec/platform-python"

print("This line of text is printed with python")

%end
```

## Procedura d'installazione _Kickstart_

1. Creazione di un file _**Kickstart**_.
2. Pubblica il file _**Kickstart**_ affinché l'installatore _Anaconda_ possa accedervi.
3. Avvia l'installatore di _Anaconda_ e puntalo al file _**Kickstart**_.

### Creazione file _Kickstart_

Due modi:

* sito web di _**Kickstart generator**_
* editor di testo

#### _Kickstart generator_

Accedere a [ https://access.redhat.com/labs/kickstartconfig ](https://access.redhat.com/labs/kickstartconfig) :&#x20;

<figure><img src="../.gitbook/assets/image (17).png" alt=""><figcaption></figcaption></figure>

#### Creazione file di testo

Creare un file Kickstart da zero è complesso, quindi prova prima a modificare un file esistente. Ogni installazione crea un file `/root/anaconda-ks.cfg` che contiene le direttive Kickstart utilizzate nell'installazione. \
Questo file è un buon punto di partenza per creare un file _Kickstart_.&#x20;

Il pacchetto <mark style="color:yellow;">`pykickstart`</mark> fornisce le utility `ksvalidator` e `ksverdiff`.

L'utilità <mark style="color:orange;">`ksvalidator`</mark> controlla la presenza di errori di sintassi in un file _Kickstart_. Assicura che le parole chiave e le opzioni siano utilizzate correttamente, ma non convalida i percorsi URL, i singoli pacchetti, i gruppi, né alcuna parte degli script `%post` o `%pre`. \
Ad esempio, se la direttiva `firewall --disabled` è scritta male, allora il comando `ksvalidator` potrebbe produrre uno dei seguenti errori:

<pre class="language-bash"><code class="lang-bash"><strong>[user@host ~]$ ksvalidator /tmp/anaconda-ks.cfg
</strong>The following problem occurred on line 10 of the kickstart file:

Unknown command: frewall

<strong>[user@host ~]$ ksvalidator /tmp/anaconda-ks.cfg
</strong>The following problem occurred on line 10 of the kickstart file:

no such option: --dsabled
</code></pre>

L'utility <mark style="color:orange;">`ksverdiff`</mark>  mostra differenze di sintassitra versioni differenti di OS:

<pre class="language-bash"><code class="lang-bash"><strong>[user@host ~]$ ksverdiff -f RHEL8 -t RHEL9
</strong>The following commands were removed in RHEL9:
device deviceprobe dmraid install multipath

The following commands were deprecated in RHEL9:
autostep btrfs method

The following commands were added in RHEL9:
timesource
...output omitted...
</code></pre>

### Pubblicazione del _file Kickstart_ su _Anaconda_

Fornisci il file Kickstart all'installatore posizionandolo in una di queste posizioni:

* Un server di rete disponibile al momento dell'installazione tramite FTP, HTTP o NFS.
* Una chiavetta USB disponibile o un CD-ROM.
* Un disco rigido locale sul sistema.

L'installatore deve accedere al file Kickstart per iniziare un'installazione automatizzata. \
Di solito, il file è disponibile tramite un _server FTP_, _web_ o _NFS_.&#x20;

\
I server di rete facilitano la gestione dei file _Kickstart_, poiché le modifiche possono essere apportate una sola volta e poi utilizzate immediatamente per future installazioni.

Fornendo i file Kickstart su _USB_ o _CD-ROM_ è anche conveniente. \
Il file Kickstart può essere integrato nel supporto di avvio che avvia l'installazione. \
Tuttavia, quando il file Kickstart viene modificato, è necessario generare nuovi supporti di installazione.

Fornire il file Kickstart su un _disco locale_ consente una rapida ricostruzione di un sistema.

### Avvia _Anaconda_ e seleziona il file _Kickstart_

Dopo aver scelto un metodo Kickstart, si comunica all'installer dove trovare il file _Kickstart_ passando il parametro <mark style="color:orange;">`inst.ks=`</mark>_<mark style="color:orange;">`LOCATION`</mark>_ al kernel di installazione.&#x20;

Considera i seguenti esempi:

* `inst.ks=http://`_`server`_`/`_`dir`_`/`_`file`_
* `inst.ks=ftp://`_`server`_`/`_`dir`_`/`_`file`_
* `inst.ks=nfs:`_`server`_`:/`_`dir`_`/`_`file`_
* `inst.ks=hd:`_`device`_`:/`_`dir`_`/`_`file`_
* `inst.ks=cdrom:`_`device`_&#x20;

<figure><img src="../.gitbook/assets/image (18).png" alt=""><figcaption></figcaption></figure>

Durante le installazioni di macchine virtuali tramite Virtual Machine Manager o `virt-manager`, l'URL _Kickstart_ può essere specificato in una casella sotto le Opzioni URL.&#x20;

Quando installi macchine fisiche, avvia utilizzando il supporto di installazione e premi il tasto **Tab** per interrompere il processo di avvio.\
Aggiungi un parametro `inst.ks=`_`LOCATION`_ al kernel di installazione.
