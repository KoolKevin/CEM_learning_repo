Abbiamo visto che riusciamo ad avere ILP se non abbiamo alee che causano stalli
- in particolare, consideriamo alee di dato e di controllo
- da analisi si è scoperto che il numero di istruzioni che non hanno questo tipo di alee nel codice (basic block) è di 3-6 istruzioni ...
- dobbiamo migliorare

Per alleggerire l'effetto delle alee di controllo possiamo fare loop unrolling per distanziare le branche incrementando così la dimensione del basic block e, di conseguenza, ILP

Il loop unrolling può essere utilizzato anche per alleggerire l'effetto delle alee di dato dato che introduce delle istruzioni indipendenti aggiuntive che possono venire utilizzate per spaziare le istruzioni appartenenti all'alea. Questo e il riordinamento del codice

Per riconoscere dipendenze di dato a livello di RF basta confrontare registri sorgente e registro destinazione

Per riconoscere dipendenze di dato a livello della memoria non basta confrontare i registri
- l'indirizzamento viene fatto con immediato + registro
- bisogna confrontare per forza gli indirizzi calcolati

## Dipendenze di dato
Scordiamoci delle forwarding unit o altre soluzioni microarchitetturali alle dipendenze di dato, e concentriamoci sulle proprietà del codice

Definizione:
- Two instructions are independent (there is no dependency) if we can reverse their order without changing the result. This is called "data flow".
    - utile dato che vogliamo riordinare il codice
- In reality, to preserve the program's correctness, it is necessary to ensure that the new order preserves how exceptions occur. This condition is called "exception behavior"

Dipendenze di nome:
- non c'è un passaggio di dato da un'istruzione precedente ad una successiva
- in entrambi i casi si sarebbe potuto utilizzare qualsiasi altro registro
- antidipendenza:
- dipendenza d'usicta:
- le dipendenze di nome si risolvono rinominando i registri
    - lo può fare il compilatore staticamente
    - oppure l'hardware della pipeline utilizzando dei registri segreti

Nella nostra pipeline a 5 stadi le dipendenze di nome non causano alee. Vedremo però che in architetture che eseguono commit (scritture in RF/MEM) fuori ordine, queste dipendenze possono causare alee.
- questo perchè il momento in cui si legge il RF è distante rispetto a quello in cui si fa il commit (WB con i registri)



# Architetture multiciclo (riprenderemo con quelle superscalari dopo)
ipotizziamo che ora nella nostra pipeline, nello stadio di EX, oltre alla ALU abbiamo anche un'altra unità funzionale FPU.

la FPU è molto pià lenta rispetto alla ALU
- supponiamo 5us per FPU e 1us per ALU

Per far si che la cpu funzioni, dobbiamo abbassare la frequenza di clock per lasciare tempo alla FPU di fare i suoi calcoli...
- ma così le istruzioni intere eseguono ad un quinto della velocità

Possiamo allora pensare di applicare pipelining anche alla FPU dividendola in 5 stadi che possono avanzare anche con un clock di 1us

Assumiamo che la FPU sia bloccante, ovvero che non sia una vera e propria pipeline ma piuttosto la FPU non può accettare un'altra istruzione FP fino a che non ha completato quella al suo interno
- in questo caso si sta solo spezzando l'esecuzione in più stadi per poter rispettare il clock, non si sta applicando un vero pipelining
- istruzioni fp successive stallano in ID fino a quando quella dentro la FPU non termina
- Notiamo che se un'istruzione fp stalla in ID a causa di una FPU occupata, istruzioni successive intere rimangono bloccate in IF


**NB**: in questa architettura multiciclo è possibile che un'istruzione fp precedente termini dopo un istruzione intera successiva. Ho i commit fuori ordine!

A questo punto allora potrei avere dei problemi
- alee strutturali 
    - in MEM 
    - ma anche in WB in realtà dato che istruzioni intere / fp possono accedere al RF l'uno dell'altro; ad esempio, nel caso di istruzioni di conversione
        - fdiv x4, f5, f6
    - ma anche EX ???
- alee di dato tra istruzioni fp
    - qua lo stadio EX ci mette molto di più a completare e quindi è molto più probabile che un'istruzione FP dipendente debba stallare
        - anche con logica di forwarding, anzi, più è lunga l'esecuzione meno conviene il forwarding
- suscettibilità a alee di dato WAW
     - fsub f4, f5, f6
     - ld f4, 0(x1)


Qua non sono però suscettibile a alee di dato WAR (antidipendenze) 
- questo perchè, mentre il commit viene fatto fuori ordine, l'ID viene effettuato in ordine
- issue

Ultimo problema di questa microarchitettura è che non consente interrupt precisi
- interrupt precisi significa eseguire l'interrupt handler esattamente nel ciclo successivo in cui l'interrupt è arrivato


### come possiamo mitigare questi problemi?

1. alea strutturale per la memoria
    - stalliamo di un ciclo ma in che fase?
        - i'm cooked
        - se stallo nel WB, backpressure

2. alee waw
    - vogliamo ordinare le WB delle istruzioni che hanno lo stesso registro destinazione (è questo il caso in cui ho un WAW) 
    - anche qua stallo nell'ID

3. interrupt precise
    - non ha soluzione, è legato al caso di star facendo commit out of order

























