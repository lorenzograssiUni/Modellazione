# 02 - Bash, LaTeX, Overleaf, Diagrammi di Flusso e Pseudocodice

---

## 1. Programmazione Procedurale — Riepilogo

> "Il programma dice passo passo cosa la macchina deve fare."

Caratteristiche:
- È la metodologia di base che già si conosce
- Il programma è una **sequenza di istruzioni** eseguite in ordine
- Il **controllo del flusso è esplicito**
- Esempi reali: script Bash/PowerShell, fogli Excel con macro, piccoli programmi in C o Python

> **Principio chiave:** Non basta che funzioni! Due programmi possono produrre lo stesso risultato oggi, ma quello organizzato meglio costerà minuti da modificare, l'altro ore o giorni.

---

## 2. Shell e Bash

### Cos'è una Shell?

Una **shell** è un programma che permette all'utente di comunicare con il sistema operativo.

**Funzionamento:**
- Legge i comandi dell'utente
- Li interpreta
- Chiede al kernel di eseguirli

**Schema concettuale:**
```
Utente → Shell → Kernel → Hardware
```

### Cos'è Bash?

**Bash** (Bourne Again SHell) è una specifica shell Unix/Linux. È allo stesso tempo:
- Una **shell** (interprete interattivo)
- Un **linguaggio di scripting**
- L'**interprete più diffuso** su Linux

| Sistema | Shell equivalente |
|---|---|
| Linux / macOS | **Bash** |
| Windows | CMD, PowerShell |

### Script Bash — Struttura base

Uno script Bash è un programma scritto nel linguaggio di scripting Bash ed eseguito dall'interprete Bash.

```bash
#!/bin/bash
# La prima riga (shebang) indica l'interprete da usare

numero=0
for i in {1..10}
do
    numero=$((numero + 1))
done
echo "Il valore finale è: $numero"
```

**Versione non strutturata (equivalente, ma da evitare):**
```bash
#!/bin/bash
numero=0
numero=$((numero + 1))
numero=$((numero + 1))
# ... ripetuto 10 volte
echo "Il valore finale è: $numero"
```

> 🔗 Per provare online: https://www.onlinegdb.com/online_bash_shell

### Sintassi Bash essenziale

| Costrutto | Esempio | Significato |
|---|---|---|
| Variabile | `nome="Mario"` | Assegnazione (no spazi!) |
| Lettura input | `read variabile` | Legge da tastiera |
| Stampa | `echo "testo $var"` | Output a schermo |
| Aritmetica | `$((a + b))` | Operazione matematica |
| Condizione | `if [ cond ]; then ... fi` | Selezione |
| Ciclo | `for i in {1..10}; do ... done` | Iterazione |
| Confronto numeri | `-ge`, `-le`, `-eq`, `-ne`, `-gt`, `-lt` | Operatori di confronto |
| Confronto stringhe | `=`, `!=` | Uguaglianza stringhe |
| AND logico | `&&` | Entrambe le condizioni vere |

---

## 3. LaTeX e Overleaf

### Cos'è LaTeX?

LaTeX è il **modo in cui gli informatici scrivono documenti "seri"** — è letteralmente il modo di *programmare un PDF*.

- **Non è Word**: non si trascina nulla, si scrive codice che viene compilato
- Usato per **articoli scientifici, tesi, documentazione tecnica**
- Produce output di altissima qualità tipografica, specialmente per **formule matematiche**

**Distinzione tra i componenti:**

| Componente | Ruolo |
|---|---|
| **LaTeX** | Il linguaggio (comandi e sintassi) |
| **TeX** | Il motore di composizione tipografica |
| **Overleaf** | L'ambiente online dove scrivere e compilare |

### Overleaf CE

[Overleaf](https://www.overleaf.com) è un editor LaTeX online:
- Nessuna installazione locale necessaria
- Compilazione istantanea nel browser
- Collaborazione in tempo reale (come Google Docs per LaTeX)
- Utilizzato per la stesura della **tesi di laurea**

### Struttura base di un documento LaTeX

```latex
\documentclass{article}   % tipo di documento

\usepackage[utf8]{inputenc}  % pacchetto per caratteri speciali
\usepackage[italian]{babel}  % lingua italiana

\title{Il mio documento}
\author{Nome Cognome}
\date{\today}

\begin{document}

\maketitle  % genera titolo, autore, data

\section{Introduzione}
Questo è un paragrafo di testo.

\subsection{Sottosezione}
Alcuni concetti:
\begin{itemize}
    \item Primo punto
    \item Secondo punto
\end{itemize}

% Formula matematica
La formula è: $E = mc^2$

\end{document}
```

---

## 4. Diagrammi di Flusso

### Definizione

Un **diagramma di flusso** è una rappresentazione grafica di un processo o algoritmo. Mostra visivamente le sequenze di azioni, decisioni e percorsi possibili.

**A cosa serve:**
- Visualizzare un processo in modo chiaro
- Analizzare e migliorare procedure
- Progettare algoritmi informatici
- Facilitare la comunicazione tra persone o team

**Storia:**
- Anni '20: introdotto da Frank e Lillian Gilbreth per analizzare i processi industriali
- Anni '40–'50: adottato nell'informatica per rappresentare algoritmi e programmi

### Simboli principali

| Simbolo | Forma | Significato |
|---|---|---|
| Inizio / Fine | Ovale (ellisse) | Punto di start o stop del programma |
| Istruzione / Azione | Rettangolo | Un'operazione da eseguire |
| Decisione | Rombo | Condizione con rami Sì/No |
| Input / Output | Parallelogramma | Lettura dati o stampa risultato |
| Flusso | Freccia | Direzione di esecuzione |
| Procedura (chiamata) | Rettangolo con doppie linee laterali | Chiamata a una sottoprocedura |

---

## 5. Pseudocodice

### Definizione

Il **pseudocodice** è un modo semplice e strutturato per descrivere la logica di un algoritmo, **senza usare la sintassi rigida di un linguaggio di programmazione**.

**Serve per:**
- Capire il problema
- Progettare la soluzione
- **Pensare prima di scrivere codice**

### Struttura tipo

```
INIZIO
  LEGGI dati
  SE condizione ALLORA
    istruzioni
  ALTRIMENTI
    istruzioni
  FINE SE
  SCRIVI risultato
FINE
```

### Parole chiave del pseudocodice

| Parola chiave | Significato |
|---|---|
| `INIZIO` / `FINE` | Delimitano l'algoritmo |
| `LEGGI` | Lettura di un dato in input |
| `SCRIVI` | Output di un risultato |
| `SE ... ALLORA ... ALTRIMENTI ... FINE SE` | Selezione condizionale |
| `MENTRE ... FARE ... FINE MENTRE` | Ciclo con condizione in testa |
| `RIPETI ... FINCHÉ` | Ciclo con condizione in coda |
| `PER i DA ... A ... FARE` | Ciclo con contatore |
| `←` | Assegnazione di valore |

---

## 6. Esercizi Tipo Esame

### Esercizio 1 — Somma da 1 a N

**Traccia:** Algoritmo che legge un numero intero positivo N, calcola la somma di tutti i numeri da 1 a N e stampa il risultato.

**Pseudocodice:**
```
INIZIO
  LEGGI N
  i ← 1
  somma ← 0
  MENTRE i <= N FARE
    somma ← somma + i
    i ← i + 1
  FINE MENTRE
  SCRIVI somma
FINE
```

**Traduzione in Bash:**
```bash
#!/bin/bash
echo "Inserisci N:"
read N
somma=0
i=1
while [ $i -le $N ]; do
    somma=$((somma + i))
    i=$((i + 1))
done
echo "Somma: $somma"
```

### Esercizio 2 — Verifica superamento esame (con procedura)

**Versione con procedura separata (pseudocodice):**
```
Procedura CalcolaSomma(N)
  i ← 1
  somma ← 0
  MENTRE i <= N FARE
    somma ← somma + i
    i ← i + 1
  FINE MENTRE
  SCRIVI somma
Fine Procedura

Main
  LEGGI N
  CalcolaSomma(N)
Fine Main
```

### Esercizio 3 — Verifica esame universitario

**Traccia:** Determinare se uno studente ha superato l'esame. Input: `voto_scritto` e `voto_progetto`. Entrambi devono essere >= 18.

**Pseudocodice (versione semplice):**
```
INIZIO
  LEGGI voto_scritto
  LEGGI voto_progetto
  SE voto_scritto >= 18 E voto_progetto >= 18 ALLORA
    SCRIVI "Esame superato"
  ALTRIMENTI
    SCRIVI "Esame non superato"
  FINE SE
FINE
```

**Pseudocodice (con calcolo voto finale):**
```
INIZIO
  LEGGI voto_scritto
  LEGGI voto_progetto
  SE voto_scritto >= 18 E voto_progetto >= 18 ALLORA
    voto_finale ← voto_scritto * 0.7 + voto_progetto * 0.3
    SCRIVI "Esame superato"
    SCRIVI "Voto finale: ", voto_finale
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
    voto_finale=$(( (voto_scritto * 7 + voto_progetto * 3) / 10 ))
    echo "Esame superato"
    echo "Voto finale: $voto_finale"
else
    echo "Esame non superato"
fi
```

> ⚠️ In Bash non si possono usare i decimali direttamente → si moltiplica per 10 e si divide alla fine.

**Traduzione in C:**
```c
#include <stdio.h>

int main() {
    float voto_scritto;
    float voto_progetto;
    float voto_finale;

    printf("Inserisci voto scritto: ");
    scanf("%f", &voto_scritto);
    printf("Inserisci voto progetto: ");
    scanf("%f", &voto_progetto);

    if (voto_scritto >= 18 && voto_progetto >= 18) {
        voto_finale = voto_scritto * 0.7 + voto_progetto * 0.3;
        printf("Esame superato\n");
        printf("Voto finale: %.2f\n", voto_finale);
    } else {
        printf("Esame non superato\n");
    }

    return 0;
}
```

> 🔗 Per provare online C: https://www.onlinegdb.com/online_c_compiler

---

## 7. Relazione tra Pseudocodice, Diagramma di Flusso e Codice

```
Problema (linguaggio naturale)
        ↓
   Pseudocodice          Diagramma di Flusso
   (testo strutturato)   (rappresentazione grafica)
        ↓                       ↓
         Codice (Bash, C, Java, Python...)
```

> Pseudocodice e diagramma di flusso **sono equivalenti** — descrivono lo stesso algoritmo in forme diverse. Scegliere quale usare dipende dal contesto e dalla preferenza.

---

## 8. Punti Chiave per l'Esame ✅

- **Shell** = intermediario tra utente e kernel; **Bash** = shell specifica di Linux/Unix
- La prima riga di uno script Bash è `#!/bin/bash` (shebang)
- In Bash: niente spazi nell'assegnazione (`a=5`, non `a = 5`)
- In Bash: confronti numerici con `-ge`, `-le`, `-eq`; confronti stringhe con `=`
- **LaTeX** = linguaggio; **TeX** = motore; **Overleaf** = editor online
- LaTeX si usa per documenti tecnici/scientifici e per la **tesi di laurea**
- Il **diagramma di flusso** mostra un algoritmo in forma grafica con simboli standard (ovale, rettangolo, rombo, parallelogramma)
- Lo **pseudocodice** descrive un algoritmo in forma testuale strutturata, indipendente dal linguaggio
- Le parole chiave dello pseudocodice: `INIZIO`, `FINE`, `LEGGI`, `SCRIVI`, `SE...ALLORA...ALTRIMENTI`, `MENTRE...FARE`, `←`
- **Procedura nel diagramma** = rettangolo con doppie linee laterali; nel pseudocodice si definisce con `Procedura NomeProcedura(parametri)`
- Bash non supporta i float nativamente per l'aritmetica → si usano trucchi (moltiplicare per 10, poi dividere)
