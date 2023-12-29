---
description: Comandi Linux generali
---

# 💻 Linux commands

### FG e BG

* CTRL+Z = mettere in background.
* Tornare all'attivitá:

```Shell
fg
```

***

### Formattare l'history

* ```Shell
  HISTTIMEFORMAT="%Y-%m-%d %T "

  history
  ...
  ```

in .bashrc :

* ```Shell
  # non aggiunge duplicati o comandi che iniziano con lo spazio nell'history:
  HISTCONTROL=ignoreboth

  ```

***

#### Scrivere usando i caratteri di escape:

* `echo -e "hello world\n\nwelcome here!"`

***

#### Visualizzare i processi che consumano piú CPU e salvarli in un file:

* `top -b -n l > top.$(date +%F)_$(date +%R)`

***

#### Mostrare proprietá del sistema corrente:

* `hostnamectl status`

***

#### Visualizzare numero righe senza # (ossia quelle decommentate):

* `grep ^[^#] /etc/ssh/ssh_config | wc -l`&#x20;

***

