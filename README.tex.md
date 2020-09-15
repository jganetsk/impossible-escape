# The Solution to the "Impossible Escape"

## The Puzzle

A jailer takes two prisoners and tells them "we're going to play a game. If you win, both of you will be set free. I will take prisoner #1 into a room. The room will contain an empty chessboard. I will point to one of the squares on the chessboard. This is the 'magic square'. I will then take 64 coins, and put a coin on each square of the chessboard. I will choose which coins are heads-side up and which are tails-side up. Prisoner #1 must then choose exactly one coin on the board and toggle it. I will then take prisoner #1 out of the room, and bring prisoner #2 into the room. Prisoner #2 will look at the chessboard, and must find the 'magic square'. There is only one guess. Sounds good? Before we start, you are allowed to talk to each other and plan out a strategy, but I will stand here and listen to you. Once the game starts, you will not be able to communicate with each other." What's the strategy?

## The Solution

### Prisoner #1 and Prisoner #2, prior to the game:
Enumerate all 64 squares on the board, 0..63. We will work with these values in binary, as 6-bit unsigned integers: 000000, 000001, 000010, etc.

For example, on a 4x4 board, you might enumerate the squares this way:
|||||
|-|-|-|-|
| 0 | 1 | 2 | 3 |
| 4 | 5 | 6 | 7 |
| 8 | 9 | 10 | 11|
| 12 | 13 | 14 | 15|

Or, in binary:

|||||
|-|-|-|-|
| 0000 | 0001 | 0010 | 0011 |
| 0100 | 0101 | 0110 | 0111 |
| 1000 | 1001 | 1010 | 1011|
| 1100 | 1101 | 1110 | 1111|

### Code

#### Common to Both Prisoners

```cpp
enum CoinState {
  Heads,
  Tails
};

// uint6 is a 6-bit unsigned integer, from 0 to 63
CoinState GetCoinState(uint6 square_index);

uint16 ComputeXOROfHeads() {
  uint6 xor_of_heads = 0;
  for (uint6 i = 0; i < 64; i++) {
    if (GetCoinState(i) == Heads) {
      xor_of_heads = xor_of_heads ^ i;  // This is a bitwise XOR
    }
  }
}

```

#### Prisoner #1

```cpp

void ToggleCoin(uint6 square_index);
uint6 GetMagicSquareIndex();

ToggleCoin(ComputeXOROfHeads() ^ GetMagicSquareIndex());
```

#### Prisoner #2

``` cpp

void GuessMagicSquare(uint6 square_index);

GuessMagicSquare(ComputeXOROfHeads());
```

### Note Well
- The number of squares on the board must be a power of 2. Otherwise, there is no solution. I will explore this in more detail below, using group theory. For an alternate explanation of this based on graph-coloring, please view https://www.youtube.com/watch?v=wTJI_WuZSwE. 
- There are ways to optimize the above algorithm to make it more human-friendly to compute.  For a description, please view https://www.youtube.com/watch?v=as7Gkm7Y7h4.

## The Explanation

### Set-up

From now on, we will encode tails as $0$ and heads as $1$. We will commonly refer to the field $\{0, 1\}$.

We are interested in understanding 2 sets:

- $C$, the set of all possible chessboard states. Note $|C|=2^{64}$
- $S$, the set of all squares. Note $|S|=64$

We would like to define a function that the prisoners can use to encode a square with the chessboard state:

$$f : C \rightarrow S$$

- Our objective is to convey a secret $s \in S$.
- Given some initial chessboard state $c_{0} \in C$ chosen by the jailer
- Prisoner #1 will want to change the chessboard state from $c_{0}$ to $c_{1} \in C$ by toggling a single coin such that $f(c_{1}) = s$
- Prisoner #2 will observe the chessboard state $c_{1}$ and compute $f(c_{1})$, using that to guess $s$

Since we want to understand how the function $f$ behaves with respect to changes in its input, we will need more algebraic structure. We need to make $C$
and $S$ groups, and make $f$ a group homomorphism.

### Review of Group Theory

#### What is a Group?
- A group $G$ is the tuple $(S, e, +)$.
- It is a set $S$ with an associative binary operator $+$
- The set contains an identity element $e \in S$ such that $\forall s \in S : s + e = s$
- Each element $s \in S$ has an inverse element $-s \in S$ such that $\forall s \in S : s + (-s) = e$
- The $+$ operator may or may not be commutative (when it is commutative, we call the group "Abelian")

#### Additional Definitions
- A group $G$ might have a basis $B = \{b_{1}, b_{2}, \dots, b_{n}\}, B \subset G$. This means that every element in the group can be expressed as a sum of some subset of basis elements: $\forall g \in G : g = \displaystyle\sum_{i} b_{i}$
- A vector space of $F^{n}$ is a group consisting of vectors with $n$-dimensions over the field $F$.
- A standard basis is a basis for a vector space where each element is a vector of the form $\left(\begin{smallmatrix}1 & 0 & 0 & \dots & 0 \end{smallmatrix}\right)$ or $\left(\begin{smallmatrix}0 & 1 & 0 & \dots & 0 \end{smallmatrix}\right)$, etc
- A group homomorphism is a function $f : F \rightarrow G$ from group $F$ to group $G$ such that $\forall a, b \in F : f(a + b) = f(a) + f(b)$
- As corollaries to the above, for any group homomorphism $f$, the following are true: $f(e_{F}) = e_{G}$ and $\forall a \in F : f(-a) = -f(a)$
- The image of a set $C$ under $f$ is $f[C] = \{f(c) \mid c \in C\}$
- The preimage or inverse image of a set $S$ under $f$ is $f^{-1}[S] = \{c \in C \mid f(c) \in S\}$

### Applying Group Theory to the Problem

In order to make $C$ a group, we need to be able to answer the question *what does it mean to add two chessboard states together?* We can answer this by reinterpreting $C$ to be a set of chessboard state *deltas*. Note, this is a formality. There is essentially no difference between a group of states and a group of state deltas, except the latter motivates the intuition of what it means to add two group elements together. The identify element $e \in C$ could be interpreted as the zero delta (when interpreting as chessboard state deltas) or as the state of all 64 coins being tails-side up (when interpreting as chessboard states). Conceptually, prisoner #1 must choose an element from a set of 64 chessboard state deltas, which is a subset of $C$.

We will choose to make $C$ the vector space $\{0, 1\}^{64}$. In other words, $C$ is the group of 64-bit bitvectors under the XOR operator. Incidentally, these groups are also Abelian, in case we happen to use commutativity below.

- Note the standard basis $B \subset C$ is the set of valid moves that prisoner #1 can make, since each standard basis element is a delta that toggles exactly one coin.
- Let $\Delta c \in B$ be the move that prisoner #1 will make. Therefore, $c_{1} = c_{0} + \Delta c$
- Let $f$ be a group homomorphism from $f : C \rightarrow S$
- We know that $S$ is a set with 64 elements, but we have not yet decided how to make it into a group. That will come later.

Let's now solve for $\Delta c$, the move prisoner #1 should make:

$$f(c_{1}) = s$$

$$f(c_{0} + \Delta c) = s$$

$$f(c_{0}) + f(\Delta c) = s$$

$$f(\Delta c) = s - f(c_{0})$$

We will now introduce the usage of the inverse image, $f^{-1}$

$$f^{-1}[f(\Delta c)] = f^{-1}[s - f(c_{0})]$$

Therefore:

$$\Delta c \in f^{-1}[s - f(c_{0})]$$

The set on the right-hand side contains all deltas that can be applied to $c_{0}$ to make $f(c_{1}) = s$. We need to guarantee that this set contains at least one legal move. In other words, it should contain at least one element $b$ in the standard basis $b \in B$.

To do this, we must guarantee that each element $s \in S$ must have at least one standard basis element $b \in B$ such that $f(b) = s$. In other words $f[B] = S$. This is simple to guarantee. There are 64 elements in $B$ (any basis of a 64-dimension vector space has 64 elements) and there are 64 elements in $S$ (as it is the set of all squares). This is made even easier by the fact that every standard basis element toggles exactly one coin, so it has already "picked a square". We can then construct a straightforward one-to-one mapping between the two sets $B$ and $S$. 

There are two main questions that remain:

1. We have defined the values of $f(b)$ where $b \in B$. We must now define the remaining values of $f(c)$ where $c \in C$.
1. We must still need to make $S$ into a group. Without this, we cannot compute $s - f(c_{0})$, for we have not defined the $+$ operator yet.

And the answers:

1. This is straightforward. Each element $c \in C$ can be expressed as a sum of basis elements $b_{i} \in B$, so $$c = \displaystyle\sum_{i} b_{i}$$ therefore $$f(c) = f(\displaystyle\sum_{i} b_{i})$$ and finally $$f(c) = \displaystyle\sum_{i} f(b_{i})$$ And we already have definitions of $f(b)$ for all $b \in B$, so we are done here.
1. Now comes the interesting part. What do use as a group for $S$?

#### Choosing a group for $S$

Much of what we've discussed above is mere formalism. Now we get to the core of the problem: choosing a group for $S$.

The first intuition many seem to have when trying to solve this problem is to make $S$ the additive group of integers mod 64, also known as $\mathbb{Z}/64\mathbb{Z}$. This is often chosen because it is a simple, well-known group. It seems promising at first, but you quickly run into problems when you discover that for almost all chessboard states $c \in C$, not every square $s \in S$ is reachable with a single coin toggle.

The theoretical reason for this there are no valid nontrivial homomorphisms from the group $\{0, 1\}^{64}$ to $\mathbb{Z}/64\mathbb{Z}$, so we would not be able to construct $f$.

The main insight is in realizing that $C$ is *self-inverting*: $\forall c \in C: c + c = e$. In other words, x XOR x is always zero. If we apply $f$, then we realize that $S$ must also be self-inverting, or $$\forall c \in C: f(c) + f(c) = e$$ or in other words $$\forall s \in S: s + s = e$$ or $$\forall s \in S: s = -s$$ 

$\mathbb{Z}/64\mathbb{Z}$ is not self-inverting. For example, $f$ would require that $-2 = 2$, but in this group $-2 = 62$. 

Which group has 64 elements and is self-inverting? The group $\{0, 1\}^{6}$. In other words, $S$ is 6-dimensional vector space over the field $\{0, 1\}$, also known as the group of 6-bit bitvectors under the XOR operator.

Recall that $$\Delta c \in f^{-1}[s - f(c_{0})]$$ Using the fact that $-s = s$ we can simplify this to $$\Delta c \in f^{-1}[s + f(c_{0})]$$


