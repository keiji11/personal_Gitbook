# 📄 Creare, visionare e modificare file di testo

### Redirect

Output Redirection Operators

<figure><img src=".gitbook/assets/image.png" alt=""><figcaption><p>Output Redirection Operators</p></figcaption></figure>

| Usage          | Explanation                                                         | Visual aid                                                                                                                                                                                                         |
| -------------- | ------------------------------------------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| > _file_       | Redirect `stdout` to overwrite a file.                              | <table data-header-hidden><thead><tr><th></th></tr></thead><tbody><tr><td><img src="https://rol.redhat.com/rol/static/static_file_cache/rh124-9.0/edit/redirection-overview.svg" alt=""></td></tr></tbody></table> |
| >> _file_      | Redirect `stdout` to append to a file.                              | <table data-header-hidden><thead><tr><th></th></tr></thead><tbody><tr><td><img src="https://rol.redhat.com/rol/static/static_file_cache/rh124-9.0/edit/redirection-append.svg" alt=""></td></tr></tbody></table>   |
| 2> _file_      | Redirect `stderr` to overwrite a file.                              | <table data-header-hidden><thead><tr><th></th></tr></thead><tbody><tr><td><img src="https://rol.redhat.com/rol/static/static_file_cache/rh124-9.0/edit/redirection-error.svg" alt=""></td></tr></tbody></table>    |
| 2> /dev/null   | Discard `stderr` error messages by redirecting them to `/dev/null`. | <table data-header-hidden><thead><tr><th></th></tr></thead><tbody><tr><td><img src="https://rol.redhat.com/rol/static/static_file_cache/rh124-9.0/edit/dev-null.svg" alt=""></td></tr></tbody></table>             |
| > _file_ 2>&1  | Redirect `stdout` and `stderr` to overwrite the same file.          | <table data-header-hidden><thead><tr><th></th></tr></thead><tbody><tr><td><img src="https://rol.redhat.com/rol/static/static_file_cache/rh124-9.0/edit/combine-overwrite.svg" alt=""></td></tr></tbody></table>    |
| &> _file_      |                                                                     |                                                                                                                                                                                                                    |
| >> _file_ 2>&1 | Redirect `stdout` and `stderr` to append to the same file.          | <table data-header-hidden><thead><tr><th></th></tr></thead><tbody><tr><td><img src="https://rol.redhat.com/rol/static/static_file_cache/rh124-9.0/edit/combine-append.svg" alt=""></td></tr></tbody></table>       |
| &>> _file_     |                                                                     |                                                                                                                                                                                                                    |

***

### Tee

#### Equivalente a `>` ma mostra a schermo il contenuto del file:

* `ip a`` `**`| tee`**` ``my_system`

#### Equivalente a `>>` ma mostra a schermo il contenuto del file:

* `ip a`` `**`| tee -a`**` ``my_system`

***

### Heredoc

* `cat > my_file << EOF`

&#x20;           `> This is my multiline file`

&#x20;           `> and now finish.`

&#x20;           `> EOF`&#x20;

***

### Edit files interactively

#### VIM Editor

* `vimtutor`
* Lesson 3.2 -> `r+<RIGHT_CHAR>` = rimpiazza il carattere con quello indicato
* Lesson 3.3 -> `ce` = cambiare il restante della parola, cancella e bisogna riscrivere con la modalitá INSERT; `cc` = stessa cosa ma cancella fino a fine riga
* Lesson 4.1 -> `CTRL+g` = mostra info alla fine della pagina visiva; `G` = sposta il cursore a fine pagina; `gg` = sposta il cursore ad inzio pagina; `<N_line>`G = posiziona il cursore sulla linea indicata
* Lesson 4.3 -> `%` = posizionato il cursore su una parentesi aperta indicherá la corrispondente chiusa
* Lesson 4.4 -> `:s/old/new/g`  = substitute command; `:#,#s/old/new/g` = "da linea # a linea # sostituisci i caratteri; `:%s/old/new/g` = sostituisce nell'intero file; `:$s/old/new/gc` = come il precedente ma chiede conferma
* Lesson 5.1-5.2 -> `:!<COMMAND>` = esegue il comando fuori da vim
* Lesson 5.3 -> in modalitá VISUAL selezionare il testo, premere `:` e apparirá `:'<,'>` in modo che solo il testo selezionato verrá "copiato" e si puó salvare digitando `:'<,'>w NOME_FILE`&#x20;
* Lesson 5.4 -> `:r NOME_FILE` = copia il contenuto del FILE esterno in vim (sotto il cursore)
* Lesson 6.3 -> `:R+SOME_STRING` = rimpiazza piú di un carattere
* Lesson 6.5 -> SET OPTIONS -> `:set ic` = ignore case; `:set hls is` = evidenzia ricerche("highlight search"); `:set noic` = annulla ignore case; `:nohlsearch` = annulla evidenziatori
* Lesson 7.3 -> COMPLETION -> `CTRL+D` = completa i comandi se iniziamo a digitare `:e` ad esempio

PATH CONFIG:  `/etc/vimrc`

PATH migliore, creare: `~/.vimrc`

#### Command Mode

{% code lineNumbers="true" %}
```sh
cw - change word 
d$ - cancella dal cursore fino a fine riga
ZZ - esci e salva
```
{% endcode %}

#### Visual Mode

{% code lineNumbers="true" %}
```bash
Visual Line - SHIFT+V
              > - indenta la linea
Visual block - CTRL+V

```
{% endcode %}

***

