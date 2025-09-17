# end of Dennard Scaling
ricorda la formula per la potenza di un transitor: `P = alpha*C*V^2*f`

Il Dennard Scaling (1974) diceva che se rimpicciolivi i transistor (riducendo la lunghezza del canale L, e lo spessore dell’ossido di gate), allora:
- la tensione di alimentazione V poteva essere ridotta proporzionalmente,
- la corrente e la potenza per unità di area restavano costanti,
- la frequenza di clock poteva aumentare,
- quindi la potenza totale del chip rimaneva gestibile.

In pratica:
- diminuire le dimensioni del transistor mi riduce proporzionalmente la capacità -> posso raddoppiare la frequenza
- con transistor grandi la metà, posso mettercene il quadruplo nella stessa area -> questo fattore viene compensato con il dimezzamento della tensione

Negli anni 2000 è arrivato il limite fisico dello spessore dell’ossido di gate:
- Non si può renderlo infinitamente sottile (si è arrivati a pochi atomi di spessore).
- Se diventa troppo sottile, gli elettroni attraversano l’ossido per effetto tunnel → grandi correnti di perdita (leakage).
- Quindi non si può più abbassare la tensione di alimentazione V senza peggiorare la stabilità e le perdite.

Tuttavia si può continuare a rimpicciolire i transitor in lunghezza e larghezza 
- il fattore capacità nella formula della potenza dinamica viene ridotto
- la legge di moore continua ad essere valida (seppur rallentata)

Cosa comporta questo
- V non scende più come dovrebbe → la potenza dinamica 𝑃 ∝ 𝑉^2 x 𝑓 x 𝐶  non si riduce abbastanza.
- La frequenza di clock f non può crescere (perché più frequenza = più calore).

Risultato: i transistor continuano a diventare più piccoli, ma i chip scaldano troppo se proviamo a seguire la vecchia legge di Dennard.

**Come si è andati avanti**
- Non potendo più aumentare la frequenza, si è aumentato il numero di core per chip
    - questo fa crescere la potenza linearmente compensando la diminuizione legata alla riduzione della lunghezza
- nasce la programming crisis (serve parallelizzare i software).

L’ = L / 2 → dimezzo la lunghezza del canale.
V’ = V / 2 → dimezzo anche la tensione.
F’ = F × 2 → posso raddoppiare la frequenza.
D’ = 1 / L² = 4D → aumento della densità dei transistor (quattro volte più transistor per area).
P’ = P → la potenza rimane costante, anche se ho più transistor e più velocità → tutto fantastico.

👉 Questo era il Dennard scaling classico: prestazioni migliori senza aumento dei consumi.

V’ = V / 2 → do not hold anymore!

Quindi, in realtà:
V’ ≈ V (non dimezza più).
F’ ≈ F × 2 → la frequenza teoricamente potrebbe crescere, ma nella pratica è limitata dal calore.
D’ = 4D → la densità di transistor continua ad aumentare.
P’ = 4P → qui sta la catastrofe: la potenza per unità di area esplode!








# architetture moderne
la differenza tra in ordine e superscalare out-of-order è puramente architetturale. Entrambe possono eseguire la stessa ISA

per continuare a scalare mantenendo l'efficenza, bisogna specializzare le architetture per un determinato workload

# iron law
abbiamo tre termini, quali termini vengono influenzati dalla tecnologia? quali dall'architettura? quali dall'isa?
- frequenza di clock 
  - influenzata dalla tecnologia
- cpi
  - influenzati dall'isa
  - ma anche dalla tecnologia (considera per esempio istruzioni di load/store dalla memoria, se miglioro la memoria miglioro i cpi per queste istruzioni)
  - influenzati anche dall'architettura
    - considera per esempio un architettura che mi fa stallare molto a confronto con una che evita stalli
- num istruzioni
  - influenzati dall'isa


# obiettivi del corso

