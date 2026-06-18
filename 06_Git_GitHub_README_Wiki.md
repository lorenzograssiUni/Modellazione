# 06 - Git, GitHub, README, Wiki

---

## 1. Chiarimenti sul Progetto d'Esame

> **Il gioco NON è il fine, è il contesto.**

| Cosa NON è necessario | Cosa vogliamo davvero |
|---|---|
| Un gioco RPG completo e complesso | Un sistema software ben progettato |
| Decine di funzionalità, mappe, combattimenti avanzati | Il gioco RPG come *scenario applicativo* |
| Grafica sofisticata | Codice modulare, estendibile, manutenibile |
| Perdersi in dettagli "da videogioco commerciale" | Scelte architetturali motivate |

> **Il valore del progetto NON è nel gioco, ma in come lo progettate e lo sviluppate.**
> Il vostro obiettivo è farci capire, tramite questo progetto, che avete capito **TUTTI** i concetti visti a lezione.

- Il multiplayer **non è obbligatorio**
- Un "dungeon exploration" va benissimo come scenario RPG
- Focus su: struttura del codice, uso di Git, documentazione (README + Wiki)

---

## 2. Recap: Remote Repository — Collegare Locale e Remoto

### Creare un repository su GitHub

1. Clicca **"New repository"** dalla dashboard
2. Scrivi il nome del repository
3. Aggiungi una descrizione (opzionale)
4. Scegli **Privato** inizialmente (poi reso pubblico prima della consegna)
5. Clicca "Create repository"

### Collegare il repository locale al remoto

```bash
# 1. Crea e inizializza il repository locale
mkdir NomeRepository
cd NomeRepository
git init

# 2. Aggiungi il remote (soprannome "origin")
git remote add origin https://github.com/<TuoGitHubID>/<NomeRepository>.git

# 3. Verifica che il remote sia configurato correttamente
git remote -v
# Output:
# origin https://github.com/TuoNome/NomeRepo.git (fetch)
# origin https://github.com/TuoNome/NomeRepo.git (push)

# 4. Primo push: invia il branch main al remote e imposta l'upstream
git push -u origin main

# 5. Se hai più branch e vuoi tracciare tutti
git push -u origin --all
```

> `origin` è solo un soprannome locale per il repository remoto. Si potrebbe usare un altro nome, ma `origin` è di gran lunga il più comune.

---

## 3. Commit vs Push vs Pull

> Tre operazioni distinte, spesso confuse tra loro.

| Comando | Dove agisce | Cosa fa |
|---|---|---|
| `git commit` | Repository **locale** | Salva uno snapshot nella storia locale |
| `git push` | Remote (GitHub) | Invia i commit locali al repository remoto |
| `git pull` | Repository **locale** | Scarica le modifiche dal remote e le integra in locale |

### Schema visivo

```
Working Dir  →  git add  →  Staging  →  git commit  →  Local Repo
                                                               |
                                                          git push
                                                               ↓
                                                        Remote (GitHub)
                                                               |
                                                          git pull
                                                               ↓
                                                        Local Repo
```

### git pull in pratica

```bash
# Scarica le modifiche remote e le integra nel branch corrente
git pull

# Output esempio (dopo che qualcuno ha aggiunto un README su GitHub):
# From https://github.com/FabrizioFornari/MDP-MGC-2526
# * branch main -> FETCH_HEAD
# Updating be1cf3f..a7a591a
# Fast-forward
# README.md | 1 +
# 1 file changed, 1 insertion(+)
```

---

## 4. git fetch — Scaricare senza Integrare

`git fetch` scarica le modifiche remote nel repository locale, **ma senza fare il merge**. Permette di vedere cosa è cambiato prima di integrare.

```bash
# Scarica le modifiche remote (senza merge)
git fetch origin main

# Confronta il branch locale con quello remoto
git diff main origin/main

# Integra manualmente dopo aver visto le differenze
git merge origin/main
```

| Comando | Merge automatico? | Uso tipico |
|---|---|---|
| `git pull` | ✅ Sì | Aggiornamento rapido e integrazione immediata |
| `git fetch` + `git diff` + `git merge` | ❌ No (manuale) | Vuoi vedere le modifiche prima di integrarle |

---

## 5. Ignorare File — .gitignore

Spesso ci sono file che non si vuole tracciare con Git (file generati automaticamente, file di build, file di sistema).

```bash
# Crea il file .gitignore nella root del progetto
# e aggiungi i pattern dei file da ignorare
```

### Esempi di pattern `.gitignore`

```
# Ignora tutti i file .a
*.a

# Ma tieni traccia di lib.a (eccezione)
!lib.a

# Ignora solo il file TODO nella directory corrente
/TODO

# Ignora tutti i file .DS_Store (macOS)
*.DS_Store

# Ignora la cartella build
/build/

# Ignora tutti i file .log
*.log
```

> 🔗 Raccolta di `.gitignore` pronti all'uso: https://github.com/github/gitignore

> ⚠️ IntelliJ crea automaticamente un `.gitignore` nel progetto — verificate che sia presente prima di fare il primo push.

---

## 6. Collaborare su GitHub

### Ruoli

| Ruolo | Permessi |
|---|---|
| **Owner (Proprietario)** | Accesso completo; può aggiungere collaboratori |
| **Collaborator** | Può clonare, fare push/pull, aprire issue |

### Flusso per aggiungere un collaboratore

1. Il Proprietario va su **Settings → Collaborators** nel repository GitHub
2. Invita il collaboratore tramite username GitHub
3. Il collaboratore riceve un'email e accetta l'invito
4. Il collaboratore può ora clonare il repository:

```bash
git clone https://github.com/OwnerNome/NomeRepository.git /percorso/locale
```

### Flusso di lavoro collaborativo (da seguire ogni volta)

```bash
# 1. Prima di iniziare a lavorare: aggiorna il repository locale
git pull

# 2. Apporta le modifiche

# 3. Metti le modifiche in staging
git add .

# 4. Commit
git commit -m "Descrizione delle modifiche"

# 5. Carica su GitHub
git push
```

> **Regola d'oro:** Esegui sempre `git pull` **prima** di iniziare a lavorare, soprattutto quando si collabora con altri.

---

## 7. Conflitti di Merge — The Dark Side of Collaboration

Un conflitto si verifica quando due collaboratori modificano lo stesso file e uno dei due cerca di fare push mentre il remote ha già aggiornamenti che non ha ancora scaricato.

### Cosa succede

```
Collaboratore A → modifica File.txt → push ✅
Collaboratore B → modifica File.txt → push ❌ (Git rifiuta!)
```

> Git rifiuta il push perché rileva che il repository remoto contiene nuovi aggiornamenti non ancora incorporati nel branch locale.

### Come risolvere un conflitto

```bash
# 1. Git segnala il conflitto
git status
# Output: "both modified: File.txt"

# 2. Apri il file in conflitto — Git ha aggiunto i marcatori:
# <<<<<<< HEAD
# (tue modifiche locali)
# =======
# (modifiche del collaboratore dal remote)
# >>>>>>> origin/main

# 3. Risolvi manualmente il conflitto (tieni una versione, o unisci)
# Rimuovi i marcatori <<<, ===, >>>

# 4. Aggiungi il file risolto alla staging area
git add File.txt

# 5. Commit del merge
git commit -m "Risolto conflitto su File.txt"

# 6. Push
git push
```

### Buone pratiche per evitare conflitti

- **`git pull` frequente**, soprattutto prima di iniziare nuovo lavoro
- Usa **branch** per separare il lavoro; fai merge su `main` solo quando è completo
- Fai **piccoli commit** frequenti
- Suddividi file grandi in file più piccoli → riduce la probabilità di conflitti

---

## 8. Cosa si Scrive Lavorando a un Progetto Software

| Tipo di testo | Visibilità | Scopo |
|---|---|---|
| **Codice** | Pubblico | Logica applicativa |
| **Commenti al codice** | Pubblico (nel sorgente) | Spiegare scelte implementative |
| **Commenti Git (commit message)** | Pubblico (nel log) | Descrivere le modifiche introdotte |
| **README** | Pubblico (homepage repo) | Presentare il progetto ai visitatori |
| **Wiki** | Pubblico/Privato | Documentazione estesa del progetto |

---

## 9. File README

Il README è spesso il **primo elemento** che un visitatore vede quando visita il repository.

### Dove posizionarlo

GitHub riconosce automaticamente il README se si trova in:
- Directory principale (root) del repository ✅ ← consigliato
- Cartella `doc/`
- Cartella nascosta `.github/`

### Cosa include tipicamente

- **Cosa fa** il progetto
- **Perché** il progetto è utile
- **Come** gli utenti possono iniziare a usarlo (installazione, uso)
- **Dove** ottenere aiuto
- **Chi** mantiene e contribuisce al progetto

### Formato

Il README si scrive in **Markdown** (file `README.md`).

```markdown
# Nome del Progetto

Breve descrizione del progetto.

## Come usare

```bash
# Esempio di avvio
java -jar gioco.jar
```

## Struttura del Progetto

- `src/` — codice sorgente
- `docs/` — documentazione

## Autore

Nome Cognome — Università di Camerino
```

---

## 10. GitHub Wiki

Ogni repository GitHub è dotato di una sezione dedicata all'hosting della documentazione: la **Wiki**.

> La Wiki serve per condividere contenuti più estesi, come le modalità di utilizzo, le scelte di progettazione o i principi fondamentali.

### Visibilità

| Tipo di repository | Visibilità Wiki |
|---|---|
| Repository **pubblico** | Wiki **pubblica** (leggibile da tutti) |
| Repository **privato/interno** | Wiki accessibile solo a chi ha accesso al repo |

### Modifica della Wiki

- Direttamente su GitHub (interfaccia web)
- In locale, clonando la wiki:

```bash
git clone https://github.com/YOUR_USERNAME/YOUR_REPOSITORY.wiki.git
```

> Di default, solo le persone con **accesso in scrittura** al repository possono modificare la Wiki (in un repo pubblico si può abilitare la contribuzione da chiunque).

### Template Wiki per il Progetto d'Esame (MDP-MGC-2526)

La Wiki del progetto deve includere (basato sul template fornito a lezione):

- **Home** — panoramica del progetto
- **Descrizione del dominio** — cos'è il gioco RPG sviluppato
- **Modello del dominio** — diagramma UML delle classi principali
- **Scelte di progettazione** — motivazione delle scelte architetturali
- **Guida all'uso** — come avviare e giocare
- **Dichiarazione uso AI** — se usato, quali tool e per cosa

---

## 11. Materiale Aggiuntivo

- 📖 **Pro Git** (libro completo, gratuito online): https://git-scm.com/book/en/v2
- 🖼️ **Visual Git Cheat Sheet** (interattivo): https://ndpsoftware.com/git-cheatsheet.html

---

## 12. Punti Chiave per l'Esame ✅

- Il progetto d'esame non valuta la qualità del gioco ma la **qualità del software** (modularità, estendibilità, manutenibilità)
- **`git remote add origin <url>`** = collega il repository locale al remote
- **`origin`** = soprannome locale per il repository remoto (convenzionale, non obbligatorio)
- **`git push -u origin main`** = primo push + imposta l'upstream (poi basta `git push`)
- **`git push -u origin --all`** = invia tutti i branch al remote
- **Commit** = aggiorna il repository **locale**; **Push** = aggiorna il repository **remoto**
- **Pull** = scarica dal remote e integra nel locale (= fetch + merge)
- **`git fetch`** = scarica senza fare merge (puoi vedere le diff prima con `git diff main origin/main`)
- **`.gitignore`** = file che elenca pattern di file da non tracciare (generati automaticamente, file di sistema)
- Un **conflitto** nasce quando due persone modificano lo stesso file; si risolve con pull → edit manuale dei marcatori → add → commit → push
- **README.md** = file Markdown nella root del repo; primo elemento visibile dai visitatori
- **Wiki** = sezione GitHub per documentazione estesa; clonabile con `git clone ...wiki.git`
- Per il progetto: repository inizialmente **privato**, reso **pubblico** prima della consegna
- Consegna = link al repository GitHub su Moodle, **10 giorni prima dell'esame**, nessun commit dopo la scadenza
