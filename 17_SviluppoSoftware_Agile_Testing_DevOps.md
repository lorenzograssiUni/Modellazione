# 17 - Sviluppo Software, Agile, Software Testing, DevOps

---

## 1. Recap: Modelli di Esecuzione

Breve riepilogo dalla lezione 16 — il modello di esecuzione risponde a *"come il codice vive nel tempo"*:

| Modello | Descrizione | Keyword |
|---|---|---|
| **Sequenziale** | Un'istruzione alla volta, in ordine | Semplice, prevedibile |
| **Concorrente** | Più attività gestite nello stesso intervallo (interleaving) | Thread, `Runnable`, `synchronized` |
| **Parallelo** | Più operazioni eseguite fisicamente nello stesso istante su più core | `ExecutorService`, `parallelStream()` |
| **Event-Driven** | Il flusso di esecuzione è guidato da eventi esterni | GUI, click, messaggi |

### Concorrenza: il problema di `count++`

`count++` **non è atomica**: sono 3 operazioni separate (leggi → modifica → scrivi). Se due thread le eseguono contemporaneamente si sovrappongono → **Race Condition** → risultato imprevedibile.

```java
// PROBLEMA: senza sincronizzazione
public class Counter {
    private int count = 0;
    public void increment() { count++; }  // NON atomica!
}

// SOLUZIONE: synchronized
public class Counter {
    private int count = 0;
    public synchronized void increment() { count++; }  // un thread alla volta
}
```

### `ExecutorService` e parallelismo pratico

```java
// Crea un pool di 2 thread (gestiti automaticamente)
ExecutorService executor = Executors.newFixedThreadPool(2);

Future<?> f1 = executor.submit(() -> lavoroPesante());  // avvia su thread del pool
Future<?> f2 = executor.submit(() -> lavoroPesante());

f1.get();  // aspetta che f1 finisca
f2.get();  // aspetta che f2 finisca
executor.shutdown();
```

- Sequenziale `~660ms` → con `ExecutorService` `~330ms` → con `parallelStream()` `~150ms`
- La JVM può usare più core ma **non possiamo forzare un thread su uno specifico core** (violarebbe "Write Once, Run Anywhere")

> Il parallelismo esiste ma non è garantito: il sistema operativo decide come schedulare i thread.

---

## 2. Sviluppo Software — Cenni Storici

### La Crisi del Software (fine anni '60)

> *"La causa principale della crisi del software è che le macchine sono diventate parecchi ordini di grandezza più potenti!"* — Dijkstra, 1972

Sintomi della crisi:
- Progetti fallimentari o oltre il budget
- Software consegnato in ritardo
- Software di scarsa qualità, con errori
- Software che non rispettava i requisiti
- Codice ingestibile e difficile da mantenere

### Risposta: nasce l'Ingegneria del Software

Lo sviluppo software viene riconosciuto come **processo complesso**, analogo a quello degli altri prodotti ingegneristici. Si identificano fasi strutturate:

| Fase | Contenuto |
|---|---|
| **Pianificazione** | Obiettivi, organizzazione del team, scadenze |
| **Analisi dei Requisiti** | Cosa il prodotto dovrà fare e per quali utenti |
| **Progettazione** | Modelli che descrivono le componenti del software |
| **Sviluppo** | Scrittura del codice |
| **Collaudo** | Testing per cercare bug prima del rilascio |
| **Rilascio** | Messa in funzione del software |
| **Manutenzione** | Modifiche e aggiornamenti |
| **Documentazione** | Scrittura di manuali per l'utilizzo |

---

## 3. Modello a Cascata (Waterfall)

Il modello classico: ogni fase è **completamente separata** dalle altre, completata prima di passare alla successiva. Analogo a una **staffetta**: il testimone passa solo quando la fase precedente è conclusa.

```
Analisi Requisiti
       ↓
 Progettazione
       ↓
  Sviluppo
       ↓
  Collaudo
       ↓
  Rilascio
       ↓
 Manutenzione
```

**Vantaggi:**
- Guidato dalla produzione di documenti
- Progressi visibili e quantificabili
- Documenti utili per gestire il cambio di personale

**Svantaggi:**
- Troppo concentrato sui documenti
- Il software rilasciato solo alla fine di tutte le fasi
- Cliente coinvolto **solo nella fase iniziale**
- Cambiamenti nei requisiti non previsti → difficile adattarsi

---

## 4. Agile

### Contesto Storico

Nel 2001, 17 esperti di sviluppo software (tra cui **Robert C. Martin** — Uncle Bob, co-autore del Manifesto Agile e promotore di SOLID) firmano il **Manifesto Agile**.

> 🔗 https://agilemanifesto.org/iso/it/manifesto.html

### I 4 Valori del Manifesto Agile

| Preferire... | ...rispetto a |
|---|---|
| **Individui e interazioni** | Processi e strumenti |
| **Software funzionante** | Documentazione esaustiva |
| **Collaborazione col cliente** | Negoziazione dei contratti |
| **Rispondere al cambiamento** | Seguire un piano |

### Caratteristiche del Modello Agile

- Il lavoro complesso viene diviso in **piccole parti**
- Grandi organizzazioni vengono divise in **piccoli team**
- Lunghi progetti vengono divisi in liste di task da portare a termine in un **periodo breve**
- I requisiti possono cambiare in ogni momento

---

## 5. SCRUM

Framework Agile tra i più usati. Si basa su 4 componenti: **Ruoli, Sprint, Artefatti, Eventi**.

### Ruoli

| Ruolo | Responsabilità |
|---|---|
| **Product Owner** | Massimizza il valore del prodotto; comunica con il cliente; definisce priorità e requisiti |
| **Scrum Team** | 3–9 persone versatili con varie competenze; lavoro autogestito |
| **Scrum Master** | Mentor e coach del team; facilita il lavoro; garantisce che il team segua i valori Scrum |

> **Scrum Master ≠ Project Manager**: lo Scrum Master *guida* senza gestire (non ha autorità sul team). È come un coach.

### Sprint

- Periodo di tempo di **2–4 settimane**
- Inizia con la definizione delle attività
- Termina con il **rilascio del software** (incremento funzionante)

### Artefatti

- **Product Backlog**: lista completa di requisiti e funzionalità da implementare (gestita dal Product Owner)
- **Sprint Backlog**: sottoinsieme di user story selezionate per lo sprint corrente
- **Prodotto Software**: l'incremento funzionante consegnato a fine sprint

### User Story (formato standard)

```
Come [tipo di utente]
Voglio [funzionalità]
Così da [beneficio]

Es:
Come utente registrato
Voglio poter effettuare il login
Così da accedere alle mie informazioni personali
```

### Eventi (check-point dello Sprint)

1. **Pianificazione dello Sprint** — cosa fare nello sprint
2. **Scrum Giornaliero** (Daily Scrum) — 15 min di aggiornamento quotidiano
3. **Revisione dello Sprint** — demo del software prodotto
4. **Retrospettiva** — cosa è andato bene/male, come migliorare

### Il Ciclo SCRUM

```
Product Backlog → Sprint Planning → Sprint Backlog
                                          ↓
                          [Sprint 2-4 settimane]
                          Daily Scrum ogni giorno
                                          ↓
                              Incremento Software
                                          ↓
                    Sprint Review + Retrospective
                                          ↓
                             (prossimo Sprint)
```

---

## 6. Software Testing

### Cos'è il Testing?

> Il testing è l'attività che consiste nel **verificare se una porzione di codice produce il comportamento previsto**.

Non significa "dimostrare che il programma è perfetto".

**Testing vs Debugging:**
- **Testing** → scopo: *scovare* i bug
- **Debugging** → scopo: *risolvere* i bug trovati in fase di testing

### La Test Stack (Google Testing Blog)

| Livello | Nome | Cosa testa | Rete | DB | Tempo max |
|---|---|---|---|---|---|
| **Small** | Unit Test | Singola unità (metodo/classe) | No | No | 60s |
| **Medium** | Integration Test | Gruppi di moduli | localhost | Sì | 300s |
| **Large** | System Test | Sistema completo e integrato | Sì | Sì | 900s+ |

---

## 7. Unit Test con JUnit

**JUnit** è un framework di test per Java che usa **annotazioni** per identificare i metodi di test.

```java
// Classe da testare
public class MyClass {
    public int multiply(int a, int b) {
        return (a * b);
    }
}

// Classe di test
public class MyClassTest {

    @Test
    public void multiplicationOfZeroIntegersShouldReturnZero() {
        MyClass tester = new MyClass();

        // assert: verifica che il risultato atteso corrisponda all'effettivo
        assertEquals(0, tester.multiply(10, 0), "10 x 0 must be 0");
        assertEquals(0, tester.multiply(0, 10), "0 x 10 must be 0");
        assertEquals(0, tester.multiply(0,  0), "0 x 0 must be 0");
    }
}
```

**Regole di base:**
- I test sono metodi `public void` annotati con `@Test`
- La classe di test si chiama `NomeClasseTest` (es. `LibrettoTest`)
- I test si organizzano in **package** paralleli al codice sorgente
- Si usa `assertEquals(atteso, effettivo, messaggio)` e altri metodi `assert`

---

## 8. Extreme Programming (XP) e TDD

### Extreme Programming (XP)

Methodologia Agile che prevede:
- Rilasci frequenti in cicli di sviluppo brevi
- **Pair programming** (programmare in coppia) o revisione del codice
- Test unitari per ogni singolo pezzo di codice
- Semplicità e chiarezza del codice (**KISS**)
- Comunicazione frequente con il cliente
- Aspettarsi cambiamenti nei requisiti

XP utilizza **TDD** e il **refactoring** per individuare la progettazione più efficace.

### Test-Driven Development (TDD)

> **Scrivi il test PRIMA del codice da testare.**

**Il ciclo TDD:**

```
1. 🔴 RED    → Scrivi un test che fallisce (il codice non esiste ancora)
2. 🟢 GREEN  → Scrivi il minimo codice per far passare il test
3. 🔵 REFACTOR → Migliora il codice mantenendo il test verde
4. Ripeti.
```

- *"Un piccolo test, un po' di codice, un piccolo test, un po' di codice..."*
- Ogni ciclo dura **pochi minuti**: piccoli passi → difficile commettere errori
- I test **specificano** cosa dovrebbe fare il codice
- I test **verificano** che il codice faccia ciò che dovrebbe
- Il refactoring può essere eseguito in sicurezza **solo con un solido sistema di test**

> TDD è una vera e propria pratica di **design e coding**, non solo di testing.

---

## 9. DevOps

### The Product Pipeline

```
Osservazioni → Idee → Design → Codice → Test → Deploy → Release → Software in Produzione
              ←────────────────────────────────────────────────────────→
                          Continuous Design
              ←──────────────────────────────────────────────→
                        Continuous Integration
              ←──────────────────────────────────────────────────────→
                              Continuous Delivery
                                                  ←────────────────────→
                                            Continuous Deployment
                                              (automatico vs manuale)
```

### Ruoli Tradizionali

| Ruolo | Input | Output |
|---|---|---|
| **Developer** | User story | Progettazione e implementazione del software |
| **Tester** | Software funzionante + note | Software validato |
| **Operator** | Note di implementazione validate | Sistemi funzionanti, monitoraggio, verifica |

Nel modello classico, questi ruoli erano **isolati**. DevOps li integra.

### Cos'è DevOps?

> *"DevOps, termine che unisce sviluppo (Dev) e operazioni (Ops), rappresenta l'integrazione di persone, processi e tecnologie per fornire valore in modo continuativo ai clienti."* — Azure.microsoft.com

DevOps consente a ruoli precedentemente isolati (sviluppo, operazioni IT, ingegneria della qualità, sicurezza) di coordinarsi e collaborare per:
- Rispondere meglio alle esigenze dei clienti
- Aumentare la fiducia nelle applicazioni
- Raggiungere più rapidamente gli obiettivi aziendali

### CI/CD — Continuous Integration / Continuous Delivery

| Pratica | Descrizione |
|---|---|
| **Continuous Integration (CI)** | Ogni commit viene automaticamente compilato e testato; i bug emergono subito |
| **Continuous Delivery (CD)** | Il software è sempre in uno stato rilasciabile; il deploy è manuale |
| **Continuous Deployment** | Il deploy avviene automaticamente dopo ogni build verde |

> **GitHub Actions** (che usate già nel progetto) è uno strumento di CI/CD: ogni push può eseguire automaticamente build e test.

---

## 10. Collegamento con il Progetto RPG

| Concetto | Applicazione nel progetto |
|---|---|
| **Agile / Scrum** | Il progetto è strutturato come "sprint": deliverable incrementali con scadenza |
| **User Story** | Descrivere le funzionalità RPG come user story aiuta a pianificare |
| **Unit Test (JUnit)** | Testare `calcolaMedia()`, `registraEsameSuperato()`, `equals()` |
| **TDD** | Scrivere prima il test di `getVoto()` → poi implementarlo in `Libretto` |
| **DevOps / CI** | GitHub repository pubblico = già parte del ciclo CI/CD |
| **Documentazione** | La Wiki su GitHub è un artefatto del processo (come nel Waterfall, ma agile) |

---

## 11. Punti Chiave per l'Esame ✅

- **Crisi del Software** (fine anni '60): progetti fallimentari, software di scarsa qualità → nasce l'Ingegneria del Software
- **Waterfall**: fasi sequenziali e separate; documentazione centrale; cliente solo all'inizio; difficile adattarsi ai cambiamenti
- **Agile**: 4 valori (individui > processi; software funzionante > documentazione; collaborazione > contratti; cambiamento > piano)
- **SCRUM**: framework Agile con Ruoli (Product Owner, Scrum Team, Scrum Master), Sprint (2-4 sett.), Artefatti (Product Backlog, Sprint Backlog, Incremento), Eventi (Planning, Daily, Review, Retrospective)
- **User Story**: formato "Come [utente] voglio [feature] così da [beneficio]"
- **Scrum Master ≠ Project Manager**: guida senza gestire, è un coach/facilitatore
- **Testing**: verificare se il codice produce il comportamento previsto; NON dimostrare la perfezione
- **Testing vs Debugging**: testing = *trovare* bug; debugging = *risolvere* bug
- **Test Stack**: Unit (Small) → Integration (Medium) → System (Large)
- **JUnit**: framework Java per unit test; annotazione `@Test`; metodi `assertEquals`; classi `NomeClasseTest`
- **TDD**: scrivi il test PRIMA del codice; ciclo Red → Green → Refactor; piccoli passi; è anche una pratica di design
- **XP (Extreme Programming)**: metodologia Agile; include TDD, pair programming, refactoring, KISS
- **DevOps**: integra Dev + Ops per rilasci continui e collaborazione tra team
- **CI/CD**: Continuous Integration (ogni commit buildato e testato) → Continuous Delivery (sempre rilasciabile, deploy manuale) → Continuous Deployment (deploy automatico)
- **Race Condition**: `count++` non è atomica; `synchronized` risolve; `ExecutorService` per parallelismo gestito
- **`parallelStream()`**: sfrutta più core automaticamente; il parallelismo esiste ma non è garantito
