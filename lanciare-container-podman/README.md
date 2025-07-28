# Lanciare Container (podman)

## Introduzione ai container

### Descrivere i container

Nei sistemi informatici, un _container_ è un processo incapsulato che include le dipendenze necessarie per eseguire il programma. \
I contenitori hanno le loro librerie specifiche, indipendenti da quelle del sistema operativo host. Mentre le librerie non specifiche sono fornite dal sistema operativo. \
Il motore del contenitore crea un file system unendo diversi livelli dell'immagine del contenitore e aggiunge un livello scrivibile per le modifiche a runtime. \
I contenitori sono _effimeri_, significa che il livello scrivibile viene rimosso quando il contenitore viene eliminato.

<div data-full-width="true"><figure><img src="../.gitbook/assets/image (23).png" alt=""><figcaption></figcaption></figure></div>

I contenitori utilizzano funzionalità del kernel Linux come _namespaces_, _cgroups_, _SELinux_ e _secure computing mode (seccomp)_. \
I namespaces, in particolare, isolano i processi all'interno dei contenitori tra loro e dal sistema host. Sebbene l'ambiente nei contenitori sia basato su Linux, queste caratteristiche spesso vengono virtualizzate dai motori di container sui sistemi non Linux. \
La tecnologia della containerizzazione deriva da strumenti come <mark style="color:orange;">`chroot`</mark> e si è evoluta fino all'_<mark style="color:orange;">Open Container Initiative (OCI)</mark>_, che stabilisce standard per creare e gestire i _container_. \
La maggior parte dei motori di contenitori segue le specifiche OCI, permettendo agli sviluppatori di costruire artefatti eseguibili come contenitori OCI. \
I contenitori utilizzano SELinux e secure computing mode per applicare confini di sicurezza e limitare le funzionalità disponibili.

## Immagini VS Istanze

I container possono essere distinti in due concetti simili ma distinti: _<mark style="color:orange;">immagini di container</mark>_ e _<mark style="color:yellow;">istanze di container</mark>_.&#x20;

* Un'_<mark style="color:orange;">immagine di container</mark>_ contiene dati immutabili che definiscono un'applicazione e le sue librerie. Le immagini di container vengono utilizzate per creare&#x20;
* _<mark style="color:yellow;">istanze di container</mark>_<mark style="color:yellow;">,</mark> versioni eseguibili dell'immagine che includono riferimenti a rete, dischi, e altre necessità di runtime. È possibile utilizzare una singola immagine di container per creare molte istanze di container distinte su più host. \
  L'applicazione all'interno di un container è indipendente dall'ambiente host.

Nota: le immagini di container OCI sono definite dalla specifica <mark style="color:orange;">`image-spec`</mark>, mentre le istanze sono definite dalla specifica <mark style="color:yellow;">`runtime-spec`</mark><mark style="color:yellow;">.</mark> \
Un modo per comprendere la differenza è che un'istanza sta a un'immagine come un oggetto sta a una classe nella programmazione orientata agli oggetti.

### VMs vs Containers

Le macchine virtuali e i container utilizzano software diversi per la gestione e la funzionalità.&#x20;

Gli hypervisor, come _KVM_, _Xen_, _VMware_ e _Hyper-V_, sono applicazioni che forniscono la funzionalità di virtualizzazione per le macchine virtuali. \
L'equivalente di un hypervisor per i container è un motore di container, come _<mark style="color:orange;">Podman</mark>_.

<table><thead><tr><th valign="top">X</th><th valign="top">VMs</th><th valign="top">Containers</th></tr></thead><tbody><tr><td valign="top">Funzionalitá <strong>livello-macchina</strong></td><td valign="top">Hypervisor</td><td valign="top">Container engine</td></tr><tr><td valign="top"><strong>Gestione</strong></td><td valign="top">Interfaccia di gestione VM</td><td valign="top">Container engine o software di orchestrazione</td></tr><tr><td valign="top"><strong>Livello di</strong> <strong>Virtualizzazione</strong> </td><td valign="top">Ambiente completamente virtualizzato </td><td valign="top">Solo le parti necessarie</td></tr><tr><td valign="top"><strong>Grandezza</strong></td><td valign="top">Misurata in <strong>gigabytes</strong></td><td valign="top">Misurata in <strong>megabytes</strong></td></tr><tr><td valign="top"><strong>Portabilitá</strong></td><td valign="top">Generalmente solo su stesso hypervisor</td><td valign="top">Qualunque <em><strong>OCI-compliant engine</strong></em></td></tr></tbody></table>

Gestione degli hypervisor con software di gestione supplementare. \
Questo può essere incluso con l'hypervisor o essere esterno, come _Virtual Machine Manager con KVM_.&#x20;

Per i container, puoi gestirli direttamente tramite l'engine del container o usare strumenti di orchestrazione come _Red Hat OpenShift Container Platform (RHOCP_) che può gestire sia container che macchine virtuali da un'unica interfaccia e _Kubernetes_ per eseguirli su larga scala. \


Le VM raramente funzionano su hypervisor diversi, mentre i container compatibili con lo standard OCI funzionano con vari motori di container senza problemi di compatibilità.

### Distribuzione su larga scala

Entrambi i container e le VM possono funzionare bene a vari livelli. \
Poiché un container richiede considerevolmente meno risorse di una VM, i container offrono vantaggi in termini di prestazioni e risorse su larga scala. \
Un metodo comune in ambienti su larga scala è utilizzare container che girano all'interno di VM. \
Questa configurazione sfrutta i punti di forza di ciascuna tecnologia.

## Sviluppo con i Container

La containerizzazione offre molti vantaggi per il processo di sviluppo, come _test_ e _deployment_ più semplici, fornendo strumenti per la stabilità, sicurezza e flessibilità.

### Testing e Ciclo di lavoro (Workflows)

Uno dei maggiori vantaggi dei container per gli sviluppatori è la possibilità di scalare. \
Gli sviluppatori possono scrivere software, testarlo localmente e poi distribuirlo su un server cloud o un cluster dedicato con poche o nessuna modifica.&#x20;

Questo flusso di lavoro è utile per creare microservizi, container piccoli e temporanei progettati per avviarsi e spegnersi secondo le necessità. \
Inoltre, i container facilitano l'uso di pipeline di CI/CD per distribuire applicazioni in vari ambienti. In particolare, RHOCP offre integrazioni con pipeline CI/CD.

### Stabilitá

Le immagini dei container sono un obiettivo stabile per gli sviluppatori. \
Le applicazioni software richiedono specifiche versioni di librerie per il deployment, evitando problemi di dipendenze o requisiti di OS. \
Integrando le librerie nei container, gli sviluppatori si assicurano che non ci siano problemi tra l'ambiente di test e quello di produzione. \
Ad esempio, un container con una specifica versione di Python garantisce che la stessa versione sia utilizzata in ogni ambiente.

### Applicazioni Multi-containerizzate

Puoi eseguire i container dalla stessa immagine per repliche ad _alta disponibilità (HA)_, oppure da diverse immagini. \
Ad esempio, uno sviluppatore può creare un'applicazione che include un container di database che funziona separatamente dal container dell'API web dell'applicazione. \
Un'applicazione può fare affidamento sul software di gestione dei container per fornire repliche HA con opzioni multi-container, come _<mark style="color:orange;">**Podman Pods**</mark>_ o la specifica <mark style="color:orange;">`compose-spec`</mark>.

