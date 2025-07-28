# Gestione sicurezza di rete

## Gestione firewall del server

### Concetti di Architettura del firewall

Il kernel Linux fornisce il framework <mark style="color:orange;">`netfilter`</mark> per operazioni sul traffico di rete come il filtraggio dei pacchetti, la traduzione degli indirizzi di rete e la traduzione delle porte. \
Il framework `netfilter` include _hook_ per i moduli del kernel per interagire con i pacchetti di rete mentre attraversano lo stack di rete del sistema. Fondamentalmente, gli hook di `netfilter` sono routine del kernel che intercettano eventi (ad esempio, un pacchetto che entra in un'interfaccia) ed eseguono altre routine correlate (ad esempio, regole del firewall).

### Il framework <mark style="color:orange;">`nftables`</mark>&#x20;

Il framework di classificazione dei pacchetti `nftables` si basa su `netfilter` per applicare regole firewall al traffico di rete e, in Red Hat Enterprise Linux 9, sostituisce il framework `iptables`, ormai deprecato.

`nftables` offre vantaggi come una maggiore usabilità e set di regole più efficienti. \
A differenza di `iptables`, che richiedeva una regola separata per ciascun protocollo, `nftables` consente di gestire traffico IPv4 e IPv6 contemporaneamente tramite l'unico strumento `nft`. \
Per convertire le configurazioni `iptables` in `nftables`, utilizzare le utilità `iptables-translate` e `ip6tables-translate`.

### Il servizio <mark style="color:orange;">firewalld</mark>

Il servizio `firewalld` è un gestore firewall dinamico per Linux Enterprise Red Hat 9 e funziona come interfaccia per `nftables`. \
Facilita la gestione classificando il traffico di rete in _zone_, assegnando pacchetti in base all'indirizzo IP o all'interfaccia di rete. Ogni zona ha una lista di porte e servizi aperti o chiusi. \
Per dispositivi mobili, <mark style="color:yellow;">`NetworkManager`</mark> può regolare automaticamente la zona firewall a seconda della rete connessa. \
Se un pacchetto non ha una zona specifica, viene utilizzata la zona dell'interfaccia di rete o quella predefinita. \
La <mark style="color:orange;">zona</mark> <mark style="color:orange;"></mark><mark style="color:orange;">`trusted`</mark> consente tutto il traffico, a differenza delle altre, che lo bloccano se non soddisfa criteri specifici.

### Zone predefinite

<table><thead><tr><th width="127.59991455078125" valign="top">nome Zona</th><th valign="top">Configurazione di default </th></tr></thead><tbody><tr><td valign="top"><code>trusted</code></td><td valign="top">Consente tutto il traffico entrante</td></tr><tr><td valign="top"><code>home</code></td><td valign="top">Rifiuta il traffico in entrata a meno che non sia correlato al traffico in uscita o corrispondente ai servizi predefiniti <code>ssh</code>, <code>mdns</code>, <code>ipp-client</code>, <code>samba-client</code> o <code>dhcpv6-client</code>.</td></tr><tr><td valign="top"><code>internal</code></td><td valign="top">Rifiuta il traffico in entrata a meno che non sia correlato al traffico in uscita o corrispondente ai servizi predefiniti <code>ssh</code>, <code>mdns</code>, <code>ipp-client</code>, <code>samba-client</code> or <code>dhcpv6-client</code>  (come la zona <code>home</code> iniziale).</td></tr><tr><td valign="top"><code>work</code></td><td valign="top">Rifiuta il traffico in entrata a meno che non sia correlato al traffico in uscita o coincida con i servizi predefiniti <code>ssh</code>, <code>ipp-client</code> o <code>dhcpv6-client</code>.</td></tr><tr><td valign="top"><code>public</code></td><td valign="top">Rifiuta il traffico in entrata a meno che non sia correlato al traffico in uscita o coincida con i servizi predefiniti <code>ssh</code> o <code>dhcpv6-client</code>. <br><em>Zona predefinita per interfacce di rete aggiunte piú recentemente.</em></td></tr><tr><td valign="top"><code>external</code></td><td valign="top">Rifiuta il traffico in entrata a meno che non sia correlato al traffico in uscita o coincida con i servizi predefiniti <code>ssh</code>. <br>Il traffico IPv4 in uscita che viene inoltrato attraverso questa zona viene <em><strong>mascherato</strong></em> in modo da sembrare proveniente dall'indirizzo IPv4 dell'interfaccia di rete in uscita.</td></tr><tr><td valign="top"><code>dmz</code></td><td valign="top">Rifiuta il traffico in entrata a meno che non sia correlato al traffico in uscita o coincida con i servizi predefiniti <code>ssh</code>.</td></tr><tr><td valign="top"><code>block</code></td><td valign="top">Rifiuta il traffico in entrata a meno che non sia correlato al traffico in uscita.</td></tr><tr><td valign="top"><code>drop</code></td><td valign="top">Elimina tutto il traffico in entrata a meno che non sia correlato al traffico in uscita (non rispond nemmeno con <em><strong>errori ICMP</strong></em>).</td></tr></tbody></table>

### Servizi predefiniti

Il servizio `firewalld` include configurazioni predefinite per servizi comuni, semplificando l'impostazione delle regole del firewall. \
Ad esempio, invece di cercare le porte rilevanti per un server NFS, è possibile usare la configurazione predefinita <mark style="color:orange;">`nfs`</mark> per creare regole appropriate. \
La tabella seguente elenca alcune configurazioni di servizio predefinite che potrebbero essere attive nella tua zona `firewalld` predefinita.

<table><thead><tr><th width="183.5999755859375" valign="top">Nome Servizio</th><th valign="top">Configurazione</th></tr></thead><tbody><tr><td valign="top"><code>ssh</code></td><td valign="top">Server locale SSH. <br>Traffico in <strong>22/tcp</strong>.</td></tr><tr><td valign="top"><code>dhcpv6-client</code></td><td valign="top">Client locale DHCPv6. <br>Traffico in <strong>546/udp</strong> su rete IPv6 fe80::/64.</td></tr><tr><td valign="top"><code>ipp-client</code></td><td valign="top">Stampa locale IPP. <br>Traffico verso <strong>631/udp</strong>.</td></tr><tr><td valign="top"><code>samba-client</code></td><td valign="top">File locali Windows e client di condivisione stampanti. <br>Traffico verso <strong>137/udp</strong> e <strong>138/udp</strong>.</td></tr><tr><td valign="top"><code>mdns</code></td><td valign="top">Risoluzione nomi <em>local-link</em> Multicast DNS (mDNS).<br>Traffico verso <strong>5353/udp</strong> agli indirizzi multicast 224.0.0.251 (IPv4) o ff02::fb (IPv6).</td></tr><tr><td valign="top"><code>cockpit</code></td><td valign="top">Interfaccia web Red Hat Enterprise Linux per la gestione e il monitoraggio del sistema locale e remoto. <br>Traffico verso la porta <strong>9090</strong>.</td></tr></tbody></table>

<pre class="language-bash"><code class="lang-bash"><strong># ELENCO CONFIGURAZIONI DI SERVIZIO PREDEFINITE fw
</strong><strong>[root@host ~]\# firewall-cmd --get-services
</strong>RH-Satellite-6 RH-Satellite-6-capsule amanda-client amanda-k5-client amqp amqps
apcupsd audit bacula bacula-client bb bgp bitcoin bitcoin-rpc bitcoin-testnet
bitcoin-testnet-rpc bittorrent-lsd ceph ceph-mon cfengine cockpit collectd
...output omitted...
</code></pre>

## Configurazione demone `firewalld`&#x20;

L'elenco seguente mostra due modi comuni che gli amministratori di sistema utilizzano per interagire con il servizio `firewalld`:&#x20;

* <mark style="color:orange;">**GUI**</mark> console web.
* lo strumento da riga di comando <mark style="color:orange;">**`firewall-cmd`**</mark>.

### GUI

Con gli adeguati provilegi per accedervi, fai clic sull'opzione _<mark style="color:orange;">**Networking**</mark>_ nel menu di navigazione a sinistra per visualizzare la sezione _<mark style="color:orange;">**Firewall**</mark>_ nella pagina principale di networking. Fai clic sul pulsante  _<mark style="color:orange;">**Edit rules and zone**</mark>_ per navigare alla pagina _<mark style="color:orange;">**Firewall**</mark>_.

<figure><img src=".gitbook/assets/image (12).png" alt=""><figcaption></figcaption></figure>

<figure><img src=".gitbook/assets/image (13).png" alt=""><figcaption></figcaption></figure>

<figure><img src=".gitbook/assets/image (14).png" alt=""><figcaption></figcaption></figure>

<figure><img src=".gitbook/assets/image (15).png" alt=""><figcaption></figcaption></figure>

<figure><img src=".gitbook/assets/image (16).png" alt=""><figcaption></figcaption></figure>

### `firewall-cmd` CLI

<table><thead><tr><th width="376.4000244140625" valign="top">comandi firewall-cmd</th><th valign="top">Spiegazione</th></tr></thead><tbody><tr><td valign="top"><code>--get-default-zone</code></td><td valign="top">Mostra la zona di default corrente.</td></tr><tr><td valign="top"><code>--set-default-zone=</code><em><code>ZONE</code></em></td><td valign="top">Imposta la zona di default. <br>La zona di default cambia sia al runtime che nella configurazione permanente.</td></tr><tr><td valign="top"><code>--get-zones</code></td><td valign="top">Mostra tutte le zone disponibili.</td></tr><tr><td valign="top"><code>--get-active-zones</code></td><td valign="top">Mostra tutte le zone in uso (con un'interfaccia o una fonte ad essi collegata), insieme alle informazioni relative alla loro interfaccia e fonte.</td></tr><tr><td valign="top"><code>--add-source=</code><em><code>CIDR</code></em><code> [--zone=</code><em><code>ZONE</code></em><code>]</code></td><td valign="top">Instrada tutto il traffico dall'indirizzo IP o dalla rete/maschera di rete alla zona specificata. <br>Se non viene fornita l'opzione <code>--zone=</code>, verrà utilizzata la zona predefinita..</td></tr><tr><td valign="top"><code>--remove-source=</code><em><code>CIDR</code></em><code> [--zone=</code><em><code>ZONE</code></em><code>]</code></td><td valign="top">Rimuovi la regola che instrada tutto il traffico dalla zona che proviene dall'indirizzo IP o dalla rete. <br>Se non viene fornita l'opzione <code>--zone=</code>, viene utilizzata la zona predefinita..</td></tr><tr><td valign="top"><code>--add-interface=</code><em><code>INTERFACE</code></em><code> [--﻿zone=</code><em><code>ZONE</code></em><code>]</code></td><td valign="top">Reindirizza tutto il traffico dall'<em><code>INTERFACE</code></em> alla zona specificata. <br>Se non viene fornita l'opzione <code>--zone=</code>, verrà utilizzata la zona predefinita..</td></tr><tr><td valign="top"><code>--change-interface=</code><em><code>INTERFACE</code></em><code> [--﻿zone=</code><em><code>ZONE</code></em><code>]</code></td><td valign="top">Associa l'interfaccia con una ZONA invece della sua zona attuale. Se non viene fornita l'opzione <code>--zone=</code>, verrà utilizzata la zona predefinita..</td></tr><tr><td valign="top"><code>--list-all [--zone=</code><em><code>ZONE</code></em><code>]</code></td><td valign="top">Elenca tutte le interfacce configurate, le fonti, i servizi e le porte per <em><code>ZONE</code></em>. <br>Se non viene fornita l'opzione <code>--zone=</code>, viene utilizzata la zona predefinita.</td></tr><tr><td valign="top"><code>--list-all-zones</code></td><td valign="top">Questo comando recupererà e mostrerà tutte le configurazioni e i dettagli pertinenti per ogni zona configurata.</td></tr><tr><td valign="top"><code>--add-service=</code><em><code>SERVICE</code></em><code> [--﻿zone=</code><em><code>ZONE</code></em><code>]</code></td><td valign="top">Consenti il traffico verso <em><code>SERVICE</code></em>. <br>Se non viene fornita l'opzione <code>--zone=</code>, verrà utilizzata la zona predefinita.</td></tr><tr><td valign="top"><code>--add-port=</code><em><code>PORT/PROTOCOL</code></em><code> [--﻿zone=</code><em><code>ZONE</code></em><code>]</code></td><td valign="top">Consenti il traffico verso le porte <em><code>PORT/PROTOCOL</code></em>. <br>Se non viene fornita l'opzione <code>--zone=</code>, verrà utilizzata la zona predefinita.</td></tr><tr><td valign="top"><code>--remove-service=</code><em><code>SERVICE</code></em><code> [--﻿zone=</code><em><code>ZONE</code></em><code>]</code></td><td valign="top">Rimuovere <em><code>SERVICE</code></em> dall'elenco consentito per la zona. <br>Se non viene fornita l'opzione <code>--zone=</code>, viene utilizzata la zona predefinita.</td></tr><tr><td valign="top"><code>--remove-port=</code><em><code>PORT/PROTOCOL</code></em><code> [--﻿zone=</code><em><code>ZONE</code></em><code>]</code></td><td valign="top">Rimuovi le porte <em><code>PORT/PROTOCOL</code></em> dall'elenco consentito per la zona. <br>Se non viene fornita l'opzione <code>--zone=</code>, viene utilizzata la zona predefinita.</td></tr><tr><td valign="top"><code>--reload</code></td><td valign="top">Elimina la configurazione runtime e applica la configurazione persistente.</td></tr></tbody></table>

L'esempio seguente imposta la zona predefinita su `dmz`, assegna tutto il traffico proveniente dalla rete `192.168.0.0/24` alla zona `internal` e apre le porte di rete per il servizio `mysql` nella zona `internal` :&#x20;

<pre class="language-bash"><code class="lang-bash"><strong>[root@host ~]\# firewall-cmd --set-default-zone=dmz
</strong><strong>[root@host ~]\# firewall-cmd --permanent --zone=internal --add-source=192.168.0.0/24
</strong><strong>[root@host ~]\# firewall-cmd --permanent --zone=internal --add-service=mysql
</strong><strong>[root@host ~]\# firewall-cmd --reload
</strong></code></pre>

per aggiungere tutto il traffico entrante proveniente dall'indirizzo IPv4 singolo `172.25.25.11` alla zona `public` :&#x20;

<pre class="language-bash"><code class="lang-bash"><strong>[root@host ~]\# firewall-cmd --permanent --zone=public --add-source=172.25.25.11/32
</strong><strong>[root@host ~]\# firewall-cmd --reload
</strong></code></pre>
