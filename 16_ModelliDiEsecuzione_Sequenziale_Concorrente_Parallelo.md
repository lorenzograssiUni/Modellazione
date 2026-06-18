# 16 - Modelli di Esecuzione: Sequenziale, Concorrente, Parallelo

> ⚠️ Nota: questa lezione è stata ricostruita sugli argomenti indicati dal titolo, seguendo lo stile e il contesto del corso. Integrare con le slide originali quando disponibili.

---

## 1. I Tre Modelli di Esecuzione

Finora abbiamo scritto programmi **sequenziali**: un'istruzione dopo l'altra, un passo alla volta. Ma i moderni sistemi software richiedono spesso di gestire più attività contemporaneamente.

| Modello | Descrizione | Esempio |
|---|---|---|
| **Sequenziale** | Un'operazione alla volta, in ordine | Il nostro sistema universitario finora |
| **Concorrente** | Più attività *gestite* nello stesso intervallo di tempo (interleaving) | Un'app che scarica file e aggiorna la UI |
| **Parallelo** | Più operazioni eseguite *fisicamente* nello stesso istante su hardware distinto | Elaborazione su CPU multi-core |

> Come ha sintetizzato Rob Pike (co-creatore di Go):
> **"La concorrenza riguarda la *gestione* di molte cose contemporaneamente. Il parallelismo riguarda il *fare* molte cose contemporaneamente."**

---

## 2. Sequenziale

Il modello più semplice: le istruzioni vengono eseguite una alla volta, nell'ordine in cui sono scritte.

```java
// Esecuzione sequenziale: A poi B poi C
public static void main(String[] args) {
    caricaDatiDaFile();      // A: si aspetta che finisca
    elaboraDati();           // B: inizia solo dopo A
    stampaRisultati();       // C: inizia solo dopo B
}
```

**Vantaggi**: semplice da scrivere, leggere e debuggare. Nessun rischio di conflitti tra operazioni.

**Limite**: se `caricaDatiDaFile()` impiega 10 secondi, il programma è bloccato per 10 secondi. Non può fare nient'altro.

---

## 3. Concorrente

La **concorrenza** è un modello in cui più attività vengono *gestite* nello stesso intervallo di tempo. Non significa che avvengano fisicamente nello stesso istante: il sistema operativo le **intercala** (context switching), dando l'illusione di simultaneità.

> Il giardino è **condiviso**: i thread si alternano nell'usarlo.

```
Thread A: ██░░██░░██░░
Thread B: ░░██░░██░░██
             (stesso core, interleaving)
```

In Java, la concorrenza si realizza con i **Thread**.

### Creare un Thread in Java

**Modo 1: estendere `Thread`**

```java
public class MioThread extends Thread {
    @Override
    public void run() {
        // codice che eseguirà questo thread
        System.out.println("Thread in esecuzione: " + Thread.currentThread().getName());
    }
}

// Nel main:
MioThread t = new MioThread();
t.start();  // NON chiamare run() direttamente!
```

**Modo 2: implementare `Runnable`** (preferibile — segue SRP e DIP)

```java
public class MioTask implements Runnable {
    @Override
    public void run() {
        System.out.println("Task in esecuzione");
    }
}

// Nel main:
Thread t = new Thread(new MioTask());
t.start();

// Oppure con lambda:
Thread t = new Thread(() -> System.out.println("Task con lambda"));
t.start();
```

> ⚠️ Chiamare `run()` direttamente **non** crea un nuovo thread: esegue il metodo nel thread corrente come una normale chiamata. Bisogna usare `.start()`.

---

## 4. Parallelo

Il **parallelismo** è una proprietà dell'*esecuzione*: più operazioni vengono effettivamente eseguite nello stesso istante fisico, su hardware distinto (più core, più CPU).

> Ogni thread ha il **proprio giardino** e lavora indipendentemente.

```
Core 0: Thread A ████████████
Core 1: Thread B ████████████
             (hardware diverso, vera simultaneità)
```

Un programma concorrente ben progettato **può essere eseguito in parallelo** se l'hardware lo permette. La concorrenza nel design è un prerequisito per il parallelismo.

---

## 5. Il Problema della Concorrenza: Race Condition

Quando più thread **condividono** risorse (variabili, oggetti), possono interferire tra loro in modo imprevedibile.

```java
// Variabile condivisa
private int contatore = 0;

// Due thread eseguono questo metodo contemporaneamente
public void incrementa() {
    contatore++;  // NON è atomico! È: leggi, incrementa, scrivi
}
```

**Cosa può succedere:**

```
Thread A legge contatore = 0
Thread B legge contatore = 0     ← interruzione!
Thread A scrive contatore = 1
Thread B scrive contatore = 1    ← il secondo incremento è perso!

Risultato atteso: 2   Risultato ottenuto: 1
```

Questo è una **race condition**: il risultato dipende dall'ordine imprevedibile in cui i thread accedono alla sezione critica.

---

## 6. Soluzioni: Sincronizzazione

### `synchronized`

La parola chiave `synchronized` garantisce che **un solo thread alla volta** possa eseguire un blocco di codice.

```java
// Metodo sincronizzato
public synchronized void incrementa() {
    contatore++;
}

// Oppure: blocco sincronizzato
public void incrementa() {
    synchronized(this) {
        contatore++;
    }
}
```

Java associa ad ogni oggetto un **monitor** (lock intrinseco). `synchronized` acquisisce il lock all'ingresso e lo rilascia all'uscita.

### Variabili Atomiche

Per operazioni semplici su singole variabili, `java.util.concurrent.atomic` offre classi thread-safe senza `synchronized`:

```java
import java.util.concurrent.atomic.AtomicInteger;

private AtomicInteger contatore = new AtomicInteger(0);

public void incrementa() {
    contatore.incrementAndGet();  // atomico per definizione
}
```

---

## 7. Il Deadlock

Il **deadlock** è una situazione in cui due (o più) thread si bloccano a vicenda, ognuno in attesa che l'altro rilasci una risorsa.

```
Thread A ha il lock su Risorsa 1 e aspetta Risorsa 2
Thread B ha il lock su Risorsa 2 e aspetta Risorsa 1
→ Nessuno dei due può procedere. Il programma si blocca per sempre.
```

**Come evitarlo**: acquisire sempre i lock nello stesso ordine, usare timeout, o usare strutture dati thread-safe già pronte (`ConcurrentHashMap`, `BlockingQueue`, ecc.).

---

## 8. Modello di Esecuzione e Progetto RPG

Nel contesto del progetto, la concorrenza non è richiesta come funzionalità obbligatoria. Tuttavia, capire i modelli di esecuzione è utile per:

- Comprendere **perché certi bug sono difficili da riprodurre** (race condition)
- Saper scegliere la struttura dati giusta: `HashMap` non è thread-safe → usare `ConcurrentHashMap` in contesto concorrente
- Dimostrare comprensione avanzata dell'architettura software

---

## 9. Riepilogo Visivo

```
SEQUENZIALE:    A ──► B ──► C
                (un task alla volta, in ordine)

CONCORRENTE:    A ─┐
                B ─┤ (interleaving su 1 core, gestione di più task)
                C ─┘

PARALLELO:      Core 0: A ────────────
                Core 1: B ────────────
                (esecuzione fisicamente simultanea, hardware multi-core)
```

---

## 10. Punti Chiave per l'Esame ✅

- **Sequenziale**: un'istruzione alla volta, in ordine; semplice ma bloccante
- **Concorrente**: più task *gestiti* nello stesso intervallo (interleaving); non necessariamente fisicamente simultanei
- **Parallelo**: più operazioni eseguite *fisicamente* nello stesso istante su hardware distinto (più core)
- **Concorrenza ≠ Parallelismo**: la concorrenza è nel *design*, il parallelismo è nell'*esecuzione*
- **Thread in Java**: `extends Thread` oppure (meglio) `implements Runnable`; si avviano con `.start()`, mai con `.run()` direttamente
- **Lambda con Thread**: `new Thread(() -> { ... }).start()`
- **Race condition**: il risultato dipende dall'ordine imprevedibile dei thread; si verifica quando più thread accedono e modificano dati condivisi
- **Sezione critica**: la parte di codice che accede a risorse condivise
- **`synchronized`**: garantisce l'accesso esclusivo; Java usa i monitor (lock intrinseci) per ogni oggetto
- **Variabili atomiche** (`AtomicInteger`): operazioni thread-safe senza `synchronized`, per casi semplici
- **Deadlock**: due thread si bloccano a vicenda aspettando risorse che l'altro detiene; il programma si blocca per sempre
- **`HashMap` non è thread-safe** → in contesti concorrenti usare `ConcurrentHashMap`
- **Collegamento SOLID**: usare `Runnable` invece di `extends Thread` rispetta SRP e DIP (la classe non è *un* Thread, *ha* un comportamento eseguibile)
