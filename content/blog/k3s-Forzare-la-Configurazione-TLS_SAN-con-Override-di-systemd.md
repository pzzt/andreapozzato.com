---
title:  'K3S forzare la configurazione TLS SAN con override di systemd'
date: '2025-11-21T19:34:19+01:00'
slug: "k3s-forzare-la-configurazione-tls_san-con-override-di-systemd"
# weight: 1
# aliases: ["/first"]

tags: ["k3s", "kubernetes", "systemd", "tls", "troubleshooting", "sicurezza", "linux"]

author: "Andrea Pozzato"
# author: ["Me", "You"] # multiple authors
showToc: true
TocOpen: true
draft: false
hidemeta: false
comments: false
# description: "Desc Text."
canonicalURL: "https://www.andreapozzato.com/blog/k3s-forzare-la-configurazione-tls_san-con-override-di-systemd"
disableHLJS: false # to disable highlightjs
disableShare: false
hideSummary: false
searchHidden: false
ShowReadingTime: true
ShowBreadCrumbs: true
ShowPostNavLinks: true
ShowWordCount: true
ShowRssButtonInSectionTermList: true
UseHugoToc: true
cover:
    image: "img/017-c2.png" # image path/url
    alt: "abstract" # alt text
    caption: "<text>" # display caption under cover
    relative: false # when using page bundles set this to true
    hidden: false # only hide on current single page
editPost:
    URL: "https://github.com/pzzt/"
    Text: "Suggest Changes" # edit text
    appendFilePath: true # to append file path to Edit link
---
Stai configurando un cluster k3s usando Tailscale e il tuo nodo `agent` non riesce a registrarsi al `server`? L'errore che vedi nei log punta a un problema di certificato, simile a questo:
`x509: certificate is valid for ..., not tuo-agent.tailnet.ts.net`

La soluzione ovvia sembra modificare il file `/etc/rancher/k3s/config.yaml` sul server per aggiungere i nomi host di Tailscale.

Ad esempio, hai aggiunto questo al tuo `config.yaml` senza successo:

```yaml
tls-san:
  - "IL_TUO_IP" # L'IP di Tailscale del server
  - "IL_TUO_FQDN" # Il nome host completo di Tailscale del server
```

## Causa del Problema: Gerarchia delle Configurazioni

La ragione per cui il tuo `config.yaml` viene ignorato è legata alla gerarchia di configurazione di k3s. I parametri passati direttamente al demone `k3s server` **hanno sempre la precedenza** su quelli definiti nel file di configurazione.

Questo scenario si verifica comunemente quando k3s viene installato con un comando che specifica già un parametro, ad esempio:
`curl ... | sh -s - --tls-san=un-valore-specifico`

Quel parametro viene "bucato" nel file del servizio `systemd` di k3s. Di conseguenza, ogni volta che k3s si avvia, legge quel parametro hardcoded dalla riga di comando e ignora completamente la direttiva `tls-san` presente nel file `config.yaml`.

## Soluzione: Forzare i Parametri con un Override di systemd

Invece di modificare direttamente il file di servizio (una pratica sconsigliata che potrebbe essere sovrascritta dagli aggiornamenti), usiamo `systemctl edit` per creare un "override". Questo metodo è il modo pulito e sicuro per sovrascrivere le impostazioni di un servizio.

### Passaggi

1.  **Apri l'editor di override per il servizio k3s:**
    Questo comando aprirà un file vuoto in un editor di testo, pronto per le tue modifiche.
    ```bash
    sudo systemctl edit k3s.service
    ```

2.  **Incolla la configurazione di override:**
    Nel file che si apre, incolla il seguente contenuto.
    ```ini
    [Service]
    ExecStart=
    ExecStart=/usr/local/bin/k3s server --tls-san=IL_TUO_FQDN --tls-san=IL_TUO_IP
    ```
    *   Sostituisci `IL_TUO_FQDN` con il tuo nome host completo (es. `k3s-server.tailnet.ts.net`).
    *   Sostituisci `IL_TUO_IP` con il tuo indirizzo IP (es. l'IP di Tailscale).
    *   La prima riga `ExecStart=` "azzera" il comando di avvio precedente.
    *   La seconda riga imposta il nuovo comando con i parametri `--tls-san` corretti.

3.  **Ferma il servizio k3s e cancella i vecchi certificati:**
    Per forzare la rigenerazione completa del certificato con le nuove impostazioni, è necessario eliminare la directory esistente.
    ```bash
    sudo systemctl stop k3s
    sudo rm -rf /var/lib/rancher/k3s/server/tls
    ```

4.  **Ricarica la configurazione di systemd e riavvia k3s:**
    Applica le modifiche e avvia il servizio, che ora genererà un nuovo certificato.
    ```bash
    sudo systemctl daemon-reload
    sudo systemctl start k3s
    ```

5.  **Verifica che il nuovo certificato sia stato generato correttamente:**
    Esegui questo comando **sul server k3s** per ispezionare il certificato appena creato.
    ```bash
    openssl s_client -connect IL_TUO_FQDN:6443 -servername IL_TUO_FQDN 2>/dev/null | openssl x509 -noout -text | grep -A1 "Subject Alternative Name"
    ```
    L'output ora dovrebbe includere `DNS:IL_TUO_FQDN` tra i nomi validi.

A questo punto, il tuo nodo agent dovrebbe riuscire a connettersi senza problemi.
