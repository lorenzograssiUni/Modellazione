# 01 - Storia della Programmazione, Paradigmi, Modelli di Esecuzione e Programmazione Procedurale

---

## 1. Cos'è la Programmazione?

> **Informale:** "La programmazione è il nostro modo di dire al computer cosa fare."
>
> **Formale:** "Programmare significa descrivere in modo formale un comportamento che una macchina deve eseguire."

### Cos'è un Linguaggio di Programmazione?

> **Informale:** "Un linguaggio di programmazione è il modo in cui spieghiamo alla macchina cosa vogliamo che faccia."
>
> **Formale:** "Un linguaggio di programmazione è un linguaggio formale con sintassi e semantica che descrive computazioni."

---

## 2. Tappe nella Storia della Programmazione

| Periodo | Era | Linguaggi / Tecnologie | Focus principale |
|---|---|---|---|
| 🧮 Anni '40–'50 | Linguaggio Macchina e Assembly | Codice binario, Assembly (ADD, MOV, STORE) | Controllo totale della macchina |
| 🧾 Fine anni '50 | Primi linguaggi ad alto livello | FORTRAN (1957), COBOL (1959), LISP (1958) | Astrazione dal linguaggio macchina |
| 🏗 Anni '60–'70 | Strutturazione e Metodologia | Pascal (1970), C (1972) | Programmazione strutturata, addio `goto` |
| 🧱 Anni '80 | Programmazione OOP | C++ (1983) | Oggetti, classi, incapsulamento, riuso |
| 🌐 Anni '90 | Internet e Virtual Machine | Python (1991), Java (1991–1995), JavaScript (1995) | Portabilità, linguaggi multiparadigma, web |
| ⚡ Anni 2000 | Multi-paradigma e Framework | C#, PHP, Scala | Integrazione tra paradigmi, astrazione più alta |
| ☁ Anni 2010 | Concorrenza, Cloud, Mobile | Go, Rust, Kotlin, Swift | Sicurezza, parallelismo, performance |
| 🤖 Anni 2020 | AI, DSL e Automazione | Python dominante per AI/ML/DL, low-code | Programmazione sempre più ad alto livello |
| 🧠 2020+ | Programmazione Assistita dall'AI | GitHub Copilot (2021), ChatGPT (2022) | Prompt-driven development, collaborazione uomo-AI |

### Punto chiave: AI e Programmazione

Usare l'AI senza capire il codice è **il nuovo "cut and paste programming"**.

- Devi comunque capire cosa fa il codice generato
- Devi verificarne la correttezza
- Devi integrarlo e mantenerlo nel tempo
- **L'AI non si assume la responsabilità del codice: lo sviluppatore sì.**

> "Studiare programmazione non serve per scrivere codice a mano. Serve per **capire il codice** che qualcuno — o qualcosa — scrive per noi."

---

## 3. Architettura di Von Neumann

Modello teorico fondamentale alla base dei moderni computer digitali, ideato dal matematico **John von Neumann nel 1945**.

- Unità di elaborazione (CPU)
- Memoria (dati + istruzioni nello stesso spazio)
- Unità I/O
- Bus di comunicazione

È la base concettuale della **programmazione imperativa**: il programma manipola uno stato in memoria attraverso istruzioni eseguite in sequenza.

---

## 4. Il problema del `goto` e la Programmazione Strutturata

### Origini: controllo del flusso nei primi programmi

Nei primi programmi tutto era scritto in sequenza con **salti espliciti** (`JMP` in Assembly, `GOTO` in C):

```assembly
start:
  MOV AX, 10
loop_start:
  DEC AX
  CMP AX, 0
  JE loop_end
  JMP loop_start
loop_end:
  HLT
```

Con la crescita della complessità, il codice diventava:
- Difficile da leggere
- Difficile da modificare
- Difficile da capire dopo mesi
- Difficile da riutilizzare

→ Nasce il termine **"Spaghetti Code"**.

### Teorema di Böhm-Jacopini (1966)

> Enunciato dagli informatici **Corrado Böhm** e **Giuseppe Jacopini**.

**Enunciato:** Qualsiasi algoritmo può essere espresso usando solo **tre strutture di controllo**:
1. **Sequenza** (composizione)
2. **Selezione** (`if` / `switch`)
3. **Iterazione** (`for` / `while`)

> Il teorema dimostra che il `goto` **non è necessario** per esprimere algoritmi, ma non dimostra che sia inutile dal punto di vista pratico.

**Conseguenza:** In Java e Python il `goto` è stato **rimosso**. In C è stato mantenuto perché C privilegia la libertà del programmatore.

### E. Dijkstra (1968) — "Go To Statement Considered Harmful"

Dijkstra usa il teorema come base per sostenere:
- La **programmazione strutturata**
- L'importanza della **leggibilità**
- La **riduzione dei salti arbitrari**

---

## 5. Paradigmi di Programmazione

> Un **paradigma di programmazione** è uno stile, un approccio o un modello concettuale utilizzato per progettare, organizzare e strutturare il codice sorgente.
>
> Rappresenta un **"modo di pensare" prima ancora di scrivere il codice**.

### Mappa dei Paradigmi

```
Imperativa
├── Procedurale
│     └── Strutturata
│           └── Object-Oriented (OOP)
Dichiarativa
├── Logica
└── Funzionale
```

### 5.1 Programmazione Imperativa

- Il programma è una **sequenza di istruzioni**
- Lo **stato della memoria cambia** nel tempo
- Il programmatore **controlla esplicitamente il flusso**

```c
x = 5;
y = 3;
z = x + y;  // assegnamento, modifica di stato, ordine esplicito
```

### 5.2 Programmazione Procedurale

- Il programma è organizzato in **procedure/funzioni**
- I **dati sono separati** dalle funzioni
- Le funzioni **operano su strutture dati**
- Linguaggi: **C, Pascal, Fortran**

```c
void decrementa(int* x) {
    (*x)--;  // funzione separata dai dati
}
```

> Una procedura: incapsula comportamento, crea un'unità logica, ha parametri, può essere riutilizzata → **evita duplicazione**.

### 5.3 Programmazione Strutturata

> "Procedurale senza goto, con blocchi e strutture di controllo."

Costruita usando **solo** i tre costrutti del Teorema di Böhm-Jacopini:

```c
// NON strutturato (con goto)
int i = 0;
inizio:
  if (i >= 10) goto fine;
  printf("%d\n", i);
  i++;
  goto inizio;
fine:

// STRUTTURATO
for (int i = 0; i < 10; i++) {
    printf("%d\n", i);
}
```

### 5.4 Programmazione Orientata agli Oggetti (OOP)

- Il programma è organizzato in **oggetti**
- **Dati e metodi sono incapsulati** insieme
- Si usa **ereditarietà** e **polimorfismo**
- Linguaggi: **Java, C++, C#, Python, Ruby**

```java
class Contatore {
    private int valore;         // dato
    void incrementa() {         // metodo
        valore++;               // stato + comportamento nella stessa entità
    }
}
```

> Java è multiparadigma, ma **nasce OOP**.

### 5.5 Programmazione Dichiarativa

- Si **descrive il risultato desiderato**, non i passi dettagliati
- Linguaggio tipico: **SQL**

```sql
SELECT nome
FROM studenti
WHERE voto > 28;
-- Non specifichiamo COME trovare i dati, ma COSA vogliamo
```

### 5.6 Programmazione Logica

- Il programma è un insieme di **fatti e regole**
- L'esecuzione è una **dimostrazione logica**
- Si fanno **query** al sistema
- Linguaggio principale: **Prolog**

```prolog
padre(mario, luca).
padre(luca, anna).
?- padre(mario, X).   % Query
X = luca              % Risposta
```

### 5.7 Programmazione Funzionale

- Il programma è composto da **funzioni matematiche**
- **Niente stato mutabile**
- **Nessun effetto collaterale** (idealmente)
- Linguaggi: **Haskell, Lisp, OCaml**

```haskell
somma x y = x + y   -- nessuna variabile che cambia valore
```

> Eccellente per calcolo matematico e per evitare bug legati alla gestione dei dati.

---

## 6. Modelli di Esecuzione

> **Distinzione fondamentale:**
> - Il **paradigma** risponde a *"come organizzo il codice"*
> - Il **modello di esecuzione** risponde a *"come il codice vive nel tempo"*

### 6.1 Esecuzione Sequenziale

Una istruzione alla volta, **in ordine**.

```java
System.out.println("A");
System.out.println("B");
System.out.println("C");
// Output: A, B, C (sempre in questo ordine)
```

### 6.2 Esecuzione Concorrente

Più attività **avanzano nello stesso intervallo di tempo** (non necessariamente nello stesso istante → **alternanza**).

```java
Thread t = new Thread(() -> {
    System.out.println("Secondario");
});
t.start();
System.out.println("Principale");
// ⚠ L'ordine può cambiare — può essere su 1 solo core
```

### 6.3 Esecuzione Parallela

Più istruzioni eseguite **nello stesso istante reale**. Richiede **più core o CPU**.

```java
List.of(1, 2, 3, 4)
    .parallelStream()
    .map(x -> x * 2)
    .forEach(System.out::println);
// La JVM può usare più core
```

> Verifica i processori disponibili: `Runtime.getRuntime().availableProcessors()`

### 6.4 Esecuzione Event-Driven

Il programma **reagisce agli eventi** (click, messaggi, input). Non esegue tutto in sequenza: è in attesa finché non accade un evento.

```java
button.addActionListener(e -> {
    System.out.println("Click!");
});
// Il codice si attiva solo quando accade l'evento
```

Usato in: **JavaScript (browser), Java GUI, C# .NET GUI, Swift (iOS)**

### Riepilogo Modelli di Esecuzione

| Modello | Idea | Metafora |
|---|---|---|
| Sequenziale | Una cosa alla volta | Una persona che lavora |
| Concorrente | Più attività avanzano insieme | Una persona che alterna compiti |
| Parallelo | Più attività nello stesso istante | Più persone che lavorano |
| Event-Driven | Il flusso dipende dagli eventi | Una persona che aspetta richieste |

> ⚠️ I modelli **non sono categorie chiuse**: un sistema può essere Event-Driven **e** concorrente, OOP **e** parallelo, ecc.

### Modelli Avanzati (cenno)

- **Asincrono** → operazioni non bloccanti
- **Reactive** → sistemi reattivi a flussi di eventi
- **Distribuito** → esecuzione su più macchine in rete
- **Actor Model** → componenti indipendenti che comunicano tramite messaggi
- **Dataflow / Stream-based** → esecuzione guidata dal flusso dei dati

---

## 7. Programmazione Procedurale — Approfondimento

### Definizione

> "Il programma è una sequenza di istruzioni eseguite in ordine. Il controllo del flusso è esplicito. **Il programma dice passo passo cosa fare.**"

### Quando si usa?

- Script di automazione (backup, elaborazione file, task ripetitivi)
- Programmi scientifici e numerici (simulazioni, calcolo matriciale)
- Utility e tool di sistema (parsing input, trasformazioni dati)
- Programmi piccoli o ben delimitati (un compito, un flusso chiaro)

Esempi pratici: script Bash/PowerShell, macro Excel, piccoli programmi in C o Python.

### I tre elementi fondamentali della programmazione procedurale

#### 1. Dati
- Variabili
- Strutture dati (array, liste, record...)
- **Rappresentano lo stato del programma** (ciò che il programma "sa" in un certo momento)

#### 2. Procedure / Funzioni
- Incapsulano operazioni
- Ricevono dati in ingresso (parametri)
- Producono risultati o effetti

> "Le funzioni dicono come lavorare sui dati. I dati da soli non fanno nulla."

#### 3. Strutture di Controllo
- **Sequenza** — le istruzioni si eseguono nell'ordine scritto
- **Selezione** — `if` / `switch` (scelte condizionali)
- **Iterazione** — `for` / `while` (cicli)

### Come lavorano insieme

```
[Flusso di controllo]
        ↓
  chiama Funzione
        ↓
  opera su Dati
        ↓
  modifica Stato
```

> "Nel modello procedurale c'è una **'regia centrale'** che decide cosa succede e quando. I dati non fanno nulla da soli: sono le funzioni chiamate dal flusso di controllo a operare su di essi."

### Esercizio tipo — Classificazione

```
saldo          → DATO
calcolaTotale()→ FUNZIONE/PROCEDURA
if             → STRUTTURA DI CONTROLLO
listaContatti  → DATO
stampaReport() → FUNZIONE/PROCEDURA
```

---

## 8. Diagramma di Flusso

Una **rappresentazione grafica** di un processo o algoritmo. Mostra visivamente sequenze di azioni, decisioni e percorsi possibili.

**Storia:**
- Anni '20: introdotto da Frank e Lillian Gilbreth per analizzare i processi industriali
- Anni '40–'50: adottato nell'informatica per rappresentare algoritmi e programmi

**A cosa serve:**
- Visualizzare un processo in modo chiaro
- Analizzare e migliorare procedure
- Progettare algoritmi informatici
- Facilitare la comunicazione tra persone o team

### Simboli principali

| Simbolo | Forma | Significato |
|---|---|---|
| Inizio/Fine | Ovale / Rettangolo arrotondato | Punto di start/stop |
| Istruzione | Rettangolo | Operazione o azione |
| Decisione | Rombo | Condizione (sì/no) |
| I/O | Parallelogramma | Lettura/scrittura dati |
| Flusso | Freccia | Direzione del flusso |

### Esercizio tipo da esame

**Traccia:** Scrivere l'algoritmo che:
1. Legga un numero intero positivo `N`
2. Calcoli la somma di tutti i numeri da 1 a N
3. Stampi il risultato finale

**Pseudocodice:**
```
INIZIO
  Leggi N
  somma ← 0
  i ← 1
  MENTRE i <= N FARE
    somma ← somma + i
    i ← i + 1
  FINE MENTRE
  Stampa somma
FINE
```

---

## 9. Perché esistono metodologie diverse?

- Problemi diversi richiedono strumenti diversi (calcolo numerico, gestione dati, interfacce, sistemi distribuiti...)
- La **complessità cresce** con la dimensione del software → serve organizzare meglio codice e responsabilità
- Il software **cambia nel tempo** → manutenzione ed estendibilità contano quanto "funzionare"
- **Collaborazione in team** → servono regole/strutture che rendano il codice leggibile e condivisibile

> "Due programmi possono produrre lo stesso risultato oggi. Ma se uno è organizzato meglio dell'altro, domani — quando dovremo modificarlo — uno ci costerà pochi minuti, l'altro ore o giorni."

---

## 10. Punti Chiave per l'Esame ✅

- **Programmazione** = descrivere in modo formale un comportamento che una macchina deve eseguire
- La storia della programmazione segue un trend di **crescente astrazione** (da bit fisici → linguaggio macchina → assembly → alto livello → OOP → AI-assisted)
- Il **Teorema di Böhm-Jacopini (1966)**: qualsiasi algoritmo si può esprimere con Sequenza + Selezione + Iterazione → rende il `goto` non necessario
- **Dijkstra (1968)**: "Go To Statement Considered Harmful" → base della programmazione strutturata
- I **paradigmi** definiscono *come si organizza il codice*; i **modelli di esecuzione** definiscono *come il codice vive nel tempo*
- Differenza chiave **Concorrente vs Parallelo**: concorrente = alternanza su 1+ core; parallelo = stesso istante su più core
- I 3 elementi della programmazione procedurale: **Dati + Funzioni + Strutture di controllo**
- **"Non basta che funzioni"** → il codice deve essere leggibile, manutenibile ed estendibile
- Usare AI senza capire il codice = nuovo "cut and paste programming"
- `Java` è multiparadigma ma **nasce OOP** — è il linguaggio del progetto d'esame
