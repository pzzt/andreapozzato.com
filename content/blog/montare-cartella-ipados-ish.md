---
title:  'Come montare una cartella di iPadOS in iSH'
date: '2024-12-20T18:09:19+01:00'
# weight: 1
# aliases: ["/first"]

tags: ["iPadOS", "iSH", "File.app"]

author: "Andrea"
# author: ["Me", "You"] # multiple authors
showToc: true
TocOpen: true
draft: true
hidemeta: false
comments: false
# description: "Scopri come montare una cartella di File.app in iSH su iPadOS con un semplice comando. Una guida veloce per integrare i tuoi file iOS in iSH."
canonicalURL: "https://www.andreapozzato.com/blog/"
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
cover:
    image: "<image path/url>" # image path/url
    alt: "<alt text>" # alt text
    caption: "<text>" # display caption under cover
    relative: false # when using page bundles set this to true
    hidden: true # only hide on current single page
editPost:
    URL: "https://github.com/<path_to_repo>/content"
    Text: "Suggest Changes" # edit text
    appendFilePath: true # to append file path to Edit link
---
Montare una cartella di File.app dentro a iSH su iPadOS è semplice usando il comando:  
```bash
mount -t ios <src> <dst>
```
### Come trovare il percorso della cartella  
1. Apri File.app e individua la cartella desiderata.  
2. Usa un’app come [FileBrowser Professional](https://apps.apple.com) o un terminale SSH per accedere al percorso esatto.  
3. Sostituisci `<src>` con il percorso assoluto e `<dst>` con la directory di destinazione in iSH.  

Con questa procedura, puoi gestire facilmente i tuoi file su iSH.
## Note:

