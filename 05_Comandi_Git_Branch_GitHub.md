# 05 - Comandi Git, Branch e GitHub

---

## 1. Recap: Flusso Git Base

Le tre sezioni di Git e il flusso fondamentale:

```
Working Directory  →  Staging Area  →  Repository
  (modifiche)      git add        git commit
```

| Sezione | Descrizione |
|---|---|
| **Working Directory** | Dove si modificano i file sul disco |
| **Staging Area** | Area intermedia: prepara il prossimo commit |
| **Repository (.git)** | Storico permanente di tutti i commit |

---

## 2. Comandi Git: Stage e Unstage

```bash
# Aggiunge un file alla staging area
git add nomeFile

# Aggiunge tutti i file modificati/nuovi
git add .

# Rimuove un file dalla staging area (torna a untracked)
git restore --staged nomeFile
```

**Esempio pratico:**
```bash
# Nuovo File 4.txt creato
git add .                          # lo mette in staging
git status                         # mostra: "new file: File4.txt"
git restore --staged File4.txt     # lo toglie dalla staging area
git status                         # mostra: "Untracked files: File4.txt"
```

---

## 3. Modifica e Ripristino di un File

Se si modifica un file per sbaglio e si vuole recuperare la versione precedente:

```bash
# Mostra che il file è stato modificato
git status
# Output:
# Changes not staged for commit:
#   modified: File3.txt

# Ripristina il file allo stato dell'ultimo commit
git checkout File3.txt

# Verifica
git status
# Output: nothing to commit, working tree clean
```

> ⚠️ `git checkout <nomeFile>` **scarta le modifiche non committate** e non possono essere recuperate. Usarlo con attenzione!

---

## 4. Visualizzare la Commit History

```bash
# Lista cronologica inversa dei commit
git log

# Varianti utili
git log -p -2                                    # mostra diff degli ultimi 2 commit
git log --stat                                   # statistiche file per commit
git log --pretty=oneline                         # una riga per commit
git log --pretty=format:"%h -%an, %ar: %s"       # formato personalizzato
git log --all                                    # mostra tutti i branch (anche nascosti)
git log --oneline --graph --all --decorate       # visualizzazione grafica ad albero
```

**Esempio output `git log`:**
```
commit 3f8f4b33... (HEAD -> main)
Author: Fabrizio Fornari <fabrizio.fornari@unicam.it>
Date:   Sun Mar 1 11:23:07 2026 +0100
adding third file

commit 5ecdcd74...
...
```

---

## 5. Tornare a un Commit Precedente (git checkout CommitID)

```bash
# Ottieni l'hash dal log
git log

# Torna a uno stato precedente (bastano le prime 4-7 cifre dell'hash)
git checkout 5ecdcd7

# Visualizza la situazione con tutti i branch
git log --all
```

**Cosa succede:**
- I file tornano allo stato di quel commit
- Git entra in stato **"detached HEAD"**
- `main` punta ancora all'ultimo commit; `HEAD` punta al commit scelto

```
main → Commit A → Commit B → Commit C
                       ↑
                     HEAD (detached)
```

### ⚠️ Detached HEAD — Attenzione!

| Caratteristica | Descrizione |
|---|---|
| **Nessun puntatore mobile** | Normalmente un commit sposta il puntatore del branch in avanti. In detached HEAD il puntatore è fisso: nessun branch segue i nuovi commit |
| **Rischio Garbage Collection** | Se ci si sposta su un altro branch senza aver creato un branch per i commit fatti in detached HEAD, quei commit diventano **orfani** e vengono eliminati da `git gc` (dopo ~30 giorni) |

> **Soluzione:** creare subito un nuovo branch con `git switch -c nuovo-branch` per ancorare i commit.

---

## 6. Branch

Il **branching** è una delle principali caratteristiche di Git.

> Un branch è un ramo indipendente di sviluppo. Ogni branch ha la propria storia di commit.

### Comandi branch

```bash
# Crea un nuovo branch e ci si sposta
git switch -c nuovo-branch

# Solo crea (senza spostarsi)
git branch nuovo-branch

# Spostarsi su un branch esistente
git switch main
# oppure (forma vecchia)
git checkout main

# Lista tutti i branch
git branch

# Lista branch + grafo visuale
git log --oneline --graph --all --decorate
```

### Scenario tipico: lavorare su un nuovo branch

```bash
# 1. Creare e spostarsi sul nuovo branch
git switch -c nuovo-branch

# 2. Fare modifiche, aggiungere file, committare
git add .
git commit -m "adding fourth file"

# 3. Visualizzare la situazione
git log --oneline --graph --all --decorate
```

**Output visuale:**
```
* 32ab62f (HEAD -> nuovo-branch) adding fourth file
| * ce7a5b0 (main) adding third file
|/
* 5c05889 adding second file
* f33f7e5 adding first file
```

La storia si è **biforcata**: `main` e `nuovo-branch` hanno sviluppi indipendenti.

---

## 7. Merge

`git merge` unifica le modifiche provenienti da uno o più branch all'interno del branch corrente.

```bash
# 1. Tornare sul branch di destinazione
git switch main

# 2. Eseguire il merge
git merge nuovo-branch

# 3. Verificare
git log --oneline --graph --all --decorate
```

**Output dopo il merge:**
```
* e8bbe61 (HEAD -> main) Merge branch 'nuovo-branch'
|\  
| * 32ab62f (nuovo-branch) adding fourth file
* | ce7a5b0 adding third file
|/  
* 5c05889 adding second file
* f33f7e5 adding first file
```

> 💡 La struttura risultante è un **DAG — Directed Acyclic Graph** (Grafo Aciclico Diretto): ogni commit punta al predecessore, senza cicli.

---

## 8. Trunk-Based Development

Un modello di branching dove tutti gli sviluppatori lavorano su un unico branch principale (`main`/`trunk`) con branch di vita breve (feature branch).

```
main ────────────────────────────────────────
          └── feature-A ──┘
                    └── feature-B ──┘
```

- I branch feature restano aperti il meno possibile
- Si integra frequentemente su `main`
- Riduce i conflitti di merge

---

## 9. Git in Remoto

Finora Git è stato usato **in locale**. Ma:
- Se il laptop va a fuoco?
- Se si vuole collaborare con altri?

> Un **repository remoto** è un repository archiviato da qualche altra parte (server su internet).

### Servizi di hosting

| Servizio | URL | Note |
|---|---|---|
| **GitHub** | https://github.com | Il più diffuso; usato per questo corso |
| **GitLab** | https://gitlab.com | Open source, self-hosting possibile |
| **BitBucket** | https://bitbucket.org | Integrazione con Atlassian (Jira) |

### Comandi remote

```bash
# Aggiunge un repository remoto chiamato 'origin'
git remote add origin https://github.com/utente/repo.git

# Visualizza i remote configurati
git remote -v

# Invia i commit locali al remote (primo push)
git push -u origin main

# Push successivi
git push

# Scarica le modifiche dal remote e le integra
git pull

# Clona un repository remoto in locale
git clone https://github.com/utente/repo.git
```

### Schema locale ↔ remoto

```
Local Repository  ── git push ──→  Remote Repository (GitHub)
                  ── git pull ──←

Nuova macchina:   git clone ──←  Remote Repository (GitHub)
```

---

## 10. GitHub

**GitHub** è una piattaforma di hosting per repository Git con funzionalità collaborative aggiuntive.

### Setup account
1. Vai su https://github.com
2. Scegli un nome utente
3. Inserisci email e password
4. Clicca "Sign up for GitHub"

> ⚠️ Per il corso: registrare nome, cognome, GitHub ID e email dell'account su Moodle.

### Flusso tipico per il progetto

```bash
# 1. Creare repository su GitHub (via interfaccia web)

# 2. Collegare il repository locale al remoto
git remote add origin https://github.com/tuoNome/nomeRepo.git

# 3. Primo push
git push -u origin main

# 4. Ciclo di lavoro quotidiano
git add .
git commit -m "descrizione modifiche"
git push
```

---

## 11. Riepilogo Comandi Git

### Inizializzazione
```bash
git init                        # crea repository locale
git clone <url>                 # clona repository remoto
git config --global user.name "Nome"
git config --global user.email "email"
git config --list               # verifica configurazione
```

### Stato e diff
```bash
git status                      # stato dei file
git diff                        # working dir vs staging
git diff --staged               # staging vs ultimo commit
```

### Stage e commit
```bash
git add <file>                  # singolo file in staging
git add .                       # tutti i file modificati
git restore --staged <file>     # rimuove da staging
git commit -m "messaggio"       # crea commit
```

### History e navigazione
```bash
git log                         # cronologia commit
git log --oneline --graph --all --decorate
git checkout <commitID>         # torna a un commit (detached HEAD)
git checkout <nomeFile>         # ripristina file all'ultimo commit
```

### Branch
```bash
git branch                      # lista branch
git switch -c <nome>            # crea e sposta su nuovo branch
git switch <nome>               # sposta su branch esistente
git merge <branch>              # unisce branch nel corrente
```

### Remote
```bash
git remote add origin <url>     # aggiunge remote
git remote -v                   # mostra remote
git push -u origin main         # primo push
git push                        # push successivi
git pull                        # scarica e integra modifiche
```

---

## 12. Punti Chiave per l'Esame ✅

- **`git restore --staged <file>`** = toglie un file dalla staging area (non elimina le modifiche)
- **`git checkout <file>`** = scarta le modifiche non committate (irreversibile!)
- **`git checkout <commitID>`** = torna a un commit precedente → entra in **detached HEAD**
- **Detached HEAD**: nessun branch segue i nuovi commit; rischio garbage collection dopo ~30 giorni
- **Soluzione detached HEAD**: `git switch -c nuovo-branch` per ancorare i commit
- Un **branch** è un puntatore mobile all'ultimo commit di una linea di sviluppo
- **`git switch -c nome`** = crea e si sposta sul nuovo branch (modo moderno)
- **`git merge <branch>`** = porta le modifiche di `<branch>` nel branch corrente; si usa dopo `git switch main`
- Il risultato di commit e merge forma un **DAG** (Directed Acyclic Graph)
- **`git log --oneline --graph --all --decorate`** = visualizza tutta la storia ad albero
- Un **repository remoto** è un repository archiviato su server esterno (GitHub, GitLab, BitBucket)
- **`git push`** = invia commit locali al remote; **`git pull`** = scarica e integra modifiche dal remote
- Per il progetto d'esame: repository **GitHub pubblico**, link su Moodle, **nessun commit dopo la scadenza**
- Il **Trunk-Based Development** prevede branch di breve durata integrati frequentemente su `main`
