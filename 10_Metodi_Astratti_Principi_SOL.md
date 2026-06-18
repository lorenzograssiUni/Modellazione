# 10 - Metodi Astratti e Principi SOLID (S, O, L)

---

## 1. Metodi Astratti

### Il problema: due metodi che esprimono la stessa idea

```java
// In Studente:
public void saluta() {
    System.out.println("Ciao, sono lo studente " + getNomeCompleto());
}

// In Professore:
public void presenta() {
    System.out.println("Sono il professore " + getNomeCompleto() + " del settore " + settore);
}
```

Entrambi i metodi esprimono la stessa idea: *ogni persona del sistema si presenta*. Ma si presentano in modo diverso → non possiamo scrivere un'unica versione valida per tutti.

### La soluzione: metodo `abstract` nella superclasse

```java
public abstract class Persona {
    private String nome;
    private String cognome;

    public Persona(String nome, String cognome) {
        this.nome = nome;
        this.cognome = cognome;
    }

    public String getNomeCompleto() { return nome + " " + cognome; }

    // Metodo astratto: esiste, ma non ha implementazione
    // Ogni sottoclasse è OBBLIGATA a fornirla
    public abstract void presentati();
}
```

### Implementazione nelle sottoclassi

```java
public class Studente extends Persona {
    @Override
    public void presentati() {
        System.out.println("Ciao, sono lo studente " + getNomeCompleto());
    }
}

public class Professore extends Persona implements Valutatore {
    @Override
    public void presentati() {
        System.out.println("Sono il professore " + getNomeCompleto() + " del settore " + settore);
    }
}
```

### Polimorfismo con i metodi astratti

```java
// Grazie al metodo astratto, possiamo usare il polimorfismo:
Persona p1 = new Studente("Anna", "Rossi", 1001);
Persona p2 = new Professore("Luca", "Bianchi", "Informatica");
p1.presentati();  // → comportamento di Studente
p2.presentati();  // → comportamento di Professore
```

### Regole dei metodi astratti

- Un metodo `abstract` **non ha corpo** (nessuna `{}`)
- Può essere dichiarato **solo in una classe `abstract`**
- Ogni sottoclasse **concreta** (non astratta) deve implementarlo con `@Override`
- Se una sottoclasse non lo implementa, **anche lei deve essere `abstract`**

---

## 2. Contesto Storico: i Principi SOLID

### Il problema negli anni '90: *Software Rot* (Software Marcito)

- **Rigidità**: ogni modifica ne richiede altre cento
- **Fragilità**: modificare una parte ne rompe un'altra non correlata
- **Immobilità**: non si può riutilizzare pezzi di codice perché sono troppo intrecciati

### Robert C. Martin — "Uncle Bob"

Ingegnere informatico e autore statunitense, tra i principali esperti di sviluppo software.
- Co-autore del **Manifesto Agile**
- Promotore dei principi **SOLID**
- Autore di *Clean Code* e *Clean Architecture*

Il soprannome "Uncle Bob" nacque negli anni '90 quasi per scherzo tra colleghi, e lui ha poi trasformato in un brand professionale.

---

## 3. S — Single Responsibility Principle (SRP)

> **Ogni classe dovrebbe avere una ed una sola responsabilità, interamente incapsulata al suo interno.**  
> Una classe deve avere un solo motivo per cambiare.

Autore: **Robert C. Martin** — *Agile Software Development, Principles, Patterns, and Practices* (2002)

### Esempio: classe `Studente` rispetta SRP

```java
public class Studente extends Persona {
    private final int matricola;

    public Studente(String nome, String cognome, int matricola) {
        super(nome, cognome);
        this.matricola = matricola;
    }

    public int getMatricola() { return matricola; }

    @Override
    public void presentati() {
        System.out.println("Ciao, sono lo studente " + getNomeCompleto());
    }
}
```

→ Se cambia qualcosa **dello studente**, cambio questa classe.  
→ Se cambia qualcosa di `Esame` o `Corso` → **NON** cambio questa classe.

### Classe `Libretto` e violazioni SRP

```java
// Composizione forte: lo studente viene creato subito con il suo libretto
public class Studente extends Persona {
    private final Libretto libretto;
    public Studente(...) {
        ...
        this.libretto = new Libretto(this);  // il libretto non esiste senza studente
    }
}
```

| Responsabilità di `Libretto` | SRP rispettato? |
|---|---|
| Associare il libretto a uno studente | ✅ Sì |
| Registrare gli esami superati con il voto | ✅ Sì |
| Conservare l'elenco degli esami | ✅ Sì |
| Calcolare la media dei voti | ✅ Sì |
| Decidere se lo studente può laurearsi (`puoLaurearsi()`) | ⚠️ **No** — è una **regola di carriera**, non una responsabilità del libretto |
| Stampare gli esami sostenuti (`stampaEsamiSuperati()`) | ⚠️ Borderline — è **responsabilità di presentazione**; va bene finché è semplice, ma se cresce va separata |

> **Se la regola di laurea cambia, devi cambiare `Libretto`. Ma `Libretto` non dovrebbe sapere nulla di politiche di laurea!**

### Esercizio per casa

- Risolvere le violazioni SRP di `Libretto` (`puoLaurearsi()` → spostare in una classe dedicata, es. `RegolaLaurea`)
- Analizzare la classe `Esame` per trovare eventuali ulteriori violazioni

---

## 4. O — Open/Closed Principle (OCP)

> **Il software deve essere aperto all'estensione (nuove funzionalità) ma chiuso alla modifica del codice esistente.**

Autore: **Bertrand Meyer** — *Object-Oriented Software Construction* (1988)

### Esempio: il problema di `Esame` legato a `Professore`

```java
// Versione SBAGLIATA: aggiungere un nuovo valutatore richiede di modificare Esame
public void sostieniEsame(Studente studente) {
    int voto;
    if (tipoValutazione.equals("professore")) { ... }
    else if (tipoValutazione.equals("commissione")) { ... }
    else if (tipoValutazione.equals("automatico")) { ... }  // aggiungere qui ogni volta
    else { ... }
}
```

### Esempio: versione corretta con interfaccia `Valutatore`

```java
// Versione CORRETTA: Esame è chiuso alla modifica, aperto all'estensione
public class Esame {
    private Valutatore valutatore;  // dipende dall'interfaccia, non dalla classe concreta

    public void sostieniEsame(Studente studente) {
        int voto = valutatore.assegnaVoto(studente);  // non cambia mai
        String votoStr = (voto == 31) ? "30 e lode" : String.valueOf(voto);
    }
}

// Per aggiungere un nuovo modo di valutare: SOLO una nuova classe, Esame non si tocca!
public class SistemaAutomatico implements Valutatore {
    @Override
    public int assegnaVoto(Studente studente) {
        return ...;  // implementazione diversa
    }
}
```

> **OCP in pratica:** aperto all'estensione (aggiungo `SistemaAutomatico`) — chiuso alla modifica (non tocco `Esame`).

---

## 5. L — Liskov Substitution Principle (LSP)

> **Gli oggetti dovrebbero poter essere sostituiti con dei loro sottotipi, senza alterare il comportamento del programma che li utilizza.**

Autore: **Barbara Liskov** — conferenza 1987; articolo scientifico con Jeannette Wing, 1994. Premio Turing 2008.

### Polimorfismo e LSP sono collegati

```java
// Ereditarietà:
Persona p = new Studente("Anna", "Rossi", 1001);  // Studente sostituisce Persona
p.getNomeCompleto();  // funziona! Studente è un tipo più specifico di Persona

// Interfacce:
Valutatore v1 = new Professore(...);
Valutatore v2 = new Commissione(...);
v1.assegnaVoto(s);  // stessa chiamata, comportamento diverso
v2.assegnaVoto(s);  // → polimorfismo!
```

### Il contratto implicito

```java
public interface Valutatore {
    int assegnaVoto(Studente studente);
}
```

Quando uso `Valutatore`, mi aspetto:
- Un numero tra 18 e 30
- Oppure 31 = 30 e lode
- **NON** valori negativi o numeri assurdi
- Comportamento coerente e sensato

Queste sono le **aspettative implicite** — il "contratto implicito" del tipo.

### Violazione di LSP

```java
// ❌ Viola LSP: restituisce -10, rompendo le aspettative
public class ValutatoreStrano implements Valutatore {
    @Override
    public int assegnaVoto(Studente studente) {
        return -10;  // voto impossibile!
    }
}

// Conseguenza: il client non può fidarsi del tipo Valutatore
int voto = v.assegnaVoto(studente);
if (voto < 18 || voto > 31) {  // se devo fare questo controllo ovunque, LSP è violato!
    throw new RuntimeException("Valutatore non valido");
}
```

### Come prevenire violazioni: rendere esplicite le aspettative

```java
public interface Valutatore {
    /**
     * Restituisce un voto tra 18 e 30,
     * oppure 31 per la lode.
     */
    int assegnaVoto(Studente studente);

    // Metodo static per validare il voto
    static int validaVoto(int voto) {
        if (voto < 18 || voto > 31) {
            throw new IllegalStateException("Voto non valido");
        }
        return voto;
    }
}

// Ogni implementazione usa il validatore:
public class Professore implements Valutatore {
    @Override
    public int assegnaVoto(Studente studente) {
        return Valutatore.validaVoto(28);  // garantisce il rispetto del contratto
    }
}
```

### Non tutto ciò che sembra "brutto" viola LSP

```java
// Restituisce sempre 18: implementazione povera, ma...
public class ValutatoreMinimale implements Valutatore {
    @Override
    public int assegnaVoto(Studente studente) {
        return 18;  // voto valido! Non viola LSP
    }
}
```

> LSP non dice che tutte le implementazioni devono essere intelligenti o utili allo stesso modo. Dice che devono restare **compatibili con il comportamento atteso**.

### Regole LSP

- **Definire chiaramente il comportamento atteso** → specificare cosa ci si aspetta da un tipo, non solo quali metodi ha
- **Rispettarlo in tutte le implementazioni** → ogni classe deve comportarsi in modo coerente con le aspettative
- **Non introdurre comportamenti "speciali"** → evitare logiche inattese che costringono il codice client a trattamenti particolari
- **Non rompere i metodi ereditati** → le sottoclassi non devono alterare il comportamento previsto dei metodi della classe base

---

## 6. Riepilogo dei Principi SOLID visti

| Principio | Sigla | Autore | In breve | Violazione tipica |
|---|---|---|---|---|
| Single Responsibility | **S** | R. C. Martin (2002) | Una classe = una responsabilità | `Libretto.puoLaurearsi()` — regola di carriera nel libretto |
| Open/Closed | **O** | B. Meyer (1988) | Aperto all'estensione, chiuso alla modifica | `if (tipo == "professore") ... else if (tipo == "commissione")` in `Esame` |
| Liskov Substitution | **L** | B. Liskov (1987) | I sottotipi devono sostituire il tipo base senza sorprese | `ValutatoreStrano.assegnaVoto()` restituisce -10 |

> I principi **I** (Interface Segregation) e **D** (Dependency Inversion) verranno trattati nelle prossime lezioni.

---

## 7. Stato del Sistema Universitario (aggiornato)

```
it.unicam.mdp2526.universita
├── Persona.java          (abstract class — con metodo abstract presentati())
├── Studente.java         (extends Persona — libretto a composizione)
├── Professore.java       (extends Persona, implements Valutatore)
├── Commissione.java      (implements Valutatore — List<Valutatore>)
├── SistemaAutomatico.java (implements Valutatore)
├── Valutatore.java       (interface — con static validaVoto())
├── Libretto.java         (registra esami, media — SRP parzialmente violato)
├── Corso.java
└── Esame.java            (usa Valutatore via OCP)
```

---

## 8. Punti Chiave per l'Esame ✅

- **Metodo `abstract`** = metodo senza corpo, dichiarato solo nella superclasse astratta; ogni sottoclasse concreta deve implementarlo con `@Override`
- Una classe `abstract` può avere sia metodi concreti (con implementazione) che astratti (senza)
- `public abstract void presentati();` → Persona definisce il "cosa", Studente/Professore definiscono il "come"
- **SRP (S):** ogni classe ha un solo motivo per cambiare; `Libretto.puoLaurearsi()` viola SRP perché applica regole di carriera
- **OCP (O):** aggiungere funzionalità = nuova classe, non modificare codice esistente; `Esame + Valutatore` è l'esempio perfetto
- **LSP (L):** ogni implementazione di un'interfaccia deve rispettare il "contratto implicito" del tipo; `ValutatoreStrano` con -10 viola LSP
- Un tipo non è solo una firma di metodi: è anche un insieme di **aspettative sul comportamento**
- `Valutatore.validaVoto()` come metodo `static` nell'interfaccia → modo per rendere esplicito il contratto implicito
- `ValutatoreMinimale` (sempre 18) **non viola** LSP: implementazione povera, ma comportamento coerente
- "Software Rot": rigidità, fragilità, immobilità — i sintomi del codice mal progettato che i principi SOLID risolvono
- I principi SOLID completano il quadro OOP: incapsulamento, ereditarietà, polimorfismo, interfacce + principi di progettazione
- **I** (Interface Segregation) e **D** (Dependency Inversion) → prossime lezioni
