**Date:** October 2, 2026

## 1. Trace Semantics Example

> [!example] Program Execution Trace
> Consider the following program and an initial memory state where $x$ is mapped to $2$:
> 
> **Initial State:** $m = [x \to 2]$
> 
> **Program Code:**
> ```c
> 1) i = 0;
> 2) while (i < x)
> 3)    i = i + 1;
> 4) 
> ```
> 
> **Execution Trace (Post-states):**
> Tracing the memory state evolution (where superscripts denote the trace step or program point):
> $$[x \to 2]^0 \rightarrow [x \to 2, i \to 0]^1 \rightarrow [x \to 2, i \to 1]^3 \rightarrow [x \to 2, i \to 2]^3 \rightarrow [x \to 2, i \to 2]^4$$

> [!info] Inferring Partial Traces
> We aim to infer the set of all possible partial execution traces reaching each specific program point:
> 
> - **At point 1:** 
>   $\{ [x \to 2]^0 \rightarrow [x \to 2, i \to 0]^1 \}$
> 
> - **At point 3:** 
>   $\{ [x \to 2]^0 \rightarrow [x \to 2, i \to 0]^1 \rightarrow [x \to 2, i \to 1]^3, \\ \quad [x \to 2]^0 \rightarrow [x \to 2, i \to 0]^1 \rightarrow [x \to 2, i \to 1]^3 \rightarrow [x \to 2, i \to 2]^3 \}$
> 
> - **At point 4:** 
>   $\{ [x \to 2]^0 \rightarrow [x \to 2, i \to 0]^1 \rightarrow [x \to 2, i \to 1]^3 \rightarrow [x \to 2, i \to 2]^3 \rightarrow [x \to 2, i \to 2]^4 \}$


## 2. State Abstraction / Collecting Semantics ($\alpha$)

We might want to obtain all possible values of $i$ and $x$ at each program point instead of keeping the entire trace history. This state-based abstraction is formally known as **Collecting Semantics**.

> [!abstract] Applying the Abstraction ($\alpha$)
> By applying an abstraction transformation ($\alpha$), we effectively take the union of the values present for each variable in the *final* state of all traces reaching that point.
> 
> | Program Point | $i$ values | $x$ values |
> | :---: | :---: | :---: |
> | **1** | $\{0\}$ | $\{2\}$ |
> | **3** | $\{1, 2\}$ | $\{2\}$ |
> | **4** | $\{2\}$ | $\{2\}$ |

> [!warning] Precision Loss
> By doing this abstraction, we return to the state-based approach seen in the previous lecture, **losing the relationality** (temporal relationship) between the variables across the execution traces.

> [!info] Trace Semantics vs. Collecting Semantics
> The terminology is correct:
> - **Trace Semantics** (often called Maximal Trace Semantics) keeps the exact temporal sequence of states.
> - **Collecting Semantics** is the abstraction we just applied ($\alpha$), which "collects" (takes the union of) all possible states reaching each specific program point, losing the temporal relationships.
> 


## 3. Formalizing Traces

Let $X = \mathbb{M} \times \mathbb{L}$ be the set of labeled states, where $\mathbb{M}$ is the memory state and $\mathbb{L}$ is the program label (the current line of code to execute). We define:
- $X^*$ : The set of all **finite traces** (sequences of labeled states, including the empty trace).
- $X^\omega$ : The set of all **infinite traces**.
- $X^{\infty} = X^* \cup X^\omega$ : The set of **all possible traces**.

> [!example] Semantic Transfer Function over Traces
> Let $I \subseteq \mathbb{M}$ be the set of initial memory states. We define a transfer function $F$ over the powerset of finite traces $\wp(X^*)$:
> $$F : \wp(X^*) \to \wp(X^*)$$
> 
> The trace semantics of the program is computed as the least fixpoint of $F$, starting from the empty set of traces (the BOTTOM of our powerset lattice):
> $$lfp^{\subseteq}_{\emptyset} F$$

## 4. Definition of the Transfer Function $F(T)$

Given a set of finite traces $T \in \wp((\mathbb{M} \times \mathbb{L})^*)$, the function $F(T)$ constructs the next set of traces by considering three components:

1. **Initial States**
   We start with traces of length 1 representing the initial configurations:
   $$ \{ i^0 \mid i \in I \} $$

2. **Not (yet) Ending Executions**
   We extend existing traces in $T$ by taking one execution step. We assume a function `next(l)` that returns the next statement to execute.
   $$ \left\{ i_0^{l_0} \to i_1^{l_1} \to \dots \to i_{n-1}^{l_{n-1}} \to i_n^{l_n} \ \middle| \ 
   \begin{aligned}
   & (i_0^{l_0} \to \dots \to i_{n-1}^{l_{n-1}}) \in T \\
   & \land \ \lbrack\!\lbrack \text{next}(l_{n-1}) \rbrack\!\rbrack \ i_{n-1} = i_n^{l_n}
   \end{aligned}
   \right\} $$

3. **Blocking Executions (Terminated Traces)**
   We carry over the traces in $T$ that have terminated and cannot be extended further (i.e., there are no more statements to execute):
   $$ \left\{ i_0^{l_0} \to i_1^{l_1} \to \dots \to i_n^{l_n} \ \middle| \ 
   \begin{aligned}
   & (i_0^{l_0} \to \dots \to i_n^{l_n}) \in T \\
   & \land \ \text{next}(l_n) = \cdot
   \end{aligned}
   \right\} $$

> [!note] The Terminal Label ($\cdot$)
> In this notation, $\cdot$ represents a special **terminal label** (or empty continuation). When $\text{next}(l_n)$ evaluates to $\cdot$, it formally means that the control flow has reached the end of the program and there are no further instructions to execute. Therefore, the trace becomes "blocking" (it cannot be extended with new states).

The complete function $F(T)$ is the **union** ($\cup$) of these three sets.


## 5. Fixpoint Computation Example

**Program Code:**
```c
1) i = 0;
2) while (i < x)
3)    i++;
4) skip;
```
**Initial States:** $I = \{ [x \to 2] \}$

> [!info] The `next` Function (Control Flow)
> In trace semantics, the function `next(l, m)` relies on the **Control Flow Graph (CFG)** of the program. It formally determines the next program label to execute based on the current label $l$ and the current memory state $m$ (the state is necessary to evaluate conditional branches like `while` loops).

For this specific execution, the control flow transitions evaluate as follows:
- `next(0, m)` returns line **1** (entry point).
- `next(1, m)` evaluates the loop condition `i < x` in state $[x \to 2, i \to 0]$. Since $0 < 2$, it returns line **3**.
- `next(3, m)` re-evaluates the loop condition `i < x`. When $i=1$, it returns line **3**; when $i=2$, it returns line **4** (False branch).
- `next(4, m) = \cdot` (program termination).

Now, let's compute the least fixpoint iteratively by applying $F$:

- **Iteration 0** (Start with BOTTOM):
  $$F^0 = \emptyset$$

- **Iteration 1** (Add initial states):
  $$F^1 = F(F^0) = \{ {\color{#e67e22} [x \to 2]^0} \}$$

- **Iteration 2** (`next(0)` leads to line 1):
  $$ \begin{aligned} F^2 = F(F^1) &= F^1 \cup \{ {\color{#e67e22} [x \to 2]^0 \to [x \to 2, i \to 0]^1} \} \\ &= \left\{ \begin{aligned} & [x \to 2]^0, \\ & {\color{#e67e22} [x \to 2]^0 \to [x \to 2, i \to 0]^1} \end{aligned} \right\} \end{aligned} $$

- **Iteration 3** (`next(1)` evaluates $0 < 2$, executes line 3):
  $$ \begin{aligned} F^3 = F(F^2) &= F^2 \cup \{ {\color{#e67e22} [x \to 2]^0 \to [x \to 2, i \to 0]^1 \to [x \to 2, i \to 1]^3} \} \\ &= \left\{ \begin{aligned} & [x \to 2]^0, \\ & [x \to 2]^0 \to [x \to 2, i \to 0]^1, \\ & {\color{#e67e22} [x \to 2]^0 \to [x \to 2, i \to 0]^1 \to [x \to 2, i \to 1]^3} \end{aligned} \right\} \end{aligned} $$

- **Iteration 4** (`next(3)` evaluates $1 < 2$, executes line 3):
  $$ \begin{aligned} F^4 = F(F^3) &= F^3 \cup \{ {\color{#e67e22} [x \to 2]^0 \to \dots \to [x \to 2, i \to 1]^3 \to [x \to 2, i \to 2]^3} \} \\ &= \left\{ \begin{aligned} & [x \to 2]^0, \\ & [x \to 2]^0 \to [x \to 2, i \to 0]^1, \\ & [x \to 2]^0 \to [x \to 2, i \to 0]^1 \to [x \to 2, i \to 1]^3, \\ & {\color{#e67e22} [x \to 2]^0 \to [x \to 2, i \to 0]^1 \to [x \to 2, i \to 1]^3 \to [x \to 2, i \to 2]^3} \end{aligned} \right\} \end{aligned} $$

- **Iteration 5** (`next(3)` evaluates $2 < 2$ (False), executes line 4):
  $$ \begin{aligned} F^5 = F(F^4) &= F^4 \cup \{ {\color{#e67e22} [x \to 2]^0 \to \dots \to [x \to 2, i \to 1]^3 \to [x \to 2, i \to 2]^3 \to [x \to 2, i \to 2]^4} \} \\ &= \left\{ \begin{aligned} & [x \to 2]^0, \\ & [x \to 2]^0 \to [x \to 2, i \to 0]^1, \\ & [x \to 2]^0 \to [x \to 2, i \to 0]^1 \to [x \to 2, i \to 1]^3, \\ & [x \to 2]^0 \to [x \to 2, i \to 0]^1 \to [x \to 2, i \to 1]^3 \to [x \to 2, i \to 2]^3, \\ & {\color{#e67e22} [x \to 2]^0 \to [x \to 2, i \to 0]^1 \to [x \to 2, i \to 1]^3 \to [x \to 2, i \to 2]^3 \to [x \to 2, i \to 2]^4} \end{aligned} \right\} \end{aligned} $$

- **Iteration 6** (Trace is blocking at point 4, fixpoint reached!):
  $$F^6 = F(F^5) = F^5$$

The least fixpoint $lfp^{\subseteq}_{\emptyset} F$ is exactly $F^5$, which contains the set of all partial and maximal execution traces of the program.