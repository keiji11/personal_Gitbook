# Analizzare Server e chiedere supporto

### Analizzare e gestire server remoti

**Descrivere la Console Web**

La console web è un'interfaccia di gestione basata sul web per Red Hat Enterprise Linux. L'interfaccia è progettata per la gestione e il monitoraggio dei server ed è basata sul servizio open-source **Cockpit**.

È possibile utilizzare la console web per monitorare i registri di sistema e visualizzare i grafici delle prestazioni del sistema. Inoltre, è possibile utilizzare il browser web per modificare le impostazioni utilizzando gli strumenti grafici dell'interfaccia della console web, compresa una sessione di terminale interattivo completamente funzionale.

#### Abilitare la console Web

A partire da Red Hat Enterprise Linux 7, la console web è installata per impostazione predefinita in tutte le varianti di installazione, ad eccezione dell'installazione minima. È possibile utilizzare il seguente comando per installare la console web:

`[root@host ~]# dnf install cockpit`&#x20;

Quindi, abilitate e avviate il servizio cockpit.socket, che esegue un server web. Questo passo è necessario se si desidera connettersi al sistema attraverso l'interfaccia web.

`[root@host ~]# systemctl enable --now cockpit.socket`&#x20;

Creato il collegamento simbolico&#x20;

`/etc/systemd/system/sockets.target.wants/cockpit.socket -> /usr/lib/systemd/system/cockpit.socket`&#x20;

Se si utilizza un profilo di firewall personalizzato, è necessario aggiungere il servizio cockpit a firewalld per aprire la porta 9090 nel firewall:

`[root@host ~]# firewall-cmd --add-service=cockpit --permanent`

`[root@host ~]# firewall-cmd --reload`

#### Log in nella Web Console

La console web dispone di un proprio server web. Avviare il browser web per accedere alla console web.

Aprire `https://servername:9090` nel browser web, dove servername è il nome host o l'indirizzo IP del server. La console web protegge la connessione con una sessione Transport Layer Security (TLS). Per impostazione predefinita, il servizio cockpit installa la console Web con un certificato TLS autofirmato. Quando ci si connette alla console web per la prima volta, il browser web probabilmente visualizza un avviso di sicurezza. La pagina man di cockpit-ws(8) fornisce istruzioni su come sostituire il certificato TLS con uno firmato correttamente.

Per accedere alla console web, inserire il nome utente e la password nella schermata di accesso. È possibile accedere con il nome utente e la password di qualsiasi account locale del sistema, compreso l'utente root.

<figure><img src="https://rol.redhat.com/rol/static/static_file_cache/rh124-9.0/support/cockpit_title_bar_privileged.png" alt=""><figcaption><p>Barra dove selezionare i privilegi utente</p></figcaption></figure>

