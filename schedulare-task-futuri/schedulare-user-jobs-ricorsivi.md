# Schedulare user jobs ricorsivi

## Descrizione

Nei sistemi Red Hat Enterprise Linux, il demone `crond` è attivato e avviato di default per eseguire lavori ricorsivi in modo programmato. _<mark style="background-color:yellow;">**Questo demone legge file di configurazione sia a livello di sistema che a livello utente**</mark>**.**_ Ogni utente può modificare il proprio file di lavoro tramite :

`crontab -e`

Se non si specifica un reindirizzamento del risultato, `crond` invia eventuali output o errori per email al proprietario del lavoro.

## Comandi principali per schedulare user jobs ricorsivi

<table><thead><tr><th valign="top">Comando</th><th valign="top">Spiegazione uso</th></tr></thead><tbody><tr><td valign="top"><code>crontab -l</code></td><td valign="top">Elenca i jobs per l'user corrente</td></tr><tr><td valign="top"><code>crontab -r</code></td><td valign="top">Rimuove tutti i jobs per l'user corrente</td></tr><tr><td valign="top"><code>crontab -e</code></td><td valign="top">Modifica i jobs per l'user corrente</td></tr><tr><td valign="top"><code>crontab </code><em><code>filename</code></em></td><td valign="top">Rimuove tutti i jobs, e li rimpiazza con i jobs scritti nel <em>filename</em>. Il comando usa <code>stdin</code> quando nessun file é specificato</td></tr></tbody></table>

Un utente privilegiato potrebbe utilizzare l'opzione `-u` del comando `crontab` per gestire i processi di un altro utente.&#x20;

<mark style="background-color:red;">**Il comando**</mark><mark style="background-color:red;">**&#x20;**</mark><mark style="background-color:red;">**`crontab`**</mark><mark style="background-color:red;">**&#x20;**</mark><mark style="background-color:red;">**non è mai utilizzato per gestire i processi di sistema**</mark> e _<mark style="background-color:yellow;">**l'uso del comando**</mark><mark style="background-color:yellow;">**&#x20;**</mark><mark style="background-color:yellow;">**`crontab`**</mark><mark style="background-color:yellow;">**&#x20;**</mark><mark style="background-color:yellow;">**come utente**</mark><mark style="background-color:yellow;">**&#x20;**</mark><mark style="background-color:yellow;">**`root`**</mark><mark style="background-color:yellow;">**&#x20;**</mark><mark style="background-color:yellow;">**non è consigliato a causa della possibilità di sfruttare processi personali configurati per essere eseguiti come**</mark><mark style="background-color:yellow;">**&#x20;**</mark><mark style="background-color:yellow;">**`root`**</mark><mark style="background-color:yellow;">**.**</mark>_&#x20;

## Descrizione formato user jobs

## Uso del comando `crontab -e`

Il comando `crontab -e` apre l'editor `vim` per default, a meno che la variabile d'ambiente `EDITOR` non sia impostata per un altro editor. Ogni attività deve usare una linea unica nel file `crontab`. Segui queste raccomandazioni per inserimenti validi quando scrivi jobs ricorrenti:

* **Linee vuote:** per migliorare la leggibilità
* **Commenti:** su linee che iniziano con il simbolo `#`
* **Variabili d'ambiente:** nel formato `NOME=valore`, che influenzano tutte le linee dopo dove sono dichiarate.
* **Impostazioni standard delle variabili:**
  * `SHELL`: per dichiarare la shell utilizzata per interpretare le linee successive nel file `crontab`.
  * `MAILTO`: determina chi riceverà l'output via email.

#### Nota

L'invio di email richiede una configurazione aggiuntiva del sistema per un server di posta locale o un relay SMTP.

I campi nel file `crontab` appaiono nel seguente ordine:

* Minuti
* Ore
* Giorno del mese
* Mese
* Giorno della settimana
* Comando

Il comando viene eseguito quando i campi _Giorno del mese_ o _Giorno della settimana_ hanno lo stesso valore, diverso dal carattere `*`. Ad esempio, per eseguire un comando l'11esimo giorno di ogni mese e ogni venerdì alle 12:15 (formato 24 ore), utilizza il seguente formato del job:

```bash
15 12 11 * Fri comando
```

## Guida alla Sintassi:

I primi cinque campi seguono le stesse regole di sintassi:

* Utilizza il carattere `*` per eseguire ogni possibile istanza del campo.
* Un numero per specificare minuti o ore, una data o un giorno della settimana.
  * Nei giorni della settimana, `0` equivale a Domenica, `1` a Lunedì, `2` a Martedì, e così via. `7` equivale anche a Domenica.
* Usa `x-y` per un intervallo, includendo i valori `x` e `y`.
* Usa `x,y` per liste. Le liste possono includere intervalli, ad esempio, `5,10-13,17` nella colonna `Minuti`, per eseguire un'attività a 5, 10, 11, 12, 13, e 17 minuti dopo l'ora.

### Intervalli e Abbreviazioni

* `*/x` indica un intervallo di `x`; ad esempio, `*/7` nella colonna `Minuti` esegue un'attività ogni sette minuti.
* Sono utilizzate abbreviazioni in inglese di 3 lettere per i mesi o i giorni della settimana, come Jan, Feb, Mon, Tue.

### Esecuzione del Comando

* L'ultimo campo contiene il comando completo con opzioni e argomenti da eseguire con il shell predefinito.
* Se il comando contiene un segno di percentuale non _escape_ (`%`), quel segno di percentuale è trattato come un carattere di nuova riga, e tutto ciò che segue sarà passato al comando come input `stdin`.

## Esempi di user jobs Ricorrenti

Il seguente job esegue il comando `/usr/local/bin/yearly_backup` esattamente alle 09:00 il 3 febbraio di ogni anno:

```
0 9 3 2 * /usr/local/bin/yearly_backup
```

Il seguente job invia un'email che contiene la parola `Chime` al proprietario di questo job ogni cinque minuti tra le 09:00 e le 16:00, ma solo ogni venerdì di luglio:

```
*/5 9-16 * Jul 5 echo "Chime"
```

La gamma `9-16` per le ore significa che il timer del job inizia alla nona ora (09:00) e continua fino alla fine della sedicesima ora (16:59). Il job inizia a essere eseguito alle `09:00` con l'ultima esecuzione alle `16:55`, perché cinque minuti dopo le `16:55` è `17:00`, che è oltre l'intervallo di ore specificato.

Se per le ore è specificata una gamma invece di un singolo valore, allora tutte le ore all'interno della gamma coincideranno. Pertanto, con le ore `9-16`, questo esempio si ripete ogni cinque minuti dalle 09:00 alle 16:55.

### Esempio

Questo _job_ di esempio invia l'output come email, poiché `crond` riconosce che il _job_ ha permesso all'output di andare al canale `STDIO` senza redirezione. Poiché i cron job funzionano in un ambiente di background senza un dispositivo di output (noto come _terminale di controllo_), `crond` memorizza l'output e crea un'email per inviarla all'utente specificato nella configurazione. Per i _jobs_ di sistema, l'email viene inviata all'account `root`.

Il seguente _job_ esegue il comando `/usr/local/bin/daily_report` ogni giorno lavorativo (da lunedì a venerdì) due minuti prima della mezzanotte:

```plaintext
58 23 * * 1-5 /usr/local/bin/daily_report
```

Il seguente _job_ esegue il comando `mutt` per inviare il messaggio email _"Controllo avvenuto"_ al destinatario `developer@example.com` ogni giorno lavorativo alle 9 del mattino:

{% code fullWidth="true" %}
```plaintext
0 9 * * 1-5 mutt -s "Controllo avvenuto" developer@example.com % Ciao, mi sto solo assicurando che tutto sia a posto.
```
{% endcode %}

## Esempi pratici:

*   Configurare un _job_ ricorrente per l'utente `student` che aggiunge la data e l'ora correnti al file `/home/student/my_first_cron_job.txt` ogni 2 minuti, utilizza il comando `date` per ottenere la data e l'ora attuali. Questa attività deve essere impostata per funzionare 1 giorno prima e 1 giorno dopo l'ora corrente e non deve essere eseguita in altri giorni:

    <pre class="language-bash"><code class="lang-bash"><strong>[student@servera ~]$ crontab -e
    </strong>>*/2 * * * Tue-Thu /usr/bin/date >> /home/student/my_first_cron_job.txt

    ...output omitted...
    crontab: installing new crontab
    [student@servera ~]$
    </code></pre>
*   Istruisci la shell: attendi fino a quando il file `/home/student/my_first_cron_job.txt` viene creato a seguito dell'esecuzione corretta del _job_ programmato ricorsivo. Aspetta che il prompt della shell ritorni:

    ```bash
    [student@servera ~]$ while ! test -f my_first_cron_job.txt; do sleep 1s; done
    ```
