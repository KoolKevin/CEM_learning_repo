slide 4

- stiamo facendo un'assunzione molto forte
- stiamo assumendo che il trasferimento dei dati e la parte di compute possano avvenire in completa concorrenza
  - non ci sono dati che dobbiamo prendere che si basano su dei calcoli precedenti
- questa è l'assunzione del modello roofline

la retta che delimita il bound della bandwidth è una retta (x*k) dove la pendenza è data dalla bandwidth della memoria

- siccome siamo in log scale la retta non interseca l'asse x nell'origine

punto di machine balance

- questo è il punto in cui noi vorremmo che le nostre applicazioni eseguissero in quanto è il più efficente
- se ci troviamo in questo punto
  - con una AI appena più bassa siamo limitati dalla banda -> esubero di fpu unit
  - con una AI appena più alta siamo limitati dalla capacità di calcolo dell'architettura -> esubero di banda
- se siamo nel punto di machine balance non stiamo sprecando niente
- ogni architettura nasce con un punto di machine balance preciso e quindi nasce per eseguire determinate applicazioni

...

tutto quello che abbiamo visto sopra è dal punto di vista teorico

## Le misure reali sono vicine ai limiti teorici?

...

se siamo memory bound:

- non stiamo sfruttando tutto l'hardware che abbiamo a disposizione
- potremmo risparmiare soldi comprando un calcolatore meno potente (meno core, non ha AVX512, ...)

se siamo compute bound:

- comprare un calcolatore più grosso migliora le performance
- migliorare solo il sistema di memoria non migliora le performance

Missing lines = mancano delle rete per i tetti

ogi livello di memoria ha nella pratica la propria AI
