# Storage con accesso di rete condiviso

## Gestione Storage di rete condiviso con _NFS_

### Accesso alle directory NFS esportate

Il _<mark style="color:orange;">Network File System</mark>_ <mark style="color:orange;"></mark><mark style="color:orange;">(NFS)</mark> è un protocollo standard utilizzato da Linux, UNIX e sistemi operativi simili per il file system di rete. \
NFS è uno standard aperto che supporta i permessi nativi di Linux. \
Red Hat Enterprise Linux 9 utilizza NFS versione 4.2 di default, e supporta sia NFSv3 che NFSv4. \
NFSv3 può usare TCP o UDP, mentre NFSv4 supporta solo TCP. \
&#xNAN;_<mark style="color:yellow;">I server NFS esportano directory, mentre i client NFS le montano.</mark>_

I metodi per montare le directory includono:

* **Manuale:** con il comando `mount`.
* **Persistente all'avvio:** configurando `/etc/fstab`.
* **Su richiesta:** usando metodi automounter, come `autofs` o `systemd.automount`.

È necessario il pacchetto <mark style="color:orange;">`nfs-utils`</mark> per i client NFS.

### Interrogare (fare delle query)le directory NFS esportate di un server

Il server risponde con la porta 111 per il servizio NFS. \
Il comando <mark style="color:orange;">`showmount`</mark> serve ad interrogare gli export disponibili su un server NFSv3 basato su RPC.

<pre class="language-bash"><code class="lang-bash"><strong>[root@host ~]\# showmount --exports server
</strong>
Export list for server
/shares/test1
/shares/test2
</code></pre>

Usa il comando `showmount` su un server che supporta solo NFSv4, perché il servizio `rpcbind` non sta girando sul server. Interrogare il server NFSv4 é piú semplice che per il server NFSv3.

<pre class="language-bash"><code class="lang-bash"><strong>[root@host ~]\# mkdir /mountpoint
</strong><strong>[root@host ~]\# mount server:/ /mountpoint
</strong><strong>[root@host ~]\# ls /mountpoint
</strong></code></pre>

### Montare manualmente directory NFS esportate

<pre class="language-bash"><code class="lang-bash"><strong>[root@host ~]\# mkdir /mountpoint
</strong><strong>
</strong><strong>[root@host ~]\# mount -t nfs -o rw,sync server:/export /mountpoint
</strong></code></pre>

Montare manualmente é temporaneo, per rendere permanente modificare il file `/etc/fstab`&#x20;

### Montare directory NFS esportate in modo permanente

<pre class="language-bash"><code class="lang-bash"><strong>[root@host ~]\# vim /etc/fstab
</strong>...
server:/export  /mountpoint  nfs  rw  0 0
</code></pre>

In seguito montare la directory NFS:

<pre class="language-bash"><code class="lang-bash"><strong>[root@host ~]\# mount /mountpoint
</strong></code></pre>

### Smontare directory NFS esportate

Smontaggio provvisorio:

<pre class="language-bash"><code class="lang-bash"><strong>[root@host ~]\# umount /mountpoint
</strong></code></pre>

Se fallisce controllare che tutti i file o processi nella directory siano attivi, e se sí, disattivarli:

<pre class="language-bash"><code class="lang-bash"><strong>[root@host ~]\# lsof  /mountpoint
</strong>COMMAND  PID   USER  FD   TYPE  DEVICE  SIZE/OFF  NODE  NAME
program  5534  user  txt  REG   252.4   910704    128   /home/user/program
</code></pre>

oppure forzare l'`umount` ( <mark style="color:orange;">`-f`</mark> ) ma cancella il processo in esecuzione sul file:

<pre class="language-bash"><code class="lang-bash"><strong>[root@host ~]\# umount -f /mountpoint
</strong></code></pre>
