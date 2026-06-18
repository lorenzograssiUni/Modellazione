# 15 - Stream

---

## 1. Il problema: come usiamo le collection?

Una volta costruita una collection (`List`, `Set`, `Map`), le operazioni tipiche sono:

- Cercare qualcosa
- **Filtrare** elementi che soddisfano una condizione
- **Contare** o **sommare**
- **Trasformare** i dati in un'altra forma

Tutte queste operazioni richiedono di iterare sugli elementi. Il modo classico è il ciclo `for`, ma Java offre un'alternativa più espressiva: le **Stream API**.

---

## 2. Cos'è uno Stream?

> Uno **Stream** è un flusso di dati su cui possiamo applicare operazioni in modo **dichiarativo** (stile programmazione funzionale).

**Non è una struttura dati**: non memorizza elementi. È un'interfaccia che permette di elaborare sequenze di dati descrivendo *cosa* fare, non *come* farlo passo per passo.

```java
// Stile imperativo (for loop)
@Override
public int calcolaMedia() {
    if (voti.isEmpty()) return 0;
    int somma = 0;
    for (int voto : voti.values()) {
        somma += (voto == 31) ? 30 : voto;
    }
    return somma / voti.size();
}

// Stile dichiarativo (Stream)
@Override
public int calcolaMedia() {
    return (int) voti.values().stream()
        .mapToInt(voto -> voto == 31 ? 30 : voto)
        .average()
        .orElse(0);
}
```

Lettura della versione Stream: *"Prendi i voti, trasformali gestendo la lode (31 → 30), calcola la media, e restituisci il risultato come intero. Se non ci sono voti, restituisci 0."*

---

## 3. Lambda Expression

Una **lambda** è una funzione scritta "al volo" in forma compatta.

```java
voto -> voto == 31 ? 30 : voto
```

- **Input** → `voto` (parametro)
- **Output** → il valore trasformato (espressione)
- Forma generale: `(parametri) -> espressione`

Le lambda rappresentano l'implementazione di un'**interfaccia funzionale** (interfaccia con un solo metodo astratto) e possono essere trattate come oggetti. Questa notazione richiama la **programmazione funzionale**, dove si descrive una trasformazione da input a output.

### Collegamento: Programmazione Funzionale

La programmazione funzionale è un paradigma in cui il programma è composto da funzioni matematiche. Caratteristiche chiave:
- Nessuno stato mutabile
- Nessun effetto collaterale (idealmente)
- Linguaggi: Haskell, Lisp, OCaml

> Approfondimento a [ST1420] - Paradigmi Avanzati di Programmazione.

---

## 4. Operazioni Intermedie e Terminali

Uno stream si costruisce come una **pipeline**:
1. Sorgente (`.stream()` su una collection)
2. Zero o più **operazioni intermedie** (trasformano lo stream)
3. Una **operazione terminale** (chiude lo stream e produce un risultato)

### Operazioni Intermedie (lazy — non fanno nulla finché non c'è un'operazione terminale)

| Operazione | Descrizione |
|---|---|
| `filter(Predicate)` | Esclude gli elementi che non soddisfano una condizione |
| `map(Function)` | Trasforma ogni elemento in qualcos'altro |
| `mapToInt(Function)` | Come `map`, ma produce un `IntStream` |
| `sorted()` | Ordina gli elementi |
| `distinct()` | Rimuove i duplicati (usa `equals()`) |
| `limit(n)` | Prende solo i primi `n` elementi |

### Operazioni Terminali (chiudono lo stream, producono un risultato)

| Operazione | Scopo |
|---|---|
| `collect(Collectors.toList())` | Trasforma lo stream in una `List` (la più usata) |
| `collect(Collectors.toSet())` | Trasforma lo stream in un `Set` (rimuove duplicati) |
| `forEach(Consumer)` | Esegue un'azione per ogni elemento (es. stampa) |
| `count()` | Conta quanti elementi rimangono dopo i filtri |
| `anyMatch(Predicate)` | Controlla se almeno un elemento soddisfa la condizione |
| `findFirst()` | Recupera il primo elemento della sequenza |
| `average()` | Calcola la media (su `IntStream`) |
| `min()` / `max()` | Trova il valore minimo o massimo |

> **Senza un'operazione terminale, il codice non fa nulla.** Le operazioni intermedie sono *lazy*: vengono eseguite solo quando la terminale lo richiede.

---

## 5. Perché usare gli Stream?

1. **Leggibilità**: il codice sembra quasi una frase in inglese → *Code as Prose* (Clean Code)
2. **Meno bug**: si evitano cicli `for` infiniti o variabili contatore gestite male

> Se l'operazione è semplicissima, un vecchio caro `for` è ancora validissimo. Ricordare **KISS**!

---

## 6. Pipeline analizzata passo per passo

```java
return (int) voti.values()   // Collection<Integer>
    .stream()                // trasforma in Stream<Integer>
    .mapToInt(voto -> voto == 31 ? 30 : voto)  // IntStream: gestisce la lode
    .average()               // OptionalDouble: calcola la media
    .orElse(0);              // se vuoto → 0; (int) → cast finale
```

| Passo | Tipo | Ruolo |
|---|---|---|
| `.stream()` | — | Crea lo stream dalla collection |
| `.mapToInt(...)` | Intermedia | Trasforma ogni elemento in `int` |
| `.average()` | Terminale | Calcola la media, restituisce `OptionalDouble` |
| `.orElse(0)` | Su `Optional` | Gestisce il caso in cui non ci siano dati |
| `(int)` | Cast | `average()` restituisce `double`, `calcolaMedia()` vuole `int` |

---

## 7. Catene Lunghe e Clean Code

Le Stream possono diventare catene molto lunghe. Quando una pipeline fa troppe cose, si rischia di violare il principio **Single Responsibility**.

### ❌ Versione con lambda inline (difficile da leggere e testare)

```java
public static void stampaStudentiConMediaAlta(List<Studente> studenti) {
    studenti.stream()
        .filter(studente -> studente.getLibretto().calcolaMedia() > 28)
        .sorted((s1, s2) -> Integer.compare(
            s2.getLibretto().calcolaMedia(),
            s1.getLibretto().calcolaMedia()))
        .map(studente -> studente.getMatricola() + " - " +
            studente.getNomeCompleto() + " | media: " +
            studente.getLibretto().calcolaMedia())
        .forEach(System.out::println);
}
```

### ✅ Versione Clean Code: estrarre le lambda in metodi privati descrittivi

```java
// Classe ReportStudenti
public void stampaStudentiConMediaAlta(List<Studente> studenti) {
    studenti.stream()
        .filter(this::haMediaAlta)
        .sorted(this::confrontaPerMediaDecrescente)
        .map(this::formattaStudente)
        .forEach(System.out::println);
}

// Metodi privati
private boolean haMediaAlta(Studente studente) {
    return studente.getLibretto().calcolaMedia() > 28;
}

private int confrontaPerMediaDecrescente(Studente s1, Studente s2) {
    return Integer.compare(
        s2.getLibretto().calcolaMedia(),
        s1.getLibretto().calcolaMedia());
}

private String formattaStudente(Studente studente) {
    return studente.getMatricola() + " - " +
        studente.getNomeCompleto() + " | media: " +
        studente.getLibretto().calcolaMedia();
}
```

Lettura dello stream: *"Prendi gli studenti, tieni quelli con media alta, ordina per media decrescente, formattali e stampali."* → **lo stream sembra una frase leggibile, non un intero capitolo di un libro.**

> **Regola pratica**: se la pipeline contiene troppe regole con lambda lunghe, estrarle in metodi con nomi chiari. Si separa il *"cosa"* dal *"come"*.

---

## 8. `Optional` — gestire l'assenza di un valore

`average()` restituisce un `OptionalDouble` perché se lo stream è vuoto, non esiste una media. `Optional` è un contenitore che può contenere o meno un valore.

```java
.average()           // OptionalDouble (potrebbe essere vuoto)
.orElse(0)           // se vuoto → usa 0 come default
```

Pattern comune con `Optional`:
- `.orElse(default)` → usa un valore di default
- `.orElseThrow()` → lancia un'eccezione se vuoto
- `.isPresent()` → controlla se c'è un valore
- `.ifPresent(consumer)` → esegui qualcosa solo se il valore esiste

---

## 9. Method Reference (`this::metodo`)

La sintassi `this::haMediaAlta` è una **method reference**: un modo compatto per passare un metodo esistente come lambda.

```java
// Lambda esplicita
.filter(studente -> studente.getLibretto().calcolaMedia() > 28)

// Method reference equivalente
.filter(this::haMediaAlta)
```

Tipi di method reference:
- `Classe::metodoStatico`
- `oggetto::metodoIstanza`
- `this::metodo`
- `Classe::new` (constructor reference)

---

## 10. Riflessione per casa 💭

> Dovendo elaborare dei dati, quali differenze ci sono tra farlo direttamente in Java con le Stream API e farlo a livello di database?

**Spunto**: le Stream operano in memoria (JVM), il database usa SQL con ottimizzazioni sul disco e sugli indici. Per grandi quantità di dati, spostare il filtraggio a livello di query SQL è più efficiente che caricare tutto in memoria e poi filtrare con gli Stream.

---

## 11. Punti Chiave per l'Esame ✅

- **Stream** non è una struttura dati: è un flusso di dati su cui applicare operazioni, non memorizza elementi
- **Dichiarativo vs Imperativo**: Stream = descrivi *cosa* fare; `for` = descrivi *come* farlo passo per passo
- **Lambda** `(parametri) -> espressione`: funzione anonima "al volo", implementa un'interfaccia funzionale (1 solo metodo astratto)
- **Operazioni intermedie** (`filter`, `map`, `sorted`, `distinct`, `limit`) sono **lazy**: non eseguono nulla senza un'operazione terminale
- **Operazioni terminali** (`collect`, `forEach`, `count`, `anyMatch`, `average`, `findFirst`) **chiudono** lo stream e producono il risultato
- **Senza operazione terminale → il codice non fa nulla**
- **`mapToInt`** converte in `IntStream` per operazioni numeriche (`.average()`, `.sum()`, `.min()`, `.max()`)
- **`Optional`**: contenitore che gestisce l'assenza di un valore; `.orElse(default)` per fornire un fallback
- **Catene lunghe → SRP**: se la pipeline fa troppe cose, estrarre le lambda in metodi privati con nomi descrittivi
- **Method reference** `this::metodo` o `Classe::metodo`: forma compatta per passare un metodo come lambda
- **`System.out::println`** è una method reference usabile in `forEach`
- **Quando usare Stream vs `for`**: Stream per leggibilità e operazioni complesse su collection; `for` per operazioni semplici (KISS)
- **Collegamento con SOLID**: catene Stream troppo lunghe → violazione SRP → estrarre in classe dedicata (es. `ReportStudenti`)
