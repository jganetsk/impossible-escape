# The Solution to the "Impossible Escape"

## The Puzzle

A jailer takes two prisoners and tells them "we're going to play a game. If you win, both of you will be set free. I will take prisoner #1 into a room. The room will contain an empty chessboard. I will point to one of the squares on the chessboard. This is the 'magic square'. I will then take 64 coins, and put a coin on each square of the chessboard. I will choose which coins are heads-side up and which are tails-side up. Prisoner #1 must then choose exactly one coin on the board and toggle it. I will then take prisoner #1 out of the room, and bring prisoner #2 into the room. Prisoner #2 will look at the chessboard, and must find the 'magic square'. There is only one guess. Sounds good? Before we start, you are allowed to talk to each other and plan out a strategy, but I will stand here and listen to you. Once the game starts, you will not be able to communicate with each other." What's the strategy?

## The Solution

### Prisoner #1 and Prisoner #2, prior to the game:
Enumerate all 64 squares on the board, 0..63. We will work with these values in binary, as 6-bit unsigned integers: 000000, 000001, 000010, etc.

For example, on a 4x4 board, you might enumerate the squares this way:
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
- Given some initial chessboard state $c_{0} \in C$
- Prisoner #1 will want to change the chessboard state from $c_{0}$ to $c_{1} \in C$ by toggling a single coin such that $f(c_{1}) = s$

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
- A group $G$ might have a basis $B = \{b_{1}, b_{2}, \dots, b_{n}\}$. This means that every element in the group can be expressed as a sum of basis elements. $\forall g \in G : g = \displaystyle\sum_{i} b_{i}$
- A vector space of $F^{n}$ is a group consisting of vectors with $n$-dimensions over the field $F$.
- A standard basis is a basis for a vector space where each element is of the form $\left(\begin{smallmatrix}1 & 0 & 0 & \dots & 0 b\end{smallmatrix}\right)$
- A group homomorphism is a function $f : F \rightarrow G$ from group $F$ to group $G$ such that $\forall a, b \in F : f(a + b) = f(a) + f(b)$
- As corollaries to the above, for any group homomorphism $f$, the following are true: $f(e_{F}) = e_{G}$ and $\forall a \in F : f(-a) = -f(a)$

### Applying Group Theory to the Problem

In order to make $C$ a group, we need to be able to answer the question *what does it mean to add two board states together?* We can answer this by reinterpreting $C$ to be a set of chessboard state *deltas*. Note, this is a formality. A set of chessboard states is isomorphic to a set of chessboard state deltas. 

Concretely, We will make $C$ the 64-dimensional vector space over the field $\{0, 1\}$. This is isomorphic to the group of 64-bit bitvectors, using XOR as the $+$ operator. This may be a more intuitive representation for computer programmers.

- Note that the group $C$ has a basis number of $64$. Note also that the standard basis, $B \subset C$ is equivalent to the set of valid moves that prisoner #1 can make.
- We will now make $f$ a group homomorphism
- $\Delta c \in B$ is the move that prisoner #1 will make. Therefore, $c_{1} = c_{0} + \Delta c$

