# 08 - OOP: Ereditarietà, Protected, Interfacce

---

## 1. Recap: Ereditarietà con `extends`

L'ereditarietà permette di estrarre il codice **comune** tra più classi in una superclasse, evitando duplicazione.

### Il problema (codice duplicato)

```java
// Studente
private String nome;
private String cognome;
public String getNomeCompleto() { return nome + " " + cognome; }

// Professore
private String nome;     // ← STESSO!
private String cognome;  // ← STESSO!
public String getNomeCompleto() { return nome + " " + cognome; }  // ← STESSO!
```

### La soluzione: superclasse `Persona`

```java
// SUPERCLASSE
public class Persona {
    private String nome;
    private String cognome;

    public Persona(String nome, String cognome) {
        setNome(nome);
        setCognome(cognome);
    }

    public String getNome() { return nome; }
    public String getNomeCompleto() { return nome + " " + cognome; }
    // ... setter con validazione
}

// SOTTOCLASSE
public class Studente extends Persona {
    private final int matricola;

    public Studente(String nome, String cognome, int matricola) {
        super(nome, cognome);  // chiama il costruttore della superclasse
        if (matricola <= 0) {
            throw new IllegalArgumentException("Matricola non valida");
        }
        this.matricola = matricola;
    }
}

public class Professore extends Persona {
    private String settore;

    public Professore(String nome, String cognome, String settore) {
        super(nome, cognome);
        this.settore = settore;
    }
}
```

### Punti chiave di `extends`

- `extends` **crea il legame** di ereditarietà tra le classi
- Significa: *"prendi tutto quello che è e sa fare una Persona e aggiungici delle cose nuove che caratterizzano uno Studente"*
- `super(nome, cognome)` chiama il costruttore della **classe padre** e deve essere **sempre la prima riga** del costruttore figlio
- Il **Main non cambia**: `Studente s1 = new Studente("Anna", "Rossi", 1001)` funziona esattamente come prima

> Si chiama **refactoring**: migliorare la struttura interna del codice senza cambiare il comportamento esterno.

---

## 2. Polimorfismo con l'ereditarietà

Dopo aver creato `Persona`, possiamo scrivere codice che lavora con persone **in generale**:

```java
// Una variabile di tipo Persona può riferire un oggetto Studente!
Persona p = new Studente("Anna", "Rossi", 1001);
System.out.println(p.getNomeCompleto());  // funziona!
```

> Ci torneremo con il polimorfismo.

---

## 3. Modificatore `protected`

`protected` = accessibile nel **package** e nelle **sottoclassi**.

### Quando usarlo

```java
// Versione con private (campo non accessibile direttamente dalla sottoclasse)
public class Persona {
    private String nome;    // solo Persona può accedere
    private String cognome;
}

// La sottoclasse NON può fare:
public class Studente extends Persona {
    public void saluta() {
        // System.out.println(nome); // ❌ ERRORE! nome è private di Persona
        System.out.println(getNomeCompleto()); // ✅ deve usare il metodo
    }
}
```

```java
// Versione con protected
public class Persona {
    protected String nome;    // accessibile dalle sottoclassi
    protected String cognome;
}

// La sottoclasse può accedere direttamente:
public class Studente extends Persona {
    public void saluta() {
        System.out.println("Ciao, sono lo studente " + nome + " " + cognome);  // ✅
    }
}
```

### Quando preferire `private` vs `protected`

| Situazione | Consiglio |
|---|---|
| Caso generale | `private` + getter/setter (più controllo) |
| Hai ereditarietà e la sottoclasse **deve** accedere direttamente ai dati | `protected` |

> **Regola pratica:** Usando `protected`, dai più accesso → quindi meno controllo. Se puoi usare i metodi, non serve esporre i dati.

---

## 4. Classe vs Interfaccia

| | **Classe** | **Interfaccia** |
|---|---|---|
| Rappresenta | "Cosa **è**" un oggetto (struttura + stato) | "Cosa **può fare**" un oggetto (comportamento) |
| Contiene | Dati + implementazione dei metodi | Solo firme di metodi (contratto) |
| Ereditarietà | `extends` (una sola superclasse) | `implements` (più interfacce) |

---

## 5. Il Problema che Motiva le Interfacce

Nel sistema universitario, la classe `Esame` era legata a `Professore`:

```java
public class Esame {
    private String nome;
    private Professore professore;  // ⚠️ rigido! solo un Professore può valutare

    public void sostieniEsame(Studente studente) {
        int voto = professore.assegnaVoto(studente);
        System.out.println("Voto assegnato: " + voto);
    }
}
```

**Il problema:** e se il voto lo assegna una **commissione** o un **sistema automatico**?

- `Professore`, `Commissione`, `SistemaAutomatico` sono oggetti **diversi**
- **Non sono nella stessa gerarchia** (non ha senso che uno estenda l'altro)
- Ma tutti **sanno assegnare un voto**

> Legare `Esame` a `Professore` è limitante. Il codice è rigido e difficile da estendere.

---

## 6. Le Interfacce

> Un'**interfaccia** definisce un insieme di operazioni che una classe deve offrire. È un **contratto** tra chi usa un servizio e chi lo implementa. Descrive **cosa** può fare, non **come** è implementato.

### Definire un'interfaccia

```java
public interface Valutatore {
    int assegnaVoto(Studente studente);  // solo la firma, nessuna implementazione
}
```

### Implementare l'interfaccia in `Professore`

```java
public class Professore extends Persona implements Valutatore {
    // ...

    @Override
    public int assegnaVoto(Studente studente) {
        return random.nextInt(31);  // implementazione concreta: voto casuale 0-30
    }
}
```

### Aggiornare `Esame` per usare l'interfaccia

```java
public class Esame {
    private String nome;
    private Valutatore valutatore;  // ✅ ora accetta qualsiasi Valutatore!

    public Esame(String nome, Valutatore valutatore) {
        setNome(nome);
        setValutatore(valutatore);
    }

    public void sostieniEsame(Studente studente) {
        int voto = valutatore.assegnaVoto(studente);
        System.out.println("Voto assegnato: " + voto);
    }
}
```

### Il Main quasi non cambia

```java
// Funziona perché p1 è un Professore che implementa Valutatore
Esame esame = new Esame("Esame di Programmazione", p1);
esame.sostieniEsame(s1);

// Ma ora possiamo anche usare un SistemaAutomatico!
Valutatore va = new SistemaAutomatico(...);
Esame esame2 = new Esame("Esame di Programmazione", va);
esame2.sostieniEsame(s1);
```

### `@Override`

`@Override` è un'**annotation** che indica che stiamo ridefinendo un metodo già esistente (in una superclasse o interfaccia).
- Non è obbligatoria, ma è fortemente consigliata: il compilatore segnala errore se il metodo non corrisponde
- Recuperare la spiegazione completa in `03 - Annotation.pdf`

---

## 7. Ereditarietà vs Interfacce: quando usarle

| Meccanismo | Quando usarlo |
|---|---|
| **Ereditarietà** (`extends`) | Oggetti simili che condividono **codice comune** (struttura e stato) |
| **Interfacce** (`implements`) | Oggetti diversi che condividono un **comportamento**, ma lo realizzano in modo diverso |

> Con l'ereditarietà abbiamo raccolto ciò che è comune tra oggetti simili. Con le interfacce modelliamo comportamenti che possono essere condivisi anche da oggetti diversi.

### Le interfacce servono per condividere **comportamento**, non codice

```
Valutatore (interfaccia)
    |
    |-- Professore      implements Valutatore  (extends Persona)
    |-- Commissione     implements Valutatore
    |-- SistemaAutomatico  implements Valutatore
```

L'esame lavora con `Valutatore`: non gli interessa **chi** assegna il voto, solo che **sappia** assegnarlo.

---

## 8. Esercizio per Casa

Implementare una nuova classe che assegna il voto in modo diverso (`Commissione` o `SistemaAutomatico`) e usarla nel `Main`:

```java
public class SistemaAutomatico implements Valutatore {
    @Override
    public int assegnaVoto(Studente studente) {
        // implementazione diversa da quella del Professore
        return 18 + (studente.getMatricola() % 13);
    }
}

// Nel Main:
Valutatore va = new SistemaAutomatico(...);
Esame esame = new Esame("Esame di Programmazione", va);
esame.sostieniEsame(s1);
```

---

## 9. Punti Chiave per l'Esame ✅

- **`extends`** crea il legame di ereditarietà: la sottoclasse eredita dati e metodi della superclasse
- **`super(args)`** nel costruttore figlio = chiama il costruttore del padre; **deve essere la prima riga**
- **Refactoring** = migliorare la struttura interna senza cambiare il comportamento esterno (Main non cambia)
- **`protected`** = accessibile nel package E nelle sottoclassi (via di mezzo tra `private` e `public`)
- Preferire sempre `private` + getter/setter; usare `protected` solo se la sottoclasse deve accedere direttamente ai dati
- Un'**interfaccia** è un contratto: definisce **cosa** deve fare una classe, non **come**
- `interface` → contiene solo firme di metodi (nessuna implementazione)
- `implements` → la classe si impegna a implementare tutti i metodi dell'interfaccia
- `@Override` indica la ridefinizione di un metodo di superclasse/interfaccia (il compilatore verifica)
- Differenza: `extends` = "cosa è" (struttura); `implements` = "cosa può fare" (comportamento)
- Ereditarietà per **codice comune** tra oggetti simili; Interfacce per **comportamento condiviso** tra oggetti diversi
- Il problema di legare `Esame` a `Professore`: rigido. La soluzione `Valutatore` (interfaccia) lo rende flessibile
- Una classe può fare entrambe: `extends UnaClasse implements UnInterfaccia, AltraInterfaccia`
- `Persona p = new Studente(...)` → variabile di tipo superclasse che riferisce un oggetto sottoclasse (polimorfismo, approfondiremo)
