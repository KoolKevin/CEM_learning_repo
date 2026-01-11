...

Con memoria virtuale, possiamo vedere la memoria come cache per il disco

- ora i processi possono generare più indirizzi di quelli che possono stare in memoria

Le pagine posso vederle come cache line

La MMU è quel pezzo di hardware che si occupa di capire se un determinato virtual address è presente, e se si, dove in memoria (simile al tag store di una cache)

Memory Management è un qualcosa che sta a metà tra hardware e software

- hardware si occupa della traduzione degli indirizzi virtuali emessi
- software si occupa di tutto il resto

ogni pagina nella tabella delle pagine ha 3 stati:

- cached: indirizzi virtuali associati alla pagina sono stati usati dal processo in passato, e si trovano in memoria (indirizzi virtuali mappati ad indirizzi fisici)
- uncached: il processo ha usato gli indirizzi virtuali associati alla pagina in passato, ma non si trova in main memory
- unallocated: gli indirizzi virtuali associati alla pagina non sono mai stati usati
  - è molto probabile che la maggioranza delle pagine siano unallocated dato che lo spazio di indirizzamento disponibile a ogni processo è enorme

## MMU

si occupa della traduzione degli indirizzi e di accellerare quest'ultima

..

## Quanti accessi in memoria devo fare con la memoria virtuale?

- 2, uno per trovare la traduzione dell'indirizzo virtuale e l'altro per accedere all'indirizzo fisico effettivo

**Quindi se introduciamo memoria virtuale raddoppiamo tutti gli accessi alla memoria?**

Da qui la prima necessità di introdurre una logica hardware

- il programma non emette più load e store verso il bus degli indirizzi verso la memoria, emette load e store verso la MMU
- solo grazie alla logica hardware la memoria virtuale diventa trasparente ai programmi

TLB

take home message: senza particolari precauzioni, la memoria virtuale raddoppia gli accessi alla memoria e potrebbe causare latenze pesanti in caso di page fault

l'ISA determina la dimensione dello spazio di indirizzamento virtuale e la dimensione delle pagine

...

siccome la memoria virtuale è molto più grande rispetto a quella fisica, è probabile che si incorra nella necessità di dover rimpiazzare una pagina fisica

- la tabella delle pagine contiene informazioni riguardanti la politica di rimpiazzo delle pagine fisiche
- notiamo poi che mentre nelle cache la scelta su quale blocco rimpiazzare viene fatta in hardware dato che la latenza è molto importante, nel caso di un page fault la scelta su quale pagine sostituire può essere fatta in software dato che la latenza di questa scelta è minuscola rispetto a quella dell'operazione di swap da disco

Quali informazioni deve conoscere la MMU per accedere ad una PTE?

- PTBR + VPN*sizeof(PTE)

### Page fault e interrupt imprecisi

se un processo accede ad un indirizzo non mappato nella tabella delle pagine, questo scatena un page fault.

Nel caso la pagina sia presente nel disco questo page fault viene gestito dal SO per fare lo swap.

Al termine di questo handler, ritorniamo nel codice del processo e rieseguiamo l'accesso che aveva generato il page fault al fine di accedere effettivamente in memoria

- la pipeline deve riemettere lo load/store medesima

**NB**: se però abbiamo una pipeline che esegue istruzioni fuori ordine, potremmo aver già eseguito delle istruzioni che vengono dopo all'accesso in memoria che ha generato l'eccezione, e quindi diventa difficile capire in che punto del programma tornare indietro. Questo è ciò che si intende con interrupt impreciso

In questi casi diventa fondamentale avere un reorder buffer (che ricorda tiene traccia di quali istruzioni generano delle eccezioni) che emette le eccezioni generate solamente qunado fa il commit in ordine delle istruzioni.

- il ROB solleva eccezioni solo al momento del commit, rendendole precise.

# Multilevel page table

riesco a far stare in memoria la tabella delle pagine, ma ho più accessi in memoria

la MMU mette a disposizione una logica per fare il page table walking

## ASID

il PID del processo viene utilizzato per disambiguare lo stesso va di processi diversi

# TLB

con tabelle delle pagine gerarchiche abbiamo tanti accessi in memoria

questa è una cache che salva le traduzioni VPN -> PPN

- completamente associativa
- piccola (16/32 pte), dato che deve essere super veloce

tengo in memoria le ultime 16/32 traduzioni degli indirizzi

una miss sul TLB porta a fare walk delle tabelle delle pagine

esiste un TLB sia per istruzioni che per dati

- dato che gli indirizzi sono totalmente diversi

**NB**: da qui sorge la necessità di disambiguare VPN di processi diversi. Se non sto attento faccio accedere istruzioni/dati di un processo ad un altro processo. Per questo motivo si incorpora il PID del processo con il VPN nel TLB

- il TLB è condiviso dai processi, mentre le tabelle delle pagine no. Senza TLB quindi VPN identici non danno problemi
- una soluzione naive potrebbe essere fare il flush del TLB ad ogni cambio di contesto...
  - pago un sacco di TLB miss
  - ad ogni cambio di contesto devo ririempire il TLB

il TLB ha alta località temporale

- codici con alto TLB miss sono codici che accedono a dati molto distanti in memoria (salti >= della dimensione della pagina, vedi grafi oppure immagini 4k in cui righe diverse stanno a pagine diverse)

# MMU

slide 43 contiene un buon riassunto

MMU fa parte del processore/core

abbiamo che le cache contengono indirizzi fisici già tradotti

la MMU contiene 4 cose

- un registro che contiene l'indirizzo base della tabella delle pagine
- una logica che fa  page table walking
- TLB
- una logica che solleva una eccezione in caso di mancata traduzione durante il page table walk o in caso di violazione dei bit di accesso

# Memory protection

...

i livelli di privilegio possono anche definire quali regioni di memoria possono essere accedibili dal processo

- per esempio, pensa alle pagine trampolino e trapframe in xv6

nota: se si esegue del codice in ad un livello di privilegio analogo a machine mode, tipicamente li la MMU è disattivata e quindi tutti gli indirizzi emessi sono fisici

## Virtual memory a supporto della virtualizzazione

in ISA che supportano nativamente la virtualizzazione, si ha un livello di traduzione delle pagine ulteriore

in generale, nei sistemi che supportano nativamente (accelerano) la virtualizzazione si possono fare due operazione macroscopiche aggiuntive

- poter avere un livello superiore di tabella delle pagine per i singoli OS che si sta virtualizzando
  - così come si tiene traccia della memoria accedibile ai processi con una tabella delle pagine
  - l'hypervisor tiene traccia della memoria accedibile al VM tramite una tabella delle pagine
- poter propagare le eccezioni ai SO che si sta virtualizzando in maniera tale che le eccezioni generate a livello HW siano visibili solamente al SO che ha scatenato l'eccezione

# Complessità che emergono con la memoria virtuale

- Homonym: Same VA can map to two different PAs
  - VA is in different processes
- Synonym: Different VAs can map to the same PA
  - Different pages can share the same physical frame within or across processes

accedo alla cache con indirizzi fisici o virtuali?

- se uso indirizzi fisici
  - prima di accedere alla cache devo fare la traduzione e quindi aspetto un sacco
- se uso indirizzi virtuali
  - ho omonimi e sinonimi
  - in particolare sono gli omonimi ad essere problematici dato che mi faccio hit ma accedo ad un indirizzo di un altro processo
  - **devo fare flush ad ogni context switch**
    - se lo stesso VA può puntare a PA diversi (dati diversi) in caso di context switch, la cache conterrà un dato stale ma **ci sarà comunque un hit dato che il VA è lo stesso**

## VIPT caches

Virtual-physical caches sono molto comuni per L0

Siccome l'accesso a una cache ha due stadi (selezione del set e controllo del tag), e dato che **solamente dopo aver selezionato il set possiamo controllare il tag**, nelle VIPT caches l'idea è **tradurre l'indirizzo contemporaneamente alla selezione del set e poi confrontare i tag dei risultati**

- **accesso contemporaneo a cache e TLB**
- virtual tag e physical index
  - nel mentre si accede al set con i bit di indice, contemporaneamente si svolge la traduzione dei bit di tag
- **NB**: questa cosa funziona bene se la capacità della cache rispetta la condizione, ovvero **se i bit di indice ricadono tutti nei bit di page offset e quindi non cambiano tra indirizzo virtuale e fisico**
  - siccome i bit di indice non cambiano, è come se stessimo accedendo alla cache dopo aver fatto la truduzione dell'indirizzo virtuale
  - i bit di tag sono invece quelli dell'indirizzo fisico (d'altronde se abbiamo una miss, abbiamo anche tempo di fare la traduzione)
  - questo è il motivo per cui le cache di livello 0 hanno la stessa dimensione da anni, sono vincolate dalla formula vista sopra (pagine fisse a 4k)
- in questa maniera riusciamo ad avere i pregi di entrambi gli approcci
  - non devo aspettare la traduzione prima di accedere alla cache -> tempo di accesso molto più veloce
  - non ho problemi con omonimi e quindi non devo fare flush dopo context switch

Verifichiamo la condizione:

- abbiamo che la capacità di una cache è data da: C = Numero di set * (line_size × associativity)
- ma allora la condizione: C ≤ page_size × associativity
- diventa: numero_set * line_size ≤ page_size
- e se si considera il numero di bit dei termini tutto torna

Se la condizione non è rispettata

- abbiamo che VA diversi che puntano allo stesso PA (sinonimi) apparterranno a set diversi nella cache
  - prima invece questo non accadeva dato che i bit di indice di PA e VA erano identici
- siccome però il TAG è definito dal PPN (o meglio il pezzo senza i bit di indice), e quest'ultimo è uguale, **abbiamo che lo stesso PA esisterà in due posti nella cache**
- sprechiamo spazio in e creiamo problemi di coerenza
  - se faccio una scrittura ad uno dei due VA sinonimi, attivo la logica di HIT, e aggiorno solo una delle due locazioni dato che i bit di indice sono diversi
  - creo aliasing
