# 11 - I, D (SOLID completo), Package e Clean Code

---

## 1. SOLID: I — Interface Segregation Principle (ISP)

> **È meglio avere interfacce piccole e mirate, piuttosto che un'unica interfaccia troppo grande. Nessuna classe dovrebbe essere costretta a implementare metodi che non le servono.**

Autore: **Robert C. Martin** — formulato mentre lavorava come consulente per la Xerox. L'azienda aveva problemi con un software che costringeva ogni modulo a ricompilarsi anche se le modifiche riguardavano funzioni che non utilizzava.

### `Valutatore` rispetta ISP ✅

```java
public interface Valutatore {
    int assegnaVoto(Studente studente);
}
```

È una buona interfaccia perché:
- è piccola
- esprime una sola capacità
- chi la implementa deve fare una cosa chiara: valutare

### `GestoreUniversitario` viola ISP ❌

```java
// Interfaccia TROPPO GRANDE: viola ISP
public interface GestoreUniversitario {
    int calcolaMedia();
    void stampaEsamiSuperati(Studente studente);
    int assegnaVoto(Studente studente);
    void iscriviStudente(Studente studente, Corso corso);
    void approvaPianoDiStudi(Studente studente);
}
```

Una classe come `Libretto` che implementa `GestoreUniversitario` è costretta a definire un'implementazione per ogni metodo, anche quelli che non le appartengono.

### Soluzione: dividere in interfacce piccole e mirate

```java
public interface CalcolatoreMedia {
    int calcolaMedia();
}

public interface VisualizzatoreEsami {
    void stampaEsamiSuperati();
}

public interface Valutatore {
    int assegnaVoto(Studente studente);
}

public interface GestoreIscrizioni {
    void iscriviStudente(Studente studente, Corso corso);
}

public interface ApprovatorePianoDiStudi {
    void approvaPianoDiStudi(Studente studente);
}
```

Così facendo ogni classe implementa **solo ciò che usa davvero**:

```java
// Libretto implementa solo le interfacce di sua competenza
public class Libretto implements CalcolatoreMedia, VisualizzatoreEsami {
    // non è costretto a implementare Valutatore o GestoreIscrizioni
}
```

### ISP vale anche per le classi astratte

```java
// VIOLAZIONE ISP nelle classi astratte:
public abstract class Persona {
    private String nome;
    private String cognome;
    private final int matricola;  // ← i Professori non hanno matricola!
    ...
}
```

Una classe astratta che cerca di fare troppo "trascina dentro" roba inutile. Deve contenere **solo ciò che è davvero comune a tutte le sottoclassi**.

---

## 2. SOLID: D — Dependency Inversion Principle (DIP)

> **Bisogna dipendere dalle astrazioni, non dalle implementazioni concrete.**

Autore: **Robert C. Martin** — descritto per la prima volta in un articolo sulla rivista *C++ Report* nel 1996, poi nel libro del 2002.

### Il problema: `Esame` legato a `Professore`

```java
// SBAGLIATO: Esame dipende dal concreto
public class Esame {
    private String nome;
    private Professore professore;  // ← dipende dalla classe concreta!
}
```

Significa: `Esame` lavora **solo** con `Professore`. Il codice è rigido e difficile da estendere.

### La soluzione: dipendere dall'astrazione `Valutatore`

```java
// CORRETTO: Esame dipende dall'astrazione
public class Esame {
    private String nome;
    private Valutatore valutatore;  // ← dipende dall'interfaccia!
}

public class Professore extends Persona implements Valutatore {
    @Override
    public int assegnaVoto(Studente studente) { return 28; }
}
```

Schema delle dipendenze:

```
Esame → Valutatore ← Professore
         (interfaccia)
```

Sia `Esame` che `Professore` dipendono dall'astrazione, **non tra di loro**.

### Due tipi di dipendenza da `Valutatore`

| Chi | Tipo di dipendenza | Significato |
|---|---|---|
| `Esame` | Dipendenza **di utilizzo** | Usa `Valutatore`, chiama i suoi metodi |
| `Professore` | Dipendenza **di implementazione** | Si adatta all'astrazione, la realizza |

> "Dipendere dall'astrazione è come dire: *ci basta qualcuno che sappia fare questa cosa*."

Anche con `Commissione` molti avevano sbagliato legandola a `Professore`:
```java
// SBAGLIATO:
private List<Professore> membri;

// CORRETTO:
private List<Valutatore> membri;  // aperto al futuro
```

---

## 3. Riassunto SOLID

| Principio | Sigla | Autore | In breve | Esempio nel progetto |
|---|---|---|---|---|
| Single Responsibility | **S** | R. C. Martin (2002) | Una classe = una responsabilità | `Libretto.puoLaurearsi()` → spostare in `RegolaLaurea` |
| Open/Closed | **O** | B. Meyer (1988) | Aperto all'estensione, chiuso alla modifica | Aggiungere `SistemaAutomatico` senza toccare `Esame` |
| Liskov Substitution | **L** | B. Liskov (1987) | Sottotipi sostituibili senza sorprese | `ValutatoreStrano(-10)` viola LSP; `ValidaVoto()` lo previene |
| Interface Segregation | **I** | R. C. Martin (~1994) | Interfacce piccole e mirate | Dividere `GestoreUniversitario` in 5 interfacce specifiche |
| Dependency Inversion | **D** | R. C. Martin (1996) | Dipendere dalle astrazioni | `Esame` dipende da `Valutatore`, non da `Professore` |

> I principi SOLID **non sono regole rigide**, sono linee guida per progettare codice più chiaro, flessibile e robusto.  
> **Nel progetto d'esame sono obbligatori e su di essi verrete valutati.**

### Domande da porsi prima di consegnare

1. Questo codice è facile da cambiare o mi costringe a modificarne troppo?
2. Una persona che non ne sa nulla sarebbe in grado di prendere il progetto, capirlo ed estenderlo?

---

## 4. Package

Un **package** è una cartella logica di classi che serve per organizzare il codice quando cresce.

```java
package org.unicam.mdp;

public class Studente {
    String nome;
    int matricola;
    void saluta() { System.out.println("Ciao"); }
}
```

I package corrispondono a cartelle nel filesystem:
```
├── org/
│   └── unicam/
│       └── mdp/
│           ├── Studente.java
│           ├── Esame.java
│           └── Professore.java
```

### Package del progetto d'esame

```
it.unicam.cs.mpgc.rpg.<matricola>
```

*(vedi specifica completa su elearning.unicam.it)*

### Organizzazione del Sistema Universitario per responsabilità

```
it.unicam.mdp2526.universita
├── model/                  ← concetti del dominio
│   ├── Corso.java
│   ├── Esame.java
│   ├── Libretto.java
│   ├── Professore.java
│   └── Studente.java
├── interfaces/             ← contratti (interfacce)
│   ├── ApprovatorePianoDiStudi.java
│   ├── CalcolatoreMedia.java
│   ├── GestoreIscrizioni.java
│   ├── GestoreUniversitario.java  ← da dividere/rimuovere
│   ├── Valutatore.java
│   └── VisualizzatoreEsami.java
└── Main.java               ← avvio del sistema
```

---

## 5. Clean Code

> SOLID ci aiuta a **progettare** sistemi che resistono al cambiamento.  
> Clean Code ci aiuta a rendere il codice **leggibile**.

Autore di riferimento: **Robert C. Martin** — *Clean Code* (2008)

### Principio 1: Nomi Significativi

I nomi devono essere **significativi**: devono spiegare il significato, evitare abbreviazioni e ambiguità.

```java
// ❌ Ambiguo
int v = valutatore.assegnaVoto(s);
if (v >= 18) {
    s.getLibretto().registraEsameSuperato(this, v);
}
// Cos'è v? volume? vittorie? velocità?

// ✅ Chiaro
int voto = valutatore.assegnaVoto(studente);
if (voto >= 18) {
    studente.getLibretto().registraEsameSuperato(this, voto);
}
```

### Principio 2: Small Functions (Funzioni Piccole)

Le funzioni dovrebbero essere **piccole**, svolgere un'**unica funzione** e avere un numero ridotto di argomenti (tra zero e due).

```java
// ❌ Versione lunga: fa troppe cose
public void sostieniEsame(Studente studente) {
    int voto = valutatore.assegnaVoto(studente);
    String votoString = (voto == 31) ? "30 e lode" : String.valueOf(voto);
    System.out.println("Esame di " + nome);
    System.out.println("Studente: " + studente.getNomeCompleto());
    System.out.println("Voto assegnato: " + votoString);
    if ((voto >= 18 && voto <= 30) || voto == 31) {
        studente.getLibretto().registraEsameSuperato(this, voto);
        System.out.println("Esame registrato nel libretto.");
    } else {
        System.out.println("Esame non superato, nessuna registrazione nel libretto.");
    }
}
// Fa: calcola voto, formatta voto, stampa, decide superamento, aggiorna libretto
```

```java
// ✅ Versione Clean Code: ogni metodo fa una cosa sola
public void sostieniEsame(Studente studente) {
    int voto = valutatore.assegnaVoto(studente);
    stampaEsito(studente, voto);
    if (esameSuperato(voto)) {
        registraEsito(studente, voto);
        stampaEsameRegistrato();
    } else {
        stampaEsameNonSuperato();
    }
}

private void stampaEsito(Studente studente, int voto) {
    System.out.println("Esame di " + nome);
    System.out.println("Studente: " + studente.getNomeCompleto());
    System.out.println("Voto assegnato: " + formattaVoto(voto));
}

private String formattaVoto(int voto) {
    return (voto == 31) ? "30 e lode" : String.valueOf(voto);
}

private boolean esameSuperato(int voto) {
    return voto >= 18 || voto == 31;
}

private void registraEsito(Studente studente, int voto) {
    studente.getLibretto().registraEsameSuperato(this, voto);
}

private void stampaEsameRegistrato() {
    System.out.println("Esame registrato nel libretto.");
}

private void stampaEsameNonSuperato() {
    System.out.println("Esame non superato, nessuna registrazione nel libretto.");
}
```

**Vantaggi della versione Clean Code:**
- Il metodo principale è più leggibile
- I nomi spiegano il comportamento
- Le regole sono isolate
- I dettagli sono nascosti

### Principio 3: Code as Prose (Il codice come prosa)

Un buon codice si può leggere come una frase:

```java
// Si legge quasi in italiano:
if (esameSuperato(voto)) {
    registraEsito(studente, voto);
    stampaEsameRegistrato();
} else {
    stampaEsameNonSuperato();
}
// "Se l'esame è superato: registra l'esito e stampa 'registrato'; altrimenti stampa 'non superato'"
```

### SOLID vs Clean Code: differenza chiave

| | SOLID | Clean Code |
|---|---|---|
| **Obiettivo** | Progettare sistemi che resistono al cambiamento | Rendere il codice leggibile |
| **Focus** | Architettura, struttura delle classi | Esperienza dello sviluppatore |
| **Risultato** | Codice estendibile e manutenibile | Codice comprensibile e pulito |

---

## 6. Punti Chiave per l'Esame ✅

- **ISP (I):** interfacce piccole e mirate; `GestoreUniversitario` con 5 metodi viola ISP → dividere in `CalcolatoreMedia`, `VisualizzatoreEsami`, `Valutatore`, `GestoreIscrizioni`, `ApprovatorePianoDiStudi`
- ISP vale anche per classi astratte: `Persona` con `matricola` obbliga `Professore` ad avere un campo che non gli appartiene
- **DIP (D):** dipendere dall'interfaccia, non dalla classe concreta; `Esame → Valutatore ← Professore` è il pattern corretto; `Esame → Professore` è sbagliato
- Due tipi di dipendenza: **di utilizzo** (usa il metodo) vs **di implementazione** (fornisce il metodo)
- `Commissione` deve avere `List<Valutatore>`, non `List<Professore>` — applicazione di DIP
- **SOLID completo:** S (responsabilità) + O (estensione) + L (sostituzione) + I (segregazione) + D (inversione) — acronimo che forma la parola SOLID
- Prima di consegnare: "Qualcuno che non lo conosce, potrebbe capirlo ed estenderlo?"
- **Package:** cartella logica di classi; il package del progetto è `it.unicam.cs.mpgc.rpg.<matricola>`
- Struttura consigliata: `model/` per le classi di dominio, `interfaces/` per i contratti, `Main.java` alla radice
- **Clean Code — Nomi significativi:** evitare `v`, `s`, `p`; usare `voto`, `studente`, `professore`
- **Clean Code — Small Functions:** ogni metodo fa una sola cosa; 0-2 argomenti idealmente
- **Clean Code — Code as Prose:** il codice deve potersi leggere come una frase in italiano
- Scrivere codice che funziona **non basta**: deve essere leggibile, comprensibile e mantenibile
