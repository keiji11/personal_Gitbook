# Convalidare la configurazione di rete

`ip link`

* Il comando "ip link" elenca tutte le interfacce di rete disponibili sul sistema.

`ip addr`

* Il comando "ip addr" mostra informazioni sui dispositivi e gli indirizzi IP, sia IPv4 che IPv6. Un'interfaccia può avere più indirizzi.

`ip -s link`

* "ip -s link" mostra statistiche sulle prestazioni della rete, come pacchetti ricevuti, trasmessi, errori.

`ping`

* "ping" e "ping6" verificano la connettività rispettivamente su IPv4 e IPv6. È possibile usare "ping6" per trovare altri nodi IPv6 sulla rete locale.

`ip route`

* "ip route" mostra la tabella di routing IPv4, "ip -6 route" quella IPv6.

`traceroute` e `tracepath`

* "tracepath/traceroute" (IPv4) e "tracepath6/traceroute -6" (IPv6) tracciano il percorso del traffico attraverso i router.

`ss`

* "ss" mostra statistiche sui socket TCP/UDP, sostituendo "netstat". Mostra porte in ascolto e connessioni stabilite.

`mtr <IP o indirizzo "DNSato">`

* "mtr" é un ottimo tool per le statistiche di route

**Tabella. Opzioni per ss e netstat**

<table><thead><tr><th width="204">Option</th><th>Description</th></tr></thead><tbody><tr><td><code>-n</code></td><td>Mostra numeri invece di nomi per interfacce e porte.</td></tr><tr><td><code>-t</code></td><td>Mostra TCP sockets.</td></tr><tr><td><code>-u</code></td><td>Mostra UDP sockets.</td></tr><tr><td><code>-l</code></td><td>Mostra solo socket in ascolto</td></tr><tr><td><code>-a</code></td><td>Mostra tutti i sockets.</td></tr><tr><td><code>-p</code></td><td>Mostra i porocessi che usano i sockets.</td></tr><tr><td><code>-A inet</code></td><td>Visualizza le connessioni attive (ma non i socket in ascolto) per la famiglia di indirizzi <code>inet</code>. Ciò significa ignorare i socket di dominio UNIX locali. Per il comando <code>ss</code>, vengono visualizzate sia le connessioni IPv4 che quelle IPv6. Per il comando <code>netstat</code>, vengono visualizzate solo le connessioni IPv4. (Il comando <code>netstat -A inet6</code> mostra le connessioni IPv6, e il comando <code>netstat -46</code> mostra contemporaneamente le connessioni IPv4 e IPv6.)</td></tr></tbody></table>

`nmap -sS <IP o indirizzo "DNSato">`

* ci permette di scannerizzare porte ed apporta statistiche approfondite
