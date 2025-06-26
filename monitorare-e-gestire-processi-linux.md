# 🎓 Monitorare e Gestire Processi Linux

## Comandi `ps`

<pre class="language-sh"><code class="lang-sh"># Visualizza tutti i processi, compresi quelli senza terminale di controllo.
# e li sto filtrando dalla shell bash
ps aux | grep bash
> USER    PID    %CPU    %MEM    VSZ    RSS    TTY    <a data-footnote-ref href="#user-content-fn-1">STAT    </a>START    TIME    COMMAND
> root    1       0.1     0.2 171820   16140    ?      Ss     16:47    0:01    /usr/lib/systemd/systemd ...
</code></pre>

```sh
# esempio peggior comando possibile: dd
ps aux | grep dd
```

```sh
# varianti di comandi ps
ps aux | head -5

ps lax | head -5

ps -ef | head -5
```

## Schemi

<figure><img src=".gitbook/assets/image (3) (1).png" alt=""><figcaption><p>Lifecycle dei processi</p></figcaption></figure>

<figure><img src=".gitbook/assets/image (4) (1).png" alt=""><figcaption><p>Stati dei procesi Linux</p></figcaption></figure>

**Table 8.1. Linux Process States**

<table data-full-width="true"><thead><tr><th>Name</th><th>Flag</th><th>Nome e descrizione dello stato kernel-defined</th></tr></thead><tbody><tr><td>Running</td><td><code>R</code></td><td>TASK_RUNNING: il processo è in esecuzione su una CPU o in attesa di essere eseguito. Il processo può eseguire routine utente o routine del kernel (chiamate di sistema), oppure essere in coda e pronto quando si trova nello stato Running (o Runnable).</td></tr><tr><td>Sleeping</td><td><code>S</code></td><td>TASK_INTERRUPTIBLE: Il processo è in attesa di una condizione: una richiesta hardware, l'accesso a una risorsa di sistema o un segnale. Quando un evento o un segnale soddisfa la condizione, il processo torna in <em>Running</em>.</td></tr><tr><td></td><td><code>D</code></td><td>TASK_UNINTERRUPTIBLE: Anche questo processo dorme, ma a differenza dello stato S non risponde ai segnali. Viene utilizzato solo quando l'interruzione del processo potrebbe causare uno stato imprevedibile del dispositivo.</td></tr><tr><td></td><td><code>K</code></td><td>TASK_KILLABLE: Come lo stato D ininterrotto, ma modificato per consentire a un'attività in attesa di rispondere al segnale per ucciderla (uscire completamente). Spesso le utility visualizzano i processi <em>Killable</em> come stato D.</td></tr><tr><td></td><td><code>I</code></td><td>TASK_REPORT_IDLE: Un sottoinsieme dello stato D. Il kernel non conta questi processi nel calcolo della media del carico. Viene utilizzato per i thread del kernel. Sono impostati i flag TASK_UNINTERRUPTIBLE e TASK_NOLOAD. È simile a TASK_KILLABLE ed è anch'esso un sottoinsieme dello stato D. Accetta segnali fatali.</td></tr><tr><td>Stopped</td><td><code>T</code></td><td>TASK_STOPPED: Il processo viene arrestato (sospeso), di solito su segnalazione di un utente o di un altro processo. Il processo può essere continuato (ripreso) da un altro segnale per tornare in esecuzione.</td></tr><tr><td></td><td><code>T</code></td><td>TASK_TRACED: Anche un processo in fase di debug viene temporaneamente fermato e condivide il flag di stato T.</td></tr><tr><td>Zombie</td><td><code>Z</code></td><td>EXIT_ZOMBIE: Un processo figlio segnala al suo genitore la sua uscita. Tutte le risorse, tranne l'identità del processo (PID), vengono rilasciate.</td></tr><tr><td></td><td><code>X</code></td><td>EXIT_DEAD: Quando il genitore pulisce (raccoglie) la struttura del processo figlio rimanente, il processo viene rilasciato completamente. Questo stato non può essere osservato nelle utilità di elencazione dei processi.</td></tr></tbody></table>

## Job di controllo

### Lanciare job in background <mark style="background-color:orange;">&</mark>  :

```sh
# <NOME_APP> &
gnome-characters &
>[1] 122084
```

### Visualizzare jobs attivi:

```sh
jobs
>[N_JOB]+    STATO        COMANDO
>[1]+        Running    sleep 1000
```

### Riportare in foreground i job in background e viceversa:&#x20;

<pre class="language-sh"><code class="lang-sh"># 1 é il numero del job
fg %1

# <a data-footnote-ref href="#user-content-fn-2">CTRL+Z interrompe il processo in esecuzione</a>

bg %1
</code></pre>

### Visualizzare stati jobs:&#x20;

```sh
ps j | grep sleep
```

```sh
ps jT
```

## Uccidere i processi (_KILL_)

### Controllo dei processi con _signal_

**Table 8.2.** _Signal_ fondamentali per la gestione dei processi

<table data-full-width="true"><thead><tr><th>Signal</th><th>Nome</th><th>Definizione</th></tr></thead><tbody><tr><td>1</td><td>HUP</td><td><code>Hangup</code> : Segnala la terminazione del processo di controllo di un terminale. Richiede anche la reinizializzazione del processo (ricarica della configurazione) senza terminazione.</td></tr><tr><td>2</td><td>INT</td><td><code>Keyboard interrupt</code> : Provoca l'interruzione del programma. Può essere bloccata o gestita. Viene inviato premendo la sequenza di tasti INTR (Interrupt)(<strong>Ctrl</strong>+<strong>C</strong>).</td></tr><tr><td>3</td><td>QUIT</td><td><code>Keyboard quit</code> : Simile a SIGINT; aggiunge un dump del processo al momento della terminazione. Viene inviato premendo la sequenza di tasti QUIT (<strong>Ctrl</strong>+<strong>\</strong>).</td></tr><tr><td>9</td><td>KILL</td><td><code>Kill, unblockable</code> : Provoca una brusca terminazione del programma. Non può essere bloccata, ignorata o gestita; è sempre fatale.</td></tr><tr><td><p>15 </p><p><em>default</em></p></td><td>TERM</td><td><code>Terminate</code> : Provoca la terminazione del programma. A differenza di SIGKILL, può essere bloccato, ignorato o gestito. È il modo "pulito" di chiedere a un programma di terminare; consente al programma di completare le operazioni essenziali e di autopulirsi prima di terminare.</td></tr><tr><td>18</td><td>CONT</td><td><code>Continue</code> : Inviato a un processo per riprenderlo se è stato interrotto. Non può essere bloccato. Anche se gestito, riprende sempre il processo.</td></tr><tr><td>19</td><td>STOP</td><td><code>Stop, unblockable</code> : Sospende il processo. Non può essere bloccato o gestito.</td></tr><tr><td>20</td><td>TSTP</td><td><code>Keyboard stop</code> : A differenza di SIGSTOP, può essere bloccato, ignorato o gestito. Inviato premendo la sequenza di tasti di sospensione (<strong>Ctrl</strong>+<strong>Z</strong>).</td></tr></tbody></table>

### Invio Signals su richiesta

```sh
# Lista signals
kill -l
 1) SIGHUP      2) SIGINT      3) SIGQUIT     4) SIGILL      5) SIGTRAP
 6) SIGABRT     7) SIGBUS      8) SIGFPE      9) SIGKILL    10) SIGUSR1
11) SIGSEGV    12) SIGUSR2    13) SIGPIPE    14) SIGALRM    15) SIGTERM
16) SIGSTKFLT  17) SIGCHLD    18) SIGCONT    19) SIGSTOP    20) SIGTSTP
```

* 19\) SIGSTOP = Manutenzione processo. Impedisce ad un processo di ottenere il tempo dalla CPU
* 18\) SIGCONT = Riprende il processo dal `kill`` `**`-19`**` ``PID`
* 2\) SIGINT = é come premere CTRL+c per interrompere un processo
* 15\) SIGTERM = `kill PID`  termina il processo in modo pulito
* 9\) SIGKILL = si usa quando un processo non risponde a SIGTERM&#x20;
* 1\) SIGHUP = fa da RELOAD del processo

### `pidof`

```sh
# indica il PID dell'applicazione/processo
pidof <NOME_APP>
```

### `pkill` e `killall`

```sh
# Segnala uno o più processi che corrispondono ai criteri di selezione.
# I criteri di selezione possono essere il nome di un comando, 
# un processo di cui è proprietario un utente specifico o tutti i processi del sistema.
pkill <NOME_COMANDO o NOME_PROCESSO/I>
# Uccide i processi per nome 
killall <NOME_PROCESSO> 
```

### `w`

Informazioni su utenti che hanno effettuato l'accesso e come lo hanno fatto

<pre class="language-sh"><code class="lang-sh">w -u bob
>USER     TTY      FROM             LOGIN@   IDLE   JCPU   PCPU WHAT
<strong>>bob      tty3                      18:37    5:04   0.03s  0.03s -bash
</strong></code></pre>

### `pgrep`

Cerca pid e nomi processi&#x20;

```sh
# cerca processi relativi all'utenza student (-l = lista)
pgrep -l -u student
```

### `pstree`

Visualizza in modo gerarchico i processi

```sh
# specifica un utente
pstree -p <NOME_USER>
```

## Comando `top`

**Table 8.3. Fundamental Keystrokes in `top` Command**

<table data-full-width="true"><thead><tr><th></th><th>Funzione</th></tr></thead><tbody><tr><td><strong>?</strong> <em>or</em> <strong>h</strong></td><td>Help per i tasti interattivi.</td></tr><tr><td><strong>l</strong>, <strong>t</strong>, <strong>m</strong></td><td>Toggle per linee load, threads e memory header.</td></tr><tr><td><strong>1</strong></td><td>Toggle or individual CPUs or a summary for all CPUs in the header.</td></tr><tr><td><strong>s</strong></td><td>Change the refresh (screen) rate, in decimal seconds (such as 0.5, 1, 5).</td></tr><tr><td><strong>b</strong></td><td>Toggle reverse highlighting for <code>Running</code> processes; the default is bold only.</td></tr><tr><td><strong>Shift</strong>+<strong>b</strong></td><td>Enables bold use in display, in the header, and for <em>Running</em> processes.</td></tr><tr><td><strong>Shift</strong>+<strong>h</strong></td><td>Toggle threads; show process summary or individual threads.</td></tr><tr><td><strong>u</strong>, <strong>Shift</strong>+<strong>u</strong></td><td>Filter for any username (effective, real).</td></tr><tr><td><strong>Shift</strong>+<strong>m</strong></td><td>Sort process listing by memory usage, in descending order.</td></tr><tr><td><strong>Shift</strong>+<strong>p</strong></td><td>Sort process listing by processor use, in descending order.</td></tr><tr><td><strong>k</strong></td><td>Kill a process. When prompted, enter <code>PID</code>, and then <code>signal</code>.</td></tr><tr><td><strong>r</strong></td><td>Renice a process. When prompted, enter <code>PID</code>, and then <code>nice_value</code>.</td></tr><tr><td><strong>Shift</strong>+<strong>w</strong></td><td>Write (save) the current display configuration for use at the next <code>top</code> restart.</td></tr><tr><td><strong>q</strong></td><td>Quit.</td></tr><tr><td><strong>f</strong></td><td>Manage the columns by enabling or disabling fields. You can also set the sort field for <code>top</code>.</td></tr></tbody></table>

### Colonne del comando `top`

* Process ID (`PID`).
* Username dell' **owner** del processo(`USER`).
* La memoria **virtuale** (VIRT) è tutta la memoria utilizzata dal processo, compreso il set residente, le librerie condivise e qualsiasi pagina di memoria mappata o swappata (etichettata VSZ nel comando ps).
* La memoria **residente** (RES) è la memoria fisica utilizzata dal processo, compresi gli oggetti residenti e condivisi (etichettati RSS nel comando ps).
* Stati dei processi (`S`) :
  * `D` = Uninterruptible Sleeping
  * `R` = Running or Runnable
  * `S` = Sleeping
  * `T` = Stopped or Traced
  * `Z` = Zombie
* CPU time (`TIME`) è il tempo totale di elaborazione dall'inizio del processo. Si può selezionare per includere un tempo cumulativo di tutti i child precedenti.
* Nome del comando del processo (`COMMAND`).



[^1]: IMPORTANTE!!!\
    stato processi

[^2]: !!!!!!!!!!!!!!
