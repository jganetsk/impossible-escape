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

We are interested in understanding 2 sets:

- <img src="https://render.githubusercontent.com/render/math?math=C"> the set of all possible chessboard states. Note <img src="https://render.githubusercontent.com/render/math?math=|C|=2^{64}">
- $$C$$ the set of all possible chessboard states. Note $$|C|=2^{64}$$
- $$S$$ the set of all squares. Note $$|S|=64$$
