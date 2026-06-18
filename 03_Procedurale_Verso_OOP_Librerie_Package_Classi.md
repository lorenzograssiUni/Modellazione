# 03 - Programmazione Procedurale, Verso OOP, Riutilizzo del Codice, Librerie, Package, Classi

---

## 1. Riflessione sulla Programmazione Procedurale

La programmazione procedurale funziona bene per programmi semplici, ma presenta limiti evidenti con la crescita della complessità.

### Problemi del modello procedurale

Considerando un programma che gestisce un singolo studente:

```c
#include <stdio.h>
#include <string.h>
int main() {
    float voto_scritto, voto_progetto, voto_finale;
    char risposta[10];
    // ...
}
```

| Problema | Descrizione |
|---|---|
| **Scalabilità** | Funziona per 1 studente. Con 200 studenti bisogna modificare tutto o usare array/file |
| **Dati sparsi** | Le informazioni dello studente (`voto_scritto`, `voto_progetto`, `risposta`) sono variabili locali sparse, senza un contenitore naturale |
| **Logica sparsa** | Se la regola dell'esame cambia, la logica è distribuita nel flusso e difficile da localizzare |
| **Nessuna entità** | Il programma rappresenta *i passi da eseguire*, non *le entità del problema reale* |

> **La programmazione procedurale non è sbagliata.** Semplicemente, quando il problema contiene molte entità con dati e comportamenti propri, può diventare meno naturale da organizzare.

### Algoritmo = logica, Linguaggio = modo di scriverla

La stessa logica può essere scritta in C o Bash; la logica non cambia, cambia solo la sintassi:

| Concetto | C | Bash |
|---|---|---|
| Leggere input | `scanf` | `read` |
| Condizione | `if (...)` | `if [ ... ]` |
| AND | `&&` | `&&` |
| Output | `printf` | `echo` |

> **Differenza filosofica:** C è un linguaggio completo per programmare; Bash è un linguaggio per *orchestrare* programmi già esistenti.

> ⚠️ Bash non gestisce i decimali nativamente → per calcolare `0.7 * voto_scritto + 0.3 * voto_progetto` si usa il trucco `(voto_scritto * 7 + voto_progetto * 3) / 10`.

---

## 2. Esercizio Completo: Modalità Esame (Pseudocodice)

Versione completa dell'algoritmo che modella l'esame con colloquio orale opzionale:

```
INIZIO
  LEGGI voto_scritto
  LEGGI voto_progetto
  SE voto_scritto >= 18 E voto_progetto >= 18 ALLORA
    SCRIVI "Esame superato"
    voto_finale ← voto_scritto * 0.7 + voto_progetto * 0.3
    SCRIVI "Voto attuale: ", voto_finale
    SCRIVI "Vuoi sostenere il colloquio orale? (SI/NO)"
    LEGGI risposta
    SE risposta == "SI" ALLORA
      SCRIVI "Buona fortuna"
    ALTRIMENTI
      SCRIVI "Voto confermato: ", voto_finale
    FINE SE
  ALTRIMENTI
    SCRIVI "Esame non superato"
  FINE SE
FINE
```

**Traduzione in Bash:**
```bash
#!/bin/bash
echo "Inserisci voto scritto:"
read voto_scritto
echo "Inserisci voto progetto:"
read voto_progetto

if [ "$voto_scritto" -ge 18 ] && [ "$voto_progetto" -ge 18 ]; then
    echo "Esame superato"
    voto_finale=$(( (voto_scritto * 7 + voto_progetto * 3) / 10 ))
    echo "Voto attuale: $voto_finale"
    echo "Vuoi sostenere il colloquio orale? (SI/NO)"
    read risposta
    if [ "$risposta" = "SI" ]; then
        echo "Buona fortuna"
    else
        echo "Voto confermato: $voto_finale"
    fi
else
    echo "Esame non superato"
fi
```

**Traduzione in C:**
```c
#include <stdio.h>
#include <string.h>

int main() {
    float voto_scritto, voto_progetto, voto_finale;
    char risposta[10];

    printf("Inserisci voto scritto: ");
    scanf("%f", &voto_scritto);
    printf("Inserisci voto progetto: ");
    scanf("%f", &voto_progetto);

    if (voto_scritto >= 18 && voto_progetto >= 18) {
        printf("Esame superato\n");
        voto_finale = voto_scritto * 0.7f + voto_progetto * 0.3f;
        printf("Voto attuale: %.2f\n", voto_finale);
        printf("Vuoi sostenere il colloquio orale? (SI/NO): ");
        scanf("%s", risposta);
        if (strcmp(risposta, "SI") == 0) {
            printf("Buona fortuna\n");
        } else {
            printf("Voto confermato: %.2f\n", voto_finale);
        }
    } else {
        printf("Esame non superato\n");
    }
    return 0;
}
```

### Esercizio per casa: pseudocodice completo modalità esame

Steps da modellare:
1. Sottomissione progetto
2. Ricezione voto progetto
3. Superato il progetto? (sì/no)
4. Faccio l'esame scritto?
5. Ricezione voto finale
6. Calcolo voto scritto
7. Faccio l'orale?
8. Voto finale

---

## 3. Evoluzione dei Paradigmi: dalla Procedurale all'OOP

```
Programmazione Procedurale
        ↓
Programmazione Strutturata    (problema: salti goto ovunque)
        ↓
Programmazione Orientata agli Oggetti    (problema: organizzazione dei dati)
```

| Fase | Problema risolto | Come |
|---|---|---|
| **Procedurale** | Codice duplicato | Funzioni riutilizzabili |
| **Strutturata** | Salti `goto` illeggibili | Blocchi con sequenza/selezione/iterazione |
| **OOP** | Dati sparsi, entità non rappresentate | Oggetti con dati + comportamento uniti |

> **Finora abbiamo organizzato bene:** il flusso, le istruzioni, i blocchi del programma.
> **Il problema rimasto:** i dati sono separati, passati alle funzioni, modificati dall'esterno.

---

## 4. Verso la Programmazione Orientata agli Oggetti

### Il cambio di prospettiva

> "E se invece di pensare prima al programma e ai suoi passi, pensassimo prima alle **cose che esistono nel problema**?"

In un contesto universitario, le entità reali sono:
- `Studente`
- `Esame`
- `Docente`
- `Corso`

### Confronto tra i due approcci

| | Procedurale | OOP |
|---|---|---|
| **Domanda principale** | Quali passi deve eseguire il programma? | Quali oggetti esistono? Che dati hanno? Cosa sanno fare? Come collaborano? |
| **Focus** | Il flusso di esecuzione | La struttura del sistema |
| **Prima pensa a...** | I passi (leggi, calcola, controlla, stampa) | Le entità (Studente, Esame, Libretto) |

### Esempio: entità Studente in OOP

Uno `Studente` ha:

**Dati:**
- `matricola`
- `nome`, `cognome`
- `anno_di_immatricolazione`
- `numero_cfu`
- `media_voti`
- `lista_esami`

**Operazioni (metodi):**
- `iscrivi_esame()`
- `registra_voto()`
- `calcola_media()`
- `aggiungi_cfu()`
- `stampa_libretto()`

**Regole (logica che protegge i dati):**
- Regola 1: Uno studente non può registrare un esame se il voto è < 18
- Regola 2: Uno studente non può registrare lo stesso esame due volte

> ⚠️ **Le regole stanno dentro i metodi.** Non sono sparse nel flusso principale.

```java
void registraEsame(int voto) {
    if (voto < 18)
        throw new Error("Esame non superato");
    esami.add(voto);
}
```

---

## 5. Modello Concettuale dell'OOP

### Un oggetto è:

> Un'entità che **incapsula dati e comportamento** e **garantisce che il proprio stato rimanga valido**.

- Ogni cerchio = un oggetto
- Il centro = dati (stato interno)
- Attorno al centro = metodi (comportamento)
- Le frecce = interazioni tra oggetti

### Il flusso in OOP

Il flusso non sparisce in OOP! A runtime:
- Il programma parte da un punto iniziale
- Vengono chiamati metodi
- Un metodo ne chiama un altro
- Le istruzioni vengono eseguite in ordine

**Quello che cambia** è che non si descrive più il sistema come una *catena unica di passi*, ma come:
- Oggetti con stato interno
- Metodi
- Messaggi/chiamate tra oggetti

> "Il flusso c'è ancora, ma non è più il **centro del modello mentale**."

### Classi vs Oggetti

| Termine | Definizione | Esempio |
|---|---|---|
| **Classe** | Tipo/modello — descrive struttura e comportamento di un insieme di oggetti | `Studente`, `Esame`, `Docente` |
| **Oggetto** | Istanza concreta di una classe — esiste in memoria a runtime | `studenteMarco`, `esameMDP`, `docenteRossi` |

```
Classe = stampo
Oggetto = prodotto creato con quello stampo
```

### Cosa viene duplicato quando si istanziano più oggetti?

```java
class Studente {
    String nome;       // dato
    int matricola;     // dato
    void saluta() {    // metodo
        System.out.println("Ciao");
    }
}

Studente s1 = new Studente();
Studente s2 = new Studente();
Studente s3 = new Studente();
```

- ✅ I **dati (stato)** vengono duplicati per ogni oggetto → ogni oggetto ha il proprio `nome` e `matricola`
- ❌ I **metodi** NON vengono duplicati → `saluta()` esiste una sola volta in memoria, condivisa da tutti gli oggetti

> **Identità dell'oggetto ≠ valore dei dati**
> Tre oggetti creati senza impostare valori sono *oggetti diversi* con *dati inizialmente uguali* (valori di default: `null`, `0`).

```java
System.out.println(s1 == s2);   // false → oggetti distinti
Studente s4 = s1;
System.out.println(s1 == s4);   // true → stessa istanza
```

### Valori di default in Java

| Tipo | Valore default |
|---|---|
| `int`, `float`, `double` | `0` / `0.0` |
| `boolean` | `false` |
| `String` / oggetti | `null` |

---

## 6. Storia dell'OOP

| Anno | Linguaggio | Contributo |
|---|---|---|
| 1967 | **Simula 67** | Nascono i concetti di oggetto e classe; creato da Ole-Johan Dahl e Kristen Nygaard al Norwegian Computing Center (NCC) per simulazioni di sistemi reali |
| Anni '70 | **Smalltalk** | L'idea OOP diventa radicale: *tutto è un oggetto*; il programma = oggetti che si scambiano messaggi |
| 1983 | **C++** | OOP integrata nel C |
| 1995 | **Java** | OOP per il web e la portabilità |

> "Quando il software è piccolo → il flusso è il problema principale. Quando il software è enorme → il problema diventa l'**organizzazione**."
>
> **L'OOP è una risposta al problema della complessità strutturale, non del flusso.**

**Riferimento:** Robert C. Martin (Uncle Bob) — co-autore del Manifesto Agile, promotore dei principi SOLID, autore di *Clean Code* e *Clean Architecture*.

---

## 7. UML — Unified Modeling Language

> UML = Unified Modeling Language
>
> È un **linguaggio grafico** per descrivere sistemi software. È un modo standard per disegnare oggetti, classi e le loro relazioni.

**Definizione ufficiale (OMG):**
> "A specification defining a graphical language for visualizing, specifying, constructing, and documenting the artifacts of distributed object systems."

### Tipi di diagrammi UML principali

| Diagramma | Cosa descrive |
|---|---|
| **Class Diagram** | Classi, attributi, metodi, relazioni tra classi |
| **Object Diagram** | Istanze concrete (oggetti) a runtime |
| **Sequence Diagram** | Interazioni tra oggetti nel tempo (messaggi) |
| **Use Case Diagram** | Funzionalità del sistema dal punto di vista dell'utente |
| **Activity Diagram** | Flusso di attività / algoritmi (simile al diagramma di flusso) |

> ℹ️ UML sarà approfondito nel corso di **Ingegneria del Software** dell'anno successivo.

---

## 8. Riutilizzo del Codice: Librerie, Package, Classi

### Gerarchia del riutilizzo

```
Libreria
  └── Package (cartella logica di classi)
        └── Sottopacchetto
              └── Classe
                    └── Oggetto
                          ├── Dati (stato)
                          └── Metodi (comportamento)
```

### Libreria

Un insieme organizzato di package. Non si parte da zero in Java:

| Package | Contenuto |
|---|---|
| `java.lang` | Classi fondamentali (String, Math, Object...) — importata automaticamente |
| `java.util` | Strutture dati (ArrayList, HashMap, Scanner...) |
| `java.io` | Input/Output (file, stream) |
| `java.math` | Operazioni matematiche avanzate |

> Questi sono tutti package diversi che fanno parte della **Java Standard Library**.

### Package

Un **package** è una cartella logica di classi. Serve per organizzare il codice quando cresce.

```java
package org.unicam.mdp;  // dichiarazione del package

public class Studente { ... }
```

```java
import org.unicam.mdp.Studente;  // usare una classe di un altro package
```

I package corrispondono a **cartelle nel filesystem**:
```
org/
  unicam/
    mdp/
      Studente.java
      Esame.java
      Docente.java
```

### Convenzione di naming dei package

Il nome è organizzato **dal generale al particolare**, usando il dominio web invertito:

```
it.unicam.mdp2526.universita
│    │       │        │
│    │       │        └── dominio applicativo
│    │       └── corso / anno accademico
│    └── Università di Camerino
└── Italia
```

**Indirizzo completo della classe Studente:**
```
it.unicam.mdp2526.universita.Studente
```

### Classe

Una classe è un **modello** che descrive struttura e comportamento di un insieme di oggetti.

```java
package org.unicam.mdp;

public class Studente {
    String nome;
    int matricola;

    void saluta() {
        System.out.println("Ciao");
    }
}
```

### Esempio pratico: ArrayList

```java
import java.util.ArrayList;

ArrayList<String> nomi = new ArrayList<>();
nomi.add("Marco");
nomi.add("Giulia");
```

| Elemento | Valore nell'esempio |
|---|---|
| Classe | `ArrayList` (da `java.util`) |
| Oggetto | `nomi` |
| Metodo | `add()` |
| Dati | Gli elementi dentro la lista |

> `ArrayList<T>` → lista di oggetti di tipo T (generics). `T` viene sostituito dal tipo reale al momento della dichiarazione (es. `String`).

---

## 9. Esercizio in Classe: Classe Studente in Java

**Traccia:** Definire la classe `Studente` con `nome` e `matricola`. Istanziare 3 oggetti e stampare nome e matricola per ognuno senza assegnare valori.

```java
package org.unicam.mdp;

public class Studente {
    String nome;
    int matricola;

    void saluta() {
        System.out.println("Ciao");
    }

    public static void main(String[] args) {
        Studente s1 = new Studente();
        Studente s2 = new Studente();
        Studente s3 = new Studente();

        System.out.println(s1.nome + "\n" + s1.matricola);
        System.out.println(s2.nome + "\n" + s2.matricola);
        System.out.println(s3.nome + "\n" + s3.matricola);
    }
}
```

**Output:**
```
null
0
null
0
null
0
```

> I dati ci sono anche se non li abbiamo impostati: Java assegna valori di default (`null` per String, `0` per int).

Dopo si possono differenziare:
```java
s1.nome = "Marco";
s2.nome = "Giulia";
s3.nome = "Luca";
```

---

## 10. Punti Chiave per l'Esame ✅

- La **programmazione procedurale** fallisce per grandi sistemi perché i dati sono sparsi e non c'è un contenitore naturale per le entità del dominio
- Il passaggio a OOP risolve il problema dell'**organizzazione**, non del flusso (il flusso c'è ancora in OOP!)
- In OOP la domanda principale cambia: *"Quali passi?"* → *"Quali oggetti esistono e come collaborano?"*
- Un **oggetto** incapsula dati + comportamento e garantisce la validità del proprio stato
- Una **classe** è il modello/tipo; un **oggetto** è l'istanza concreta
- Quando si istanziano più oggetti, i **dati vengono duplicati**, i **metodi no**
- **Identità ≠ stato**: due oggetti con gli stessi dati sono comunque oggetti diversi (`s1 == s2` → `false`)
- Un **package** è una cartella logica di classi; la convenzione di naming usa il dominio web invertito
- `import` serve per usare classi definite in altri package
- **UML** è il linguaggio grafico standard per modellare sistemi OOP (Class Diagram, Object Diagram, Sequence Diagram, Use Case Diagram, Activity Diagram)
- L'OOP nasce con **Simula 67 (1967)** — Ole-Johan Dahl e Kristen Nygaard
- In Bash: i decimali non sono supportati nativamente nell'aritmetica → usare il trucco `*7 *3 /10`
- `java.lang` è l'unico package importato **automaticamente** da Java
