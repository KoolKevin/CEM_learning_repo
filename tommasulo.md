Nel primo esempio con scoreboarding, la fsub non riesce a completare issue al ciclo 3 dato che trova l'unità funzionale occupata nella entry dello scoreboard, nonostante la FU non sia effettivamente occupata.

- **alea strutturale sull'unità scoreboard**, non sulla FU

Secondo esempio

Nel terzo esempio siamo obbligati a eseguire le istruzioni in quell'ordine a causa del dataflow. Rinominando i registri risolviamo le dipendenze di nome e possiamo eseguire fuori ordine

# Algoritmo di Tommasulo

Introduces register renaming in hardware

rimuove il FU status register e lo sostituisce con il concetto di Reservation Station

- sostanzialmente delle code di istruzioni con metadati
- gli operandi vengono immagazzinati nella reservation station appena disponibili
- l'id della reservation station può anche dirmi la sorgente di un operando
  - reservation station implementano register renaming

i tag identificano una entry di una reservation station

Ogni unità funzionale ha associata una reservation station che fa da buffer per le istruzioni

Ogni reservation station è una tabella formata da:

- Busy: questa indica **se la reservation station è libera**, NON la FU. questo campo viene utilizzato durante issue
- Op
- qj, qk: tag della reservation station che contiene l'istruzione che fornirà gli operandi
- vj, vk: valori degli operandi

Ogni entry nel RF ha un Register Result Status associato analogo a quello dell'algoritmo di scoreboarding

Durante l'issue di un'istruzione

- controlliamo se ci sono alee strutturali: c'è qualche reservation station (della FU desiderata) libera?
- copiamo il valore dei registri sorgente (incluso il tag Q) nelle entry della reservation station
- aggiorniamo il RF marchiando il registro sorgente come invalido e scrivendo il tag della reservation station
- **NB**: adesso, in issue copiamo e basta dal RF, l'unico controllo che facciamo è quello sulla disponibilità di una reservation station
- **NB2**: notiamo che in Tommasulo non abbiamo più uno stadio di read operands distinto
  - leggiamo gli operandi (oppure il tag di chi li produce) durante issue quando copiamo dal RF

i risultati prodotti dalle FU vengono fatti viaggere sul Common Data Bus

il CDB entra in input alle reservation station e al RF

sul CDB i dati viaggiano con una forma <tag, val>

- il tag indica chi ha prodotto il risultato con valore val
- chiunque stava attendendo un operando () lo aspetta da un entità indicata dal suo tag

Passo da una modalità in cui gli operandi venivano sempre letti dal RF, ad una modalità in cui leggo dal RF in issue e, in caso non siano pronti in quello stadio, leggo dal CDB

Sto facendo renaming degli operandi, con il nome del tag

Durante l'issue si segna la entry del RF relativa al registro destinazione dell'istruzione, il tag della reservation station in cui verranno parcheggiata l'istruzione da eseguire. In questo modo si marchia la entry nel RF come invalida (esattamente come avveniva nel register result status dello scoreboarding)

La reservation station si libera quando vede circolare sul CDB il proprio tag -> in WR

slide 9 -> IS al posto di ID

Nel caso della memory unit abbiamo un po' di casino

- in generale, le store devono essere eseguite in program order
  - altrimenti WAW
- le load e le store devono essere eseguite seguendo l'ordine reciproco all'interno del programma
  - altrimenti WAR

load buffer ~= reservation station per l'istruzione di load

nel caso di istruzioni di store, oltre all'indirizzo con cui vogliamo accedere alla memoria, abbiamo anche il valore che vogliamo scrivere.

Questo potrebbe essere una entry nel RF, oppure tag per reservation station che mi produrranno il risultato

l'ordine delle istruzioni di load/store viene serializzato dalla address unit

- l'ordine del programma viene rispettato

### implementazione hardware delle reservation station [16:10 - 20/10]

quello che sembra un mini FFD rappresenta il reset

aumentare il numero di Reservation station implica

- aggiungere tutto il blocco 'Reservation Station' più volte
- aumentare la dimensione del multiplexer per la logica di arbitraggio per la FU

# Gestione delle alee con Tommasulo

Le alee strutturali possono avvenire su

- reservation station durante issue
- sulle unità funzionali dopo issue ma con unità funzionale occupata (consideriamo le unità funzionali bloccanti)
  - consideriamo uno stato di attesa dopo issue

Come mai non ci sono più alee RAW?

- le entry nel RF possono essere marchiate come valide o non valide grazie al campo Q
- durante issue copiamo la entry nel RF nella reservation station
- se uno degli operandi non è valido l'istruzione rimarrà in uno stato di wait nella reservation station fino a quando l'operando sarà disponibile (= prodotto dall'unità funzionale)
- consideriamo uno stato di attesa nella reservation station dopo issue

Come mai non ci sono più alee WAW?

- se faccio issue di un istruzione con una antidipendenza, nel RF andrò a sovrascrivere il tag dell'istruzione precedente
- siccome le entry del RF vengono copiate, chi aspettava il tag precedente continua ad aspettarlo quando viene pubblicato sul CDB. Chi aspetta il tag nuovo (vedi RF) non sarà sensibile alla publlicazione sul CDB del risultato associato al vecchio tag
  - è impossibile sovrascrivere il registro con il valore vecchio
- l'algoritmo di tommasulo mi ha implementato il renaming dei registri con i tag

Come mai non ci sono più alee WAR?

- sempre perchè stiamo facendo renaming

# Esempio Tommasulo

in IS

- nel primo semiperiodo controllo se c'è una RS disponibile
- nel secondo semiperiodo si accede al RF

in WR

- nel primo semiperiodo si scrive il risultato sul CDB
- e RF e RF controllano ed eventualmente si aggiornano

NB: questa scelta potrebbe causare stalli aggiuntivi

# Tommasulo vs scoreboarding

...

- con Tommasulo abbiamo una logica per le gestione degli accessi alla memoria
  - operazioni di load/store producono risultati che vengono propagati sul CDB
    - le load producono il dato proveniente dalla memoria
    - le store producono niente
  - le reservation station delle load contengono
    - q, v dell'operando sorgente
    - immediato e address con cui accedere in memoria
    - come prima il RF si salva che il registro destinazione verrà prodotto dalla load/store unite con un tag
  - le reservation station delle store contengono
    - la stessa roba ma con due q e v dato che ho due registri sorgente

  - **NB**: le reservation station per le load/store devono rispettare delle logiche di ordinamento e quindi diventano delle code/buffer nel senso che le entry nella reservation station vengono messe in esecuzion in ordine
    - le reservation station per l'unità load/store vengono gestite in maniera particolare a causa di questi vincoli di ordinamento

## Chiedi per instruction queue. Non facevamo fetch di un'istruzione alla volta?
