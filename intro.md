# end of Dennard Scaling









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

