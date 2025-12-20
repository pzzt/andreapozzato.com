---
title:  'Guida Cert-Manager: Certificati SSL Automatici con Cloudflare su K3S'
date: '2025-12-19T13:03:36+01:00'
slug: "guida-cert-manager-certificati-ssl-automatici-cloudflare-k3s"
# weight: 1
# aliases: ["/first"]

tags: ["kubernetes", "cloudflare", "cert-manager"]

author: "Andrea Pozzato"
# author: ["Me", "You"] # multiple authors
showToc: true
TocOpen: true
draft: true
hidemeta: false
comments: false
# description: "Desc Text."
canonicalURL: "https://www.andreapozzato.com/blog/guida-cert-manager-certificati-ssl-automatici-cloudflare-k3s"
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
    image: "img/013-c2.png" # image path/url
    alt: "abstract" # alt text
    caption: "<text>" # display caption under cover
    relative: false # when using page bundles set this to true
    hidden: false # only hide on current single page
editPost:
    URL: "https://github.com/pzzt/"
    Text: "Suggest Changes" # edit text
    appendFilePath: true # to append file path to Edit link
---
Questa guida ti mostrerà come configurare `cert-manager` per ottenere e rinnovare automaticamente certificati SSL gratuiti da Let's Encrypt per i tuoi servizi Kubernetes, utilizzando una sfida `DNS-01` tramite Cloudflare. Questo metodo è ideale se i tuoi servizi non sono esposti pubblicamente su Internet (ad esempio, se si trovano dietro un VPN come Tailscale).

### Prerequisiti

* Un cluster k3s funzionante.
* Un dominio gestito su Cloudflare.
* `cert-manager` installato sul tuo cluster k3s.
* `kubectl` configurato per comunicare con il tuo cluster.
* Un servizio (es. un'applicazione web) in esecuzione nel tuo cluster a cui vuoi assegnare un dominio.

---

### Passo 1: Creare un API Token su Cloudflare

Per permettere a `cert-manager` di modificare i tuoi record DNS, devi creare un API Token con i permessi minimi necessari.

1. Vai al tuo dashboard Cloudflare: `My Profile -> API Tokens`.
2. Clicca su **Create Token**.
3. Scegli il template **Custom token**.
4. Configura i permessi come segue:
    * **Permissions**:
        * `Zone` : `Zone` : `Read`
        * `Zone` : `DNS` : `Edit`
    * **Zone Resources**:
        * `Include` : `Specific zone` : `iltuodominio.com` (sostituisci con il tuo dominio)
5. Clicca su **Continue to summary** e poi su **Create Token**.
6. **Copia il token generato**. Non sarà più visibile.

### Passo 2: Creare il Secret Kubernetes per l'API Token

Ora, salva l'API Token che hai copiato in un Secret di Kubernetes. `cert-manager` lo userà per autenticarsi con l'API di Cloudflare.

Esegui questo comando sulla tua macchina, sostituendo `<IL-TUO-API-TOKEN>` con il token ottenuto al Passo 1.

```bash
kubectl create secret generic cloudflare-api-token-secret \
  --from-literal=api-token=<IL-TUO-API-TOKEN> \
  --namespace cert-manager
```

**Nota:** I `ClusterIssuer` cercano i secret nel namespace `cert-manager` per convenzione.

### Passo 3: Configurare il `ClusterIssuer`

Il `ClusterIssuer` è una risorsa Kubernetes che definisce come `cert-manager` deve ottenere i certificati. Creane uno nuovo (o modifica quello esistente) per usare la sfida `DNS-01`.

Crea un file chiamato `cluster-issuer.yaml` con il seguente contenuto:

```yaml
apiVersion: cert-manager.io/v1
kind: ClusterIssuer
metadata:
  name: letsencrypt-prod
spec:
  acme:
    # La tua email per le notifiche di Let's Encrypt
    email: tua-email@esempio.com
    server: https://acme-v02.api.letsencrypt.org/directory
    privateKeySecretRef:
      name: letsencrypt-prod-account-key
    solvers:
    - dns01:
        cloudflare:
          # La tua email di login a Cloudflare
          email: la-tua-email-cloudflare@esempio.com
          apiTokenSecretRef:
            # Deve corrispondere al nome del secret creato al Passo 2
            name: cloudflare-api-token-secret
            # Deve corrispondere alla chiave usata nel Passo 2
            key: api-token
```

Applica la configurazione:

```bash
kubectl apply -f cluster-issuer.yaml
```

### Passo 4: Configurare l'Ingress per il Tuo Servizio

Ora, modifica la risorsa `Ingress` del tuo servizio per richiedere un certificato TLS e instradare il traffico HTTPS.

Assicurati che il tuo file `ingress.yaml` assomigli a questo, personalizzando i campi `host`, `secretName`, `service.name` e `service.port.number`.

```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: mio-servizio-ingress
  namespace: default
spec:
  tls:
  - hosts:
    # Il dominio completo che vuoi proteggere
    - servizio.iltuodominio.com
    # Il nome del secret dove cert-manager salverà il certificato
    secretName: mio-servizio-tls
  rules:
  - host: "servizio.iltuodominio.com" # Deve corrispondere a tls.hosts[0]
    http:
      paths:
      - path: /
        pathType: Prefix
        backend:
          service:
            # Il nome del tuo servizio Kubernetes
            name: nome-del-tuo-servizio
            port:
              # La porta su cui il tuo servizio è in ascolto
              number: 8080
```

Applica la configurazione dell'Ingress:

```bash
kubectl apply -f ingress.yaml
```

### Passo 5: Verifica

`cert-manager` rileverà le modifiche e inizierà il processo di emissione del certificato. Puoi monitorare lo stato con:

```bash
kubectl get certificate mio-servizio-tls -n default -w
```

Dovresti vedere lo stato passare da `False` a `True` dopo qualche minuto. Una volta che il certificato è `Ready`, potrai accedere al tuo servizio in modo sicuro all'indirizzo `https://servizio.iltuodominio.com`.

---

## Troubleshooting

Se qualcosa non funziona, ecco i problemi più comuni e come risolverli.

### Problema: La risorsa `Challenge` fallisce con l'errore `secrets "..." not found`

**Causa:** Questo errore significa che `cert-manager` non riesce a trovare il Secret che contiene il tuo API Token di Cloudflare. Questo accade se il nome del Secret o la chiave (`key`) specificati nel `ClusterIssuer` non corrispondono a quelli effettivamente creati.

**Soluzione:**

1. Verifica il nome esatto del Secret che hai creato:

    ```bash
    kubectl get secret -n cert-manager
    ```

2. Controlla il contenuto del tuo `ClusterIssuer`:

    ```bash
    kubectl get clusterissuer letsencrypt-prod -o yaml
    ```

3. Assicurati che i valori in `spec.solvers.dns01.cloudflare.apiTokenSecretRef` corrispondano esattamente al nome e alla chiave del Secret. Correggi il `ClusterIssuer` con `kubectl edit clusterissuer letsencrypt-prod` se necessario.

#### Problema: Il certificato viene emesso, ma il browser mostra l'errore `NET::ERR_CERT_AUTHORITY_INVALID`

**Causa:** Questo errore indica che il certificato SSL presentato dal server non è valido per il dominio che stai visitando. La causa più comune è una discrepanza tra il dominio specificato nella risorsa `Certificate` (che deriva dall'`Ingress`) e l'indirizzo che stai digitando nel browser.

**Soluzione:**

1. Controlla i `dnsNames` nel certificato:

    ```bash
    kubectl describe certificate mio-servizio-tls -n default
    ```

2. Verifica che il dominio elencato sia esattamente quello che vuoi usare.
3. Modifica il tuo file `ingress.yaml` e assicurati che i campi `spec.tls.hosts` e `spec.rules.host` contengano **esattamente** lo stesso dominio completo che desideri proteggere (es. `servizio.iltuodominio.com`).
4. Riapplica l'Ingress con `kubectl apply -f ingress.yaml`. `cert-manager` rileverà la modifica e richiederà un nuovo certificato per il dominio corretto.

#### Problema: Il certificato rimane in stato `pending` per molto tempo

**Causa:** Potrebbe esserci un problema a livello di `Order` o `Challenge` che non è immediatamente visibile a livello di `Certificate`.

**Soluzione:**
Ispeziona le risorse `Order` e `Challenge` per trovare l'errore dettagliato nella sezione `Events`.

1. Trova l'ordine associato al certificato:

    ```bash
    kubectl get order -n default
    ```

2. Descrivi l'ordine per vedere gli eventi:

    ```bash
    kubectl describe order <nome-dell-order> -n default
    ```

3. Se necessario, fai lo stesso per la risorsa `Challenge` (il cui nome è indicato nell'output del `describe order`). La sezione `Events` conterrà il messaggio di errore specifico restituito da Let's Encrypt o dal provider DNS.

## Note

[Guida per installare Cert-Manager](https://gist.github.com/pythonhacker/86c40478da075b821fb27fb5ae45ea9d)
