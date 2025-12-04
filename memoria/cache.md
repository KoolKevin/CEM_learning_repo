...

la cache lavora più che altro sulla ripetibilità degli accessi

- non predice indirizzi e non va a prenderseli
- esiste un componente chiamato prefetcher che fa delle load predicendo indirizzi (e quindi prima del programma)

Dividiamo la memoria in blocchi logici di uguale dimensione (cache line) **per sfruttare la località spaziale**

...

non invio in parallelo le richieste ma vado in sequenza livello per livello

# le Cache

come è fatta la cache?

è divisa in due parti

- una che immagazzina i blocchi di memoria
- un tag store che immagazzina quali indirizzi (un certo blocco di memoria really) sono presenti nella cache e altri metadati

## Direct mapped

OSS: quando si dice che una cache è di 16k si fa riferimento solo alla dimensione del data store

Riusciamo a fare tutto con uno solo comparatore

Addresses with same index contend for the same location

- se ho un pattern di accesso infelice, che salta tra blocchi che contendono per lo stesso slot, ottengo dei miss nonostante ci sia spazio in cache
- Cause conflict misses

## Associative

chiamiamo una riga (blocco) nella cache come **set**

- con i bit di indice identifichiamo un set all'interno della cache
- con la cache directly mapped un set corrisponde ad un blocco

nel caso di associatività 2, vediamo la cache come divisa in 2

- adesso posso avere due blocchi con lo stesso indice in cache, i tag saranno diversi

ho più comparatori e una latenza maggiore

- a volte si vuole la latenza minima e quindi si utilizzano le cache directly mapped

con higher associativity il discorso è lo stesso

Quando arrivo ad avere un grado di associatività pari alla dimensione in blocchi dalla cache ho raggiunto la **full-associativity**

- ora un blocco può essere posizionato in qualunque posizione nella cache (non ci sono più bit di indice nell'indirizzo)
- le miss dovute ai conflitti sono state eliminate (pagando però il prezzo di latenza e area)

**NB**: Cambia molto il grado di associatività in base a quale livello della gerarchia

- cache piccole, associatività basse per avere la latenza minore possibile (ricorda formula ricorsiva per il tempo di accesso)
- cache grandi, associatività alte. Qua la latenza è già abbastanza alta e quindi posso permettermi un'associatività più alta per ridurre il miss rate dobuto a conflitti

# Problematiche nelle cache set-associative

Con le cache associative abbiamo più slot in cui possiamo inserire un determinato blocco di memoria.

- Abbiamo nuove problematiche assenti nelle cache directly-mapped, dato che li un blocco può appartenere ad un singolo slot

Per gestire queste problematiche associamo ai blocchi in cache una **priorità** che indica l'importanza di mantenere quel blocco in cache

In particolare abbiamo 3 decisioni da prendere:

- in caso di cache hit: modifichiamo la priorità del blocco?
  - promotion
- in caso di cache miss: che blocco sostituiamo? come cambiamo le priorità?
  - inserimento, eviction

**NB**: sottolineo che queste decisioni vanno prese solo in cache associative in cui si hanno più blocchi in un set.

- in una cache directly mapped, se ho una hit sono contento, se ho una miss non ho nulla da decidere, sostituisco il blocco presente all'index di cui ho fatto miss (solo uno)
- nelle cache associative devo decidere quale tra i vari blocchi del mio set sostituire

In generale le politiche di eviction sono due:

- random
- o LRU
  - con la logica che se non uso questo blocco da un po', probabilmente ho finito di utilizzarlo

### LRU o Random?

La politica LRU soffre di pattern di accesso patologici (ciclici e periodici) che portano ad avere un hit-rate pari a 0

- A, B, C, D, E, A, ...
- sostituisco proprio il blocco che mi serve per il prossimo accesso

**Set thrashing**:

- When the “program working set” in a set is larger than set associativity
  - program working set == insieme degli indirizzi a cui il programma accede
- Il set thrashing si verifica quando troppi blocchi di memoria vengono mappati nello stesso set, superando il numero di slot disponibili.
  - Tutti quei blocchi competono per lo stesso set.
  - La cache deve continuamente sostituire blocchi nel set, anche se altri set restano inutilizzati.
  - Il risultato è un alto tasso di miss, anche se la cache nel complesso non è piena (situazione simile a conflict-misses)
- in generale è quando un accesso invalida quello precedente ciclicamente
- Random replacement policy is better when thrashing occurs

In practice:

- Depends on workload
- **Average hit rate of LRU and Random are similar**

## Handling writes

dirty-bit mi indica che un determinato blocco è stato scritto, senza aver aggiornato il relativo blocco in memoria (o i livello sottostanti nella gerarchia)

nota: il bit di validità mi serve effettivamente solo quando abbiamo più core che devono accedere alla cache (protocolli di coerenza)

Quando scrivere i dati modificati nel livello successivo della cache?

- Write through: Nel momento in cui avviene la scrittura
- Write back: Quando il blocco viene rimosso

Abbiamo visto che quando abbiamo una miss in lettura carichiamo il blocco in cache, vale lo stesso anche in scrittura?

Allochiamo un blocco di cache in una write miss?

- Alloca in caso di write miss (Write Allocate): Yes
- Non-alloca in caso di write miss (Write Around): No

## should we unify instruction-cache and data-cache?

Tenere separate le cache per le istruzioni e la cache dei dati ci fornisce di fatto due memorie separate il che previene alee strutturali verso la memoria tra gli stadi di fetch e writeback

Inoltre, i pattern di accesso sono molto diversi, unificarli potrebbe causare thrashing

I primi livelli sono sicuramente separati, tuttavia dal secondo livello in poi vengono di solito unificate

- l'overhead di avere cache separate consiste in: frammentazione e area occupata maggiore a parità di dimensione
- unificarle quando cominciano ad essere grandi diventa conveniente

# Esempi

notare che nel secondo esempio con le miss, l'avere la cache porta a:

- spostare 32byte dalla memoria nella cache rispetto a 8 se non l'avessimo avuta
  - banda maggiore utilizzata
- avere una latenza maggiore dato che dobbiamo controllare prima la cache della memoria

in questo caso di alto miss-rate l'avere la cache è peggio che non averla

...

# I vari tipi di miss

1. Miss obbligatorie (Compulsory miss)
    - first reference to a block always results in a miss
    - è impossibile avere una hit su un blocco che non ho mai caricato in cache

2. Miss di Capacità (Capacity miss)
    - cache is too small to hold the program working set (la cache è piena)
    - qualcosa verrà per forza evictato e al prossimo riferimento si avrà una miss

3. Miss di Conflitto (Conflict miss)
    - defined as any miss that is neither a compulsory nor a capacity miss
    - causate da un numero di vie troppo limitato della cache
    - anche se c'è spazio in cache, non sono riuscito a mantenere il mio blocco in cache dato che è stato sostituito da un altro con lo stesso set-id. Al prossimo riferimento avrà una miss

## Iperparametri delle cache

Con blocchi di dimensione maggiore

- **il numero di set diminuisce (a parità di dimensione della cache)**
- località spaziale migliore
  - meno miss compulsorie
- località temporale peggiora
  - diventa più probabile che qualcuno spinga via il mio blocco -> miss di capacità
  - diventano più probabili anche i conflitti (ho meno set-id) -> miss di conflitto
- C'è un tradeoff nella scelta della dimensione del blocco
  - Blocchi troppo piccoli non sfruttano bene la località spaziale
  - Blocchi troppo larghi -> tanto replacement -> non sfrutta bene la località temporale

Con cache di dimensione maggiore

- si ha più località temporale dato che i dati rimangono in cache per più tempo prima di essere rimpiazzati (se associatività sufficentemente alta)
- quelle che prima sarebbero state miss di capacità diventano hit
- cache più grandi hanno anche latenze più elevate e quindi c'è un tradeoff
  - cache troppo piccole rimpiazzano costantemente i dati utili
  - cache troppo grandi hanno latenze elevate anche in caso di hit
  - la grandezza della cache dovrebbe essere grande tanto quanto il working set e non di più

Grado di associatività

- Larger associativity
  - minore miss rate (reduce i conflitti)
  - maggiore hit latency e costo area (plus diminishing returns)
- Smaller associativity
  - minor costo
  - minore hit latency
  - Molto importante per L1 caches

## Come ridurrre i vari tipi di miss

per ridurre le miss compulsorie:

- aumentare la dimensione del blocco può aiutare se si ha alta località spaziale nel pattern di accesso del programma
- Prefetching: Anticipate which blocks will be needed soon

per ridurre miss di conflitto:

- aumentare associatività della cache

per ridurre miss di capacità:

- Utilize cache space better: keep blocks that will be referenced again in cache
- questo si traduce in:
  - usare politica LRU per rimuovere il blocco con meno probabilità di essere riutilizzato (quello con meno località temporale)
  - Software management: divide working set and computation such that each “computation phase” fits in cache
  - Al termine di ogni computation fase possiamo caricare la cache con un nuovo mini-working set (pagando le miss compulsorie) e dimenticarci dei dati precedenti che non verranno più referenziati

# Software Approaches for Higher Hit Rate

esempio slide 54: notare come ci siano solo miss compulsorie dato che ogni dato viene acceduto (doppiamente) solo una volta

esempio matmul:

- z non ha località spaziale dato che viene acceduto sulla colonna
- vogliamo quindi cercare di farcelo stare in cache per sfruttare località temporale
- sfruttiamo tiling per diminuire la dimensione del working set e diminuire le miss di capacità (?)

trasformare array di strutture in strutture di array
