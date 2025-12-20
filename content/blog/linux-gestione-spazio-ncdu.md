---
title:  'Gestione dello spazio su disco in Linux: come usare ncdu'
date: '2024-12-30T21:23:49+01:00'
slug: "gestione-spazio-linux-ncdu"
# weight: 1
# aliases: ["/first"]

tags: ["Command-line", "Linux", "Ncdu"]

author: "Andrea"
# author: ["Me", "You"] # multiple authors
showToc: true
TocOpen: true
draft: false
hidemeta: false
comments: false
# description: "Desc Text."
canonicalURL: "https://www.andreapozzato.com/blog/gestione-spazio-linux-ncdu"
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
    image: "img/ncdu-conver-c6.png" # image path/url
    alt: "alacritty con ncdu in esecuzione" # alt text
    caption: "Alacritty e ncdu" # display caption under cover
    relative: false # when using page bundles set this to truelc
    hidden: false # only hide on current single page
editPost:
    URL: "https://github.com/pzzt/"
    Text: "Suggest Changes" # edit text
    appendFilePath: true # to append file path to Edit link
---
Se sulle macchine Linux ti trovi spesso a corto di spazio, **ncdu** può aiutarti a semplificare il lavoro di eliminazione dei file non più necessari. Questo strumento, abbreviazione di *NCurses Disk Usage*, è una versione interattiva del comando **du**, progettata per chi lavora da terminale.

Ncdu utilizza un’interfaccia testuale basata su *ncurses (TUI - Text User Interface)*, che consente di navigare facilmente tra file e cartelle per identificare rapidamente quelli più ingombranti.

Personalmente, lo utilizzo per tenere sotto controllo le cartelle dei log e quelle degli snapshot sui sistemi con file system btrfs. Alle volte il ciclo di snapshot retention potrebbe non funzionare correttamente e queste directory possono crescere rapidamente consumando spazio prezioso. Grazie a ncdu, posso individuare velocemente le cartelle problematiche ed eliminare i dati inutili senza perdere tempo in una ricerca manuale con **du**.

## Come si installa?

Arch:

~~~bash
sudo pacman -S ncdu
~~~

Debian:

~~~bash
sudo apt install ncdu
~~~

Fedora:

~~~
sudo yum install ncdu
~~~

## Come si usa?

Di base si usa richiamandolo e lui farà la scansione della cartella in cui vi trovate

~~~bash
ncdu
~~~

Potete passargli la cartella da scansionare con:

~~~bash
ncdu [flag] [path/to/folder]
~~~

Se non è già attivo di default uso il flag **-e** che attiva la visualizzazione di più informazioni

~~~bash
ncdu -e /path/to/folder
~~~

Si può esportare il json della scansione su stdout:

~~~bash
ncdu -o-
~~~

oppure

~~~bash
ncdu -o file.json /path/to/folder
~~~

Una volta finita la scansione uso spesso i seguenti tasti:

|Tasto|Descrizione|
|---|---|
|g|Prendendolo più volte fa una rotazione tra percentuale, grafico, entrambi o niente.|
|i|Mostra informazioni dettagliate sulla cartella evidenziata.|
|d|Elimina il file selezionato previa conferma.|
|m|Visualizza la data dell'ultima modifica. (Richiede -e)|
|r|Riscansione la cartella|

Non starò qui a ricopiare il manuale del software quindi vi consiglio di darci un'occhiata:

~~~bash
man ncdu
~~~

## Note

Sito dello sviluppatore: [https://dev.yorhel.nl/ncdu](https://dev.yorhel.nl/ncdu?ref=andreapozzato.com)
