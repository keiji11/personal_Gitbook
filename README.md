# 🪜 Gerarchia Linux filesystems

[![GitBook](https://img.shields.io/static/v1?message=Documented%20on%20GitBook\&logo=gitbook\&logoColor=ffffff\&label=%20\&labelColor=5c5c5c\&color=3F89A1)](https://www.gitbook.com/preview?utm_source=gitbook_readme_badge\&utm_medium=organic\&utm_campaign=preview_documentation\&utm_content=link)

## 🪜 Gerarchia Linux filesystems

* ```shell
  # view ad albero, livello 1 di profonditá, directory:
  tree -L 1 /
  ```
* /etc sta per Extended Text Configuration
* Come scoprire se il commando precedente é andato a buon fine?

```shell
echo $?
# se 0 OK
# se 1 KO
```

#### RHEL FS

* il filesystem di RHEL é XFS ed é composto da 512 bytes/file

```shell
xfs_info /

# -i sta per inode
ls -li
```

#### Creare un LINK

{% code lineNumbers="true" %}
```bash
ln <TARGET_FILE> <LINKFILE_NAME>

# esempio: voglio creare un link per raggiungere il file hello_world
ln hello_world hello_link

# contenuti uguali
$(cat hello_world) == $(cat hello_link)
```
{% endcode %}

#### HARD link e SOFT link

* I primi collegano directory ed i secondi solo file

{% code lineNumbers="true" %}
```bash
# HARD LINK o Collegamenti SIMBOLICI per cartelle (per file senza -s)
ln -s training_sources/ learning_material

# Se eliminiamo dove punta il link ( ossia training_resources/ ) si creerá un link interrotto e sará rosso
```
{% endcode %}

#### Matcha nomi dei file con le shell extension

**rename**

{% code lineNumbers="true" %}
```bash
rename <porzione_da_rinominare> <come_rinominare> <quali_file_rinominare>


# esempio
rename .htm .html *
```
{% endcode %}

* `?` sta per "qualsiasi carattere" -> `f????` potrebbe stare per `fetch` o `fired`.
* `cd ~student` mi porterá alla home di quell'user anche se sono nell'env di root ,ad esempio.
* `[!b]*` matcha solo nomi di file che non iniziano per b
* `*[[:digit:]]*` matcha file che contengono numeri nel nome
* `[[:upper:]]*` matcha file i cui nomi iniziano con la maiuscola
* `???*` matcha nomi di file che abbiano almeno tre caratteri

**variabili e stringhe**

{% code lineNumbers="true" %}
```bash
FIRSTNAME=Ricardo
SURNAME="da Costa"

echo "I am $FIRSTNAME_$SURNAME" -> I am da Costa

echo "I am ${FIRSTNAME}_${SURNAME}" -> I am Ricardo_da Costa

echo "I am \$FIRSTNAME $SURNAME" => I am $FIRSTNAME da Costa
```
{% endcode %}

{% code lineNumbers="true" %}
```bash
# cp per fare un backup di un file

ls .
-> my_config
cp my_config{,.bak}
ls .
-> my_config my_config.bak

cp my_config{,.$(date +%F)}
ls .
-> my_config my_config.bak my_config.2023-12-12
```
{% endcode %}
