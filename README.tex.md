# The Solution to the "Impossible Escape"

## The Puzzle

A jailer takes two prisoners and tells them "we're going to play a game. If you win, both of you will be set free. I will take prisoner #1 into a room. The room will contain an empty chessboard. I will point to one of the squares on the chessboard. This is the 'magic square'. I will then take 64 coins, and put a coin on each square of the chessboard. I will choose which coins are heads-side up and which are tails-side up. Prisoner #1 must then choose exactly one coin on the board and flip it. I will then take prisoner #1 out of the room, and bring prisoner #2 into the room. Prisoner #2 will look at the chessboard, and must find the 'magic square'. There is only one guess. Sounds good? Before we start, you are allowed to talk to each other and plan out a strategy, but I will stand here and listen to you. Once the game starts, you will not be able to communicate with each other." What's the strategy?

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

###  Prisoner #1
```cpp
enum CoinState {
  Heads,
  Tails
};

CoinState GetCoinState(int6 square_index);
void FlipCoin(int6 square_index);
int6 GetMagicSquare();

int6 square_to_flip = 0;
for (int6 i = 0; i < 64; i++) {
  if (GetCoinState(i) == Heads) {
    square_to_flip ^= i;  // This is a bitwise XOR
  }
}
square_to_flip ^= GetMagicSquare();

FlipCoin(square_to_flip);
```

### Prisoner #2
```cpp
enum Coin {
  Heads,
  Tails
};

Coin GetCoinState(int6 square_index);
void GuessMagicSquare(int6 square_index);

int6 square_to_guess = 0;
for (int6 i = 0; i < 64; i++) {
  if (GetCoinState(i) == Heads) {
    square_to_guess ^= i;  // This is a bitwise XOR
  }
}
GuessMagicSquare(square_to_guess);
```

### Note Well
- The number of squares on the board must be a power of 2. Otherwise, there is no solution. I will explore this in more detail below, using group theory. For an alternate explanation of this based on graph-coloring, please view https://www.youtube.com/watch?v=wTJI_WuZSwE. 
- There are ways to optimize the above algorithm to make it more human-friendly to compute.  For a description, please view https://www.youtube.com/watch?v=as7Gkm7Y7h4.

## The Group-Theoretic Explanation

### Heads vs Tails

From now on, we will encode tails as $0$ and heads as $1$. We will commonly refer to the field $\{0, 1\}$.

### Sets

We are interested in understanding 2 sets:

- $C$, the set of all possible chessboard states. Note $|C|=2^{64}$
- $S$, the set of all squares. Note $|S|=64$

We would like to define a function that the prisoners can use to encode a square with the chessboard state:

$$f : C \rightarrow S$$

- The name of the game is to convey a secret $s \in S$.
- Given some initial chessboard state $c_{0} \in C$
- Prisoner #1 will want to change the chessboard state from $c_{0}$ to $c_{1} \in C$ such that $f(c_{1}) = s$

### Groups

Since we want to understand how the function $f$ behaves with respect to changes in its input, we will need more algebraic structure. We need to make $C$
and $S$ groups. A group is a set with an associative binary operator, $+$. One element in the group must be the identity, and each element must have an inverse in the group.

In order to make $C$ a group, we need to be able to answer the question *what does it mean to add two board states together?* We can answer this by reinterpreting $C$ to be a set of chessboard state *deltas*. Note, this is a formality. The set of chessboard states is isomorphic to the set of chessboard state deltas. We can make $C$ the 64-dimensional vector space over the field $\{0, 1\}$. This is isomorphic to the group of 64-bit bitvectors with XOR as the $+$ operator, which may be a more intuitive representation for computer programmers.

- Note that the group $C$ has a basis number of $64$. Note also that the standard basis, $B \subset C$ is equivalent to the set of valid moves that prisoner #1 can make.
- We will now make $f$ a group homomorphism
- $\Delta c \in B$ is the move that prisoner #1 will make. Therefore, $c_{1} = c_{0} + \Delta c$

