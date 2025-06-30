# Bash-Script minimale per aumentare la produttivitá

* "She-bang"

Direttiva dell'interprete che indica l'interprete dei comandi e le opzioni di comando per elaborare le righe rimanenti nel file. Per i file di script con sintassi Bash, la prima riga è la seguente direttiva:

```bash
#!/usr/bin/bash
```

* Esecuzione script

Se uno script non si trova in una directory PATH, eseguire lo script utilizzando il suo nome di percorso assoluto, che è possibile determinare interrogando il file con il comando which.

```bash
which hello
> ~/bin/hello

echo $PATH
> /home/user/.local/bin:/home/user/bin:/sbin:/bin:/usr/sbin:/usr/bin:/usr/local/sbin:/usr/local/bin
```

* Caratteri speciali nei commenti:

> Le virgolette singole conservano il significato letterale di tutti i caratteri che racchiudono.

```bash
# \ come ESCAPE:
echo # not a comment
echo \# not a comment
># not a comment
echo \# not a comment \#
># not a comment #
echo '# not a comment #'
># not a comment #
```

> Utilizza le virgolette doppie per sopprimere il globbing (corrispondenza dei pattern dei nomi dei file) e l'espansione della shell, ma consente comunque la sostituzione dei comandi e delle variabili. La sostituzione delle variabili è concettualmente identica alla sostituzione dei comandi, ma può utilizzare una sintassi con parentesi graffe opzionale.&#x20;

```bash
[user@host ~]$ var=$(hostname -s); echo $var
host
[user@host ~]$ echo "***** hostname is ${var} *****"
***** hostname is host *****
[user@host ~]$ echo Your username variable is \$USER.
Your username variable is $USER.
[user@host ~]$ echo "Will variable $var evaluate to $(hostname -s)?"
Will variable host evaluate to host?
[user@host ~]$ echo 'Will variable $var evaluate to $(hostname -s)?'
Will variable $var evaluate to $(hostname -s)?
[user@host ~]$ echo "\"Hello, world\""
"Hello, world"
[user@host ~]$ echo '"Hello, world"'
"Hello, world"
```
