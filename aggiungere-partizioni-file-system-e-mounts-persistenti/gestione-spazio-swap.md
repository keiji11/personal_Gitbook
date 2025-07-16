# Gestione spazio SWAP

## Concetto di spazio SWAP

Lo _<mark style="color:orange;">spazio di swap</mark>_ è un'area del disco gestita dal sottosistema di gestione della memoria del kernel Linux. \
Serve a integrare la RAM del sistema, ospitando pagine di memoria inattive. \
La _<mark style="color:orange;">memoria virtuale</mark>_ di un sistema include la <mark style="color:yellow;">RAM</mark> e lo <mark style="color:yellow;">spazio di swap</mark>.&#x20;

Quando l'utilizzo della memoria supera un certo limite, il kernel cerca pagine inattive nella RAM, le scrive nell'area di swap e riassegna le pagine RAM libere a nuovi processi. \
Se un programma necessita di una pagina specifica, il kernel la recupera dall'area di swap, rendendo lo swap più lento rispetto alla RAM.&#x20;

<mark style="color:red;">Lo swap non è una soluzione sostenibile per la mancanza di RAM</mark>.

### Calcolo dello spazio SWAP

<table><thead><tr><th valign="top">RAM</th><th valign="top">Swap space</th><th width="292.727294921875" valign="top">Swap space se consentita l'hibernation</th></tr></thead><tbody><tr><td valign="top">2 GB o meno</td><td valign="top">2 volte la RAM</td><td valign="top">3 volte la RAM</td></tr><tr><td valign="top">Tra 2 GB e 8 GB</td><td valign="top">come la RAM</td><td valign="top">2 volte la RAM</td></tr><tr><td valign="top">Tra 8 GB e 64 GB</td><td valign="top">Almeno 4 GB</td><td valign="top">1.5 volte la RAM</td></tr><tr><td valign="top">Piú di 64 GB</td><td valign="top">Almeno 4 GB</td><td valign="top">Hibernation non raccomandata</td></tr></tbody></table>

La funzione di ibernazione dei laptop e dei desktop utilizza lo spazio di swap per salvare i contenuti della RAM prima di spegnere il sistema. \
Quando riaccendi il sistema, il kernel ripristina i contenuti della RAM dallo spazio di swap e non ha bisogno di un avvio completo. _Per questi sistemi, lo spazio di swap deve essere maggiore della quantità di RAM._

## Creazione spazio SWAP

Per creare uno spazio di swap, eseguire i seguenti passaggi:&#x20;

* Crea una partizione con un tipo di file-system <mark style="color:orange;">`linux-swap`</mark>.&#x20;
* Posiziona una _<mark style="color:orange;">firma di swap</mark>_ sul dispositivo.

### Creare una partizione di swap

<pre class="language-bash"><code class="lang-bash"><strong># Creazione partizione da 256 MB
</strong><strong>[root@host ~]\# parted /dev/vdb
</strong>GNU Parted 3.4
Using /dev/vdb
Welcome to GNU Parted! Type 'help' to view a list of commands.
<strong>(parted) print
</strong>Model: Virtio Block Device (virtblk)
Disk /dev/vdb: 5369MB
Sector size (logical/physical): 512B/512B
Partition Table: gpt
Disk Flags:

Number  Start   End     Size    File system  Name  Flags
 1      1049kB  1001MB  1000MB               data

<strong>(parted) mkpart
</strong><strong>Partition name?  []? swap1
</strong><strong>File system type?  [ext2]? linux-swap
</strong><strong>Start? 1001MB
</strong><strong>End? 1257MB
</strong><strong>(parted) print
</strong>Model: Virtio Block Device (virtblk)
Disk /dev/vdb: 5369MB
Sector size (logical/physical): 512B/512B
Partition Table: gpt
Disk Flags:

Number  Start   End     Size    File system     Name   Flags
 1      1049kB  1001MB  1000MB                  data
 2      1001MB  1257MB  256MB   linux-swap(v1)  swap1

<strong>(parted) quit
</strong>Information: You may need to update /etc/fstab.
</code></pre>

<pre class="language-bash"><code class="lang-bash"><strong>[root@host ~]\# udevadm settle
</strong></code></pre>

### Formattare lo spazio SWAP

Il comando <mark style="color:orange;">`mkswap`</mark> applica una _<mark style="color:orange;">firma di swap</mark>_ al dispositivo. \
A differenza di altre utility di formattazione, il comando `mkswap` scrive un singolo blocco di dati all'inizio del dispositivo e lascia il resto del dispositivo non formattato, consentendo al kernel di usarlo per memorizzare le pagine di memoria.

<pre class="language-bash"><code class="lang-bash"><strong>[root@host ~]\# mkswap /dev/vdb2
</strong>Setting up swapspace version 1, size = 244 MiB (255848448 bytes)
no label, UUID=39e2667a-9458-42fe-9665-c5c854605881
</code></pre>

### Attivare lo spazio SWAP

Usa <mark style="color:orange;">`swapon`</mark> con il dispositivo come parametro, oppure utilizza `swapon -a` per attivare tutti gli spazi di swap elencati nel file `/etc/fstab`. \
Utilizza i comandi `swapon --show` e `free` per controllare gli spazi di swap disponibili.

<pre class="language-bash"><code class="lang-bash"><strong>[root@host ~]\# free
</strong>              total        used        free      shared  buff/cache   available
Mem:        1873036      134688     1536436       16748      201912     1576044
Swap:             0           0           0
<strong>[root@host ~]\# swapon /dev/vdb2
</strong><strong>[root@host ~]\# free
</strong>              total        used        free      shared  buff/cache   available
Mem:        1873036      135044     1536040       16748      201952     1575680
Swap:        249852           0      249852
</code></pre>

Puoi disattivare uno spazio di swap con il comando <mark style="color:orange;">`swapoff`</mark>. \
\- Se le pagine sono scritte nello spazio di swap, il comando `swapoff` cerca di spostarle in altri spazi di swap attivi o di riportarle nella memoria. \
\- Se il comando `swapoff` non riesce a scrivere dati in altri posti, fallisce con un errore e lo spazio di swap rimane attivo.

### Attivare lo spazio SWAP in modo persistente

Creare un' entry in `/etc/fstab` per assicurare uno spazio di swap all'avvio del sistema:

{% code fullWidth="true" %}
```bash
# riga aggiunta al file /etc/fstab
# UUID o nome_device                PUNTO_DI_MONTAGGIO     TIPO_fs    OPTIONS    DUMP    fsck_ORDER
UUID=39e2667a-9458-42fe-9665-c5c854605881   swap           swap       defaults   0         0
```
{% endcode %}

dopo aver modificato il file `/etc/fstab` riavviare :&#x20;

```bash
[root@host ~]\# systemctl daemon-reload
```

### Impostare la prioritá dello spazio SWAP

La priorità predefinita per gli spazi di swap è bassa e i nuovi spazi di swap hanno priorità inferiore rispetto a quelli più vecchi. \
Quando gli spazi di swap hanno la stessa priorità, il kernel scrive su di essi in modo circolare. Per impostare la priorità, utilizzare l'opzione <mark style="color:orange;">`pri`</mark> nel file `/etc/fstab`. \


Il kernel utilizza per primo lo spazio di swap con la priorità più alta. La priorità predefinita è `-2`.

```bash
# TERZA ENTRY -> prioritá piú alta
UUID=af30cbb0-3866-466a-825a-58889a49ef33   swap   swap   defaults  0 0
UUID=39e2667a-9458-42fe-9665-c5c854605881   swap   swap   pri=4     0 0
UUID=fbd7fa60-b781-44a8-961b-37ac3ef572bf   swap   swap   pri=10    0 0
```
