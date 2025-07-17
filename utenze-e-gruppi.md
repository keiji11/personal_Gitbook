# Utenze e gruppi

### ID Processi

* processi -> PID
* user -> UID
* group -> GID
* files -> INODES

### ID's

* `id [USER]`
  * `0` é l'utente privilegiato
* `ps -au` = comando per processi attivi che indicano l'utente

### Passwd

* `/etc/passwd` = directory dove abbiamo le utenze
* `grep student /etc/passwd`
  * `-> USERNAME:PASSWORD(x):UID:GID:Description:HOME_PATH:SHELL_PATH`

### Groups

* `/etc/group` = directory dove abbiamo i gruppi

### Shadow

* `/etc/shadow` = directory dove abbiamo le password (criptate)
  * `-> USERNAME:CRYPT_PSSWD:`

#### Shell root per utente(se omesso é root):

* `sudo -i [USERNAME]`

#### Shell login per utente(se omesso é root):

* `sudo -l [USERNAME]`

### Sudoers

* `/etc/sudoers`&#x20;
* `/etc/sudoers.d/file_config`
  * `-> devops ALL=(ALL) NOPASSWD:ALL` -> puó essere eseguito da dovunque e non serve password
* `sudo su [USERNAME]` = privilegi solo su directory ma utenza rimane
* `su - [USERNAME] oppure sudo -i` = privilegi directory e utenza cambia

### Gestire utenti locali

#### Useradd

* `useradd <USERNAME>`
*   `usermod [OPTIONS] [ARG] <USERNAME>`

    | `usermod` options:      | Usage                                                                                                                                                                           |
    | ----------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
    | `-a, --append`          | Use it with the `-G` option to add the supplementary groups to the user's current set of group memberships instead of replacing the set of supplementary groups with a new set. |
    | `-c, --comment COMMENT` | Add the `COMMENT` text to the comment field.                                                                                                                                    |
    | `-d, --home HOME_DIR`   | Specify a home directory for the user account.                                                                                                                                  |
    | `-g, --gid GROUP`       | Specify the primary group for the user account.                                                                                                                                 |
    | `-G, --groups GROUPS`   | Specify a comma-separated list of supplementary groups for the user account.                                                                                                    |
    | `-L, --lock`            | Lock the user account.                                                                                                                                                          |
    | `-m, --move-home`       | Move the user's home directory to a new location. You must use it with the `-d` option.                                                                                         |
    | `-s, --shell SHELL`     | Specify a particular login shell for the user account.                                                                                                                          |
    | `-U, --unlock`          | Unlock the user account.                                                                                                                                                        |


* `/etc/login.defs` = definizioni per utenti
* `userdel` = cancellare utenza

#### Passwd

* `passwd <USERNAME>` = cambio password
* `passwd -S <USERNAME>` = stato password

#### Extra

* posso usare:
  * `grep developers /etc/group`
  * <mark style="background-color:green;">`getent`</mark>` ``group developers`

### Gestire gruppi

#### Range GID o UID DI SISTEMA:

* `grep SYS_GID_M /etc/login.defs`
* `grep SYS_UID_M /etc/login.defs`

#### Range GID o UID:

* `grep GID_M /etc/login.defs`
* `grep UID_M /etc/login.defs`

#### Aggiungere gruppi all'utente oltre a quello principale:

* `useradd -G wheels,devops,admins <USERNAME>`

#### Cambiare gruppo primario:

* `newgrp admins`

#### Cambiare id a gruppo:

* `groupmod -g <NUOVO_GID> <GROUPNAME>`

### Gestire password

* `grep <USERNAME> /etc/shadow`
  * -> `<USERNAME>:<HASHED_PASSWORD>:<DAYS_FROM_LAST_PSWD_CHANGE>:<DAYS_AFTER_USER_CAN_CHANGE_PSWD>:<MAX_DAYS_PSWD_EXPIRED>:<DAYS_BEFORE_WARNING>:<N_DAYS_INACTIVE>:<DAYS_TO_EXPIRING_PSWD>`
  * -> ex. `student:`HASH o `!!`(NO\_PSWD)`:19130:0:99999:7::::`

#### Cambio password policies:

* <pre><code><strong>chage -m 0 -M 90 -W 7 -I 14 sysadmin05
  </strong></code></pre>

```
# Imposto data scadenza dopo 30 giorni
[root@host ~]# chage -E $(date -d "+30 days" +%F) cloudadmin10 3
# Visualizzo data scadenza
[root@host ~]# chage -l cloudadmin10 | grep "Account expires" 4
Account expires						: Apr 09, 2022
```

<figure><img src=".gitbook/assets/image (1) (1) (1) (1) (1) (1).png" alt=""><figcaption><p>chage Schema</p></figcaption></figure>

* <pre><code><strong># Cambio forzato password per l'utente cloudadmin10
  </strong><strong>[root@host ~]# chage -d 0 cloudadmin10
  </strong></code></pre>
