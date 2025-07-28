# Registri delle immagini per i container

## Registri dei container

Un'immagine del container è una versione pacchettizzata della tua applicazione, con tutte le dipendenze necessarie per l'esecuzione dell'applicazione. \
Puoi utilizzare i registri delle immagini per archiviare le immagini dei container e condividerle in modo controllato. \
Alcuni esempi di registri delle immagini includono _**Quay.io**_, _**Red Hat Registry**_, _**Docker Hub**_ e _**Amazon ECR.**_

<pre class="language-bash"><code class="lang-bash"><strong># IMMAGINE PRESA dal RED HAT REGISTRY registry.redhat.io
</strong><strong>
</strong><strong>[user@host ~]$ podman pull registry.redhat.io/ubi9/ubi:9.1
</strong>Trying to pull registry.redhat.io/ubi9/ubi:9.1...
Getting image source signatures
...output omitted...
Writing manifest to image destination
Storing signatures
3434...8f6b
</code></pre>

## Registro Red Hat

Red Hat distribuisce immagini dei container utilizzando due registri:

* <mark style="color:orange;">`registry.access.redhat.com`</mark>: non richiede autenticazione
* <mark style="color:orange;">`registry.redhat.io`</mark>: richiede autenticazione

&#x20;altrimenti dal catalogo Red Hat [https://catalog.redhat.com/](https://catalog.redhat.com/) :&#x20;

<figure><img src="../.gitbook/assets/image (24).png" alt=""><figcaption></figcaption></figure>

esempio info immagine:

<figure><img src="../.gitbook/assets/image (25).png" alt=""><figcaption></figcaption></figure>

### Immagini utili del container

<table><thead><tr><th valign="top">Immagine</th><th valign="top">Fornitori</th><th valign="top">Descrizione</th></tr></thead><tbody><tr><td valign="top"><code>registry.access.redhat.com/ubi9</code></td><td valign="top">Universal Base Image (UBI), Version 9</td><td valign="top">A base image to create other images that is based on RHEL 9.</td></tr><tr><td valign="top"><code>registry.access.redhat.com/ubi9/python-312</code></td><td valign="top">Python 3.12</td><td valign="top">A UBI-based image with the Python 3.12 runtime.</td></tr><tr><td valign="top"><code>registry.access.redhat.com/ubi9/nodejs-18</code></td><td valign="top">Node.js 18</td><td valign="top">A UBI-based image with the Node.js 18 runtime.</td></tr><tr><td valign="top"><code>registry.access.redhat.com/ubi9/go-toolset</code></td><td valign="top">Go Toolset</td><td valign="top">A UBI-based image with the Go runtime. The Go version depends on the image tag.</td></tr></tbody></table>

_**Red Hat UBIs**_ sono immagini container conformi allo standard _OCI_, progettate per applicazioni containerizzate. \
Includono componenti di Red Hat Enterprise Linux e offrono runtime di linguaggio predefiniti. \
Sono liberamente distribuibili su piattaforme sia Red Hat che non, senza bisogno di abbonamenti. Tuttavia, il supporto completo è garantito solo su piattaforme Red Hat.

## Quay.io

Il registro Red Hat memorizza solo immagini da Red Hat e fornitori certificati, ma puoi utilizzare il registro _<mark style="color:orange;">**Quay.io**</mark>_ per memorizzare le tue immagini personalizzate. \
Le immagini pubbliche su Quay.io sono memorizzate gratuitamente, mentre i clienti paganti ricevono vantaggi aggiuntivi, come i repository privati. \
Gli sviluppatori possono anche implementare un'istanza locale di Quay per configurare un registro di immagini nella propria infrastruttura.&#x20;

Per accedere a Quay.io, puoi usare il tuo account developer di Red Hat:

<figure><img src="../.gitbook/assets/image (26).png" alt=""><figcaption></figcaption></figure>

## Gestione Registry con Podman

Quando scarichi un'immagine del container, fornisci diversi dettagli. \
Ad esempio, il nome dell'immagine `registry.access.redhat.com/ubi9/nodejs-18:latest` è composto dalle seguenti informazioni:&#x20;

* Indirizzo del registro: `registry.access.redhat.com`&#x20;
* Utente o organizzazione: `ubi9`&#x20;
* Repository dell'immagine: `nodejs-18`&#x20;
* Tag dell'immagine: `latest`&#x20;

Gli sviluppatori possono specificare un nome abbreviato e non qualificato, che omette l'indirizzo del registro. \
Ad esempio, potresti abbreviare `registry.redhat.io/ubi9/nodejs-18:latest` a `ubi9/nodejs-18:latest` :&#x20;

<pre class="language-bash"><code class="lang-bash"><strong>[user@host ~]$ podman pull ubi9/nodejs-18
</strong></code></pre>

Se non fornisci l'URL del registro, Podman utilizza il file <mark style="color:red;">`/etc/containers/registries.conf`</mark> per cercare altri registri di container che potrebbero contenere il nome dell'immagine. \
Questo file elenca i registri che Podman controlla per trovare l'immagine, in ordine di preferenza.&#x20;

Ad esempio, con la seguente configurazione, Podman ricerca prima nel Registro Red Hat. Se l'immagine non viene trovata nel Registro Red Hat, Podman la cerca nel registro Docker Hub:

```vim
unqualified-search-registries == ['registry.redhat.io', 'docker.io']
```

blocco _pulling_ da docker.io:

```vim
[[registry]]
location="docker.io"
blocked=true
```

Su Microsoft Windows, esegui il comando `podman machine ssh` per connetterti alla macchina virtuale basata su Linux che avvia i tuoi container. \
Nella macchina virtuale, puoi trovare il file `/etc/containers/registries.conf`:

<pre class="language-bash"><code class="lang-bash"><strong>[user@host ~]$ podman machine ssh
</strong><strong>[user@DESKTOP-AA1A111 ~]$ ls /etc/containers/containers.conf
</strong>/etc/containers/containers.conf
</code></pre>

## Gestione Registry con Skopeo

**Skopeo** è uno strumento da riga di comando per lavorare con immagini di container. \
Gli sviluppatori possono usare Skopeo per diverse attività:

* Ispezionare immagini di container remote.
* Copiare un'immagine di container tra registri.
* Firmare un'immagine con chiavi OpenPGP.
* Convertire formati di immagini, ad esempio da `Docker` a formato `OCI`.

Skopeo può ispezionare immagini remote o trasferirle tra registri senza usare lo storage locale. \
Il comando <mark style="color:orange;">`skopeo`</mark> utilizza il formato `transport:image`, come `docker://immagine_remota`, `dir:percorso`, o `oci:percorso:tag`.&#x20;

<pre class="language-bash"><code class="lang-bash"><strong>[user@host ~]$ skopeo inspect docker://registry.access.redhat.com/ubi9/nodejs-18
</strong>{
    "Name": "registry.access.redhat.com/ubi9/nodejs-18",
    "Digest": "sha256:741b...22e0",
    "RepoTags": [
...output omitted...
</code></pre>

<pre class="language-bash"><code class="lang-bash"><strong># COPIARE IMMAGINI TRA REGISTRY
</strong><strong>
</strong><strong>[user@host ~]$ skopeo copy docker://registry.access.redhat.com/ubi9/nodejs-18 \
</strong><strong> docker://quay.io/myuser/nodejs-18
</strong>Getting image source signatures
...output omitted...
</code></pre>

<pre class="language-bash"><code class="lang-bash"><strong># CAMBIO DEL FORMATO DI TRASPORTO PER SCARICARE IN LOCALE L'IMMAGINE
</strong><strong>
</strong><strong>[user@host ~]$ skopeo copy docker://registry.access.redhat.com/ubi9/nodejs-18 \
</strong><strong> dir:/var/lib/images/nodejs-18
</strong>Getting image source signatures
...output omitted...
</code></pre>

Usa il comando `skopeo inspect` per leggere i metadati di un'immagine.

## Gestione credenziali dei registry con Podman

Alcuni registri richiedono agli utenti di autenticarsi, come il registro `registry.redhat.io` :&#x20;

<pre class="language-bash"><code class="lang-bash"><strong>[user@host ~]$ podman pull registry.redhat.io/rhel9/nginx-120
</strong>Trying to pull registry.redhat.io/rhel9/nginx-120:latest...
Error: initializing source docker://registry.redhat.io/ubi9/httpd-24:latest: unable to retrieve auth token: invalid username/password: unauthorized: Please login to the Red Hat Registry using your Customer Portal credentials. Further instructions can be found here: https://access.redhat.com/RegistryAuthentication
</code></pre>

Puoi scegliere un'immagine diversa che non richiede autenticazione, come l'immagine UBI 9 dal registro `registry.access.redhat.com:`

<pre class="language-bash"><code class="lang-bash"><strong>[user@host ~]$ podman pull registry.access.redhat.com/ubi9:latest
</strong>Trying to pull registry.access.redhat.com/ubi9:latest...
Getting image source signatures
Checking if image destination supports signatures
...output omitted...
</code></pre>

Altro comando di autenticazione `podman login` :&#x20;

<pre class="language-bash"><code class="lang-bash"><strong>[user@host ~]$ podman login registry.redhat.io
</strong><strong>Username: YOUR_USER
</strong><strong>Password: YOUR_PASSWORD
</strong>Login Succeeded!
<strong>[user@host ~]$ podman pull registry.redhat.io/rhel9/nginx-120
</strong>Trying to pull registry.redhat.io/rhel9/nginx-120:latest...
Getting image source signatures
...output omitted...
</code></pre>

Podman salva le credenziali nel file <mark style="color:orange;">`${XDG_RUNTIME_DIR}/containers/auth.json`</mark>, dove `${XDG_RUNTIME_DIR}` si riferisce ad una specifica directory dello user corrente. \
Le credenziali sono codificate nel formato `base64` :

<pre class="language-bash"><code class="lang-bash"><strong>[user@host ~]$ cat ${XDG_RUNTIME_DIR}/containers/auth.json
</strong>{
	"auths": {
		"registry.redhat.io": {
			"auth": "dXNlcjpodW50ZXIy"
		}
	}
}
<strong>[user@host ~]$ echo -n dXNlcjpodW50ZXIy | base64 -d
</strong>user:hunter2
</code></pre>

Skopeo utilizza lo stesso file `${XDG_RUNTIME_DIR}/containers/auth.json` per accedere ai dettagli di autenticazione per ogni registro.
