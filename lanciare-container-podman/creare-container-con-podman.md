# Creare container con Podman

## Introduzione a Podman

Podman è uno strumento open-source per gestire i container localmente. \


Consente di **trovare**, **eseguire**, **costruire** e **distribuire** container e immagini OCI senza l'uso di un daemon, che può rappresentare un punto di fallimento e una problema per la sicurezza. \
&#xNAN;_<mark style="color:red;">**Podman**</mark>_ funziona tramite una <mark style="color:yellow;">CLI</mark>, un' <mark style="color:yellow;">API RESTful</mark> e un'applicazione desktop chiamata _<mark style="color:yellow;">Podman Desktop</mark>_.

## Lavorare con <mark style="color:red;">Podman</mark>

<pre class="language-bash"><code class="lang-bash"><strong># VISULIZZARE VERSIONE di PODMAN
</strong><strong>
</strong><strong>[user@host ~]$ podman -v
</strong>podman version VERSION
</code></pre>

### _Pulling_ e Visualizzare Immagini

Con Podman, recuperi le immagini dei container dai registri delle immagini utilizzando il comando `podman`` `<mark style="color:orange;">`pull`</mark>. \
Ad esempio, il seguente comando recupera una versione containerizzata di Red Hat Enterprise Linux 9 dal Red Hat Registry:

<pre class="language-bash"><code class="lang-bash"><strong>[user@host ~]$ podman pull registry.redhat.io/rhel9/rhel-guest-image:9.4
</strong>Trying to pull registry.redhat.io/rhel9/rhel-guest-image:9.4...
Getting image source signatures
...output omitted...
Writing manifest to image destination
Storing signatures
b85986059f7663c1b89431f74cdcb783f6540823e4b85c334d271f2f2d8e06d6
</code></pre>

Dopo aver eseguito il comando `pull`, l'immagine è memorizzata localmente nel tuo sistema. \
Puoi elencare le immagini nel tuo sistema utilizzando il comando `podman`` `<mark style="color:orange;">`images`</mark> :

<pre class="language-bash"><code class="lang-bash"><strong>[user@host ~]$ podman images
</strong>REPOSITORY                        TAG         IMAGE ID      CREATED       SIZE
registry.redhat.io/rhel9/rhel-guest-image     9.4         52617ef413bd  2 weeks ago   216 MB
</code></pre>

### Container ed immagini container

Un container è un ambiente di runtime isolato dove le applicazioni vengono eseguite come processi isolati. \
L'isolamento dell'ambiente di runtime garantisce che non interrompano altri container o processi di sistema. \
Un'immagine del container contiene una versione confezionata della tua applicazione, con tutte le dipendenze necessarie per l'esecuzione dell'applicazione. \
Le immagini possono esistere senza container, ma _i container dipendono dalle immagini perché le utilizzano per costruire un ambiente di runtime per eseguire le applicazioni_.

### Lanciare e Visualizzare container

<mark style="color:orange;">`run`</mark> (<mark style="color:orange;">puó essere eseguito anche senza il pull dell'immagine, la</mark> <mark style="color:orange;"></mark>_<mark style="color:orange;">pullerá</mark>_ <mark style="color:orange;"></mark><mark style="color:orange;">automaticamente se non esistente</mark>):&#x20;

<pre class="language-bash"><code class="lang-bash"><strong>[user@host ~]$ podman run registry.redhat.io/rhel9/rhel-guest-image:9.4 \
</strong><strong>echo 'Red Hat'
</strong>Red Hat
</code></pre>

Quando il container termina l'esecuzione del comando `echo`, si arresta perché nessun altro processo lo mantiene attivo. \
Puoi elencare i container in esecuzione utilizzando il comando <mark style="color:orange;">`podman ps`</mark> :&#x20;

<pre class="language-bash"><code class="lang-bash"><strong>[user@host ~]$ podman ps
</strong>CONTAINER ID  IMAGE       COMMAND     CREATED     STATUS      PORTS       NAMES
</code></pre>

<pre class="language-bash"><code class="lang-bash"><strong># MOSTRA TUTTI i CONTAINER, ATTIVI e STOPPATI
</strong><strong>
</strong><strong>[user@host ~]$ podman ps --all
</strong>CONTAINER ID  IMAGE                              COMMAND       CREATED       STATUS                   PORTS       NAMES
20236410bcef  registry.redhat.io/rhel9/rhel-guest-image:9.4  echo Red Hat  1 second ago  Exited (0) 1 second ago              hungry_mclaren
</code></pre>

<pre class="language-bash"><code class="lang-bash"><strong># ELIMINA IL CONTAINER con --rm DOPO AVERLO CREATO al momento in cui SI STOPPA
</strong><strong>
</strong><strong>[user@host ~]$ podman run --rm registry.redhat.io/rhel9/rhel-guest-image:9.4 echo 'Red Hat'
</strong>Red Hat
<strong>[user@host ~]$ podman ps --all
</strong>CONTAINER ID  IMAGE                              COMMAND       CREATED       STATUS                   PORTS       NAMES
</code></pre>

<pre class="language-bash"><code class="lang-bash"><strong># ASSEGNAZIONE NOME CONTIANER PERSONALIZZATO
</strong><strong>
</strong><strong>[user@host ~]$ podman run --name podman_rhel9 \
</strong><strong>registry.redhat.io/rhel9/rhel-guest-image:9.4 echo 'Red Hat'
</strong>Red Hat
</code></pre>

<pre class="language-bash"><code class="lang-bash"><strong># RECUPERARE INFO SPEECIFICHE DEL CONTAINER
</strong><strong>
</strong><strong>[user@host ~]$ podman ps --all --format=json
</strong>{
    "AutoRemove": false,
    "Command": [
      "echo",
      "Red Hat"
    ],
...output omitted...
<strong>    "Id": "2024...b0b2", # UUID del container
</strong>    "Image": "registry.redhat.io/rhel9/rhel-guest-image:9.4",
...output omitted...
</code></pre>

### Esporre i container

Molte applicazioni, come server web o database, operano in modo continuo per gestire connessioni. Pertanto, i container per queste applicazioni devono anche funzionare continuamente.&#x20;

Di solito è fondamentale che tali applicazioni siano accessibili esternamente tramite un protocollo di rete. \
Puoi utilizzare l'opzione <mark style="color:orange;">`-p`</mark> per mappare una porta nel tuo computer locale a una porta all'interno del container. \
Questo permette al traffico nella tua porta locale di essere inoltrato alla porta nel container, consentendoti di accedere all'applicazione dal tuo computer. \


Ad esempio, per avviare un container con Apache HTTP server, puoi mappare la porta 8080 locale alla porta 8080 del container:

<pre class="language-bash"><code class="lang-bash"><strong># MAPPARE IL CONTAINER SU UNA PORTA locale 8080
</strong><strong>
</strong><strong>[user@host ~]$ podman run -p 8080:8080 \
</strong><strong> registry.access.redhat.com/ubi9/httpd-24:latest
</strong>...output omitted...
[Thu Jun 18 12:58:57.048491 2024] [ssl:warn] [pid 1:tid 140259.33613248] AH01909: 10.0.2.100:8443:0 server certificate does NOT include an ID which matches the server name
[Thu Jun 18 12:58:57.048899 2024] [:notice] [pid 1:tid 140259.33613248] ModSecurity for Apache/2.9.2 (http://www.modsecurity.org/) configured.
...output omitted...
[Thu Jun 18 12:58:57.136272 2024] [mpm_event:notice] [pid 1:tid 140259.33613248] AH00489: Apache/2.4.37 (Red Hat Enterprise Linux) OpenSSL/1.1.1k configured -- resuming normal operations
[Thu Jun 18 12:58:57.136332 2024] [core:notice] [pid 1:tid 140259.33613248] AH00094: Command line: 'httpd -D FOREGROUND'
</code></pre>

<pre class="language-bash"><code class="lang-bash"><strong># LANCIARE IL CONTAINER IN BACKGROUND con l'opzione -d
</strong><strong>
</strong><strong>[user@host ~]$ podman run -d -p 8080:8080 registry.access.redhat.com/ubi9/httpd-24:latest
</strong>b7eb467781106e4f416ba79cede91152239bfc74f6a570c6d70baa4c64fa636a
</code></pre>

### Utilizzo variabili d'ambiente

Le variabili d'ambiente sono variabili utilizzate nelle applicazioni, impostate al di fuori del programma. \
Il sistema operativo o l'ambiente di esecuzione fornisce il valore. \
Sono utili per configurare applicazioni in base all'ambiente, ad esempio con host database specifici.&#x20;

Per passarle a un container, si usa l'opzione <mark style="color:orange;">`-e`</mark>.&#x20;

Ad esempio, si passa la variabile <mark style="color:orange;">`NAME`</mark> con il valore `Red Hat` e la si stampa con <mark style="color:orange;">`printenv`</mark> all'interno del container:

<pre class="language-bash"><code class="lang-bash"><strong>[user@host ~]$ podman run -e NAME='Red Hat' \
</strong><strong>registry.redhat.io/rhel9/rhel-guest-image:9.4 printenv NAME
</strong>Red Hat
</code></pre>

## Podman Desktop

_<mark style="color:orange;">Podman Desktop</mark>_ è una GUI per gestire e interagire con i container in ambienti locali. \
Utilizza il motore Podman di default, supportando anche altri motori come Docker. \
Al lancio, mostra una dashboard che visualizza lo stato del motore Podman, segnalando eventuali avvisi o errori, ad esempio, in caso di mancata installazione del motore Podman o problemi di compatibilità Docker.

<figure><img src="../.gitbook/assets/image.png" alt=""><figcaption></figcaption></figure>

con Podman Desktop é possibile _creare_, _elencare_ ed _eseguire_ container dalla sezione _Container_, come mostrato nella seguente immagine:

<figure><img src="../.gitbook/assets/image (1).png" alt=""><figcaption></figcaption></figure>

Sezione Immagini podman:

<figure><img src="../.gitbook/assets/image (2).png" alt=""><figcaption></figcaption></figure>

_Podman Desktop_ è un'aggiunta al CLI di `podman`, non una sostituzione. \
È utile per chi preferisce ambienti grafici per compiti comuni o per i principianti che imparano sui container. \
È modulare ed estensibile, con estensioni come quella per _<mark style="color:orange;">Red Hat OpenShift</mark>_, che permette il deploy su _Red Hat OpenShift Container Platform_. \
Disponibile per Linux, MacOS e Windows, per l'installazione specifica consultare la documentazione.
