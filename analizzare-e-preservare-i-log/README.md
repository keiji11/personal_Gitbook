# Analizzare e preservare i log

## Architettura dei log di sistema

Red Hat Enterprise Linux utilizza un sistema di log standard basato sul protocollo syslog per registrare i messaggi di sistema. I servizi `systemd-journald` e `rsyslog` gestiscono questi messaggi.

### Servizio `systemd-journald`

* Raccoglie messaggi da:
  * Kernel di sistema
  * Fasi iniziali di avvio
  * Output e errori standard dai demoni
  * Eventi syslog
* Riorganizza i log in formato strutturato
* Scrive in un diario indicizzato che non persiste tra i riavvii

### Servizio `rsyslog`

* Legge i messaggi syslog dal `systemd-journald`
* Processo e registra in file di log permanenti in `/var/log`
* Ordina i messaggi per tipo e priorità

<table data-full-width="true"><thead><tr><th valign="top">Log file</th><th width="800" valign="top">Type of stored messages</th></tr></thead><tbody><tr><td valign="top"><code>/var/log/messages</code></td><td valign="top"><p>La maggior parte dei messaggi di syslog vengono registrati qui. </p><p>Fanno eccezione messaggi relativi ad autenticazione/sicurezza, server email, esecuzione <em>job</em> schedulati e messaggi puramente di <em>debug</em>.</p></td></tr><tr><td valign="top"><code>/var/log/secure</code></td><td valign="top">Messaggi <em>syslog</em> sugli eventi di sicurezza e autenticazione.</td></tr><tr><td valign="top"><code>/var/log/maillog</code></td><td valign="top">Messaggi <em>syslog</em> riguardanti il server mail.</td></tr><tr><td valign="top"><code>/var/log/cron</code></td><td valign="top">Messaggi <em>syslog</em> riguardanti l'esecuzione dei <em>job</em> schedulati.</td></tr><tr><td valign="top"><code>/var/log/boot.log</code></td><td valign="top">Messaggi console <em>non-syslog</em> riguardanti l'avvio del sistema.</td></tr></tbody></table>

### Note Aggiuntive

* Alcune applicazioni, come Apache, gestiscono autonomamente i loro log sotto `/var/log`.
