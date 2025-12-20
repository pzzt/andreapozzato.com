---
title:  'Exchange Online: Come mettere un dominio in whitelist o in blacklist'
date: '2024-12-16T23:45:20+01:00'
# weight: 1
# aliases: ["/first"]

tags: ["ExchangeOnline", "Microsoft365", "Spam"]

author: "Andrea"
# author: ["Me", "You"] # multiple authors
showToc: false
TocOpen: true
draft: false
hidemeta: false
comments: false
description: "Con questi semplici passaggi, puoi facilmente gestire i domini e i mittenti consentiti o bloccati su Exchange Online."
canonicalURL: "https://www.andreapozzato.com/blog/exchange-online-whitelist-blacklist-dominexchange-online-whitelist-blacklist-domini"
disableHLJS: false # to disable highlightjs
disableShare: false
# disableHLJS: false
hideSummary: false
searchHidden: false
ShowReadingTime: true
ShowBreadCrumbs: true
ShowPostNavLinks: true
ShowWordCount: true
ShowRssButtonInSectionTermList: true
UseHugoToc: true
cover:
    image: "img/aig003-c4.png" # image path/url
    alt: "Abstract image" # alt text
#    caption: "<text>" # display caption under cover
#    relative: false # when using page bundles set this to true
#    hidden: false # only hide on current single page
editPost:
    URL: "https://www.andreapozzato.com/"
    Text: "Suggest Changes" # edit text
    appendFilePath: true # to append file path to Edit link
---
## Passaggi

1. **Accedi con un account amministratore**

   - Vai su [admin.microsoft.com](https://admin.microsoft.com/).
   - Oppure accedi direttamente al portale di sicurezza: [security.microsoft.com](https://security.microsoft.com/antispam).

2. **Apri la policy anti-spam**

   - Seleziona **"Anti-spam inbound policy (Default)"**.

3. **Modifica mittenti e domini consentiti/bloccati**

   - Scorri fino a trovare la sezione **"Allowed and blocked senders and domains"**.
   - Clicca su **"Edit allowed and blocked senders and domains"**.

4. **Configura la whitelist o blacklist**

   - Da questo pannello puoi:
     - **Consentire (whitelist)**: Permettere la ricezione della posta da un dominio o singolo indirizzo email.
     - **Bloccare (blacklist)**: Impedire la ricezione della posta da un dominio o singolo indirizzo email.

## Note

- I domini e gli indirizzi configurati nella whitelist o blacklist saranno applicati a livello globale per tutti gli utenti dell'organizzazione.
- Assicurati di salvare le modifiche prima di uscire dal pannello.
