# Scoreboarding

Scoreboarding is a technique for allowing instructions to execute out of order when there are sufficient resources and no data dependences

- RAW hazards are avoided by executing an instruction only when its operands are available
  - which is exactly what the simpler scoreboarding approach provides.
- WAR and WAW hazards, which arise from name dependences, are eliminated by register renaming."

Ricorda:

- issue = parcheggiare in attesa di leggere gli operandi
- issue avviene in ordine, ma RO e il resto no

## Scoreboarding activities

- Keep track of instructions, functional units, and registers to handle hazards
  - Goal: CPI=1 (abbiamo un architettura comunque scalare e quindi sotto ad 1 non scendiamo)
    - per avere un CPI < 1 dobbiamo per forza avere un architettura superscalare
- “Pull out” **independent** instructions later in the **issue window**
  - issue window è una sorta di instruction queue dove finiscono le istruzioni di cui si è fatto fetch
- **Wait queue** after Issue to hold stalled instructions waiting for operands
  - It may not really exist => implemented by the scoreboard!
- Multiple or pipelined functional units
  - the main goal of out-of-order execution is to increment ILP, allowing the execution of multiple instructions at once
  - Having multiple instructions in execution at once requires multiple functional units, pipelined functional units, or both (sostanzialmente equivalenti).
  - recall: during issue, we stall on a structural hazard; with multiple/pipelined functional units we reduce structural stalls

**L'algoritmo di scoreboard tiene traccia di 3 strutture dati per il suo funzionamento** (scoreboarding tiene traccia dello stato di istruzioni, registri e FU) :

1. Instruction status
    - current step (of 4, vedi dopo) for an instruction

2. Register result status
    - ha una entry per ogni registro nel RF
    - **holds which FU will write a given register** (used to set Qj, Qk when issuing)
        - utilizzo questa informazione per gestire le RAW in RO
        - se voglio leggere un registro ma vedo che nello scoreboard una FU lo sta scrivendo, aspetto

3. Functional Unit (FUs) status
    - descrivo lo stato di ogni unità funzionale (una riga per unità funzionale):
    - Busy Unit: (busy or not)
    - Op: Operation to perform (e.g., +, -)
    - Fi: Destination register
    - Fj,Fk: Source register numbers
    - Qj,Qk: FUs producing sources Fj,Fk
        - se non c'è scritto niente, vuol dire che il valore nel RF è valido
        - se c'è scritto qualcosa vuol dire che questo operando viene prodotto dalla FU segnata
        - **quando un'istruzione viene issued, si va a controllare il Register Result Status register per popolare questi due campi**
    - Rj,Rk: Flags indicating Fj,Fk are ready
        - praticamente uguali al negato di Qj e Qk
        - se i registri sorgente sono validi Qj e Qk sono nulli, e viceversa

## New pipeline stages with scoreboard Control

Dopo IF la nuova pipeline presenta i seguenti stadi:

1. Issue (I)
    - decode instruction, check for structural and WAW hazards (alee WAR gestite in WR, vedi dopo), stall when necessary
        - decodificata l'istruzione, si conoscono la FU richiesta e i registri da usare, si può quindi controllare la presenza di alee con altre istruzioni
    - **NB**:
        - prima ci accorgevamo di eventuali alee nello stadio di ID.
        - Ora, quando decodifichiamo l'istruzione nello stadio di ISSUE, non sappiamo quando l'istruzione verrà effettivamente eseguita
        - questo perchè non sappiamo in questo stadio quando i suoi operandi saranno pronti, lo sa solo lo scoreboard
        - sarà quindi lo scoreboard a gestire alee strutturali e WAW
    - **NB**: stalli in questo stadio **SONO BLOCCANTI**
        - non permettiamo ad istruzioni che vengono dopo di essere accolte all'interno della pipeline e di entrare in esecuzione prima di altre istruzioni bloccate in attesa di operandi
        - solamente però in caso di alee strutturali/WAW; non più in caso di alee RAW

2. Read Operands (RO)
    - wait until no RAW hazards (== operandi disponibili nel RF), then read operands, send operation to FU

3. Execution (EX)
    - FU starts execution upon receiving the operands & notifies scoreboard when it’s completed

4. Write Result (WR)
    - **Scoreboard checks for WAR hazards** and stalls write to register file to avoid them

Notiamo che sembra mancare lo stadio di memory. **La memoria diventa una functional unit**

Notiamo anche come lo scoreboard gestisce tutti i tipi di alea di dato e le alee strutturali

## Come vengono gestite le alee?

- Structural hazards (I):
  - ensure that a functional unit is available (makes a “reservation”)
    - quando un'istruzione passa I, riserva la FU anche se gli operandi non sono stati letti
    - fare issue significa prenotare una futura esecuzione in quell'unità funzionale
- WAW hazards (I):
  - ensure that no previous active instruction has the same destination
- RAW hazards (RO):
  - check that no previous active instruction writes a source register
  - se questo avviene, l'istruzione corrente rimane parcheggiata nello scoreboard in attesa che i suoi operandi diventino pronti
  - nel frattempo magari si riesce a mandare in esecuzione un altra istruzione issued
- WAR hazards (WR):
  - before writing a result, check if any previous instructions that haven’t gone past RO need that register as a source

Oss:

- Nella pipeline multiciclo poteva succedere che più istruzioni potessere accedere allo stadio di MEM/WB in contemporanea
  - per risolvere il problema utilizzavamo un registro a scorrimento e logica in ID per rilevare questi conflitti in modo da poter stallare la pipeline
- Ora abbiamo una scoreboard che gestisce questi problemi strutturali / di alee
  - the data structures and the logic of the scoreboard detect and handle hazards

**NB**: interessante notare che mediante lo studio delle dipendenze di dato possiamo definire dei **thread** di istruzioni che devono eseguire in sequenza.

- A noi interessante quindi le dipendenze di dato presenti nel codice che ci definiscono dei DAG per l'ordinamento
- Non conta più la distanza tra due istruzioni, quello era interessante solamente per lo scheduling statico effettuato da un compilatore per una particolare architettura
  - e.g. se so che ho una forwarding unit ...

- la microarchitettura si costruisce dei DAG per segnare le istruzioni dipendenti
- io vado a pescare istruzioni indipendenti da DAG separati
- se non ci sono abbastanza istruzioni indipendenti stallo

# Scoreboard example

al secondo ciclo la seconda load non entra in issue dato che l'unità intera è occupata (la prima load non ha ancora eseguito il calcolo dell'immediato)

**NB**: notiamo che stalliamo dato che lo scoreboard non ha un parcheggio per la seconda load. Se ci fosse un'altra unità intera non avremmo dovuto stallare

Al termine del terzo ciclo non posso liberare l'unità intera? NO!

- non ho ancora fatto WR e non so se riesco a farla subito dopo dato che magari ci sono delle WAR che causano stalli e mi tengono occupata la FU
- Perché?
  - Perché il scoreboard è responsabile di tenere l’unità fino a che il suo risultato non è stato ufficialmente scritto nel registro destinazione.
  - Finché il risultato non è scritto, l’unità funzionale è l’unico posto dove il risultato “vive” (non c’è un buffer che lo conserva).
  - Se liberassi l’unità prima del WR, perderesti il riferimento a dove si trova il risultato temporaneo.
  - Inoltre, il WR potrebbe non avvenire subito (potresti dover stallare per evitare una dipendenza WAR con un’istruzione precedente che deve ancora leggere lo stesso registro).

Notiamo quello che avevamo già detto

- issue in ordine
- RO fuori ordine

Nel settimo ciclo la fmul si blocco in RO dato che f2 non è pronto (RAW)

Al ciclo 9, f2 è stato scritto e quindi la dipendenza RAW è stata risolta (non al ciclo 8 dato che in quel ciclo non ho ancora scritto il RF)

- le due istruzioni che stavano aspettando posso entrare in RO (se il RF ha sufficenti porte, altrimenti avremmo un alea strutturale che bloccherebbe uno dei due)

# pro e contro scoreboarding

problemi:

- issue in ordine e bloccante
  - problema principale: stalliamo dato che c'è solo un parcheggio per ogni FU
    - potremmo aumentare il numero di FU ma questo ha un costo
  - le alee strutturali che abbiamo in Issue il più delle volte non corrispondono al fatto che la FU sia veramente occupata; piuttosto, stalliamo dato che non c'è una entry disponibile nello scoreboard
    - se lo slot è occupato, non posso emettere una nuova istruzione, anche se fisicamente l’unità potrebbe iniziare a lavorare su un’altra istruzione
    - ad esempio una load che ha già calcolato l’indirizzo e sta solo aspettando di fare WB, tenendo occupata la entry.

- a volte abbiamo delle alee WAR che si potrebbero evitare se si leggesse il RF quando il dato è disponibile nel RF, invece di aspettare che entrambi gli operandi siano pronti
  - sarebbe più intelligente salvare in Rj e Rk il valore del registro appena diventa disponibile
  - Rj e Rk diventa Vj e Vk ovvero delle copie del valore del registro
  - di fatto stiamo facendo register renaming dato che non stiamo usando più il registro ma una sua copia

pro:

- risolve automaticamente le dipendenze di dato
