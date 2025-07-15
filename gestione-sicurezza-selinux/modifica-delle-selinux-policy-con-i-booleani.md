# Modifica delle SELinux policy con i booleani

## Booleani SELinux

Gli sviluppatori definiscono comportamenti permessi delle applicazioni tramite policy mirate di SELinux. I Booleani SELinux consentono di abilitare o disabilitare comportamenti opzionali specifici per l'applicazione. Questi comportamenti devono essere identificati e selezionati per ciascuna applicazione specifica.

### Utilizzo dei Comandi

* **Lista dei Booleani**: Usa <mark style="color:orange;">`getsebool`</mark> per visualizzare i Booleani disponibili e il loro stato corrente.
* **Modifica dei Booleani**: Usa <mark style="color:yellow;">`setsebool`</mark> per abilitare o disabilitare comportamenti; aggiungi l'opzione <mark style="color:yellow;">`-P`</mark> per rendere la modifica persistente.

Solo gli utenti privilegiati possono modificare i Booleani SELinux. Per ulteriori dettagli, consultare le pagine man di SELinux tramite il pacchetto `selinux-policy-doc`

```bash
[root@host ~]\# getsebool -a
abrt_anon_write --> off
abrt_handle_event --> off
abrt_upload_watch_anon_write --> on
...output omitted...
```

### Esempio httpd policy booleana

Il servizio `httpd` include il Booleano `httpd_enable_homedirs`, che consente la condivisione delle directory home con `httpd`. \
Le directory home non sono condivise usando `https` di default e non sono disponibili tramite un browser.

```bash
[root@host ~]\# getsebool httpd_enable_homedirs
httpd_enable_homedirs --> off
```

Quando abilitato, il servizio `httpd` condivide le directory home etichettate con il contesto file `user_home_dir_t`. Gli utenti possono quindi accedere e gestire i file della loro directory home da un browser.

## Gestione policy booleane

<pre class="language-bash"><code class="lang-bash"><strong>[root@host ~]\# semanage boolean -l | grep httpd_enable_homedirs
</strong>httpd_enable_homedirs          (off   ,  off)  Allow httpd to enable homedirs
# Senza -P LA MODIFICA É SOLO TEMPORANEA!!!!!
<strong>[root@host ~]\# setsebool httpd_enable_homedirs on
</strong><strong>[root@host ~]\# semanage boolean -l | grep httpd_enable_homedirs
</strong>httpd_enable_homedirs          (on   ,  off)  Allow httpd to enable homedirs
<strong>[root@host ~]\# getsebool httpd_enable_homedirs
</strong>httpd_enable_homedirs --> on
</code></pre>

Per elencare solo i Booleani con un'impostazione corrente diversa dall'impostazione predefinita all'avvio, utilizza il comando <mark style="color:orange;">`semanage boolean -l -C`</mark> ( `-l` = _list_, `-C` = _current)_ : &#x20;

<pre class="language-bash"><code class="lang-bash"><strong>[root@host ~]\# semanage boolean -l -C
</strong>SELinux boolean                State  Default Description

httpd_enable_homedirs          (on   ,   off)  Allow httpd to enable homedirs
</code></pre>

modifica persistente ( opzione <mark style="color:red;">`-P`</mark> ):

<pre class="language-bash"><code class="lang-bash"><strong>[root@host ~]\# setsebool -P httpd_enable_homedirs on
</strong><strong>[root@host ~]\# semanage boolean -l | grep httpd_enable_homedirs
</strong>httpd_enable_homedirs          (on   ,   on)  Allow httpd to enable homedirs
</code></pre>
