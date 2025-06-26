# 🥅 Gestione Networking

<table data-header-hidden data-full-width="true"><thead><tr><th width="187"></th><th></th></tr></thead><tbody><tr><td><strong>Scopo</strong></td><td>Configurare interfacce di rete ed impostazioni su server Red Hat Enterprise Linux.</td></tr><tr><td><strong>Obiettivi</strong></td><td><ul><li>Concetti fondamentali di indirizzamento di rete e routing di un server.</li><li>Testare ed ispezionare l'attuale configurazione di rete con utility da linea di comando.</li><li>Gestire impostazioni di rete e dispositivi con il comando <code>nmcli</code>.</li><li>Modificare la configurazione di rete modificando i file di configurazione.</li><li>Configurare l'hostname statico di un server e la sua risoluzione dei nomi e testare i risultati.</li></ul></td></tr><tr><td><strong>Sezioni</strong></td><td><ul><li>Descrivere i concetti di rete (e Quiz)</li><li>Validare la configurazione di rete (e Esercitazione guidata)</li><li>Configurare la rete dalla linea di comando (e Esercitazione guidata)</li><li>Modificare i file di configurazione di rete (e Esercitazione guidata)</li><li>Configurare hostname e risoluzione dei nomi (e Esercitazione guidata)</li></ul></td></tr><tr><td><strong>Lab</strong></td><td>Gestione Networking</td></tr></tbody></table>

### Descrivere i concetti di rete

#### Obiettivi

Descrivere i concetti fondamentali di indirizzamento e instradamento di rete per un server.

**Modello di Rete TCP/IP**

Il modello di rete TCP/IP è un insieme di protocolli di comunicazione a quattro livelli che descrive come le comunicazioni di dati vengano pacchettizzate, indirizzate, trasmesse, instradate e ricevute tra computer su una rete.

Il protocollo è specificato dal RFC 1122, Requisiti per gli Host Internet - Strati di Comunicazione.

Di seguito sono riportati i quattro strati del modello di rete TCP/IP:

* **Applicazione -** Ogni applicazione ha specifiche per la comunicazione in modo che client e server possano comunicare tra piattaforme diverse. I protocolli comuni includono SSH, HTTPS (web sicuro), FTP (condivisione di file) e SMTP (consegna di posta elettronica).
* **Trasporto -** TCP e UDP sono protocolli di trasporto. TCP è una comunicazione affidabile orientata alla connessione, mentre UDP è un protocollo di datagrammi senza connessione. I protocolli applicativi possono utilizzare porte TCP o UDP. Un elenco di porte note e registrate si trova nel file /etc/services. Quando un pacchetto viene inviato sulla rete, la combinazione della porta del servizio e dell'indirizzo IP forma un socket. Ogni pacchetto ha un socket di origine e un socket di destinazione. Queste informazioni possono essere utilizzate durante il monitoraggio e il filtraggio del traffico di rete.
* **Internet -** Lo strato internet, o strato di rete, trasporta i dati dall'host sorgente all'host destinatario. I protocolli IPv4 e IPv6 sono protocolli dello strato internet. Ogni host ha un indirizzo IP e un prefisso per determinare gli indirizzi di rete. I router sono usati per connettere le reti.
* **Link -** Lo strato di collegamento, o strato di accesso al mezzo, fornisce la connessione ai media fisici. I tipi più comuni di reti sono Ethernet cablata (802.3) e Wi-Fi wireless (802.11). Ogni dispositivo fisico ha un indirizzo di controllo dell'accesso al mezzo (MAC), noto anche come indirizzo hardware, per identificare la destinazione dei pacchetti sul segmento di rete locale.

<div data-full-width="false"><figure><img src="../.gitbook/assets/image (5) (1).png" alt=""><figcaption></figcaption></figure></div>

