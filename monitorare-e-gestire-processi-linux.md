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

<figure><img src=".gitbook/assets/image (3).png" alt=""><figcaption><p>Lifecycle dei processi</p></figcaption></figure>

<figure><img src=".gitbook/assets/image (4).png" alt=""><figcaption><p>Stati dei procesi Linux</p></figcaption></figure>

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



[^1]: IMPORTANTE!!!\
    stato processi

[^2]: !!!!!!!!!!!!!!
