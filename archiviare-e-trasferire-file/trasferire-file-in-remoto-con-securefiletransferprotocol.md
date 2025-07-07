# Trasferire file in remoto con SecureFileTransferProtocol

## SFTP

Si usa per fare l'upload e il download dei file tramite server SSH (suite OpenSSH) _**in modo interattivo**_.

```bash
[user@host ~]$ sftp remoteuser@remotehost
remoteuser@remotehost's password: password
Connected to remotehost.
sftp>
```

```bash
# Comandi sftp interattivo, come bash o sh con l'aggiunta di qualche comando in piú:
sftp>help
...
sftp>lpwd # Locale
sftp>pwd # Remota

# UPLOAD (-r ricorsivo)
sftp> put -r /etc/
Uploading /etc/ to /home/remoteuser/etc/
...

# DOWNLOAD
sftp> get /etc/yum.conf
Fetching /etc/yum.conf to yum.conf
/etc/yum.conf                              100%  813     0.8KB/s   00:00
```

### SFTP non interattivo

```bash
# [remoteuser@] -> opzionale
[user@host ~]$ sftp remoteuser@remotehost:/home/remoteuser/remotefile
Connected to remotehost.
Fetching /home/remoteuser/remotefile to remotefile
remotefile                                               100%    7    15.7KB/s   00:00
```

## Trasferimento file con SecureCopyProtocol

#### <mark style="color:red;">WARNING</mark>

Nelle versioni precedenti a RHEL 9, il comando `scp` era basato su un protocollo storico `rcp` che non era stato progettato con considerazioni di sicurezza. \
&#xNAN;_**Il protocollo****&#x20;****`scp`****&#x20;****ha una nota****&#x20;**<mark style="color:red;">**vulnerabilità**</mark>**&#x20;****di iniezione di codice tale che un attaccante potrebbe eseguire comandi arbitrari sul server remoto. Per questo motivo, il protocollo****&#x20;****`scp`****&#x20;****non è trattato in questo corso.**_
