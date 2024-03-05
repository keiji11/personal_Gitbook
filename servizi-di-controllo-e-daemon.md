# 👺 Servizi di controllo e Daemon

## Identificare i processi di sistema avviati automaticamente

### _<mark style="background-color:red;">SYSTEMD</mark>_

**É il primo processo del sistema! PID 1**

SYSTEMD è un sistema di init e gestore di sistema per Linux, usato come standard in molte distribuzioni moderne. Gestisce l'inizializzazione di sistema tramite unità (servizi, mount point, socket, timer, dispositivi, target) e include strumenti per identificare processi avviati automaticamente al boot.

#### Elenco dei servizi con `systemctl`

Per visualizzare tutti i servizi attivi al momento, puoi utilizzare il comando:

```bash
systemctl list-units --type=service
systemctl list-units -t service
```

Questo comando mostra un elenco di tutti i servizi systemd attualmente attivi. Per vedere anche i servizi che sono falliti o che non sono stati avviati, puoi aggiungere l'opzione `--all`:

```bash
systemctl list-units -t service -a
```

#### Abilitare e Disabilitare Servizi

Per configurare l'avvio automatico di servizi specifici durante il boot, `systemctl` mette a disposizione i comandi `enable` e `disable`. Per esempio, per abilitare il servizio `example.service` in modo che si avvii automaticamente all'accensione del sistema, utilizza il comando:

```bash
systemctl enable example.service
```

Allo stesso modo, per impedire che un servizio venga avviato all'avvio, puoi usarne la disabilitazione:

```
systemctl disable example.service
```

È importante notare che disabilitare un servizio non lo ferma se è già in esecuzione. Per fare ciò, dovrai usare il comando `stop`:

```bash
systemctl stop example.service
```

#### Verificare lo Stato di un Servizio

Per controllare lo stato corrente di un servizio, inclusi i dettagli su se sia attivo, inattivo o abbia riscontrato errori, puoi utilizzare:

```bash
systemctl status example.service
```

Questo comando fornisce informazioni dettagliate sul servizio, compresi i log recenti, che possono essere utili per la risoluzione dei problemi.

#### Abilitazione Ritardata dei Servizi

Alcuni servizi potrebbero aver bisogno di essere avviati dopo il raggiungimento di determinate condizioni nel sistema. `systemd` permette di gestire anche questi casi attraverso lo scheduling di servizi con il tipo `timer`.

Per visualizzare tutti i timer attivi, usa:

```bash
systemctl list-timers --all
```

Questo ti darà una panoramica dei timer configurati sul tuo sistema, inclusi quelli che avviano servizi in modo ritardato o periodico.

#### Abilitare in modo persistente i Servizi

Per abilitare in modo persistente ed avviare immediatamente il servizio Nginx sul tuo sistema, esegui il seguente comando:

```bash
systemctl enable --now nginx.service
```

Questo comando assicura che Nginx si avvii automaticamente all'avvio del sistema, oltre a farlo partire immediatamente.

#### Verificare le Dipendenze dei Servizi e Mascherare Servizi

Per visualizzare le dipendenze di un servizio specifico, puoi utilizzare il comando `systemctl list-dependencies`. Questo è utile per capire come i vari servizi sono interconnessi.

```bash
systemctl list-dependencies nginx.service
```

Per impedire l'avvio di un servizio, anche nel caso in cui altri servizi lo richiedano, è possibile "mascherare" tale servizio. Mascherare un servizio lo rende inaccessibile agli altri servizi, impedendone l'avvio automatizzato o manuale. Per mascherare e smascherare il servizio Nginx, segui questi comandi:

```bash
# Per mascherare il servizio Nginx
systemctl mask nginx.service

# Per smascherare il servizio Nginx
systemctl unmask nginx.service
```

