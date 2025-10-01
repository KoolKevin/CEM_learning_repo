Lo stesso vale anche per pipeline lunghe
- ho critical path più corti e quindi posso aumentare la frequenza di clock
- ma ho più alee di dato e quindi il CPI medio si abbassa
- questo ed altri problemi
fino ad ora abbiamo visto pipeline scalari
- un'istruzione alla volta può essere issued

Un'altro modo per aumentare ILP e passare ad architetture superscalari, ovvero con più pipeline

...

possiamo avere alee che ci portano ad avere un CPI >=1

T/S + C
- c è un residuo dovuto al tempo di setup dei registri buffer tra gli stadi

...



Con superscalarità il CPI diventa >= 1 / P
- ottengo uno speedup teorico pari a P!

Abbiamo però delle problematiche:
- alee strutturali! Se abbiamo un grado di superscalarità pari a 4
    - abbiamo 4 volte gli stadi di ID e WB che accedono al R
        - la progettazione logica del RF diventa complessa ed occupa più area dato che dobbiamo pensare 4 porte in lettura e scrittura per accessi contemporanei
    - stesso discorso per la MEM
- alee di controllo
    - che indirizzi prendiamo in 4 IF contemporanei

Quand'è che architetture superscalari diventano convenienti?
- specializzare per istruzioni specifiche determinate pipeline rispetto ad altre
    - es. con grado di superscalarità 2, potremmo mettere su una pipeline le istruzioni intere e sull'altra quelle fp. Queste istruzioni hanno RF separati e quindi non abbiamo alee strutturali (o poche in caso di istruzioni di conversione)

In generale architetture superscalari di grado 2/4 risultano convenienti


