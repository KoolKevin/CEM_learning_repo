Abbiamo visto che riusciamo ad avere ILP se non abbiamo alee che causano stalli

- in particolare, consideriamo alee di dato e di controllo
- da analisi si è scoperto che il numero di istruzioni che non hanno questo tipo di alee nel codice (basic block) è di 3-6 istruzioni ...
- dobbiamo ridurre gli stalli se vogliamo ottenere un grado di ILP più alto (blocco di codice parallelizzabile di dimensione più grande)

Per alleggerire l'effetto delle alee di controllo possiamo fare loop unrolling per distanziare le branch incrementando così la dimensione del basic block e, di conseguenza, ILP

Il loop unrolling può essere utilizzato anche per alleggerire l'effetto delle alee di dato dato che introduce delle istruzioni indipendenti aggiuntive che possono venire utilizzate per spaziare le istruzioni appartenenti all'alea. Questo insieme al riordinamento del codice (che è un'ottimizzazione indipendente) riducono le alee di dato

Per riconoscere dipendenze di dato a livello di RF basta confrontare registri sorgente e registro destinazione

Per riconoscere dipendenze di dato a livello della memoria non basta confrontare i registri

- l'indirizzamento viene fatto con immediato + registro
- bisogna confrontare per forza gli indirizzi calcolati

## Dipendenze di dato

Scordiamoci delle forwarding unit o altre soluzioni microarchitetturali alle dipendenze di dato, e concentriamoci sulle proprietà del codice

Definizione:

- Two instructions are independent (there is no dependency) if we can reverse their order without changing the result. This is called "data flow".
  - dato che vogliamo riordinare il codice ci interessa capire quali istruzioni possiamo scambiare senza rompere nulla
- In reality, to preserve the program's correctness, it is necessary to ensure that the new order preserves how exceptions occur. This condition is called "exception behavior"
  - boh

Dipendenze di nome:

- antidipendenza:
- dipendenza d'usicta:
- non c'è un passaggio di dato da un'istruzione precedente ad una successiva
  - non causano quindi stalli (or do they? vedrai dopo che non causano stalli solo se vengono issued e committed in ordine)
  - bisogna che l'ordinamento sia conservato, i prima di j
- le dipendenze di nome si risolvono rinominando i registri
  - in entrambi i casi si sarebbe potuto utilizzare qualsiasi altro registro
  - lo può fare il compilatore staticamente
  - oppure l'hardware della pipeline utilizzando dei registri segreti

Nella nostra pipeline a 5 stadi le dipendenze di nome non causano alee.

- questo perchè il momento in cui si legge il RF è distante rispetto a quello in cui si fa il commit (WB con i registri)

Vedremo però che in architetture che eseguono commit (scritture in RF/MEM) fuori ordine, queste dipendenze possono causare alee.

In particolare, abbiamo:

- Write after write (WAW):
  - j scrive un operando (registro/memoria) prima che i lo scriva. La scrittura avviene nell’ordine sbagliato, lasciando il valore scritto da i al posto di quello scritto da j.
  - Possibile solo in pipeline che permettono la scrittura in più stadi o che permettono alle istruzioni di eseguire fuori ordine.
- Write after read (WAR):
  - j scrive un operando (registro/memoria) prima che i lo legga , i erroneamente legge il valore “nuovo”.
  - Non può succedere se la pipeline legge gli operandi in ID, quindi prima del fetch dell’istruzione successiva.
  - Può invece accadere se alcune istruzioni scrivono nei primi stadi della pipeline o se le istruzioni eseguono fuori ordine.

# Architetture multiciclo (riprenderemo con quelle superscalari dopo)

ipotizziamo che ora nella nostra pipeline, nello stadio di EX, oltre alla ALU abbiamo anche un'altra unità funzionale FPU.

la FPU è molto pià lenta rispetto alla ALU

- supponiamo 5us per FPU e 1us per ALU (5 clock e 1 clock)

Per far si che la cpu funzioni, dobbiamo abbassare la frequenza di clock per lasciare tempo alla FPU di fare i suoi calcoli...

- ma così le istruzioni intere eseguono ad un quinto della velocità

Possiamo allora pensare di applicare pipelining anche alla FPU dividendola in 5 stadi che possono avanzare anche con un clock di 1us

Assumiamo che la FPU sia bloccante, ovvero che non sia una vera e propria pipeline, piuttosto, la FPU non può accettare un'altra istruzione FP fino a che non ha completato quella al suo interno

- in questo caso si sta solo spezzando l'esecuzione in più stadi per poter rispettare il clock, non si sta applicando un vero pipelining
- istruzioni fp successive stallano in ID fino a quando quella dentro la FPU non termina
- Notiamo che se un'istruzione fp stalla in ID a causa di una FPU occupata, istruzioni successive intere rimangono bloccate in IF

**NB**: in questa architettura multiciclo è possibile che **un'istruzione fp precedente termini dopo un istruzione intera successiva**. Ho i commit fuori ordine!

In sostanza abbiamo:

- Attivazione sequenziale => IN ORDER ISSUE
  - La attivazione (issue) è il passaggio dallo stadio ID allo stadio successivo
- Inizio della Esecuzione sequenziale => IN ORDER EXECUTION
- Completamento fuori ordine => OUT OF ORDER COMPLETION

A questo punto allora potrei avere dei problemi

- alee strutturali
  - sopratutto in MEM dato che istruzioni fp e intere usano la stessa memoria
  - ma anche in WB dato che istruzioni intere / fp possono accedere al RF l'uno dell'altro;
    - ad esempio, nel caso di istruzioni di conversione
  - ma anche EX ???
- alee di dato tra istruzioni fp
  - qua lo stadio EX ci mette molto di più a completare, quindi è molto più probabile che un'istruzione FP successiva debba stallare fino al completamento dell'EX dell'istruzione precedente
    - anche con logica di forwarding, anzi, più è lunga l'esecuzione meno conviene il forwarding
- suscettibilità ad alee di dato WAW
  - fsub f4, f5, f6
  - ld f4, 0(x1)
  - qua la load termina prima della fsub dato che quest'ultima ci impiega di più (i commit sono fuori ordine)

Non sono però suscettibile a alee di dato WAR (antidipendenze)

- questo perchè, mentre il commit viene fatto fuori ordine, l'ID viene effettuato in ordine
- è impossibile che una scrittura venga completata prima di una lettura
- issue

Ultimo problema di questa microarchitettura è che non consente **interrupt precisi**

- si dice che una pipeline gestisce le eccezioni in modo preciso se la pipeline può essere fermata in modo che tutte le istruzioni precedenti l’istruzione in cui l’eccezione si è verificata siano completate, mentre tutte le istruzioni successive non modificano lo stato della CPU prima che l’eccezione sia stata servita.
- Se l’eccezione è un interrupt esterno, l’interrupt è gestito in modo preciso se esiste una istruzione nella pipeline tale per cui tutte le istruzioni precedenti sono completate prima che l’interrupt sia servito mentre tutte le istruzioni successive ripartono da zero dopo il servizio della interruzione stessa
- Esempio: si puo’ verificare una F.P. exception su FDIV quando le istruzioni successive sono gia’ state eseguite

In pipeline multiciclo abbiamo quindi un nuovo concetto di initiation interval

- Latency: the number of intervening cycles between an instruction that produces a result and an instruction that uses the result.
- Initiation Interval: the number of cycles that must elapse between issuing two operations of a given type.

### come possiamo mitigare questi problemi?

1. alea strutturale per la memoria
    - stalliamo di un ciclo ma in che fase?
        - i'm cooked
        - se stallo nel WB, backpressure
        - stallo in ID

2. alee waw
    - vogliamo ordinare le WB delle istruzioni che hanno lo stesso registro destinazione (è questo il caso in cui ho un WAW)
    - come possiamo fare? anche qua stallando nell'ID

3. interrupt precise
    - non ha soluzione, è legato al caso di star facendo commit out of order

Poi abbiamo sempre le alee RAW che vengono gestite nello stesso modo, solo che adesso non possono essere sempre evitate con forwarding dato che il dato ci mette più cicli ad essere prodotto

**NB**: in generale nello stadio di ID cerchiamo di capire se eseguendo l'istruzione corrente avremmo delle alee. In ID facciamo quindi

- decodifica dell'istruzione leggendo opcode e capendo quali sono i registri sorgente e destinazione
- alee di dato (e di nome)
- alee strutturali
  - in EX unità funzionale è bloccante ed è occupata
  - oppure in MEM / WB se memoria o RF sono occupati da istruzioni precedenti che però ci mettono molti cicli ad eseguire
- leggiamo gli operandi solo se non ci sono alee

### more realistic fp pipeline (comincia a leggere le slide da qua)

possiamo avere un CPI medio minore di 1?

- no, nella pipeline entra un'istruzione per clock e quindi alla meglio esce un'istruzione per clock
- inoltre anche se ne entrasse più di una, avremmo delle alee strutturali in uscita

Più stadi ci sono nello stadio di EX (/più la pipeline è profonda), più gli stalli causate da alee mi fanno crollare il CPI  (devo stallare per più cicli (anche con logica di forwarding))

- se le unità di esecuzione sono strutturate come pipeline non bloccanti, riusciamo ad ottenere un CPI che tende ad 1
- se invece sono bloccanti, il CPI tende alla latenza dello stadio di esecuzione

Lo stesso vale anche per pipeline lunghe

- ho critical path più corti e quindi posso aumentare la frequenza di clock
- ma ho più alee di dato e quindi il CPI medio si abbassa
- questo ed altri problemi
