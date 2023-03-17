---
marp: true
theme: gaia
_class: invert
---

# Turing Completeness of Neural Networks

### D'Ambrosi Denis
##### March 2023

---

## Outline

1. Introduction
2. Theoretical Turing completeness of RNNs
3. Pratictical implementations of Turing-complete networks
4. Conclusions

---

## 1. Introduction

- Turing Machines
- Neural Networks

---

## Turing machines
##### Definition

Defined as a quadruple

$$(Q \cup \{q_{accept}, q_{reject}\}, \Gamma, \delta, q_0)$$

- $Q$ is a finite set of states
- $q_{accept}, q_{reject}$ are the final states that indicate the result of the computation
- $\Gamma$ is a finite alphabet of tape symbols
- $\delta$ is the transition function
- $q_0$ is the starting state

---

## Turing machines
##### Istantaneous descriptions

To study the evolution of a Turing Machine it's useful to consider the definition of an _istantaneous description_:

$$(q,l,r)$$

- $q$ is the current state of the machine
- $l$ is the tape to the left of the head
- $r$ is the tape to the right of the head

> Istantaneous descriptions can be considered as a "snapshot" of the machine at a given step

---

## Neural networks
##### Definition

Intuitively, a neural network is a directed graph of interconnected computing nodes (called _neurons_).

The local computation of each neuron $n$ can be summarized as the composition of three distinct functions:

- A _net_ function, which aggregates the values previously computed by the nodes that have an outgoing edge that terminates in $n$
- An _activation_ function, which combines the values computed by the net function and a threshold to determine the activation of the neuron
- An _output_ function, which transforms the activation into a value then passed to the following nodes within the graph

---

## Neural networks
##### Types of networks

There are many possible topologies of neural networks, but we can effectively classify them based on the presence or absence of loops within the _computational graph_:

- _FeedForward Neural Networks_ are acyclic, thus they only have a pre-determined number of steps before the computation comes to an halt
- _Recurrent Neural Networks_ allow loops in the topology to process input sequences of unbounded length

---

## 2. Theoretical Turing completeness of RNNs

- Turing completeness of general RNNs
- Turing completeness of modern architectures

---

## Turing completeness of general RNNs
##### Unbounded precision

We assume to have a discrete machine that is able to store and manipulate rational numbers with unbounded precision

We want to prove that, given a Turing machine $\mathcal{M}$ and an istantaneous description $x$, we are able to simulate $\mathcal{M}(x)$ through three cycles of computation of a RNN $\mathcal{T}_{W,b}$ executed with an encoding of $x$ as internal starting state.

---

## Turing completeness of general RNNs
##### Unbounded precision

Firstly we need to introduce the encoding function $\rho$.

$\rho$ concatenates multiple embeddings obtained through three auxiliary mappings:

- $\rho^{(q)}:Q \to \{0,1\}^{\lceil \log_2 Q \rceil}$ encodes the states $q_i \in Q$ as a binary enumeration
- $\rho^{(s)}: \Gamma^* \to \mathbb{Q}$ embeds the tape to the left or the right of the head using fractal encoding
- $\rho^{(r)}: \Gamma \to \{0,1\}^{|\Gamma|-1}$ encodes a single symbol of the alphabet $\Gamma$

---

## Turing completeness of general RNNs
###### Unbounded precision

A configuration of the machine can be thus encoded as

$$\rho(q, l, r) = \rho^{(q)}(q) \oplus \rho^{(s)}(l) \oplus \rho^{(s)}(r) \oplus \rho^{(r)}(l_1) \oplus \rho^{(r)}(r_1) \oplus \mathbf{0}$$

where $\oplus$ is the concatenation operator and $\mathbf{0}$ is a $|Q||\Gamma|+5$ vector of null values (that will be used to store auxiliary information during the simulation).

Note that $\rho$ is injective, thus we can define an inverse $\rho^{-1}$ that allows us to calculate the configuration corresponding to a valid encoding stored in the state of a RNN

---

## Turing completeness of general RNNs
##### Unbounded precision

The RNN we are going to construct uses the following functions:

- A weighted sum as _net_ function
- The linear staturated activation $\sigma$ as _activation_ function

$$\sigma(z) = 
\begin{cases}
    0 & \text{if } z < 0\\
    z & \text{if } 0 \leq z \leq 1\\
    1 & \text{if } z > 1
\end{cases}$$

- The identity as _output_ function

---

## Turing completeness of general RNNs
##### Unbounded precision

The state of the RNN at each timestamp $x(t)$ is defined recursively as following:

$$x(t+1) = \sigma(Wx(t) + b)$$

Where $x(0) = \rho(s)$ with $s$ configuration of the Turing machine $\mathcal{M}$ we want to simulate

---

## Turing completeness of general RNNs
##### Unbounded precision

Neurons within the network are partitioned into 6 main groups:

1. Stage neurons, that keep track in which of the 3 stages the RNN is
2. Entry neurons, which compute the combination of state and symbol under the head to determine the right transition according to $\mathcal{M}$'s $\delta$
3. Temporary tape neurons, which serve as a buffer to comute the transition of the head on the tape
4. Tape neurons, that encode the left and right tape in fractal encoding
5. Readout neurons, which encode the first symbol to the left and the right of the head
6. State neurons, that encode $\mathcal{M}$'s state

---

## Turing completeness of general RNNs
##### Unbounded precision

The simulation is executed during 3 main steps:

1. Entry neurons compute the state-symbol combination in order to determine the next transition according to $\mathcal{M}$'s $\delta$
2. State neurons and temporary tape neurons are updated accordin to the transition determined in the previous stage
3. In the third stage, tape neurons are updated with the temporary neurons' values

---

## Turing completeness of general RNNs
##### Unbounded precision

By applying the result we just outlined multiple times, we can simulate the complete computation of a Turing machine $\mathcal{M}$ over an arbitrary input string and compute the same function (if defined)

---

## Turing completeness of general RNNs
##### Growing memory modules

To remove the unbounded precision requirement, we can avoid to encode the entire tape into a single value by introducing two auxiliary stacks, each one controlled by two neurons $u$ and $o$ of the network

- if $u(t) > 0$, then a new neuron with the value $u(t)$ is pushed onto the stack and $u(t+1) = 0$.
- if $o(t) = 0$ and the stack is not empty, then the top neuron $n$ is popped from the stack and $o(t+1) = n$.
- if $o(t) = 0$ and the stack is empty, then $o(t+1) = c$ where $c$ is a default value.

---

## Turing completeness of general RNNs
##### Growing memory modules

We have to slighty modify the definition of the previously introduced $\rho$ to accomodate these changes:

- $\rho^{(s)}$ does not need to account for the infinite amount of blank symbols of the tape anymore
- $\rho^{(h)}:\{1,...,p\} \to \mathbb{Q}$ is used to encode the number of neurons of a tape $h(|l|), h(|r|)$ stored in the network at a given time ($p$ is the precision of the machine)
- $\rho^{(M)}:\Gamma^* \to \mathbb{Q}^*$ encodes the tape of the machine as a sequence of fractal encodings (computed with $\rho^{(s)}$) of at most $p$ symbols each

---

## Turing completeness of general RNNs
##### Growing memory modules

$\rho$ is thus defined as

$$\begin{align*}
\rho(q,s,l) = ( &\rho^{(q)}(q) \oplus \rho^{(s)}(l_{1:h(|l|)}) \oplus \rho^{(s)}(r_{1:h(|r|)})\\&\oplus \rho^{(s)}(l_{h(|l|)+1:h(|l|)+p}) \oplus \rho^{(s)}(l_{h(|r|)+1:h(|r|)+p})\\&\oplus \rho^{(r)}(l_1) \oplus \rho^{(r)}(r_1) \oplus \rho^{(h)}(h(|l|)) \oplus \rho^{(h)}(h(|r|)) \oplus \mathbf{0},\\&\rho^{(M)}(l), \rho^{(M)}(r))
\end{align*}$$

---

## Turing completeness of general RNNs
##### Growing memory modules

The "criterion" that allows the RNN to choose the stack-updating operation are the values of $h(|l|)$ and $h(|r|)$

Taking into consideration the left tape $l$, the value of $h(|l|)$ is updated to $h(|l'|)$ through the following cases:

$$
h(|l'|)=
    \begin{cases}
        h(|l|)-1 & \textrm{ if } d = L \textrm{ and } h(|l|) \geq 2\\
        p & \textrm{ if } d = L \textrm{ and } h(|l|) = 1\\
        h(|l|)+1 & \textrm{ if } d = R \textrm{ and } h(|l|) \leq p-1\\
        1 & \textrm{ if } d = R \textrm{ and } h(|l|) = p\\
    \end{cases}
$$

---

## Turing completeness of general RNNs
##### Growing memory modules

To succesfully simulate $\mathcal{M}$ with such architecture, we need 3 more categories of neurons:

7. Guard neurons, which will keep track of $h(|l|)$ and $h(|r|)$
8. Buffer neurons, that store temporary the values popped from the stacks
9. Push-pop neurons $(u_l, u_r)$ and $(o_l, o_r)$, which access the memory modules as described previously

---

## Turing completeness of general RNNs
##### Growing memory modules

The simulation is executed again during three main (analogous) steps. The main differences are:

- Tape neurons not only take into consideration temporary tapes, but also the values of the buffer neurons
- Readout neurons take into consideration also buffer neurons in the case of execution of a popping operation

---

## Turing completeness of general RNNs
##### Growing memory modules

As in the previous proof, by applying the RNN multiple (of 3) times, we can simulate the computation of the Turing machine $\mathcal{M}$

Although this architecture is technically implementable, the non-continuity of the stack-updating operations make all the learning methods based on differentiation unusable

---

## Turing completeness of general RNNs
##### Unbounded neurons

Another strategy to build a Turing-machine simulating network would be to allow an unlimited number of neurons in the architecture to overcome the problem of unbounded precision

To do so, we should, again, modify the encoding function $\rho$

- $\rho^{(M)}:Q^* \to \mathbb{Q}^{\rceil \frac{|l|+|r|}{p}\lceil}$ does the same thing as before, but instead of stacking the encoded parts of the tapes, it concatenates them into a vector
- $\rho^{(d)}:Q^* \to \{0,1\}^{\rceil \frac{|l|+|r|}{p}\lceil}$ embeds (with a one-hot-encoding) the coordinate of the last neuron encoding at least a non-blank symbol

---

## Turing completeness of general RNNs
##### Unbounded neurons

Again, we need to include some additional categories of nodes (but we can exclude pop-push neurons of course):

9. Stack neurons are introduced to replace growing memory modules
10. Pointer neurons, that must keep track of which neurons contain relevant information about the encoded tapes

Moreover, buffer neurons do not access values within the external modules, but, instead, read values from the last non-zero values of the stack neurons

---

## Turing completeness of general RNNs
##### Unbounded neurons

The simulation is analogous to the previous one, apart from the last stage, where the newly introduced neurons are used to update the "emulated" stacks: if the tape neuron holds less than $1$ symbol after the update, we pop the stack node (that is, setting the last non-zero stack neuron to $0$, changing the relative pointer nodes and updating the tape and readout values). On the other hand, if the tape neuron holds more than $p$ symbols after the update, we push the bottom $p$ symbols of the tape node onto the stack neurons (by setting the first zero neuron pointed by the pointer nodes to the value to be pushed).

---

## Turing completeness of modern architectures