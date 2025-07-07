# Archiviare e trasferire file

## Gestione archivi tar compressi

### Utilizzo del comando `tar` su Linux

Su Linux, l'utilità `tar` è il comando comune per creare, gestire ed estrarre archivi. Usa il comando `tar` per raccogliere più file in un unico file di archivio. \
Un _archivio tar_ è una sequenza strutturata di metadati e dati dei file con un indice, in modo da poter estrarre singoli file. \
I file possono essere compressi durante la creazione utilizzando uno degli algoritmi di compressione supportati. \
Il comando `tar` può elencare il contenuto di un archivio senza estrarlo, e può estrarre i file originali direttamente da archivi compressi e non compressi.

### Opzioni per il comando `tar` :&#x20;

* **`-c`** o **`--create`** : creazione file di archivio.
* **`-t`** o **`--list`** : lista contenuto dell'archivio.
* **`-x`** o **`--extract`** : estrazione archivio.

Secondarie:

* **`-v`** o **`--verbose`** : mostra i file che sono stati archiviati o estratti durante l'operazione `tar`
* **`-f`** o **`--file`** : Segui questa opzione con il nome del file di archivio da creare o aprire.
* **`-p`** o **`--preserve-permissions`** : preserva i permessi del file originale quando estrai.
* **`--xattrs`** : abilita supporto attributo esteso.
* **`--selinux`** : abilita contesto di supporto SELinux.

Opzioni per selezione algoritmo:

* **`-a`** o **`--auto-compress`** : algoritmo di default.
* **`-z`** o **`--gzip`** : Usa l'algoritmo di compressione `gzip` , con suffisso `.tar.gz`.
* **`-j`** o **`--bzip2`** : Usa l'algoritmo di compressione  `bzip2` , con suffisso `.tar.bz2`.
* **`-J`** o **`--xz`** : Usa l'algoritmo di compressione `xz` , con suffisso `.tar.xz`.

### Creazione archivio

```bash
tar [options] [archive name] [file(s)_to_compress]

[user@host ~]$ tar -cf mybackup.tar myapp1.log myapp2.log myapp3.log
[user@host ~]$ ls mybackup.tar
mybackup.tar
```

* sui file che dobbiamo comprimere dobbiamo avere almeno i permessi di scrittura o essere owner altrimenti quei file specifici non verranno compressi, esempio:

```bash
[root@host ~]\# tar -cf /root/etc-backup.tar /etc
tar: Removing leading `/' from member names
```

### Mostrare contenuto archivio

```bash
[root@host ~]\# tar -tf /root/etc.tar
etc/
etc/fstab
etc/crypttab
etc/mtab
...output omitted...
```

### Estrarre contenuto archivio

```bash
[root@host ~]\# mkdir /root/etcbackup
[root@host ~]\# cd /root/etcbackup
[root@host etcbackup]\# tar -tf /root/etc.tar
etc/
etc/fstab
etc/crypttab
etc/mtab
...output omitted...
[root@host etcbackup]\# tar -xf /root/etc.tar

# -p mantiene i permessi iniziali, altrimenti sará l'utente che ha estratto l'owner
[user@host scripts]\# tar -xpf /home/user/myscripts.tar
...output omitted...
```

### Creazione archivio compresso

* compressione _<mark style="color:green;">**`gzip`**</mark>**&#x20;:**_&#x20;

<pre><code><strong>[root@host ~]# tar -czf /root/etcbackup.tar.gz /etc
</strong>tar: Removing leading `/' from member names
</code></pre>

* compressione _<mark style="color:blue;">`bzip2`</mark>_ :

<pre><code><strong>[root@host ~]$ tar -cjf /root/logbackup.tar.bz2 /var/log
</strong>tar: Removing leading `/' from member names
</code></pre>

* compressione _<mark style="color:orange;">`xz`</mark>_ :

<pre><code><strong>[root@host ~]$ tar -cJf /root/sshconfig.tar.xz /etc/ssh
</strong>tar: Removing leading `/' from member names
</code></pre>

### Estrazione contenuto dell'archivio compresso

Il comando `tar` può determinare automaticamente quale compressione è stata utilizzata, quindi non è necessario specificare l'opzione di compressione.

```bash
# Esempio di errore
[root@host ~]\# tar -xzf /root/etcbackup.tar.xz

gzip: stdin: not in gzip format
tar: Child returned status 1
tar: Error is not recoverable: exiting now

# Correzione
[root@host ~]\# tar -xf /root/etcbackup.tar.xz
```

Per decomprimere un singolo file compresso o un archivio compresso senza estrarre il contenuto, utilizza i comandi autonomi `gunzip`, `bunzip2` e `unxz` :&#x20;

```bash
# opzione -l serve a visualizzare l'output
[user@host ~]$ gzip -l file.tar.gz
         compressed        uncompressed  ratio uncompressed_name
          221603125           303841280  27.1% file.tar
[user@host ~]$ xz -l file.xz
Strms  Blocks   Compressed Uncompressed  Ratio  Check   Filename
    1       1    195.7 MiB    289.8 MiB  0.675  CRC64   file.xz
```

