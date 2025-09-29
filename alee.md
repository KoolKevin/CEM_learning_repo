Grazie al pipeling, il risultato finale è che riusciamo ad eseguire in parallelo 5 PORZIONI di istruzioni
- il CPI medio risulta 1 (se la pipeline è piena e non ci sono stalli)
- CPI medio si calcola come #cycles/#exe_instr
- tende a 1 quando la pipeline è piene e non ci sono stalli
    - CPI sempre >= 1
- notiamo che la singola istruzione ha latenza pari a 5 clock
    - pipeline più lunghe hanno anche latenze più lunghe, di conseguenza il cpi medio tende sempre a 1
    - il vantaggio di pipeline più lunghe è che permettono una frequenza più alta (ogni stadio deve fare meno roba)
    - di conseguenza (iron law) i tempi di esecuzione diminuiscono

ILP possibile solo in assenza di alee

# Alee strutturali
quando un'unità funzionale viene usata da più istruzioni contemporaneamente.

abbiamo già visto degli esempi prima





# Alee di dato
Un istruzione dipende dal termine della lettura/scrittura di un'altra istruzione

### Forwarding
quando abbiamo alee di dato del tipo RAW, se non facciamo niente, la seconda istruzione deve bloccarsi nello stadio ID fino a che la prima istruzione non ha completato il suo stadio di WB, nonostante il dato sia pronto dopo lo stadio EX di quest'ultima.
- NB: non c'è bisogno di aspettare dopo lo stadio EX; l'ID della seconda istruzione può essere contemporaneo dato che le letture del RF vengono fatte nel secondo semiperiodo, dopo che le scritture sono già state effettuate.

L'idea del forwarding è quella di fornire alla seconda istruzione il risultato che è già presente nello stadio di ex
- Use result when it is computed
    - Don’t wait for it to be stored in a register
    - Requires extra connections in the datapath
- questo si traduce nell'salvare nel registro buffer EX/MEM alu-output, e riutilizzare questo valore nello stadio di execute della prossima istruzione
- riusciamo a risolvere anche una RAW ritardata di un'istruzione, basta aggiungere un altro fowrwarding tra mem/wb e id/ex

In sostanza il valore di un registro sorgente non viene prelevato dal RF (dato che non è ancora aggiornato), piuttosto viene preso da uno stadio intermedio della pipeline

**come facciamo ad accorgerci che dobbiamo stallare?**
- in questo esempio di RAW, ho bisogno di una logica che mi confronta il valore dei registri di istruzioni presenti in stadi diversi della pipeline (queste informazione sono presenti)
    - id/ex.rs1(o rs2) == ex/mem.rd || mem/wb.rd 
        - uno dei miei registri sorgente è il registro destinazione di un'istruzione più in fondo nella pipeline
    - notiamo che di nuovo c'è bisogno di propagare dei valori intermedi nella pipeline nei registri buffer degli stadi

come si stalla?
- possiamo modificare l'opcode dell'istruzione da stallare per renderla una nop
    - le nop non modificano lo stato del processore (non scrivono mem e RF)
- inoltre, blocco lo stadio di IF in maniera da non modificare il PC
- e infine rieseguo lo stadio di ID

oppure
- posso prevedere un segnale di handshake tra i registri della pipeline che mi dicono se uno stadio a valle è ready o meno

Non riusciamo però a risolvere tutte le possibili alee di dato con logica di forwarding.
- in particolare quando il dato lo dobbiamo prelevare dalla memoria. es load after use: sub dopo una ld
- in questo caso anche facciamo un forwarding del risultato della load da mem/wb a id/ex, la seconda istruzione deve comunque stallare per un ciclo dato che il risultato è presente solo dopo che è terminato lo stadio di memory
- questo problema è intrinseco nella microarchitettura che si è deciso di utilizzare (tradeoff)

Nota: slide 39, contiamo 9 cicli e non 13 dato che assumiamo che il codice sia già in mezzo a dell'altro codice, in altri consideriamo la pipelline piena.


è possibile riordinare le istruzioni per diminuire stalli (e CPI)
- notiamo che queste ottimizzazioni sono compito del compilatore ma dipendono dalla **microarchitettura**
- in generale l'ottimizzazione basasata sul riordinamento del codice ha due varianti:
    - questa è un ottmizzazione basata su scheduling statico. 
    - Esiste anche uno scheduling dinamico (fatto a runtime) nelle microarchitetture fuori ordine.



# Alee di controllo
Quando eseguiamo una branch, il prossimo PC non si conosce fino al termine dello stadio di EX
- al prossimo ciclo di clock la pipeline non può quindi fare il fetch dell'istruzione corretta in sicurezza 

Una soluzione alle alee di controllo ingenua è quella di stallare
- Questa soluzione però non scala con pipeline più lunghe
- Pipeline corta (5 stadi): se sbagli un salto, devi svuotare (trasformare in nop) solamente un'istruzione.
- Pipeline lunga (20 stadi): se sbagli, devi svuotare molteplici istruzioni.
    - Qui la lunghezza moltiplica la gravità dello stallo.

ci sono due aspetti da considerare nella gestione di un alea di controllo tramite prediction
- aspetto probabilistico, nel mio codice è più facile saltare o no?
- aspetto microarchitetturale, è più facile assumere sempre preso o non preso?

Tralasciamo l'aspetto probabilistico
- se assumo sempre preso
    - la prossima fetch deve comunque stallare due cicli in attesa che venga calcolato l'indirizzo del salto
- se assumo sempre non preso
    - il prossimo indirizzo è PC+4 e quindi non devo stallare se azzecco la mia prediction
    - l'esito del salto sempre dopo lo stadio di execute dell'istruzione precedente
    - se non ci azzecco, devo trasformare l'istruzione in una nop, ed eseguire l'IF all'indirizzo giusto
        - di fatto pago lo stesso prezzo di stallare due cicli

Questa è una logica di branch prediction statica (prendo sempre la stessa decisione). Posso fare anche branch prediction dinamica considerando la storia passata del codice che si sta eseguendo!

Inoltre, posso predire anche quale sarà l'indirizzo di salto (esempio di loop)
- mi basta aggiungere una piccola cache in cui memorizzo l'indirizzo di salto precedente
- in questa maniera non devo aspettare 2 cicli che venga calcolato l'indirizzo target

Se sbaglio una prediction, devo trasformare l'istruzione in una nop in modo da non modificare lo stato del processore. La prossima fetch verrà fatta all'indirizzo giusto

Lo stato del processore definisce quando un'istruzione diventa irreversibile
- questo accade quando sovrascrivo register file / memoria
- quindi negli stadi di MEM e WB


### Pipeline Summary
- Pipelining improves performance by increasing instruction throughput
    - Executes multiple instructions in parallel
    - utilizes (or tries) all of the hardware at the same time
    - Each instruction has the same latency
- Subject to hazards
    - Structure, data, control
- Instruction set design affects complexity of pipeline implementation



