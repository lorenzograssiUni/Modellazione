# 07 - OOP, Inizio Progetto Sistema Universitario, Modificatori di Accesso e di Visibilità, Ereditarietà

---

## 1. Il Modello Concettuale OOP applicato al Progetto

Quando si progetta un sistema OOP, le domande fondamentali da porsi sono:

| Domanda | Traduzione tecnica |
|---|---|
| **Quali entità esistono nel problema?** | Quali classi devo modellare? |
| **Quali informazioni descrivono ogni entità?** | Quali campi (attributi) devono avere le classi? |
| **Quali azioni possono compiere queste entità?** | Quali metodi devono esporre le classi? |
| **Come interagiscono tra loro le entità?** | Quali relazioni (associazioni, dipendenze) tra classi? |

---

## 2. Progetto di Esempio: Sistema Universitario

> Iniziamo a costruire un piccolo sistema universitario. Non vogliamo fare un gestionale completo, ma un contesto abbastanza ricco da permetterci di introdurre molti concetti del corso.

### Le 3 entità principali

| Entità | Dati | Metodi |
|---|---|---|
| **Studente** | `nome`, `cognome`, `matricola` | `getNomeCompleto()`, `saluta()` |
| **Professore** | `nome`, `cognome`, `settore` | `getNomeCompleto()`, `presenta()` |
| **Corso** | `nome`, `docente` (Professore), `listaStudenti` | `iscriviStudente()`, `stampaIscritti()` |

**Relazioni:**
- Un corso ha **un** professore
- Un corso ha **molti** studenti

### Classi Studente e Professore

```java
public class Studente {
    private String nome;
    private String cognome;
    private int matricola;

    public String getNomeCompleto() {
        return nome + " " + cognome;
    }
}

public class Professore {
    private String nome;
    private String cognome;
    private String settore;

    public String getNomeCompleto() {
        return nome + " " + cognome;
    }
}
```

### Main di esempio

```java
public static void main(String[] args) {
    Studente s1 = new Studente("Anna", "Rossi", 1001);
    Studente s2 = new Studente("Marco", "Verdi", 1002);
    Professore p1 = new Professore("Luca", "Bianchi", "Programmazione");
    Corso corso = new Corso("Metodologie di Programmazione", p1);

    s1.saluta();
    p1.presenta();
    corso.iscriviStudente(s1);
    corso.iscriviStudente(s2);
    corso.stampaIscritti();
}
```

> 🔗 Repository di riferimento del corso: https://github.com/FabrizioFornari/MDP-MGC-2526

---

## 3. Incapsulamento

> L'**incapsulamento** è il principio per cui i dati di un oggetto sono nascosti e accessibili solo tramite metodi controllati.

### Il problema con `public`

```java
// Con campi pubblici: chiunque può fare qualsiasi cosa
public class Studente {
    public String nome;
    public int matricola;
}

Studente s = new Studente();
s.nome = "Anna";
s.matricola = -100;  // ⚠️ matricola negativa: dati corrotti!
```

### La soluzione con `private` + metodi controllati

```java
// Con campi privati: l'accesso passa per metodi che applicano regole
public class Studente {
    private String nome;
    private String cognome;
    private int matricola;
}
```

```java
// Il metodo setter controlla la validità del dato
public void setNome(String nome) {
    if (nome == null || nome.isEmpty()) {
        throw new IllegalArgumentException("Nome non valido");
    }
    this.nome = nome;
}
```

> **Nota:** Non vietiamo la modifica... la **controlliamo**.

---

## 4. Modificatori di Visibilità

Definiscono **chi può accedere** a classi, attributi e metodi.

| Modificatore | Dove è accessibile | Quando usarlo | Esempio |
|---|---|---|---|
| **`public`** | Ovunque | Quando il metodo/dato deve essere usato da altre classi | `public String getNome()` |
| **`protected`** | Package + sottoclassi | Quando si usa l'ereditarietà (vedremo meglio) | `protected int eta` |
| **`private`** | Solo nella classe | Per proteggere i dati interni | `private String nome` |
| *(nessuno)* | Solo nel package | Accesso package-private (raro) | `int valore` |

### Regola pratica

- I **dati (attributi)** → quasi sempre `private`
- I **metodi pubblici** → `public` (interfaccia della classe)
- I **metodi interni di supporto** → `private`

---

## 5. Modificatori di Accesso

Definiscono **come vengono usati** attributi e metodi (comportamento e vincoli).

| Modificatore | Significato | Esempio |
|---|---|---|
| **`static`** | Appartiene alla **classe**, non agli oggetti; condiviso da tutte le istanze | `public static void main(String[] args)` |
| **`final`** | Non modificabile (costante per le variabili; non sovrascrivibile per i metodi) | `private final int matricola` |

### Perché `main` è `public static`?

```java
public static void main(String[] args) {
    // punto di ingresso del programma
}
```

- **`public`** → deve essere visibile alla JVM (che è "esterna" alla nostra classe)
- **`static`** → può essere eseguito **senza creare oggetti** (il programma parte senza istanziare un oggetto Main)

### `final` con la matricola

```java
public class Studente {
    private String nome;
    private String cognome;
    private final int matricola;  // assegnata una sola volta!

    public Studente(String nome, String cognome, int matricola) {
        setNome(nome);
        setCognome(cognome);
        if (matricola <= 0) {
            throw new IllegalArgumentException("Matricola non valida");
        }
        this.matricola = matricola;
    }
}
```

- `final int matricola` → può essere assegnata nel costruttore, ma **non può essere riassegnata** dopo
- `final int MAX_STUDENTI = 100` → costante di classe

> **`final` = non modificabile dopo l'assegnazione**

---

## 6. Osservazione: Codice Duplicato

Guardando Studente e Professore:

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

**Problema:**
- Entrambe le classi hanno `nome`, `cognome` e `getNomeCompleto()`
- Probabilmente entrambe avranno anche getter/setter simili
- "Se domani volessi cambiare come gestisco il nome completo... quante classi devo modificare?"

**Soluzione → Ereditarietà**

---

## 7. Ereditarietà

> L'**ereditarietà** permette di specializzare una classe: la sottoclasse eredita dati e metodi della superclasse e può aggiungerne di propri.

### Sintassi con `extends`

```java
// Superclasse (classe padre/base)
public class Persona {
    private String nome;
    private String cognome;

    public String getNomeCompleto() {
        return nome + " " + cognome;
    }
}

// Sottoclasse (classe figlia/derivata)
public class Studente extends Persona {
    private int matricola;  // solo ciò che è specifico di Studente
}

public class Professore extends Persona {
    private String settore;  // solo ciò che è specifico di Professore
}
```

### Gerarchia

```
        Persona
       (nome, cognome)
       (getNomeCompleto)
           /\
          /  \
   Studente  Professore
 (matricola)  (settore)
```

- **Superclasse** `Persona` → contiene il codice comune
- **Sottoclasse** `Studente` extends `Persona` → specializzazione di Persona
- **Sottoclasse** `Professore` extends `Persona` → specializzazione di Persona

### Vantaggi dell'ereditarietà

- **Evita la duplicazione** del codice (DRY: Don't Repeat Yourself)
- **Centralizza** la logica comune nella superclasse
- **Facilita la manutenzione**: modifica in un posto solo
- **Estendibilità**: aggiungere nuove specializzazioni senza toccare il codice esistente

---

## 8. Cos'è la JVM?

La **Java Virtual Machine (JVM)** è il motore di esecuzione di Java:

- Il codice Java viene compilato in **bytecode** (file `.class`), non direttamente in codice macchina
- La JVM **interpreta** (o compila JIT) il bytecode ed è diversa per ogni sistema operativo
- Questo permette il principio **"Write Once, Run Anywhere"**: lo stesso `.class` gira su Windows, Mac, Linux

```
Codice Java (.java)
       ↓  javac (compilatore)
Bytecode (.class)
       ↓  JVM (interpreta)
Esecuzione sul sistema operativo
```

---

## 9. Struttura Package del Sistema Universitario

```
it.unicam.mdp2526.universita
├── Persona.java
├── Studente.java
├── Professore.java
└── Corso.java
```

```java
package it.unicam.mdp2526.universita;

import it.unicam.mdp2526.universita.Studente;
```

---

## 10. Punti Chiave per l'Esame ✅

- Le 4 domande OOP: entità → classi; informazioni → attributi; azioni → metodi; relazioni → associazioni
- **Incapsulamento** = dati `private` + accesso controllato tramite metodi (setter con validazione)
- **`private`** = solo la classe stessa può accedere direttamente ai dati
- **`public`** = accessibile da ovunque
- **`protected`** = accessibile nel package e nelle sottoclassi
- **Setter con validazione**: non si vieta la modifica, si **controlla** (`throw new IllegalArgumentException`)
- **`static`** = appartiene alla classe, non agli oggetti; non serve istanziare per chiamarlo
- **`final`** su variabile = non può essere riassegnata dopo la prima assegnazione
- **`final`** su metodo = non può essere sovrascritto nelle sottoclassi
- `public static void main` = `public` per la JVM + `static` perché parte senza creare oggetti
- **Ereditarietà** = `extends`; la sottoclasse eredita tutto dalla superclasse e aggiunge solo il suo specifico
- **Superclasse** (padre/base) → **Sottoclasse** (figlia/derivata)
- Il codice duplicato tra Studente e Professore si elimina estraendo una superclasse `Persona`
- **JVM** = esegue il bytecode Java su ogni sistema operativo → "Write Once, Run Anywhere"
- Gerarchia: `Libreria → Package → Classe → Oggetto`; package = cartella logica di classi
