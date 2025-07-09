# Individuare e risolvere i problemi con Red Hat Insights

La piattaforma Red Hat Insights analizza i dati ricevuti e visualizza i risultati sul sito

[console\_Insights](https://console.redhat.com/insights)

<figure><img src="../.gitbook/assets/image (2).png" alt=""><figcaption><p>Schema ad alto livello di RH Insights</p></figcaption></figure>

#### Installazione Client di Red Hat Insights

Insights è incluso in Red Hat Enterprise Linux 9 come parte dell'abbonamento. Le versioni precedenti di Red Hat Enterprise Linux richiedono l'installazione del pacchetto insights-client sul sistema. Il pacchetto insights-client ha sostituito il pacchetto redhat-access-insights a partire da Red Hat Enterprise Linux 7.5. La sezione seguente fornisce una guida dettagliata per installare il pacchetto insights-client e per registrare il sistema a Red Hat Insights.

Il client Insights aggiorna periodicamente i metadati forniti a Insights. Utilizzare il comando insights-client per aggiornare i metadati del client.

```bash
[root@host ~] insights-client 
Starting to collect Insights data for host.example.com
Uploading Insights data.
Successfully uploaded report from host.example.com to account 1460291.
View details about this system on console.redhat.com:
https://console.redhat.com/insights/inventory/dc480efd-4782-417e-a496-cb33e23642f0
```

#### Registrazione a RHEL System e Red Hat Insights

La registrazione di un server RHEL in Red Hat Insights è un'operazione rapida.

Registrate interattivamente il sistema con il servizio Red Hat Subscription Management.

<pre><code><strong>[root@host ~]# subscription-manager register --auto-attach
</strong></code></pre>

Assicurarsi che il pacchetto `insights-client` sia installato sul sistema. Il pacchetto è installato per default sui sistemi RHEL 8 e successivi.

<pre><code><strong>[root@host ~]# dnf install insights-client
</strong></code></pre>

Utilizzare il comando `insights-client --register` per registrare il sistema con il servizio Insights e caricare i metadati iniziali del sistema.

<pre><code><strong>[root@host ~]# insights-client --register
</strong></code></pre>

Su Red Hat Insights (`https://console.redhat.com/insights`), assicurarsi di aver effettuato l'accesso e che il sistema sia visibile nella sezione Inventario dell'interfaccia web.

## Navigazione nella console Red Hat Insights

<figure><img src="../.gitbook/assets/image (2) (1).png" alt=""><figcaption><p>Inventario di Insights sul portale Cloud</p></figcaption></figure>

Rilevare i problemi di configurazione con il **servizio Advisor**:

<figure><img src="../.gitbook/assets/image (3).png" alt=""><figcaption></figcaption></figure>

Valutare la sicurezza con il **servizio di vulnerabilità**:

<figure><img src="../.gitbook/assets/image (4).png" alt=""><figcaption></figcaption></figure>

#### Analizzare la conformità utilizzando il servizio Compliance

Il **servizio Compliance** analizza i sistemi e ne riporta il livello di conformità a un criterio OpenSCAP. Il progetto OpenSCAP implementa strumenti per verificare la conformità di un sistema rispetto a un insieme di regole. Red Hat Insights fornisce le regole per valutare i vostri sistemi rispetto a diversi criteri, come ad esempio il Payment Card Industry Data Security Standard (PCI DSS).

#### Aggiornare i pacchetti con il servizio Patch

Il **servizio Patch** elenca gli avvisi di prodotto Red Hat applicabili ai vostri sistemi. Può anche generare un Ansible Playbook, che può essere eseguito per aggiornare i pacchetti RPM pertinenti per gli avvisi applicabili. Per accedere all'elenco degli avvisi per un sistema specifico, utilizzare il menu `Patch → Sistemi`. Fare clic sul pulsante Applica tutti gli avvisi applicabili per generare il Playbook Ansible per un sistema.

<figure><img src="../.gitbook/assets/image (5).png" alt=""><figcaption><p>patchare un sistema</p></figcaption></figure>

#### Attivare gli alerts con il servizio Policies

Utilizzando il **servizio Policies**, è possibile creare regole per monitorare i sistemi e inviare avvisi quando un sistema non rispetta le regole. Red Hat Insights valuta le regole ogni volta che un sistema sincronizza i suoi metadati. È possibile accedere al servizio Criteri dal menu Criteri.

<figure><img src="../.gitbook/assets/image (6).png" alt=""><figcaption><p>dettaglio di un ruolo custom</p></figcaption></figure>

#### Inventario, Remediation Playbook e Monitoraggio delle sottoscrizioni

La pagina **Inventario** fornisce un elenco dei sistemi registrati con Red Hat Insights. La colonna _Ultimo visto_ visualizza l'ora dell'ultimo aggiornamento dei metadati per ogni sistema. Facendo clic sul nome di un sistema, è possibile esaminare i suoi dettagli e accedere direttamente ai _servizi Advisor, Vulnerabilità, Conformità e Patch_ per quel sistema.

La pagina delle **Remediations** elenca tutti i playbook Ansible creati per la correzione. È possibile scaricare i playbook da questa pagina.

Utilizzando la pagina **Subscription** , è possibile monitorare l'utilizzo dell'abbonamento Red Hat.
