Dynamic scheduling, a technique by which the hardware reorders the instruction execution to reduce the stalls while maintaining data flow (== rispettare le dipendenze di dato) and exception behavior.

Dynamic scheduling offers several advantages:

- First, it allows code that was compiled with one pipeline in mind to run efficiently on a different pipeline, eliminating the need to have multiple binaries and recompile for a different microarchitecture
  - se ti ricordi, lo static scheduling del compilatore dipendeva dalla specifica microarchitettura
- Second, it enables handling some cases when dependences are unknown at compile time; for example, they may involve a memory reference or a data-dependent branch, or they may result from a modern programming environment that uses dynamic linking or dispatching.
- Third, and perhaps most importantly, it allows the processor to tolerate unpredictable delays, such as cache misses, by executing other code while waiting for the miss to resolve.

The advantages of dynamic scheduling are gained at the cost of a significant increase in hardware complexity.

### Idea

A major limitation of simple pipelining techniques is that they use in-order instruction issue and execution:

- instructions are issued in program order
- and if an instruction is stalled in the pipeline, **no later instructions can proceed**.

If instruction j depends on a long-running instruction i, currently in execution in the pipeline, then all instructions after j must be stalled until i is finished and j can execute.

- If there are multiple functional units, **these units could lie idle**.

For example, consider this code:

```
fdiv.d f0,f2,f4
fadd.d f10,f0,f8
fsub.d f12,f8,f14
```

- The fsub.d instruction cannot execute because the dependence of fadd.d on fdiv.d causes the pipeline to stall
- yet, **fsub.d is not data-dependent on anything in the pipeline.**
- This hazard creates a performance limitation that **can be eliminated by not requiring instructions to execute in program order.**
  - Siccome queste pipeline permettono ad istruzioni indipendenti di proseguire in caso di stalli, vengono anche dette **non bloccanti**

The main idea in non blocking architectures is incrementing ILP by allowing execution of independent instructions even if out of program order

### Come funziona?

In the classic five-stage pipeline, both structural and data hazards could be checked during instruction decode (ID):

- when an instruction could execute without hazards, it was issued from ID

Now we want an instruction to begin execution as soon as its operands are available even if a predecessor is stalled.

To allow us to begin executing the fsub.d in the preceding example, **we must separate the issue process into two parts**:

- checking for any structural hazards (and decode the instruction)
  - emettiamo un'istruzione solo se la FU è libera
- and waiting for the absence of a data hazard.
  - cominciamo ad eseguire un'istruzione solo se i suoi operandi sono pronti (=== non ci sono data hazard)

In a dynamically scheduled pipeline

- all instructions pass through the issue stage in order (**in-order issue**)
  - in-order instruction issue (i.e., instructions issued in program order)
- however, they can be stalled or can bypass each other in the second stage (read operands)
- and thus enter execution out of order because we want an instruction to begin execution as soon as its data operands are available.
- Such a pipeline does **out-of-order execution, which implies out-of-order completion**

To allow out-of-order execution:

- ID stage split into 2 stages:
  - Issue: Decode instruction, check for structural hazards.
  - Read Operands: Wait for data hazard to be resolved, read the operands
- IF unchanged.
- EX considered multicycle, Two phases:
  - Instruction begin execution
  - Instruction complete execution

### Complicazioni

**Out-of-order execution introduces the possibility of WAR and WAW hazards**, which do not exist in the classic five-stage pipeline

Consider the following RISC-V floating-point code sequence:

```
fdiv.d f0,f2,f4
fmul.d f6,f0,f8
fadd.d f0,f10,f14
```

There is an antidependence between the fmul.d and the fadd.d, and **if the pipeline executes the fadd.d before the fmul.d (which is waiting for the fdiv.d)**, it will violate the antidependence, yielding a WAR hazard.

Likewise, to avoid violating output dependences, such as the write of f0 by fadd.d before fdiv.d completes, WAW hazards must be handled.

As we will see, both these hazards are **avoided by the use of register renaming.**

**NB**: abbiamo spezzato un'assunto fatto fino ad ora

- era impossibile che un'istruzione che viene dopo modificasse gli operandi di un'istruzione precedente (alea WAR)
  - questo perchè gli operandi venivano letti in ID e scritti in WB
