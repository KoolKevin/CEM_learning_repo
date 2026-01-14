L'architettura di Tommasulo che abbiamo visto stalliamo solo nel caso di alee strutturali e di controllo

Inoltre, fino ad adesso abbiamo visto che le istruzioni fanno

- fanno issue in ordine
- eseguono fuori ordine
- fanno commit (terminano l'esecuzione) fuori ordine

Questo ci va bene dato che tutte le istruzioni che stiamo mettendo in esecuzioni siamo sicuri di volerle eseguire

Questo tuttavia non è sempre vero se teniamo conto delle istruzioni di branch

Architetture speculative sono architetture che permettono di fare fetch delle istruzioni senza effettivamente sapere se queste bisogna metterle in esecuzione o meno

reorder buffer permette di fare il commit delle istruzioni in ordine

L'algoritmo di tommasulo funziona con un carburante che sono le istruzioni fetched. Se non riusciamo a fare fetch tutto si blocca

Bisogna inserire nello stadio di IF la capacità di poter sapere sempre il prossimmo PC di cui andare a fare fetch anche in caso di branch, salti incondizionati, chiamate e ritorni da funzione

Abbiamo quindi bisogno di Dynamic Branch prediction per poter continuare a fare fetch di istruzioni riducendo gli stalli dovuti a istruzioni di controllo:

- (non approfondiremo tanto come si fa branch prediction)
- branch prediction buffer mi dice solo se un branch è taken o no
- branch target buffer è una tabella \[instr PC | target pc\] che mi dice dove saltare una volta che incontro un determinato PC
  - qua dentro registriamo solo i branch taken
  - (d'altronde dei branch not taken conosciamo già il NPC = PC + 4)
- return address stack: tutte le volte che si salta dentro ad una funzione si fa push dell'indirizzo di ritorno dentro ad uno stack hardware

## esempio da slide 9 in poi

Ciascuno iterazione del loop è indipendente (no dipendenze di dato) e quindi può in teoria essere messa in esecuzione in parallelo (tutte le istruzioni nella issue window)

- tuttavia, se non gestiamo le alee di controllo, le istruzioni successive alla branch non riescono a fare fetch (e issue)
- con loop unrolling possiamo ottenere un buon grado di ILP e nascondere la latenza dell'alea di controllo... ma questo solo se non ci sono dipendenze di dato nel loop!

Possiamo allora provare a speculare, cioè ad eseguire istruzioni ancora prima di sapere se dobbiamo farlo per davvero

Ipotizzando di avere reservation station infinite, grazie alla speculazione riusciamo ad arrivare ad avere un CPI pari ad 1 **anche se abbiamo dipendenze di dato e/o istruzioni che con una latenza alta**

- questo perchè **riusciamo a fare issue di un istruzione ad ogni ciclo di clock nascondendo in questo modo la latenza delle operazioni a lungo termine**
- continuiamo ad ampliare la issue window e quindi il grado di concorrenza
- prima o poi arriveremo ad uno steady state in cui riusciamo a completare un istruzione per clock

Il problema è che la speculazione non è perfetta, a volte potremmo eseguire delle istruzioni che non dovevamo eseguire

- abbiamo bisogno del reorder buffer per annullarne l'effetto

# Scheduling dinamico con esecuzione speculativa

- Obiettivo: stallare la pipeline solo in caso di alee strutturali
- Soluzione: predizione, evitando azioni “irrevocabili” prima di essere certi che tale azione deve essere effettuata (verifichiamo di aver predetto correttamente)
- Conseguenza: ***si amplia la finestra su cui cercare il parallelismo a livello di istruzione***
- Attivazione sequenziale: PREDICTION BASED IN-ORDER DISPATCHING
- Esecuzione fuori ordine OUT-OF-ORDER EXECUTION
- Completamento fuori ordine: OUT-OF-ORDER COMPLETION
- Avanzamento in ordine: IN-ORDER COMMIT

## Reorder buffer

Introduciamo uno stadio di instruction commit con cui andiamo a modificare lo stato del RF e della memoria solo quando siamo sicuri che l'instruzione in considerazione non è più speculativa

Need an additional piece of hardware to prevent any irrevocable action until an instruction commits: Reorder buffer

– holds the result of instruction between completion and commit

Four fields:

- Instruction type:
  - branch: Serve per ricordare dove era avvenuta la predizione e poter verificare in seguito se era corretta
  - store
  - register
- Destination field: register number
- Value field: output value
- Ready field: completed execution?

Reorder buffer contiene anche un vettore di flag che mi dicono se quell'istruzione ha generato un'eccezione o meno

- **NB**: in questo senso notiamo che il sollevamento di un'eccezione è anch'esso una modifica dello stato della CPU è va portato a termine solo se l'istruzione che ha sollevato l'eccezione andava effettivamente esecuita (per questo il reorder buffer contiene anche quest'informazione)
- non chiamiamo l'interrupt service routine fino a che non siamo sicuri che quell'istruzione la dovevamo eseguire o meno (commit)

Il reorder buffer ha delle entry che vengono **allocate durante issue**

- **possiamo quindi avere alee strutturali** come con le reservation station
- non c'è spazio quando head == tail (buffer circolare)

### Modifiche importanti

L'introduzione del ROB richiede di modificare le reservation stations:

- Operand source is now reorder buffer instead of functional unit (reservation station tag)

Il CDB non scrive più valori nel RF e nelle store unit (che vuol dire fare commit delle istruzioni).

- ora scrive nel ROB, e sarà quest'ultimo a propagare i valori nel RF
- fin tanto che non facciamo commit i valori rimangono nel ROB

### Funzionamento ROB

- Viene caricato in ordine dallo stadio ID contestualmente con il caricamento della Reservation Station
- Nell’intervallo tra il completamento dell’esecuzione e la fase di commit, il ROB (e non il register file) mette a disposizione i risultati
  - Pertanto: Il TAG: (Unit_ID, RS_ID) diventa: TAG: (Indice nel ROB)
- Il risultato di una istruzione diventa “impegnativo”, cioè passa ad aggiornare lo stato del “sistema” se:
  - L’istruzione è stata eseguita
  - I risultati di tutte le istruzioni precedenti sono “impegnativi”

### Nuova pipeline

- Issue:
  - Allocate RS and ROB, read available operands
- Execute:
  - Begin execution when operand values are available
- Write result:
  - Write result and ROB tag on CDB
- Commit:
  - When a predicted branch reaches head of ROB, update register
  - When a mispredicted branch reaches head of ROB, discard all entries

...

quando facciamo una issue muoviamo la coda

la prima issue assegna la testa all'ultima entry del ROB

non aggiorno i registri quando la FU genera un valore

**il commit lo faccio solo quando un'istruzione puntata dalla testa del ROB fa il commit**

- il commit è in ordine dato che il ROB viene popolato con l'ordine delle issue (che sono in ordine)

L'avere i commit in ordine mi permette di sapere se un'istruzione è speculativa

- se una istruzione arriva alla testa del ROB allora non è più speculativa dato che branch precedenti hanno già fatto il commit e non hanno fatto flush
- inoltre mi semplifica la gestione delle eccezioni rendendole precise

## Esempio
