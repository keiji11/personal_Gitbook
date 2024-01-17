# 🚫 Permessi ed accesso file

### Sticky bit

* `chmod`` `**`1`**`750` o `chmod o`**`+t`**
* serve ad impedire l'eliminazione dei file all'interno di una cartella il cui proprietario non é l'utente che vuole eliminare i file

### SetGID bit

* `chmod`` `**`2`**`750` o `chmod g`**`+s`**
* agisce sia sulle dir che sui file e dá gli stessi permessi di gruppo della directory padre

### SetUID bit

* `chmod`` `**`4`**`750` o `chmod u`**`+s`**
* Solo su file e permette azioni come se si fosse proprietario del file

## umask

* sottrae le cifre ottali del permesso iniziale:

<figure><img src=".gitbook/assets/image (2).png" alt="tabella UMASK"><figcaption><p>TABELLA umask</p></figcaption></figure>

