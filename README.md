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

From now on, we will encode tails as <img src="/tex/29632a9bf827ce0200454dd32fc3be82.svg?invert_in_darkmode&sanitize=true" align=middle width=8.219209349999991pt height=21.18721440000001pt/> and heads as <img src="/tex/034d0a6be0424bffe9a6e7ac9236c0f5.svg?invert_in_darkmode&sanitize=true" align=middle width=8.219209349999991pt height=21.18721440000001pt/>. We will commonly refer to the field <img src="/tex/842a3ba6459f9c7d0b7724742b431bc1.svg?invert_in_darkmode&sanitize=true" align=middle width=40.18272059999999pt height=24.65753399999998pt/>.

We are interested in understanding 2 sets:

- <img src="/tex/9b325b9e31e85137d1de765f43c0f8bc.svg?invert_in_darkmode&sanitize=true" align=middle width=12.92464304999999pt height=22.465723500000017pt/>, the set of all possible chessboard states. Note <img src="/tex/e9c240a6f50a76c838e20dccd38baa35.svg?invert_in_darkmode&sanitize=true" align=middle width=65.29901894999999pt height=26.76175259999998pt/>
- <img src="/tex/e257acd1ccbe7fcb654708f1a866bfe9.svg?invert_in_darkmode&sanitize=true" align=middle width=11.027402099999989pt height=22.465723500000017pt/>, the set of all squares. Note <img src="/tex/727d3c9e2b77bcd0cb11822adb5a15af.svg?invert_in_darkmode&sanitize=true" align=middle width=58.51588049999999pt height=24.65753399999998pt/>

We would like to define a function that the prisoners can use to encode a square with the chessboard state:

<p align="center"><img src="/tex/6beef70c90af8c131a8bc1d8e304254b.svg?invert_in_darkmode&sanitize=true" align=middle width=73.03847264999999pt height=14.611878599999999pt/></p>

- Our objective is to convey a secret <img src="/tex/2d8cca33f0ee74986943da285a93a659.svg?invert_in_darkmode&sanitize=true" align=middle width=38.82401819999999pt height=22.465723500000017pt/>.
- Given some initial chessboard state <img src="/tex/87a45f06900f2b8979d64cee76c9d52a.svg?invert_in_darkmode&sanitize=true" align=middle width=47.504046149999986pt height=22.465723500000017pt/> chosen by the jailer
- Prisoner #1 will want to change the chessboard state from <img src="/tex/18c0a8d8429a250dc2f045fa67126640.svg?invert_in_darkmode&sanitize=true" align=middle width=13.666351049999989pt height=14.15524440000002pt/> to <img src="/tex/4ff28fe9f85fb1920ff4608be08af94d.svg?invert_in_darkmode&sanitize=true" align=middle width=47.504046149999986pt height=22.465723500000017pt/> by toggling a single coin such that <img src="/tex/7e0b0f84da75b2556c33a223c49083e0.svg?invert_in_darkmode&sanitize=true" align=middle width=66.71422064999999pt height=24.65753399999998pt/>
- Prisoner #2 will observe the chessboard state <img src="/tex/ce647d2c93bc38bf88d19d014430752e.svg?invert_in_darkmode&sanitize=true" align=middle width=13.666351049999989pt height=14.15524440000002pt/> and compute <img src="/tex/b322d107b998b55dd97d7a267bb2b1fa.svg?invert_in_darkmode&sanitize=true" align=middle width=37.09111064999999pt height=24.65753399999998pt/>, using that to guess <img src="/tex/6f9bad7347b91ceebebd3ad7e6f6f2d1.svg?invert_in_darkmode&sanitize=true" align=middle width=7.7054801999999905pt height=14.15524440000002pt/>

Since we want to understand how the function <img src="/tex/190083ef7a1625fbc75f243cffb9c96d.svg?invert_in_darkmode&sanitize=true" align=middle width=9.81741584999999pt height=22.831056599999986pt/> behaves with respect to changes in its input, we will need more algebraic structure. We need to make <img src="/tex/9b325b9e31e85137d1de765f43c0f8bc.svg?invert_in_darkmode&sanitize=true" align=middle width=12.92464304999999pt height=22.465723500000017pt/>
and <img src="/tex/e257acd1ccbe7fcb654708f1a866bfe9.svg?invert_in_darkmode&sanitize=true" align=middle width=11.027402099999989pt height=22.465723500000017pt/> groups, and make <img src="/tex/190083ef7a1625fbc75f243cffb9c96d.svg?invert_in_darkmode&sanitize=true" align=middle width=9.81741584999999pt height=22.831056599999986pt/> a group homomorphism.

### Review of Group Theory

#### What is a Group?
- A group <img src="/tex/5201385589993766eea584cd3aa6fa13.svg?invert_in_darkmode&sanitize=true" align=middle width=12.92464304999999pt height=22.465723500000017pt/> is the tuple <img src="/tex/68c91478b407d56f3e1870f1c19cc33f.svg?invert_in_darkmode&sanitize=true" align=middle width=57.950908949999985pt height=24.65753399999998pt/>.
- It is a set <img src="/tex/e257acd1ccbe7fcb654708f1a866bfe9.svg?invert_in_darkmode&sanitize=true" align=middle width=11.027402099999989pt height=22.465723500000017pt/> with an associative binary operator <img src="/tex/df33724455416439909c33a7db76b2bc.svg?invert_in_darkmode&sanitize=true" align=middle width=12.785434199999989pt height=19.1781018pt/>
- The set contains an identity element <img src="/tex/24f21c4f924a49dc5f3d6cd7b1cb4e89.svg?invert_in_darkmode&sanitize=true" align=middle width=38.77267679999999pt height=22.465723500000017pt/> such that <img src="/tex/4fef05c55c708a7fa54121562a8332a3.svg?invert_in_darkmode&sanitize=true" align=middle width=126.72878789999999pt height=22.831056599999986pt/>
- Each element <img src="/tex/2d8cca33f0ee74986943da285a93a659.svg?invert_in_darkmode&sanitize=true" align=middle width=38.82401819999999pt height=22.465723500000017pt/> has an inverse element <img src="/tex/a3d9b1b249a83ed17fc41dae0cd03536.svg?invert_in_darkmode&sanitize=true" align=middle width=51.60945239999998pt height=22.465723500000017pt/> such that <img src="/tex/994bf83f590b888995ca453eb1ef6a8a.svg?invert_in_darkmode&sanitize=true" align=middle width=152.29965464999998pt height=24.65753399999998pt/>
- The <img src="/tex/df33724455416439909c33a7db76b2bc.svg?invert_in_darkmode&sanitize=true" align=middle width=12.785434199999989pt height=19.1781018pt/> operator may or may not be commutative (when it is commutative, we call the group "Abelian")

#### Additional Definitions
- A group <img src="/tex/5201385589993766eea584cd3aa6fa13.svg?invert_in_darkmode&sanitize=true" align=middle width=12.92464304999999pt height=22.465723500000017pt/> might have a basis <img src="/tex/ed1f6d6ae453d46e49efd345103b65c4.svg?invert_in_darkmode&sanitize=true" align=middle width=140.34599864999998pt height=24.65753399999998pt/>. This means that every element in the group can be expressed as a sum of some set of basis elements: <img src="/tex/66982f9d5ccf3d48b0faf92d9053094f.svg?invert_in_darkmode&sanitize=true" align=middle width=132.8146908pt height=38.79006780000001pt/>
- A vector space of <img src="/tex/e293fa0f67d04b8f8108558c7ce4bde1.svg?invert_in_darkmode&sanitize=true" align=middle width=20.979946349999988pt height=22.465723500000017pt/> is a group consisting of vectors with <img src="/tex/55a049b8f161ae7cfeb0197d75aff967.svg?invert_in_darkmode&sanitize=true" align=middle width=9.86687624999999pt height=14.15524440000002pt/>-dimensions over the field <img src="/tex/b8bc815b5e9d5177af01fd4d3d3c2f10.svg?invert_in_darkmode&sanitize=true" align=middle width=12.85392569999999pt height=22.465723500000017pt/>.
- A standard basis is a basis for a vector space where each element is a vector of the form <img src="/tex/ab7126e4ad76a713043e3acadc024961.svg?invert_in_darkmode&sanitize=true" align=middle width=76.69167164999999pt height=24.65753399999998pt/>
- A group homomorphism is a function <img src="/tex/396ebc256c33dd601356a3467916d899.svg?invert_in_darkmode&sanitize=true" align=middle width=74.8650012pt height=22.831056599999986pt/> from group <img src="/tex/b8bc815b5e9d5177af01fd4d3d3c2f10.svg?invert_in_darkmode&sanitize=true" align=middle width=12.85392569999999pt height=22.465723500000017pt/> to group <img src="/tex/5201385589993766eea584cd3aa6fa13.svg?invert_in_darkmode&sanitize=true" align=middle width=12.92464304999999pt height=22.465723500000017pt/> such that <img src="/tex/0ff4d02bb1960f671388dfe24832f757.svg?invert_in_darkmode&sanitize=true" align=middle width=240.22221629999999pt height=24.65753399999998pt/>
- As corollaries to the above, for any group homomorphism <img src="/tex/190083ef7a1625fbc75f243cffb9c96d.svg?invert_in_darkmode&sanitize=true" align=middle width=9.81741584999999pt height=22.831056599999986pt/>, the following are true: <img src="/tex/2c43d9f3a62f8128b9c6bb7789635316.svg?invert_in_darkmode&sanitize=true" align=middle width=80.9913819pt height=24.65753399999998pt/> and <img src="/tex/9cd13c7a6ba13e3cf1c4fa8efca4fd6e.svg?invert_in_darkmode&sanitize=true" align=middle width=174.53758079999997pt height=24.65753399999998pt/>
- The preimage or inverse image of a set <img src="/tex/e257acd1ccbe7fcb654708f1a866bfe9.svg?invert_in_darkmode&sanitize=true" align=middle width=11.027402099999989pt height=22.465723500000017pt/> under <img src="/tex/190083ef7a1625fbc75f243cffb9c96d.svg?invert_in_darkmode&sanitize=true" align=middle width=9.81741584999999pt height=22.831056599999986pt/> is <img src="/tex/8fac22af20becca270bc855e487906cb.svg?invert_in_darkmode&sanitize=true" align=middle width=26.643980549999988pt height=26.76175259999998pt/> such that <img src="/tex/d2d20a0e7c0a100fb08694e79172bfbf.svg?invert_in_darkmode&sanitize=true" align=middle width=188.9652171pt height=26.76175259999998pt/>

### Applying Group Theory to the Problem

In order to make <img src="/tex/9b325b9e31e85137d1de765f43c0f8bc.svg?invert_in_darkmode&sanitize=true" align=middle width=12.92464304999999pt height=22.465723500000017pt/> a group, we need to be able to answer the question *what does it mean to add two chessboard states together?* We can answer this by reinterpreting <img src="/tex/9b325b9e31e85137d1de765f43c0f8bc.svg?invert_in_darkmode&sanitize=true" align=middle width=12.92464304999999pt height=22.465723500000017pt/> to be a set of chessboard state *deltas*. Note, this is a formality. There is essentially no difference between a group of states and a group of state deltas, except the latter motivates the intuition of what it means to add two group elements together. The identify element <img src="/tex/b8554c00e366606e9293d43bcd472ed8.svg?invert_in_darkmode&sanitize=true" align=middle width=40.66991939999998pt height=22.465723500000017pt/> could be interpreted as the zero delta (when interpreting as chessboard state deltas) or as the state of all 64 coins being tails-side up (when interpreting as chessboard states). Conceptually, prisoner #1 must choose an element from a set of 64 chessboard state deltas, which is a subset of <img src="/tex/9b325b9e31e85137d1de765f43c0f8bc.svg?invert_in_darkmode&sanitize=true" align=middle width=12.92464304999999pt height=22.465723500000017pt/>.

We will choose to make <img src="/tex/9b325b9e31e85137d1de765f43c0f8bc.svg?invert_in_darkmode&sanitize=true" align=middle width=12.92464304999999pt height=22.465723500000017pt/> the vector space <img src="/tex/a3e5d1ccf05bfb09b34a944361a4dcfa.svg?invert_in_darkmode&sanitize=true" align=middle width=53.28781424999998pt height=26.76175259999998pt/>. In other words, <img src="/tex/9b325b9e31e85137d1de765f43c0f8bc.svg?invert_in_darkmode&sanitize=true" align=middle width=12.92464304999999pt height=22.465723500000017pt/> is the group of 64-bit bitvectors, using XOR as the <img src="/tex/df33724455416439909c33a7db76b2bc.svg?invert_in_darkmode&sanitize=true" align=middle width=12.785434199999989pt height=19.1781018pt/> operator. Incidentally, these groups are also Abelian, in case we happen to use commutativity below.

- We choose this group as the standard basis <img src="/tex/e501c3fbaf8b825ac927568e9ecd1beb.svg?invert_in_darkmode&sanitize=true" align=middle width=48.13567769999999pt height=22.465723500000017pt/> is the set of valid moves that prisoner #1 can make.
- <img src="/tex/163af9574bd73b91a67e73d0a25c7ad6.svg?invert_in_darkmode&sanitize=true" align=middle width=54.19702364999999pt height=22.465723500000017pt/> is the move that prisoner #1 will make. Therefore, <img src="/tex/06e008cdad753b62dc068ef78d610d13.svg?invert_in_darkmode&sanitize=true" align=middle width=91.79782589999999pt height=22.465723500000017pt/>
- <img src="/tex/190083ef7a1625fbc75f243cffb9c96d.svg?invert_in_darkmode&sanitize=true" align=middle width=9.81741584999999pt height=22.831056599999986pt/> is a group homomorphism from <img src="/tex/87339665b58a744eea2d393d1988a61a.svg?invert_in_darkmode&sanitize=true" align=middle width=73.03847264999999pt height=22.831056599999986pt/>
- We know that <img src="/tex/e257acd1ccbe7fcb654708f1a866bfe9.svg?invert_in_darkmode&sanitize=true" align=middle width=11.027402099999989pt height=22.465723500000017pt/> is a set with 64 elements, but we have not yet decided how to make it into a group. That will come later.

Let's now solve for <img src="/tex/da6e5b842edb0a8ada1035cc77dd710f.svg?invert_in_darkmode&sanitize=true" align=middle width=20.812476299999993pt height=22.465723500000017pt/>, the move prisoner #1 should make:

<p align="center"><img src="/tex/d311c5dc9ce770fc5752203a12bb1202.svg?invert_in_darkmode&sanitize=true" align=middle width=66.71422065pt height=16.438356pt/></p>

<p align="center"><img src="/tex/2d18aaeb8d10623a282300584f7ac179.svg?invert_in_darkmode&sanitize=true" align=middle width=107.61788729999999pt height=16.438356pt/></p>

<p align="center"><img src="/tex/9ee287c4bf39272b96a5cb0a100bb270.svg?invert_in_darkmode&sanitize=true" align=middle width=130.22073404999998pt height=16.438356pt/></p>

<p align="center"><img src="/tex/319903fe16074cddbd22ceb6c9fef0c6.svg?invert_in_darkmode&sanitize=true" align=middle width=130.22073404999998pt height=16.438356pt/></p>

We will now introduce the usage of the inverse image, <img src="/tex/8fac22af20becca270bc855e487906cb.svg?invert_in_darkmode&sanitize=true" align=middle width=26.643980549999988pt height=26.76175259999998pt/>

<p align="center"><img src="/tex/ca9aaa36ef41a66615b4a197fd1d35d8.svg?invert_in_darkmode&sanitize=true" align=middle width=183.3259824pt height=18.312383099999998pt/></p>

<p align="center"><img src="/tex/230420a18b039dc4b4746b3a393a1026.svg?invert_in_darkmode&sanitize=true" align=middle width=133.25726534999998pt height=18.312383099999998pt/></p>
