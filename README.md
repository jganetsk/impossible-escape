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
- A group <img src="/tex/5201385589993766eea584cd3aa6fa13.svg?invert_in_darkmode&sanitize=true" align=middle width=12.92464304999999pt height=22.465723500000017pt/> might have a basis <img src="/tex/721ff0c64addc839dad321a81ebb19f8.svg?invert_in_darkmode&sanitize=true" align=middle width=195.78756119999997pt height=24.65753399999998pt/>. This means that every element in the group can be expressed as a sum of some subset of basis elements: <img src="/tex/66982f9d5ccf3d48b0faf92d9053094f.svg?invert_in_darkmode&sanitize=true" align=middle width=132.8146908pt height=38.79006780000001pt/>
- A vector space of <img src="/tex/e293fa0f67d04b8f8108558c7ce4bde1.svg?invert_in_darkmode&sanitize=true" align=middle width=20.979946349999988pt height=22.465723500000017pt/> is a group consisting of vectors with <img src="/tex/55a049b8f161ae7cfeb0197d75aff967.svg?invert_in_darkmode&sanitize=true" align=middle width=9.86687624999999pt height=14.15524440000002pt/>-dimensions over the field <img src="/tex/b8bc815b5e9d5177af01fd4d3d3c2f10.svg?invert_in_darkmode&sanitize=true" align=middle width=12.85392569999999pt height=22.465723500000017pt/>.
- A standard basis is a basis for a vector space where each element is a vector of the form <img src="/tex/ab7126e4ad76a713043e3acadc024961.svg?invert_in_darkmode&sanitize=true" align=middle width=76.69167164999999pt height=24.65753399999998pt/> or <img src="/tex/7ea33751c9927247cb3cdaa34e6e3db2.svg?invert_in_darkmode&sanitize=true" align=middle width=76.69167164999999pt height=24.65753399999998pt/>, etc
- A group homomorphism is a function <img src="/tex/396ebc256c33dd601356a3467916d899.svg?invert_in_darkmode&sanitize=true" align=middle width=74.8650012pt height=22.831056599999986pt/> from group <img src="/tex/b8bc815b5e9d5177af01fd4d3d3c2f10.svg?invert_in_darkmode&sanitize=true" align=middle width=12.85392569999999pt height=22.465723500000017pt/> to group <img src="/tex/5201385589993766eea584cd3aa6fa13.svg?invert_in_darkmode&sanitize=true" align=middle width=12.92464304999999pt height=22.465723500000017pt/> such that <img src="/tex/0ff4d02bb1960f671388dfe24832f757.svg?invert_in_darkmode&sanitize=true" align=middle width=240.22221629999999pt height=24.65753399999998pt/>
- As corollaries to the above, for any group homomorphism <img src="/tex/190083ef7a1625fbc75f243cffb9c96d.svg?invert_in_darkmode&sanitize=true" align=middle width=9.81741584999999pt height=22.831056599999986pt/>, the following are true: <img src="/tex/2c43d9f3a62f8128b9c6bb7789635316.svg?invert_in_darkmode&sanitize=true" align=middle width=80.9913819pt height=24.65753399999998pt/> and <img src="/tex/9cd13c7a6ba13e3cf1c4fa8efca4fd6e.svg?invert_in_darkmode&sanitize=true" align=middle width=174.53758079999997pt height=24.65753399999998pt/>
- The image of a set <img src="/tex/9b325b9e31e85137d1de765f43c0f8bc.svg?invert_in_darkmode&sanitize=true" align=middle width=12.92464304999999pt height=22.465723500000017pt/> under <img src="/tex/190083ef7a1625fbc75f243cffb9c96d.svg?invert_in_darkmode&sanitize=true" align=middle width=9.81741584999999pt height=22.831056599999986pt/> is <img src="/tex/2e140ce473466495ad522e51466335b6.svg?invert_in_darkmode&sanitize=true" align=middle width=153.77519849999996pt height=24.65753399999998pt/>
- The preimage or inverse image of a set <img src="/tex/e257acd1ccbe7fcb654708f1a866bfe9.svg?invert_in_darkmode&sanitize=true" align=middle width=11.027402099999989pt height=22.465723500000017pt/> under <img src="/tex/190083ef7a1625fbc75f243cffb9c96d.svg?invert_in_darkmode&sanitize=true" align=middle width=9.81741584999999pt height=22.831056599999986pt/> is <img src="/tex/549ed9332e691a07b360f1de8c200533.svg?invert_in_darkmode&sanitize=true" align=middle width=200.64492194999997pt height=26.76175259999998pt/>

### Applying Group Theory to the Problem

In order to make <img src="/tex/9b325b9e31e85137d1de765f43c0f8bc.svg?invert_in_darkmode&sanitize=true" align=middle width=12.92464304999999pt height=22.465723500000017pt/> a group, we need to be able to answer the question *what does it mean to add two chessboard states together?* We can answer this by reinterpreting <img src="/tex/9b325b9e31e85137d1de765f43c0f8bc.svg?invert_in_darkmode&sanitize=true" align=middle width=12.92464304999999pt height=22.465723500000017pt/> to be a set of chessboard state *deltas*. Note, this is a formality. There is essentially no difference between a group of states and a group of state deltas, except the latter motivates the intuition of what it means to add two group elements together. The identify element <img src="/tex/b8554c00e366606e9293d43bcd472ed8.svg?invert_in_darkmode&sanitize=true" align=middle width=40.66991939999998pt height=22.465723500000017pt/> could be interpreted as the zero delta (when interpreting as chessboard state deltas) or as the state of all 64 coins being tails-side up (when interpreting as chessboard states). Conceptually, prisoner #1 must choose an element from a set of 64 chessboard state deltas, which is a subset of <img src="/tex/9b325b9e31e85137d1de765f43c0f8bc.svg?invert_in_darkmode&sanitize=true" align=middle width=12.92464304999999pt height=22.465723500000017pt/>.

We will choose to make <img src="/tex/9b325b9e31e85137d1de765f43c0f8bc.svg?invert_in_darkmode&sanitize=true" align=middle width=12.92464304999999pt height=22.465723500000017pt/> the vector space <img src="/tex/a3e5d1ccf05bfb09b34a944361a4dcfa.svg?invert_in_darkmode&sanitize=true" align=middle width=53.28781424999998pt height=26.76175259999998pt/>. In other words, <img src="/tex/9b325b9e31e85137d1de765f43c0f8bc.svg?invert_in_darkmode&sanitize=true" align=middle width=12.92464304999999pt height=22.465723500000017pt/> is the group of 64-bit bitvectors under the XOR operator. Incidentally, these groups are also Abelian, in case we happen to use commutativity below.

- Note the standard basis <img src="/tex/e501c3fbaf8b825ac927568e9ecd1beb.svg?invert_in_darkmode&sanitize=true" align=middle width=48.13567769999999pt height=22.465723500000017pt/> is the set of valid moves that prisoner #1 can make, since each standard basis element is a delta that toggles exactly one coin.
- Let <img src="/tex/163af9574bd73b91a67e73d0a25c7ad6.svg?invert_in_darkmode&sanitize=true" align=middle width=54.19702364999999pt height=22.465723500000017pt/> be the move that prisoner #1 will make. Therefore, <img src="/tex/06e008cdad753b62dc068ef78d610d13.svg?invert_in_darkmode&sanitize=true" align=middle width=91.79782589999999pt height=22.465723500000017pt/>
- Let <img src="/tex/190083ef7a1625fbc75f243cffb9c96d.svg?invert_in_darkmode&sanitize=true" align=middle width=9.81741584999999pt height=22.831056599999986pt/> be a group homomorphism from <img src="/tex/87339665b58a744eea2d393d1988a61a.svg?invert_in_darkmode&sanitize=true" align=middle width=73.03847264999999pt height=22.831056599999986pt/>
- We know that <img src="/tex/e257acd1ccbe7fcb654708f1a866bfe9.svg?invert_in_darkmode&sanitize=true" align=middle width=11.027402099999989pt height=22.465723500000017pt/> is a set with 64 elements, but we have not yet decided how to make it into a group. That will come later.

Let's now solve for <img src="/tex/da6e5b842edb0a8ada1035cc77dd710f.svg?invert_in_darkmode&sanitize=true" align=middle width=20.812476299999993pt height=22.465723500000017pt/>, the move prisoner #1 should make:

<p align="center"><img src="/tex/d311c5dc9ce770fc5752203a12bb1202.svg?invert_in_darkmode&sanitize=true" align=middle width=66.71422065pt height=16.438356pt/></p>

<p align="center"><img src="/tex/2d18aaeb8d10623a282300584f7ac179.svg?invert_in_darkmode&sanitize=true" align=middle width=107.61788729999999pt height=16.438356pt/></p>

<p align="center"><img src="/tex/9ee287c4bf39272b96a5cb0a100bb270.svg?invert_in_darkmode&sanitize=true" align=middle width=130.22073404999998pt height=16.438356pt/></p>

<p align="center"><img src="/tex/319903fe16074cddbd22ceb6c9fef0c6.svg?invert_in_darkmode&sanitize=true" align=middle width=130.22073404999998pt height=16.438356pt/></p>

We will now introduce the usage of the inverse image, <img src="/tex/8fac22af20becca270bc855e487906cb.svg?invert_in_darkmode&sanitize=true" align=middle width=26.643980549999988pt height=26.76175259999998pt/>

<p align="center"><img src="/tex/0c2522212dfa8ffa37a2fb6a680506c9.svg?invert_in_darkmode&sanitize=true" align=middle width=203.41737074999998pt height=18.312383099999998pt/></p>

Therefore:

<p align="center"><img src="/tex/dd429c45ce74c72091aa2141e3e7946a.svg?invert_in_darkmode&sanitize=true" align=middle width=142.3897134pt height=18.312383099999998pt/></p>

The set on the right-hand side contains all deltas that can be applied to <img src="/tex/18c0a8d8429a250dc2f045fa67126640.svg?invert_in_darkmode&sanitize=true" align=middle width=13.666351049999989pt height=14.15524440000002pt/> to make <img src="/tex/7e0b0f84da75b2556c33a223c49083e0.svg?invert_in_darkmode&sanitize=true" align=middle width=66.71422064999999pt height=24.65753399999998pt/>. We need to guarantee that this set contains at least one legal move. In other words, it should contain at least one element <img src="/tex/4bdc8d9bcfb35e1c9bfb51fc69687dfc.svg?invert_in_darkmode&sanitize=true" align=middle width=7.054796099999991pt height=22.831056599999986pt/> in the standard basis <img src="/tex/760318998b5f14fb225b3862a8ff1901.svg?invert_in_darkmode&sanitize=true" align=middle width=40.43934509999999pt height=22.831056599999986pt/>.

To do this, we must guarantee that each element <img src="/tex/2d8cca33f0ee74986943da285a93a659.svg?invert_in_darkmode&sanitize=true" align=middle width=38.82401819999999pt height=22.465723500000017pt/> must have at least one standard basis element <img src="/tex/760318998b5f14fb225b3862a8ff1901.svg?invert_in_darkmode&sanitize=true" align=middle width=40.43934509999999pt height=22.831056599999986pt/> such that <img src="/tex/60b2610f4905c8c159e74ff3539bb734.svg?invert_in_darkmode&sanitize=true" align=middle width=59.280752849999985pt height=24.65753399999998pt/>. In other words <img src="/tex/71cb5ef0976705bb1329f4431a68cb9b.svg?invert_in_darkmode&sanitize=true" align=middle width=65.18829734999998pt height=24.65753399999998pt/>. This is simple to guarantee. There are 64 elements in <img src="/tex/61e84f854bc6258d4108d08d4c4a0852.svg?invert_in_darkmode&sanitize=true" align=middle width=13.29340979999999pt height=22.465723500000017pt/> (any basis of a 64-dimension vector space has 64 elements) and there are 64 elements in <img src="/tex/e257acd1ccbe7fcb654708f1a866bfe9.svg?invert_in_darkmode&sanitize=true" align=middle width=11.027402099999989pt height=22.465723500000017pt/> (as it is the set of all squares). This is made even easier by the fact that every standard basis element toggles exactly one coin, so it has already "picked a square". We can then construct a straightforward one-to-one mapping between the two sets <img src="/tex/61e84f854bc6258d4108d08d4c4a0852.svg?invert_in_darkmode&sanitize=true" align=middle width=13.29340979999999pt height=22.465723500000017pt/> and <img src="/tex/e257acd1ccbe7fcb654708f1a866bfe9.svg?invert_in_darkmode&sanitize=true" align=middle width=11.027402099999989pt height=22.465723500000017pt/>. 

There are two main questions that remain:

1. We have defined the values of <img src="/tex/bc47eb54c896114e803bd8d0e087caf4.svg?invert_in_darkmode&sanitize=true" align=middle width=29.657642849999988pt height=24.65753399999998pt/> where <img src="/tex/760318998b5f14fb225b3862a8ff1901.svg?invert_in_darkmode&sanitize=true" align=middle width=40.43934509999999pt height=22.831056599999986pt/>. We must now define the remaining values of <img src="/tex/01b5d07b6b39ee917e3eb6119953d672.svg?invert_in_darkmode&sanitize=true" align=middle width=29.716650149999992pt height=24.65753399999998pt/> where <img src="/tex/374ff7af31c1eefcd9d7c9f484d61d43.svg?invert_in_darkmode&sanitize=true" align=middle width=40.12958564999999pt height=22.465723500000017pt/>.
1. We must still need to make <img src="/tex/e257acd1ccbe7fcb654708f1a866bfe9.svg?invert_in_darkmode&sanitize=true" align=middle width=11.027402099999989pt height=22.465723500000017pt/> into a group. Without this, we cannot compute <img src="/tex/0ec35fad8786a167574244d347967252.svg?invert_in_darkmode&sanitize=true" align=middle width=64.88778119999999pt height=24.65753399999998pt/>, for we have not defined the <img src="/tex/df33724455416439909c33a7db76b2bc.svg?invert_in_darkmode&sanitize=true" align=middle width=12.785434199999989pt height=19.1781018pt/> operator yet.

And the answers:

1. This is straightforward. Each element <img src="/tex/374ff7af31c1eefcd9d7c9f484d61d43.svg?invert_in_darkmode&sanitize=true" align=middle width=40.12958564999999pt height=22.465723500000017pt/> can be expressed as a sum of basis elements <img src="/tex/d2b3d983e09b253946f6fe3c72098eda.svg?invert_in_darkmode&sanitize=true" align=middle width=45.91214099999999pt height=22.831056599999986pt/>, so <p align="center"><img src="/tex/7223690b61ae166c528c67916071d7e3.svg?invert_in_darkmode&sanitize=true" align=middle width=67.2211287pt height=36.6554298pt/></p> therefore <p align="center"><img src="/tex/a8112cdbf2fc8e87d688a8b8c998f32e.svg?invert_in_darkmode&sanitize=true" align=middle width=113.2487202pt height=36.6554298pt/></p> and finally <p align="center"><img src="/tex/3c93abebd3b56d1fbd29ce4f417a8734.svg?invert_in_darkmode&sanitize=true" align=middle width=113.2487202pt height=36.6554298pt/></p> And we already have definitions of <img src="/tex/bc47eb54c896114e803bd8d0e087caf4.svg?invert_in_darkmode&sanitize=true" align=middle width=29.657642849999988pt height=24.65753399999998pt/> for all <img src="/tex/760318998b5f14fb225b3862a8ff1901.svg?invert_in_darkmode&sanitize=true" align=middle width=40.43934509999999pt height=22.831056599999986pt/>, so we are done here.
1. Now comes the interesting part. What do use as a group for <img src="/tex/e257acd1ccbe7fcb654708f1a866bfe9.svg?invert_in_darkmode&sanitize=true" align=middle width=11.027402099999989pt height=22.465723500000017pt/>?

#### Choosing a group for <img src="/tex/e257acd1ccbe7fcb654708f1a866bfe9.svg?invert_in_darkmode&sanitize=true" align=middle width=11.027402099999989pt height=22.465723500000017pt/>

Much of what we've discussed above is mere formalism. Now we get to the core of the problem: choosing a group for <img src="/tex/e257acd1ccbe7fcb654708f1a866bfe9.svg?invert_in_darkmode&sanitize=true" align=middle width=11.027402099999989pt height=22.465723500000017pt/>.

The first intuition many seem to have when trying to solve this problem is to make <img src="/tex/e257acd1ccbe7fcb654708f1a866bfe9.svg?invert_in_darkmode&sanitize=true" align=middle width=11.027402099999989pt height=22.465723500000017pt/> the additive group of integers mod 64, also known as <img src="/tex/0d9ce7c28dd16a4a3368542c3ca86a10.svg?invert_in_darkmode&sanitize=true" align=middle width=46.57551029999998pt height=24.65753399999998pt/>. This is often chosen because it is a simple, well-known group. It seems promising at first, but you quickly run into problems when you discover that for almost all chessboard states <img src="/tex/374ff7af31c1eefcd9d7c9f484d61d43.svg?invert_in_darkmode&sanitize=true" align=middle width=40.12958564999999pt height=22.465723500000017pt/>, not every square <img src="/tex/2d8cca33f0ee74986943da285a93a659.svg?invert_in_darkmode&sanitize=true" align=middle width=38.82401819999999pt height=22.465723500000017pt/> is reachable with a single coin toggle.

The theoretical reason for this there are no valid nontrivial homomorphisms from the group <img src="/tex/a3e5d1ccf05bfb09b34a944361a4dcfa.svg?invert_in_darkmode&sanitize=true" align=middle width=53.28781424999998pt height=26.76175259999998pt/> to <img src="/tex/0d9ce7c28dd16a4a3368542c3ca86a10.svg?invert_in_darkmode&sanitize=true" align=middle width=46.57551029999998pt height=24.65753399999998pt/>, so we would not be able to construct <img src="/tex/190083ef7a1625fbc75f243cffb9c96d.svg?invert_in_darkmode&sanitize=true" align=middle width=9.81741584999999pt height=22.831056599999986pt/>.

The main insight is in realizing that <img src="/tex/9b325b9e31e85137d1de765f43c0f8bc.svg?invert_in_darkmode&sanitize=true" align=middle width=12.92464304999999pt height=22.465723500000017pt/> is *self-inverting*: <img src="/tex/17b1e895acf0f8e13f0ff6ca0d49cddf.svg?invert_in_darkmode&sanitize=true" align=middle width=126.85101659999998pt height=22.831056599999986pt/>. In other words, x XOR x is always zero. If we apply <img src="/tex/190083ef7a1625fbc75f243cffb9c96d.svg?invert_in_darkmode&sanitize=true" align=middle width=9.81741584999999pt height=22.831056599999986pt/>, then we realize that <img src="/tex/e257acd1ccbe7fcb654708f1a866bfe9.svg?invert_in_darkmode&sanitize=true" align=middle width=11.027402099999989pt height=22.465723500000017pt/> must also be self-inverting, or <p align="center"><img src="/tex/a216f6a14527bc196bee9cb1f1dff1c6.svg?invert_in_darkmode&sanitize=true" align=middle width=172.05670844999997pt height=16.438356pt/></p> or in other words <p align="center"><img src="/tex/86ce8846af16bf5f531b68504cd11a1e.svg?invert_in_darkmode&sanitize=true" align=middle width=126.72878789999999pt height=12.785402849999999pt/></p> or <p align="center"><img src="/tex/a06ed81cafc6f1f8c27d2733a9122134.svg?invert_in_darkmode&sanitize=true" align=middle width=111.76889129999999pt height=12.785402849999999pt/></p> 

<img src="/tex/0d9ce7c28dd16a4a3368542c3ca86a10.svg?invert_in_darkmode&sanitize=true" align=middle width=46.57551029999998pt height=24.65753399999998pt/> is not self-inverting. For example, this would require that <img src="/tex/3f85da2e02a67cb0022399fe72188984.svg?invert_in_darkmode&sanitize=true" align=middle width=51.14148269999999pt height=21.18721440000001pt/>, but in this group <img src="/tex/98706f43aab06e1c6a51059f936547cc.svg?invert_in_darkmode&sanitize=true" align=middle width=59.36069204999998pt height=21.18721440000001pt/>. 

Which group has 64 elements and is self-inverting? The group <img src="/tex/7446d816a12be7b02fbd14ddf09acf47.svg?invert_in_darkmode&sanitize=true" align=middle width=46.73526824999998pt height=26.76175259999998pt/>. In other words, <img src="/tex/e257acd1ccbe7fcb654708f1a866bfe9.svg?invert_in_darkmode&sanitize=true" align=middle width=11.027402099999989pt height=22.465723500000017pt/> is 6-dimensional vector space over the field <img src="/tex/842a3ba6459f9c7d0b7724742b431bc1.svg?invert_in_darkmode&sanitize=true" align=middle width=40.18272059999999pt height=24.65753399999998pt/>, also known as the group of 6-bit bitvectors under the XOR operator.

Recall that <p align="center"><img src="/tex/dd429c45ce74c72091aa2141e3e7946a.svg?invert_in_darkmode&sanitize=true" align=middle width=142.3897134pt height=18.312383099999998pt/></p> Using the fact that <img src="/tex/b5f95cbd811663d2f02cb72efc6d8fe0.svg?invert_in_darkmode&sanitize=true" align=middle width=50.11402274999998pt height=19.1781018pt/> we can simplify this to <p align="center"><img src="/tex/d964132a340ebc808c8b2ee0c59457d0.svg?invert_in_darkmode&sanitize=true" align=middle width=142.3897134pt height=18.312383099999998pt/></p>


