Dynamic scheduling, a technique by which the hardware reorders the instruction execution to reduce the stalls while maintaining data flow (== rispettare le dipendenze di dato) and exception behavior.

Dynamic scheduling offers several advantages:

- First, it allows code that was compiled with one pipeline in mind to run efficiently on a different pipeline, eliminating the need to have multiple binaries and recompile for a different microarchitecture
  - se ti ricordi, lo static scheduling del compilatore dipendeva dalla specifica microarchitettura
- Second, it enables handling some cases when dependences are unknown at compile time; for example, they may involve a memory reference or a data-dependent branch, or they may result from a modern programming environment that uses dynamic linking or dispatching.
- Third, and perhaps most importantly, it allows the processor to tolerate unpredictable delays, such as cache misses, by executing other code while waiting for the miss to resolve.

The advantages of dynamic scheduling are gained at the cost of a significant increase in hardware complexity.

### Idea

A major limitation of simple pipelining techniques is that they use in-order instruction issue and execution:

- instructions are issued in program order
- and if an instruction is stalled in the pipeline, **no later instructions can proceed**.

If instruction j depends on a long-running instruction i, currently in execution in the pipeline, then all instructions after j must be stalled until i is finished and j can execute.

- If there are multiple functional units, **these units could lie idle**.

For example, consider this code:

```
fdiv.d f0,f2,f4
fadd.d f10,f0,f8
fsub.d f12,f8,f14
```

- The fsub.d instruction cannot execute because the dependence of fadd.d on fdiv.d causes the pipeline to stall

- yet, **fsub.d is not data-dependent on anything in the pipeline.**

- This hazard creates a performance limitation that **can be eliminated by not requiring instructions to execute in program order.**
  - Siccome queste pipeline permettono ad istruzioni indipendenti di proseguire in caso di stalli, vengono anche dette **non bloccanti**

The main idea in non blocking architectures is incrementing ILP by allowing execution of independent instructions even if out of program order

### Come funziona?

In the classic five-stage pipeline, both structural and data hazards could be checked during instruction decode (ID):

- when an instruction could execute without hazards, it was issued from ID

Now we want an instruction to begin execution as soon as its operands are available even if a predecessor is stalled.

To allow us to begin executing the fsub.d in the preceding example, **we must separate the issue process into two parts**:

- checking for any structural hazards
  - emettiamo un'istruzione solo se la FU è libera
- and waiting for the absence of a data hazard.
  - cominciamo ad eseguire un'istruzione solo se i suoi operandi sono pronti (=== non ci sono data hazard)

In a dynamically scheduled pipeline

- all instructions pass through the issue stage in order (**in-order issue**)
  - in-order instruction issue (i.e., instructions issued in program order)
- however, they can be stalled or can bypass each other in the second stage (read operands)
- and thus enter execution out of order because we want an instruction to begin execution as soon as its data operands are available.
- Such a pipeline does **out-of-order execution, which implies out-of-order completion**

To allow out-of-order execution:

- ID stage split into 2 stages:
  - Issue: Decode instruction, check for structural hazards.
  - Read Operands: Wait for data hazard to be resolved, read the operands
- IF unchanged.
- EX considered multicycle, Two phases:
  - Instruction begin execution
  - Instruction complete execution

### Complicazioni

**Out-of-order execution introduces the possibility of WAR and WAW hazards**, which do not exist in the classic five-stage pipeline

Consider the following RISC-V floating-point code sequence:

```
fdiv.d f0,f2,f4
fmul.d f6,f0,f8
fadd.d f0,f10,f14
```

There is an antidependence between the fmul.d and the fadd.d, and **if the pipeline executes the fadd.d before the fmul.d (which is waiting for the fdiv.d)**, it will violate the antidependence, yielding a WAR hazard.

Likewise, to avoid violating output dependences, such as the write of f0 by fadd.d before fdiv.d completes, WAW hazards must be handled.

As we will see, both these hazards are **avoided by the use of register renaming.**

**NB**: abbiamo spezzato un'assunto fatto fino ad ora

- era impossibile che un'istruzione che viene dopo modificasse gli operandi di un'istruzione precedente (alea WAR)
  - questo perchè gli operandi venivano letti in ID e scritti in WB

### Scoreboarding

Scoreboarding is a technique for allowing instructions to execute out of order when there are sufficient resources and no data dependences

"RAW hazards are avoided by executing an instruction only when its operands are available, which is exactly what the simpler scoreboarding approach provides. WAR and WAW hazards, which arise from name dependences, are eliminated by register renaming."

Ricorda:

- issue = parcheggiare in attesa di leggere gli operandi
- issue avviene in ordine, ma RO e il resto no

Scoreboarding activities:

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

If there is a matching reservation station that is empty, issue the instruction to the station with the operand values, if they are currently in the registers. If there is not an empty reservation station, then there is a structural hazard, and the instruction issue stalls until a station or buffer is freed. If the operands are not in the reg- isters, keep track of the functional units that will produce the operands. This step renames registers, eliminating WAR and WAW hazards. Lo scoreboard è un tabellone che determina quando un'istruzione

**L'algoritmo di scoreboard tiene traccia di 3 strutture dati per il suo funzionamento** (scoreboarding tiene traccia dello stato di istruzioni, registri e FU) :

1. Instruction status
    - current step (of 4, vedi dopo) for an instruction

2. Register result status
    - ha una entry per ogni registro nel RF
    - **holds which FU will write a given register** (used to set Qj, Qk when issuing)
        - utilizzo questa informazione per gestire le RAW in RO

3. Functional Unit (FUs) status
    - descrivo lo stato di ogni unità funzionale (una riga per unità funzionale):
    - Busy Unit: (busy or not)
    - Op: Operation to perform (e.g., +, -)
    - Fi: Destination register
    - Fj,Fk: Source register numbers
    - Qj,Qk: FUs producing sources Fj,Fk
        - se non c'è scritto niente, vuol dire che il valore nel RF è valido
        - se c'è scritto qualcosa vuol dire che questo operando viene prodotto dalla FU segnata
        - quando un'istruzione viene issued, si va a controllare il Register Result Status register per popolare questi due campi
    - Rj,Rk: Flags indicating Fj,Fk are ready
        - praticamente uguali al negato di Qj e Qk
        - se i registri sorgente sono validi Qj e Qk sono nulli, e viceversa

**Stage of Scoreboard Control:**
Dopo IF la nuova pipeline presenta i seguenti stadi:

1. Issue (I)
    - decode instruction, check for structural and WAW hazards, stall when necessary
        - una volta che si conosce il tipo di istruzione, si conosce anche il tipo di FU richiesta e si può quindi controllare la presenza di alee strutturali
    - **NB**:
        - prima ci accorgevamo di eventuali alee nello stadio di ID.
        - Ora, quando decodifichiamo l'istruzione nello stadio di ISSUE, non sappiamo quando l'istruzione verrà effettivamente eseguita
        - questo perchè non sappiamo in questo stadio quando i suoi operandi saranno pronti
        - lo sa solo lo scoreboard
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

**Come vengono gestite le alee?**

- Structural hazards (I):
  - ensure that a functional unit is available (makes a “reservation”)
    - quando un'istruzione passa I, riserva la FU anche se gli operandi non sono stati letti
    - fare issue significa prenotare una futura esecuzione in quell'unità funzionale
- WAW hazards (I):
  - ensure that no previous active instruction has the same destination
- RAW hazards (RO):
  - check that no previous active instruction writes a source register
  - se questo avviene, l'istruzione corrente rimane parcheggiata in issue in attesa che i suoi operandi diventino pronti
  - nel frattempo magari si riesce a mandare in esecuzione un altra istruzione issued
- WAR hazards (WR):
  - before writing a result, check if any previous instructions that haven’t gone past RO need that register as a source

NB: quand'è che abbiamo una WAR?

- abbiamo un'istruzione di cui è stata fatta issue ma non ancora RO (sta attendendo i suoi operandi)
- l'istruzione corrente è WR ed ha come registro destinazione uno dei registri sorgente dell'istruzione sopra
- l'alea si presenta quando l'istruzione che sta attendendo gli operandi ha come opearando sorgente quello destinazione dell'istruzione scrivente, ma non sta attendendo il risultato da quell'unità funzionale

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

- NB: notiamo che stalliamo dato che lo scoreboard non ha un parcheggio per la seconda load. Se ci fosse un'altra unità intera non avremmo dovuto stallare
Al termine del terzo ciclo non posso liberare l'unità intera? NO!
  - non ho ancora fatto WR e non so se riesco a farla subito dopo dato che magari ci sono delle WAR che causano stalli e mi tengono occupata la FU
  - Perché?
    - Perché il scoreboard è responsabile di tenere l’unità fino a che il suo risultato non è stato ufficialmente scritto nel registro destinazione.
    - Finché il risultato non è scritto, l’unità funzionale è l’unico posto dove il risultato “vive” (non c’è un buffer che lo conserva).
    - Se liberassi l’unità prima del WR, perderesti il riferimento a dove si trova il risultato temporaneo.
    - Inoltre, il WR potrebbe non avvenire subito (potresti dover stallare per evitare una dipendenza WAR con un’istruzione precedente che deve ancora leggere lo stesso registro).

A quanto pare facciamo il fetch di un'istruzione alla volta

- se c'è uno stallo in issue blocco anche il fetch

Notiamo quello che avevamo già detto

- issue in ordine
- RO fuori ordine

Nel settimo ciclo la fmul si blocco in RO dato che f2 non è pronto (RAW)

Al ciclo 9, f2 è stato scritto e quindi la dipendenza RAW è stata risolta (non al ciclo 8 dato che in quel ciclo non ho ancora scritto il RF)

- le due istruzioni che stavano aspettando posso entrare in RO (se il RF ha sufficenti porte, altrimenti avremmo un alea strutturale che bloccherebbe uno dei due)

scoreboard (functional unit status) è una CAM

- in Issue ci accediamo per indice di riga, ovvero per FU
- negli altri stadi ci accediamo per contenuto

# pro e contro scoreboarding

problemi:

- issue in ordine e bloccante
  - problema principale: stalliamo dato che c'è solo un parcheggio per ogni FU
    - potremmo aumentare il numero di FU ma questo ha un costo
  - le alee strutturali che abbiamo in Issue il più delle volte non corrispondono al fatto che la FU sia veramente occupata; piuttosto, stalliamo dato che non c'è una entry disponibile nello scoreboard
    - se lo slot è occupato, non posso emettere una nuova istruzione, anche se fisicamente l’unità potrebbe iniziare a lavorare su un’altra istruzione
    - ad esempio una load che ha già calcolato l’indirizzo e sta solo aspettando di fare WB, tenendo occupata la entry.

- a volte abbiamo delle alee WAR che si potrebbero evitare se si leggesse il RF quando il dato è disponibile nel RF, invece di aspettare che entrambi gli operandi siano pronti
  - sarebbe più intelligente salvare in Rj e Rk il valore del registro quando diventa disponibile
  - Rj e Rk diventa Vj e Vk ovvero delle copie del valore del registro
  - di fatto stiamo facendo register renaming dato che non stiamo usando più il registro ma una sua copia

pro:

- risolve automaticamente le dipendenze di dato
