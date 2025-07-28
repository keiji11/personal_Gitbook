# Gestione Sicurezza SELinux

## Modifica della modalità di applicazione (<mark style="color:orange;">Enforcement mode</mark>) di SELinux

### Architettura SELinux

Security Enhanced Linux (SELinux) è una funzione di sicurezza critica per Linux. \
Controlla l'accesso a file, porte e altre risorse a livello granulare. \
I processi possono accedere solo alle risorse specificate dalle impostazioni di SELinux, che includono policy applicative. Queste policy, note come _politiche mirate_, determinano le azioni per ogni file ed eseguibile.&#x20;

Le autorizzazioni dei file definiscono chi può leggere, scrivere o eseguire un file, ma non controllano come viene utilizzato, potenzialmente esponendolo a usi indesiderati.&#x20;

_<mark style="color:orange;">**SELinux garantisce che le applicazioni agiscano solo entro i limiti delle politiche definite.**</mark>_

### Uso SELinux

SELinux applica regole di accesso che definiscono azioni consentite tra processi e risorse. Le applicazioni con scarso design di sicurezza sono comunque protette. Esistono due domini d'esecuzione:

* **Confinato**: con una politica mirata.
* **Non confinato**: senza protezione SELinux.

#### Modalità

1. **Enforcing**: Applica policy caricate (predefinito in RHEL).
2. **Permissive**: Registra violazioni senza applicarle; utile per test.
3. **Disabled**: SELinux è disattivato; le violazioni non vengono registrate.

**Nota Importante**: Da RHEL 9, per disabilitare SELinux completamente, usare `selinux=0` al boot. Impostare `SELINUX=disabled` nel file di configurazione comporta l'attivazione di SELinux senza caricare policy, negando tutte le azioni. Questa misura blocca tentativi di aggiramento delle protezioni.

### Concetti base SELinux

**SELinux** è progettato per proteggere i dati degli utenti dall'uso improprio di applicazioni o servizi compromessi. A differenza del modello standard di <mark style="color:yellow;">**Controllo di Accesso Discrezionale (DAC)**</mark>, SELinux utilizza il <mark style="color:orange;">**Controllo di Accesso Obbligatorio (MAC)**</mark>, applicando regole granulari che non possono essere bypassate.

#### Funzionamento

Ogni risorsa (file, processo, directory, porta) ha un'etichetta chiamata _**contesto SELinux**,_ che corrisponde a una regola di policy definita. Senza una regola esplicita, l'accesso è negato. Le etichette comprendono campi come `utente`, `ruolo`, `tipo`, e `livello di sicurezza`.

#### Politiche Targeted

La policy predefinita su RHEL è la _<mark style="color:orange;">Targeted Policy</mark>_, che utilizza contesti di tipo, riconoscibili dal suffisso `_t`, per definire le regole di accesso.

<figure><img src="../.gitbook/assets/image (2) (1).png" alt=""><figcaption></figcaption></figure>

### Concetti relativi alle policy dell regole di accesso

Ad esempio unn processo server web Apache è etichettato con il contesto di tipo `httpd_t`. \
I <mark style="color:yellow;">file e le directory del server web</mark>, come quelli in `/var/www/html/`, hanno il contesto di tipo `httpd_sys_content_t`. \
I <mark style="color:yellow;">file temporanei</mark> nelle directory `/tmp` e `/var/tmp` sono etichettati con il contesto di tipo `tmp_t`. \
Le <mark style="color:yellow;">porte del server web</mark> hanno il contesto `http_port_t`.

Per impostazione predefinita, il server Apache può accedere solo ai file etichettati con `httpd_sys_content_t`. \
Non può accedere ai file `tmp_t`, proteggendo da accessi non autorizzati. \
Un server MariaDB utilizza il contesto `mysqld_t` e accede ai file con il contesto `mysqld_db_t`, ma non ai file del server web.

<figure><img src="../.gitbook/assets/image (1) (1) (1) (1) (1).png" alt="diagramma di flusso SELinux "><figcaption></figcaption></figure>

#### Uso dell'opzione _<mark style="color:orange;">-Z</mark>_ per gestione contesto SELinux

<pre class="language-bash"><code class="lang-bash"><strong>[root@host ~]\# ps axZ
</strong>LABEL                               PID TTY      STAT   TIME COMMAND
system_u:system_r:kernel_t:s0         2 ?        S      0:00 [kthreadd]
system_u:system_r:kernel_t:s0         3 ?        I&#x3C;     0:00 [rcu_gp]
system_u:system_r:kernel_t:s0         4 ?        I&#x3C;     0:00 [rcu_par_gp]
...output omitted...
<strong>[root@host ~]\# systemctl start httpd
</strong><strong>[root@host ~]\# ps -ZC httpd
</strong>LABEL                               PID TTY          TIME CMD
system_u:system_r:httpd_t:s0       1550 ?        00:00:00 httpd
system_u:system_r:httpd_t:s0       1551 ?        00:00:00 httpd
system_u:system_r:httpd_t:s0       1552 ?        00:00:00 httpd
system_u:system_r:httpd_t:s0       1553 ?        00:00:00 httpd
system_u:system_r:httpd_t:s0       1554 ?        00:00:00 httpd
<strong>[root@host ~]\# ls -Z /var/www
</strong>system_u:object_r:httpd_sys_script_exec_t:s0 cgi-bin
system_u:object_r:httpd_sys_content_t:s0 html
</code></pre>

### Modifica della modalitá SELinux

`getenforce` mostra la modalitá SELinux corrente, \
`setenforce` modifica la modalitá SELinux :&#x20;

<pre class="language-bash"><code class="lang-bash"><strong>[root@host ~]\# getenforce
</strong>Enforcing
<strong>[root@host ~]\# setenforce
</strong>usage:  setenforce [ Enforcing | Permissive | 1 | 0 ]
<strong>[root@host ~]\# setenforce 0
</strong><strong>[root@host ~]\# getenforce
</strong>Permissive
<strong>[root@host ~]\# setenforce Enforcing
</strong><strong>[root@host ~]\# getenforce
</strong>Enforcing
</code></pre>

Alternativamente, per avviare il sistema in modalità `permissive`, all'avvio impostare il parametro del kernel `enforcing=0`; \
per la modalità `enforcing`, utilizzare `enforcing=1`.&#x20;

Disabilitare SELinux con `selinux=0`, oppure abilitarlo con `selinux=1`.&#x20;

Red Hat consiglia di riavviare il server quando si cambia da `Permissive` a `Enforcing` per assicurarsi che i servizi siano confinati correttamente al prossimo avvio.

### Settare la modalitá SELinux di default

Per configurare SELinux in modo permanente, utilizzare il file `/etc/selinux/config`.

<pre class="language-bash"><code class="lang-bash"># This file controls the state of SELinux on the system.
# SELINUX= can take one of these three values:
#     enforcing - SELinux security policy is enforced.
#     permissive - SELinux prints warnings instead of enforcing.
#     disabled - No SELinux policy is loaded.
...output omitted...
#
# NOTE: In earlier Fedora kernel builds, SELINUX=disabled would also
# fully disable SELinux during boot. If you need a system with SELinux
# fully disabled instead of SELinux running with no policy loaded, you
# need to pass selinux=0 to the kernel command line. You can use grubby
# to persistently set the bootloader to boot with selinux=0:
#
#    grubby --update-kernel ALL --args selinux=0
#
# To revert back to SELinux enabled:
#
#    grubby --update-kernel ALL --remove-args selinux
#
<strong>SELINUX=enforcing
</strong># SELINUXTYPE= can take one of these three values:
#     targeted - Targeted processes are protected,
#     minimum - Modification of targeted policy. Only selected processes are protected.
#     mls - Multi Level Security protection.
SELINUXTYPE=targeted
</code></pre>

Il sistema legge questo file all'avvio e avvia SELinux di conseguenza.&#x20;

Gli argomenti del kernel `selinux=0|1` e `enforcing=0|1` hanno la precedenza su questa configurazione.
