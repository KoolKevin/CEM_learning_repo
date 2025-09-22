# RISC vs CISC
le isa cisc sono utili per i programmatori che scrivono codice direttamente in assembly in quanto offrono una semantica più ricca.

Questo però tuttavia non è più necessario in quanto al giorno d'oggi scriviamo in linguaggi di alto livello e sfruttiamo dei compilatori.

isa cisc con le loro caratteristiche (istruzioni di lunghezza variabile, ...) portano ad una microarchitettura complessa e quindi poco efficente

inoltre, microarchitetture semplice -> efficenza energetica maggiore -> ottimi per dispositivi a batteria -> ARM

processori intel traducono codice CISC x86 -> in microcodice RISC

interessante: avere un isa modulare obbliga a chi deve fornire codice precompilato a dover conoscere esattamente l'isa e i moduli utilizzati. Questo ha portato a definire vari **profili** che congelano alcuni moduli in un pacchetto predefinito per supportare la compatibilità


# ISA RISC-V
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
- il registro (destinazione?/sorgente?) si trova sempre nella stessa posizione

dato che i registri sono a 64 bit, quando carico un dato di dimensione minore bisogna estendere il bit di segno nel caso signed, scrivere degli zeri nel caso unsigned
- per questo motivo esitono delle load signed e unsigned

### istruzioni condizionali
le label contengono un **offset relativo** che indica di quanto bisogna spostare il pc quando si salta
- se il salto fosse specificato come indirizzo assoluto
    - si creerebbe una relocation entry aggiuntiva non necessaria
    - l'indirizzo occuperebbe molti byte nell'istruzione (8 byte per macchine a 64bit)
- l'assembler si occupa di calcolare l'offset



# Come vengono rappresentate in codice macchina le istruzioni?
6 formati di istruzione

al contrario degli immediati negli altri formati di istruzione, gli immediati nelle istruzioni di branch sono sempre pari (multipli di 2)
- questo perchè nelle istruzioni di branch stiamo modificando il pc
- le istruzioni sono sempre grandi 4 byte o 2 per istruzioni compresse
- non ha quindi senso saltare a indirizzi dispari

jump and link => salta e salva indirizzo di ritorno nel registro destinazione

...

### Modalità di indirizzamento
**NB**: con modalità di indirizzamento non intendiamo solo come si accede alla memoria. Piuttosto, intendiamo come una qualsiasi istruzioni trova (indirizza) i suoi operandi

In risc-v ci sono 4 modalità di indirizzamento:

1. Immediate addressing
    - L’operando immediato è contenuto direttamente nell’istruzione.
    - Esempio:
        - addi x5, x0, 10   # x5 = 0 + 10 → carica il valore immediato 10 in x5
    - Qui il valore 10 è direttamente dentro l’istruzione.
2. Register addressing
    - L’operazione usa solo registri (nessun immediato o memoria).
    - Esempio:
        - add x6, x5, x4    # x6 = x5 + x4
    - Tutti gli operandi sono registri.
3. Base addressing (Register + Immediate)
    - Si usa un registro come base + un offset immediato per accedere alla memoria.
    - Esempi:
        - lw x10, 0(x5)     # carica in x10 la word all’indirizzo contenuto in x5
        - sw x11, 4(x5)     # salva x11 in memoria all’indirizzo (x5 + 4)
    - Qui x5 contiene un indirizzo base, l’immediato è l’offset.
4. PC-relative addressing
    - L’indirizzo è calcolato come PC + immediato.
    - Esempi:
        - jal x1, 16        # salta a PC+16 e salva il ritorno in x1 (ra)
        - beq rs1, rs2, L1
    - Tipicamente usato per i salti



### Floating point
si usa un register file separato sui cui le istruzioni fp agiscono



