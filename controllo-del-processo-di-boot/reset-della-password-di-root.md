# Reset della password di root

## Resettare la password di root dal bootloader

Esistono diversi metodi per impostare una nuova password per `root`. Un amministratore di sistema potrebbe, ad esempio, avviare il sistema utilizzando un Live CD, montare il file system root da lì e modificare `/etc/shadow`. \
Questa sezione esplora un metodo che non richiede l'uso di supporti esterni.

Per reimpostare una password `root` persa, è necessario utilizzare il kernel di recupero. Per accedere a quella shell `root`, seguire questi passaggi:

1. Riavviare il sistema.
2. Interrompere il conto alla rovescia del boot-loader premendo qualsiasi tasto, eccetto **Invio**.
3. Spostare il cursore sulla voce del kernel di recupero da avviare (la voce con la parola _<mark style="color:orange;">rescue</mark>_ nel suo nome).
4. Premere <mark style="color:yellow;">**e**</mark> per modificare la voce selezionata.
5. Spostare il cursore sulla linea di comando del kernel (la linea che inizia con `linux`).
6. Aggiungere <mark style="color:orange;">`rd.break`</mark>. Con questa opzione, il sistema si interromperà appena prima di trasferire il controllo dall'immagine <mark style="color:orange;">`initramfs`</mark> al sistema reale.
7. Premere <mark style="color:yellow;">**Ctrl**</mark><mark style="color:yellow;">+</mark><mark style="color:yellow;">**x**</mark> per avviare con le modifiche.
8. Premere <mark style="color:yellow;">**Invio**</mark> per effettuare la manutenzione quando richiesto.

In questo momento, il sistema presenta una shell `root`, e il file system principale sul disco è montato in modalità sola lettura su `/sysroot`. \
Poiché la risoluzione dei problemi richiede spesso la modifica del file system principale, è necessario rimontarlo in modalità lettura/scrittura. \
Il passaggio seguente mostra come l'opzione `remount,rw` del comando `mount` rimonta il file system con la nuova opzione (`rw`).&#x20;

<mark style="color:purple;">Importante</mark>:&#x20;

Poiché il sistema <mark style="color:yellow;">non ha ancora abilitato SELinux</mark>, qualsiasi file creato non avrà il contesto SELinux. Alcuni strumenti, come il comando <mark style="color:orange;">`passwd`</mark>, creano prima un file temporaneo e poi lo sostituiscono con il file che si intende modificare, creando di fatto un file senza contesto SELinux. \
Per questo motivo, quando si utilizza il comando `passwd` con `rd.break`, il file `/etc/shadow` non riceve il contesto SELinux.

#### Procedura reset password&#x20;

1. Per rimontare `/sysroot` in modalità lettura/scrittura, eseguire:

```bash
sh-5.1\# mount -o remount,rw /sysroot
```

2. Passa a un ambiente `chroot`, dove `/sysroot` viene trattato come radice del file-system:

```shell
sh-5.1\# chroot /sysroot
```

3. Imposta una nuova password per `root`:

```shell
sh-5.1\# passwd root
```

4. Assicurati che tutti i file non etichettati, incluso `/etc/shadow`, vengano rietichettati durante l'avvio:

```shell
sh-5.1\# touch /.autorelabel
```

Digita `exit` due volte. Il primo comando esce dall'ambiente `chroot`, il secondo esce dalla shell di debug `initramfs`.

### Ripristino di un sistema basato su immagine cloud

Se il sistema è stato creato usando un'immagine cloud Red Hat invece del metodo di installazione tradizionale, il processo di avvio potrebbe differire leggermente. Ecco i passaggi per usare l'opzione `rd.break` per accedere alla shell di root:

1. **Assenza di Kernel di Recupero**: Le immagini cloud Red Hat non includono un kernel di recupero per impostazione predefinita.
2. **Entrare in Modalità Manutenzione**: Utilizzare l'opzione `rd.break` con il kernel predefinito per accedere alla modalità di manutenzione senza necessità della password di root.
3. **Console Multiple**: Il comando potrebbe includere molti argomenti `console=`. La shell di root usa l'ultima console specificata. Se non si ottiene il prompt, riorganizzare temporaneamente gli argomenti `console=` nel bootloader.

Questa procedura consente di risolvere problemi senza alterare l'installazione standard del sistema.

## Controllare i log

Ricorda che, per impostazione predefinita, i registri di sistema sono conservati nella directory <mark style="color:orange;">`/run/log/journal`</mark>, e vengono cancellati quando il sistema viene riavviato. \
Per conservare i registri nella directory `/var/log/journal`, che persiste tra i riavvii, <mark style="color:yellow;">imposta il parametro</mark> <mark style="color:yellow;"></mark><mark style="color:yellow;">`Storage`</mark> <mark style="color:yellow;"></mark><mark style="color:yellow;">su</mark> <mark style="color:yellow;"></mark><mark style="color:yellow;">`persistent`</mark> <mark style="color:yellow;"></mark><mark style="color:yellow;">nel file</mark> <mark style="color:yellow;"></mark><mark style="color:yellow;">`/etc/systemd/journald.conf`</mark><mark style="color:yellow;">.</mark>

<pre class="language-bash"><code class="lang-bash"><strong>[root@host ~]\# vim /etc/systemd/journald.conf
</strong>...output omitted...
[Journal]
<strong>Storage=persistent
</strong>...output omitted...
<strong>[root@host ~]\# systemctl restart systemd-journald.service
</strong></code></pre>

<pre class="language-bash"><code class="lang-bash"><strong># -b -1 INDICA L'ULTIMO BOOT EFFETTUATO; -p err INDICA SOLO MSG D'ERRORE
</strong><strong>[root@host ~]\# journalctl -b -1 -p err
</strong></code></pre>

## Riparazione problemi di avvio di Systemd

Entrare nella `root` shell su `TTY9` (<mark style="color:yellow;">**Ctrl**</mark><mark style="color:yellow;">+</mark><mark style="color:yellow;">**Alt**</mark><mark style="color:yellow;">+</mark><mark style="color:yellow;">**F9**</mark>); disabilitare il servizio `debug-shell.service` quando si fa il DEBUG.

Invece usando il <mark style="color:orange;">GRUB2:</mark>

1. Riavvia il sistema.&#x20;
2. Interrompi il conto alla rovescia del bootloader premendo qualsiasi tasto, tranne <mark style="color:yellow;">**Invio**</mark>.&#x20;
3. Sposta il cursore sull'entrata del kernel da avviare.&#x20;
4. Premi <mark style="color:yellow;">**e**</mark> per modificare l'entrata selezionata.&#x20;
5. Sposta il cursore sulla riga di comando del kernel (la riga che inizia con `linux`).&#x20;
6. Aggiungi <mark style="color:orange;">`systemd.debug-shell`</mark>. Con questo parametro, il sistema si avvia nella shell di debug.&#x20;
7. Premi <mark style="color:yellow;">**Ctrl**</mark><mark style="color:yellow;">+</mark><mark style="color:yellow;">**x**</mark> per avviare con le modifiche.

**Utilizzo dei Target di Emergenza e Soccorso**

Aggiungendo `systemd.unit=rescue.target` o `systemd.unit=emergency.target` alla riga di comando del kernel dal boot loader, il sistema entra in una shell di soccorso o emergenza invece di avviarsi normalmente. Entrambe richiedono la password di `root`. Il target di emergenza monta il file system root in modalità di sola lettura, mentre il target di soccorso attende il completamento dell'unità `sysinit.target` per inizializzare più servizi di sistema. \
L'utente root può risolvere problemi che impediscono l'avvio normale del sistema. \
È possibile modificare `/etc/fstab` solo dopo aver rimontato il drive in modalità lettura-scrittura con `mount -o remount,rw /`. Uscire da queste shell continua il normale avvio del sistema.

**Identificazione dei Job Bloccati**

Durante l'avvio, `systemd` avvia vari job. Se alcuni di questi non completano, bloccano altri job. Per controllare la lista dei job attuali, utilizzare il comando <mark style="color:orange;">`systemctl list-jobs`</mark>. \
I job in esecuzione devono essere completati prima che quelli in attesa possano continuare.
