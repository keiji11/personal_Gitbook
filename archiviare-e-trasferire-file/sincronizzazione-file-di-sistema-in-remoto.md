# Sincronizzazione file di sistema in remoto

## `rsync`&#x20;

Un altro modo per copiare i file da un sistema a un altro in modo sicuro. \
Lo strumento utilizza un algoritmo che riduce al minimo i dati copiati sincronizzando solo le parti modificate dei file. \
Se due file o directory sono simili tra due server, il comando `rsync` copia solo le differenze tra i file system.

Un vantaggio del comando `rsync` è che copia i file in modo sicuro ed efficiente tra un sistema locale e uno remoto. \
Mentre una sincronizzazione iniziale della directory richiede all'incirca lo stesso tempo di una copia, le sincronizzazioni successive copiano solo le differenze sulla rete, accelerando notevolmente gli aggiornamenti.&#x20;

### Opzione `-n`  (simulazione/dry-run)

Usa l'opzione `-n` del comando `rsync` per una simulazione (dry-run). \
Una simulazione mostra le modifiche che il comando `rsync` eseguirebbe. Esegui una simulazione prima dell'operazione `rsync` effettiva per garantire che nessun file critico venga sovrascritto o eliminato.&#x20;

### Opzione `-v` (verbose)

L'opzione `-v` o `--verbose` del comando `rsync` fornisce un output più dettagliato, utile per la risoluzione dei problemi e per visualizzare il progresso in tempo reale.&#x20;

### Opzione `-a` (modalitá archivio)

L'opzione `-a` o `--archive` del comando `rsync` abilita la "modalità archivio". \
Questa opzione consente la copia ricorsiva e attiva molte opzioni utili per preservare la maggior parte delle caratteristiche dei file.&#x20;

La modalità archivio è equivalente a specificare le seguenti opzioni:

<table><thead><tr><th valign="top">Opzione</th><th valign="top">Descrizione</th></tr></thead><tbody><tr><td valign="top"><code>-r</code>, <code>--recursive</code></td><td valign="top">Sincronizza l'intera gerarchia delle directory</td></tr><tr><td valign="top"><code>-l</code>, <code>--links</code></td><td valign="top">Sincronizza link simbolici</td></tr><tr><td valign="top"><code>-p</code>, <code>--perms</code></td><td valign="top">Preserva permessi</td></tr><tr><td valign="top"><code>-t</code>, <code>--times</code></td><td valign="top">Preserva time stamps</td></tr><tr><td valign="top"><code>-g</code>, <code>--group</code></td><td valign="top">Preserva il gruppo proprietario</td></tr><tr><td valign="top"><code>-o</code>, <code>--owner</code></td><td valign="top">Preserva il proprietario dei file</td></tr><tr><td valign="top"><code>-D</code>, <code>--devices</code></td><td valign="top">Preserva file di dispositivo</td></tr><tr><td valign="top"><code>-H</code></td><td valign="top">Preserva hard links </td></tr><tr><td valign="top"><code>-A</code></td><td valign="top">Preserva AccessControlLists (ACLs)</td></tr><tr><td valign="top"><code>-X</code></td><td valign="top">Preserva i file SELinux</td></tr></tbody></table>

## Esempi

`rsync` funziona grosso modo come `sftp`&#x20;

```bash
# Sincronizza local /var/log su remote /tmp nell'hosta
[root@host ~]\# rsync -av /var/log hosta:/tmp
root@hosta's password: password
receiving incremental file list
log/
log/README
log/boot.log
...output omitted...
sent 9,783 bytes  received 290,576 bytes  85,816.86 bytes/sec
total size is 11,585,690  speedup is 38.57
```

```bash
# viceversa:
[root@host ~]\# rsync -av hosta:/var/log /tmp
root@hosta's password: password
receiving incremental file list
log/boot.log
log/dnf.librepo.log
log/dnf.log
...output omitted...

sent 9,783 bytes  received 290,576 bytes  85,816.86 bytes/sec
total size is 11,585,690  speedup is 38.57
```

```bash
# stessa macchina
[user@host ~]$ su -
Password: password
[root@host ~]\# rsync -av /var/log /tmp
receiving incremental file list
log/
log/README
...output omitted...
log/tuned/tuned.log

sent 11,592,423 bytes  received 779 bytes  23,186,404.00 bytes/sec
total size is 11,586,755  speedup is 1.00
[user@host ~]$ ls /tmp
log  ssh-RLjDdarkKiW1
```

## Nota bene

Se omettiamo lo slash finale ad una directory verrá creata la directory se non esiste, invece specifichiamo lo slash alla fine della directory verranno copiati/sincronizzati solo i file al suo interno.
