# Riparazione problemi del file-system al boot

## Problemi al file-system

Errori nel file `/etc/fstab` o sistemi di file corrotti possono impedire a un sistema di completare il processo di avvio. \
In alcuni scenari di errore, il sistema interrompe il processo di avvio e apre una shell di emergenza che richiede la password dell'utente `root`.

#### File-system corrotto

Il servizio systemd tenta di riparare il file system. Se il problema non può essere risolto automaticamente, il sistema apre una shell di emergenza.

#### Dispositivo non esistente o UUID

L'attesa del dispositivo da parte del servizio `systemd` scade. \
Se il dispositivo non risponde, il sistema apre una shell di emergenza.

## Riparazione problemi del file-system al boot

Per accedere a un sistema che non può completare l'avvio a causa di problemi con il file system, l'architettura `systemd` fornisce un <mark style="color:orange;">target di avvio</mark> <mark style="color:orange;"></mark><mark style="color:orange;">`emergency`</mark>, che apre una shell di emergenza che richiede la password di `root` per l'accesso:

```bash
...output omitted...
[*     ] A start job is running for /dev/vda2 (27s / 1min 30s)
[ TIME ] Timed out waiting for device /dev/vda2.
[DEPEND] Dependency failed for /mnt/mountfolder
[DEPEND] Dependency failed for Local File Systems.
[DEPEND] Dependency failed for Mark need to relabel after reboot.
...output omitted...
[  OK  ] Started Emergency Shell.
[  OK  ] Reached target Emergency Mode.
...output omitted...
Give root password for maintenance
(or press Control-D to continue): 
```

Il demone `systemd` non è riuscito a montare il dispositivo `/dev/vda2` ed è andato in timeout. Poiché il dispositivo non è disponibile, il sistema apre una shell di emergenza per l'accesso di manutenzione.

Utilizza il comando `mount` per trovare quali file system sono attualmente montati dal demone `systemd` :&#x20;

<pre class="language-bash"><code class="lang-bash"><strong>[root@host ~]\# mount
</strong>...output omitted...
/dev/vda1 on / type xfs (ro,relatime,seclabel,attr2,inode64,noquota)
...output omitted...
</code></pre>

<pre class="language-bash"><code class="lang-bash"><strong>[root@host ~]\# mount -o remount,rw /
</strong></code></pre>

Questa opzione monta i processi su ogni voce del file system, ma salta quei file system già montati. \
Il comando visualizza eventuali errori che si verificano durante il montaggio di un file system:&#x20;

<pre class="language-bash"><code class="lang-bash"><strong>[root@host ~]\# mount --all
</strong>mount: /mnt/mountfolder: mount point does not exist.
</code></pre>

Crea la directory `/mnt/mountfolder` prima di riprovare il montaggio. Altri messaggi di errore possono verificarsi, inclusi errori di battitura negli ingressi, nomi di dispositivo o UUID errati. Dopo aver corretto tutti i problemi nel file `/etc/fstab`, informa il demone `systemd` di registrare il nuovo file `/etc/fstab` utilizzando il comando `systemctl daemon-reload`. Quindi, riprova a montare tutte le voci.

<pre class="language-bash"><code class="lang-bash"><strong>[root@host ~]\# systemctl daemon-reload
</strong><strong>[root@host ~]\# mount --all
</strong></code></pre>
