# 1. Installazione degli Strumenti di Base (Python & Git)

L'esecuzione di ComfyUI richiede la presenza nel sistema di versioni aggiornate di Python e Git, installabili tramite il gestore di pacchetti **Homebrew**.

*Verificare il corretto posizionamento con i comandi `python3 --version` e `git --version`.*

# 2. Clonazione della Repository

L'utilizzo della struttura basata su repository Git consente una gestione diretta dei parametri hardware e l'accesso immediato agli aggiornamenti ufficiali.

### 2.1 Clonazione della Repository

Eseguire il download del progetto nella cartella di destinazione desiderata quindi nel proprio spazio di lavoro `Spazio_Lavoro`:

```bash
git clone https://github.com/Comfy-Org/ComfyUI
```
# 3. Configurazione del Virtual Environment (`venv`)
### 3.1 Creazione dell'Ambiente Virtuale (`venv`)

Il ricorso a un ambiente virtuale (`venv`) isola le librerie del progetto, impedendo conflitti di versione con altri script di Deep Learning (es. modelli MLP o CNN).

Eseguire la creazione:

```bash
python3 -m venv my_comfy_venv

```
Questo comando genera una cartella `my_comfy_venv` all'interno della directory corrente, contenente una copia isolata di Python e dei pacchetti installati successivamente.

### 3.2 Attivazione e Disattivazione

L'ambiente isolato deve essere attivato a ogni riavvio del Terminale prima di lanciare ComfyUI:

```bash
source my_comfy_venv/bin/activate

```
Per disattivare l'ambiente virtuale e tornare alla shell di sistema, eseguire:
```bash
deactivate
```

### 3.3 Controllo dello Stato del venv
La presenza della dicitura `(my_comfy_venv)` a inizio riga conferma la corretta attivazione dell'ambiente virtuale. In alternativa, per verificare l'attivazione, è possibile eseguire:

```bash
which python3
```
se il percorso restituito punta alla cartella `my_comfy_venv/bin/python3`, l'ambiente è attivo e pronto per l'installazione dei pacchetti.

### 3.4 Condivisione del venv tra più progetti
In caso di installazioni multiple di ComfyUI sul medesimo sistema, è possibile associare le diverse cartelle al medesimo ambiente virtuale. È sufficiente eseguire il comando `source` puntando al percorso del primo `venv` creato, risparmiando circa 10GB di spazio su disco.

### 3.5 Struttura della Cartella del Progetto
```
📂 Spazio_Lavoro/
├── 📂 ComfyUI/              <-- Solo codice sorgente da GitHub
├── 📂 venv_comfy_standard/  <-- Ambiente 1 (es. per Stable Diffusion)
└── 📂 venv_comfy_video/     <-- Ambiente 2 (es. con librerie specifiche per SVD)
```

### 3.6 Esempio
```bash
% which python3
/opt/homebrew/bin/python3

% pip3 list
Package Version
------- -------
pip 26.1.1
wheel 0.47.0

% source my_comfy_venv/bin/activate

(my_comfy_venv) % which python3
/Users/utente/Spazio_Lavoro/my_comfy_venv/bin/python3

(my_comfy_venv) % pip3 list
Package Version
----------------- --------
filelock 3.29.0
...
torchvision 0.27.0
typing_extensions 4.15.0
```


# 4. Installazione dei Pacchetti e dei `requirements.txt`

### 4.1 Installazione di PyTorch per Apple Silicon

Installare il framework ottimizzato per l'accelerazione hardware MPS (*Metal Performance Shaders*) del chip M5 Pro:

```bash
pip install torch torchvision torchaudio

```

### 4.2 Installazione dei Requisiti di Progetto

Eseguire l'installazione cumulativa delle librerie di supporto catalogate dagli sviluppatori:

```bash
pip install -r ComfyUI/requirements.txt

```

*I pacchetti verranno allocati esclusivamente all'interno del `venv` attivo.*



# 5. Configurazione di ComfyUI-Manager e Download del Modello SVD

### 5.1 Installazione del Gestore delle Estensioni

ComfyUI-Manager centralizza l'installazione e l'aggiornamento dei nodi personalizzati:

```bash
cd ComfyUI/custom_nodes
git clone https://github.com/Comfy-Org/ComfyUI-Manager
cd ../..
```

### 5.2 Download e Posizionamento del Modello SVD

1. Scaricare il file del modello **`svd_xt.safetensors`** (versione XT, ottimizzata per sequenze da 25 frame).
2. Collocare il file nel percorso: `ComfyUI/models/checkpoints/`



# 6. Esecuzione Ottimizzata su Apple Silicon (24GB RAM)
### 6.1 Avvio dell'Interfaccia Grafica 
L'avvio dell'interfaccia grafica deve avvenire tramite l'applicazione di flag specifici per ottimizzare l'uso della memoria unificata ed evitare fenomeni di swap sul disco SSD:

```bash
python ComfyUI/main.py --force-fp16 --highvram
```

* `--force-fp16`: Attiva il calcolo a mezza precisione, dimezzando l'occupazione della VRAM senza riduzioni visibili della qualità.
* `--highvram`: Ottimizza la memorizzazione nella cache dei modelli pesanti all'interno dei 24GB di RAM.

L'interfaccia grafica è accessibile da browser all'indirizzo locale: **`http://127.0.0.1:8188`**.

Si può anche eseguire il server offline:
```bash
python3 ComfyUI/main.py --force-fp16 --highvram --workspace-mode offline
```
- Nessun download automatico: ComfyUI-Manager non può scaricare nuovi modelli o installare i nodi mancanti quando si importa un workflow esterno.
- Blocco aggiornamenti: Impossibile aggiornare il software e le estensioni tramite il pulsante "Update All".
- Crash di nodi dinamici: Alcuni nodi avanzati falliscono l'esecuzione perché non possono scaricare file di supporto necessari al primo avvio.
- Prestazioni hardware: Restano al 100% identiche, poiché il calcolo di SVD è interamente locale.

### 6.2 Chiusura del Server

Per disattivare il server è sufficiente chiudere il Terminale o interrompere il processo con `CTRL + C`.


# 7. Struttura del Workflow Base (Image-to-Video)

Il flusso di lavoro per la generazione video da immagini locali si articola sequenzialmente attraverso i seguenti nodi principali:

### I Nodi Fondamentali

1. **Load Image:** Consente l'upload di file grafici standard (`.jpg`, `.png`, `.webp`) estratti dalla galleria locale.
2. **ImageResize (Ridimensionatore):** Adeguia le dimensioni dell'immagine di input ai requisiti nativi del modello.
* *Rilevanza tecnica:* Stable Video Diffusion richiede risoluzioni standardizzate (tipicamente **1024x576** per il formato 16:9 o **576x1024** per il formato verticale). L'inserimento di immagini non ridimensionate (es. formati nativi in 4:3 ad alta risoluzione) comporta distorsioni geometriche nei fotogrammi o interruzioni del processo per saturazione della memoria (*Out of Memory*).


3. **SVD_img2vid_Conditioning:** Associa l'immagine ai parametri cinematici del modello.
* `width` / `height`: Devono corrispondere ai valori impostati nel nodo di resize (es. 1024x576).
* `motion_bucket_id`: Regola l'intensità del movimento (valori operativi compresi tra 50 e 120).
* `augmentation_level`: Determina la fedeltà all'immagine iniziale (valori consigliati tra 0.01 e 0.05 per evitare alterazioni del soggetto).


4. **KSampler:** Genera i singoli fotogrammi riducendo progressivamente il rumore matematico. Configurazione per Mac: Steps `20`, CFG `2.5`, Sampler `euler`, Scheduler `karras`.
5. **Video Combine:** Compila la sequenza di frame renderizzati esportando il file finale in formato `.mp4` o `.webp`.



# 8. Gestione dei Workflow e Multi-Progetto

La gestione di differenti flussi di lavoro non richiede la duplicazione della cartella di progetto o l'apertura di istanze multiple. Tutta l'organizzazione può essere centralizzata tramite funzioni di importazione ed esportazione.

### 8.1 Esportazione e Importazione in Formato JSON

I dati relativi alla disposizione dei nodi e alla configurazione dei parametri risiedono nella cache del browser e non vengono salvati nella cartella Git.

* **Salvataggio:** Selezionare il comando **Save** nel menu laterale destro per scaricare un file di configurazione in formato `.json` (es. `SVD_Cinematic.json`).
* **Ripristino:** Selezionare il comando **Load** per caricare un file memorizzato, o trascinare direttamente il file `.json` all'interno dell'area di lavoro vuota di ComfyUI.
