# Indagare e risolvere i problemi relativi a SELinux

## Risoluzione problemi SELinux

Quando le applicazioni non funzionano a causa di dinieghi di accesso SELinux, esistono metodi e strumenti per risolvere questi problemi. È utile iniziare comprendendo i concetti fondamentali:

* **Politiche Mirate**: SELinux utilizza politiche che definiscono le azioni consentite tramite etichette per i processi e le risorse. Se manca un'entry specifica per un'azione, questa viene negata.
* **Registrazione**: I tentativi negati vengono registrati con informazioni utili.
* **Problemi Comuni**: Gli errori comuni spesso riguardano contesti errati su file nuovi, copiati o spostati.

### Suggerimenti per la risoluzione

1. **Controllo dei Contesti**: Verificare il tipo di etichetta nel man page `_selinux` e i contesti di processo e file.
2. **Booleani di Configurazione**: Alcune funzionalità opzionali richiedono configurazioni aggiuntive.
3. **Conformità alle Politiche**: Assicurare che le applicazioni seguano le regole esistenti e che i contesti siano appropriati.

## Monitoraggio violazioni SELinux

Il servizio di risoluzione dei problemi di SELinux, fornito dal pacchetto <mark style="color:orange;">`setroubleshoot-server`</mark>, offre strumenti per diagnosticare problemi di SELinux. \
Quando SELinux nega un'azione, un messaggio _<mark style="color:yellow;">AVC (Access Vector Cache)</mark>_ viene registrato nel file di log di sicurezza `/var/log/audit/audit.log`.&#x20;

Il servizio di risoluzione dei problemi di SELinux monitora gli _eventi AVC_ e invia un riepilogo degli eventi al file `/var/log/messages`. \
Il _riepilogo AVC_ include un identificatore univoco dell'evento (UUID). \
Usa il comando <mark style="color:green;">`sealert -l _UUID_`</mark> per <mark style="color:green;">visualizzare dettagli completi del rapporto per l'evento specifico.</mark> \
Usa il comando <mark style="color:blue;">`sealert -a /var/log/audit/audit.log`</mark> per <mark style="color:blue;">visualizzare tutti gli eventi esistenti.</mark>&#x20;

Considera la seguente sequenza di comandi su un server web Apache standard. \
Crei `/root/mypage` e lo sposti nella directory di contenuti di default di Apache (`/var/www/html`). \
Poi, dopo aver avviato il servizio Apache, provi a recuperare il contenuto del file.

<pre class="language-bash"><code class="lang-bash"><strong>[root@host ~]\# touch /root/mypage
</strong><strong>[root@host ~]\# mv /root/mypage /var/www/html
</strong><strong>[root@host ~]\# systemctl start httpd
</strong><strong>[root@host ~]\# curl http://localhost/mypage
</strong>&#x3C;!DOCTYPE HTML PUBLIC "-//IETF//DTD HTML 2.0//EN">
&#x3C;html>&#x3C;head>
&#x3C;title>403 Forbidden&#x3C;/title>
&#x3C;/head>&#x3C;body>
&#x3C;h1>Forbidden&#x3C;/h1>
&#x3C;p>You don't have permission to access this resource.&#x3C;/p>
&#x3C;/body>&#x3C;/html>
</code></pre>

<mark style="color:red;">Errore</mark>: Il server web non visualizza il contenuto e restituisce un errore di `permesso negato`. \
Un evento AVC viene registrato nei file `/var/log/audit/audit.log` e `/var/log/messages`.&#x20;

<mark style="color:red;">Prendi nota del comando</mark> <mark style="color:red;"></mark><mark style="color:red;">`sealert`</mark> <mark style="color:red;"></mark><mark style="color:red;">suggerito e dell'UUID nel messaggio dell'evento di</mark> <mark style="color:red;"></mark><mark style="color:red;">`/var/log/messages`</mark> :&#x20;

<pre class="language-bash"><code class="lang-bash"><strong>[root@host ~]\# tail /var/log/audit/audit.log
</strong>...output omitted...
type=AVC msg=audit(1649249057.067:212): avc:  denied  { getattr } 
for  pid=2332 comm="httpd" path="/var/www/html/mypage" dev="vda4" ino=9322502 
scontext=system_u:system_r:httpd_t:s0 tcontext=unconfined_u:object_r:admin_home_t:s0 
tclass=file permissive=0
...output omitted
<strong>[root@host ~]\# tail /var/log/messages
</strong>...output omitted...
Apr  6 08:44:19 host setroubleshoot[2547]: SELinux is preventing /usr/sbin/httpd 
from getattr access on the file /var/www/html/mypage. 
For complete SELinux messages run: sealert -l 95f41f98-6b56-45bc-95da-ce67ec9a9ab7
...output omitted...
</code></pre>

->

<pre class="language-bash"><code class="lang-bash"><strong>[root@host ~]\# sealert -l 95f41f98-6b56-45bc-95da-ce67ec9a9ab7
</strong>SELinux is preventing /usr/sbin/httpd from getattr access on the file /var/www/html/mypage.

*****  Plugin restorecon (99.5 confidence) suggests   ************************

If you want to fix the label.
/var/www/html/mypage default label should be httpd_sys_content_t.
Then you can run restorecon. The access attempt may have been stopped due to insufficient permissions to access a parent directory in which case try to change the following command accordingly.
Do
# /sbin/restorecon -v /var/www/html/mypage

*****  Plugin catchall (1.49 confidence) suggests   **************************

If you believe that httpd should be allowed getattr access on the mypage file by default.
Then you should report this as a bug.
You can generate a local policy module to allow this access.
Do
allow this access for now by executing:
# ausearch -c 'httpd' --raw | audit2allow -M my-httpd
# semodule -X 300 -i my-httpd.pp


Additional Information:
Source Context                system_u:system_r:httpd_t:s0
Target Context                unconfined_u:object_r:admin_home_t:s0
Target Objects                /var/www/html/mypage [ file ]
Source                        httpd
Source Path                   /usr/sbin/httpd
...output omitted...

Raw Audit Messages
type=AVC msg=audit(1649249057.67:212): avc:  denied  { getattr } for  pid=2332 comm="httpd" path="/var/www/html/mypage" dev="vda4" ino=9322502 scontext=system_u:system_r:httpd_t:s0 tcontext=unconfined_u:object_r:admin_home_t:s0 tclass=file permissive=0

type=SYSCALL msg=audit(1649249057.67:212): arch=x86_64 syscall=newfstatat success=no exit=EACCES a0=ffffff9c a1=7fe9c00048f8 a2=7fe9ccfc8830 a3=100 items=0 ppid=2329 pid=2332 auid=4294967295 uid=48 gid=48 euid=48 suid=48 fsuid=48 egid=48 sgid=48 fsgid=48 tty=(none) ses=4294967295 comm=httpd exe=/usr/sbin/httpd subj=system_u:system_r:httpd_t:s0 key=(null)

Hash: httpd,httpd_t,admin_home_t,file,getattr
</code></pre>

Per cercare eventi AVC nel file di log `/var/log/audit/audit.log` utilizza il comando <mark style="color:orange;">`ausearch`</mark>. \
Usa l'opzione <mark style="color:yellow;">`-m`</mark> per specificare il <mark style="color:yellow;">tipo di messaggio</mark> <mark style="color:yellow;"></mark><mark style="color:yellow;">`AVC`</mark> e l'opzione <mark style="color:purple;">`-ts`</mark> con un'i<mark style="color:purple;">ndicazione temporale, come</mark> <mark style="color:purple;"></mark><mark style="color:purple;">`recent`</mark> :&#x20;

<pre class="language-bash"><code class="lang-bash"><strong>[root@host ~]\# ausearch -m AVC -ts recent
</strong>----
time->Tue Apr  6 13:13:07 2019
type=PROCTITLE msg=audit(1554808387.778:4002): proctitle=2F7573722F7362696E2F6874747064002D44464F524547524F554E44
type=SYSCALL msg=audit(1554808387.778:4002): arch=c000003e syscall=49 success=no exit=-13 a0=3 a1=55620b8c9280 a2=10 a3=7ffed967661c items=0 ppid=1 pid=9340 auid=4294967295 uid=0 gid=0 euid=0 suid=0 fsuid=0 egid=0 sgid=0 fsgid=0 tty=(none) ses=4294967295 comm="httpd" exe="/usr/sbin/httpd" subj=system_u:system_r:httpd_t:s0 key=(null)
type=AVC msg=audit(1554808387.778:4002): avc:  denied  { name_bind } for  pid=9340 comm="httpd" src=82 scontext=system_u:system_r:httpd_t:s0 tcontext=system_u:object_r:reserved_port_t:s0 tclass=tcp_socket permissive=0
</code></pre>

## Risoluzione dei problemi relativi a SELinux con la console web

Il pannello della web console di RHEL include strumenti per la risoluzione dei problemi di SELinux. \
Vai su SELinux nel menu laterale per controllare lo stato attuale e visualizzare eventuali errori di accesso:

<div data-full-width="true"><figure><img src="../.gitbook/assets/image (8).png" alt=""><figcaption></figcaption></figure></div>

Clicca sul carattere **>** per visualizzare i dettagli dell'evento. \
Clicca su _**solution details**_ per visualizzare tutti i dettagli dell'evento e i consigli. Puoi cliccare su _**Apply the solution**_&#x20;

Dopo aver corretto il problema, la sezione degli errori di controllo accesso di SELinux dovrebbe rimuovere quell'evento dalla visualizzazione. Se appare il messaggio `Nessun avviso SELinux`, allora hai corretto tutti i problemi SELinux attuali.
