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

- <img src="/tex/9b325b9e31e85137d1de765f43c0f8bc.svg?invert_in_darkmode&sanitize=true" align=middle width=12.92464304999999pt height=22.465723500000017pt/>, the set of all possible chessboard states. Note <img src="/tex/e9c240a6f50a76c838e20dccd38baa35.svg?invert_in_darkmode&sanitize=true" align=middle width=65.29901894999999pt height=26.76175259999998pt/>
- <img src="/tex/e257acd1ccbe7fcb654708f1a866bfe9.svg?invert_in_darkmode&sanitize=true" align=middle width=11.027402099999989pt height=22.465723500000017pt/>, the set of all squares. Note <img src="/tex/727d3c9e2b77bcd0cb11822adb5a15af.svg?invert_in_darkmode&sanitize=true" align=middle width=58.51588049999999pt height=24.65753399999998pt/>

We would like to define a function that the prisoners can use to encode a square with the chessboard state:

<p align="center"><img src="/tex/6beef70c90af8c131a8bc1d8e304254b.svg?invert_in_darkmode&sanitize=true" align=middle width=73.03847264999999pt height=14.611878599999999pt/></p>

The name of the game is to convey a secret <img src="/tex/2d8cca33f0ee74986943da285a93a659.svg?invert_in_darkmode&sanitize=true" align=middle width=38.82401819999999pt height=22.465723500000017pt/>. Given some initial chessboard state <img src="/tex/87a45f06900f2b8979d64cee76c9d52a.svg?invert_in_darkmode&sanitize=true" align=middle width=47.504046149999986pt height=22.465723500000017pt/>, prisoner #1 will want to change the
chessboard state to <img src="/tex/4ff28fe9f85fb1920ff4608be08af94d.svg?invert_in_darkmode&sanitize=true" align=middle width=47.504046149999986pt height=22.465723500000017pt/> such that <img src="/tex/7e0b0f84da75b2556c33a223c49083e0.svg?invert_in_darkmode&sanitize=true" align=middle width=66.71422064999999pt height=24.65753399999998pt/>

### Groups

Since we want to understand how the function <img src="/tex/190083ef7a1625fbc75f243cffb9c96d.svg?invert_in_darkmode&sanitize=true" align=middle width=9.81741584999999pt height=22.831056599999986pt/> behaves with respect to changes in its input, we will need more algebraic structure. We need to make <img src="/tex/9b325b9e31e85137d1de765f43c0f8bc.svg?invert_in_darkmode&sanitize=true" align=middle width=12.92464304999999pt height=22.465723500000017pt/>
and <img src="/tex/e257acd1ccbe7fcb654708f1a866bfe9.svg?invert_in_darkmode&sanitize=true" align=middle width=11.027402099999989pt height=22.465723500000017pt/> groups. A group is a set with an associative binary operator, <img src="/tex/df33724455416439909c33a7db76b2bc.svg?invert_in_darkmode&sanitize=true" align=middle width=12.785434199999989pt height=19.1781018pt/>. One element in the group must be the identity, and each element must have an inverse in the group.

We can make <img src="/tex/9b325b9e31e85137d1de765f43c0f8bc.svg?invert_in_darkmode&sanitize=true" align=middle width=12.92464304999999pt height=22.465723500000017pt/> a group by making it a 64-dimensional vector space over the field <img src="/tex/e3d7babdf84d997af0eefde61fc68c05.svg?invert_in_darkmode&sanitize=true" align=middle width=23.744301899999993pt height=21.18721440000001pt/>
