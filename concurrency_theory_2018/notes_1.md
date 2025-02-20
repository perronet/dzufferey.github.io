# Finite state machines

## Definitions


#### DFA Example

Words finishing with 1 over the alphabet $\\{0,1\\}$.

```graphviz
digraph finite_state_machine {
	rankdir=LR;
	node [shape = circle];
	init [shape = none, label = ""];
	B [shape = doublecircle];
    init -> A;
	A -> B [ label = "1" ];
	B -> A [ label = "0" ];
	A -> A [ label = "0" ];
	B -> B [ label = "1" ];
}
```

_Notation._
* States with double circle are accepting state.
* The state with an arrow coming from nowhere is the initial state.
* In the text, we write state as `(.)`, accepting state as `((.))`, and transition as `→`.

#### DFA

A _deterministic finite automaton_ (DFA) $M$ is a 5-tuple $(Q, Σ, δ, q_0, F)$ where

* $Q$ is a finite set of states
* $Σ$ is a finite alphabet (set of input symbols)
* $δ$ is the transition function ($Q × Σ → Q$)
* $q_0$ is the initial state ($q_0 ∈ Q$)
* $F$ is the set of accepting states ($F ⊆ Q$)

Let $w = a_1 a_2 … a_n$ be a word over the alphabet $Σ$.
The automaton $M$ accepts $w$ if there is a sequence of states, $r_0 r_1 … r_n$, such that:
* $r_0 = q_0$
* $r_{i+1} = δ(r_i, a_{i+1})$ for $i = 0, …, n−1$
* $r_n ∈ F$


#### NFA Example

Word with 1 as the 3rd symbol before the end

```graphviz
digraph finite_state_machine {
	rankdir=LR;
	node [shape = circle];
	init [shape = none, label = ""];
	D [shape = doublecircle];
    init -> A;
	A -> B [ label = "1" ];
	B -> C [ label = "0,1" ];
	C -> D [ label = "0,1" ];
	A -> A [ label = "0,1" ];
}
```

#### NFA

A _non-deterministic finite automaton_ (NFA) $M$ is a 5-tuple $(Q, Σ, δ, q_0, F)$ where

* $Q$ is a finite set of states
* $Σ$ is a finite alphabet (set of input symbols)
* $δ$ is the transition function ($Q × Σ → 2^Q$)
* $q_0$ is the initial state ($q_0 ∈ Q$)
* $F$ is the set of accepting states ($F ⊆ Q$)

Let $w = a_1 a_2 … a_n$ an be a word over the alphabet $Σ$.
The automaton $M$ accepts $w$ if there is a sequence of states, $r_0 r_1 … r_n$, such that:
* $r_0 = q_0$
* $r_{i+1} ∈ δ(r_i, a_{i+1})$ for $i = 0, …, n−1$
* $r_n ∈ F$

#### Language

The _language_ of an automaton $M$, denoted $L(M)$, is the set of words accepted by $M$.

#### Trace

A _trace_ of an automaton $M$ is an alternating sequence $r_0 a_1 r_1 a_2 … a_n r_n$ such that
* $r_0 = q_0$
* $r_{i+1} ∈ δ(r_i, a_{i+1})$ for $i = 0, …, n−1$

A trace is _accepting_ if $r_n ∈ F$.


## Operation (Closure properties)

* intersection/union
* complementation
* emptiness/universality

_Remark._
The intersection and union are defined for automata with the same alphabet.
Complementation preserves the alphabet.

_Remark._
Intersection and union are computed with a product construction.
Complementation is easy for DFA but hard for NFA.
Emptiness and universality are solved by minimizing the automaton.


### Synchronized Product

Given automaton $M_1$ and $M_2$, the synchronized product $M_1 ⊗ M_2$ is the automaton $M$ where:

* $Q = Q_1 × Q_2$
* $Σ = Σ_1 ∪ Σ_2$
* $δ$ is the transition function
  - $δ((q_1,q_2), a) = (δ_1(q_1, a), δ_2(q_2, a))$ if $a ∈ Σ_1$ and $a ∈ Σ_2$
  - $δ((q_1,q_2), a) = (q_1, δ_2(q_2, a))$ if $a ∉ Σ_1$ and $a ∈ Σ_2$
  - $δ((q_1,q_2), a) = (δ_1(q_1, a), q_2))$ if $a ∈ Σ_1$ and $a ∉ Σ_2$
* $q₀ = (q₀_1,q₀_2)$
* $F = F_1 × F_2$


#### Example

Here the accepting states describe executions allowed by the program/lock.

**NFA representing a lock**

```graphviz
digraph finite_state_machine {
	rankdir=LR;
	node [shape = doublecircle];
	init [shape = none, label = ""];
    init -> U;
	U -> L [ label = "lock" ];
	L -> U [ label = "unlock" ];
}
```

**Program using a lock**
```
int balance;

void increase(int x) {
    lock();
    balance += x;
    unlock();
}
```

**Control-flow automaton (CFA)**

```graphviz
digraph finite_state_machine {
	rankdir=LR;
	node [shape = doublecircle];
	init [shape = none, label = ""];
    init -> 0;
	0 -> 1 [ label = "lock" ];
	1 -> 2 [ label = "balance += x" ];
	2 -> 3 [ label = "unlock" ];
}
```

A CFA is an automaton which states are control locations of the program and the edges are statments of the program.
It is called CFA because it only captures the part of the state related to the control flow and ignores the values of the variables.


**Synchronized Product**

```graphviz
digraph finite_state_machine {
	rankdir=LR;
	node [shape = doublecircle];
    0 [label = "0, U"];
    1 [label = "1, L"];
    2 [label = "2, L"];
    3 [label = "3, U"];
	init [shape = none, label = ""];
    init -> 0;
	0 -> 1 [ label = "lock" ];
	1 -> 2 [ label = "balance += x" ];
	2 -> 3 [ label = "unlock" ];
}
```

This is the _lazy_ construction that only constructs state if needed.
The full construction would generate extra states: `((0, L))`, `((1, U))`, `((2, U))`, `((3, L))`.



### Determinization (Powerset construction)

_Assuming no ε-transitions_

Given a NFA $N$ we construct a DFA $D$ with:
* $Q_D = 2_{Q_N}$
* $Σ_D = Σ_N$
* $δ_D(q_D, a) = \bigcup_{q∈q_D} δ_N(q, a) = \\{ q' ~|~ ∃ q ∈ q_D. q' ∈ δ_N(q, a) \\}$
* $q_0^D = \\{ q_0^N \\} $
* $F_D = \\{ q ~|~ q ∩ F_N ≠ ∅ \\}$

_Notation._
We use $N$, $D$ as sub- or superscript to identify wether the element belongs to $N$ or $D$.

__Theorem.__ $L(N) = L(D)$

_Proof._

In two steps:
1. $w ∈ L(N) ⇒ w ∈ L(D)$
2. $w ∈ L(D) ⇒ w ∈ L(N)$

__(1)__ $w ∈ L(N) ⇒ w ∈ L(D)$
* Let $t_N$ be an accepting trace of $N$ for $w$.
  $t_N$ exists by definition of $L(N)$.
* Let $t_D$ be the trace of $w$ on $D$.
  We show that $t_D$ "contains" $t_N$ by induction on the traces:
  - $0$:
    + $q_0^N ∈ \\{ q_0^N \\} = q_0^D$ by definition,
  - $i$ → $i+1$:
    + by induction hypothesis: $r_i^N ∈ r_i^D$
    + by definition of $t_N$ and $δ_D$: $r_{i+1}^N ∈ δ_N(r_i^N, a_i)$ and, therefore, $r_{i+1}^n ∈ r_{i+1}^D$.
* By hypothesis, the last state of $t_N$ is $q_n^N$ with $q_n^N ∈ F_N$.
* By the above, we have $q_n^N ∈ q_n^D$.
* Therefore, $q_n^D ∩ F_N ≠ ∅$.
* Thus, $t_D$ is accepting.

__(2)__ $w ∈ L(D) ⇒ w ∈ L(N)$
* As homework …

##### Example of Determinization

Let the following NFA where it is possible to more `s`traight or `d`iagonally:

```graphviz
digraph finite_state_machine {
	rankdir=LR;
	node [shape = circle];
    edge [dir=both,arrowhead=normal,arrowtail=normal];
    subgraph top {
	    init [shape = none, label = ""];
        init -> 1 [arrowtail=none];
	    1 -> 2 [ label = "s" ];
        2 -> 3 [ label = "s" ];
    }
    subgraph bottom {
	    dummy [shape = none, label = ""];
        6 [shape = doublecircle];
        dummy -> 4 [ style=invis];
	    4 -> 5 [ label = "s" ];
	    5 -> 6 [ label = "s" ];
    }
    edge [ constraint=false ];
	1 -> 4 [ label = "s" ];
	1 -> 5 [ label = "d" ];
	2 -> 5 [ label = "s" ];
	2 -> 4 [ label = "d" ];
	2 -> 6 [ label = "d" ];
	3 -> 6 [ label = "s" ];
	3 -> 5 [ label = "d" ];
}
```

Determinizing it gives:
```graphviz
digraph finite_state_machine {
	rankdir=LR;
	node [shape = circle, width=0.75];
    subgraph top {
	    init [shape = none, label = ""];
        init -> 1;
        1 [label = "{1}"];
        24 [label = "{2,4}"];
        135 [label = "{1,3,5}"];
	    1 -> 24 [ label = "s" ];
        24 -> 135 [ label = "s" ];
        135 -> 135 [ label = "d" ];
    }
    subgraph bottom {
	    dummy [shape = none, label = ""];
        5 [label = "{5}"];
        13 [label = "{1,3}"];
        246 [shape = doublecircle, label = "{2,4,6}"];
        dummy -> 5 [ style=invis];
	    5 -> 13 [ label = "d", dir=both ];
	    13 -> 246 [ label = "s" ];
	    5 -> 246 [ label = "s" ];
	    246:s -> 246 [ label = "d" ];
    }
    edge [ constraint=false ];
	1 -> 5 [ label = "d" ];
    24 -> 246 [ label = "d" ];
	135 -> 246 [ label = "s", dir = both ];
}
```

## Verifying Properties

### Properties

Properties of concurrent systems are broadly divided in two categories:
* _Safety_: properties that are violated by finite traces
* _Liveness_: properties that can only be violated by infinite traces

In this class we focus on _safety_ properties and a few _eventuality_ properties.
Eventuality properties are simple liveness properties that says something eventually happens, e.g., a program eventually terminates.
General classes of temporal properties (LTL, CTL, μ-calculus, weak/strong fairness, …) are out of scope.

#### Example

* Assertion is a safety property.
* Termination is a liveness property.
* Termination within 15 steps is a safety property.
* Deadlock-freedom is a safety property.
* Livelock-freedom is a liveness property.


## Verification

As Paths in graphs:
- Safety is reachability: path from the initial state to an error state.
- Eventuality is nested reachability: lasso path with the stem starting at the initial state and the loop does not go to any "progress" state where the progress state are the states that should eventually happen.

Verification can be done as automata construction.
Let us assume we have a program $A$ and a safety property $B$ represented as automata with the same alphabet $Σ$.
The program satisfy the property is a language inclusion check: $L(A) ⊆ L(B)$ which reduces to $L(A) ∩ (Σ^*∖L(B)) = L(A) ∩ L(¬B) = ∅$.

Instead doing the check $L(A) ∩ L(¬B) = ∅$ on the language (potentially infinite sets), we can to it on the automata as $A ⊗ ¬B = ∅$.
This formulation also work when $A$ and $B$ have different alphabets.


#### Example

Using the lock+program above, we can check that the program uses the lock correctly.

First, we complement the lock:
```graphviz
digraph finite_state_machine {
	rankdir=LR;
	node [shape = circle]; U; L;
	node [shape = doublecircle]; Err;
	init [shape = none, label = ""];
    init -> U;
	U -> L [ label = "lock" ];
	L -> U [ label = "unlock" ];
    L -> Err [ label = "lock" ];
    U -> Err [ label = "unlock" ];
    Err -> Err [ label = "lock, unlock" ];
}
```
We can also add the error state to the automaton representing the program and, then, take the product.

The product is show below.
The colors represent the following:
* In blue, we show the "expected" reachable states.
* In black, we show the remaining reachable states.
* In grey, we show the unreachable states.
Notice that, no accepting state is reachable and, therefore, the program is safe.

```graphviz
digraph finite_state_machine {
    rankdir=LR;
    node [shape = circle, color = blue, fontcolor = blue]; a0; a1; a2; a3;
    node [shape = circle, color = black, fontcolor = black]; e4; e5; e6;
    node [shape = doublecircle, color = grey, fontcolor = grey]; e0; e1; e2; e3;
    node [shape = circle, color = grey, labelfontcolor = grey];
    a0 [label = "0, U"];
    a1 [label = "1, L"];
    a2 [label = "2, L"];
    a3 [label = "3, U"];
    b0 [label = "0, L"];
    b1 [label = "1, U"];
    b2 [label = "2, U"];
    b3 [label = "3, L"];
    e0 [label = "0, Err"];
    e1 [label = "1, Err"];
    e2 [label = "2, Err"];
    e3 [label = "3, Err"];
    e4 [label = "Err, U"];
    e5 [label = "Err, L"];
    e6 [label = "Err, Err"];
    edge [color = blue, fontcolor = blue];
    init [shape = none, label = ""];
    init -> a0;
    a0 -> a1 [ label = "lock" ];
    a1 -> a2 [ label = "balance += x" ];
    a2 -> a3 [ label = "unlock" ];
    edge [color = black, fontcolor = black];
    a0 -> e4 [ label = "balance += x" ];
    a0 -> e6 [ label = "unlock" ];
    a1 -> e6 [ label = "lock" ];
    a1 -> e4 [ label = "unlock" ];
    a2 -> e6 [ label = "lock" ];
    a2 -> e5 [ label = "balance += x" ];
    a3 -> e5 [ label = "lock" ];
    a3 -> e4 [ label = "balance += x" ];
    a3 -> e6 [ label = "unlock" ];
    
    e4 -> e5 [ label = "lock" ];
    e4 -> e4 [ label = "balance += x" ];
    e4 -> e6 [ label = "unlock" ];
    e5 -> e6 [ label = "lock" ];
    e5 -> e5 [ label = "balance += x" ];
    e5 -> e4 [ label = "unlock" ];
    e6 -> e6 [ label = "*" ];

    edge [color = grey, fontcolor = grey];
    b0 -> e1 [ label = "lock" ];
    b0 -> e5 [ label = "balance += x" ];
    b0 -> e6 [ label = "unlock" ];
    b1 -> e4 [ label = "lock" ];
    b1 -> e2 [ label = "balance += x" ];
    b1 -> e6 [ label = "unlock" ];
    b2 -> e5 [ label = "lock" ];
    b2 -> e4 [ label = "balance += x" ];
    b2 -> e3 [ label = "unlock" ];
    b3 -> e6 [ label = "lock" ];
    b3 -> e5 [ label = "balance += x" ];
    b3 -> e4 [ label = "unlock" ];
    
    e0 -> e1 [ label = "lock" ];
    e0 -> e6 [ label = "balance += x, unlock" ];
    e1 -> e6 [ label = "lock, unlock" ];
    e1 -> e2 [ label = "balance += x" ];
    e2 -> e6 [ label = "lock, balance += x" ];
    e2 -> e3 [ label = "unlock" ];
    e3 -> e6 [ label = "*" ];
}
```


### State-space Exploration (Model Checking)

Checking safety properties reduces to an emptiness check, we need to find an accepting path.
Since we negate the property to check, an accepting path is a counterexample!

Basic algorithm for checking safety properties:

```
F = {q₀}    // frontier
V = {}      // visited
while F ≠ ∅  do
    choose s in F
    F ← F ∖ {s}
    if s ∉ V then
        V ← V ∪ {s}
        if ¬safe(s)
            return UNSAFE
        else
            F ← F ∪ δ(s,_)
return SAFE
```
The algorithm above has a complexity of $O(|Q|⋅|Σ|)$.

Variations:
* using a queue for F makes a BFS
* using a stack for F makes a DFS
* this is a forward search, it is also possible do a backward search (start from the error state and computes the predecessors)


#### Encoding data as automaton

We can encode _bounded_ datatypes as finite automaton:
* boolean value `b`
    ```graphviz
    digraph finite_state_machine {
        rankdir=LR;
        node [shape = doublecircle];
        init [shape = none, label = ""];
        init -> false;
        false -> false [ label = "b = false" ];
        false -> true [ label = "b = true" ];
        true -> false [ label = "b = false" ];
        true -> true [ label = "b = true" ];
    }
    ```
* integer `i`
    ```graphviz
    digraph finite_state_machine {
        rankdir=LR;
        node [shape = doublecircle]; 0; 1;
        m1 [ label = "-1"];
        node [shape = none, label = ""]; init; dummy0; dummy1;
        mdots [shape = none, label = "..."];
        dots [shape = none, label = "..."];
        mdots -> m1 [style=invis];
        m1 -> 0 [label = "i += 1"];
        0 -> 1 [label = "i += 1"];
        1 -> dots  [style=invis];
        dummy0 -> dummy1  [style=invis];
        dummy1 -> init  [style=invis];
        edge [constraint = false];
        init -> 0;
        0 -> m1 [label = "i -= 1"];
        1 -> 0 [label = "i -= 1"];
    }
    ```

However, this is very expensive.
Numbers are exponentially more succinct than automaton.


### The Spin Model-Checker

You can download spin at: http://spinroot.com/

To run spin, we recommend using the following script: https://github.com/tcruys/runspin

#### Promela

You can find more explanation in the [manual](http://spinroot.com/spin/Man/Manual.html) or even [Wikipedia](https://en.wikipedia.org/wiki/Promela).

The input language for Spin is _Promela_ (Process/Protocol Meta Language).

_Primitive Types:_
* `bool`
* `byte` (unsigned: 0 to 255)
* `short` (16-bits signed integer)
* `int` (32-bits signed integer)

It is possible to have arrays.
`bool ready[2] = false;` is an array of size 2 containing boolean values that are initialized to false.

_Statements:_
* assignments: `x = 1;`
* test: `x == 1;` (also `!=`, `<`, `>`, `<=`, `>=`)
* `(condition -> valueTrue : valueFalse)`
* if-blocks
    ```
    if
    :: condition_1 -> body_1
    :: condition_2 -> body_2
    ...
    :: condition_n -> body_n
    :: else -> ...
    fi;
    ```
    If multiple alternatives are enabled, spin will explore all of them.
    `else` is optional.
    `else` is enabled only if no other condition is enabled.
* do-loops
    ```
    do
    :: condition_1 -> body_1
    :: condition_2 -> body_2
    ...
    :: condition_n -> body_n
    :: else -> ...
    od;
    ```
    Like `if` in a `while(true)`.
    The `break` statement is the only way to exit a loop.
* assertions: `assert(condition);`
* atomic blocs:
    ```
    atomic {
        ...
    }
    ```
    executes all the statement in the block as a single step (no interleaving).
* `printf` as in C.
* `skip` doesn't do anything.

`;` and `->` are separators.
In particular, `;` is not needed after the last statement.

Tests are normal statement that executes only when they are true and block otherwise.
`x != 0` simulates `while (x == 0) {}`.

Each statement executes atomically;

_Processes:_
* processes
  ```
  proctype name(args){
      ...
  }
  ```
  Processes are started with `run name(values)`.

  Alternatively it is possible to mark processes as active:
  ```
  active [2] proctype client(){
      ...
  }
  ```
  `[2]` is an optional parameter that starts 2 processes.

  The `_pid` variable is a special integer variable which as unique for each process.
* `init { ... }` block is like the `main` in C but without arguments.
  If there are active processes, `init` is not needed.
  
* if there are multiple active processes, Spin builds a product automaton and traverses it using a specified strategy (BFS or DFS)

##### Examples

You can find examples of spin programs [here](https://github.com/dzufferey/dzufferey.github.io/tree/master/concurrency_theory_2017/spin_examples).

Here are the particularity of the examples
* `ex01.pml`: simple while loop. Notice that the number of states explored by Spin is propotional to the sizes of the data.
* `ex02.pml`: same as 1 but without a way of exiting the loop. Spin report a deadlock as the program get stuck.
* `ex03.pml`: same as 2 but with printing. It is possible to replay the error trace with `spin -t ex03.pml`.
* `ex04.pml`: infinite loop. Even though the loop is infinite, the analysis terminates because it has a finite number of states.
* `ex05.pml`: if statement
* `ex06.pml`: unsafe if statement. All the enabled branches are explored, not only the first one.
* `ex07.pml`: more data, more states
* `ex08.pml`: even more data and even more states. To explore all the state, we need to increase the depth of the search (`-mN` option).
* `peterson.pml`: an encoding of [Peterson's mutual exclusion protocol](https://en.wikipedia.org/wiki/Peterson%27s_algorithm) in Spin
* `peterson2.pml`: also Peterson's protocol but a more compact version which uses more advanced features


## Additional References

If you need a deeper refresher or want to learn more about automata theory.
I recommend the following two books:
* Introduction to the Theory of Computation by Michael Sipser
* Introduction to Automata Theory, Languages, and Computation by John Hopcroft and Jeffrey Ullman
