# 12 - Clean Code — Part 2

---

## 1. Recap: i principi Clean Code della Lezione 11

| Principio | Significato |
|---|---|
| Nomi significativi | Usare nomi chiari e descrittivi; evitare `v`, `s`, `p` |
| Funzioni piccole | Metodi brevi, una sola responsabilità, 0-2 argomenti |
| Code as Prose | Il codice si legge come una frase in italiano |

---

## 2. Clean Code: Organizzazione Verticale del Codice

> **Il codice correlato deve stare vicino e seguire un flusso logico dall'alto verso il basso.**

```java
// ❌ Sbagliato: variabile usata lontano da dove è dichiarata
int voto = valutatore.assegnaVoto(studente);
// ... 50 righe di codice ...
String votoString = formattaVoto(voto);  // voto dichiarato lontano

// ✅ Corretto: dichiarazione e uso vicini
int voto = valutatore.assegnaVoto(studente);
String votoString = formattaVoto(voto);  // subito dopo
```

**Regola:** la funzione che *chiama* sta sopra, quella *chiamata* sta sotto.

```java
// In alto: logica principale
public void sostieniEsame(Studente studente) {
    stampaEsito(studente, voto);  // ← chiama stampaEsito
}

// In basso: i dettagli
private void stampaEsito(Studente studente, int voto) { ... }  // ← definita dopo
```

---

## 3. Clean Code: DRY — Don't Repeat Yourself

> **Non ti ripetere. La duplicazione è uno dei problemi più pericolosi nel codice.**

### Il problema: logica duplicata

```java
// In Esame:
private String formattaVoto(int voto) {
    return (voto == 31) ? "30 e lode" : String.valueOf(voto);
}

// In Libretto (stessa logica!):
String votoString = (voti.get(i) == 31) ? "30 e lode" : String.valueOf(voti.get(i));
```

`voto == 31 ? "30 e lode" : ...` è una **regola del dominio** — è duplicata — deve stare in **un solo punto**.

### La soluzione: classe di utilità

Una **classe di utilità** contiene metodi generici riutilizzabili che non appartengono a un oggetto specifico.

```java
// Centralizzare la logica in un unico posto
public class FormattatoreVoto {
    public static String formatta(int voto) {
        return (voto == 31) ? "30 e lode" : String.valueOf(voto);
    }
}
// Il metodo è static: appartiene alla classe, non all'oggetto
// Si usa: FormattatoreVoto.formatta(voto)  -- NO: new FormattatoreVoto()
```

Così facendo, se domani cambia la regola del voto (es. la lode diventa 32), si modifica **un solo punto**.

> **Errori comuni:**
> - Istanziare la classe: `FormattatoreVoto f = new FormattatoreVoto();` → sbagliato, usa `static`
> - Mettere `FormattatoreVoto` in un package a caso → organizzare il codice in modo coerente

---

## 4. Clean Code: Data Clumping (Raggruppamento di dati)

> **Se due dati viaggiano sempre insieme, probabilmente sono un oggetto.**

### Il problema: due liste parallele

```java
// In Libretto:
private final List<Esame> esamiSuperati;
private final List<Integer> voti;

public void registraEsameSuperato(Esame esame, int voto) {
    esamiSuperati.add(esame);
    voti.add(voto);
}
```

`esamiSuperati[0]` e `voti[0]` fanno riferimento allo stesso esame — sono **liste parallele** legate dall'indice. Problemi:
- Rischio di **inconsistenza** (un indice disallineato e il dato è sbagliato)
- Relazione **implicita**, non visibile nel codice
- Codice meno leggibile

### La soluzione: creare un oggetto che rappresenta il concetto

```java
public class EsameSuperato {
    private final Esame esame;
    private final int voto;

    public EsameSuperato(Esame esame, int voto) {
        // ← i controlli vanno qui (vedi Error Handling)
        this.esame = esame;
        this.voto = voto;
    }

    public Esame getEsame() { return esame; }
    public int getVoto() { return voto; }
}
```

Libretto aggiornato:

```java
public class Libretto implements CalcolatoreMedia, VisualizzatoreEsami {
    private final Studente studente;
    private List<EsameSuperato> esamiSuperati = new ArrayList<>();

    public void registraEsameSuperato(Esame esame, int voto) {
        esamiSuperati.add(new EsameSuperato(esame, voto));  // i controlli sono in EsameSuperato
    }
}
```

---

## 5. Clean Code: Error Handling (Gestione degli Errori)

> **Valida subito. Fallisci subito. Messaggi chiari.**

I controlli devono essere **vicini ai dati che devono proteggere**.

```java
public EsameSuperato(Esame esame, int voto) {
    if (esame == null) {
        throw new IllegalArgumentException("Esame non valido");
    }
    if (voto < 18 || voto > 31) {
        throw new IllegalArgumentException("Voto non valido");
    }
    this.esame = esame;
    this.voto = voto;
}
```

> Non basta che il codice funzioni: deve anche **impedire stati sbagliati**.
> Il codice senza controlli può produrre stati non validi: `EsameSuperato(null, 18)` buildà — ma non è corretto!

---

## 6. Clean Code: Commenti

### Evitare commenti inutili ❌

```java
// stampa gli esami  ← inutile: il nome del metodo lo dice già!
stampaEsamiSuperati();

// Verifica se il dipendente ha diritto a tutti i benefit  ← meglio rinominare il metodo!
if ((employee.flags & HOURLY_FLAG) && (employee.age > 65))
```

Versione corretta (senza commento):
```java
if (employee.isEligibleForFullBenefits())  // si legge da solo!
```

### Aggiungere commenti utili ✅

I commenti sono utili quando **chiariscono il contratto implicito** che il nome non può esprimere:

```java
public interface Valutatore {
    /**
     * Restituisce un voto tra 18 e 30,
     * oppure 31 per la lode.
     */
    int assegnaVoto(Studente studente);
}
```

### Codice commentato: rimuoverlo! ❌

```java
// SBAGLIATO: codice commentato rimasto nel progetto
//    public void stampaEsamiSuperati() {
//        for (int i = 0; i < esamiSuperati.size(); i++) {
//            String votoString = (voti.get(i) == 31) ? "30 e lode" : ...
//        }
//    }

// VERSIONE NUOVA (quella che usa)
@Override
public void stampaEsamiSuperati() {
    System.out.println("Libretto di " + studente.getNomeCompleto());
    for (EsameSuperato es : esamiSuperati) {
        System.out.println(es.getEsame().getNome() + " - " + FormattatoreVoto.formatta(es.getVoto()));
    }
}
```

> Commentare il vecchio codice per "tornare indietro se va male" è comprensibile, ma il codice commentato **deve essere rimosso** quando non serve più.  
> Avete git per tornare a versioni precedenti: non servono commenti come backup!

---

## 7. Clean Code: KISS — Keep It Simple, Stupid

> **Privilegia la semplicità all'ingegnosità. Scrivi codice semplice e facile da comprendere.**

```java
// ❌ Inutilmente complesso
if (voto >= 18) {
    if (voto <= 30) {
        return voto;
    } else if (voto == 31) {
        return voto;
    } else {
        throw new IllegalStateException("Voto troppo alto");
    }
} else {
    throw new IllegalStateException("Voto insufficiente");
}

// ✅ Semplice e leggibile — stesso risultato
if (voto >= 18 && voto <= 31) {
    return voto;
}
throw new IllegalStateException("Voto non valido");
```

> Codice semplice non significa codice banale: significa codice che fa esattamente quello che serve, senza complessità aggiuntiva.

---

## 8. Riepilogo: tutti i principi Clean Code

| Principio | Significato | Obiettivo |
|---|---|---|
| **Nomi significativi** | Usare nomi chiari e descrittivi | Rendere il codice autoesplicativo |
| **Funzioni piccole** | Metodi brevi, una sola responsabilità | Facilitare lettura e comprensione |
| **Code as Prose** | Il codice si legge come una frase | Comprendere il flusso senza sforzo |
| **Organizzazione verticale** | Codice correlato vicino, flusso dall'alto verso il basso | Migliorare la leggibilità |
| **DRY** (Don't Repeat Yourself) | Evitare duplicazione | Ridurre errori e manutenzione |
| **Data Clumping** | Raggruppare dati correlati in oggetti | Migliorare coerenza e struttura |
| **Error Handling** | Gestire errori in modo chiaro e coerente | Rendere il codice affidabile |
| **Commenti utili** | Usare commenti solo quando necessario | Evitare rumore nel codice |
| **KISS** | Preferire soluzioni semplici | Ridurre complessità inutile |

---

## 9. Evoluzione del sistema universitario (fine lezione 12)

```
it.unicam.mdp2526.universita
├── model/
│   ├── Corso.java
│   ├── Esame.java              (usa FormattatoreVoto, dipende da Valutatore)
│   ├── EsameSuperato.java      (NUOVO: oggetto che raggruppa Esame + voto)
│   ├── Libretto.java           (usa List<EsameSuperato> invece di due liste parallele)
│   ├── Professore.java
│   └── Studente.java
├── interfaces/
│   ├── ApprovatorePianoDiStudi.java
│   ├── CalcolatoreMedia.java
│   ├── GestoreIscrizioni.java
│   ├── Valutatore.java
│   └── VisualizzatoreEsami.java
├── util/                        ← (o simile)
│   └── FormattatoreVoto.java    (NUOVO: classe di utilità, metodo static)
└── Main.java
```

---

## 10. Punti Chiave per l'Esame ✅

- **Organizzazione verticale:** funzione che chiama sta sopra, quella chiamata sta sotto; variabili dichiarate vicino a dove vengono usate
- **DRY:** ogni regola del dominio deve stare in **un solo punto**; `voto == 31 ? "30 e lode"` non deve essere copiato in più classi
- **Classe di utilità** (`FormattatoreVoto`): metodi `static`, non si istanzia, centralizza logica riutilizzabile; va messa in un package coerente (es. `util/`)
- **Data Clumping:** due liste parallele (`esamiSuperati[]` + `voti[]`) legate dall'indice = segnale di errore → creare un oggetto `EsameSuperato`
- **`EsameSuperato`**: contiene `Esame` + `int voto` immutabili (campi `final`), i controlli di validità sono nel costruttore
- **Error Handling:** `if (esame == null) throw new IllegalArgumentException(...)` — i controlli vicini ai dati, messaggio chiaro; `EsameSuperato(null, 18)` builda ma viola il contratto → va bloccato nel costruttore
- **Commenti inutili:** `// stampa gli esami` prima di `stampaEsamiSuperati()` = rumore; il nome del metodo è sufficiente
- **Commenti utili:** Javadoc su interfacce (es. cosa restituisce `assegnaVoto`) = documentazione del contratto implicito
- **Codice commentato:** da rimuovere sempre; git è il vero "backup", non i commenti
- **KISS:** se il codice è inutilmente complesso, semplificarlo; codice semplice non = codice banale
- `stampaEsamiSuperati()` dopo il refactoring usa `for (EsameSuperato es : esamiSuperati)` + `FormattatoreVoto.formatta(es.getVoto())` — esempio di tutti i principi applicati insieme
