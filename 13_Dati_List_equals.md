# 13 - Dati, List e equals

---

## 1. Responsabilità di una Classe: i Dati

Ogni classe ha una responsabilità chiara su **due cose**:

1. **Il proprio stato** (i dati che gestisce)
2. **Il proprio comportamento** (le operazioni che esegue)

Finora ci siamo concentrati sulle relazioni tra classi e sul comportamento. Ora ci concentriamo sui **dati**: liste di studenti, esami, voti.

---

## 2. Il problema: `List` non impedisce duplicati

```java
Corso corso = new Corso("Metodologie di Programmazione", p1);
Studente s1 = new Studente("Anna", "Rossi", 1001);

corso.iscriviStudente(s1);
corso.iscriviStudente(s1);   // stesso oggetto, due volte
corso.stampaIscritti();      // Anna Rossi appare DUE volte!
```

> **`List` non impedisce duplicati** — aggiungere lo stesso oggetto due volte è permesso.

Ma c'è un problema ancora più subdolo:

```java
// Due oggetti diversi in memoria, stesso significato nel dominio
corso.iscriviStudente(new Studente("Anna", "Rossi", 1001));
corso.iscriviStudente(new Studente("Anna", "Rossi", 1001));
// Quanti studenti? 2 — ma nel dominio è lo stesso studente!
```

---

## 3. `==` vs `equals`: identità vs uguaglianza

```java
Studente s3 = new Studente("Marco", "Verdi", 1002);
Studente s4 = new Studente("Marco", "Verdi", 1002);

System.out.println(s3 == s4);        // false — confronta l'indirizzo in memoria
System.out.println(s3.equals(s4));   // false (di default!) — usa lo stesso criterio di ==
```

**Di default**, `equals` in Java confronta l'indirizzo in memoria, NON i dati dell'oggetto.

> Java non conosce il dominio. Java non sa cosa significa "stesso studente". **Dobbiamo dirglielo noi.**

---

## 4. Quando nasce il problema dei duplicati?

Il problema non emerge solo quando si crea due volte lo stesso oggetto nel `Main`. Emerge quando il sistema cresce e gli oggetti vengono **ricreati da fonti diverse**:

- Letti da un file JSON
- Caricati da un database
- Ricevuti da una API

```java
Studente studente1 = new Studente("Anna", "Rossi", 1001);    // creato nel codice
Studente studenteDaJson = gson.fromJson(reader, Studente.class); // letto da file

System.out.println(studente1.equals(studenteDaJson));  // false (senza override)
// Java vede due oggetti diversi, anche se rappresentano la stessa persona
```

---

## 5. Lettura dati da JSON con Gson

### Da file esterno (FileSystem)

```java
// Per dati DINAMICI che cambiano nel tempo (es. studenti reali)
Gson gson = new Gson();
try (FileReader reader = new FileReader("/percorso/studente.json")) {
    Studente studenteDaJson = gson.fromJson(reader, Studente.class);
} catch (IOException e) {
    throw new RuntimeException(e);
}
```

> ⚠️ Il path assoluto è fragile: funziona solo su quella macchina.

### Da cartella `resources` (con Gradle)

```java
// Per dati STATICI: configurazioni iniziali, dati hardcoded del sistema
// I file in resources vengono inclusi nel JAR → read-only a runtime
Gson gson2 = new Gson();
InputStream is = Main.class.getClassLoader().getResourceAsStream("studente.json");
InputStreamReader reader2 = new InputStreamReader(is);   // da byte a caratteri
Studente studenteDaResources = gson2.fromJson(reader2, Studente.class);
```

| Approccio | Quando usarlo | Note |
|---|---|---|
| `FileReader` + path assoluto | Dati dinamici (cambiano a runtime) | Fragile: path dipende dalla macchina |
| `resources` + `getResourceAsStream` | Dati statici, configurazioni iniziali | Incluso nel JAR, read-only |

**Dipendenza Gradle da aggiungere:**
```groovy
dependencies {
    implementation("com.google.code.gson:gson:2.13.2")
}
```

---

## 6. Override di `equals`

Nel sistema universitario, **due oggetti rappresentano lo stesso studente se hanno la stessa matricola**.

```java
@Override
public boolean equals(Object obj) {
    if (this == obj) return true;                      // 1. stesso oggetto in memoria → uguale
    if (!(obj instanceof Studente)) return false;      // 2. non è uno Studente → diverso
    Studente other = (Studente) obj;                   // 3. cast: ora posso usarlo come Studente
    return this.matricola == other.matricola;           // 4. confronto il campo che definisce l'identità
}
```

### Spiegazione riga per riga

1. **`if (this == obj) return true`** — ottimizzazione: se è proprio lo stesso riferimento, sono uguali per definizione
2. **`if (!(obj instanceof Studente)) return false`** — se non è uno Studente, non possono essere uguali
3. **`Studente other = (Studente) obj`** — **cast**: a questo punto sappiamo che è uno Studente, quindi possiamo trattarlo come tale
4. **`return this.matricola == other.matricola`** — la matricola è l'identità dello studente nel dominio

> In Java **tutti gli oggetti derivano dalla classe `Object`**. Per questo `equals` riceve un parametro di tipo `Object` (può rappresentare qualsiasi oggetto) e non direttamente `Studente`.

### Risultato dopo l'override

```java
System.out.println(studenteDaJsonFS.equals(studenteDaJsonResources));  // true ✅
System.out.println(studenteDaJsonFS.equals(studente1));                  // true ✅
System.out.println(studente1.equals(studenteDaJsonResources));           // true ✅
```

---

## 7. Effetti di `equals` sulle Collezioni

`equals` non serve solo per il confronto diretto: viene usato **internamente da `List`** nei metodi `contains`, `remove`, `indexOf`.

```java
Studente s1 = new Studente("Anna", "Rossi", 1001);
Studente s2 = new Studente("Anna", "Rossi", 1001);  // oggetto diverso, stessa matricola

List<Studente> iscritti = new ArrayList<>();
iscritti.add(s1);
```

| Operazione | Senza `equals` | Con `equals` |
|---|---|---|
| `iscritti.contains(s2)` | `false` | `true` ✅ |
| `iscritti.remove(s2)` + `size()` | `1` (non rimuove) | `0` ✅ (rimuove) |
| `iscritti.indexOf(s2)` | `-1` (non trovato) | `0` ✅ |

> **Senza `equals`**: la lista non riconosce `s2` come lo stesso studente di `s1`.  
> **Con `equals`**: la lista capisce che hanno la stessa identità nel dominio.

---

## 8. Modello vs Implementazione

```
Modello del dominio:          esiste UN solo studente con matricola 1001
Implementazione in Java:      possono esistere MOLTI oggetti con matricola 1001

equals = ponte tra i due mondi
```

`equals` serve a collegare il **modello reale** (il dominio universitario) con il **codice** (gli oggetti Java).

---

## 9. Quando implementare `equals`?

> **Regola pratica:** implementa `equals` quando una classe ha un **identificatore univoco e stabile** che definisce l'identità nel dominio.

| Classe | Identificatore | Ha senso `equals`? |
|---|---|---|
| `Studente` | `matricola` (final) | ✅ Sì |
| `Corso` | nome? codice corso? | ✅ Probabilmente sì |
| `Esame` | ? | ✅ Valutare |
| `EsameSuperato` | — | ⚠️ Dipende |

> Implementiamo `equals` solo quando abbiamo un'identità **chiara e stabile**.

---

## 10. Punti Chiave per l'Esame ✅

- **`List` non impedisce duplicati** — aggiungere lo stesso oggetto (o due oggetti "equivalenti") è sempre permesso
- **`==`** confronta gli **indirizzi in memoria** (identità di riferimento), non i dati
- **`equals` di default** usa lo stesso criterio di `==` → restituisce `false` anche per due oggetti con gli stessi dati
- **Override di `equals`**: permette di definire **quando due oggetti rappresentano la stessa entità** nel dominio
- **Struttura dell'override**: `this == obj` → `instanceof` → cast → confronto campi identificativi
- **`Object`**: in Java tutte le classi ereditano da `Object`; per questo `equals` riceve `Object obj` come parametro
- **Gson**: `gson.fromJson(reader, Studente.class)` ricrea un oggetto da JSON — senza `equals` non viene riconosciuto come uguale a uno già in memoria
- **`resources` vs `FileReader`**: usa `resources` per dati statici/configurazioni (inclusi nel JAR), `FileReader` per dati dinamici
- **Effetti sulle collezioni**: `contains`, `remove`, `indexOf` di `List` usano internamente `equals`
- **Esercizio a casa**: implementare `equals` anche sulle altre classi del sistema universitario che hanno un identificatore univoco
