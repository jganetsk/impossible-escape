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

### Sets

We are interested in understanding 2 sets:

- <p align="center"><img src="/tex/e242d5dd2ef4ef762db8b72b38a17315.svg?invert_in_darkmode&sanitize=true" align=middle width=12.924643049999998pt height=11.232861749999998pt/></p>, the set of all possible chessboard states. Note <p align="center"><img src="/tex/2e531d27250c776d8d4b1279fdda6461.svg?invert_in_darkmode&sanitize=true" align=middle width=65.29901895pt height=18.312383099999998pt/></p>
- <img src="https://render.githubusercontent.com/render/math?math=S">, the set of all squares. Note <img src="https://render.githubusercontent.com/render/math?math=|S|=64">

We would like to define a function that the prisoners can use to encode a square with the chessboard state:

<img src="https://render.githubusercontent.com/render/math?math=f : C \rightarrow S">

The name of the game is to convey a secret <img src="https://render.githubusercontent.com/render/math?math=s \in S">. Given some initial chessboard state <img src="https://render.githubusercontent.com/render/math?math=c_{0} \in C">, prisoner #1 will want to change the
chessboard state to <img src="https://render.githubusercontent.com/render/math?math=c_{1} \in C"> such that <img src="https://render.githubusercontent.com/render/math?math=f(c_{1}) = s">.

### Groups

Since we want to understand how the function <img src="https://render.githubusercontent.com/render/math?math=f"> behaves with respect to changes in its input, we will need more algebraic structure. We need to make <img src="https://render.githubusercontent.com/render/math?math=C"> 
and <img src="https://render.githubusercontent.com/render/math?math=S"> groups. A group is a set with an associative binary operator, <img src="https://render.githubusercontent.com/render/math?math=+">. One element in the group must be the identity, and each element must have an inverse in the group.

We can make <img src="https://render.githubusercontent.com/render/math?math=C"> a group by making it a 64-dimensional vector space over the field <img src="https://render.githubusercontent.com/render/math?math=\{0,1\}">
