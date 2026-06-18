# 04 - Approcci Diversi per la Stessa Soluzione, Introduzione a GIT

---

## 1. Approcci Diversi per la Stessa Soluzione

Spesso non esiste un'unica soluzione corretta. Quando più soluzioni funzionano, entrano in gioco criteri di qualità.

> **Regola d'oro:** Scegliete il modo che rende il codice più chiaro per chi lo legge. Non esiste il modo giusto in assoluto, esiste il modo migliore **in base al contesto**.

### Esempio: stampare tutti gli elementi di un ArrayList

```java
ArrayList<String> nomi = new ArrayList<>();
nomi.add("Marco");
nomi.add("Giulia");
```

| Metodo | Codice | Quando usarlo |
|---|---|---|
| **0 – Accesso diretto** | `System.out.println(nomi.get(0));` | Per prendere un elemento preciso; non per scorrere la lista |
| **1 – For classico** | `for (int i = 0; i < nomi.size(); i++) { System.out.println(nomi.get(i)); }` | Quando serve l'indice (es. stampare `i: elemento`) |
| **2 – For-each** | `for (String nome : nomi) { System.out.println(nome); }` | Semplice e chiaro, solo stampa senza indice |
| **3 – forEach (funzionale)** | `nomi.forEach(nome -> System.out.println(nome));` | Codice compatto, stile moderno/funzionale |
| **4 – toString implicito** | `System.out.println(nomi);` | Debug veloce; stampa `[Marco, Giulia]` |
| **5 – String.join** | `System.out.println(String.join(", ", nomi));` | Output su una riga formattata: `Marco, Giulia` |

### Criteri di scelta

Quando più soluzioni funzionano, valutare:
- **Leggibilità** — il prossimo che legge il codice capisce subito cosa fa?
- **Semplicità** — usa il costrutto più semplice che risolve il problema
- **Coerenza** — usa lo stesso stile nel resto del progetto
- **Manutenibilità** — sarà facile modificarlo in futuro?

---

## 2. Introduzione a GIT

### Scenario: perché serve GIT?

> Sono le 23:00 del giorno della consegna del progetto. Il progetto è quasi pronto. Scoprite un piccolo bug e decidete di fixarlo. Cambiate diverse linee di codice su diversi file. Il codice non funziona più. **Ctrl+Z non funziona.**

Questo scenario illustra perfettamente perché il controllo versione è fondamentale.

### Cos'è il Controllo delle Versioni?

> È un sistema che permette di registrare cambiamenti a file o insiemi di file così da potersi "muovere" tra le varie versioni (es. ritornare a una versione precedente se le modifiche apportate non vanno bene).

> **Versionare il codice non significa salvare file: significa gestire, proteggere e far evolvere la conoscenza che avete costruito.**

Quando scrivete codice state:
- Formalizzando una soluzione
- Codificando decisioni di design
- Esplicitando regole di business
- Traducendo concetti in strutture

### Evoluzione dei sistemi di Version Control

| Opzione | Nome | Problema |
|---|---|---|
| **0** | Manuale (copie cartelle) | Caos, errori umani, nessuna storia |
| **1** | Local VCS | Impossibile collaborare con altri |
| **2** | Centralized VCS (CVCS) | Single point of failure: se il server è down, nessuno lavora |
| **3** | Distributed VCS (DVCS) ✅ | Tutto lo storico è disponibile localmente su ogni macchina |

---

## 3. GIT

### Cos'è GIT?

- **Sistema di Controllo delle Versioni Distribuito** (DVCS)
- Progetto **open source** creato nel **2005 da Linus Torvalds**
- Strumento utilizzabile da **linea di comando**
- Risiede "sopra" al file system e manipola file

> 🔗 Sito ufficiale: https://git-scm.com/

### Chi è Linus Torvalds?

Programmatore finlandese nato nel 1969. A 22 anni, da studente universitario, iniziò a sviluppare il kernel **Linux** come progetto personale. Linux è diventato la base di sistemi operativi (Ubuntu, Debian...), server internet e Android. Ha creato anche **Git** nel 2005.

### Perché Linus ha creato Git?

Nel 2005 il kernel Linux usava **BitKeeper** per gestire il codice, ma la licenza gratuita venne revocata. Le alternative esistenti erano:
- Troppo lente
- Poco scalabili
- Difficili per team distribuiti

Torvalds creò Git con questi obiettivi:
- **Velocità**
- **Distribuito** (ogni sviluppatore ha tutto il progetto)
- **Integrità dei dati** (hash crittografici)
- **Branching e merging efficienti**

La prima versione funzionante fu pronta in pochi giorni/settimane. Poi il progetto venne affidato a **Junio Hamano**.

---

## 4. Concetti Fondamentali di GIT

### Repository

> La struttura in cui Git salva i dati è chiamata **repository**.

Si può immaginare come un **albero**: ogni ramo è una versione diversa e ogni nodo è un punto di salvataggio.

Un repository Git si ottiene in due modi:
1. **`git init`** — si prende una cartella locale e ci si crea un Git repository
2. **`git clone <url>`** — si effettua una clonazione di un repository già esistente

### Le Tre Sezioni di Git

```
Working Directory  →  Staging Area  →  Repository
  (modifiche)         (git add)        (git commit)
```

| Sezione | Descrizione |
|---|---|
| **Working Directory** | Cartella di lavoro sul disco; dove si modificano i file |
| **Staging Area** | "Area di sosta" — zona intermedia dove si prepara il prossimo commit |
| **Repository** | Storico permanente di tutti i commit (cartella `.git`) |

**Flusso Git:**
1. Modificare file nell'area di lavoro
2. Posizionare i file nell'area di sosta (`git add`)
3. Spostare i file dall'area di sosta nel repository, creando un punto di salvataggio (`git commit`)

> ⚠️ Il commit salva lo stato della **staging area**. Tutto ciò che hai modificato ma non hai messo in staging rimane nella working directory e **non viene salvato nel commit**.

### Cos'è un Commit?

Un commit contiene tre elementi principali:

| Elemento | Descrizione | Esempio |
|---|---|---|
| **Hash (SHA1)** | Stringa di 40 caratteri che identifica univocamente il commit | `a3f5c2d...` |
| **Messaggio** | Descrive le modifiche introdotte | `"Add: login feature"` |
| **Insieme di modifiche** | I cambiamenti effettivi ai file | diff dei file modificati |

> 💡 Il commit è come un **punto di salvataggio** in un videogioco — puoi sempre tornare indietro a quel punto.

### Cos'è un Hash (SHA1)?

Una **Cryptographic Hash Function (CHF)** è un algoritmo matematico che mappa dati di dimensioni arbitrarie in un array di bit di dimensioni fisse.

- **Unidirezionale**: praticamente impossibile da invertire
- Lo stesso input produce **sempre lo stesso hash**
- Una modifica minima al contenuto produce un hash **completamente diverso**

> 🔗 Prova online: http://www.sha1-online.com/

---

## 5. Comandi Git Essenziali

### Setup iniziale

```bash
# Controlla versione installata
git --version

# Configura nome ed email (usate per ogni commit)
git config --global user.name "Nome Cognome"
git config --global user.email nome.cognome@studenti.unicam.it

# Verifica configurazione
git config --list
```

### Inizializzare e osservare

```bash
# Crea un nuovo repository nella cartella corrente
git init
# → crea la cartella nascosta .git (non eliminarla mai!)

# Mostra lo stato dei file
git status
# → master/main = nome del branch corrente
# → directory pulita = nessuna modifica rilevata

# Visualizza la cronologia dei commit
git log
git log -p -2                          # mostra le diff degli ultimi 2 commit
git log --stat                         # statistiche file modificati
git log --pretty=oneline               # una riga per commit
git log --pretty=format:"%h -%an, %ar: %s"  # formato personalizzato
```

### Aggiungere e committare

```bash
# Aggiunge un file specifico alla staging area
git add nomeFile

# Aggiunge TUTTI i file modificati alla staging area
git add .

# Crea un commit con i file in staging
git commit -m "Messaggio descrittivo"
```

### Confrontare versioni

```bash
# Confronta working directory con staging area
git diff

# Confronta staging area con l'ultimo commit
git diff --staged
```

### Schema del flusso

```
[Nuovo file]
     │
  git add nomeFile
     │
     ▼
[Staging Area]
     │
  git commit -m "messaggio"
     │
     ▼
[Repository .git]
     │
  git log → lista commit
```

---

## 6. Best Practices per il Progetto d'Esame

> **È meglio fare molti commit con piccole modifiche piuttosto che un unico commit con modifiche massicce: i commit piccoli sono più facili da leggere e da revisionare.**

Regole fondamentali:
- **Non eliminare mai** la cartella `.git` → perdereste tutta la cronologia
- **Commitmate spesso** con messaggi significativi
- Il commit registra l'istantanea preparata nella staging area; ciò che non è in staging non viene salvato
- **Nessun commit dopo la scadenza** della consegna del progetto (10 giorni prima dell'esame)
- Il repository GitHub deve essere **pubblico**; il link va consegnato su Moodle

### Visualizzare i commit come albero

```
File 1 ──┐
          ├── Commit 1 ──→ Commit 2 ──→ Commit 3
File 2 ──┘                              │
                                        File 3 (aggiunto)
```

Ogni commit punta al suo predecessore → si forma una catena che rappresenta la storia del progetto.

---

## 7. Punti Chiave per l'Esame ✅

- Quando più soluzioni funzionano, si sceglie in base a **leggibilità, semplicità, coerenza, manutenibilità**
- I 6 modi per iterare un ArrayList: accesso diretto, `for` classico, `for-each`, `forEach`, `toString`, `String.join`
- **Version Control** = sistema per registrare e navigare tra le versioni di un progetto
- Evoluzione: Manuale → Local VCS → Centralized VCS → **Distributed VCS (Git)**
- **Git** = DVCS open source creato da **Linus Torvalds nel 2005**, poi sviluppato da Junio Hamano
- Git nasce perché BitKeeper (usato da Linux) revocò la licenza gratuita
- Le **3 sezioni** di Git: Working Directory, Staging Area, Repository
- **`git add`** → sposta dalla working directory alla staging area
- **`git commit`** → sposta dalla staging area al repository
- Un commit = hash SHA1 + messaggio + modifiche
- **SHA1** = funzione hash unidirezionale a 40 caratteri
- **`git status`** = comando più usato; mostra lo stato dei file e il branch corrente
- Il primo branch si chiama `master` o `main`
- La cartella `.git` contiene tutta la storia del repository → **non eliminarla mai**
- Per il progetto: molti commit piccoli > un commit enorme; nessun commit dopo la scadenza
