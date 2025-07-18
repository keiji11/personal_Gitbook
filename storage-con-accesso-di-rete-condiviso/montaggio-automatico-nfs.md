# Montaggio automatico NFS

## Montare NFS esportato con l'<mark style="color:orange;">`Automounter`</mark>&#x20;

L'_automounter_ è un servizio (<mark style="color:orange;">`autofs`</mark>) che monta automaticamente i file system e le esportazioni NFS su richiesta, e smonta automaticamente i file system e le esportazioni NFS quando le risorse montate non sono più in uso.

I sistemi di file automount non vengono necessariamente montati durante l'avvio del sistema. \
Al contrario, <mark style="color:orange;">i sistemi di file controllati dall'automount vengono montati su richiest</mark>a, quando un utente o un'applicazione tenta di accedere al punto di mount del file system per accedere ai file.

### Benfici dell'`Automounter`&#x20;

L'uso delle risorse per i file system montati automaticamente è simile a quelli montati all'avvio, con il vantaggio di <mark style="color:orange;">evitare corruzioni inattese</mark> disattivandoli quando non sono in uso, garantendo sempre una configurazione aggiornata e selezionando la connessione più veloce in presenza di server NFS ridondanti.

### Il metodo di servizio Automounter `autofs`&#x20;

Il servizio `autofs` monta dinamicamente i file system, tra cui NFS e SMB, analogamente a `/etc/fstab`, e li smonta quando non sono più utilizzati, a differenza dei montaggi permanenti.

### Casi d'uso diretti ed indiretti delle Mappature

L'automount gestisce il montaggio delle directory in due modi:&#x20;

* il montaggio <mark style="color:yellow;">diretto</mark>, con punti di montaggio fissi e permanenti.
* il montaggio <mark style="color:yellow;">indiretto</mark>, che crea punti di montaggio temporanei per directory remote quando necessario.

## Configurare il servizio di _<mark style="color:green;">Automounter</mark>_

Installazione pacchetti necessari ( <mark style="color:green;">`autofs`</mark> e <mark style="color:orange;">`nfs-utils`</mark> ) :

<pre class="language-bash"><code class="lang-bash"><strong>[root@host ~]\# dnf install autofs nfs-utils
</strong></code></pre>

### il file _Master Map_

Il file _master map_ (<mark style="color:green;">`/etc/auto.master`</mark>) è il file di configurazione predefinito per il servizio <mark style="color:orange;">`autofs`</mark>. Puoi utilizzare il file <mark style="color:yellow;">`/etc/autofs.conf`</mark> per modificare il file della mappa principale per il servizio <mark style="color:orange;">`autofs`</mark>. \
Usa la directory <mark style="color:purple;">`/etc/auto.master.d`</mark> per configurare il file della mappa principale. \
Questo file identifica la directory di base per i punti di montaggio e identifica il file di mappatura per creare i montaggi automatici.

<pre class="language-bash"><code class="lang-bash"><strong>[root@host ~]\# cat /etc/auto.master.d/demo.autofs
</strong>mount-point     map-file
#esempio:
/shares         /etc/auto.autofs
</code></pre>

### il file _Map_

I file di mappatura configurano le proprietà dei punti di mount on-demand individuali.&#x20;

L'automounter crea le directory se non esistono. \
Se le directory esistono già prima che l'automounter inizi, l'automounter non le rimuoverà al termine. \
Se è specificato un timeout, la directory viene automaticamente smontata se non viene acceduta entro il periodo di timeout. \
Se si utilizza un singolo nome di directory per il punto di mount, la directory viene montata come mount indiretto. \
Se si utilizza il percorso completo per il punto di mount, la directory viene montata come mount diretto.

Il file di mappatura utilizza il seguente formato per l'automount:

<pre class="language-bash"><code class="lang-bash"><strong>[root@host ~]\# cat /etc/auto.demo
</strong>mount-point     mount-options     source-location
</code></pre>

#### Creare un file indiretto di _Map_

<pre class="language-bash"><code class="lang-bash"><strong>[root@host ~]\# cat /etc/auto.master.d/indirect.autofs
</strong>/shares  /etc/auto.indirect
</code></pre>

<pre class="language-bash"><code class="lang-bash"><strong>[root@host ~]\# cat /etc/auto.indirect
</strong>work      -rw,sync      hosta:/shares/work
</code></pre>

Il servizio `autofs` gestisce automaticamente la creazione e la rimozione dei punti di mount.  \
\
Le <mark style="color:yellow;">opzioni di montaggio</mark> iniziano con un trattino (`-`) e sono separate da virgole, come `rw` per accesso lettura/scrittura e `sync` per sincronizzazione immediata. \
Opzioni specifiche includono `-fstype=` per definire il tipo di file system, e `-strict` per trattare gli errori come fatali. \


Gli NFS esportati seguono il formato `host:/percorso`, con le necessarie autorizzazioni su `hosta`.

#### Creare un file diretto di _Map_

<pre class="language-bash"><code class="lang-bash"><strong>[root@host ~]\# cat /etc/direct.autofs
</strong>/-  /etc/auto.direct
</code></pre>

`/-` rappresente la directory base. \
In questo caso, i dettagli del _mapping_ sono in `/etc/auto.direct`.

<pre class="language-bash"><code class="lang-bash"><strong>[root@host ~]\# cat /etc/auto.direct
</strong>/mnt/docs  -rw,sync  hosta:/shares/docs
</code></pre>

Notare che `/mnt/docs` é un _**path assoluto.**_

#### Uso delle _wildcards_ nei file _Map_

<pre class="language-bash"><code class="lang-bash"><strong>[root@host ~]\# cat /etc/auto.indirect
</strong>*  -rw,sync  hosta:/shares/&#x26;
</code></pre>

Il simbolo <mark style="color:red;">`*`</mark> rappresenta un carattere jolly che corrisponde a qualsiasi nome di directory sotto la directory di montaggio specificata. \
Il simbolo <mark style="color:red;">`&`</mark> è utilizzato come segnaposto per il nome catturato dal carattere jolly, permettendo di utilizzare lo stesso nome come parte del percorso NFS effettivo.

## Lanciare il servizio _Automounter_

<pre class="language-bash"><code class="lang-bash"><strong>[root@host ~]\# systemctl enable --now autofs
</strong>Created symlink /etc/systemd/system/multi-user.target.wants/autofs.service → /usr/lib/systemd/system/autofs.service.
</code></pre>

## Il metodo alternativo Automount

Questo metodo può essere più semplice dell'installazione e configurazione del servizio `autofs`. Tuttavia, un'unità `systemd.automount` può supportare solo punti di montaggio con percorsi assoluti, simili alle mappe dirette di `autofs`.
