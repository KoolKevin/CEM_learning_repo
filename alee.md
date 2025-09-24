Risultato finale è che riusciamo ad eseguire in parallelo 5 PORZIONI di istruzioni
- il CPI medio risulta 1 (se la pipeline è piena e non ci sono stalli)
- CPI medio si calcola come #cycles/#exe_instr
- tende a 1 quando la pipeline è piene e non ci sono stalli
    - CPI sempre >= 1
- notiamo che la singola istruzione ha latenza pari a 5 clock
    - pipeline più lunghe hanno anche latenze più lunghe, di conseguenza il cpi medio tende sempre a 1
    - il vantaggio di pipeline più lunghe è che permettono una frequenza più alta (ogni stadio deve fare meno roba)
    - di conseguenza (iron law) i tempi di esecuzione diminuiscono

ILP possibile solo in assenza di alee


# Alee di dato

### Forwarding
quando abbiamo alee di dato del tipo RAW, se non facciamo niente, la seconda istruzione deve bloccarsi nello stadio ID fino a che la prima istruzione non ha completato il suo stadio di WB, nonostante il dato sia pronto dopo lo stadio EX di quest'ultima.

--- 

NB: non c'è bisogno di aspettare dopo lo stadio EX; l'ID della seconda istruzione può essere contemporaneo dato che le letture del registrer file vengono fatte nel secondo semiperiodo dopo che le scritture sono già state effettuate.

come facciamo ad accorgerci che dobbiamo stallare?
- in questo esempio di RAW, ho bisogno di una logica che mi confronta il valore dei registri di istruzioni presenti in stadi diversi della pipeline (queste informazione sono presenti)
    - id/ex.rs1,rs2 == ex/mem.ra || id/ex
    - notiamo che di nuovo c'è bisogno di propagare dei valori intermedi nella pipeline nei registri buffer degli stadi

come si stalla?
- ripeto lo stadio

---

L'idea del forwarding è quella di fornire alla seconda istruzione il risultato che è già presente nello stadio di ex
- riusciamo a risolvere anche una RAW ritardata di un'istruzione, basta aggiungere un altro fowrwarding tra mem/wb e id/ex

non riusciamo però a risolvere tutte le possibili alee di dato con logica di forwarding. In particolare quando il dato lo dobbiamo prelevare dalla memoria
- questo perchè all'interno della nostra pipeline l'accesso alla memoria lo facciamo dopo lo stadio di execute (**CAPISCI MEGLIO**)
- questo problema è intrinseco nella microarchitettura che si è deciso di utilizzare (tradeoff)





slide 39, contiamo 9 cicli e non 13 dato che assumiamo che il codice sia già in mezzo a dell'altro codice, in altri consideriamo la pipelline piena.


riordiniamo le istruzioni per diminuire stalli (e CPI)
- notiamo che queste ottimizzazioni sono compito del compilatore ma dipendono dalla **microarchitettura**
- in generale l'ottimizzazione basasata sul riordinamento del codice ha due varianti: questa è un ottmizzazione basata su scheduling statico. Esiste anche uno scheduling dinamico (fatto a runtime) nelle microarchitetture fuori ordine.



# Alee di controllo
la pipeline non può fare il fetch di un'istruzione corretta


ci sono due aspetti da considerare nel caso in cui si ha un alea di controllo
- aspetto probabilistico, nel mio codice è più facile saltare o no?
- aspetto microarchitetturale, è più facile assumere sempre preso o non preso?

Tralasciamo l'aspetto probabilistico
- se assumo sempre preso
    - la prossima fetch deve comunque stallare due cicli in attesa che venga calcolato l'indirizzo del salto
- se assumo sempre non preso
    - conosco l'esito del salto sempre dopo lo stadio di execute dell'istruzione precedente
    - se ci azzecco tutto a posto
    - se non ci azzecco, devo trasformare l'istruzione in una nop, ed eseguire l'IF all'indirizzo giusto
        - di fatto pago lo stesso prezzo di stallare due cicli

Questa è una logica di branch prediction statica (prendo sempre la stessa decisione). Posso fare anche branch prediction dinamica considerando la storia passata del codice che si sta eseguendo!

Inoltre, posso predire anche quale sarà l'indirizzo di salto (esempio di loop)
- mi basta aggiungere una piccola cache in cui memorizzo l'indirizzo di salto precedente




Lo stato del processore definisce quando un'istruzione diventa irreversibile
- questo accade quando sovrascrivo register file / memoria
- quindi negli stadi di MEM e WB


