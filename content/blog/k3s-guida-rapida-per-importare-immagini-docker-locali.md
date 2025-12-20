---
title:  'K3S guida rapida per importare immagini docker locali'
date: '2025-12-05T18:57:23+01:00'
slug: "k3s-guida-rapida-per-importare-immagini-docker-locali"
# weight: 1
# aliases: ["/first"]

tags: ["K3s", "docker"]

author: "Andrea Pozzato"
# author: ["Me", "You"] # multiple authors
showToc: true
TocOpen: true
draft: false
hidemeta: false
comments: false
# description: "Desc Text."
canonicalURL: "https://www.andreapozzato.com/blog/k3s-guida-rapida-per-importare-immagini-docker-locali"
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
    image: "img/004-c2.png" # image path/url
    alt: "abstract" # alt text
    caption: "<text>" # display caption under cover
    relative: false # when using page bundles set this to true
    hidden: false # only hide on current single page
editPost:
    URL: "https://github.com/pzzt/"
    Text: "Suggest Changes" # edit text
    appendFilePath: false # to append file path to Edit link
---
Quando sviluppi in locale o lavori in un ambiente offline con K3s, spesso hai bisogno di eseguire il deploy di una tua immagine Docker senza pusharla su un registro. Il processo sembra semplice: salvi l'immagine, la trasferisci sul nodo e la importi.

Tuttavia, se usi il comando sbagliato, ti scontrerai con l'errore `ErrImageNeverPull`. Questa guida mostra il modo corretto e infallibile per importare le tue immagini in K3s.

## Il Problema Comune: Usare `ctr`

L'errore più frequente è tentare di importare l'immagine usando il comando standard di `containerd`, `ctr`.

```bash
# Questo comando NON funziona correttamente con K3s
ctr -n k8s.io image import mia-app.tar
```

Anche se il comando sembra avere successo, K3s non sarà in grado di usare l'immagine. I tuoi pod rimarranno bloccati in stato `ErrImageNeverPull` perché, in pratica, l'immagine è salvata su disco ma il runtime di K3s non riesce a "vederla".

## La Soluzione Corretta: Usare `k3s ctr`

K3s fornisce un comando wrapper specifico che gestisce l'importazione e la registrazione dell'immagine nel modo giusto. Segui questi passaggi per un importazione senza errori.

### 1. Costruire e Salvare l'Immagine (Sul tuo PC)

Prima, crea l'immagine Docker della tua applicazione e salvala in un file `.tar`.

```bash
# Costruisci l'immagine
docker build -t mia-app:latest .

# Salva l'immagine in un archivio tar
docker save -o mia-app.tar mia-app:latest
```

### 2. Trasferire l'Immagine sul Nodo K3s

Usa `scp` o un altro metodo per spostare il file `.tar` sulla tua macchina K3s.

```bash
scp mia-app.tar utente@ip-del-nodo-k3s:/home/utente/
```

### 3. Importare l'Immagine con il Comando K3s

Connettiti in SSH al tuo nodo K3s e usa il comando `k3s ctr`. Questo è il passaggio fondamentale.

```bash
# Sul nodo K3s
k3s ctr images import mia-app.tar
```

### 4. Verificare che l'Immagine Sia Visibile

Assicurati che K3s possa ora vedere l'immagine usando `crictl`.

```bash
k3s crictl images | grep mia-app
```

Dovresti vedere la tua immagine `mia-app:latest` nell'elenco.

### 5. Eseguire il Deploy

Nel file di configurazione del tuo Deployment, ricorda di impostare `imagePullPolicy: Never` per forzare K3s a usare l'immagine locale che hai appena importato.

```yaml
# deployment.yaml
apiVersion: apps/v1
kind: Deployment
# ...
spec:
  # ...
  template:
    # ...
    spec:
      containers:
      - name: mia-app
        image: mia-app:latest
        imagePullPolicy: Never # Politica cruciale per le immagini locali
        ports:
        - containerPort: 8080
```

## Punto Chiave

La regola fondamentale da ricordare è: **usa sempre `k3s ctr images import` invece di `ctr image import`**. Questo comando garantisce che l'immagine sia correttamente registrata e disponibile per il tuo cluster K3s, risparmiandoti ore di debug.

## Note
