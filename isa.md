# ISA RISC-V
le isa cisc sono utili per i programmatori che scrivono codice direttamente in assembly in quanto offrono una semantica più ricca. Questo però tuttavia non è più necessario in quanto al giorno d'oggi scriviamo in linguaggi di alto livello e sfruttiamo dei compilatori.

isa cisc con le loro caratteristiche (istruzioni di lunghezza variabile, ...) portano ad una microarchitettura complessa e quindi poco efficente
- inoltre, microarchitetture semplice -> efficenza energetica maggiore -> ottimi per dispositivi a batteria -> ARM

processori intel traducono codice CISC x86 -> in microcodice RISC

interessante: avere un isa modulare obbliga a chi deve fornire codice precompilato a dover conoscere esattamente l'isa e i moduli utilizzati. Questo ha portato a definire vari **profili** che congelano alcuni moduli in un pacchetto predefinito per supportare la compatibilità



riscv è un isa register-register
- le uniche istruzioni che possono accedere alla memoria sono le load e le store
    - sono le uniche che possono avere locazioni di memoria come operandi 
    - l'unica modalità di indirizzamento è registro + immediato
- tutte le altre istruzioni hanno come operandi i registri


i registri sono grandi quanto la dimensione della word nella architettura in considerazione
- in RV64I i registri sono grandi 64bit; in RV32I, 32 bit; ecc...



il fatto che x0 sia sempre 0 permette di non avere una istruzione mov
- se voglio spostare il contenuto di un registro in un altro, posso fare: add x1, x0, x7 ; x1 = x7


riscv codifica nel codice menmonico delle istruzioni il tipo di dato trattato (es. ld sta per load double)



### istruzioni store e load
nb: differiscono in formato in quanto le store hanno due registri sorgente, mentre le load hanno un registro sorgente ed uno destinazione
- questo è importante quando si considera la posizione degli indici dei registri nel formato delle istruzioni
- i registri (destinazione?/sorgente?) si trova sempre nella stessa posizione

dato che i registri sono a 64 bit, quando carico un dato di dimensione minore bisogna estendere il bit di segno nel caso signed, scrivere degli zeri nel caso unsigned
- per questo motivo esitono delle load signed e unsigned



### istruzioni condizionali
le label contengono un **offset relativo** che indica di quanto bisogna spostare il pc quando si salta
- se il salto fosse specificato come indirizzo assoluto
    - si creerebbe una relocation entry aggiuntiva non necessaria
    - l'indirizzo occuperebbe molti byte nell'istruzione (8 byte per macchine a 64bit)
- l'assembler si occupa di calcolare l'offset



### Come vengono rappresentate in codice macchina le istruzioni?

6 formati di istruzione


al contrario degli immediati negli altri formati di istruzione, gli immediati nelle istruzioni di branch sono sempre pari (multipli di 2)
- questo perchè nelle istruzioni di branch stiamo modificando il pc
- le istruzioni sono sempre grandi 4 byte o 2 per istruzioni compresse
- non ha quindi senso saltare a indirizzi dispari



jump and link => salta e salva indirizzo di ritorno nel registro destinazione
