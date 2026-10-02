**Date:** October 1, 2026

## 1. Syntax of IMP (Imperative Language)

The syntax of **IMP**, a simple imperative programming language, can be formally defined using BNF (Backus-Naur Form):

> [!info] IMP Grammar
> We define the domains $E$ (Arithmetic Expressions), $B$ (Boolean Expressions), and $S$ (Statements).
> 
> **Arithmetic Expressions ($e \in E$):**
> $$e ::= X \mid n \mid e_1 \ op_a \ e_2$$
> Where:
> - $n \in \mathbb{Z}$ (integers)
> - $op_a \in \{+, -, *, /\}$ (arithmetic operators)
> 
> **Boolean Expressions ($b \in B$):**
> $$b ::= \text{true} \mid \text{false} \mid \neg b \mid b_1 \ op_b \ b_2 \mid e_1 \ op_c \ e_2$$
> Where:
> - $op_b \in \{||, \&\&\}$ (logical operators)
> - $op_c \in \{==, !=, <, >\}$ (relational operators)
> 
> **Statements ($S \in \text{Stmt}$):**
> $$S ::= X = e; \mid \text{skip} \mid S_1 \ S_2 \mid \text{if } b \text{ then } S_1 \text{ else } S_2 \mid \text{while } b \text{ do } S$$

---

## 2. Operational Semantics

When defining how a program executes, we generally use two approaches:

> [!example] Big-Step vs Small-Step Semantics
> Given an initial memory state where $x \to 2$ and $y \to 4$, let's evaluate the expression $(x+y)+5$:
> 
> **Big-Step Semantics:**
> Evaluates the entire expression to its final result in a single large transition.
> - $(x+y)+5 \implies 11$
> 
> **Small-Step Semantics:**
> Evaluates the expression one atomic computation at a time, showing the intermediate states.
> - $(x+y)+5 \rightarrow (2+y)+5 \rightarrow (2+4)+5 \rightarrow 6+5 \rightarrow 11$

---

## 3. Hoare Logic

> [!info] Hoare Triples
> To formally reason about program correctness, we use **Hoare Logic**, which is based on Hoare Triples:
> $$\{P\} \ C \ \{Q\}$$
> Where:
> - **$P$ (Precondition):** An assertion about the memory state before the command $C$ executes.
> - **$C$ (Command/Program):** The code being executed.
> - **$Q$ (Postcondition):** An assertion about the memory state after $C$ finishes executing.
> 
> **Meaning:** "If the program starts in a state where $P$ is true, and executes $C$, then if $C$ terminates, the resulting state will satisfy $Q$."
> 
> **Partial vs Total Correctness:**
> - **Partial Correctness:** Guarantees that *if* the program terminates, $Q$ holds. It does not guarantee termination (e.g., an infinite loop still satisfies partial correctness if $P$ is true).
> - **Total Correctness:** Guarantees that the program *will* terminate AND $Q$ holds.
> 
> **Example:**
> $$\{x > 0\} \ \ x = x + 1 \ \ \{x > 1\}$$
> This reads: If $x$ is strictly positive before execution, after running $x = x + 1$, $x$ will be strictly greater than $1$.
>

## 4. Memory States

> [!info] Memory State ($\mathbb{M}$)
> A memory state (denoted as $\mathbb{M}$) is defined as a function mapping variables ($X$) to integer values ($\mathbb{Z}$):
> $$\mathbb{M} : X \to \mathbb{Z}$$
> 
> A specific memory instance is usually denoted by the lowercase letter $m$. 
> 
> **Example of a memory state:**
> $m = [x \to 5, y \to 9, z \to 11]$

## 5. Expression Evaluator

> [!info] Expression Evaluator Function
> The evaluation of arithmetic expressions is defined by a semantic function $\lbrack\!\lbrack \cdot \rbrack\!\rbrack_e$ that takes an expression and a memory state, and returns an integer:
> $$\lbrack\!\lbrack \cdot \rbrack\!\rbrack_e : E \times \mathbb{M} \to \mathbb{Z}$$
> 
> *(Note: This function is inherently **partial**. Evaluation can fail if a variable $x$ is not defined in the memory $m$, or if an arithmetic operation is undefined, such as division by zero).*
> 
> **Arithmetic Semantic Rules:**
> 
> $$
> \begin{aligned}
> \lbrack\!\lbrack x \rbrack\!\rbrack_e \ m &= m(x) \\
> \lbrack\!\lbrack n \rbrack\!\rbrack_e \ m &= n \\
> \lbrack\!\lbrack e_1 \ op_a \ e_2 \rbrack\!\rbrack_e \ m &= \lbrack\!\lbrack e_1 \rbrack\!\rbrack_e \ m \ op_a \ \lbrack\!\lbrack e_2 \rbrack\!\rbrack_e \ m
> \end{aligned}
> $$
> 

## 6. Boolean Evaluator

> [!info] Boolean Evaluator Function
> Similarly, the evaluation of boolean expressions is defined by a semantic function $\lbrack\!\lbrack \cdot \rbrack\!\rbrack_b$:
> $$\lbrack\!\lbrack \cdot \rbrack\!\rbrack_b : B \times \mathbb{M} \to \{\text{tt}, \text{ff}\}$$
> 
> **Boolean Semantic Rules:**
> 
> $$
> \begin{aligned}
> \lbrack\!\lbrack \text{true} \rbrack\!\rbrack_b \ m &= \text{tt} \\
> \lbrack\!\lbrack \text{false} \rbrack\!\rbrack_b \ m &= \text{ff} \\
> \lbrack\!\lbrack \neg b \rbrack\!\rbrack_b \ m &= \neg \lbrack\!\lbrack b \rbrack\!\rbrack_b \ m \\
> \lbrack\!\lbrack b_1 \ op_b \ b_2 \rbrack\!\rbrack_b \ m &= \lbrack\!\lbrack b_1 \rbrack\!\rbrack_b \ m \ op_b \ \lbrack\!\lbrack b_2 \rbrack\!\rbrack_b \ m \\
> \lbrack\!\lbrack e_1 \ op_c \ e_2 \rbrack\!\rbrack_b \ m &= \lbrack\!\lbrack e_1 \rbrack\!\rbrack_e \ m \ op_c \ \lbrack\!\lbrack e_2 \rbrack\!\rbrack_e \ m
> \end{aligned}
> $$
> *(Note on relational operators $op_c$: The outer evaluator is $b$ because the final result is a boolean, but it internally calls the arithmetic evaluator $e$ to resolve the sub-expressions $e_1$ and $e_2$.)*
## 7. Statement Evaluator

> [!info] Statement Evaluator Function
> The evaluation of statements (commands) modifies the memory state. It is defined by the semantic function $\lbrack\!\lbrack \cdot \rbrack\!\rbrack_S$ that takes a statement and a memory state, and returns an updated memory state:
> $$\lbrack\!\lbrack \cdot \rbrack\!\rbrack_S : S \times \mathbb{M} \to \mathbb{M}$$
> 
> **Statement Semantic Rules:**
> 
> $$
> \begin{aligned}
> \lbrack\!\lbrack x = e; \rbrack\!\rbrack_S \ m &= m[x \gets \lbrack\!\lbrack e \rbrack\!\rbrack_e \ m] \\
> \lbrack\!\lbrack \text{skip} \rbrack\!\rbrack_S \ m &= m \\
> \lbrack\!\lbrack S_1 \ S_2 \rbrack\!\rbrack_S \ m &= \lbrack\!\lbrack S_2 \rbrack\!\rbrack_S \ (\lbrack\!\lbrack S_1 \rbrack\!\rbrack_S \ m) \\
> \lbrack\!\lbrack \text{if } b \text{ then } S_1 \text{ else } S_2 \rbrack\!\rbrack_S \ m &= \begin{cases} 
> \lbrack\!\lbrack S_1 \rbrack\!\rbrack_S \ m & \text{if } \lbrack\!\lbrack b \rbrack\!\rbrack_b \ m = \text{tt} \\ 
> \lbrack\!\lbrack S_2 \rbrack\!\rbrack_S \ m & \text{if } \lbrack\!\lbrack b \rbrack\!\rbrack_b \ m = \text{ff} 
> \end{cases} \\
> \lbrack\!\lbrack \text{while } b \text{ do } S \rbrack\!\rbrack_S \ m &= \begin{cases} 
> \lbrack\!\lbrack S \ (\text{while } b \text{ do } S) \rbrack\!\rbrack_S \ m & \text{if } \lbrack\!\lbrack b \rbrack\!\rbrack_b \ m = \text{tt} \\ 
> m & \text{if } \lbrack\!\lbrack b \rbrack\!\rbrack_b \ m = \text{ff} 
> \end{cases}
> \end{aligned}
> $$


## 8. Trace Semantics (Semantica delle tracce)

> [!warning] Spurious Traces (Example from Lecture 4)
> Consider the `while` loop example from our previous lecture:
> ```c
> 1  i = 0;
> 2  while (i < 2)
> 3      i++;
> 4  ...
> ```
> If our static analysis only records the **set of possible values** at each program line (e.g., at line 2, $i \in \{0, 1, 2\}$), we lose the temporal relationship between states. 
> By just looking at these independent sets, one might incorrectly stitch together a path—for example, concluding that from line 1 (where $i=0$) we could jump to line 2 with $i=3$. These impossible execution sequences generated by abstraction precision loss are called **spurious traces**.

To avoid spurious traces, we model program execution natively as **execution traces** rather than independent sets of states.

> [!info] Execution Traces
> An execution trace is a sequence of memory states across discrete time steps.
> 
> **Visualizing Execution Traces:**
> 
> The execution of a program can be viewed as a set of traces evolving over time. In the diagram below:
> - <span style='color:#43a047'>●</span> **Initial states**: Every trace starts at discrete time $0$ from a specific initial memory configuration.
> - <span style='color:#e53935'>●</span> **Intermediate states**: The trace progresses as statements are executed, updating the memory state at each time step.
> - <span style='color:#fbc02d'>◉</span> **Final states of finite traces**: When a trace reaches a point where the program terminates natively (e.g. `skip` or end of code), it becomes a *blocking execution* and stops.
> - <span style='color:#5c6bc0'>┈┈┈</span> **Infinite traces**: Traces that never terminate (e.g., due to an infinite `while` loop) continue generating new states indefinitely.
> 
> ```mermaid
> %%{init: {"flowchart": {"rankSpacing": 15, "nodeSpacing": 8, "curve": "linear", "padding": 2, "wrappingWidth": 600}}}%%
> flowchart TB
>   subgraph TR[" "]
>     direction LR
>     %% hidden anchor: every trace starts at time 0
>     h[" "] ~~~ a0 & b0 & c0 & d0 & e0 & t0
> 
>     %% finite (terminating) traces
>     a0@{shape: f-circ} --- a1@{shape: f-circ} --- a2@{shape: f-circ} --- a3@{shape: f-circ} --- a4@{shape: fr-circ}
>     b0@{shape: f-circ} --- b1@{shape: f-circ} --- b2@{shape: f-circ} --- b3@{shape: f-circ} --- b4@{shape: f-circ} --- b5@{shape: f-circ} --- b6@{shape: fr-circ}
>     c0@{shape: f-circ} --- c1@{shape: f-circ} --- c2@{shape: fr-circ}
> 
>     %% infinite traces
>     d0@{shape: f-circ} --- d1@{shape: f-circ} --- d2@{shape: f-circ} --- d3@{shape: f-circ} --- d4@{shape: f-circ} --- d5@{shape: f-circ} --- d6@{shape: f-circ} --- d7@{shape: f-circ} -.- dI@{shape: f-circ}
>     e0@{shape: f-circ} --- e1@{shape: f-circ} --- e2@{shape: f-circ} --- e3@{shape: f-circ} --- e4@{shape: f-circ} --- e5@{shape: f-circ} --- e6@{shape: f-circ} --- e7@{shape: f-circ} -.- eI@{shape: f-circ}
> 
>     %% discrete time
>     t0["0"] ~~~ t1["1"] ~~~ t2["2"] ~~~ t3["3"] ~~~ t4["4"] ~~~ t5["5"] ~~~ t6["6"] ~~~ t7["7"] ~~~ tT["time"]
>   end
> 
>   classDef ini fill:#43a047,stroke:#43a047
>   classDef mid fill:#e53935,stroke:#e53935
>   classDef fin fill:#fbc02d,stroke:#5c6bc0,stroke-width:2px
>   classDef txt fill:none,stroke:none,color:#5c6bc0
>   classDef hid fill:none,stroke:none
>   class a0,b0,c0,d0,e0 ini
>   class a1,a2,a3,b1,b2,b3,b4,b5,c1,d1,d2,d3,d4,d5,d6,d7,e1,e2,e3,e4,e5,e6,e7 mid
>   class a4,b6,c2 fin
>   class t0,t1,t2,t3,t4,t5,t6,t7,tT txt
>   class h,dI,eI hid
>   style TR fill:none,stroke:none
>   linkStyle 6,7,8,9,10,11,12,13,14,15,16,17,18,19,20,21,22,23,24,25,26,27,28,29,30,31,32,33 stroke:#5c6bc0,stroke-width:2px
> ```

### 8.1 Lattice Structure for Trace Semantics

> [!info] Trace Semantics as a Lattice
> To compute the trace semantics, we need a complete lattice structure. 
> 
> - A single execution (trace) of a program is denoted as $\tau \in X^{\infty}$, where $X^{\infty}$ represents the set of all possible execution traces.
> - We can model the concrete semantics as the powerset lattice over all traces: 
>   $$\langle \wp(X^{\infty}), \subseteq, \cup, \cap \rangle$$
>   *(This forms a complete lattice where the subset relation acts as the partial order).*
> 
> **Intuition for defining the semantics:**
> 1. Compute a **single-step** of the computation, obtaining a set of partial executions.
> 2. **Iterate** over these partial executions to compute the subsequent steps.
> 3. Continue this iteration until a **fixpoint** is reached (meaning the set of traces stabilizes and no new states are discovered).

> **Example of Trace Iteration:**
> Let's look at how a single execution trace builds up over time (where the superscript denotes the discrete time step):
> - **Step 0:** $[i \to 0]^0$
> - **Step 1:** $[i \to 0]^0 \rightarrow [i \to 0]^1$
> - **Step 2:** $[i \to 0]^0 \rightarrow [i \to 0]^1 \rightarrow [i \to 0]^2$
> - **Step 3:** $[i \to 0]^0 \rightarrow [i \to 0]^1 \rightarrow [i \to 0]^2 \rightarrow [i \to 1]^3$

### 8.2 Computing Partial Executions

> [!info] Final Partial Execution
> To compute the semantics iteratively, we build the traces step-by-step:
> - **First step:** 
>   - Compute executions of length 1.
>   - Take the union with the traces containing only the initial states (length 0).
> - **$i$-th step:**
>   - Compute executions of length $i$.
>   - Take the union with the traces of length at most $i-1$.
> - **Termination:** 
>   - Repeat until a **fixpoint** is reached (the set of traces stops growing).

### 8.3 Least Fixpoint Computation

> [!example] Formalizing the Computation
> Given a program $P$ and its semantic transfer function $F_P$, we compute the fixpoint semantics starting from *below* (BOTTOM). In our powerset lattice $\langle \wp(X^{\infty}), \subseteq \rangle$, the BOTTOM element is the empty set ($\emptyset$).
> 
> $$lfp^{\subseteq}_{\emptyset} F_P$$
> 
> By repeatedly applying $F_P$ starting from $\emptyset$, we iteratively build up the set of all valid partial executions of the program.

