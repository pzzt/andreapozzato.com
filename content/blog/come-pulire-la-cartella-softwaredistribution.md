---
title:  'Come pulire la cartella SoftwareDistribution e ripristinare gli aggiornamenti di Windows'
date: '2024-12-17T16:45:20+01:00'
# weight: 1
# aliases: ["/first"]

tags: ["Windows"]

author: "Andrea"
# author: ["Me", "You"] # multiple authors
showToc: true
TocOpen: true
draft: false 
hidemeta: false
comments: false
# description: "Desc Text."
canonicalURL: "https://www.andreapozzato.com/blog"
disableHLJS: false # to disable highlightjs
disableShare: false
disableHLJS: false
hideSummary: false
searchHidden: false
ShowReadingTime: true
ShowBreadCrumbs: true
ShowPostNavLinks: true
ShowWordCount: true
ShowRssButtonInSectionTermList: true
UseHugoToc: true
#cover:
#    image: "<image path/url>" # image path/url
#    alt: "<alt text>" # alt text
#    caption: "<text>" # display caption under cover
#    relative: false # when using page bundles set this to true
#    hidden: true # only hide on current single page
editPost:
    URL: "https://www.andreapozzato.com/"
    Text: "Suggest Changes" # edit text
    appendFilePath: true # to append file path to Edit link
---
### Stop del servizio Windows Update (wuauserv) 
   Apri il **Prompt dei comandi con privilegi elevati** (clicca con il tasto destro su "Prompt dei comandi" e seleziona "Esegui come amministratore").  

Esegui il comando:
~~~cmd
net stop wuauserv
~~~

### Pulizia della cartella Download di SoftwareDistribution
Vai manualmente alla seguente cartella e svuota il suo contenuto:
~~~cmd
C:\WINDOWS\SoftwareDistribution\Download\
~~~

### Avvio del servizio Windows Update
Una volta pulita la cartella, esegui il comando seguente per riavviare il servizio Windows Update:
~~~cmd
net start wuauserv
~~~

### Verifica
Dopo che il servizio è stato riavviato, alcune cartelle potrebbero essere ricreate automaticamente. 

## Note:
Questi passaggi possono essere utili per risolvere problemi di spazio occupato dagli aggiornamenti di Windows.

