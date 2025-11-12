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

- spostare 32byte dalla memoria nella cache rispetto a 8 se non l'avessimo avuto
  - banda maggiore utilizzata
- avere una latenza maggiore dato che dobbiamo controllare prima la cache della memoria

in questo caso di alto miss-rate l'avere la cache è peggio che non averla

...

Con blocchi di dimensione maggiore il numero di set diminuisce (a parità di dimensione della cache)

- questo significa che conflitti diventano più probabili
- località temporale peggiora
  - diventa più probabile che qualcuno spinga via il mio blocco
  - località temporale peggiore significa più miss dovute a conflitti
- località spaziale migliore
  - località spaziale peggiore significa più miss compulsorie

Miss compulsory

- miss causate dal primo accesso ad un determinato blocco che non è ancora mai stato caricato in cache
- avvengono la prima volta che la cpu accede ad un determinato blocco
- sono inevitabili

miss di capacità sono quelle dovute a working set troppo grandi anche per cache ad associatività completa

anche per cache ad associatività completa, è possibile ad avere delle miss di conflitto a causa della politica di replacement
