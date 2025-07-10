# Controllo dei contesti dei file SELinux

## Contesto SELinux iniziale

Tutte le risorse, come processi, file e porte, sono etichettati con un _<mark style="color:yellow;">contesto</mark>_ <mark style="color:yellow;"></mark><mark style="color:yellow;">SELinux</mark>. \
SELinux gestisce un database di politiche di etichettatura dei file nella directory `etc/selinux/targeted/contexts/files/`.&#x20;

I nuovi file ottengono un'etichetta predefinita se il loro nome corrisponde a una politica esistente. Se non c'è corrispondenza, il file eredita l'etichetta della directory padre. \
Quando i file vengono creati in posizioni coperte da politiche, ottengono il contesto corretto. \
In posizioni senza politica, invece, l'etichetta ereditata potrebbe non essere corretta.&#x20;

#### Nota

Copiare un file può cambiare il contesto basato sulla nuova posizione, ma può essere preservato con `cp --preserve=context`. \
Usando `mv` in uno stesso file system mantiene il contesto. Dopo copia o spostamento, verificare sempre l'etichetta SELinux e correggerla se necessario.

### Funzionamento

* Creazione di due file nella directory `/tmp`.&#x20;
* Entrambi i file ricevono il tipo di contesto `user_tmp_t`.&#x20;
* Sposta il primo file e copia il secondo file nella directory `/var/www/html`.&#x20;
  * Il file <mark style="color:orange;">spostato</mark> - <mark style="color:orange;">mantiene il contesto</mark> file etichettato dalla directory originale `/tmp`.&#x20;
  * Il file <mark style="color:yellow;">copiato</mark> - ha un nuovo _inode_ ed <mark style="color:yellow;">eredita il contesto SELinux dalla destinazione</mark> nella directory `/var/www/html`.&#x20;
* Il comando `ls -Z` <mark style="color:green;">visualizza il contesto SELinux</mark> di un file. Osserva l'etichetta dei file creati nella directory `/tmp`.

<pre class="language-bash"><code class="lang-bash"><strong>[root@host ~]\# touch /tmp/file1 /tmp/file2
</strong><strong>[root@host ~]\# ls -Z /tmp/file*
</strong>unconfined_u:object_r:user_tmp_t:s0 /tmp/file1
unconfined_u:object_r:user_tmp_t:s0 /tmp/file2
</code></pre>

### Mostrare il contesto SELinux di una specifica directory con l'opzione `-Zd`

<pre class="language-bash"><code class="lang-bash"><strong>[root@host ~]\# ls -Zd /var/www/html/
</strong>system_u:object_r:httpd_sys_content_t:s0 /var/www/html/
<strong>[root@host ~]\# ls -Z /var/www/html/index.html
</strong>unconfined_u:object_r:httpd_sys_content_t:s0 /var/www/html/index.html
</code></pre>

### File spostati

<pre class="language-bash"><code class="lang-bash"><strong>[root@host ~]\# mv /tmp/file1 /var/www/html/
</strong><strong>[root@host ~]\# cp /tmp/file2 /var/www/html/
</strong><strong>[root@host ~]\# ls -Z /var/www/html/file*
</strong>unconfined_u:object_r:user_tmp_t:s0 /var/www/html/file1
unconfined_u:object_r:httpd_sys_content_t:s0 /var/www/html/file2
</code></pre>

## Modifica del contesto SELinux

### Comandi principali

* `semanage fcontext`: Crea politiche di contesto per i file.
* `restorecon`: Applica i contesti definiti dalle politiche SELinux.
* `chcon`: Modifica direttamente il contesto senza riferimento alle politiche.

#### Metodo consigliato

1. Creare una politica di contesto con `semanage fcontext`.
2. Applicare il contesto usando `restorecon`.

#### Note importanti

* Le modifiche fatte con `chcon` sono temporanee e possono essere sovrascritte da `restorecon`.
* Un _relabel_ riapplica i contesti predefiniti a tutto il sistema.

### Creazione directory con contesto SELinux `default_t` ereditata dalla parent dir

<pre class="language-bash"><code class="lang-bash"><strong>[root@host ~]\# mkdir /virtual
</strong><strong>[root@host ~]\# ls -Zd /virtual
</strong>unconfined_u:object_r:default_t:s0 /virtual
</code></pre>

cambio contesto con `chcon` :&#x20;

<pre class="language-bash"><code class="lang-bash"><strong>[root@host ~]\# chcon -t httpd_sys_content_t /virtual
</strong><strong>[root@host ~]\# ls -Zd /virtual
</strong>unconfined_u:object_r:httpd_sys_content_t:s0 /virtual
</code></pre>

resetto il contesto al valore iniziale con `restorecon` :&#x20;

<pre class="language-bash"><code class="lang-bash"><strong>[root@host ~]\# restorecon -v /virtual
</strong><a data-footnote-ref href="#user-content-fn-1">Relabeled</a> /virtual from unconfined_u:object_r:httpd_sys_content_t:s0 to unconfined_u:object_r:default_t:s0
<strong>[root@host ~]\# ls -Zd /virtual
</strong>unconfined_u:object_r:default_t:s0 /virtual
</code></pre>

## Definire le policies di default del contesto dei file SELinux

Il comando `semanage fcontext` permette di visualizzare e modificare le politiche di contesto dei file di default.&#x20;

Utilizzando `semanage fcontext -l`, è possibile elencare tutte le regole delle politiche di contesto dei file, che usano sintassi di espressioni regolari estese per specificare percorsi e nomi di file. Un'espressione comune è `(/.*)?`, nota come _pirata_ in modo scherzoso, poiché assomiglia a un viso con una benda sull'occhio e un uncino al posto della mano. \
Questa sintassi corrisponde a una directory e a tutti i file in essa creati, incluse le sue sottodirectory. \
Per esempio, la regola `/var/www/cgi-bin(/.*)?` assegna il contesto `system_u:object_r:httpd_sys_script_exec_t:s0` alla directory specificata e a tutti i file al suo interno, a meno che una regola più specifica non la sovrasti.&#x20;

```tsconfig
/var/www/cgi-bin(/.*)?  all files  system_u:object_r:httpd_sys_script_exec_t:s0
```

### Operazioni di base sui contesti dei file

`semanage fcontext` :&#x20;

<table><thead><tr><th valign="top">Opzione</th><th valign="top">Descrizione</th></tr></thead><tbody><tr><td valign="top"><code>-a, --add</code></td><td valign="top">aggiungi un record al tipo di oggetto specifico</td></tr><tr><td valign="top"><code>-d, --delete</code></td><td valign="top">cancella un record al tipo di oggetto specifico</td></tr><tr><td valign="top"><code>-l, --list</code></td><td valign="top">mostra la lista dei record del tipo di oggetto specifico</td></tr></tbody></table>

Per gestire i contesti SELinux, installa i pacchetti `policycoreutils` e `policycoreutils-python-utils`, che contengono i comandi `restorecon` e `semanage`.

1.  Verifica contesto del file (`ls`` `<mark style="color:orange;">`-Z`</mark>):

    <pre class="language-bash"><code class="lang-bash"><strong>[root@host ~]\# ls -Z /var/www/html/file*
    </strong>unconfined_u:object_r:user_tmp_t:s0 /var/www/html/file1
    unconfined_u:object_r:httpd_sys_content_t:s0 /var/www/html/file2
    </code></pre>
2.  Mostra i contesti di default dei file SELinux con <mark style="color:orange;">`semanage`</mark>` ``fcontext -l` :

    <pre class="language-bash"><code class="lang-bash"><strong>[root@host ~]\# semanage fcontext -l
    </strong>...output omitted...
    /var/www(/.*)?       all files    system_u:object_r:httpd_sys_content_t:s0
    ...output omitted...
    </code></pre>
3.  Ripristina il contesto predefinito su tutti i file e le sottodirectory con <mark style="color:orange;">`restorecon`</mark>` ``-Rv path`:

    <pre class="language-bash"><code class="lang-bash"><strong>[root@host ~]\# restorecon -Rv /var/www/
    </strong>Relabeled /var/www/html/file1 from unconfined_u:object_r:user_tmp_t:s0 to unconfined_u:object_r:httpd_sys_content_t:s0
    <strong>[root@host ~]\# ls -Z /var/www/html/file*
    </strong>unconfined_u:object_r:httpd_sys_content_t:s0 /var/www/html/file1
    unconfined_u:object_r:httpd_sys_content_t:s0 /var/www/html/file2
    </code></pre>

Aggiungere una policy di contesto per una nuova directory.

1.  Creazione dir (`/virtual`)e file (`index.html`) e mostra contesti SELinux:&#x20;

    <pre class="language-bash"><code class="lang-bash"><strong>[root@host ~]\# mkdir /virtual
    </strong><strong>[root@host ~]\# touch /virtual/index.html
    </strong><strong>[root@host ~]\# ls -Zd /virtual/
    </strong>unconfined_u:object_r:default_t:s0 /virtual
    <strong>[root@host ~]\# ls -Z /virtual/
    </strong>unconfined_u:object_r:default_t:s0 index.html
    </code></pre>
2.  Aggiunta di una policy di contesto SELinux al file nella directory:

    <pre class="language-bash"><code class="lang-bash"><strong>[root@host ~]\# semanage fcontext -a -t httpd_sys_content_t '/virtual(/.*)?'
    </strong></code></pre>
3.  Ripristina il contesto predefinito su tutti i file e le sottodirectory con <mark style="color:orange;">`restorecon`</mark>` ``-RFvv path`:

    <pre class="language-bash"><code class="lang-bash"><strong>[root@host ~]\# restorecon -RFvv /virtual
    </strong>Relabeled /virtual from unconfined_u:object_r:default_t:s0 to system_u:object_r:httpd_sys_content_t:s0
    Relabeled /virtual/index.html from unconfined_u:object_r:default_t:s0 to system_u:object_r:httpd_sys_content_t:s0
    <strong>[root@host ~]\# ls -Zd /virtual/
    </strong>drwxr-xr-x. root root system_u:object_r:httpd_sys_content_t:s0 /virtual/
    <strong>[root@host ~]\# ls -Z /virtual/
    </strong>-rw-r--r--. root root system_u:object_r:httpd_sys_content_t:s0 index.html
    </code></pre>
4.  Visualizza eventuali personalizzazioni locali alla policy di default:

    <pre class="language-bash"><code class="lang-bash"><strong>[root@host ~]\# semanage fcontext -l -C
    </strong>SELinux fcontext     type         Context

    /virtual(/.*)?       all files    system_u:object_r:httpd_sys_content_t:s0
    </code></pre>





[^1]: notare che viene specificato
