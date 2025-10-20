Nel primo esempio con scoreboarding, la fsub non riesce a completare issue al ciclo 3 dato che trova l'unità funzionale occupata nella entry dello scoreboard, nonostante la FU non sia effettivamente occupata.

- **alea strutturale sull'unità scoreboard**, non sulla FU

Secondo esempio

Nel terzo esempio siamo obbligati a eseguire le istruzioni in quell'ordine a causa del dataflow. Rinominando i registri risolviamo le dipendenze di nome e possiamo eseguire fuori ordine

# Algoritmo di Tommasulo

Introduces register renaming in hardware

rimuove il FU status register e lo sostituisce con il concetto di Reservation Station

- gli operandi vengono immagazzinati nella reservation station appena disponibili
- l'id della reservation station può anche dirmi la sorgente di un operando
  - reservation station implementano register renaming

Ogni unità funzionale ha associata una reservation station che fa da buffer per le istruzioni

Ogni reservation station è una tabella formata da:

- Busy: questa indica **se la reservation station è libera**, NON la FU. questo campo viene utilizzato durante issue
- Op
- qj, qk
- vj, vk: valori degli operandi

Ogni entry nel RF ha un Register Result Status associato analogo a quello dell'algoritmo di scoreboarding

i risultati prodotti dalle FU vengono fatti viaggere sul Common Data Bus

il CDB entra in input alle reservation station e al RF

sul CDB i dati viaggiano con una forma <tag, val>

- il tag indica chi ha prodotto il risultato con valore val
- chiunque stava attendendo un operando () lo aspetta da un entità indicata dal suo tag

Passo da una modalità in cui gli operandi venivano sempre letti dal RF, ad una modalità in cui leggo dal RF in issue e, in caso non siano pronti in quello stadio, leggo dal CDB

Sto facendo renaming degli operandi, con il nome del tag

Durante l'issue si segna la entry del RF relativa al registro destinazione dell'istruzione, il tag della reservation station in cui verranno parcheggiata l'istruzione da eseguire. In questo modo si marchia la entry nel RF come invalida (esattamente come avveniva nel register result status dello scoreboarding)

slide 9 -> IS al posto di ID

Nel caso della memory unit abbiamo un po' di casino

- in generale, le store devono essere eseguite in program order
  - altrimenti WAW
- le load e le store devono essere eseguite seguendo l'ordine reciproco all'interno del programma
  - altrimenti WAR

load buffer ~= reservation station per l'istruzione di load

nel caso di istruzioni di store, oltre all'indirizzo con cui vogliamo accedere alla memoria, abbiamo anche il valore che vogliamo scrivere.

Questo potrebbe essere una entry nel RF, oppure tag per reservation station che mi produrranno il risultato

l'ordine delle istruzioni di load/store viene serializzato dalla address unit

- l'ordine del programma viene rispettato

### implementazione hardware delle reservation station [16:10 - 20/10]

quello che sembra un mini FFD rappresenta il reset
