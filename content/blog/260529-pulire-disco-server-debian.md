---
title:  'Come Pulire il Disco di Sistema su un Server Debian'
date: '2026-05-29T14:56:52+02:00'
slug: "pulire-disco-server-debian"
# weight: 1
# aliases: ["/first"]

tags: ["Linux", "Debian", "Manutenzione"]

author: "Andrea Pozzato"
# author: ["Me", "You"] # multiple authors
showToc: true
TocOpen: true
draft: false
hidemeta: false
comments: false
# description: "Desc Text."
canonicalURL: "https://www.andreapozzato.com/blog/pulire-disco-server-debian"
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
    image: "img/018-c2.png" # image path/url
    alt: "Space the final frontier" # alt text
    caption: "<text>" # display caption under cover
    relative: false # when using page bundles set this to true
    hidden: false # only hide on current single page
editPost:
    URL: "https://github.com/pzzt/andreapozzato.com/blob/main/content/"
    Text: "Suggest Changes" # edit text
    appendFilePath: true # to append file path to Edit link
---
La gestione dello spazio su disco è un aspetto fondamentale dell'amministrazione di un server. L'esaurimento dello spazio disponibile può causare malfunzionamenti dei servizi, interruzioni improvvise e la comparsa del temuto errore "No space left on device".

Questa guida illustra le procedure essenziali per liberare spazio su un server Debian in modo sicuro, intervenendo su cache, log di sistema, file residui e vecchi kernel.

---

## 1. Controllare lo spazio disponibile sul disco

Prima di procedere con la pulizia, è necessario analizzare lo stato di occupazione del disco. Il comando `df` fornisce una panoramica rapida. L'opzione `-h` (human-readable) rende le dimensioni leggibili (MB, GB), mentre `-T` mostra il tipo di filesystem.

```bash
df -hT
```

Per capire quali directory consumano più spazio a livello di primo livello, è possibile ricorrere a `du`:

```bash
du -h --max-depth=1 / | sort -hr
```

---

## 2. Pulizia del gestore pacchetti APT

Il gestore di pacchetti APT accumula nel tempo una quantità significativa di file obsoleti, come dipendenze non più necessarie e pacchetti scaricati nella cache locale. L'esecuzione periodica di questi tre comandi permette di recuperare spazio in modo sicuro:

```bash
sudo apt update
sudo apt autoremove -y
sudo apt clean
```

Il funzionamento dei singoli comandi è il seguente:

* **`apt update`**: Aggiorna l'indice dei pacchetti, operazione preliminare sempre consigliata.
* **`apt autoremove -y`**: Rimuove automaticamente le dipendenze installate che non sono più richieste da alcun pacchetto presente nel sistema.
* **`apt clean`**: Svuota completamente la cache locale rimuovendo i file `.deb` scaricati in precedenza. A differenza di `autoclean`, elimina tutti i file della cache, massimizzando lo spazio recuperato.

---

## 3. Pulire i log di Systemd (journald)

Systemd salva i log di sistema nella directory `/var/log/journal/`. Su server con un elevato carico di lavoro o processi che generano molti eventi, questi log possono crescere fino a occupare diversi gigabyte.

Per verificare lo spazio attualmente occupato dai log:

```bash
journalctl --disk-usage
```

Se lo spazio occupato è eccessivo, è possibile applicare criteri di rimozione basati sul tempo o sulla dimensione massima consentita:

```bash
# Mantieni i log solo degli ultimi 2 giorni
sudo journalctl --vacuum-time=2d

# Limita lo spazio totale occupato dai log a 500MB
sudo journalctl --vacuum-size=500M
```

Per rendere questa configurazione permanente ed evitare interventi manuali futuri, è possibile modificare il file `/etc/systemd/journald.conf` impostando il parametro `SystemMaxUse=500M`.

---

## 4. Analisi dello spazio con ncdu

Eliminare file senza una mappa chiara dell'utilizzo del disco può essere rischioso. `ncdu` (NCurses Disk Usage) è uno strumento da terminale che permette di analizzare l'occupazione delle directory in modo interattivo e rapido. Per un approfondimento sull'utilizzo di questo strumento, è disponibile una [guida dedicata a ncdu](https://www.andreapozzato.com/blog/gestione-spazio-linux-ncdu/) sul blog.

Installazione ed esecuzione:

```bash
sudo apt install ncdu -y
sudo ncdu -x /
```

Il flag `-x` è fondamentale sui server: impedisce a `ncdu` di attraversare filesystem diversi da quello di partenza, evitando scansioni inutili e lunghe su partizioni virtuali come `/dev` o `/proc`.

---

## 5. Rimozione dei vecchi Kernel

Con il susseguirsi degli aggiornamenti, Debian accumula le vecchie versioni del kernel. A meno che non sia necessario mantenere versioni precedenti per compatibilità o rollback, è consigliabile rimuoverle per liberare spazio prezioso sulla partizione `/boot`.

Prima di procedere, è indispensabile verificare quale kernel è attualmente in uso per evitare di rimuoverlo:

```bash
uname -r
```

Successivamente, si può ottenere la lista di tutti i kernel installati sul sistema:

```bash
dpkg -l | grep linux-image
```

Se l'output mostra versioni più vecchie rispetto a quella in uso, è possibile rimuoverle in modo sicuro tramite APT:

```bash
sudo apt autoremove --purge
```

> **Infobox: Capire l'output di dpkg**
> Nell'output del comando `dpkg -l`, le prime due lettere di ogni riga indicano lo stato del pacchetto:
>
> * **`ii`**: Il pacchetto è installato e funzionante (Installed and Installable).
> * **`rc`**: Il pacchetto è stato rimosso, ma i file di configurazione sono ancora presenti nel sistema (Removed but Config-files).

---

## 6. Rimozione dei file di configurazione residui (tag rc)

I pacchetti contrassegnati con "rc" non vengono rimossi completamente dal comando standard `apt remove`, poiché i file di configurazione vengono preservati. Per mantenere il sistema pulito ed eliminare tutti i file di configurazione residui presenti nel sistema, è possibile eseguire il seguente comando:

```bash
sudo apt purge $(dpkg -l | grep '^rc' | awk '{print $2}')
```

Il comando cerca tutti i pacchetti il cui stato inizia con "rc", estrae i loro nomi tramite `awk` e li passa a `apt purge` per la rimozione definitiva.

---

La manutenzione periodica dello spazio su disco è essenziale per garantire la stabilità e l'affidabilità di un server Debian. L'esecuzione regolare delle operazioni descritte previene il verificarsi di errori critici e ottimizza le risorse del sistema.
