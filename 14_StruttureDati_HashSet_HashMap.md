# 14 - Strutture Dati, HashSet e HashMap

---

## 1. Il problema rimasto aperto: `List` e i duplicati

Con `equals` ora sappiamo **riconoscere** due oggetti equivalenti, ma `List` non impedisce ancora i duplicati:

```java
Studente s1 = new Studente("Anna", "Rossi", 1001);
Studente s2 = new Studente("Anna", "Rossi", 1001);

List<Studente> iscritti = new ArrayList<>();
iscritti.add(s1);
iscritti.add(s2);
System.out.println(iscritti.size());  // 2! (anche con equals)
```

Una soluzione "manuale" sarebbe:
```java
if (!iscritti.contains(s2)) {
    iscritti.add(s2);
}
```

Ma questo approccio è **brutto**: codice duplicato, rischio di errore, qualcuno può dimenticarsi il controllo. Ci vuole una struttura dati che lo gestisca in automatico.

---

## 2. `Set` e `HashSet`: nessun duplicato per contratto

```java
Set<Studente> iscritti = new HashSet<>();
iscritti.add(s1);
iscritti.add(s2);
System.out.println(iscritti.size());  // ...vediamo dopo perché non funziona subito
```

- **`Set`** è un'**interfaccia** del Java Collections Framework che garantisce: *"nessun elemento duplicato"* (nessuna coppia di elementi `e1` e `e2` tale che `e1.equals(e2)`).
- **`HashSet`** è la **classe concreta** che implementa `Set` con performance quasi costante per `add`, `remove`, `contains`, `size`.

> `HashSet` è un caso particolare di `HashMap`: usa solo le **chiavi**, senza valori associati.

---

## 3. Complessità: `List.contains()` vs `HashSet.contains()`

### `List.contains()` → O(n)

Scorre tutta la lista confrontando elemento per elemento con `equals()`:
- 1.000 studenti → fino a 1.000 chiamate a `equals()`
- Lo studente non c'è? Java fa 1.000 operazioni inutili.

### `HashSet.contains()` → O(1) in media

Non scorre nulla. Fa un **salto diretto**:

1. Calcola `hashCode()` dell'oggetto (quasi istantaneo)
2. Usa quell'hash per individuare uno **"scaffale"** (bucket)
3. Se lo scaffale è vuoto → `false` immediato
4. Se c'è qualcosa → chiama `equals()` solo su quei pochi elementi

> Analogia con Git: proprio come SHA-1 genera un hash univoco dal contenuto di un commit, `hashCode()` in Java genera un numero intero dal contenuto dei campi dell'oggetto.

---

## 4. `hashCode()`: il contratto con `equals`

> **Regola fondamentale:** se due oggetti sono uguali secondo `equals()`, allora **devono** avere lo stesso `hashCode()`.

Senza `hashCode()` ridefinito, `HashSet` non funziona correttamente: due oggetti con stessa matricola finiscono in scaffali diversi e vengono trattati come distinti.

```java
@Override
public int hashCode() {
    return Objects.hash(matricola);  // coerente con equals che usa matricola
}
```

I campi scelti per `hashCode` devono essere **gli stessi** usati in `equals`. Non si scelgono a caso: si scelgono in base a cosa rende un oggetto **unico nel dominio**.

### Risultato dopo l'override di entrambi

```java
Set<Studente> iscritti = new HashSet<>();
iscritti.add(s1);  // matricola 1001
iscritti.add(s2);  // matricola 1001 (stesso hashCode, stesso scaffale, equals → true)
System.out.println(iscritti.size());  // 1 ✅
```

---

## 5. Java Collections Framework

Il **Java Collections Framework** è un insieme di classi e interfacce già pronte per gestire gruppi di oggetti. Invece di scrivere strutture dati da zero, il framework offre:

- Strutture dati: `List`, `Set`, `Map`
- Operazioni comuni: `add`, `remove`, `contains`, `size`
- Un modello coerente per usarle tutte allo stesso modo

---

## 6. `HashMap`: associare chiave → valore

### Il problema: ricerca efficiente per chiave

Con `List<EsameSuperato>`, trovare il voto di un esame richiede di scorrere tutta la lista:

```java
public int getVoto(Esame esame) {
    for (EsameSuperato es : esamiSuperati) {
        if (es.getEsame().equals(esame)) {
            return es.getVoto();
        }
    }
    throw new IllegalArgumentException("Esame non trovato");
}
// O(n): con 100 esami, fino a 100 confronti
```

### La soluzione: `Map<Esame, Integer>`

**`Map`** è un'interfaccia che memorizza dati in coppie **Chiave-Valore (Key-Value)**:
- **Chiave (Key)**: identificatore unico → non ci possono essere due chiavi uguali
- **Valore (Value)**: il dato associato alla chiave → può essere duplicato

**`HashMap`** è l'implementazione concreta: recupero in **O(1)** tramite hash della chiave.

```java
// Libretto aggiornato
public class Libretto implements CalcolatoreMedia, VisualizzatoreEsami {
    private final Studente studente;
    private Map<Esame, Integer> voti = new HashMap<>();  // Esame=chiave, voto=valore

    public void registraEsameSuperato(Esame esame, int voto) {
        voti.put(esame, voto);  // inserisce o sovrascrive
    }

    public int getVoto(Esame esame) {
        return voti.get(esame);  // O(1) grazie all'hash
    }
}
```

### Nel `Main`

```java
studente.getLibretto().registraEsameSuperato(esame1, 28);
studente.getLibretto().registraEsameSuperato(esame2, 30);

// Recupero con un nuovo oggetto Esame (stesso nome, stesso prof)
int voto = studente.getLibretto().getVoto(new Esame("Programmazione", prof));
System.out.println("Voto trovato: " + voto);  // 28 ✅ (richiede equals+hashCode su Esame!)
```

> Per far funzionare `HashMap` con `Esame` come chiave, bisogna implementare `equals()` e `hashCode()` **anche su `Esame`**. ← **Esercizio per casa!**

---

## 7. Cosa succede a `EsameSuperato`?

Prima avevamo:
```
List<EsameSuperato>  con  EsameSuperato { Esame esame; int voto; }
```

Adesso con `Map<Esame, Integer>` la classe `EsameSuperato` **scompare da Libretto**.

> **`EsameSuperato` è veramente essenziale nel dominio?** Nel libretto ci possono essere solo esami *superati*, quindi il concetto di "esame non superato" nel libretto non esiste. `Map<Esame, Integer>` rappresenta già questa semantica in modo più diretto. Ma **dipende**: se il dominio richiedesse ulteriori dati sull'esame superato (es. data, commissione), allora `EsameSuperato` tornerebbe utile.

---

## 8. Riepilogo: `List` vs `Set` vs `Map`

| Struttura | Interfaccia | Impl. | Duplicati | Ordine | Ricerca | Uso tipico |
|---|---|---|---|---|---|---|
| `ArrayList` | `List` | concreta | ✅ sì | ✅ sì (indice) | O(n) | Sequenze ordinate, accesso per posizione |
| `HashSet` | `Set` | concreta | ❌ no | ❌ no | O(1) | Insiemi senza duplicati |
| `HashMap` | `Map` | concreta | ❌ no (chiavi) | ❌ no | O(1) | Associazioni chiave→valore |

> **La scelta della struttura dati giusta influenza il comportamento del sistema.** Non basta scrivere codice corretto: bisogna anche scegliere la struttura più adatta al problema.

---

## 9. Punti Chiave per l'Esame ✅

- **`List` ammette duplicati** — anche con `equals` ridefinito, `add` non controlla; serve `contains` manuale o una struttura diversa
- **`Set`** è l'interfaccia che garantisce **nessun duplicato** (`e1.equals(e2)` non può essere `true` per due elementi distinti)
- **`HashSet`** implementa `Set` con performance **O(1)** per `add/remove/contains` grazie a `hashCode()`
- **Contratto `equals`/`hashCode`**: se `equals` → `true`, allora `hashCode` **deve** essere uguale; vanno sempre ridefiniti insieme
- **`hashCode()` in Java**: `Objects.hash(matricola)` — i campi scelti devono essere gli stessi di `equals`
- **`Map`** memorizza coppie chiave→valore; chiavi univoche, valori possono ripetersi
- **`HashMap`** implementa `Map` con accesso **O(1)** tramite hash della chiave
- **`HashSet` è un caso particolare di `HashMap`**: usa solo chiavi, nessun valore
- **`Map<Esame, Integer>`** sostituisce `List<EsameSuperato>` in Libretto: più efficiente e più leggibile
- **Per usare `Esame` come chiave di `HashMap`** bisogna implementare `equals()` e `hashCode()` anche su `Esame` ← **esercizio!**
- **Complessità**: `List.contains()` → O(n); `HashSet.contains()` / `HashMap.get()` → O(1) in media
- **La struttura dati giusta cambia il comportamento del sistema**: non è solo una questione di prestazioni
