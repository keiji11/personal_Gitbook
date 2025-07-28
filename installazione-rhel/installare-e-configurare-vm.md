# Installare e configurare VM

## Introduzione alla virtualizzazione KVM

La virtualizzazione è una funzionalità che consente di dividere una singola macchina fisica in più _macchine virtuali (VM)_, ognuna delle quali può eseguire un sistema operativo indipendente. \
Red Hat Enterprise Linux supporta _<mark style="color:red;">KVM (Kernel-based Virtual Machine)</mark>_, una soluzione di virtualizzazione completa integrata nel kernel standard di Linux. \
KVM può eseguire più sistemi operativi guest Windows e Linux.

<figure><img src="../.gitbook/assets/image (20).png" alt=""><figcaption></figcaption></figure>

In Red Hat Enterprise Linux, puoi gestire KVM dalla riga di comando con il comando <mark style="color:orange;">`virsh`</mark> o graficamente con lo strumento <mark style="color:orange;">Virtual Machines della console web</mark>. \
Red Hat Enterprise Linux, tipicamente un host "thick", supporta le VM offrendo anche servizi e funzioni di gestione. \


<mark style="color:orange;">Red Hat Virtualization (RHV)</mark> offre un'interfaccia web centralizzata per gestire un'infrastruttura virtuale completa. \


<mark style="color:orange;">Red Hat OpenStack Platform (RHOSP)</mark> fornisce la base per creare, distribuire e scalare un cloud pubblico o privato.&#x20;



<mark style="color:orange;">Red Hat OpenShift Virtualization</mark> incorpora componenti RHV per la distribuzione di contenitori su hardware bare metal.&#x20;



Quando SELinux è abilitato, _**sVirt**_ isola gli ospiti e l'_hypervisor_, assegnando etichette uniche ai processi delle macchine virtuali e ai relativi file di disco virtuali.

### Tipi di VM

Nelle versioni di Red Hat Enterprise Linux 8 e successive, sono disponibili due tipi principali di macchine virtuali per installare il sistema operativo guest:

1. <mark style="color:orange;">**i440FX**</mark>: include chipset i440FX, supporta ISA, ponti PCI, controller IDE, adattatori VGA ed Ethernet.
2. <mark style="color:orange;">**Q35**</mark>: variante più moderna con supporto per PCIe, SATA e controller SM Bus. Consigliata per la maggior parte dei casi.

Inoltre, il pacchetto <mark style="color:orange;">`edk2-ovmf`</mark> fornisce supporto UEFI e Secure Boot per le macchine virtuali.

### OS Guest suppportati

Red Hat offre _<mark style="color:orange;">KVM</mark>_ per l'uso della virtualizzazione su una singola macchina con RHEL. \
Per funzionalità più scalabili e avanzate, Red Hat offre _<mark style="color:orange;">Red Hat</mark> <mark style="color:orange;"></mark><mark style="color:orange;">**OpenShift**</mark> <mark style="color:orange;"></mark><mark style="color:orange;">Virtualization</mark>_ e _<mark style="color:orange;">Red Hat</mark> <mark style="color:orange;"></mark><mark style="color:orange;">**OpenStack**</mark> <mark style="color:orange;"></mark><mark style="color:orange;">Platform</mark>_.

## Configurare un sistema RHEL fisico come un host di virtualizzazione

### Installazione del tool di virtualizzazione

<pre class="language-bash"><code class="lang-bash"><strong>[root@host ~]\# dnf group list | grep -i virt
</strong>   Virtualization Host
<strong>[root@host ~]\# dnf group info "Virtualization Host"
</strong>...output omitted...
Environment Group: Virtualization Host
 Description: Minimal virtualization host.
 Mandatory Groups:
   Base
   Core
   Standard
   Virtualization Hypervisor
   Virtualization Tools
 Optional Groups:
   Debugging Tools
   Network File System Client
   Remote Management for Linux
   Virtualization Platform

<strong>[root@host ~]\# dnf group install "Virtualization Host"
</strong>...output omitted...
</code></pre>

### Verifica dei requisiti di sistema per la virtualizzazione

`virt-host-validate`  deve passare tutti i test:

<pre class="language-bash"><code class="lang-bash"><strong>[root@host ~]\# virt-host-validate
</strong>  QEMU: Checking for hardware virtualization                            : PASS
  QEMU: Checking if device /dev/kvm exists                              : PASS
  QEMU: Checking if device /dev/kvm is accessible                       : PASS
  QEMU: Checking if device /dev/vhost-net exists                        : PASS
  QEMU: Checking if device /dev/net/tun exists                          : PASS
  QEMU: Checking for cgroup 'cpu' controller support                    : PASS
  QEMU: Checking for cgroup 'cpuacct' controller support                : PASS
  QEMU: Checking for cgroup 'cpuset' controller support                 : PASS
  QEMU: Checking for cgroup 'memory' controller support                 : PASS
  QEMU: Checking for cgroup 'devices' controller support                : PASS
  QEMU: Checking for cgroup 'blkio' controller support                  : PASS
  QEMU: Checking for device assignment IOMMU support                    : PASS
  QEMU: Checking for secure guest support                               : PASS
</code></pre>

### Installazione di una VM da terminale

<pre class="language-bash"><code class="lang-bash"># INSTALLAZIONE del GUEST OS con virt-install 
[root@host ~]\# virt-install --name demo --memory 4096 --vcpus 2 --disk size=40 \
<strong>--os-type linux --cdrom /root/rhel.iso
</strong>...output omitted...
</code></pre>

## Gestione VM da web console

Installazione pacchetto `cockpit-machines` :

<pre class="language-bash"><code class="lang-bash"><strong>[root@host ~]\# dnf install cockpit-machines
</strong></code></pre>

Abilitazione servizio:

<pre class="language-bash"><code class="lang-bash"><strong>[root@host ~]\# systemctl enable --now cockpit.socket
</strong></code></pre>

Andare dal browser su [https://localhost/9090 ](https://localhost/9090):&#x20;

<figure><img src="../.gitbook/assets/image (21).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../.gitbook/assets/image (22).png" alt=""><figcaption></figcaption></figure>

* _**Name**_ - imposta un nome _dominio_ per la configurazione della macchina virtuale. \
  Questo nome non è correlato al nome host che assegni alla macchina virtuale durante l'installazione.
* _**Installation type**_ - é il metodo per accedere al supporto d'installazione. \
  Le opzioni includono il file system locale, o un URL HTTPS, FTP, NFS, o PXE.
* _**Installation source**_ - fornisce il percorso della sorgente di installazione.
* _**Operating system**_ - definisce il sistema operativo della macchina virtuale. I\
  l livello di virtualizzazione presenta un'emulazione hardware per essere compatibile con il sistema operativo scelto.
* _**Storage**_ - definisce se creare un volume di archiviazione o utilizzare uno esistente..
* _**Size**_ - è la dimensione del disco durante la creazione di un nuovo volume. Associa dischi aggiuntivi con la VM dopo l'installazione.
* _**Memory -**_ é l'ammontare della RAM da fornire alla nuova VM.
* _**Immediately start VM**_ - indica se la VM si avvia immediatamente dopo aver cliccato su Crea.
