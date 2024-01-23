# 😁 Controllo Accessi ai file

### chown

```bash
# settare accesso al file come utente:gruppo
chown user:pool /data
```

### chmod

```bash
# settare i permessi sui gruppi: 
chmod g+rwx /data
# oppure allo stesso modo:
chmod 740 /data # 7 = u -> rwx ; 4 = g -> r ; 0 = o -> NULL
```

