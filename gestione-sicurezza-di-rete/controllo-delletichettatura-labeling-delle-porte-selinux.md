# Controllo dell'etichettatura (labeling) delle porte SELinux

## Etichettatura (labeling) delle porte SELinux

SELinux etichetta le porte di rete con un contesto SELinux. \
SELinux controlla l'accesso alla rete etichettando le porte di rete e includendo regole nella politica mirata di un servizio. \
Ad esempio, la politica mirata SSH include la porta `22/TCP` con un'etichetta di contesto di porta `ssh_port_t`. \


Quando un processo mirato tenta di aprire una porta per l'ascolto, SELinux verifica che la politica includa voci che abilitino l'associazione del processo e del contesto. \
SELinux può quindi bloccare un servizio malevolo dall'occupare porte che altri servizi di rete legittimi utilizzano.

## Gestione Etichettatura (labeling) delle porte SELinux

Se un servizio tenta di ascoltare su una porta non standard e la porta non è etichettata con il tipo SELinux corretto, SELinux potrebbe bloccare il tentativo. \
È possibile correggere il problema cambiando il contesto SELinux sulla porta. Solitamente, la politica `targeted` etichetta già tutte le porte previste con il tipo corretto. \
Ad esempio, la porta `8008/TCP` è spesso usata per applicazioni web ed è già etichettata con `http_port_t`, il tipo di porta usato dai server web. \
Le porte possono avere solo un contesto di porta.

### Elenco etichette porta

<pre class="language-bash"><code class="lang-bash"><strong># FILTRO per NUMERO PORTA
</strong><strong>[root@host ~]\# grep gopher /etc/services
</strong>gopher          70/tcp                          # Internet Gopher
gopher          70/udp
</code></pre>

<pre class="language-bash"><code class="lang-bash"><strong># Elenco attuali assegnazioni delle etichette delle porte.
</strong><strong>[root@host ~]\# semanage port -l
</strong>...output omitted...
http_cache_port_t       tcp   8080, 8118, 8123, 10001-10010
http_cache_port_t       udp   3130
http_port_t             tcp   80, 81, 443, 488, 8008, 8009, 8443, 9000
...output omitted...
</code></pre>

<pre class="language-bash"><code class="lang-bash"><strong># Filtrare l'etichetta della porta SELinux utilizzando il nome del servizio
</strong><strong>[root@host ~]\# semanage port -l | grep ftp
</strong>ftp_data_port_t                tcp      20
ftp_port_t                     tcp      21, 989, 990
ftp_port_t                     udp      989, 990
tftp_port_t                    udp      69
</code></pre>

<pre class="language-bash"><code class="lang-bash"># Filtrare l'etichetta della porta SELinux utilizzando il numero di porta
<strong>[root@host ~]\# semanage port -l | grep -w 70
</strong>gopher_port_t                  tcp      70
gopher_port_t                  udp      70
</code></pre>

### Gestione _port bindings (associazione porte)_

Usa il comando <mark style="color:orange;">`semanage`</mark> per assegnare nuove etichette di porta, rimuovere etichette di porta e modificare quelle esistenti.

Nota: Non puoi cambiare le etichette delle porte predefinite usando il comando `semanage`. \
Invece, devi modificare e ricaricare il modulo di policy del servizio mirato.

Puoi etichettare una nuova porta con un'etichetta di contesto di porta esistente (tipo). \
L'opzione <mark style="color:yellow;">`-a`</mark> del comando <mark style="color:orange;">`semanage port`</mark> aggiunge una nuova etichetta di porta; \
l'opzione <mark style="color:yellow;">`-t`</mark> indica il tipo; e l'opzione <mark style="color:yellow;">`-p`</mark> indica il protocollo.

<pre class="language-bash"><code class="lang-bash"><strong>[root@host ~]\# semanage port -a -t port_label -p tcp|udp PORTNUMBER
</strong></code></pre>

<pre class="language-bash"><code class="lang-bash"><strong>[root@host~]\# semanage port -a -t gopher_port_t -p tcp 71
</strong></code></pre>

<pre class="language-bash"><code class="lang-bash"><strong># Visualizza modifiche locali alla policy di default
</strong><strong>[root@host~]\# semanage port -l -C
</strong>SELinux Port Type              Proto    Port Number

gopher_port_t                  tcp      71
</code></pre>

Per visualizzare un elenco di tutte le pagine man disponibili di SELinux, installa il pacchetto e quindi esegui una ricerca con `man -k` per la stringa `_selinux` :&#x20;

<pre class="language-bash"><code class="lang-bash"><strong>[root@host ~]\# dnf -y install selinux-policy-doc
</strong><strong>[root@host ~]\# man -k _selinux
</strong></code></pre>

<pre class="language-bash"><code class="lang-bash"><strong># RIMOZIONE(opzione -d) del BINDING per la PORTA 71/TCP per il tipo gopher_port_t
</strong><strong>[root@host ~]\# semanage port -d -t gopher_port_t -p tcp 71
</strong></code></pre>

<pre class="language-bash"><code class="lang-bash"><strong># MODIFICA(opzione -m) del BINDING al tipo http_port_t (da gopher_port_t)
</strong><strong>[root@server ~]\# semanage port -m -t http_port_t -p tcp 71
</strong></code></pre>

Visualizziamo i cambiamenti effettuati:&#x20;

<pre class="language-bash"><code class="lang-bash"><strong>[root@server ~]\# semanage port -l -C
</strong>SELinux Port Type              Proto    Port Number

http_port_t                    tcp      71
<strong>[root@server ~]\# semanage port -l | grep http
</strong>http_cache_port_t              tcp      8080, 8118, 8123, 10001-10010
http_cache_port_t              udp      3130
http_port_t                    tcp      71, 80, 81, 443, 488, 8008, 8009, 8443, 9000
pegasus_http_port_t            tcp      5988
pegasus_https_port_t           tcp      5989
</code></pre>
