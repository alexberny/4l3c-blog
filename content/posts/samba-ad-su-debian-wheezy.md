---
title: "Samba AD su Debian Wheezy"
date: 2014-10-19T12:34:00Z
draft: false
tags:
  - Samba
  - Debian
---


Questa guida vuole aiutare ad installare Samba4 e Bind come server di DNS su Debian Wheezy. Ho scelto di utilizzare SerNet Enterprise Samba al posto dei pacchetti presenti nel repository di Debian. Per chi non la conoscesse, Enterprise Samba si occupa da anni di patchare e pacchettizzare Samba per le distro Linux *Enterprise*.

Con poche modifiche la guida è valida anche per altre distro Linux supportate da SerNet Samba.

### Configurazioni

I parametri di configurazione di esempio seguiti in questa guida sono:

- Nome del server: **dominioad**
- Nome del dominio: **test.miarete.it**
- Nome REALM KERBEROS o NETBIOS:  **TEST**
- IP del server: **192.168.1.80**
- Server Role: **DC**
- Utente amminstratore del dominio: **Administator** (attenzione alla A maiuscola!)
- Password dell'utente Administrator: **P4ssWord** (per ora la password deve avere almeno 7 caratteri ed essere abbastanza complessa)

### Prerequisiti

I servizi di Active Directory si basano su due servizi essenziali: il DNS e la sincronizzazione oraria.
Si consiglia pertanto di installare un servizio NTP sul server in questo modo:

``` bash {linenos=table}
# apt-get install ntpdate
# ntpdate it.pool.ntp.org
# apt-get install ntp
```

Vedremo invece più avanti l'installazione di Bind9 come server DNS del dominio. Nel frattempo vediamo come impostare i file di configurazione di Debian.

Configuriamo ora l'ip statico del server editando la parte relativa alla scheda di rete nel file */etc/network/interfaces*

``` bash {linenos=table}
auto eth0
iface eth0 inet static
  address 192.168.1.80
  netmask 255.255.255.0
  gateway 192.168.1.1
```

Impostiamo il nome del server nel file */etc/hostname*

``` bash {linenos=table}
dominioad.test.miarete.it
```

Ed infine modifichiamo il file */etc/hosts*

``` bash {linenos=table}
127.0.0.1 localhost
#127.0.1.1 dominioad dominioad.test.miarete.it <-- commentare!!!
192.168.1.80 dominioad dominioad.test.miarete.it
```

L'entry 127.0.1.1 va commentata in quanto puo' portare Bind a restituire risultati errati.

Rendiamo effettive le modifiche con un reboot del server.

Installiamo infine il transport-https di apt.

``` bash {linenos=table}
# apt-get install apt-transport-https
```

### Installazione di Samba

Innanzitutto occorre registrarsi al sito [http://www.enterprisesamba.com](http://www.enterprisesamba.com) ed eseguire il login.
Una volta entrati ci vengono offerte le indicazioni per effettuare l'installazione di Samba e la username e la accesskey per effettuare il download.

Editiamo il file */etc/apt/sources.list* aggiungendo il repository:

``` bash {linenos=table}
#
# SerNet Samba 4.1 Packages
#
# (debian-wheezy)
#
deb https://USERNAME:ACCESSKEY@download.sernet.de/packages/samba/4.1/debian wheezy main
deb-src https://USERNAME:ACCESSKEY@download.sernet.de/packages/samba/4.1/debian wheezy main
```

Attenzione a sostituire *USERNAME:ACCESSKEY* con quelle presenti nella pagina di SerNet Samba!

Importiamo la chiave GPG di SerNet Samba

``` bash {linenos=table}
# wget http://ftp.sernet.de/pub/sernet-samba-keyring_1.4_all.deb
# dpkg -i sernet-samba-keyring_1.4_all.deb
```

Eseguiamo un update di apt, l'installazione di Samba, di alcuni pacchetti suggeriti e di Bind9

``` bash {linenos=table}
# apt-get update
# apt-get install sernet-samba-ad
# apt-get install fam acl attr quota
# apt-get install bind9
```

### Configurazione di Samba

Configuriamo Samba come server AD:

``` bash {linenos=table}
# samba-tool domain provision --use-rfc2307 --function-level=2008_R2 --interactive
Realm [TEST.MIARETE.IT]:
 Domain [TEST]:
 Server Role (dc, member, standalone) [dc]:
 DNS backend (SAMBA_INTERNAL, BIND9_FLATFILE, BIND9_DLZ, NONE) [SAMBA_INTERNAL]: BIND9_DLZ
Administrator password:
Retype password:
Looking up IPv4 addresses
Looking up IPv6 addresses
No IPv6 address will be assigned
Setting up secrets.ldb
Setting up the registry
Setting up the privileges database
Setting up idmap db
Setting up SAM db
Setting up sam.ldb partitions and settings
Setting up sam.ldb rootDSE
Pre-loading the Samba 4 and AD schema
Adding DomainDN: DC=test,DC=miarete,DC=it
Adding configuration container
Setting up sam.ldb schema
Setting up sam.ldb configuration data
Setting up display specifiers
Modifying display specifiers
Adding users container
Modifying users container
Adding computers container
Modifying computers container
Setting up sam.ldb data
Setting up well known security principals
Setting up sam.ldb users and groups
Setting up self join
Adding DNS accounts
Creating CN=MicrosoftDNS,CN=System,DC=test,DC=miarete,DC=it
Creating DomainDnsZones and ForestDnsZones partitions
Populating DomainDnsZones and ForestDnsZones partitions
See /var/lib/samba/private/named.conf for an example configuration include file for BIND
and /var/lib/samba/private/named.txt for further documentation required for secure DNS updates
Setting up sam.ldb rootDSE marking as synchronized
Fixing provision GUIDs
A Kerberos configuration suitable for Samba 4 has been generated at /var/lib/samba/private/krb5.conf
Setting up fake yp server settings
Once the above files are installed, your Samba4 server will be ready to use
Server Role:           active directory domain controller
Hostname:              dominioad
NetBIOS Domain:        TEST
DNS Domain:            test.miarete.it
DOMAIN SID:            S-1-5-21-1601127499-651562562-2929973973
```

L'opzione `--interactive` ci permette di configurare i parametri in modo interattivo.

L'opzione `--use-rfc2307` abilita l'estensione NIS che ci permette di gestire gli utenti/gruppi di Unix con il tool di gestione degli utenti/gruppi di AD di Microsoft. Questa opzione è facoltativa ma non puo' essere attivata in seguito. L'abilitazione non comporta nessun effetto aggiuntivo se essa non viene utilizzata.

L'opzione `--function-level=2008_R2` imposta il livello di funzionalità di Samba a quello di un Windows Server 2008 R2.

Durante la configurazione Samba vi propone i parametri di default, in questo caso vanno tutti bene (ruolo, dominio, REALM) tranne il server DNS che impostiamo a BIND9_DLZ per utilizzare Bind come server DNS.

Ultimo avviso: la password di Administrator è impostata con una scadenza a 42 giorni, vedremo in seguito come modificare tale opzione.

Modifichiamo il file */etc/default/sernet-samba* impostando i seguenti parametri

``` bash {linenos=table}
# SAMBA_START_MODE defines how Samba should be started. Valid options are one of
#   "none"    to not enable it at all,
#   "classic" to use the classic smbd/nmbd/winbind daemons
#   "ad"      to use the Active Directory server (which starts the smbd on its own)
# (Be aware that you also need to enable the services/init scripts that
# automatically start up the desired daemons.)
SAMBA_START_MODE="ad"

# SAMBA_RESTART_ON_UPDATE defines if the the services should be restarted when
# the RPMs are updated. Setting this to "yes" effectively enables the
# functionality of the try-restart parameter of the init scripts.
SAMBA_RESTART_ON_UPDATE="yes"
```

Lanciamo il demone di Samba

``` bash {linenos=table}
# service sernet-samba-ad start
```

ed effettuiamo un paio di test

``` bash {linenos=table}
# smbclient -L localhost -U%
Domain=[TEST] OS=[Unix] Server=[Samba 4.1.12-SerNet-Debian-9.wheezy]

        Sharename       Type      Comment
        ---------       ----      -------
        netlogon        Disk
        sysvol          Disk
        IPC$            IPC       IPC Service (Samba 4.1.12-SerNet-Debian-9.wheezy)
Domain=[TEST] OS=[Unix] Server=[Samba 4.1.12-SerNet-Debian-9.wheezy]

        Server               Comment
        ---------            -------

        Workgroup            Master
        ---------            -------
# smbclient //localhost/netlogon -UAdministrator -c 'ls'
Enter Administrator's password:
Domain=[TEST] OS=[Unix] Server=[Samba 4.1.12-SerNet-Debian-9.wheezy]
  .                                   D        0  Sun Oct 19 15:14:48 2014
  ..                                  D        0  Sun Oct 19 15:14:55 2014

                61467 blocks of size 131072. 48297 blocks available
```

Bene, se tutto è andato a buon fine possiamo procedere con la configurazione di Bind

### Configurazione di Bind

Per prima cosa abilitiamo le zone di Samba in Bind aggiungendo al file */etc/bind/named.conf.local* la riga:

``` bash {linenos=table}
include "/var/lib/samba/private/named.conf";
```

Poi effettuiamo le seguenti modifiche al file */etc/bind/named.conf.options* :

- abilitiamo il forwards verso un server DNS esterno (in questo caso OpendDNS)

``` bash {linenos=table}
forwarders {
208.67.222.222;
208.67.220.220;
};
```

- abilitiamo le sottoreti che posso effettuare query al nostro server

``` bash {linenos=table}
allow-query { 127.0.0.1; 192.168.1.0/24; } ;
allow-transfer { none; } ;
notify no;
empty-zones-enable no;
allow-recursion { 127.0.0.1; 192.168.1.0/24; } ;
```

- modifichiamo la riga

``` bash {linenos=table}
//auth-nxdomain no;    # conform to RFC1035
auth-nxdomain yes;
```

- abilitiamo l'autenticazione Samba aggiungendo la riga

``` bash {linenos=table}
tkey-gssapi-keytab "/var/lib/samba/private/dns.keytab";
```

e modifichiamo i permessi della directory *private* di Samba

``` bash {linenos=table}
# chown root:bind private
```

Modifichiamo il file */etc/resolv.conf* per aggiungere il dominio alla ricerca e facciamo puntare il server DNS sul server Bind interno

``` bash {linenos=table}
search test.miarete.it
nameserver 192.168.1.80
```

Effettuiamo un *reboot* per rendere effettive tutte le modifiche e un po' di test sul server DNS

``` bash {linenos=table}
# host -t SRV _ldap._tcp.test.miarete.it
_ldap._tcp.test.miarete.it has SRV record 0 100 389 dominioad.test.miarete.it.
# host -t SRV _kerberos._udp.test.miarete.it
_kerberos._udp.test.miarete.it has SRV record 0 100 88 dominioad.test.miarete.it.
# host -t A dominioad.test.miarete.it
dominioad.test.miarete.it has address 192.168.1.80
```

Se non ci sono errori il server DNS risulta ben impostato.

### Configurazione di Kerberos

Installiamo e configuriamo Kerberos.

``` bash {linenos=table}
# apt-get install krb5-user
```

Copiamo o linkiamo i file di configurazione creati da Samba

``` bash {linenos=table}
cp /var/lib/samba/private/krb5.conf /etc/krb5.conf
```

e modifichiamolo così:

``` bash {linenos=table}
[libdefaults]
        default_realm = TEST.MIARETE.IT
        dns_lookup_realm = true
        dns_lookup_kdc = true
```

A questo punto un paio di test

``` bash {linenos=table}
# kinit Administrator@TEST.MIARETE.IT
Password for Administrator@TEST.MIARETE.IT:
Warning: Your password will expire in 41 days on Sun Nov 30 16:02:49 2014
```

e

``` bash {linenos=table}
# klist
Ticket cache: FILE:/tmp/krb5cc_0
Default principal: Administrator@TEST.MIARETE.IT

Valid starting       Expires              Service principal
19/10/2014 17:13:35  20/10/2014 03:13:35  krbtgt/TEST.MIARETE.IT@TEST.MIARETE.IT
        renew until 20/10/2014 17:13:31
```

### Aggiunta di un client Windows

Ora non resta che aggiungere i client al server. Unica accortezza: ogni client Windows deve avere l'IP del server Samba come indirizzo DNS primario.

### Amminstrazione di Samba con i tool di Microsoft in ambiente Windows

Samba puo' essere gestito con i Remote Server Administration Tool di Microsoft in ambiente Windows.
Tali strumenti devono essere scaricati in base al proprio sistema operativo. Questi sono i link per

[Windows Vista](http://www.microsoft.com/it-it/download/details.aspx?id=21090)

[Windows 7](http://www.microsoft.com/it-it/download/details.aspx?id=7887)

[Windows 8](http://www.microsoft.com/it-it/download/details.aspx?id=28972)

[Windows 8.1](http://www.microsoft.com/it-it/download/details.aspx?id=39296)

[Windows 10](http://www.microsoft.com/en-us/download/details.aspx?id=44280)

### Gestione di Samba da linea di comando in ambiente Linux

Il comando *samba-tool* è lo strumento in ambito Linux per la gestione di Samba. Consultare la pagina man di [samba-tool](https://www.samba.org/samba/docs/man/manpages/samba-tool.8.html) per una descrizione dettagliata.

### Trucchetti

``` bash {linenos=table}
- Resettare la password di Administrator

# smbpasswd  Administrator
New SMB password:
Retype new SMB password:


- Mostrare le policy della password

samba-tool domain passwordsettings show


- Togliere l'expire date alla password di Samba

# samba-tool user setexpiry Administrator --noexpiry
Expiry for user 'Administrator' disabled.


- Abilitare/disabilitare la complessità della password

samba-tool domain passwordsettings set --complexity=on|off


- Disabilitare la history della password

samba-tool domain passwordsettings set --history-length=0


- Modificare la lunghezza minima della password

samba-tool domain passwordsettings set --min-pwd-length=TUOVALORE


- Modificare la durata minima/massima della passoword

samba-tool domain passwordsettings set --min-pwd-age=TUOVALORE
samba-tool domain passwordsettings set --max-pwd-age=TUOVALORE
```

### Riferimenti

Questa guida è stata redatta anche grazie al contributo di altre guide:

[Samba wiki](https://wiki.samba.org/index.php/Main_Page)

[Samba e OpenLDAP: creare un controller di dominio Active Directory con Debian Wheezy](http://guide.debianizzati.org/index.php/Samba_e_OpenLDAP:_creare_un_controller_di_dominio_Active_Directory_con_Debian_Wheezy)

[L' onnipresente Google](http://www.google.com)
