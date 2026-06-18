# 09 - Interfacce, Classi Astratte e Polimorfismo

---

## 1. Recap: Il Pattern Contratto

Le interfacce separano **chi usa** un comportamento da **chi lo implementa**.

```
Interfaccia (contratto)        → public interface Valutatore { int assegnaVoto(Studente s); }
Implementazione (chi FA)       → public class Professore implements Valutatore { ... }
Utilizzatore (chi USA)         → public class Esame { private Valutatore valutatore; ... }
```

> Quando una classe "implementa" un'interfaccia, firma un vero e proprio contratto: promette al compilatore di fornire un'implementazione per tutti i metodi elencati in quell'interfaccia.

---

## 2. Classe `Commissione` — Usare l'Interfaccia

### Versione sbagliata (accoppiata a `Professore`)

```java
// ⚠️ rigido: la Commissione può contenere solo Professori
public class Commissione implements Valutatore {
    private List<Professore> membri;  // ← troppo specifico!
    ...
}
```

### Versione corretta (usa l'interfaccia `Valutatore`)

```java
// ✅ flessibile: la Commissione può contenere qualsiasi Valutatore
public class Commissione implements Valutatore {
    private List<Valutatore> membri;  // ← usa l'interfaccia!

    public Commissione() {
        this.membri = new ArrayList<>();
    }

    public void aggiungiMembro(Valutatore valutatore) {
        membri.add(valutatore);
    }

    @Override
    public int assegnaVoto(Studente studente) {
        int somma = 0;
        for (Valutatore v : membri) {
            int voto = v.assegnaVoto(studente);
            somma += voto;
        }
        int media = somma / membri.size();
        return media;
    }
}
```

> **Lezione chiave:** Cosa abbiamo fatto a fare l'interfaccia `Valutatore` se poi la `Commissione` la ignora usando `Professore`? Usate sempre l'interfaccia, non la classe concreta!

### `List` e `ArrayList`

| | Cos'è | Ruolo |
|---|---|---|
| **`List`** | Interfaccia della libreria standard Java | Dice **cosa** fa una lista (contratto) |
| **`ArrayList`** | Classe concreta che implementa `List` | Dice **come** lo fa |

```java
List<Valutatore> membri = new ArrayList<>();  // tipo: interfaccia, istanza: classe concreta
```

> Esercizio per casa: quante/quali implementazioni comuni ci sono di `List`? (es. `LinkedList`, `Vector`, `Stack`...)

---

## 3. Polimorfismo

> **Polimorfismo** (dal greco: *poli* = molti, *morphè* = forma): lo stesso codice può assumere comportamenti diversi a seconda dell'oggetto su cui viene eseguito.

```java
Valutatore v1 = new Professore(...);
Valutatore v2 = new Commissione(...);
Valutatore v3 = new SistemaAutomatico(...);

// Lo stesso metodo, tre comportamenti diversi:
v1.assegnaVoto(s);  // → comportamento Professore
v2.assegnaVoto(s);  // → comportamento Commissione
v3.assegnaVoto(s);  // → comportamento SistemaAutomatico
```

```java
// Una sola riga di codice, molti comportamenti possibili:
Valutatore v = ...;   // può essere qualsiasi implementazione
v.assegnaVoto(s);    // il comportamento è deciso a runtime
```

> Non ci interessa se è un professore o una commissione: chiamiamo sempre `assegnaVoto`. Implementazioni diverse della stessa interfaccia abilitano il polimorfismo.

### Schema del polimorfismo

```
        Valutatore (interfaccia)
            |
    ________|________
    |        |       |
Professore Commissione SistemaAutomatico
(impl. 1)  (impl. 2)   (impl. 3)
```

---

## 4. Classi Astratte

### Il problema: ha senso istanziare `Persona`?

```java
// Nel nostro sistema universitario NON esiste una "persona generica"
// Esistono solo: Studente e Professore

Persona p = new Persona("Anna", "Rossi");  // ⚠️ ha senso?
```

> Senza `abstract`: `new Persona(...)` crea un oggetto inutile/incoerente rispetto al dominio. Il linguaggio permette errori logici.

### La soluzione: `abstract`

```java
public abstract class Persona {
    private String nome;
    private String cognome;

    public Persona(String nome, String cognome) {
        setNome(nome);
        setCognome(cognome);
    }

    public String getNome() { ... }
    public String getNomeCompleto() { return nome + " " + cognome; }
    // ... altri metodi comuni
}
```

```java
Persona p = new Persona("Anna", "Rossi");  // ❌ ERRORE di compilazione!
Studente s = new Studente("Anna", "Rossi", 1001);  // ✅ OK
Professore pr = new Professore("Luca", "Bianchi", "Prog."); // ✅ OK
```

> Con `abstract`: la classe esiste **per il codice** (le sottoclassi la usano), ma **non per il dominio** (non si può istanziare direttamente).

### Quando usare le classi astratte

- La classe rappresenta un **concetto generale** che non deve essere istanziato direttamente
- Ci sono metodi **comuni** da non duplicare (a differenza di una semplice interfaccia)
- Vuoi **proteggere il modello**: impedire che qualcuno crei un oggetto incoerente col dominio

---

## 5. Interfacce vs Classi Astratte vs Classi Concrete

| | **Interfaccia** | **Classe Astratta** | **Classe Concreta** |
|---|---|---|---|
| **Istanziabile?** | ❌ No | ❌ No | ✅ Sì |
| **Contiene dati?** | No (solo costanti) | ✅ Sì | ✅ Sì |
| **Contiene impl. dei metodi?** | No (solo firme) | Parziale (può avere metodi concreti + astratti) | ✅ Sì, tutti |
| **Ereditarietà multipla?** | ✅ Sì (`implements A, B`) | ❌ No (una sola con `extends`) | ❌ No |
| **Parola chiave** | `interface` / `implements` | `abstract class` / `extends` | `class` |
| **Scopo** | Contratto comportamentale | Base comune non istanziabile | Oggetto concreto |
| **Esempio** | `Valutatore` | `Persona` | `Studente`, `Professore` |

---

## 6. I Tre Ruoli nel Pattern Interfaccia

```java
// 1. INTERFACCIA: il contratto
public interface Valutatore {
    int assegnaVoto(Studente studente);
}

// 2. IMPLEMENTAZIONE: chi FA il lavoro
public class Professore extends Persona implements Valutatore {
    @Override
    public int assegnaVoto(Studente studente) {
        return 28;
    }
}

// 3. UTILIZZATORE: chi USA il comportamento
public class Esame {
    private Valutatore valutatore;

    public Esame(Valutatore valutatore) {
        this.valutatore = valutatore;
    }

    public void svolgiEsame(Studente s) {
        int voto = valutatore.assegnaVoto(s);  // non sa chi è, sa solo che sa valutare
        System.out.println(voto);
    }
}
```

---

## 7. Stato del Sistema Universitario

```
it.unicam.mdp2526.universita
├── Persona.java          (abstract class)
├── Studente.java         (extends Persona)
├── Professore.java       (extends Persona, implements Valutatore)
├── Commissione.java      (implements Valutatore)
├── SistemaAutomatico.java (implements Valutatore)
├── Valutatore.java       (interface)
├── Corso.java
└── Esame.java
```

---

## 8. Riflessione per il Progetto

### Classi astratte nel progetto RPG?

Esempi possibili:
- **`Personaggio`** (abstract) → `Guerriero`, `Mago`, `Ladro` (concrete)
- **`Nemico`** (abstract) → `Goblin`, `Drago`, `Scheletro` (concrete)
- **`Oggetto`** (abstract) → `Arma`, `Pozione`, `Armatura` (concrete)

### Interfacce nel progetto RPG?

Esempi possibili:
- **`Attaccabile`** → qualsiasi entità che può ricevere attacchi
- **`Curabile`** → qualsiasi entità che può essere curata
- **`Utilizzabile`** → qualsiasi oggetto che può essere usato

---

## 9. Note su `super()` e Java 25

```java
// Regola classica (Java < 22): super() DEVE essere la prima riga
public Studente(String nome, String cognome, int matricola) {
    super(nome, cognome);  // ← PRIMA riga obbligatoria
    if (matricola <= 0) { throw new IllegalArgumentException(...); }
    this.matricola = matricola;
}
```

> In Java 25 (la versione usata nel corso) questa regola è stata **allentata**: è possibile eseguire codice prima di `super()`, utile per validare parametri prima di passarli al genitore. Verificate la versione Java del vostro ambiente con `./gradlew build`.

---

## 10. Punti Chiave per l'Esame ✅

- **Polimorfismo** = stesso codice, comportamenti diversi a seconda dell'oggetto; abilitato da interfacce e ereditarietà
- `Valutatore v = new Professore(...)` → la variabile è di tipo interfaccia, l'oggetto è concreto
- Usare **sempre il tipo dell'interfaccia** come tipo della variabile/campo (non la classe concreta), per massimizzare la flessibilità
- `List<Valutatore> membri = new ArrayList<>()` → `List` = interfaccia, `ArrayList` = implementazione concreta
- **Classe astratta** (`abstract class`) = non istanziabile direttamente; esiste per il codice ma non per il dominio
- `abstract` su una classe impedisce `new NomeClasse(...)` → errore di compilazione
- Una classe astratta **può avere** costruttori, dati, metodi concreti E metodi astratti (senza implementazione)
- Differenza: **interfaccia** = solo comportamento (no dati, no impl.); **classe astratta** = base parzialmente implementata con dati
- Nella `Commissione`: usare `List<Valutatore>` non `List<Professore>` → principio di **programmazione verso l'interfaccia**
- I tre ruoli: **Contratto** (interfaccia) / **Implementatore** (`implements`) / **Utilizzatore** (dipende dall'interfaccia)
- Nel progetto RPG: pensare a quali concetti sono astratti (non istanziabili) e quali comportamenti sono interfacce
- `super()` deve essere la prima riga del costruttore figlio (in Java < 22); in Java 25 questo vincolo è allentato
