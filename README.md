# stremio-selfhosted  
**Stremio Stack con Mammamia, Media Flow Proxy e altro ancora**

Questo repository contiene istruzioni, configurazioni e suggerimenti per il self-hosting Oracle Cloud Free Tier di un'intera istanza privata di Stremio, con plugin **Mammamia**, **media-flow-proxy**, **StreamV** e altri componenti opzionali.

---

## 📢 Disclaimer (con un piccolo rant) 

> Questo progetto è a scopo puramente educativo.  
> L'utilizzo improprio di componenti che accedono a contenuti protetti da copyright potrebbe violare le leggi del tuo paese.  
> **Usa questi strumenti solo per contenuti legalmente ottenuti.**  
> L’autore non si assume responsabilità per eventuali usi illeciti.

> ### 📣 Un pensiero personale:
> Se stai usando Stremio con mille plugin e Real-Debrid, sappilo: non sei un pirata.
> 
>  **News flash: non sei un pirata, sei un leacher da salotto con le crocs ai piedi.**
> 
> Il pirata vero seedava, uploada, si faceva il port forwarding da solo e sniffava i peer con Wireshark. Tu clicchi e guardi. Comodo, eh? Ma zero gloria.
> Stai "guardando gratis" sì, ma stai succhiando banda da server altrui senza restituire niente.

> **Non dai nulla, non condividi nulla.**  

> **Zero upload, zero sharing, zero rispetto per chi ci mette storage, tempo e skill.**
> La tua banda in upload è più vuota della cartella “Download” su eMule nel 2025.
> 
> 💰 E poi ci sono quelli che bypassano la pubblicità sui siti di streaming…
> Ma lo sai che quei 2 banner schifosi sono l’unica cosa che tiene in piedi quei siti?
> Se li togli pure quelli, poi piangi perché non trovi più il film russo del 2003 sottotitolato in polacco.

> 💀 Se sei uno che si è mai lamentato per la qualità di uno stream pirata...ti meriti il buffering perpetuo.

> Se proprio vuoi vivere ai margini del sistema, almeno fallo con un po’ di dignità.
> Usa i torrent. Condividi. Seeda. Rompiti la testa sui port forwarding.
> E soprattutto: non fare il figo con gli script di qualcun altro.

> “Steal with style. Share like it’s 2006. Respect the swarm.”

---

## 🎁 Vantaggi del Free Tier di Oracle Cloud

Oracle Cloud offre un piano gratuito **senza scadenza** con una serie di servizi utilizzabili a costo zero. I principali vantaggi includono:

- ✅ **fino a 2 istanze Ampere A1 (ARM)** fino a 4 OCPU e 24 GB RAM **totali**
- ✅ **2 istanza AMD (x86)** VM.Standard.E2.1.Micro
- ✅ **2 volumi di storage block** da 200 GB ciascuno
- ✅ **10 GB di Object Storage**
- ✅ **Rete virtuale (VCN) gratuita** con indirizzo IP pubblico
- ✅ **Accesso a strumenti avanzati** come Load Balancer, Monitoring, CLI, SDK, e Terraform
- ✅ **Zero costi permanenti**, finché resti nel Free Tier
- ✅ **Prestazioni elevate**, paragonabili a servizi cloud a pagamento

⚠️ **Importante**: nessun addebito viene effettuato a meno che non si passi manualmente al piano **"Pay As You Go"**.

---

## ✅ Requisiti  

Hai deciso di configurare una tua istanza privata di Mammamia e Media Flow Proxy, senza spendere un centesimo? Ecco cosa ti serve:

### 🔐 Account Oracle Cloud
- Vai su [Oracle Cloud Free Tier](https://www.oracle.com/cloud/free/)
- Completa la registrazione inserendo i tuoi dati personali
- Inserisci una carta di credito per la verifica (**non verranno effettuati addebiti** se resti nel Free Tier)

### 🖥️ Shape istanze e sistema operativo
- Puoi creare **una istanza AMD** (architettura x86), che per questo progetto è adeguata.
- In alternativa puoi creare fino a **2 istanze ARM (Ampere A1)** con massimo **4 OCPU e 24 GB RAM totali**, ottime per progetti più intensivi o specializzati.
  > ℹ️ **Nota**: Le istanze ARM sono spesso **non disponibili** nella regione `Italy-Milan`, quindi si consiglia di usare l'istanza AMD per iniziare e riservare l'uso delle istanze ARM a progetti specifici o regioni con disponibilità (es. `Germany-Frankfurt`, `US-Ashburn`).
- Sistema Operativo Ubuntu Server Minimal 22.04.  
  > *(Le istruzioni sono per Ubuntu, ma facilmente adattabili ad altre distro.)*

### 🌍 Accesso remoto con IP dinamico + Modifica regole Firewall su Oracle Cloud Platform
> ℹ️ In realtà Oracle Cloud Platform nel piano Free Tier permette di avere fino a 2 indirizzi ip pubblici riservati da assegnare alle istanze dopo la creazione. Ovviamente questi IP essendo riservati e assegnati staticamente alle vostre istanze non cambieranno mai nel corso del tempo. Il mio consiglio è di utilizzare questi ip per progetti più importanti... tanto la soluzione c'è

Poiché poichè l'ip assegnato alle istanze create su OCP è dinamico, è necessario un sistema per mantenere accessibile il tuo server anche quando l’IP cambia.
- Un **IP pubblico** (va bene anche se dinamico).
- Un account gratuito su [**No-IP.com**](https://www.noip.com/) per creare **hostname statici** che puntano sempre al tuo NAS.
- Normalmente l'accesso alle istanze OCP è permesso solo sulla porta 22 (SSH) è necessario aprire l'accesso anche le porte:
  - Porta **80** (HTTP) 
  - Porta **443** (HTTPS)
  - Porta **8080** (Admin Panel di Nginx Proxy Manager)

> 🔁 Questo setup è fondamentale per permettere a Nginx Proxy Manager di ottenere e rinnovare automaticamente i certificati SSL tramite Let’s Encrypt.


### 🔐 Creazione degli hostname su No-IP

Per accedere alle tue applicazioni da remoto, devi creare 3 hostname pubblici gratuiti su [No-IP.com](https://www.noip.com/).

> ⚠️ Gli hostname devono essere univoci. Il mio consiglio è quello di aggiungere un identificativo personale (es. il tuo nome o una sigla) per evitare conflitti.

#### Esempi di hostname personalizzati:
- `mammamia-mario.ddns.net`
- `mfp-mario.ddns.net`
- `streamv-mario.ddns.net`
Puoi ovviamente scegliere qualsiasi nome, purché sia disponibile e facile da ricordare.

Questi hostname punteranno sempre al tuo NAS anche se il tuo IP cambia.  
Il tutto è possibile installando un piccolo agente (Dynamic DNS client) che aggiorna automaticamente il record DNS.

> ℹ️ In realtà Oracle Cloud Platform nel piano Free Tier permette di avere fino a 2 indirizzi ip pubblici riservati da assegnare alle istanze dopo la creazione. Ovviamente questi IP essendo riservati e assegnati staticamente alle vostre istanze non cambieranno mai nel corso del tempo. Il mio consiglio è di utilizzare questi ip per progetti più importanti... tanto la soluzione c'è

### 🍴 Consigliato: fai un fork del repository

> ✨ E' consigliabile creare un **fork personale** di questo repository su GitHub, in modo da poterlo modificare facilmente secondo le tue esigenze.
> Per farlo ti servirà anche un account su GitHub

Per fare ciò:
1. Vai sulla pagina del repository originale
2. Clicca su **"Fork"** (in alto a destra)
3. Clona il tuo fork sul NAS:

```bash
git clone https://github.com/<il-tuo-utente>/<nome-repo>.git
cd <nome-repo>
```
---

## 🔧 Componenti del progetto

| Servizio           | Nome Servizio Docker | Porta interna | Descrizione                              |
|--------------------|----------------------|---------------|------------------------------------------|
| **[Mammamia](https://github.com/UrloMythus/MammaMia)**|mammamia       | 8080(*)          | Plugin personalizzato per Stremio        |
| **[Media Flow Proxy (MFP)](https://github.com/mhdzumair/mediaflow-proxy)**|mediaflow_proxy | 8888(*)   | Proxy per streaming video                |
| **[StreamV](https://github.com/qwertyuiop8899/StreamV)**|steamv        | 7860(*)          | Web player personalizzato (opzionale)    |
| **[Nginx Proxy Manager](https://github.com/NginxProxyManager/nginx-proxy-manager)**|npm | 80/443/8080 | Reverse proxy + certificati Let's Encrypt |
| **[No-IP DUC (Docker)](https://github.com/noipcom/linux-update-client-docker/pkgs/container/noip-duc)** |noip-updater |—         | Aggiorna il DNS dinamicamente            |

>ℹ️ (*)Le **porte elencate (tranne quelle di Nginx Proxy Manager)** sono **interne alla rete Docker** e **non sono esposte direttamente** sulla macchina host.
Questo significa che i servizi **non sono accessibili dall’esterno se non tramite Nginx Proxy Manager**, che funge da gateway sicuro con supporto a **HTTPS e Let's Encrypt**.

---

## 📝 Passaggi per creare un'istanza

### 1. 🔐 Registrazione

- Vai su [Oracle Cloud Free Tier](https://www.oracle.com/cloud/free/)
- Completa la registrazione inserendo i tuoi dati personali
- Inserisci una carta di credito per la verifica (**non verranno effettuati addebiti** se resti nel Free Tier)
  ![image](https://github.com/user-attachments/assets/38a9c92f-63ec-49c5-b6a0-98781976205b)

### 2. ☁️ Creazione dell'istanza

- Dopo l'accesso, vai su **Navigation Menu** > **Compute** > **Instances**
- Clicca su **Create Instance**
  ![image](https://github.com/user-attachments/assets/ea6646e6-fe61-460d-bf33-0e8828a8c090)

### 3. 📋 Configurazione di base

- Nella sezione **Basic Information**:
  - Inserisci un **nome** per la tua istanza
  - Clicca su **Change Image**
    - Seleziona **Ubuntu Server Minimal 22.04**
    - Scegli l'**architettura** in base al tipo di istanza che desideri (es. AMD64 o ARM)
    - La **Shape** (configurazione hardware) verrà aggiornata automaticamente
  ![image](https://github.com/user-attachments/assets/37ce9740-0fa8-40a9-94d9-695e457b3f13)
  ![image](https://github.com/user-attachments/assets/929359db-aa05-4298-8c97-8ff74003cda4)

### 4. ⚙️ Configurazione della Shape

- **Per istanze AMD (x86)**: lascia la **Shape** predefinita
- **Per istanze ARM (Ampere A1)**:
  - Clicca su **Change Shape**
    - Configura:
      - Numero di OCPU: max **4 totali** per account Free Tier
      - Quantità di RAM: max **24 GB totali** per account Free Tier
    ![image](https://github.com/user-attachments/assets/0d68b154-6a4a-47d3-b119-f01abba63baf)
    ![image](https://github.com/user-attachments/assets/49378b98-312c-436a-a77c-0bff337ebc3d)

### 5. 🔐 Security (opzionale)

- Salta la sezione **Add SSH Keys for Root Compartment**
  - Non modificare nulla
  ![image](https://github.com/user-attachments/assets/802a82e1-a3bb-455e-a5fc-ec59f8bdc425)

### 6. 🌐 Networking

- Espandi la sezione **Advanced Options**
- Inserisci un **hostname** per la tua istanza
- In **Add SSH Keys**:
  - Seleziona **Generate SSH Key Pair**
  - Scarica e **conserva in modo sicuro** la chiave privata (`.pem`) e pubblica
  ![image](https://github.com/user-attachments/assets/b69a7447-2546-4ea3-bfd9-c3f6c68512d2)
  ![image](https://github.com/user-attachments/assets/1ff4df24-0dfd-4708-9a75-5c9d6a573afa)

### 7. 💾 Storage

- Salta la sezione **Boot Volume and Storage**
  - Le impostazioni predefinite sono sufficienti
  ![image](https://github.com/user-attachments/assets/37ce66dd-d3c4-455b-b26a-0196fdc58f36)

### 8. 🔍 Review e Creazione

- Rivedi le impostazioni nella sezione **Review**
- Se tutto è corretto, clicca su **Create**

Dopo pochi minuti l'istanza sarà attiva e pronta all'uso.
Puoi connetterti via SSH utilizzando la chiave privata che hai scaricato

### 🔗 Risorse utili

- [Documentazione ufficiale OCI](https://docs.oracle.com/en-us/iaas/Content/home.htm)
- [Come connettersi via SSH](https://docs.oracle.com/en-us/iaas/Content/Compute/Tasks/accessinginstance.htm)

---

## 🔧 Configurazione hostname statici con No-IP

Gli IP pubblici assegnati alle istanze Oracle Cloud normalmente sono dinamici, No-IP ti permette di associare un hostname che si aggiorna automaticamente ogni volta che l'IP della tua istanza cambia. Ecco come fare:

### 1. Registrazione e login

- Vai su [https://www.noip.com/](https://www.noip.com/)
- Clicca su **Sign Up** e crea un account gratuito.
- Verifica la tua email e accedi con le credenziali create.

### 2. Creazione degli hostname (Dynamic DNS)

- Dopo il login, clicca su **Dashboard** → **DDNS & Remote Access** → **No-Ip Hostnames** → **Create Hostname**.
- Inserisci il nome host, ad esempio:

  - `mammamia-mario e selezionate un dominio come ad esempio .ddns.net`
  
  Scegli un nome unico che ti permetta di riconoscerlo facilmente.

- Nel campo **Record Type** lascia selezionato **DNS Host (A)**.
- Nel campo **IPv4 Address**, vedrai il tuo IP pubblico attuale. Va modificato con l'ip pubblico della nostra istanza su Oracle Cloud
- Premi **Create Hostname**.
<img width="1811" alt="Screenshot 2025-06-28 at 19 15 26" src="https://github.com/user-attachments/assets/437f32d3-7db1-40b4-b864-4fa33a072625" />

### 3. Ripeti per gli altri due hostname

- Crea altri due hostname per:

  - `mfp-mario.ddns.net`
  - `streamv-mario.ddns.net`
<img width="1811" alt="Screenshot 2025-06-28 at 19 20 04" src="https://github.com/user-attachments/assets/d432cfdb-7763-4f48-8f92-51622acd4f16" />

> ⚠️ **Attenzione:** Se vedi una scritta gialla accanto a un hostname su No-IP, significa che **l'aggiornamento automatico dell'indirizzo IP non è attivo**.  
> Andremo successivamente a configurare correttamente il client No-IP (DUC) per mantenerlo aggiornato.

> 📩 **Nota importante:** No-IP, nel piano gratuito, richiede il **rinnovo manuale mensile degli hostname**.  
> Riceverai un'email ogni 30 giorni per confermare **gratuitamente** che desideri mantenere attivo ciascun hostname.  
> Se **non li rinnovi**, gli hostname verranno disattivati e **non saranno più raggiungibili**.

### 4. Creazione del gruppo ddnskey su No-IP
Per aggiornare automaticamente gli hostname tramite client come noip-updater, è consigliato non usare direttamente la tua password dell’account, ma creare un gruppo chiamato ddnskey e generare una password dedicata all’aggiornamento IP.

🧭 Passaggi:

Vai su **Dashboard** → **DDNS & Remote Access** → **DDNS Keys**.
Clicca su "Add Group" (Aggiungi gruppo).
Dai al gruppo il nome: ddnskey_streamio (ad esempio).
Associa al gruppo gli hostname che vuoi aggiornare (es. mammamia-xxx.ddns.net, mfp-xxx.ddns.net, ecc.).
<img width="1667" alt="Screenshot 2025-06-30 at 17 47 52" src="https://github.com/user-attachments/assets/e6e7d3ad-7526-422e-8044-9fff95d4b88c" />

Inserisci una nuova password sicura per questo gruppo (diversa da quella del tuo account principale) cliccando su **Add DDNS Key** e successivamente **Generate DDNS Key**.
Salva.<img width="1666" alt="Screenshot 2025-06-30 at 17 54 11" src="https://github.com/user-attachments/assets/8bee49a0-8ebf-46e3-8d8e-0ba80039131a" /><img width="1666" alt="Screenshot 2025-06-30 at 17 54 11" src="https://github.com/user-attachments/assets/1344ee82-c504-4d2d-8c44-a248cd72e11a" />


📌 Questa password sarà quella da inserire nel .env per il container noip-updater.

---

## 🌐 Configurazione DNS globale (Cloudflare, Quad9, ecc.)

Se il tuo sistema utilizza systemd-resolved (come avviene per default in Ubuntu e derivati), non modificare direttamente il file /etc/resolv.conf, perché viene gestito automaticamente.

1. Apri il file di configurazione:

```bash
sudo nano /etc/systemd/resolved.conf
```

2. Cerca la sezione [Resolve] e modifica come segue:

```text
[Resolve]
DNS=1.1.1.1 1.0.0.1
FallbackDNS=9.9.9.9
DNSStubListener=yes
```

* 1.1.1.1 = Cloudflare
* 9.9.9.9 = Quad9 (opzionale fallback)

3. Riavvia il servizio:

```bash
sudo systemctl restart systemd-resolved
```

4. Verifica che i DNS siano attivi:

```bash
resolvectl status
```

---

## 📦 Docker + Docker Compose

> Questo progetto usa **Docker** per semplificare l’installazione e l’isolamento dei servizi.

### 📥 Installazione Docker

```bash
# 🔁 Rimuovi eventuali versioni precedenti
sudo apt remove docker docker-engine docker.io containerd runc

# 🔄 Aggiorna l’elenco dei pacchetti
sudo apt update

# 📦 Installa i pacchetti richiesti per aggiungere il repository Docker
sudo apt install -y ca-certificates curl gnupg lsb-release

# 🗝️ Aggiungi la chiave GPG ufficiale di Docker
sudo mkdir -p /etc/apt/keyrings
curl -fsSL https://download.docker.com/linux/$(. /etc/os-release; echo "$ID")/gpg | \
  sudo gpg --dearmor -o /etc/apt/keyrings/docker.gpg

# 📥 Aggiungi il repository Docker alle fonti APT
echo \
  "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.gpg] \
  https://download.docker.com/linux/$(. /etc/os-release; echo "$ID") \
  $(lsb_release -cs) stable" | \
  sudo tee /etc/apt/sources.list.d/docker.list > /dev/null

# 🐳 Installa Docker, Docker Compose e altri componenti
sudo apt install -y docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin

# 👤 Aggiungi il tuo utente al gruppo docker
sudo usermod -aG docker $USER

# ⚠️ Per applicare il cambiamento, esegui il logout/login oppure:
newgrp

# ✅ Verifica che Docker funzioni correttamente
docker run hello-world
```

---

## 🚀 Avvio del progetto dal repository GitHub

Il progetto è contenuto in un repository GitHub che include un file `docker-compose.yml` preconfigurato. Alcuni servizi costruiranno automaticamente le immagini Docker a partire da Dockerfile remoti ospitati su GitHub.

### 🔧 Prerequisiti

Assicurati di avere:
- Docker e Docker Compose installati (vedi sezione precedente)
- Git installato (`sudo apt install git` se non lo hai)


### 📥 Clona il repository

```bash
cd ~
git clone https://github.com/tuo-utente/tuo-repo.git
cd tuo-repo
```

### 🌐 Crea la rete Docker esterna proxy
Se non l'hai già fatto, crea la rete che verrà utilizzata da Nginx Proxy Manager e dagli altri container per comunicare tra loro:

```bash
docker network create proxy
```
>🔁 Questo comando va eseguito una sola volta. Se la rete esiste già, Docker mostrerà un errore che puoi ignorare in sicurezza.

### 🛠️ Creazione dei file .env per MammaMia,Media Flow Proxy,StreamV e NoIp-Duc
In ogni sotto cartella di questo progetto è presente un file .env_example con tutte le chiavi necessarie per il corretto funzionamento dei vari moduli.
Per ogni modulo copiare e rinominare il file .env_example in .env. I vari .env dovranno essere modificati in base alle vostre specifiche configurazioni.

**1. .env per MammaMia**
Per configurare il plugin MammaMia è necessario configurare il relativo file .env. Vi rimando al repo del progetto per i dettagli.

📄 Esempio: ./mammamia/.env
```text
# File .env per il plugin mammamia
TMDB_KEY=xxxxxxxxxxxxxxxx
PROXY=["http://xxxxxxx-rotate:xxxxxxxxx@p.webshare.io:80"]
FORWARDPROXY=http://xxxxxxx-rotate:xxxxxxxx@p.webshare.io:80/
```

**2. .env per Media Flow Proxy**
Per configurare il modulo Media Flow Proxy è necessario configurare il relativo file .env. Vi rimando al repo del progetto per i dettagli.

📄 Esempio: ./mfp/.env
```text
API_PASSWORD=password
TRANSPORT_ROUTES={"all://*.ichigotv.net": {"verify_ssl": false}, "all://ichigotv.net": {"verify_ssl": false}}
```

**3. .env per StreamV**
Per configurare il plugin StreamV è necessario configurare il relativo file .env. Vi rimando al repo del progetto per i dettagli.

📄 Esempio: ./mfp/.env
```text
TMDB_API_KEY="xxxxxxxxxxxxxxxx"
MFP_PSW="xxxxxxxxx"
MFP_URL="https://mfp-mario.ddns.net"
BOTHLINK=true
```

**4. .env per NoIp-Duc**
Per configurare correttamente il client DDNS (come noip-updater), è necessario un file .env contenente le credenziali e gli hostname o gruppi associati al tuo account No-IP.

📄 Esempio: ./noip-updater/.env
```text
# File .env per il client DDNS con DDNS Key
NOIP_USERNAME=DdnsKeyUser
NOIP_PASSWORD=DdnsKeyPass
NOIP_HOSTNAMES=all.ddnskey.com
```

🛑 Attenzione alla sicurezza: imposta i permessi del file .env in modo che sia leggibile solo dal tuo utente, ad esempio:

```bash
chmod 600 ./noip_updater/.env
```

🔁 Ricorda di sostituire:
DdnsKeyUser → con l'indirizzo email del tuo account No-IP.
DdnsKeyPass → con la password associata al gruppo ddnskey.
NOIP_HOSTNAMES → con i tuoi hostname specifici separati da virgole (host1.ddns.net,host2.ddns.net) oppure all.ddnskey.com che vuol dire tutti gli hostname del gruppo.


### 🏗️ Build delle immagini e avvio dei container

Per buildare le immagini (se definite tramite build: con URL GitHub) e avviare tutto in background:

```bash
docker compose up -d --build
```
> 🧱 Il flag --build forza Docker a scaricare i Dockerfile remoti ed eseguire la build, anche se l'immagine esiste già localmente.

### 🔍 Verifica che tutto sia partito correttamente

```bash
docker compose ps
```

Puoi anche consultare i log con:

```bash
docker compose logs -f
```

🔁 Aggiornare il repository e ricostruire tutto (quando aggiorni da GitHub)
```bash
git pull
docker compose down
docker compose up -d --build
```

---

## 🔐 Configurazione degli hostname e gestione SSL con Nginx Proxy Manager (NPM)

Per rendere accessibili le tue applicazioni web da internet in modo sicuro, useremo **Nginx Proxy Manager (NPM)**. Questo tool semplifica la gestione dei proxy inversi e automatizza l’ottenimento dei certificati SSL con Let’s Encrypt.

### 1. Creazione dei tre hostname su No-IP

Assicurati di aver creato 3 hostname statici su [No-IP.com](https://www.noip.com/) che puntino al tuo IP pubblico (anche se dinamico, aggiornato tramite l’agent No-IP):

- `mammamia-<tuo-id>.ddns.net`
- `mfp-<tuo-id>.ddns.net`
- `streamv-<tuo-id>.ddns.net`

> 🔔 **Suggerimento:** Usa un identificativo unico (`<tuo-id>`) per evitare conflitti con altri utenti No-IP.

### 2. 🔧 Modifica delle regole di firewall per abilitare le porte 80, 443, 8080 su Oracle Cloud Platform

1. Vai su **Navigation Menu** > **Networking** > **Virtual Cloud Networks** ![image](https://github.com/user-attachments/assets/e0aa996c-8577-4dd6-ab4b-03798384e68b)

2. Seleziona la **VCN** associata alla tua istanza
3. Vai al tab **Security** e clicca su **Default Security List for vcn-XXXXXXX**
4. Passa al tab **Security Rules**
5. In **Ingress Rules**, clicca su **Add Ingress Rules**
   - **Source CIDR**: `0.0.0.0/0`
   - **IP Protocol**: `TCP`
   - **Source Port Range**: `All`
   - **Destination Port Range**: `80, 443, 8080`
   ![image](https://github.com/user-attachments/assets/2fcb6879-9337-43d9-b4cc-970dd1828e80)

Salva le modifiche. Ora la tua istanza potrà ricevere connessioni su HTTP/HTTPS.
![image](https://github.com/user-attachments/assets/b021f0d7-af41-4a84-8175-2d432c9cc066)

> Questo consente a Let’s Encrypt di verificare il dominio e rilasciare i certificati.

### 3. Configurazione dei proxy host in Nginx Proxy Manager

Per ogni applicazione, crea un nuovo **Proxy Host** in NPM seguendo questi passi:
- **accedi ad http://<ip-tuo-server>:8080** (al primo accesso le credenziali di default sono **Email: admin@example.com Password: changeme**. Vi verrà chiesto di modificarle)
- **Dalla barra di menu selezionate **Hosts** → **Proxy Hosts** → **Add New Proxy**
- **Domain Names:** inserisci l’hostname corrispondente (es. `mammamia-<tuo-id>.ddns.net`)
- **Scheme:** `http`
- **Forward Hostname / IP:** il mome del servizio cosi come configurato nel docker-compose ovvero mammmia, mediaflow_proxy e streamv
- **Forward Port:** la porta interna dove l’app è in ascolto (es. `8080` per Mammamia, `8888` per mediaflow_proxy e `7860` per streamv)
- Abilita le seguenti opzioni:
  - **Block Common Exploits**
  - **Websockets Support** (se necessario)
  - **Enable HSTS** (opzionale, aumenta la sicurezza)
  ![image](https://github.com/user-attachments/assets/3ad4e778-81be-492f-9db1-3df5b51f1ed9)

- **SSL tab:** seleziona:
  - **Enable SSL**
  - **Force SSL**
  - **HTTP/2 Support**
  - Spunta **Request a new SSL certificate from Let's Encrypt**
  - Accetta i Termini di servizio di Let’s Encrypt
  - Inserisci un indirizzo email valido per la registrazione SSL
  ![image](https://github.com/user-attachments/assets/6f0ef193-45d3-48c8-a6be-7711986f7054)

- **Nel tab Advanced** aggiungete queste configurazioni :

  ```text
  proxy_set_header Host $host;
  proxy_set_header X-Real-IP $remote_addr;
  proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
  proxy_set_header X-Forwarded-Proto $scheme;
  proxy_pass_request_headers on;
  ```
  ![image](https://github.com/user-attachments/assets/92fea31d-bb8c-49c5-a887-f3d5486c9f7f)

Ripeti questa configurazione per ciascuno dei tre hostname con la rispettiva porta (ad esempio, `mfp-<tuo-id>.ddns.net` → porta `8001`, ecc.).

### 4. Verifica e manutenzione

- Dopo aver configurato i proxy host, prova ad accedere agli URL pubblici via browser.
- NPM gestirà automaticamente il rinnovo dei certificati SSL.
- Assicurati che il tuo router sia sempre configurato correttamente per il port forwarding, specialmente dopo eventuali riavvii o aggiornamenti firmware.

---


