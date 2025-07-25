# Controllo del processo di Boot

## Selezione del target di boot

### Descrizione del boot in RHEL 9

#### Avvio Iniziale

1. **Accensione**
   * Il firmware del sistema (UEFI/BIOS) esegue il Power On Self Test (POST).
   * Individuazione del dispositivo di avvio tramite MBR.
2. **Caricamento Boot Loader**
   * Il firmware legge e passa controllo a GRUB2.
   * Evitare `grub2-install` su sistemi UEFI per evitare la perdita delle firme sicure.
3. **Configurazione di GRUB2**
   * Configurazione tramite `/etc/grub.d/` e `/etc/default/grub`.
   * Generazione del file di configurazione con `grub2-mkconfig`.

#### Caricamento del Sistema Operativo

4. **Caricamento del Kernel e initramfs**
   * GRUB2 carica il kernel e initramfs in memoria.
   * initramfs contiene moduli kernel necessari e un sistema radice avviabile.
5. **Transizione al Kernel**
   * Il boot loader passa il controllo al kernel.
   * Il kernel inizializza l'hardware e esegue `/sbin/init` (systemd).

#### Configurazione di `systemd`

6. **Esecuzione di systemd**
   * systemd monta il file system radice in `/sysroot`.
   * Passaggio al file system radice installato.
   * Avvio dei target di sistema predefiniti per il login testuale o grafico.

#### File di Configurazione Chiave

* **GRUB**: `/etc/grub.d/` e `/etc/default/grub`
* **initramfs**: `/etc/dracut.conf.d/`
* **File System**: `/etc/fstab`
* **Systemd**: `/etc/systemd/system/default.target`

> Questo processo garantisce che tutti i componenti hardware e software lavorino insieme per portare il sistema a una schermata di accesso operativa.

<div data-full-width="true"><figure><img src="../.gitbook/assets/image (11).png" alt=""><figcaption></figcaption></figure></div>

## Spegnimento e riavvio

`poweroff` arresta tutti i servizi in esecuzione, smonta tutti i file system (o li rimonta in sola lettura quando non è possibile smontarli) e quindi spegne il sistema:

<pre class="language-bash"><code class="lang-bash"><strong>[user@host ~]$ systemctl poweroff
</strong></code></pre>

`reboot` arresta tutti i servizi in esecuzione, smonta tutti i file system e quindi riavvia il sistema:

<pre><code><strong>[user@host ~]$ systemctl reboot
</strong></code></pre>

`halt` non spegne il sistema, porta il sistema in uno stato in cui è possibile spegnerlo manualmente in modo sicuro:

<pre><code><strong>[user@host ~]$ systemctl halt
</strong></code></pre>

## Selezione di un _system target_

Tabella dei _**target di sistema**_ piú usati :

<table><thead><tr><th valign="top">Target</th><th valign="top">Scopo</th></tr></thead><tbody><tr><td valign="top"><code>graphical.target</code></td><td valign="top">Questo <em>target</em> supporta più utenti e fornisce accessi grafici e basati su testo.</td></tr><tr><td valign="top"><code>multi-user.target</code></td><td valign="top">Questo <em>target</em> supporta più utenti e fornisce solo accessi basati su testo.</td></tr><tr><td valign="top"><code>rescue.target</code></td><td valign="top">Questo <em>target</em> supporta un solo utente per abilitarlo nel rispristinare il sistema.</td></tr><tr><td valign="top"><code>emergency.target</code></td><td valign="top">Questo target inizializza solo il sistema minimale per poterlo solo riparare quando l'unitá <code>rescue.target</code> fallisce.</td></tr></tbody></table>

Un _target_ può far parte di un altro _target:_

<pre class="language-bash"><code class="lang-bash"><strong># LISTA DIPENDENZE DI UN TARGET
</strong><strong>[user@host ~]$ systemctl list-dependencies graphical.target | grep target
</strong>graphical.target
* └─multi-user.target
*   ├─basic.target
*   │ ├─paths.target
*   │ ├─slices.target
*   │ ├─sockets.target
*   │ ├─sysinit.target
*   │ │ ├─cryptsetup.target
*   | | ├─integritysetup.target
*   │ │ ├─local-fs.target
...output omitted...
</code></pre>

<pre class="language-bash"><code class="lang-bash"><strong># LISTA TARGET DISPONIBILI SUL SISTEMA
</strong><strong>[user@host ~]$ systemctl list-units --type=target --all
</strong>  UNIT                      LOAD      ACTIVE   SUB    DESCRIPTION
  ---------------------------------------------------------------------------
  basic.target              loaded    active   active Basic System
...output omitted...
  cloud-config.target       loaded    active   active Cloud-config availability
  cloud-init.target         loaded    active   active Cloud-init target
  cryptsetup-pre.target     loaded    inactive dead   Local Encrypted Volumes (Pre)
  cryptsetup.target         loaded    active   active Local Encrypted Volumes
...output omitted...
</code></pre>

### Selezionare un _target_ in Runtime

Su un sistema in esecuzione, gli amministratori possono passare a un target diverso utilizzando il comando `systemctl isolate` :&#x20;

<pre class="language-bash"><code class="lang-bash"><strong>[root@host ~]\# systemctl isolate multi-user.target
</strong></code></pre>

Non tutti i target possono essere isolati (é scritto nel file di configurazione del target ):&#x20;

<pre class="language-bash"><code class="lang-bash"><strong># ALLOW ISOLATE = YES
</strong><strong>[user@host ~]$ systemctl cat graphical.target
</strong># /usr/lib/systemd/system/graphical.target
...output omitted...
[Unit]
Description=Graphical Interface
Documentation=man:systemd.special(7)
Requires=multi-user.target
Wants=display-manager.service
Conflicts=rescue.service rescue.target
After=multi-user.target rescue.service rescue.target display-manager.service
AllowIsolate=yes
<strong># ALLOW ISOLATE = NO
</strong><strong>[user@host ~]$ systemctl cat cryptsetup.target
</strong># /usr/lib/systemd/system/cryptsetup.target
...output omitted...
[Unit]
Description=Local Encrypted Volumes
Documentation=man:systemd.special(7)
</code></pre>

### Impostare un _target di default_

Per selezionare un target diverso all'avvio, aggiungere l'opzione `systemd.unit=`_<mark style="color:yellow;">`target`</mark>_`.target` alla riga di comando del kernel dal boot loader:

```bash
# ESEMPIO
systemd.unit=rescue.target
```

Questa modifica di configurazione influisce solo su un singolo avvio ed è uno strumento utile per risolvere i problemi del processo di avvio.&#x20;

Per utilizzare questo metodo per la selezione di un target diverso, segui la procedura seguente:

1. Avvia o riavvia il sistema.
2. Interrompi il conto alla rovescia del menu del boot loader premendo qualsiasi tasto (eccetto **Invio**, che avvierebbe un avvio normale).
3. Sposta il cursore sull'entry del kernel da avviare.
4. Premi _<mark style="color:red;">**e**</mark>_ per modificare l'entry corrente.
5. Sposta il cursore alla linea che inizia con <mark style="color:red;">`linux`</mark>, che è la riga di comando del kernel.
6. Aggiungi `systemd.unit=`_`target`_`.target`, ad esempio, `systemd.unit=emergency.target`.
7. Premi _<mark style="color:orange;">**Ctrl**</mark><mark style="color:orange;">+</mark><mark style="color:orange;">**x**</mark>_ per avviare con queste modifiche.
