# Influenza sulla schedulazione dei processi

## Schedulazione dei processi in Linux

I moderni sistemi informatici utilizzano CPU multi-core e multi-thread in grado di eseguire vari thread di istruzioni simultaneamente. \
I supercomputer più potenti possono avere centinaia o migliaia di CPU con centinaia di core per CPU, elaborando milioni di thread in parallelo. \
Mentre un singolo utente può saturare un tipico sistema desktop, i server aziendali gestiscono migliaia di utenti e richieste, spesso portando alla saturazione della CPU. \
Sistemi come Linux usano il _<mark style="color:yellow;">time-slicing</mark>_ per la gestione dei processi, cambiando rapidamente tra i thread su ogni core disponibile per sembrare che molti processi avvengano insieme.

### Prioritá dei processi

Le _priorità dei processi_ stabiliscono l'importanza di ciascun processo. Linux utilizza le _politiche di schedulazione_ per organizzare e prioritizzare i processi in base al tempo CPU. \
Queste politiche possono gestire richieste interattive, elaborazione batch non interattiva o requisiti di applicazioni in tempo reale.

Le politiche di schedulazione in tempo reale usano priorità e code, mentre per le politiche _normali_ si utilizza il Completely Fair Scheduler (CFS). Il CFS organizza i processi in un albero di ricerca binario. I processi normali, gestiti dalla politica <mark style="color:yellow;">`SCHED_NORMAL`</mark> o <mark style="color:yellow;">`SCHED_OTHER`</mark>, hanno una priorità statica di 0, inferiore rispetto ai processi in tempo reale. \
L'algoritmo CFS ordina i thread sulla base del tempo CPU precedente accumulato, favorendo quelli con meno utilizzo.

### Valori <mark style="color:yellow;">`nice`</mark>&#x20;

Il valore _<mark style="color:yellow;">nice</mark>_ di un processo influisce sull'ordine della coda di esecuzione in un sistema operativo. Questo valore varia da -20 (priorità aumentata) a 19 (priorità diminuita), con un valore predefinito di 0. I valori nice vengono ereditati dai processi figli, e mentre tutti gli utenti possono ridurre la priorità, solo l'utente <mark style="color:yellow;">`root`</mark> può aumentarne una.

Nei sistemi con CPU non saturata, le priorità influiscono sull'ordine con cui i thread occupano le CPU. In sistemi saturati, i thread di priorità più alta occupano le CPU per primi. Il Completely Fair Scheduler bilancia l'importanza dei processi, i valori nice e il tempo di CPU utilizzato, per garantire un'equa distribuzione del tempo di CPU.

### Permessi di modifica valori <mark style="color:yellow;">`nice`</mark>&#x20;

Gli utenti privilegiati possono diminuire il valore di _<mark style="color:yellow;">nice</mark>_ di un processo, aumentando la sua priorità nel sistema e facendolo eseguire più spesso, riducendo il tempo CPU disponibile per altri processi. Gli utenti non privilegiati possono solo aumentare il valore di _<mark style="color:yellow;">nice</mark>_ dei propri processi, abbassando così la loro priorità, e non possono modificare i valori di _<mark style="color:yellow;">nice</mark>_ dei processi di altri utenti.

### Visualizzazione dei valori <mark style="color:yellow;">`nice`</mark>&#x20;

<figure><img src="../.gitbook/assets/image (2) (1) (1).png" alt=""><figcaption></figcaption></figure>

Osserviamo le colonne <mark style="color:yellow;">**PR**</mark> (3a) e <mark style="color:yellow;">**NI**</mark> (4a) del comando `top` :&#x20;

<pre class="language-bash"><code class="lang-bash"><strong>Tasks: 192 total,   1 running, 191 sleeping,   0 stopped,   0 zombie
</strong>%Cpu(s):  0.0 us,  1.6 sy,  0.0 ni, 96.9 id,  0.0 wa,  0.0 hi,  1.6 si,  0.0 st
MiB Mem :   5668.6 total,   4655.6 free,    470.1 used,    542.9 buff/cache
MiB Swap:      0.0 total,      0.0 free,      0.0 used.   4942.6 avail Mem

    PID USER      PR  NI    VIRT    RES    SHR S  %CPU  %MEM     TIME+ COMMAND
      1 root      20   0  172180  16232  10328 S   0.0   0.3   0:01.49 systemd
      2 root      20   0       0      0      0 S   0.0   0.0   0:00.01 kthreadd
      3 root       0 -20       0      0      0 I   0.0   0.0   0:00.00 rcu_gp
      4 root       0 -20       0      0      0 I   0.0   0.0   0:00.00 rcu_par_gp
</code></pre>

<pre class="language-bash"><code class="lang-bash"><strong>[user@host ~]$ ps axo pid,comm,nice,cls --sort=-nice
</strong>  PID COMMAND          NI CLS
   33 khugepaged       19  TS
   32 ksmd              5  TS
  814 rtkit-daemon      1  TS
    1 systemd           0  TS
    2 kthreadd          0  TS
    5 kworker/0:0-cgr   0  TS
    7 kworker/0:1-rcu   0  TS
    8 kworker/u4:0-ev   0  TS
   15 migration/0       -  FF
...output omitted...
</code></pre>

## Avviare processi con valori di <mark style="color:yellow;">`nice`</mark> settati dall'utente

Quando un processo viene creato, eredita il valore <mark style="color:yellow;">`nice`</mark> del suo genitore. Se avviato da CLI, eredita il valore <mark style="color:yellow;">`nice`</mark> dal processo della shell. \
I nuovi processi generalmente partono con un valore <mark style="color:yellow;">`nice`</mark> predefinito di 0. \
Nell'esempio seguente, un processo viene avviato dalla shell e visualizza il valore <mark style="color:yellow;">`nice`</mark> del processo. \
Si noti l'uso dell'opzione _PID_ nel comando `ps` per specificare l'output richiesto. Questo comando è scelto per dimostrazione per il suo basso consumo di risorse:

<pre class="language-bash"><code class="lang-bash"><strong>[user@host ~]$ sleep 60 &#x26;
</strong>[1] 2667
<strong>[user@host ~]$ ps -o pid,comm,nice 2667
</strong>  PID COMMAND          NI
 2667 sleep            0
</code></pre>

Usa l'opzione `-n` del comando `nice` per applicare un valore di priorità definito dall'utente al processo di avvio. \
Il valore predefinito è aggiungere 10 al valore attuale di priorità del processo. \
Nell'esempio seguente, viene avviato un processo in background con un valore di priorità definito dall'utente di `15` e ne viene visualizzato il risultato:

<pre class="language-bash"><code class="lang-bash"><strong>[user@host ~]$ nice -n 15 sleep 60 &#x26;
</strong>[1] 2673
<strong>[user@host ~]$ ps -o pid,comm,nice 2740
</strong>  PID COMMAND          NI
 2740 sleep            15
</code></pre>

## Modificare il valore di <mark style="color:yellow;">`nice`</mark>di un processo giá esistente

Comando <mark style="color:blue;">`renice`</mark> :&#x20;

<pre class="language-bash"><code class="lang-bash"><strong>[user@host ~]$ renice -n 19 2740
</strong>2740 (process ID) old priority 15, new priority 19
</code></pre>

Si puó modificare anche attraverso il comando `top` cliccando il tasto **r** sul processo.
