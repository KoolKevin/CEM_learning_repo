Alcune domande iniziali:

- perchè i primi livelli sono privati mentre i livelli più bassi sono shared?
- che differenza c'è tra una cache privata e una cache shared?

Cache private vedono gli accessi provenienti solamente da un singolo core

Cache condivise vedono gli accessi provenienti da tutti i core

vantaggi svantaggi

- cache private
  - sono più piccole (dato che ne dobbiamo fare una per core) e quindi più veloci
  - non bisogna sincronizzare l'accesso con altri core (non c'è contesa di una risorsa condivisa)
  - coerenza più difficile
- cache condivisa
  - il cache controller serializza gli accessi
  - performance poco predicibili a causa della contesa

## che cos'è la coerenza?

due concetti:

- consistenza della memoria
  - ordinamento degli accessi
- coerenza della memoria
  - ordinamento degli accessi allo stesso indirizzo dei diversi core
  - se abbiamo cache private diventa difficile garantire ordinamento globale degli accessi dato che in caso di hit gli accessi alla memoria dei core vengono filtrati dalla loro cache

## Come risolviamo il problema cella coerenza

se la isa mette a disposizione delle istruzioni di invalidazione delle cache, possiamo:

- prima invalidare la mia linea di cache locale, in modo da scrivere il dato aggiornato in memoria
- poi invalidare le cache remote
- NB: queste due operazioni andrebbero fatte in modo atomico in maniera da non avere delle corse critiche se anche altre cache invalidano la propria cache locale

se invalido un blocco, e quel blocco è dirty, la cache dovrà scrivere il dato in memoria

## coherence, update vs invalidate

`nelle slide di questo capitolo processor è spesso sinonimo di cache controller`

per garantire coerenza dobbiamo sapere fare due cose:

- saper propagare le scritture a tutti gli altri core
- devo serializzare secondo una qualche logica scritture contemporanee allo stesso indirizzo

**How can we safely update replicated data?**

- Option 1 (Update protocol): push an update to all copies
- Option 2 (Invalidate protocol): ensure there is only one copy (local), update it

On a Read:

- if local copy is invalid, leggi dalla memoria; altrimenti, leggi dalla cache

On a write:

- if local copy is invalid, di nuovo leggi della memoria (devo recuperare l'intero blocco); successivamente emetti un broadcast
  - se si usa update protocol: il broadcast contiene indirizzo del blocco e il suo nuovo valore
  - se si usa invalidate protocol: il broadcast contiene solo l'indirizzo del blocco da invalidare

Pro e contro:

- update:
  - if sharer set is constant and updates are infrequent, avoids the cost of invalidate-reacquire (broadcast update pattern)
  - if data is rewritten without intervening reads by other cores, updates would be useless

- invalidate
  - After invalidation broadcast, core has exclusive access rights
    - Only cores that keep reading after each write retain a copy
  - If write contention is high, leads to ping-ponging (rapid invalidation-reacquire traffic from different processors)

**NB**: Con questo algoritmo

- possiamo avere fino ad N (numero di core) copie dello stesso blocco
- ma solo uno può avere un valore diverso rispetto a quello che c'è in memoria

Di solito non si fa update subito ma ci si limita ad invalidare, dato che non è detto che quel core abbia bisogno di quel blocco nuovamente

- aggiornerei per nulla
- meglio una logica lazy

# Tecniche di cache coherence hardware

## Snoopy-bus

tutti gli accessi in memoria fatti da tutti i core sono visibili da tutti gli altri core in quanto viaggiano su un unico bus condiviso

Ci può essere solamente un attore alla volta sullo snoopy-bus

- anche se le richieste appartengono a indirizzi/blocchi diversi :(
- bus risorsa condivisa e unico punto di serializzazione

## Directory

ogni blocco ha il suo punto di serializzazione

quando un core vuole scrivere un blocco, richiede alla directory (componente hardware) il permesso

- sarà la directory a propagare a tutti gli altri core che hanno in cache il blocco scritto l'evento di avvenuta scrittura
  - NB: propaga solo ai core che hanno una copia, non faccio un broadcast come prima

La directory mantiene lo stato delle varie cache, salvando quali cache hanno quale blocco

- molto costoso in termini di spazio

### Snoopy-bus vs directory

la directory sicuramente non serializza se non necessario, tuttavia ha un costo maggiore

- sia in termini di logica aggiuntiva
- ma soprattutto in termini di dati aggiuntivi che bisogna salvare (1 bit per ogni blocco di memoria di ogni cache per capire se un blocco è presente o meno in una cache)
  - costo in area molto elevato
  - considera uno spazio di indirizzamento a 64 bit, se la block size è 2^6 = 64 bit, la directory dovrà salvare 2^58 bit per core
  - (chiaramente si utilizzano tecniche intelligenti come organizzazioni gerarchiche, e non si mantiene l'intero address space ma solo 48 bit)

snoopy-bus molto più semplice ed economico

- tuttavia scala in maniera pessima con il numero di core dato che viene a crearsi contesa sul bus

### Cache coherence come ordinamento degli accessi allo stesso blocco

le due implementazioni che abbiamo visto serializzano gli accessi, è qui che avviene l'ordinamento

i protocolli di coerenza definiscono degli stati in cui si definisce quali letture/scritture si devono fare (? aggiustare meglio)

# Protocollo Valid/Invalid

...

**OSS**: si potrebbe adattare VI anche con politica write-allocate aggiustando un arco

**NB**: perchè valid/invalid ha bisogno di write through?

- dato che le letture invalide devono trovano il valore giusto nella memoria
- per avere write-back caches bisogna adottare protocolli più avanzati come MSI

# MSI

lo stato valido implica che il dato è valido e coerente con quello che c'è nella memoria

se voglio una cache write-back ho bisogno di un terzo stato che mi rappresenta un blocco di cache corretto ma non coerente rispetto a quello che c'è in memoria

Dato che ho bisogno di 3 stati, adesso ogni blocco (nel tag store) non ha più solo uno singolo bit per la validità, piuttosto ne ha 2 per codificare i seguenti stati:

- M(odified): cache line is the **only cached** copy and is dirty
- S(hared): cache line is one of potentially several cached copies and it is clean (i.e., at least one clean cached copy)
- I(nvalid): cache line is not present in this cache

``` perchè quando abbiamo una write miss dobbiamo leggere il dato dalla memoria se poi lo modifichiamo?
perchè noi stiamo accedendo ad una intera linea di cache, non al singolo dato.

dobbiamo leggere la linea e modificare solo la parte che contiene il dato che vogliamo scrivere
```

``` perchè in MSI abbiamo sostituito busWr con busRdX?
adesso abbiamo delle cache write-back

le scritture adesso modificano solo la cache e non la memoria. In caso di write miss leggiamo la memoria con la busRdX
```

NB: il cache controller, prima di sapere in che stato si trova un determinato blocco nella cache, ha bisogno di attivare la logica che controlla se si ha una hit o una miss

- questo significa che anche operazioni che sembrano non avere alcun effetto (vedi PrRead o BusRd) in realtà effettuano comunque questa operazione che controlla se il blocco è nella cache o meno
- d'altronde l'informazione sullo stato corrente è presente nei bit di stato presenti nel tag store

flush significa scrivere la copia dirty in memoria

Oss: a seguito delle busRead c'è una sequenzialità delle azioni scatenate. Ad esempio: non leggo la memoria prima che sia stato effettuato il flush

## Problema

Abbiamo chiamato shared lo stato che mi dice che ho una copia non dirty del dato. Non distinguo se sono l'unico ad avere la copia, oppure se questa copia è effettivamente shared

Tutte le volte che modifico il dato nello stato shared devo per forza notificare gli altri che sto scrivendo.

- se hanno una copia anche loro devono invalidarla e quindi questo broadcast è servito
- se invece non ce l'hanno devono comunque attivare la logica di hit, questo potrebbe essere evitato se si aggiungesse uno stato che codifica lo stato in cui sono l'unico ad avere una copia non dirty
- inoltre, siccome le richieste passano per lo snoopy-bus che serializza, serializziamo di più le richieste

# MESI

S = risposta dal bus che mi dice che quel dato è shared (oppure ce l'ha uno exclusive)
S'= risposta dal bust che mi dice che quel dato non è shared

$transfer = è il cache controller della cache exclusive che fornisce la copia ai richiedenti (che non hanno bisogno quindi di leggera dalla lenta memoria)

queste risposte vengono dato dall'or di segnali di risposta di altri cache controller che attivano la loro logica di hit per controllare se quel blocco è presente o meno

## MOESI

questa versione permette di fornire i dati non mediante la memoria ma direttamente dai core che hanno una copia clean
