---
title:  'Neovim - Ordinare righe con :sort'
date: '2026-07-02T14:26:50+02:00'
slug: "neovim-sort"
id: 2607021426
# weight: 1
# aliases: ["/first"]

tags: ["neovim","zettelkasten","text processing"]

author: "Andrea Pozzato"
# author: ["Me", "You"] # multiple authors
showToc: true 
TocOpen: false
draft: false 
hidemeta: false
comments: false
# description: "Desc Text."
canonicalURL: "https://www.andreapozzato.com/blog/neovim-sort"
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
    image: "img/260702-089.png" # image path/url
    alt: "Nella fiction la fotografia non deve essere più bella della pubblicità" # alt text
    caption: "" # display caption under cover
    relative: false # when using page bundles set this to true
    hidden: false # only hide on current single page
editPost:
    URL: "https://github.com/pzzt/andreapozzato.com/blob/main/content/"
    Text: "Suggest Changes" # edit text
    appendFilePath: true # to append file path to Edit link
---
Il comando `:sort` ordina le righe selezionate in ordine alfabetico crescente (A-Z).

## Varianti

- `:sort` → ordine alfabetico A-Z  
- `:sort i` → ignorando maiuscole/minuscole  
- `:sort!` → ordine inverso (Z-A)  
- `:sort u` → ordina ed elimina duplicati  

## Contesto d’uso

Utile quando si lavora su:

- liste di testo
- file di configurazione
- pulizia di dati testuali
- deduplicazione rapida

## Note
