# ToastFlix Audio Sidecar

[![Deploy to Render](https://render.com/images/deploy-to-render-button.svg)](https://render.com/deploy?repo=https://github.com/qwertyuiop8899/toastflix-sidecar)

[![Deploy on Railway](https://railway.com/button.svg)](https://railway.com/template/deploy?template=https%3A%2F%2Fgithub.com%2Fqwertyuiop8899%2Ftoastflix-sidecar)

Microservizio dedicato alla sincronizzazione e fornitura dell'audio italiano per i flussi **4K / FHD Dual Audio** di ToastFlix.

- **Audio dedicato**: scarica i segmenti audio, li normalizza/converte con `ffmpeg` ed esegue l'allineamento temporale (offset acustico o da database).
- **Leggero e indipendente**: il video continua a passare normalmente dai provider o dal proxy video; il sidecar si occupa esclusivamente della traccia audio.
- **Supporto avanzato**: gestisce delay acustici, correzione framerate/deriva FPS (PAL 25fps / Cinema 23.976fps) e i tagli/ponti audio di ToastFlix Cuts Studio.

---

## 🚀 Modalità di Installazione

Scegli la modalità più adatta al tuo ambiente:

| Modalità | Difficoltà | Ideale per | Requisiti |
| :--- | :--- | :--- | :--- |
| **1. Deploy Cloud 1-Click** | 🟢 Facile | Chi non vuole gestire server o porte | Account Render (Gratuito) |
| **2. Docker con Immagine (GHCR)** | 🟡 Intermedio | VPS, NAS o Server casalingo | Docker |
| **3. Docker da Sorgente (Build)** | 🟡 Intermedio | Modifiche al codice o build custom | Docker & Git |
| **4. Locale Diretto (Python)** | 🟢 Facile | PC Windows, Mac o Linux (stesso di Stremio) | Python 3.10+ & FFmpeg |

---

### 1. ☁️ Deploy Cloud Gratuito (Render)

Il modo più rapido per avere il Sidecar attivo 24/7 con HTTPS automatico senza aprire porte sul router:

1. Clicca sul pulsante **[Deploy to Render](https://render.com/deploy?repo=https://github.com/qwertyuiop8899/toastflix-sidecar)** in alto.
2. Accedi o registrati su Render (gratuito).
3. Assegna un nome al servizio e conferma la creazione.
4. Al termine del deploy, copia l'URL HTTPS assegnato (es. `https://sidecar-xxxx.onrender.com`).
5. Inserisci questo URL come **Server audio DUAL** nella pagina `/configure` di ToastFlix.

---

### 2. 🐳 Docker con Immagine Pre-costruita (GHCR)

L'immagine ufficiale è già compilata e pronta su GitHub Container Registry:  
`ghcr.io/qwertyuiop8899/toastflix-sidecar:latest`

#### Opzione A: Con Docker Compose (Consigliato)
Crea una cartella e salva questo `compose.yml`:

```yaml
services:
  sidecar:
    image: ghcr.io/qwertyuiop8899/toastflix-sidecar:latest
    container_name: toast-audio-sidecar
    restart: unless-stopped
    ports:
      - "3169:3107"
    environment:
      - SIDECAR_PUBLIC_URL=https://audio.tuodominio.com  # opzionale se esposto pubblicamente
      - CORS_ORIGINS=*
    volumes:
      - ./sidecar-data:/app/data
```

Avvia con:
```bash
docker compose up -d
```

#### Opzione B: Con Docker Run (Comando singolo)
```bash
docker run -d \
  --name toast-audio-sidecar \
  --restart unless-stopped \
  -p 3169:3107 \
  -v ./sidecar-data:/app/data \
  ghcr.io/qwertyuiop8899/toastflix-sidecar:latest
```

---

### 3. 🛠️ Docker da Sorgente (Senza immagine pre-costruita)

Se hai clonato il repository e preferisci compilare il container localmente con il `Dockerfile`:

```bash
git clone https://github.com/qwertyuiop8899/toastflix-sidecar.git
cd toastflix-sidecar
```

#### Con Docker Compose:
```bash
docker compose up -d --build
```

#### Con Docker CLI diretto:
```bash
docker build -t toastflix-sidecar .
docker run -d \
  --name toast-audio-sidecar \
  --restart unless-stopped \
  -p 3169:3107 \
  -v ./data:/app/data \
  toastflix-sidecar
```

---

### 4. 💻 Installazione Locale Diretta (Senza Docker)

Ideale se esegui Stremio sullo stesso computer (Windows, macOS o Linux) e non vuoi usare Docker.

#### Requisiti:
- **Python 3.10** o versione successiva.
- **FFmpeg** installato e configurato nel PATH di sistema:
  - **Windows**: Scarica FFmpeg da [gyan.dev](https://www.gyan.dev/ffmpeg/builds/) ed estrai `ffmpeg.exe` aggiungendolo alle variabili d'ambiente (PATH), oppure tramite:
    ```powershell
    winget install Gyan.FFmpeg
    ```
  - **macOS**:
    ```bash
    brew install ffmpeg
    ```
  - **Linux (Ubuntu/Debian)**:
    ```bash
    sudo apt update && sudo apt install -y ffmpeg
    ```

#### Procedura:
1. Clona il repository o scarica i file:
   ```bash
   git clone https://github.com/qwertyuiop8899/toastflix-sidecar.git
   cd toastflix-sidecar
   ```

2. Crea e attiva un virtual environment:
   ```bash
   python3 -m venv venv
   # Linux/macOS:
   source venv/bin/activate
   # Windows (PowerShell):
   .\venv\Scripts\Activate.ps1
   # Windows (CMD):
   .\venv\Scripts\activate.bat
   ```

3. Installa le dipendenze:
   ```bash
   pip install -r requirements.txt
   ```

4. Avvia il server:
   ```bash
   uvicorn app:app --host 0.0.0.0 --port 3000
   ```
   Il sidecar sarà in ascolto su `http://localhost:3000`.

---

## 🔗 Collegamento a ToastFlix

Una volta avviato il Sidecar:

1. Apri la pagina di configurazione di ToastFlix (es. `/configure`).
2. Al **Passo 3 (4K Dual Audio)** seleziona **SÌ, ATTIVA DUAL AUDIO**.
3. Seleziona **Sidecar**.
4. Inserisci l'indirizzo del tuo Sidecar:
   - Se installato in **locale**: `http://localhost:3000` (oppure `http://127.0.0.1:3000`).
   - Se installato su **Render**: l'URL fornito da Render (es. `https://sidecar-xxxx.onrender.com`).
   - Se installato su **VPS con Docker e dominio HTTPS**: `https://audio.tuodominio.com`.
5. Prosegui fino al Passo 7 e installa il manifest generato su Stremio!

---

## 🌐 Configurazione Reverse Proxy HTTPS (Opzionale per VPS)

Se installi il Sidecar su una VPS e desideri usarlo su Smart TV o dispositivi esterni, è raccomandato un certificato HTTPS valido (es. con Caddy o Nginx + Certbot).

### Con Caddy (Consigliato per semplicità)
Aggiungi in `/etc/caddy/Caddyfile`:
```caddyfile
audio.tuodominio.com {
    reverse_proxy 127.0.0.1:3169
}
```
Poi ricarica Caddy:
```bash
sudo systemctl reload caddy
```

### Con Nginx + Certbot
Aggiungi nel blocco del server:
```nginx
server {
    server_name audio.tuodominio.com;
    location / {
        proxy_pass http://127.0.0.1:3169;
        proxy_set_header Host $host;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
    }
}
```
E ottieni il certificato TLS:
```bash
sudo certbot --nginx -d audio.tuodominio.com
```

---

## ⚙️ Variabili di Ambiente (.env)

| Variabile | Default | Descrizione |
| :--- | :--- | :--- |
| `SIDECAR_PORT` | `3169` | Porta host esposta dal container |
| `SIDECAR_PUBLIC_URL` | *(vuoto)* | URL pubblico HTTPS del sidecar (es. `https://audio.tuodominio.com`) |
| `SIDECAR_CACHE_DIR` | `/app/data` | Cartella di memorizzazione temporanea dei segmenti e database offset |
| `SIDECAR_AUDIO_PROXY` | *(vuoto)* | Proxy opzionale SOCKS5/HTTP impiegato **esclusivamente per Fonte 2** (es. `socks5h://172.17.0.1:1080` per WARP su VPS). **Fonte 1 resta sempre diretta** e non viene mai instradata su WARP. |
| `OFFSET_API_URL` | *(vuoto)* | URL API di ToastFlix per sincronizzare gli offset acustici con il database centrale |
| `CORS_ORIGINS` | `*` | Origini consentite per le chiamate CORS |

---

## 🛡️ Configurazione Cloudflare WARP (SOLO per Fonte 2)

Il Sidecar supporta due sorgenti audio italiane per ToastFlix:
1. **Fonte 1**: stream HLS primario ad altissima compatibilità. **Opera SEMPRE in connessione diretta** (non usa mai WARP, poiché i nodi CDN bloccano gli IP proxy dei datacenter).
2. **Fonte 2**: stream AAC secondario in chiaro ad alta fedeltà. Alcune VPS datacenter specifiche (ad esempio **Oracle Cloud**) possono avere il loro indirizzo IP filtrato dal provider di Fonte 2 (`HTTP 403`). In questo caso, WARP risolve completamente il problema.

> **Importante:** Se configuri `SIDECAR_AUDIO_PROXY`, il Sidecar applicherà il proxy **solo ed esclusivamente per i segmenti di Fonte 2**. Fonte 1 e i flussi video continueranno a passare direttamente senza alcun proxy.

### Come verificare se la tua VPS necessita di WARP:
Apri la dashboard web del Sidecar nel browser (es. `http://tuo-ip:3169` oppure `https://sidecar-xxxx.onrender.com`) e premi il pulsante:  
👉 **🔍 Verifica Connettività (Fonte 1 & Fonte 2)**
* **Fonte 1**: deve risultare **✅ Raggiungibile direttamente**.
* **Fonte 2**:
  * Se esce **✅ Raggiungibile**, non devi configurare WARP!
  * Se esce **⚠️ Accesso filtrato o non autorizzato**, segui la procedura WARP sottostante.

### Installazione WARP con Docker (per VPS):
Aggiungi il container `warp` nel tuo `compose.yml`:
```yaml
services:
  warp:
    image: caomingjun/warp
    container_name: warp
    restart: unless-stopped
    ports:
      - "1080:1080"

  sidecar:
    image: ghcr.io/qwertyuiop8899/toastflix-sidecar:latest
    container_name: toast-audio-sidecar
    restart: unless-stopped
    ports:
      - "3169:3107"
    environment:
      - SIDECAR_PUBLIC_URL=https://audio.tuodominio.com
      # Indirizzo del container WARP (applicato SOLO a Fonte 2):
      - SIDECAR_AUDIO_PROXY=socks5h://172.17.0.1:1080
    volumes:
      - ./sidecar-data:/app/data
    depends_on:
      - warp
```

> **Nota su Render:** Su Render non è possibile installare WARP. Render usa direttamente il proprio indirizzo IP (che è compatibile). Se per qualsiasi motivo Fonte 2 non dovesse rispondere, Toastflix esegue automaticamente il fallback trasparente su **Fonte 1**, garantendo che lo streaming non si interrompa mai.

---

## 🎬 Supporto Cuts Studio & Timeline Multi-Cut

Il sidecar supporta nativamente le regole avanzate generate da ToastFlix Cuts Studio:
- **Offset Hardware fMP4 (`video_start_time`)**: sincronizzazione automatica dell'offset dei flussi HLS.
- **Correzione Deriva FPS (0.85x - 1.15x)**: conversione dinamica del framerate audio (es. 25fps PAL vs 23.976fps Cinema).
- **Tagli Multipli (`c=`)**: intervalli temporali da scartare o colmare con discontinuità.
- **Ponte Audio Originale (`b=`)**: reinserimento automatico della traccia originale (inglese) nelle scene inedite o tagliate della versione italiana.

---

## 🏠 Esposizione su Rete Locale (LAN / Self-Host Casalingo)

Se installi il Sidecar sul tuo PC casalingo, Raspberry Pi o NAS locale mentre utilizzi l'istanza online di ToastFlix (es. `https://toastflix.stremio-italia.eu`), il server remoto di ToastFlix **deve poter raggiungere il tuo Sidecar** via Internet per inviare le richieste di sincronizzazione audio (`/sync` e `/dual/aprep`).

Poiché gli indirizzi IP privati di rete locale (es. `192.168.x.x` o `localhost`) non sono instradabili da Internet, è necessario esporre la porta del Sidecar (`3107` o `3169`) con un indirizzo pubblico HTTPS.

Ecco il confronto tra le soluzioni disponibili:

| Metodo | Come funziona | Vantaggi | Svantaggi |
| :--- | :--- | :--- | :--- |
| **Tailscale Funnel** ⭐ *(Consigliato)* | Il PC di casa apre un tunnel in uscita verso Tailscale, che gli assegna un dominio pubblico `https://nome-pc.tailnet.ts.net` | **Gratuito, senza carta di credito, nessun dominio da acquistare, URL fisso e stabile, HTTPS automatico.** | Richiede installare Tailscale sul PC/server locale. |
| **Cloudflare Named Tunnel** | Tunnel tramite `cloudflared` legato a un account Cloudflare Zero Trust e un dominio proprio | Stabile, veloce, dominio personalizzato | Richiede una carta di credito per attivare l'account Cloudflare Zero Trust (anche se a 0€). |
| **Cloudflare Quick Tunnel** (`trycloudflare.com`) | Un comando singolo senza account (`cloudflared tunnel --url ...`) | Immediato, senza account | **A ogni riavvio cambia l'URL**, rompendo la configurazione dell'addon su Stremio. |
| **Port Forwarding + DDNS** | Aprire la porta 3107 sul router di casa e usare un servizio Dynamic DNS (DuckDNS, No-IP, ecc.) | Diretto | **Non funziona sotto CGNAT** (Iliad, Fastweb, connessioni 4G/5G, ecc.) ed espone la porta di casa su Internet. |

### Guida Rapida: Esporre il Sidecar con Tailscale Funnel

1. **Installa Tailscale** sul dispositivo dove gira il Sidecar ([tailscale.com/download](https://tailscale.com/download)).
2. Accedi al tuo account ed esegui il login:
   ```bash
   tailscale up
   ```
3. Avvia Funnel specificando la porta del Sidecar (`3107` o `3169` a seconda della configurazione host):
   ```bash
   tailscale funnel 3107
   ```
4. Tailscale genererà il tuo indirizzo pubblico HTTPS permanente:
   ```text
   https://mio-nodo.tailnet-xyz.ts.net
   ```
5. Imposta l'URL generato come variabile d'ambiente nel tuo `compose.yml`:
   ```yaml
   environment:
     - SIDECAR_PUBLIC_URL=https://mio-nodo.tailnet-xyz.ts.net
   ```
   *In questo modo la Landing Page del Sidecar mostrerà subito il tuo link pubblico e potrai copiarlo con un clic.*
6. Incolla l'URL HTTPS nel configuratore di ToastFlix alla voce **Server audio DUAL (Sidecar)**.
