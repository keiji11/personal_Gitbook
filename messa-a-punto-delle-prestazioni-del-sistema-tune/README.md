# Messa a punto delle prestazioni del sistema (Tune)

## sistema Tune

Gli amministratori di sistema ottimizzano le prestazioni di un sistema regolando le impostazioni dei dispositivi in base ai carichi di lavoro dei vari casi d'uso. \
Il demone ottimizzato applica le regolazioni sia in modo statico che dinamico utilizzando profili di ottimizzazione che riflettono i requisiti specifici del carico di lavoro.

### Configurazione tuning _statico_

Il demone `tuned` applica le impostazioni di sistema all'avvio di un servizio o alla selezione di un nuovo profilo di ottimizzazione. \
Con l'ottimizzazione statica, il demone `tuned` imposta i parametri del kernel in base alle prestazioni complessive previste, senza modificarli al variare dei livelli di attività.

### Configurazione tuning _dinamico_

Il demone `tuned` monitora l'attività del sistema e regola automaticamente le impostazioni in base alle variazioni di comportamento. \
Ad esempio, i dispositivi di archiviazione sono molto utilizzati durante l'avvio, ma meno durante l'uso di browser e email. Allo stesso modo, CPU e rete si attivano durante i picchi lavorativi. Il demone ottimizza le prestazioni adattandole ai carichi di lavoro correnti usando profili di configurazione predefiniti.

Per monitorare e regolare le impostazioni dei parametri, il demone `tuned` utilizza moduli chiamati _<mark style="color:yellow;">**plug-in di monitoraggio e tuning**</mark>_. I plug-in di monitoraggio analizzano il sistema per raccogliere dati, che i plug-in di tuning utilizzano per ottimizzazioni dinamiche. I principali plug-in di monitoraggio sono:

* `disk`: Monitora il carico del disco.
* `net`: Monitora il carico della rete.
* `load`: Monitora il carico della CPU.

Per il tuning, i plug-in principali sono:

* `disk`: Regola i parametri del disco.
* `net`: Configura la velocità di interfaccia e la funzionalità Wake-on-LAN.
* `cpu`: Regola parametri della CPU.

**Il tuning dinamico è disabilitato per default.** Abilitarlo impostando `dynamic_tuning` su 1 nel file di configurazione `/etc/tuned/tuned-main.conf`. Ajustamenti periodici si fanno ogni `update_interval` secondi.

<pre class="language-bash"><code class="lang-bash"><strong>[root@host ~]$ cat /etc/tuned/tuned-main.conf
</strong>...output omitted...
# dispositivi tune dinamici, se disabilitati solo quelli statici saranno usati.
dynamic_tuning = 1
...output omitted...
# Aggiorna intervallo per tunings dinamici (in secondi).
# Deve essere multiplo del parametro sleep_interval.
update_interval = 10
...output omitted...
</code></pre>

## L'utilità `tuned`&#x20;

Installazione ed abilitazione:

```bash
[root@host ~]$ dnf install tuned
...output omitted...
[root@host ~]$ systemctl enable --now tuned
Created symlink /etc/systemd/system/multi-user.target.wants/tuned.service → /usr/lib/systemd/system/tuned.service.
```

L'applicazione `tuned` fornisce profili nelle seguenti categorie:

* Profili per il risparmio energetico
* Profili per il miglioramento delle prestazioni

I profili per il miglioramento delle prestazioni includono profili che si concentrano sui seguenti aspetti:

* Bassa latenza per archiviazione e rete
* Alta capacità per archiviazione e rete
* Prestazioni delle macchine virtuali
* Prestazioni dell'host di virtualizzazione

La seguente tabella mostra un elenco dei profili di tuning distribuiti con Red Hat Enterprise Linux 9:

<table><thead><tr><th width="239" valign="top">Profili Tuned</th><th valign="top">Scopo</th></tr></thead><tbody><tr><td valign="top"><code>balanced</code></td><td valign="top">Ideale per sistemi che richiedono un compromesso tra risparmio energetico e prestazioni.</td></tr><tr><td valign="top"><code>powersave</code></td><td valign="top">Ottimizza il sistema per il massimo risparmio energetico.</td></tr><tr><td valign="top"><code>throughput-performance</code></td><td valign="top">Ottimizza il sistema per la massima capacità produttiva.</td></tr><tr><td valign="top"><code>accelerator-performance</code></td><td valign="top">Sintonizza le stesse variabili di <code>throughput-performance</code> e riduce anche la latenza a meno di 100 μs.</td></tr><tr><td valign="top"><code>latency-performance</code></td><td valign="top">Ideale per sistemi server che richiedono bassa latenza a scapito del consumo energetico.</td></tr><tr><td valign="top"><code>network-throughput</code></td><td valign="top">Derivato dal profilo <code>throughput-performance</code>. Vengono applicati ulteriori parametri di ottimizzazione della rete per ottenere la massima capacità di rete.</td></tr><tr><td valign="top"><code>network-latency</code></td><td valign="top">Derivato dal profilo <code>latency-performance</code>. Abilita parametri di tuning di rete aggiuntivi per garantire una bassa latenza di rete.</td></tr><tr><td valign="top"><code>desktop</code></td><td valign="top">Derivato dal profilo <code>balanced</code>. Offre una risposta più rapida per le applicazioni interattive.</td></tr><tr><td valign="top"><code>hpc-compute</code></td><td valign="top">Profilo derivato da <code>latency-performance</code>. Ideale per il calcolo ad alte prestazioni.</td></tr><tr><td valign="top"><code>virtual-guest</code></td><td valign="top">Setta il sistema su performance massima se gira su una virtual machine.</td></tr><tr><td valign="top"><code>virtual-host</code></td><td valign="top">Setta il sistema su performance massima se opera come un host su una virtual machine.</td></tr><tr><td valign="top"><code>intel-sst</code></td><td valign="top">Ottimizzato per configurazioni con Intel Speed Select Technology. Usalo come sovrapposizione su altri profili.</td></tr><tr><td valign="top"><code>optimize-serial-console</code></td><td valign="top">Aumenta la reattività della console seriale. Usalo come overlay su altri profili.</td></tr></tbody></table>

L'applicazione `tuned` conserva i profili di tuning nelle directory `/usr/lib/tuned` e `/etc/tuned`. \
Ogni profilo ha una propria directory separata, all'interno della quale si trova il file principale di configurazione chiamato `tuned.conf` e, facoltativamente, altri file.

<pre class="language-bash"><code class="lang-bash"><strong>[root@host ~]\# cd /usr/lib/tuned
</strong><strong>[root@host tuned]\# ls
</strong>accelerator-performance  hpc-compute          network-throughput       throughput-performance
balanced                 intel-sst            optimize-serial-console  virtual-guest
desktop                  latency-performance  powersave                virtual-host
functions                network-latency      recommend.d
<strong>[root@host tuned]$ ls virtual-guest
</strong>tuned.conf
</code></pre>

<pre class="language-bash"><code class="lang-bash"><strong>[root@host tuned]\# cat virtual-guest/tuned.conf
</strong>#
# tuned configuration
#

# Sommario del profilo tuning
# Include impostazioni ereditate 
[main]
summary=Optimize for running inside a virtual guest
include=throughput-performance

[sysctl]
# Se un carico di lavoro utilizza principalmente memoria anonima e raggiunge questo limite,
# l'intero set di lavoro è bufferizzato per l'I/O, e qualsiasi ulteriore buffering 
# di scrittura richiederebbe lo swap, quindi è il momento di rallentare le scritture 
# fino a quando l'I/O può recuperare. I carichi di lavoro che utilizzano principalmente 
# mappature di file possono essere in grado di utilizzare valori ancora più elevati.
# 
# Il generatore di dati sporchi avvia il writeback a questa percentuale 
# (il valore predefinito del sistema è 20%).
vm.dirty_ratio = 30

# L'I/O del filesystem è solitamente molto più efficiente dello swapping, 
# quindi cerca di mantenere basso lo swapping. È generalmente sicuro andare ancora più 
# in basso su sistemi con archiviazione di livello server
server.vm.swappiness = 30
</code></pre>

Questo file di configurazione è utile per creare nuovi profili di tuning, poiché è possibile utilizzare uno dei profili forniti come base e poi aggiungere o modificare i parametri. \


Per creare o modificare i profili, copia i file da `/usr/lib/tuned` a `/etc/tuned` e poi modificali. \
I file in `/etc/tuned` hanno la precedenza su quelli in `/usr/lib/tuned`. Non modificare direttamente i file in `/usr/lib/tuned`. \
Le sezioni del file `tuned.conf` utilizzano i plugin per modificare i parametri. \
Ad esempio, la sezione `[sysctl]` modifica i parametri del kernel `vm.dirty_ratio` e `vm.swappiness` tramite il plugin `sysctl`.

## Gestione profili da CLI

### Comando `tune-adm`&#x20;

#### `active` :&#x20;

<pre class="language-bash"><code class="lang-bash"><strong>[root@host ~]\# tuned-adm active
</strong>Current active profile: virtual-guest
</code></pre>

`list` :&#x20;

<pre class="language-bash"><code class="lang-bash"><strong>[root@host ~]\# tuned-adm list
</strong>Available profiles:
- accelerator-performance     - Throughput performance based tuning with ...
- balanced                    - General non-specialized tuned profile
- desktop                     - Optimize for the desktop use-case
- hpc-compute                 - Optimize for HPC compute workloads
- intel-sst                   - Configure for Intel Speed Select Base Frequency
- latency-performance         - Optimize for deterministic performance at ...
- network-latency             - Optimize for deterministic performance at ...
...output omitted...
Current active profile: virtual-guest
</code></pre>

`profile_info` :&#x20;

<pre class="language-bash"><code class="lang-bash"><strong>[root@host ~]\$ tuned-adm profile_info network-latency
</strong>Profile name:
network-latency

Profile summary:
Optimize for deterministic performance at the cost of increased power consumption, focused on low latency network performance
...output omitted..
</code></pre>

Usa il comando `tuned-adm profile`` `_`nomeprofilo`_ per passare a un profilo attivo diverso che si adatta meglio ai requisiti di ottimizzazione attuali del sistema.

<pre class="language-bash"><code class="lang-bash"><strong>[root@host ~]$ tuned-adm profile throughput-performance
</strong><strong>[root@host ~]$ tuned-adm active
</strong>Current active profile: throughput-performance
</code></pre>

Il comando `tuned-adm recommend` può consigliare un profilo di tuning per il sistema. Il sistema utilizza questo meccanismo per determinare il profilo predefinito dopo la sua installazione.\
Nota bene:\
Il comando `tuned-adm recommend` basa la sua raccomandazione su varie caratteristiche del sistema, incluso se il sistema è una macchina virtuale e altre categorie selezionate durante l'installazione del sistema.

```bash
[root@host ~]$ tuned-adm recommend
virtual-guest
```

<pre class="language-bash"><code class="lang-bash"><strong>[root@host ~]$ tuned-adm off
</strong><strong>[root@host ~]$ tuned-adm active
</strong>No current active profile.
</code></pre>

## Gestione profili da Web Console

<figure><img src="../.gitbook/assets/image (1) (1).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../.gitbook/assets/image (1) (1) (1).png" alt=""><figcaption></figcaption></figure>
