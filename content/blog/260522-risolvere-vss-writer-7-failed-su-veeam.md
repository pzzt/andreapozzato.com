---
title:  'Risolvere Vss Writer 7 Failed Su Veeam'
date: '2026-05-22T21:39:46+02:00'
slug: "risolvere-vss-writer-7-failed-su-veeam"
# weight: 1
# aliases: ["/first"]

tags: ["Veeam", "VSS", "Troubleshooting"]

author: "Andrea Pozzato"
# author: ["Me", "You"] # multiple authors
showToc: true
TocOpen: true
draft: false 
hidemeta: false
comments: false
# description: "Desc Text."
canonicalURL: "https://www.andreapozzato.com/blog/risolvere-vss-writer-7-failed-su-veeam"
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
    image: "img/009-c2.png" # image path/url
    alt: "abstract" # alt text
    caption: "<text>" # display caption under cover
    relative: false # when using page bundles set this to true
    hidden: true # only hide on current single page
editPost:
    URL: "https://github.com/pzzt/andreapozzato.com/blob/main/content/blog/"
    Text: "Suggest Changes" # edit text
    appendFilePath: true # to append file path to Edit link
---
## Veeam non riesce a effettuare la Shadow Copy

Quando un job di backup Veeam fallisce segnalando un errore nella creazione della *Shadow Copy* della macchina, la causa più comune è un malfunzionamento del Servizio Copia Shadow del Volume (VSS) di Windows.
In particolare, se dal log di Veeam o verificando manualmente si riscontra un **VSS Writer State: [7]** (ovvero *Failed*), significa che uno o più writer di Windows sono bloccati in stato di errore e non permettono lo snapshot coerente della macchina.

## Procedura di risoluzione

Segui questi passaggi sulla macchina fisica o virtuale che sta fallendo il backup:

### 1. Verificare lo stato dei VSS Writers

Collegati in RDP alla macchina che non viene backuppata, apri un **Prompt dei comandi (CMD) come Amministratore** e digita:

```powershell
vssadmin list writers
```

Scorri l'output e identifica il writer che presenta lo stato **State: [7] Failed**. Prendi nota del nome del Writer (es. *ASR Writer*, *Microsoft Exchange Writer*, ecc.).

### 2. Identificare e riavviare il servizio associato

Ogni VSS Writer è gestito da un servizio di Windows. Per sbloccare il Writer in errore, è necessario riavviare il servizio ad esso collegato.

> **⚠️ ATTENZIONE: Writer di Active Directory**
> Se il Writer in errore è l'**Active Directory Domain Services Writer** (o *NTDS Writer*), **NON è possibile riavviare il servizio**. L'unico modo per resettare questo writer è **riavviare l'intero server**. Assicurati di pianificare il reboot se si tratta di un Domain Controller in produzione.

Se il Writer *non* è quello di Active Directory, procedi in questo modo:

1. Cerca su Google il nome del Writer in errore per scoprire qual è il nome esatto del servizio Windows associato (es. per l'*Exchange Writer* il servizio è *Microsoft Exchange Information Store*). Se il riavvio del servizio è sicuro, procedi al passo successivo.
2. Premi la combinazione di tasti **Win + R**, digita `services.msc` e premi Invio.
3. Individua il servizio nella lista, fai clic destro e seleziona **Riavvia** (o *Stop* e poi *Start* se il Riavvia è disabilitato).

### 3. Verifica e riprova il backup

Torna al Prompt dei comandi (sempre come amministratore) e lancia nuovamente il comando di controllo:

```powershell
vssadmin list writers
```

Verifica che lo stato del Writer precedentemente in errore sia tornato su **[1] Stable**.

Infine, ritorna sulla console di Veeam, lancia nuovamente il job di backup per la macchina in questione e verifica che lo snapshot venga completato con successo.

---

### Suggerimento extra (Opzionale)

Se hai molti writer in stato [7] e non vuoi riavviare l'intero server (e non si tratta del Writer di AD), puoi provare a riavviare il servizio **Volume Shadow Copy** (servizio *VSS* da `services.msc`). Questo spesso resetta tutti i writer allo stato stabile, ma attenzione: potrebbe causare un breve disservizio per le applicazioni che ne fanno uso in quel momento.

## Note
