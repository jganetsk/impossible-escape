# The Solution to the "Impossible Escape"

## The Puzzle

A jailer takes two prisoners and tells them "we're going to play a game. If you win, both of you will be set free. I will take prisoner #1 into a room. The room will contain an empty chessboard. I will point to one of the squares on the chessboard. This is the 'magic square'. I will then take 64 coins, and put a coin on each square of the chessboard. I will choose which coins are heads-side up and which are tails-side up. Prisoner #1 must then choose exactly one coin on the board and toggle it. I will then take prisoner #1 out of the room, and bring prisoner #2 into the room. Prisoner #2 will look at the chessboard, and must find the 'magic square'. There is only one guess. Sounds good? Before we start, you are allowed to talk to each other and plan out a strategy, but I will stand here and listen to you. Once the game starts, you will not be able to communicate with each other." What's the strategy?

## The Solution

### Prisoner #1 and Prisoner #2, prior to the game:
Enumerate all 64 squares on the board, 0..63. We will work with these values in binary, as 6-bit unsigned integers: 000000, 000001, 000010, etc.

For example:
|||||||||
|-|-|-|-|-|-|-|-|
| 0 | 1 | 2 | 3 | 4 | 5 | 6 | 7 |
| 8 | 9 | 10 | 11 | 12 | 13 | 14 | 15 |
| 16 | 17 | 18 | 19 | 20 | 21 | 22 | 23 |
| 24 | 25 | 26 | 27 | 28 | 29 | 30 | 31 |
| 32 | 33 | 34 | 35 | 36 | 37 | 38 | 39 |
| 40 | 41 | 42 | 43 | 44 | 45 | 46 | 47 |
| 48 | 49 | 50 | 51 | 52 | 53 | 54 | 55 |
| 56 | 57 | 58 | 59 | 60 | 61 | 62 | 63 |

Or, in binary:


|||||||||
|-|-|-|-|-|-|-|-|
|  000000 |  000001 |  000010 |  000011 |  000100 |  000101 |  000110 |  000111 |  
|  001000 |  001001 |  001010 |  001011 |  001100 |  001101 |  001110 |  001111 |  
|  010000 |  010001 |  010010 |  010011 |  010100 |  010101 |  010110 |  010111 |  
|  011000 |  011001 |  011010 |  011011 |  011100 |  011101 |  011110 |  011111 |  
|  100000 |  100001 |  100010 |  100011 |  100100 |  100101 |  100110 |  100111 |  
|  101000 |  101001 |  101010 |  101011 |  101100 |  101101 |  101110 |  101111 |  
|  110000 |  110001 |  110010 |  110011 |  110100 |  110101 |  110110 |  110111 |  
|  111000 |  111001 |  111010 |  111011 |  111100 |  111101 |  111110 |  111111 |  

### Code

#### Common to Both Prisoners

```cpp
enum CoinState {
  Heads,
  Tails
};

// uint6 is a 6-bit unsigned integer, from 0 to 63
CoinState GetCoinState(uint6 square_index);

uint6 ComputeXOROfHeads() {
  uint6 xor_of_heads = 0;
  for (uint6 i = 0; i < 64; i++) {
    if (GetCoinState(i) == Heads) {
      xor_of_heads = xor_of_heads ^ i;  // This is a bitwise XOR
    }
  }
  return xor_of_heads;
}

```

#### Prisoner #1

```cpp

void ToggleCoin(uint6 square_index);
uint6 GetMagicSquareIndex();

ToggleCoin(GetMagicSquareIndex() ^ ComputeXOROfHeads());
```

#### Prisoner #2

``` cpp

void GuessMagicSquare(uint6 square_index);

GuessMagicSquare(ComputeXOROfHeads());
```

### Note Well
- The number of squares on the board must be a power of 2. Otherwise, there is no solution. I will explore this in more detail below, using group theory. There is an alternate explanation of this constraint based on graph-coloring [[YouTube video](https://www.youtube.com/watch?v=wTJI_WuZSwEi)]. 
- There are ways to optimize the above algorithm to make it more human-friendly to compute [[YouTube video](https://www.youtube.com/watch?v=as7Gkm7Y7h4)].

## The Short Explanation

Prisoner #1 sees board state $c_{0}$. Then, prisoner #1 computes $c_{0} \oplus s_{m}$ and toggles that coin to make the board state $c_{0} \oplus (c_{0} \oplus s_{m})$ which is equal to $s_{m}$.

## The Long Explanation

> Feynman's Algorithm:
>
> 1. Write down the problem.
> 2. Think very hard.
> 3. Write down the solution.

I find it insufficient to merely describe the solution, I am motivated to describe the methodology used to arrive at it. The point of algebra (both in its elementary form and its various more abstract forms) is to provide a means to express the constraints of a problem such that the solution flows natrually from the constraints. Instead of anticipating a logical leap to an answer, you should continue to refine the expression of constraints until the answer has no choice but to reveal itself to you.

### Set-up

We are interested in understanding 2 sets:

- $C$, the set of all possible chessboard states. Note $|C|=2^{64}$
- $S$, the set of all squares. Note $|S|=64$

We would like to define a function that the prisoners can use to encode a square with the chessboard state:

$$f : C \rightarrow S$$

- Our objective is to convey a secret $s_{m} \in S$.
- Given some initial chessboard state $c_{0} \in C$ chosen by the jailer
- Prisoner #1 will want to change the chessboard state from $c_{0}$ to $c_{1} \in C$ by toggling a single coin such that $f(c_{1}) = s_{m}$
- Prisoner #2 will observe the chessboard state $c_{1}$ and compute $f(c_{1})$, using that to guess $s_{m}$

Since we want to understand how the function $f$ behaves with respect to changes in its input, we will need more algebraic structure. We need to make $C$
and $S$ groups, and make $f$ a group homomorphism.

### Applying Group Theory to the Problem

I have included a [review of group theory in the appendix](#review-of-group-theory).

In order to make $C$ a group, we need to be able to answer the question *what does it mean to add two chessboard states together?* We can answer this by reinterpreting $C$ to be a set of chessboard state *deltas*. Note, this is a formality. There is essentially no difference between a group of states and a group of state deltas, except the latter motivates the intuition of what it means to add two group elements together. 

We will choose to make $C$ the vector space $\{0, 1\}^{64}$, alternately represented as the set of all 64-bit bitvectors under the XOR operator. To review, in the field $\{0, 1\}$, the following identities are true:

$$0 + 0 = 0$$

$$0 + 1 = 1$$

$$1 + 0 = 1$$

$$1 + 1 = 0$$

The meaning of $0$ vs $1$ depends on whether we are interpreting the group as chessboard states or chessboard state deltas:

|Interpretation of $C$|Meaning of $0$|Meaning of $1$|
|-|-|-|
|Chessboard states|Tails|Heads|
|Chessboard state deltas|Do not toggle the coin|Toggle the coin|

- Note the standard basis $B \subset C$, consisting of vectors such as $\left(\begin{smallmatrix}1 & 0 & 0 & \dots & 0 \end{smallmatrix}\right)$, is the set of valid moves that prisoner #1 can make, since each standard basis element is a delta that toggles exactly one coin.
- The identity element of $C$, known as $e$, is the 0 vector: $\left(\begin{smallmatrix}0 & 0 & 0 & \dots & 0 \end{smallmatrix}\right)$ 
- The inverse of every element in the group is itself, $\forall c \in C: c = -c$. Or, x XOR x is always 0. The intuition captured here is that toggling the same coin twice results in an unchanged board.
- $C$ happens to be Abelian. In other words, addition is commutative, or $x + y = y + x$

- Let $\Delta c \in B$ be the move that prisoner #1 will make. Therefore, $c_{1} = c_{0} + \Delta c$
- Let $f$ be a group homomorphism from $f : C \rightarrow S$. In particular, we want to guarantee that $f$ is surjective, aka "onto", aka $f[C] = S$. Otherwise, we would not be able to encode all possible magic squares.
- We know that $S$ is a set with 64 elements, but we have not yet decided how to make it into a group. That will come later.

Let's now solve for $\Delta c$, the move prisoner #1 should make:

$$f(c_{1}) = s_{m}$$

$$f(c_{0} + \Delta c) = s_{m}$$

$$f(c_{0}) + f(\Delta c) = s_{m}$$

$$f(\Delta c) = s_{m} - f(c_{0})$$

We will now introduce the usage of the inverse image, $f^{-1}$

$$f^{-1}[f(\Delta c)] = f^{-1}[s_{m} - f(c_{0})]$$

Therefore:

$$\Delta c \in f^{-1}[s_{m} - f(c_{0})]$$

The set on the right-hand side contains all deltas that can be applied to $c_{0}$ to make $f(c_{1}) = s_{m}$. We need to guarantee that this set contains at least one legal move. 

To do this, we must guarantee that each element $s \in S$ must have at least one standard basis element $b \in B$ such that $f(b) = s$. In other words $f[B] = S$. This is straightforward. There are 64 elements in $B$ (there are 64 possible valid moves prisoner #1 can make) and there are 64 elements in $S$ (there are 64 squares on the chessboard). This is made even easier by the fact that every standard basis element toggles exactly one coin, so it has already "picked a square". We can then construct a straightforward one-to-one mapping between the two sets $B$ and $S$. 

There are two main questions that remain:

1. We have defined the values of $f(b)$ where $b \in B$. We must now define the remaining values of $f(c)$ where $c \in C$.
1. We still need to make $S$ into a group. Without this, we cannot compute $s_{m} - f(c_{0})$, for we have not defined the $+$ operator yet.

And the answers:

1. Each element $c \in C$ can be expressed as a sum of basis elements, $\forall c \in C : c = \displaystyle\sum_{i} b_{i} \in B$ for the basis elements corresponding to the squares with a heads-up coin. The intuition captured here is that every chessboard state can be arrived at by starting with an all-tails board and proceeding with a series of valid moves (aka single coin toggles) $$c = \displaystyle\sum_{i} b_{i}$$ $$f(c) = f(\displaystyle\sum_{i} b_{i})$$ $$f(c) = \displaystyle\sum_{i} f(b_{i})$$ And we already have definitions of $f(b)$ for all $b \in B$, so we are done here.
1. Now comes the interesting part. What do use as a group for $S$?

#### Choosing a group for $S$

Much of what we've discussed above is mere formalism. Now we get to the core of the problem: choosing a group for $S$.

The first intuition many seem to have when trying to solve this problem is to make $S$ the additive group of integers mod 64, also known as $\mathbb{Z}/64\mathbb{Z}$. This is often chosen because it is a simple, well-known group. It seems promising at first, but you quickly run into problems when you discover that for almost all chessboard states $c \in C$, not every square $s \in S$ is reachable with a single coin toggle. This approach of just picking your favorite group does not work; the choice of group is constrained by the homomorphism $f$. 

The main clue is that every element in $C$ is its own inverse: $\forall c \in C: c = -c$. When we apply $f$, we realize that $S$ must also be self-inverting:

$$\forall c \in C: c = -c$$

$$\forall c \in C: f(c) = f(-c)$$

$$\forall c \in C: f(c) = -f(c)$$

Since $f[C] = S$:

$$\forall s \in S: s = -s$$

$\mathbb{Z}/64\mathbb{Z}$ is not self-inverting. For example, this would require that $-2 = 2$, but in that group $-2 = 62$. Therefore, we cannot construct a homomorphism $f$ such that $f[C] = \mathbb{Z}/64\mathbb{Z}$. It is impossible.

Which group has 64 elements and is self-inverting? The vector space $\{0, 1\}^{6}$, or the set of 6-bit bitvectors under XOR. It turns out, $S$ must be the "same kind of group" as $C$, also a vector space over $\{0, 1\}$. We go into more detail about this below.

You may not yet be convinced that $f$ is a homomorphism. We can represent $f$ as $f(c) = Mc$ where $M$ is a constant $6 \times 64$ matrix, $c$ is a $64 \times 1$ vector, and $f(c)$ is a $6 \times 1$ vector. $M$ looks like this: 

$$\begin{pmatrix}
0 & 0 & 0 & 0 & 0 & 0 & 0 & \dots & 1\\
0 & 0 & 0 & 0 & 0 & 0 & 0 & \dots & 1\\
0 & 0 & 0 & 0 & 0 & 0 & 0 & \dots & 1\\
0 & 0 & 0 & 0 & 1 & 1 & 1 & \dots & 1\\
0 & 0 & 1 & 1 & 0 & 0 & 1 & \dots & 1\\
0 & 1 & 0 & 1 & 0 & 1 & 0 & \dots & 1\\
\end{pmatrix}$$

Matrix multiplication is a form of group homomorphism.

#### A Small Simplification

Recall:

$$\Delta c \in f^{-1}[s_{m} - f(c_{0})]$$

$$\forall s \in S: s = -s$$

We can make this minor simplification:

$$\Delta c \in f^{-1}[s_{m} + f(c_{0})]$$

### Mapping the Solution to Code

We will now translate the math above into the C++ code in the first section.

- Wherever we see $+$, we replace it with `^`, the bitwise XOR operator.
- Wherever we see $f(c \in C)$ where $c$ is the current board state, we replace it with `ComputeXOROfHeads()`.
- Wherever we see $f(b \in B)$, we replace it with a `uint6` representing the 6-bit encoding of the square on the chessboard.
- The implementation of `ComputeXOROfHeads` corresponds to $f(c) = \displaystyle\sum_{i} f(b_{i})$
- `GetMagicSquareIndex() ^ ComputeXOROfHeads()` corresponds to $s_{m} + f(c_{0})$

## The Number of Squares on the Board Must Be a Power of 2

### The Short Explanation

- Imagine a 9x9 board. There will be 81 squares on it. We need 7 bits to represent the square indices, 0 to 80 inclusive. However, the full range representable by 7 bits is 0 to 127 inclusive. This means some board states encode values of 81 to 127, which do not correspond to any magic square. Prisoner #1 will have 81 options out of a space of 128, and it is not guaranteed that the option prisoner #1 needs will be present in the set.
- Another explanation [[YouTube video](https://www.youtube.com/watch?v=wTJI_WuZSwE)] looks at the problem from the standpoint of graph coloring. Imagine a graph arranged like an $n$-dimensional hypercube, where $n$ is the number of squares on the chessboard. There are $n$ colors. We need to assign each node a color such that all $n$ colors are reachable from one of its direct neighbors. The number of nodes will always be a power of 2 (due to the number of possible chessboard states being $2^n$). If the number of colors is not also a power of 2, it will not divide evenly. Some colors will be represented more than others. The video has a more detailed explanation.

### The Long Explanation

Remember above we said that, in order to guarantee the existence of a legal move in $f^{-1}[s_{m} + f(c_{0})]$, we must guarantee $f[B] = S$. This was assuming $f[C] = S$. What happens if the latter is not true?
- Let's say that $S \subset f[C]$. We cannot even state at this point that $S$ is a group. It might only be a set.
- We can still have a one-to-one mapping between $B$ and $S$ (there is still the same number of legal moves as there are possible magic squares). But then $f[B] \subset f[C]$
- The jailer has control over $s_{m}$ and $c_{0}$, which implies the jailer has control over $s_{m} + f(c_{0})$. The jailer could then choose values such that $s_{m} + f(c_{0}) \notin f[B]$. And then prisoner #1 would have no legal move that could be made to put the board into state $c_{1}$ such that $f(c_{1}) = s_{m}$.

So can we guarantee that $f[C] = S$? 

The answer is yes, if and only if $|S|$ is a power of 2. Because of the [fundamental homomorphism theorem](https://en.wikipedia.org/wiki/Isomorphism_theorems#Theorem_A):

> Fundamental Homomorphism Theorem:
>
> Let $G$ and $H$ be groups, and let $f : G \rightarrow H$ be a homomorphism.
> 1. The [kernel](https://en.wikipedia.org/wiki/Kernel_(algebra)) of $f$ is a normal subgroup of $G$
> 2. The image of $f$ is a subgroup of $H$
> 3. The image of $f$ is isomorphic to the quotient group $G / ker(f)$

Now we are getting into more advanced territory. Without explaining all the derivations, we are going to quickly compose a bunch of theorems to prove that the order of the image of any homomorphism from $C$ is a power of 2. If you are interested, please do some independent research! The details behind these theorems are fairly accessible!

- We need to know the definition of "[normal subgroup](https://en.wikipedia.org/wiki/Normal_subgroup)". But every subgroup of an Abelian group is normal, and $C$ is Abelian, so we can nip that complexity in the bud.
- [LaGrange's theorem](https://en.wikipedia.org/wiki/Lagrange%27s_theorem_(group_theory)) states that for any finite group $G$, the order of every subgroup $H$ of $G$ evenly divides the order of $G$. Since the order of $G$ here is $2^{n}$, then every subgroup (and hence every possible kernel) has order $2^{m}$ where $0 \leq m \leq n$.

At this point, if we know how many elements are in the [quotient group](https://en.wikipedia.org/wiki/Quotient_group) $G / ker(f)$, then we know how many elements are in the image of $f$. And yes, it is as simple as $|G| / |ker(f)|$, which is $2^{n-m}$ where $0 \leq n-m \leq n$.

Note that the fact that the image of $f$ is isomorphic to the quotient group $G / ker(f)$ elaborates on the claim above that $S$ must be the "same kind of group" as $C$. In fact, all homomorphism images of $\{0, 1\}^{n}$ are isomorphic to $\{0, 1\}^{m}, 0 \leq m \leq n$. In other words, they are all vector spaces over the same field.

## Appendix

### Review of Group Theory

#### What is a Group?
- A group $G$ is the tuple $(S, e, +)$.
- It is a set $S$ with an associative binary operator $+$
- The set contains an identity element $e \in S$ such that $\forall s \in S : s + e = s$
- Each element $s \in S$ has an inverse element $-s \in S$ such that $\forall s \in S : s + (-s) = e$
- The $+$ operator may or may not be commutative (when it is commutative, we call the group "Abelian")

#### Additional Definitions
- A vector space $F^{n}$ is a group consisting of vectors with $n$-dimensions over the field $F$.
- Every vector space $F^{n}$ has a basis: a set of $n$ elements $B = \{b_{1}, b_{2}, \dots, b_{n}\} \subset F^{n}$ such that each element of the vector space can be expressed as a unique sum of basis elements: $\forall v \in F^{n} : v = \displaystyle\sum_{i=1}^{n} f_{i}b_{i}$ where $f_{i} \in F, b_{i} \in B$
- A standard basis is a basis of a vector space where each basis element is a vector of the form $\left(\begin{smallmatrix}1 & 0 & 0 & \dots & 0 \end{smallmatrix}\right)$ or $\left(\begin{smallmatrix}0 & 1 & 0 & \dots & 0 \end{smallmatrix}\right)$, etc
- A group homomorphism is a function $f : F \rightarrow G$ from group $F$ to group $G$ such that $\forall a, b \in F : f(a + b) = f(a) + f(b)$
- As corollaries to the above, for any group homomorphism $f$, the following are true: $f(e_{F}) = e_{G}$ and $\forall a \in F : f(-a) = -f(a)$
- The image of a set $C$ under $f$ is $f[C] = \{f(c) \mid c \in C\}$
- The preimage or inverse image of a set $S$ under $f$ is $f^{-1}[S] = \{c \in C \mid f(c) \in S\}$
