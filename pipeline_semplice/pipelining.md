L'esecuzione di tutte le istruzioni può essere scomposta in 5 passi, ciascuno eseguito in uno o più cicli di clock.

Tali passi sono detti:

1. FETCH: l'istruzione viene prelevata dalla memoria e posta in IR.
2. DECODE: l'istruzione in IR viene decodificata e vengono prelevati gli operandi sorgente dal Register File.
3. EXECUTE: elaborazione aritmetica o logica mediante la ALU.
    - per le load/store, qua si calcola l'indirizzo
    - per le branch, qua si fa la comparazione
4. MEMORY: accesso alla memoria e, nel caso di BRANCH aggiornamento del PC (branch completion).
5. WRITE-BACK: scrittura sul Register File.

(istruzioni in grigio dipendono da ottimizzazioni della pipeline)

... varie operazioni che vengono eseguite nei vari stadi ...

**NB**: Una cosa importante da capire con il pipelining è che man mano che le istruzioni viaggiano nella pipeline, esse vengono accompagnate da vari valori intermedi che rappresentano una sorta di stato dell'istruzione all'interno della pipeline, e che servono all'esecuzione corretta dell'istruzione

- e.g. A e B vengono generati nello stadio dell'instruction decode; oppure, per un'istruzione di load, l'informazione di quanti byte devo leggere viene propagata a partire dallo stadio di ID fino allo stadio MEM
- la pipeline ha quindi tanti registri buffer per questi valori intermedi
- in questo modo ogni istruzione è accompagnata dal suo stato e non da quello di un'altra istruzione

concetto di critical path

considera la slide 7 in cui non ho registri buffer, qua non posso andare a 5 istruzioni alla volta dato che il valore dei fili non viene salvato per tutto il tempo necessario allo stadio per eseguire

- critical path lungo

In una microarchitettura a pipeline andiamo ad inserire dei registri buffer tra i vari stadi in maniera tale da mantenere stabili per l'intero ciclo di clock i valori intermedi per lo stadio

- critical path corto

Tutte le informazioni necessarie per l'esecuzione di uno stadio deve essere presente nel registro buffer sorgente.

- ad esempio, nello stadio di execute viene anche salvato il tipo dell'istruzione da eseguire (c'è un mux), informazione che avevo scritto nello stadio id ID

Nota: in WB si accede al RF per sovrascrivere il valore di un registro con il risultato dell'istruzione, di conseguenza l'informazione su qual'è il registro destinazione deve essere propagata nella pipeline altrimenti scriveremmo il registro di un'altra istruzione

Nota: è possibile accedere al register file in lettura e scrittura contemporaneamente (e.g. vedi clock 5 slide 28)

- in particolare, il RF viene condiviso dallo stadio di ID e WB
- questa è un'alea strutturale ed è un problema.
- il register file deve permettere accessi contemporanei, in particolare si fa accedere in scrittura nel primo semiperiodo di clock; in lettura nel secondo semiperiodo

Nota: anche con la memoria abbiamo un alea strutturale

- accediamo contemporaneamente in IF e MEM (instruction memory e data memory)
- la RAM e una e quindi non possiamo indirizzarla con due indirizzi diversi contemporaneamente
- risolviamo utilizzando una instruction cache separata dalla cache per i dati. Di fatto è come se avessimo due memorie separate
