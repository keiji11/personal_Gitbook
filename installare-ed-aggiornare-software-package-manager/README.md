# 📦 Installare ed aggiornare software package manager

## Gestione dei Pacchetti Software con DNF

DNF (Dandified YUM) è il gestore di pacchetti che ha sostituito YUM in Red Hat Enterprise Linux 9. Con il comando `dnf`, è possibile installare, aggiornare, rimuovere e ottenere informazioni sui pacchetti software e le loro dipendenze. DNF permette anche di visualizzare la cronologia delle transazioni e di lavorare con più repository di software di Red Hat e di terze parti. Il comando di basso livello `rpm` può essere usato per installare pacchetti, ma non è progettato per lavorare con i repository di pacchetti o per risolvere automaticamente le dipendenze da molteplici fonti.

In questo corso, si lavora con il comando `dnf`. Alcune documentazioni potrebbero ancora fare riferimento al comando `yum`, tuttavia, i file sono gli stessi e sono collegati tramite link simbolico a DNF:

* yum-groups-manager -> /usr/libexec/dnf-utils
* yumdownloader -> /usr/libexec/dnf-utils
* yum-debug-restore -> /usr/libexec/dnf-utils
* yum-debug-dump -> /usr/libexec/dnf-utils
* yum-config-manager -> /usr/libexec/dnf-utils
* yum-builddep -> /usr/libexec/dnf-utils
* yum -> dnf

I comandi DNF sono funzionalmente identici ai comandi YUM. Per compatibilità, i comandi YUM esistono ancora come link simbolici a DNF.

**Trova Software con DNF**

Il comando `dnf help` mostra le informazioni sull'uso. Il comando `dnf list` mostra i pacchetti installati e disponibili.

<pre><code><strong>[user@host ~]$ dnf list 'httpd'
</strong>Available Packages
http-parser.i686               2.9.4-6.el9    rhel-9.0-for-x86_64-appstream-rpms
http-parser.x86_64             2.9.4-6.el9    rhel-9.0-for-x86_64-appstream-rpms
httpcomponents-client.noarch   4.5.13-2.el9   rhel-9.0-for-x86_64-appstream-rpms
httpcomponents-core.noarch     4.4.13-6.el9   rhel-9.0-for-x86_64-appstream-rpms
httpd.x86_64                   2.4.51-5.el9   rhel-9.0-for-x86_64-appstream-rpms
httpd-devel.x86_64             2.4.51-5.el9   rhel-9.0-for-x86_64-appstream-rpms
httpd-filesystem.noarch        2.4.51-5.el9   rhel-9.0-for-x86_64-appstream-rpms
httpd-manual.noarch            2.4.51-5.el9   rhel-9.0-for-x86_64-appstream-rpms
httpd-tools.x86_64             2.4.51-5.el9   rhel-9.0-for-x86_64-appstream-rpms
</code></pre>

Il comando `dnf search` _KEYWORD_ elenca i pacchetti in base alle parole chiave presenti nel nome e nei campi di sommario. Per cercare pacchetti con "web server" nel nome, sommario e campi di descrizione, utilizzare `search all` :

<pre><code><strong>[user@host ~]$ dnf search all 'web server'
</strong>================== Summary &#x26; Description Matched: web server ===================
nginx.x86_64 : A high performance web server and reverse proxy server
pcp-pmda-weblog.x86_64 : Performance Co-Pilot (PCP) metrics from web server logs
========================= Summary Matched: web server ==========================
libcurl.x86_64 : A library for getting files from web servers
libcurl.i686 : A library for getting files from web servers
======================= Description Matched: web server ========================
freeradius.x86_64 : High-performance and highly configurable free RADIUS server
git-instaweb.noarch : Repository browser in gitweb
http-parser.i686 : HTTP request/response parser for C
http-parser.x86_64 : HTTP request/response parser for C
httpd.x86_64 : Apache HTTP Server
mod_auth_openidc.x86_64 : OpenID Connect auth module for Apache HTTP Server
mod_jk.x86_64 : Tomcat mod_jk connector for Apache
mod_security.x86_64 : Security module for the Apache HTTP Server
varnish.i686 : High-performance HTTP accelerator
varnish.x86_64 : High-performance HTTP accelerator
...output omitted...
</code></pre>

Il comando `dnf info PACKAGENAME` restituisce informazioni dettagliate su un pacchetto, inclusi lo spazio su disco necessario per l'installazione. Ad esempio, il comando seguente recupera informazioni sul pacchetto `httpd`:

<pre><code><strong>[user@host ~]$ dnf info httpd
</strong>Available Packages
Name         : httpd
Version      : 2.4.51
Release      : 5.el9
Architecture : x86_64
Size         : 1.5 M
Source       : httpd-2.4.51-5.el9.src.rpm
Repository   : rhel-9.0-for-x86_64-appstream-rpms
Summary      : Apache HTTP Server
URL          : https://httpd.apache.org/
License      : ASL 2.0
Description  : The Apache HTTP Server is a powerful, efficient, and extensible
             : web server.
</code></pre>

Il comando `dnf provides` _`PATHNAME`_ mostra i pacchetti che corrispondono al nome del percorso specificato (i nomi dei percorsi spesso includono caratteri jolly). Ad esempio, il seguente comando trova i pacchetti che forniscono la directory `/var/www/html` :

<pre><code><strong>[user@host ~]$ dnf provides /var/www/html
</strong>httpd-filesystem-2.4.51-5.el9.noarch : The basic directory layout for the Apache HTTP Server
Repo        : rhel-9.0-for-x86_64-appstream-rpms
Matched from:
Filename    : /var/www/html
</code></pre>

**Installare e Rimuovere Software con DNF**

Il comando `dnf install` _NOME\_PACCHETTO_ ottiene e installa un pacchetto software, incluse tutte le dipendenze.

<pre><code><strong>[root@host ~]# dnf install httpd
</strong>Dependencies resolved.
================================================================================
 Package          Arch   Version       Repository                          Size
================================================================================
Installing:
 httpd            x86_64 2.4.51-5.el9  rhel-9.0-for-x86_64-appstream-rpms 1.5 M
Installing dependencies:
 apr              x86_64 1.7.0-11.el9  rhel-9.0-for-x86_64-appstream-rpms 127 k
 apr-util         x86_64 1.6.1-20.el9  rhel-9.0-for-x86_64-appstream-rpms  98 k
 apr-util-bdb     x86_64 1.6.1-20.el9  rhel-9.0-for-x86_64-appstream-rpms  15 k
 httpd-filesystem noarch 2.4.51-5.el9  rhel-9.0-for-x86_64-appstream-rpms  17 k
 httpd-tools      x86_64 2.4.51-5.el9  rhel-9.0-for-x86_64-appstream-rpms  88 k
 redhat-logos-httpd
                  noarch 90.4-1.el9    rhel-9.0-for-x86_64-appstream-rpms  18 k
Installing weak dependencies:
 apr-util-openssl x86_64 1.6.1-20.el9  rhel-9.0-for-x86_64-appstream-rpms  17 k
 mod_http2        x86_64 1.15.19-2.el9 rhel-9.0-for-x86_64-appstream-rpms 153 k
 mod_lua          x86_64 2.4.51-5.el9  rhel-9.0-for-x86_64-appstream-rpms  63 k

Transaction Summary
================================================================================
Install  10 Packages

Total download size: 2.1 M
Installed size: 5.9 M
<strong>Is this ok [y/N]: y
</strong>Downloading Packages:
(1/10): apr-1.7.0-11.el9.x86_64.rpm             6.4 MB/s | 127 kB     00:00
(2/10): apr-util-bdb-1.6.1-20.el9.x86_64.rpm    625 kB/s |  15 kB     00:00
(3/10): apr-util-openssl-1.6.1-20.el9.x86_64.rp 1.9 MB/s |  17 kB     00:00
...output omitted...
Total                                            24 MB/s | 2.1 MB     00:00
Running transaction check
Transaction check succeeded.
Running transaction test
Transaction test succeeded.
Running transaction
  Preparing        :                                                        1/1
  Installing       : apr-1.7.0-11.el9.x86_64                               1/10
  Installing       : apr-util-bdb-1.6.1-20.el9.x86_64                      2/10
  Installing       : apr-util-openssl-1.6.1-20.el9.x86_64                  3/10
...output omitted...
Installed:
  apr-1.7.0-11.el9.x86_64              apr-util-1.6.1-20.el9.x86_64
  apr-util-bdb-1.6.1-20.el9.x86_64     apr-util-openssl-1.6.1-20.el9.x86_64
...output omitted...
Complete!
</code></pre>

Il comando `dnf update` `NOME_PACCHETTO` ottiene e installa una versione più recente del pacchetto specificato, inclusi eventuali dipendenze. Generalmente, il processo cerca di preservare i file di configurazione esistenti, ma in alcuni casi, tali file potrebbero essere rinominati se gli autori del pacchetto considerano che il nome precedente non sarà più adeguato dopo l'aggiornamento. Se non viene specificato alcun `NOME_PACCHETTO`, il comando installerà tutti gli aggiornamenti pertinenti.

<pre><code><strong>[root@host ~]# dnf update
</strong></code></pre>

Utilizza il comando `dnf list kernel` per elencare tutti i kernel installati e disponibili. Per visualizzare il kernel attualmente in esecuzione, usa il comando `uname`. L'opzione `-r` del comando `uname` mostra solo la versione e il rilascio del kernel, mentre l'opzione `-a` mostra il rilascio del kernel e ulteriori informazioni. Poiché un nuovo kernel può essere testato solo avviandolo, il sistema supporta specificamente l'installazione di più versioni contemporaneamente. Se il nuovo kernel non si avvia, la versione precedente resta disponibile.&#x20;

Eseguire il comando `dnf update kernel` installa il nuovo kernel. I file di configurazione contengono un elenco di pacchetti da installare sempre, anche se l'amministratore richiede un aggiornamento.

<pre><code><strong>[user@host ~]$ dnf list kernel
</strong>Installed Packages
kernel.x86_64                       5.14.0-70.el9                        @System
<strong>[user@host ~]$ uname -r
</strong>5.14.0-70.el9.x86_64
<strong>[user@host ~]$ uname -a
</strong>Linux workstation.lab.example.com 5.14.0-70.el9.x86_64 #1 SMP PREEMPT Thu Feb 24 19:11:22 EST 2022 x86_64 x86_64 x86_64 GNU/Linux
</code></pre>

Il comando `dnf remove PACKAGENAME` rimuove un pacchetto software installato, inclusi eventuali pacchetti supportati.

<pre><code><strong>[root@host ~]# dnf remove httpd
</strong></code></pre>

#### WARNING

Il comando `dnf remove` rimuove i pacchetti elencati _e qualsiasi pacchetto che richieda i pacchetti da rimuovere_ (e i pacchetti richiesti da questi pacchetti, e così via). Questo comando può portare alla rimozione inaspettata di pacchetti, quindi è consigliabile rivedere l'elenco dei pacchetti da rimuovere.

**Installa e Rimuovi Gruppi di Software con DNF**

Il comando `dnf` ha anche il concetto di _gruppi_, che sono collezioni di software correlati che vengono installati insieme.

In Red Hat Enterprise Linux 9, il comando `dnf` può installare due tipi di gruppi di pacchetti. I gruppi regolari sono collezioni di pacchetti. I gruppi di ambiente sono collezioni di gruppi regolari. I pacchetti o gruppi che queste collezioni forniscono potrebbero essere elencati come `obbligatori` (devono essere installati se il gruppo è installato), `predefiniti` (normalmente installati se il gruppo è installato) o `opzionali` (non installati quando il gruppo è installato, a meno che non siano specificamente richiesti).

Similmente al comando `dnf list`, il comando `dnf group list` visualizza i nomi dei gruppi installati e disponibili.

<pre><code><strong>[user@host ~]$ dnf group list
</strong>Available Environment Groups:
   Server with GUI
   Server
   Minimal Install
...output omitted...
Available Groups:
   Legacy UNIX Compatibility
   Console Internet Tools
   Container Management
...output omitted...
</code></pre>

Alcuni gruppi sono normalmente installati tramite i gruppi di ambiente e sono nascosti per impostazione predefinita. Elenca questi gruppi nascosti con il comando `dnf group list hidden`.

Il comando `dnf group info` visualizza informazioni su un gruppo. Include un elenco di nomi di pacchetti obbligatori, predefiniti e opzionali.

<pre><code><strong>[user@host ~]$ dnf group info "RPM Development Tools"
</strong>Group: RPM Development Tools
 Description: Tools used for building RPMs, such as rpmbuild.
 Mandatory Packages:
   redhat-rpm-config
   rpm-build
 Default Packages:
   rpmdevtools
 Optional Packages:
   rpmlint
</code></pre>

Il comando `dnf group install` installa un gruppo che include i suoi pacchetti obbligatori e predefiniti insieme ai loro pacchetti dipendenti.

<pre><code><strong>[root@host ~]# dnf group install "RPM Development Tools"
</strong>...output omitted...
Installing Groups:
 RPM Development Tools

Transaction Summary
================================================================================
Install  19 Packages

Total download size: 4.7 M
Installed size: 15 M
Is this ok [y/N]: y
...output omitted...
</code></pre>

#### IMPORTANT

A partire da Red Hat Enterprise Linux 7, il comportamento dei gruppi Yum è cambiato, venendo trattati come oggetti e tracciati dal sistema. Se un gruppo installato viene aggiornato, e se il repository Yum ha aggiunto nuovi pacchetti obbligatori o predefiniti al gruppo, allora quei nuovi pacchetti vengono installati durante l'aggiornamento.

**Visualizza la Cronologia delle Transazioni**

Tutte le transazioni di installazione e rimozione vengono registrate nel file `/var/log/dnf.rpm.log` :

<pre><code><strong>[user@host ~]$ tail -5 /var/log/dnf.rpm.log
</strong>2022-03-23T16:46:43-0400 SUBDEBUG Installed: python-srpm-macros-3.9-52.el9.noarch
2022-03-23T16:46:43-0400 SUBDEBUG Installed: redhat-rpm-config-194-1.el9.noarch
2022-03-23T16:46:44-0400 SUBDEBUG Installed: elfutils-0.186-1.el9.x86_64
2022-03-23T16:46:44-0400 SUBDEBUG Installed: rpm-build-4.16.1.3-11.el9.x86_64
2022-03-23T16:46:44-0400 SUBDEBUG Installed: rpmdevtools-9.5-1.el9.noarch
</code></pre>

Il comando `dnf history` mostra un riepilogo delle transazioni di installazione e rimozione.

<pre><code><strong>[root@host ~]# dnf history
</strong>ID     | Command line              | Date and time    | Action(s)      | Altered
--------------------------------------------------------------------------------
     7 | group install RPM Develop | 2022-03-23 16:46 | Install        |   20
     6 | install httpd             | 2022-03-23 16:21 | Install        |   10 EE
     5 | history undo 4            | 2022-03-23 15:04 | Removed        |   20
     4 | group install RPM Develop | 2022-03-23 15:03 | Install        |   20
     3 |                           | 2022-03-04 03:36 | Install        |    5
     2 |                           | 2022-03-04 03:33 | Install        |  767 EE
     1 | -y install patch ansible- | 2022-03-04 03:31 | Install        |   80
</code></pre>

Il comando `dnf history undo` annulla una transazione.

<pre><code><strong>[root@host ~]# dnf history undo 6
</strong>...output omitted...
Removing:
 apr-util-openssl x86_64 1.6.1-20.el9 @rhel-9.0-for-x86_64-appstream-rpms  24 k
 httpd            x86_64 2.4.51-5.el9 @rhel-9.0-for-x86_64-appstream-rpms 4.7 M
...output omitted...
</code></pre>

**Sommario dei comandi DNF**

I pacchetti possono essere localizzati, installati, aggiornati e rimossi per nome o per gruppi di pacchetti.

| Task:                                          | Command:                      |
| ---------------------------------------------- | ----------------------------- |
| List installed and available packages by name. | `dnf list [NAME-PATTERN]`     |
| List installed and available groups.           | `dnf group list`              |
| Search for a package by keyword.               | `dnf search KEYWORD`          |
| Show details of a package.                     | `dnf info PACKAGENAME`        |
| Install a package.                             | `dnf install PACKAGENAME`     |
| Install a package group.                       | `dnf group install GROUPNAME` |
| Update all packages.                           | `dnf update`                  |
| Remove a package.                              | `dnf remove PACKAGENAME`      |
| Display transaction history.                   | `dnf history`                 |

**Gestire i Moduli dei Pacchetti Stream con DNF**

La gestione tradizionale di versioni alternative del pacchetto software di un'applicazione e dei suoi pacchetti correlati comportava il mantenimento di diversi repository per ciascuna versione. Questa situazione risultava tediosa da gestire sia per gli sviluppatori, che desideravano la versione più recente dell'applicazione, sia per gli amministratori, che volevano la versione più stabile dell'applicazione. Red Hat semplifica questo processo attraverso l'utilizzo di una tecnologia denominata _modularità_. Con la modularità, un singolo repository può ospitare molteplici versioni del pacchetto dell'applicazione e delle sue dipendenze.

**Introduzione a BaseOS e AppStream**

Red Hat Enterprise Linux 9 distribuisce i contenuti tramite due principali repository software: _BaseOS_ e _AppStream_

Il repository BaseOS fornisce il contenuto del sistema operativo di base per Red Hat Enterprise Linux sotto forma di pacchetti RPM. I componenti di BaseOS hanno lo stesso ciclo di vita del contenuto nelle precedenti versioni di Red Hat Enterprise Linux. Il repository Application Stream fornisce contenuti con cicli di vita variabili sia come moduli che come pacchetti tradizionali.

### Repository Application Stream

Il repository Application Stream contiene due tipi di contenuti: moduli e pacchetti RPM tradizionali. Un modulo descrive un insieme di pacchetti RPM che appartengono insieme. I moduli possono contenere diversi flussi per rendere disponibili per l'installazione più versioni delle applicazioni. Abilitare un flusso di modulo dà al sistema l'accesso ai pacchetti RPM all'interno di quel flusso di modulo. Tipicamente, i moduli organizzano i pacchetti RPM attorno a una specifica versione di un'applicazione software o linguaggio di programmazione. Un modulo tipico contiene pacchetti con un'applicazione, pacchetti con le librerie di dipendenza specifiche dell'applicazione, pacchetti con documentazione per l'applicazione e pacchetti con utilità di aiuto.

Sia BaseOS che AppStream sono parti necessarie di un sistema Red Hat Enterprise Linux 9.&#x20;

Application Stream contiene parti necessarie del sistema, così come un'ampia gamma di applicazioni che erano precedentemente disponibili come parte delle Red Hat Software Collections e altri prodotti e programmi. Ogni Application Stream ha un ciclo di vita che è lo stesso di Red Hat Enterprise Linux 9 o più breve.

#### IMPORTANT

Red Hat Enterprise Linux 9.0 viene distribuito senza moduli. Le versioni future di RHEL 9 potrebbero introdurre contenuti aggiuntivi e versioni software più recenti sotto forma di moduli. Inoltre, a partire da RHEL 9, è necessario specificare manualmente i flussi di moduli predefiniti, in quanto non sono più definiti automaticamente. È possibile definire i flussi di moduli predefiniti tramite file di configurazione nella directory `/etc/dnf/modules.defaults.d/`.

**Module Streams**

Per ogni modulo, è possibile abilitare solo uno dei suoi flussi, e questo flusso fornisce i suoi pacchetti. Ogni modulo ha uno o più _flussi di modulo_, che contengono diverse versioni del contenuto. Ognuno dei flussi riceve aggiornamenti indipendentemente. Pensate al flusso del modulo come a un repository virtuale all'interno del repository fisico dello Stream delle Applicazioni.

**Module Profiles**

L'installazione di un profilo di modulo comporta l'installazione di un particolare insieme di pacchetti dallo stream del modulo. Successivamente, è possibile installare o disinstallare pacchetti normalmente. Se non si specifica un profilo, allora il modulo installa il suo profilo predefinito. Ogni modulo può avere uno o più profili. Un profilo è un elenco di pacchetti che è possibile installare insieme per uno specifico caso d'uso, come per un server, client, sviluppo, installazione minima o altro.

**Gestire i Moduli con DNF**

Red Hat Enterprise Linux 9 supporta le caratteristiche modulari dello Stream di Applicazioni. Per gestire i contenuti modulari, puoi usare il comando `dnf module`. Altrimenti, il comando `dnf` funziona con moduli simili ai pacchetti regolari.

Vedi la lista seguente per alcuni comandi importanti per gestire i moduli:

* **`dnf module provides`** _**`pacchetto`**_ : Mostra quale modulo fornisce un pacchetto specifico.
* **`dnf module info`** _**`nome-modulo`**_ : Visualizza i dettagli di un modulo, inclusi i profili disponibili e un elenco dei pacchetti che il modulo installa. Eseguire il comando `dnf module info` senza specificare uno stream di modulo mostra i pacchetti installati dal profilo e stream predefinito. Usa il formato _nome-modulo:stream_ per visualizzare uno stream di modulo specifico. Aggiungi l'opzione `--profile` per visualizzare le informazioni sui pacchetti installati da ciascuno dei profili del modulo.
* **`dnf module list`** _**`nome-modulo`**_ : Elenca gli stream del modulo per un modulo specifico e recupera il loro stato.
* **`dnf module list`** : Elenca i moduli disponibili con il nome del modulo, stream, profili e un sommario.
