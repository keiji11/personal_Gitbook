# Schedulare task futuri

### Schedulare un job utente in modo differito

#### Attività Utente Differite

Le attività utente differite permettono di eseguire comandi in un momento futuro, come la pianificazione di una manutenzione durante la notte o il reset temporizzato di configurazioni firewall.&#x20;

In Red Hat Enterprise Linux, il comando `at`, installato di default, utilizza il demone `atd` per la pianificazione di queste attività. Gli utenti possono accodare lavori nei _**26 gruppi di coda, da `a` a `z`**_, con le code successive che hanno una priorità di sistema più bassa determinata da valori _**nice**_ più alti.

#### Pianificare Attività Utente Differite

#### Utilizzo del Comando `at` :

Il comando `at` permette di programmare l'esecuzione di lavori in un momento specificato utilizzando `TIMESPEC`.

### Sintassi

```bash
at [TEMPO]              # Inserisci i comandi manualmente
CTRL+D                  # Termina l'input
at [TEMPO] < file       # Esegui comandi da un file
```

### Specifiche Temporali (`TIMESPEC`)

* **Ora**: `02:00pm`, `15:59`
* **Parole Chiave**: `midnight`, `teatime`
* **In Futuro**: `now + 5 min`, `teatime tomorrow`
* **Con Data**: `5pm august 3 2021`

### Esempi

*   ```bash
    at now + 5min < myscript
    ```

    Esegue `myscript` tra 5 minuti.
*   ```bash
    at teatime tomorrow
    ```

    Programma l'esecuzione per il prossimo "teatime".

Consulta `man timespec` per ulteriori dettagli.

### Ispeziona e Gestisci Lavori Utente Differiti

Comando `atq` / `at -l`:

* Visualizzazione dei lavori in sospeso per l'utente corrente.

Esempio di output:

```bash
28  Lun 16 Mag 05:13:00 2022 a user
29  Mar 17 Mag 16:00:00 2022 h user
30  Mer 18 Mag 12:00:00 2022 a user
```

Descrizione della prima linea:

* **28**: ID univoco del _job_.
* **Lun 16 Mag 05:13:00 2022**: Data e ora di esecuzione.
* **a**: Coda predefinita.
* **user**: Proprietario del lavoro.

Note Importanti:

* Gli utenti normali possono gestire solo i propri lavori.
* L'utente `root` può gestire tutti i lavori.

Per ispezionare i comandi di un lavoro:

* Usare `at -c JOBNUMBER`.

### Rimozione _job_ dalla schedulazione

```bash
atrm JOBNUMBER
```

#### Esempi:

*   Schedula un'attività da eseguire tra 2 minuti. Salva l'output del comando date nel file `/home/student/myjob.txt`:

    ```bash
    [student@servera ~]$ echo "date >> /home/student/myjob.txt" | at now +2min
    warning: commands will be executed using /bin/sh
    job 1 at Thu Feb 16 18:51:16 2023
    ```
*   Schedulazione interattiva di un _**job**_ nella coda `g` che runna in orario `teatime` (16:00) stampando il messaggio `It's teatime` nel file `/home/student/tea.txt` :

    <pre class="language-bash"><code class="lang-bash"><strong>[student@servera ~]$ at -q g teatime
    </strong>warning: commands will be executed using /bin/sh
    <strong>at> echo "It's teatime" >> /home/student/tea.txt
    </strong><strong>at> Ctrl+d
    </strong>job 2 at Fri Feb 17 16:00:00 2023
    </code></pre>
*   Pianifica in modo interattivo un altro lavoro con la coda `b` che viene eseguito alle `16:05`. Il lavoro deve stampare il messaggio `"The cookies are good"` nel file `/home/student/cookies.txt` :&#x20;

    ```bash
    [student@servera ~]$ at -q b 16:05
    warning: commands will be executed using /bin/sh
    at> echo "The cookies are good" >> /home/student/cookies.txt
    at> Ctrl+d
    job 3 at Fri Feb 17 16:05:00 2023
    ```

