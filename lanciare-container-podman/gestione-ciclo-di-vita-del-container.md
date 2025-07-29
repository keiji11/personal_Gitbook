# Gestione ciclo di vita del container

## Ciclo di vita del container

Sottocomandi podman per cambiare lo stato dei container e delle immagini:

<div data-full-width="true"><figure><img src="../.gitbook/assets/image (27).png" alt=""><figcaption></figcaption></figure></div>

Podman fornisce anche una serie di sottocomandi per ottenere informazioni sui container in esecuzione e fermati. \
È possibile utilizzare questi sottocomandi per estrarre informazioni da container e immagini a scopo di debug, aggiornamento o reportistica. \
I seguenti sono i sottocomandi più comunemente usati per interrogare informazioni da container e immagini:

<div data-full-width="true"><figure><img src="../.gitbook/assets/image (28).png" alt=""><figcaption></figcaption></figure></div>

Questa lezione illustra le operazioni di base che è possibile utilizzare per gestire i container. \
I comandi spiegati in questa lezione accettano sia l'ID del container che il nome del container.

## Elenco dei container (<mark style="color:orange;">`ps`</mark>)

<pre class="language-bash"><code class="lang-bash"><strong># ELENCO CONTAINER ATTIVI
</strong><strong>
</strong><strong>[user@host ~]$ podman ps
</strong>CONTAINER ID  IMAGE         COMMAND              CREATED STATUS PORTS       NAMES
0ae7be593698  ...server     /bin/sh -c python... ...ago  Up...  ...8000/tcp httpd
c42e7dca12d9  ...helloworld /bin/sh -c nginx...  ...ago  Up...  ...8080/tcp nginx
</code></pre>

<pre class="language-bash"><code class="lang-bash"><strong># ELENCO TOTALE CONTAINER (--all o -a) 
</strong><strong>
</strong><strong>[user@host ~]$ podman ps --all
</strong>CONTAINER ID  IMAGE         COMMAND              CREATED STATUS    PORTS        NAMES
0ae7be593698  ...server     /bin/sh -c python... ...ago  Up...     ...8000/tcp  httpd
bd5ada1b6321  ...httpd-24   /usr/bin/run-http... ...ago  Exited... ...8080/tcp  upbeat...
c42e7dca12d9  ...helloworld /bin/sh -c nginx ... ...ago  Up...     ...8080/tcp  nginx
</code></pre>

## Ispezione dei container (<mark style="color:orange;">`inspect`</mark>)

Otteniamo tutte le info relative al container:

<pre class="language-bash"><code class="lang-bash"><strong>[user@host ~]$ podman inspect 7763097d11ab
</strong>[
    {
        "Id": "7763...cbc0",
        "Created": "2022-05-04T10:00:32.988377257-03:00",
        "Path": "container-entrypoint",
        "Args": [
            "/usr/bin/run-httpd"
        ],
        "State": {
            "OciVersion": "1.0.2-dev",
            "Status": "running",
            "Running": true,
            "Paused": false,
            "Restarting": false,
            "OOMKilled": false,
            "Dead": false,
            "Pid": 9746,
...output omitted...
        "Image": "d2b9...fa0a",
        "ImageName": "registry.access.redhat.com/ubi8/httpd-24:latest",
        "Rootfs": "",
...output omitted...
            "Env": [
                "PATH=/opt/app-root/src/bin:/opt/app-root/bin:/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin",
                "TERM=xterm",
                "container=oci",
                "HTTPD_VERSION=2.4",
...output omitted...
            "CpuCount": 0,
            "CpuPercent": 0,
            "IOMaximumIOps": 0,
            "IOMaximumBandwidth": 0,
            "CgroupConf": null
        }
    }
]
</code></pre>

Filtrando le info con <mark style="color:orange;">`--format`</mark>:

<pre class="language-bash"><code class="lang-bash"><strong>[user@host ~]$ podman inspect --format='{{.State.Status}}' redhat
</strong>running
</code></pre>

## Arresto corretto del container ( `stop` )

Podman invia un segnale `SIGTERM` al container. \
I processi utilizzano il segnale `SIGTERM` per implementare procedure di pulizia prima di interrompersi. \
Il seguente comando interrompe un container con un ID `1b982aeb75dd` :

<pre class="language-bash"><code class="lang-bash"><strong>[user@host ~]$ podman stop 1b982aeb75dd
</strong>1b982aeb75dd
</code></pre>

<pre class="language-bash"><code class="lang-bash"><strong>[user@host ~]$ podman stop --all
</strong>4aea...0c2a
6b18...54ea
7763...cbc0
</code></pre>

Se un contenitore non risponde al segnale `SIGTERM`, Podman invia il segnale `SIGKILL` per interrompere forzatamente il contenitore. \
Podman attende 10 secondi di default prima di inviare il segnale `SIGKILL`. \
È possibile modificare il comportamento predefinito utilizzando l'opzione <mark style="color:orange;">`--time`</mark> :&#x20;

<pre class="language-bash"><code class="lang-bash"><strong>[user@host ~]$ podman stop --time=100
</strong></code></pre>

Quando il processo containerizzato termina, il container entra nello stato "_exited_". \
Un processo può terminare per vari motivi, come un errore, uno stato _OOM_ (_out-of-memory_) o un completamento con successo. \
Podman elenca i container usciti insieme agli altri container fermati.

## Arresto forzato del container ( `kill` )

Puoi inviare il segnale <mark style="color:orange;">`SIGKILL`</mark> al container utilizzando il comando `podman`` `<mark style="color:orange;">`kill`</mark> :&#x20;

<pre class="language-bash"><code class="lang-bash"><strong>[user@host ~]$ podman kill httpd
</strong>httpd
</code></pre>

## Mettere in pausa un container ( `pause` - `unpause` )

Sia il comando `podman stop` che il comando `podman kill` inviano eventualmente un segnale `SIGKILL` al container. \
Il comando `podman`` `<mark style="color:orange;">`pause`</mark> sospende tutti i processi nel container inviando il segnale <mark style="color:orange;">`SIGSTOP`</mark> :&#x20;

<pre class="language-bash"><code class="lang-bash"><strong>[user@host ~]$ podman pause 4f2038c05b8c
</strong>4f2038c05b8c
</code></pre>

<pre class="language-bash"><code class="lang-bash"><strong>[user@host ~]$ podman unpause 4f2038c05b8c
</strong>4f2038c05b8c
</code></pre>

## Riavviare un container ( `restart` )

<pre class="language-bash"><code class="lang-bash"><strong>[user@host ~]$ podman restart nginx
</strong>1b98...75dd
</code></pre>

## Rimuovere i container ( `rm` )

Usa il comando `podman`` `<mark style="color:orange;">`rm`</mark> per rimuovere un container fermo. \
Il seguente comando rimuove un container _stoppato_ con l'ID del container `c58cfd4b90df` :&#x20;

<pre class="language-bash"><code class="lang-bash"><strong>[user@host ~]$ podman rm c58cfd4b90df
</strong>c58c...90df
</code></pre>

Per poter rimuovere un container attivo devi prima fermare il container in esecuzione e poi rimuoverlo. \
Il seguente comando cerca di rimuovere un container in esecuzione :&#x20;

<pre class="language-bash"><code class="lang-bash"><strong>[user@host ~]$ podman rm c58cfd4b90df
</strong>Error: cannot remove container c58c...90df as it is running - running or paused containers cannot be removed without force: container state improper
</code></pre>

Puoi aggiungere il flag <mark style="color:orange;">`--force`</mark> (o `-f`) per rimuovere forzatamente il container:

<pre class="language-bash"><code class="lang-bash"><strong>[user@host ~]$ podman rm c58cfd4b90df --force
</strong>c58c...90df
</code></pre>

Puoi anche aggiungere il flag `--all` (o `-a`) per rimuovere tutti i container fermati. \
Questo flag non riesce a rimuovere i container in esecuzione. \
Il comando seguente rimuove due container:

<pre class="language-bash"><code class="lang-bash"><strong>[user@host ~]$ podman rm --all
</strong>6b18...54ea
6c0d...a6fb
</code></pre>

Puoi combinare i flag `--force` e `--all` per rimuovere tutti i container, inclusi quelli in esecuzione.

## Storage persistente nei container

Per mantenere i dati persistenti, puoi montare il file system dell'host nel container usando l'opzione         <mark style="color:orange;">`--volume`</mark>` ``(-v)`. \
Assicurati che i permessi siano corrette: l'utente `mysql` deve possedere la directory `/var/lib/mysql` nel container. \
In un container senza privilegi (`rootless`), le corrispondenze UID e GID funzionano diversamente rispetto a un container root. \
Usa il comando `podman`` `<mark style="color:orange;">`unshare`</mark> per <mark style="color:orange;">eseguire comandi all'interno dello spazio utente</mark> e `podman`` `<mark style="color:yellow;">`unshare cat`</mark> per <mark style="color:yellow;">visualizzare la mappatura UID</mark>.

<pre class="language-bash"><code class="lang-bash"><strong>[user@host ~]$ podman unshare cat /proc/self/uid_map
</strong>         0       1000          1
         1     100000      65536
<strong>[user@host ~]$ podman unshare cat /proc/self/gid_map
</strong>         0       1000          1
         1     100000      65536
</code></pre>

Usa il comando `podman exec` per visualizzare l'UID e il GID dell'utente `mysql` all'interno del contenitore in esecuzione con storage effimero:

<pre class="language-bash"><code class="lang-bash"><strong>[user@host ~]$ podman exec -it db01 grep mysql /etc/passwd
</strong><strong>mysql:x:27:27:MySQL Server:/var/lib/mysql:/sbin/nologin
</strong></code></pre>

Decidi di montare la directory `/home/user/db_data` nel container `db01` per fornire spazio di archiviazione persistente nella directory `/var/lib/mysql` del container. \
Quindi crei la directory `/home/user/db_data` e utilizzi il comando `podman unshare` per impostare l'UID e il GID dello spazio dei nomi utente su `27` come proprietario della directory:

<pre class="language-bash"><code class="lang-bash"><strong>[user@host ~]$ mkdir /home/user/db_data
</strong><strong>[user@host ~]$ podman unshare chown 27:27 /home/user/db_data
</strong></code></pre>

Utilizziamo l'opzione <mark style="color:red;">`-v`</mark> del comando <mark style="color:red;">`podman run`</mark> per <mark style="color:red;">montare la directory</mark>:

<pre class="language-bash"><code class="lang-bash"><strong>[user@host ~]$ podman run -d --name db01 \
</strong><strong>-e MYSQL_USER=student \
</strong><strong>-e MYSQL_PASSWORD=student \
</strong><strong>-e MYSQL_DATABASE=dev_data \
</strong><strong>-e MYSQL_ROOT_PASSWORD=redhat \
</strong><strong>-v /home/user/db_data:/var/lib/mysql \
</strong><strong>registry.lab.example.com/rhel8/mariadb-105
</strong></code></pre>

avrai notato che il container `db01` non é in funzione:

<pre class="language-bash"><code class="lang-bash"><strong>[user@host ~]$ podman ps -a
</strong>CONTAINER ID  IMAGE            COMMAND         CREATED         STATUS                     PORTS       NAMES
dfdc20cf9a7e  registry.lab...  run-mysqld      29 seconds ago  Exited (1) 29 seconds ago              db01
</code></pre>

`podman container logs` mostra un errore di permessi su `/var/lib/mysql/db_data` :

<pre class="language-bash"><code class="lang-bash"><strong>[user@host ~]$ podman container logs db01
</strong>...output omitted...
---> 16:41:25     Initializing database ...
---> 16:41:25     Running mysql_install_db ...
mkdir: cannot create directory '/var/lib/mysql/db_data': Permission denied
Fatal error Can't create database directory '/var/lib/mysql/db_data'
</code></pre>

Questo errore si verifica a causa del <mark style="color:orange;">contesto SELinux errato</mark> impostato sulla directory `/home/user/db_data` sulla macchina host. ->

### Contesti SELinux negli storage dei container

È necessario impostare il tipo di contesto SELinux `container_file_t` prima di poter montare la directory come storage persistente su un container. \
Se la directory non ha il contesto SELinux `container_file_t`, il container non può accedere alla directory. \


È possibile aggiungere l'<mark style="color:red;">opzione</mark> <mark style="color:red;"></mark><mark style="color:red;">`Z`</mark> <mark style="color:red;"></mark><mark style="color:red;">all'argomento dell'opzione</mark> <mark style="color:red;"></mark><mark style="color:red;">`-v`</mark> <mark style="color:red;"></mark><mark style="color:red;">del comando</mark> <mark style="color:red;"></mark><mark style="color:red;">`podman run`</mark> per <mark style="color:red;">impostare automaticamente il contesto SELinux sulla directory</mark>. \


Pertanto, utilizzi il comando `podman run -v /home/user/db_data:/var/lib/mysql:`<mark style="color:red;">`Z`</mark> per impostare il contesto SELinux per la directory `/home/user/db_data` quando la monti come storage persistente per la directory `/var/lib/mysql` :&#x20;

<pre class="language-bash"><code class="lang-bash"><strong>[user@host ~]$ podman run -d --name db01 \
</strong><strong>-e MYSQL_USER=student \
</strong><strong>-e MYSQL_PASSWORD=student \
</strong><strong>-e MYSQL_DATABASE=dev_data \
</strong><strong>-e MYSQL_ROOT_PASSWORD=redhat \
</strong><strong>-v /home/user/db_data:/var/lib/mysql:Z \
</strong><strong>registry.lab.example.com/rhel8/mariadb-105
</strong></code></pre>

Verifica quindi che sia impostato correttamente il contesto SELinux sulla directory `/home/user/db_data` utilizzando il comando `ls` con l'opzione <mark style="color:orange;">`-Z`</mark> :&#x20;

<pre class="language-bash"><code class="lang-bash"><strong>[user@host ~]$ ls -Z /home/user/
</strong>system_u:object_r:container_file_t:s0:c81,c1009 db_data
...output omitted...
</code></pre>

## Avviare un servizio containerizzato al boot usando `systemd`

Gli amministratori configurano applicazioni come server web o database per avviarsi all'avvio e funzionare indefinitamente come un servizio systemd. \
&#xNAN;_**Systemd**_ è uno strumento di gestione dei servizi per Linux. \
Usa file unitari per avviare e fermare le applicazioni o abilitarle all'avvio. \
Con il comando `systemctl`, gli amministratori gestiscono queste applicazioni.

### Generare un file di unitá _systemd_

Per creare un file unità di systemd per un container di servizio specificato, \
utilizza il comando <mark style="color:orange;">`podman generate systemd`</mark>. \
Usa l'opzione <mark style="color:yellow;">`--name`</mark> per specificare il <mark style="color:yellow;">nome del container</mark>. \
L'opzione <mark style="color:green;">`--files`</mark> <mark style="color:green;">genera file</mark> invece di stampare su standard output (STDOUT) :&#x20;

<pre class="language-bash"><code class="lang-bash"><strong>[user@host ~]$ podman generate systemd --name web --files
</strong>/home/user/container-web.service
</code></pre>

Il comando precedente crea il file unità `container-web.service` per un contenitore chiamato `web`. Dopo aver esaminato e adattato la configurazione del servizio generata alle tue esigenze, salva il file unità nella directory di configurazione di systemd dell'utente (`~/.config/systemd/user/`).

Dopo aver aggiunto o modificato i file unità, è necessario usare il comando `systemctl` per ricaricare la configurazione di systemd:

<pre class="language-bash"><code class="lang-bash"><strong>[user@host ~]$ systemctl --user daemon-reload
</strong></code></pre>

### Gestire i servizi containerizzati

Per gestire un servizio containerizzato, utilizza il comando `systemctl` :&#x20;

<pre class="language-bash"><code class="lang-bash"><strong>[user@host ~]$ systemctl --user [start, stop, status, enable, disable] container-web.service
</strong></code></pre>

Utilizzando l'opzione <mark style="color:orange;">`--user`</mark>, per impostazione predefinita, <mark style="color:orange;">systemd avvia il servizio quando effettui l'accesso e lo interrompe quando effettui il logout.</mark> \
Puoi avviare i tuoi servizi abilitati all'avvio del sistema operativo e arrestarli allo spegnimento, eseguendo rispettivamente i comandi <mark style="color:yellow;">`loginctl enable-linger`</mark> e <mark style="color:yellow;">`loginctl disable-linger`</mark> :&#x20;

<pre class="language-bash"><code class="lang-bash"><strong>[user@host ~]$ loginctl enable-linger
</strong></code></pre>

<pre class="language-bash"><code class="lang-bash"><strong>[user@host ~]$ loginctl disable-linger
</strong></code></pre>
