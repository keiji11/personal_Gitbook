# Abilitare le software repositories DNF

#### Abilitare le Red Hat Software Repositories

I sistemi spesso hanno accesso a numerosi repository Red Hat. Il comando `dnf repolist all` elenca tutti i repository disponibili e i loro stati:

<pre><code><strong>[user@host ~]$ dnf repolist all
</strong>repo id                                repo name                status
rhel-9.0-for-x86_64-appstream-rpms     RHEL 9.0 AppStream       enabled
rhel-9.0-for-x86_64-baseos-rpms        RHEL 9.0 BaseOS          enabled
</code></pre>

#### NOTA

Le sottoscrizioni Red Hat concedono l'accesso a specifici repository. In passato, gli amministratori dovevano associare le sottoscrizioni su base per-sistema. L'Accesso Contenuto Semplice (SCA, Simple Content Access) semplifica il modo in cui i sistemi accedono ai repository. Con SCA, i sistemi possono accedere a qualsiasi repository da qualsiasi sottoscrizione che acquisti, senza dover associare una sottoscrizione. Puoi abilitare SCA sul Portale Clienti Red Hat all'interno di Le Mie Sottoscrizioni → Allocazioni Sottoscrizioni, o sul tuo server Red Hat Satellite.

Il comando `dnf config-manager` può abilitare e disabilitare i repository. Ad esempio, il seguente comando abilita il repository `rhel-9-server-debug-rpms`:&#x20;

<pre><code><strong>[user@host ~]$ dnf config-manager --enable rhel-9-server-debug-rpms
</strong></code></pre>

Le fonti esterne a Red Hat forniscono software tramite repository di terze parti. Ad esempio, Adobe offre alcuni dei suoi software per Linux attraverso repository DNF. In un'aula Red Hat, il server `content.example.com` ospita repository DNF. Il comando `dnf` può accedere ai repository da un sito web, un server FTP o il file system locale.

È possibile aggiungere un repository di terze parti in due modi. Si può creare un file `.repo` nella directory `/etc/yum.repos.d/`, oppure è possibile aggiungere una sezione `[repository]` al file `/etc/dnf/dnf.conf`. Red Hat consiglia di utilizzare i file `.repo` e di riservare il file `dnf.conf` per configurazioni aggiuntive dei repository. Il comando `dnf` cerca in entrambe le posizioni per impostazione predefinita; tuttavia, i file `.repo` hanno la precedenza. Un file `.repo` contiene l'URL del repository, un nome, se usare GPG per verificare le firme dei pacchetti e, in caso affermativo per quest'ultimo, l'URL per puntare alla chiave GPG di fiducia.

**Aggiungere DNF Repositories**

Il comando `dnf config-manager` può anche aggiungere repository alla macchina. Il seguente comando crea un file `.repo` utilizzando l'URL di un repository esistente.

<pre><code><strong>[user@host ~]$ dnf config-manager \
</strong><strong>--add-repo="https://dl.fedoraproject.org/pub/epel/9/Everything/x86_64/"
</strong>Adding repo from: https://dl.fedoraproject.org/pub/epel/9/Everything/x86_64/
</code></pre>

Il file `.repo` corrispondente è visibile nella directory `/etc/yum.repos.d` :

<pre data-full-width="true"><code><strong>[user@host ~]$ cd /etc/yum.repos.d
</strong><strong>[user@host yum.repos.d]$ cat \
</strong><strong>dl.fedoraproject.org_pub_epel_9_Everything_x86_64_.repo
</strong>[dl.fedoraproject.org_pub_epel_9_Everything_x86_64_]
name=created by dnf config-manager from https://dl.fedoraproject.org/pub/epel/9/Everything/x86_64/
baseurl=https://dl.fedoraproject.org/pub/epel/9/Everything/x86_64/
enabled=1
</code></pre>

Il comando `rpm` utilizza chiavi GPG per firmare i pacchetti e importa le chiavi pubbliche per verificare l'integrità e l'autenticità dei pacchetti. Il comando `dnf` utilizza i file di configurazione dei repository per fornire le posizioni delle chiavi pubbliche GPG e importa le chiavi per verificare i pacchetti. Le chiavi sono memorizzate in varie posizioni sul sito del repository remoto, come `http://dl.fedoraproject.org/pub/epel/RPM-GPG-KEY-EPEL-9`. Gli amministratori dovrebbero scaricare la chiave in un file locale piuttosto che permettere al comando `dnf` di recuperare la chiave da una fonte esterna. Ad esempio, il seguente file `.repo` utilizza il parametro `gpgkey` per fare riferimento a una chiave locale:

```
[EPEL]
name=EPEL 9
baseurl=https://dl.fedoraproject.org/pub/epel/9/Everything/x86_64/
enabled=1
gpgcheck=1
gpgkey=file:///etc/pki/rpm-gpg/RPM-GPG-KEY-EPEL-9
```

**RPM Configuration Packages per Repository locali**

Alcuni repository forniscono un file di configurazione e una chiave pubblica GPG come parte di un pacchetto RPM per semplificarne l'installazione. Puoi importare la chiave pubblica GPG utilizzando il comando `rpm --import`. Il comando `dnf install` può scaricare e installare questi pacchetti RPM.

Ad esempio, il seguente comando importa la chiave pubblica GPG `RPM-GPG-KEY-EPEL-9` (EPEL) e installa il RPM del repository Extra Packages for Enterprise Linux (EPEL) di RHEL9:

<pre><code><strong>[user@host ~]$ rpm --import \
</strong><strong>https://dl.fedoraproject.org/pub/epel/RPM-GPG-KEY-EPEL-9
</strong><strong>[user@host ~]$ dnf install \
</strong><strong>https://dl.fedoraproject.org/pub/epel/epel-release-latest-9.noarch.rpm
</strong></code></pre>

#### WARNING

Importa la chiave GPG RPM prima di installare pacchetti firmati, per assicurarti che i pacchetti provengano da una fonte affidabile. Se la chiave GPG RPM non viene importata, il comando `dnf` non riesce ad installare pacchetti firmati.

L'opzione `--nogpgcheck` del comando `dnf` ignora le chiavi GPG mancanti, ma potrebbe portare all'installazione di pacchetti compromessi o falsificati.

I file `.repo` elencano spesso più riferimenti a repository in un unico file. Ogni riferimento al repository inizia con un nome di una sola parola tra parentesi quadre.

<pre><code><strong>[user@host ~]$ cat /etc/yum.repos.d/epel.repo
</strong>[epel]
name=Extra Packages for Enterprise Linux $releasever - $basearch
#baseurl=https://download.example/pub/epel/$releasever/Everything/$basearch/
metalink=https://mirrors.fedoraproject.org/metalink?repo=epel-$releasever&#x26;arch=$basearch&#x26;infra=$infra&#x26;content=$contentdir
enabled=1
gpgcheck=1
countme=1
gpgkey=file:///etc/pki/rpm-gpg/RPM-GPG-KEY-EPEL-$releasever
...output omitted...
[epel-source]
name=Extra Packages for Enterprise Linux $releasever - $basearch - Source
#baseurl=https://download.example/pub/epel/$releasever/Everything/source/tree/
metalink=https://mirrors.fedoraproject.org/metalink?repo=epel-source-$releasever&#x26;arch=$basearch&#x26;infra=$infra&#x26;content=$contentdir
enabled=0
gpgkey=file:///etc/pki/rpm-gpg/RPM-GPG-KEY-EPEL-$releasever
gpgcheck=1
</code></pre>

Per definire un repository, ma non cercarlo di default, inserire il parametro `enabled=0`. Sebbene il comando `dnf config-manager` abiliti e disabiliti persistentemente i repository, le opzioni del comando `dnf` `--enablerepo=`_`PATTERN`_ e `--disablerepo=`_`PATTERN`_ abilitano e disabilitano temporaneamente i repository mentre il comando è in esecuzione.
