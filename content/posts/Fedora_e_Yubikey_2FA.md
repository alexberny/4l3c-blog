---
title: "Fedora e YubiKey: 2FA login"
date: 2020-12-30T22:32:22Z
draft: false
tags:
  - Fedora
  - YubuKey
  - sicurezza
---

In termini di sicurezza, si sa, la prudenza non è mai troppa. Così da qualche anno ho iniziato ad utilizzare il 2FA (secondo fattore di autenticazione) su tutti i siti e app che lo permettono. Oltre all'ormai desueto sms e alla più moderna app authenticator per la generazione di OTP, ho deciso di provare anche un token fisico compatibile con gli standard *FIDO2* e *WebAuthn*. La scelta è caduta su una (anzi due per ragioni di sicurezza) [YubiKey 5 NFC](https://www.yubico.com/products/yubikey-5-overview/ "YubiKey 5 NFC"). La motivazione che mi ha portato a questo token esula dall'intento di questa guida ma in rete si trovano molte pagine di paragone con altri prodotti simili.

All'inizio ho utilizzato la chiave solo per l'accesso ai siti compatibili con gli standard FIDO2 e WebAuthn, approfondendo però le potenzialità del prodotto, ho scoperto che è utilizzabile anche come U2F (Universal 2nd Factor) per il login su Linux. Ho deciso quindi di attivarlo subito sulla Fedora del mio notebook. Dopo oltre un anno di utilizzo, complice il passaggio alla Fedora 33, ho voluto scrivere questa guida per aiutare quanti volessero, anche solo per curiosità, provare ad aggiungere un fattore di sicurezza al proprio pc. Con pochi adattamenti è possibile eseguire l'installazione anche sulle altre distro RH derivate e con qualche sforzo in più anche su quelle Debian-based.

Per chi avesse una versione diversa della YubiKey, consiglio di verificare sul sito del produttore la compatibilità con lo standard U2F.

## Installazione del software necessario

Lanciare il seguente comando per installare i pacchetti necessari.

```bash
dnf install pam-u2f pamu2fcfg yubikey-manager
```

## Configurazione

Fedora, utilizza *authselect* per configurare i metodi di accesso al sitema. In questo modo non è necessario modificare a mano i vari file *pam.d/\** o *nsswitch*. Per abilitare il modulo per *u2f* lanciare il seguente comando:

```bash
authselect select sssd with-pam-u2f-2fa without-nullok
```

Per ogni utente bisogna creare il file che conterrà le chiavi associate. Per una configurazione globale è possibile creare il file in */etc*.

```bash
mkdir ~/.config/Yubico
```

L'ultimo passaggio consiste nell'abilitare le singole chiavi.
Per la prima si usa:

```bash
pamu2fcfg > ~/.config/Yubico/u2f_keys
```

toccare il pulsante a sfioramento quando lampeggia.

Configurare le ulteriori chiavi lanciando, per ognuna, il seguente comando:

```bash
pamu2fcfg -n >> ~/.config/Yubico/u2f_keys
```

**Importante !!!** Prima di riavviare il sistema, testare tutte le chiavi digitando in un terminale

```bash
sudo echo test
```

Si deve ottenere

```bash
sudo echo test
Please touch the device.
```

a questo punto il token inizierà a lampeggiare e basterà sfiorarlo ed il sistema chiederà la password.

Il risultato finale è:

```bash
sudo echo test
Please touch the device.
[sudo] password di user:
test
```

**E' importante ripete il test con tutte le chiavi prima di effettuare il riavvio.**

In rete si trovano anche guide per l'utilizzo della chiave crittografica contenuta nel token al posto dell' u2f, ma ho deciso di non applicarla in quanto il pc deve essere connesso ad Internet per la verifica della stessa e questo non sempre è possibile: quante volte in treno si ha una ricezione scarsa o nulla!

Link utili:

- Pagina ufficiale Yubico del modulo pam-u2f: <https://developers.yubico.com/pam-u2f/>

- Pagina ufficiale di authselect: <https://github.com/authselect/authselect>

- Una guida sul post-installazione di Fedora: <https://mutschler.eu/linux/install-guides/fedora-post-install/#get-thunderbolt-dock-to-work-and-adjust-monitors>
