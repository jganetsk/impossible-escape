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

## The Short Explanation

Prisoner #1 sees board state <img src="svgs/18c0a8d8429a250dc2f045fa67126640.svg?invert_in_darkmode" align=middle width=13.666455pt height=14.15535pt/>. Then prisoner #1 computes <img src="svgs/1948750e43d256dbb893cb3234c97208.svg?invert_in_darkmode" align=middle width=53.949885pt height=19.17828pt/> and toggles a coin to make the board state <img src="svgs/671706266bd1ac19ca4c7552f0ab84fe.svg?invert_in_darkmode" align=middle width=102.13665pt height=24.6576pt/> which is equal to <img src="svgs/a427362f81f2b0712802d9f2af34017e.svg?invert_in_darkmode" align=middle width=19.37034pt height=14.15535pt/>.

## The Long Explanation

Personally, I find it insufficient to merely describe the solution. I need to describe the methodology used to arrive at it. Arriving at a good solution should not require neither guessing nor large logical leaps, but should rather be uncovered methodically as a small step in a reasoned argument.

### Set-up

From now on, we will encode tails as <img src="svgs/29632a9bf827ce0200454dd32fc3be82.svg?invert_in_darkmode" align=middle width=8.219277pt height=21.18732pt/> and heads as <img src="svgs/034d0a6be0424bffe9a6e7ac9236c0f5.svg?invert_in_darkmode" align=middle width=8.219277pt height=21.18732pt/>. The field <img src="svgs/842a3ba6459f9c7d0b7724742b431bc1.svg?invert_in_darkmode" align=middle width=40.18278pt height=24.6576pt/> will be commonly referenced below.

We are interested in understanding 2 sets:

- <img src="svgs/9b325b9e31e85137d1de765f43c0f8bc.svg?invert_in_darkmode" align=middle width=12.92478pt height=22.46574pt/>, the set of all possible chessboard states. Note <img src="svgs/e9c240a6f50a76c838e20dccd38baa35.svg?invert_in_darkmode" align=middle width=65.29908pt height=26.76201pt/>
- <img src="svgs/e257acd1ccbe7fcb654708f1a866bfe9.svg?invert_in_darkmode" align=middle width=11.027445pt height=22.46574pt/>, the set of all squares. Note <img src="svgs/727d3c9e2b77bcd0cb11822adb5a15af.svg?invert_in_darkmode" align=middle width=58.51593pt height=24.6576pt/>

We would like to define a function that the prisoners can use to encode a square with the chessboard state:

<p align="center"><img src="svgs/6beef70c90af8c131a8bc1d8e304254b.svg?invert_in_darkmode" align=middle width=73.038405pt height=14.611872pt/></p>

- Our objective is to convey a secret <img src="svgs/9d46dbf986fa639431e969f51d4d4322.svg?invert_in_darkmode" align=middle width=51.310875pt height=22.46574pt/>.
- Given some initial chessboard state <img src="svgs/87a45f06900f2b8979d64cee76c9d52a.svg?invert_in_darkmode" align=middle width=47.50416pt height=22.46574pt/> chosen by the jailer
- Prisoner #1 will want to change the chessboard state from <img src="svgs/18c0a8d8429a250dc2f045fa67126640.svg?invert_in_darkmode" align=middle width=13.666455pt height=14.15535pt/> to <img src="svgs/4ff28fe9f85fb1920ff4608be08af94d.svg?invert_in_darkmode" align=middle width=47.50416pt height=22.46574pt/> by toggling a single coin such that <img src="svgs/ea91a6e64281baeada5eb2450dfe7641.svg?invert_in_darkmode" align=middle width=78.379125pt height=24.6576pt/>
- Prisoner #2 will observe the chessboard state <img src="svgs/ce647d2c93bc38bf88d19d014430752e.svg?invert_in_darkmode" align=middle width=13.666455pt height=14.15535pt/> and compute <img src="svgs/b322d107b998b55dd97d7a267bb2b1fa.svg?invert_in_darkmode" align=middle width=37.091175pt height=24.6576pt/>, using that to guess <img src="svgs/a427362f81f2b0712802d9f2af34017e.svg?invert_in_darkmode" align=middle width=19.37034pt height=14.15535pt/>

Since we want to understand how the function <img src="svgs/190083ef7a1625fbc75f243cffb9c96d.svg?invert_in_darkmode" align=middle width=9.8175pt height=22.83138pt/> behaves with respect to changes in its input, we will need more algebraic structure. We need to make <img src="svgs/9b325b9e31e85137d1de765f43c0f8bc.svg?invert_in_darkmode" align=middle width=12.92478pt height=22.46574pt/>
and <img src="svgs/e257acd1ccbe7fcb654708f1a866bfe9.svg?invert_in_darkmode" align=middle width=11.027445pt height=22.46574pt/> groups, and make <img src="svgs/190083ef7a1625fbc75f243cffb9c96d.svg?invert_in_darkmode" align=middle width=9.8175pt height=22.83138pt/> a group homomorphism.

### Review of Group Theory

#### What is a Group?
- A group <img src="svgs/5201385589993766eea584cd3aa6fa13.svg?invert_in_darkmode" align=middle width=12.92478pt height=22.46574pt/> is the tuple <img src="svgs/68c91478b407d56f3e1870f1c19cc33f.svg?invert_in_darkmode" align=middle width=57.95097pt height=24.6576pt/>.
- It is a set <img src="svgs/e257acd1ccbe7fcb654708f1a866bfe9.svg?invert_in_darkmode" align=middle width=11.027445pt height=22.46574pt/> with an associative binary operator <img src="svgs/df33724455416439909c33a7db76b2bc.svg?invert_in_darkmode" align=middle width=12.78552pt height=19.17828pt/>
- The set contains an identity element <img src="svgs/24f21c4f924a49dc5f3d6cd7b1cb4e89.svg?invert_in_darkmode" align=middle width=38.77269pt height=22.46574pt/> such that <img src="svgs/4fef05c55c708a7fa54121562a8332a3.svg?invert_in_darkmode" align=middle width=126.72891pt height=22.83138pt/>
- Each element <img src="svgs/2d8cca33f0ee74986943da285a93a659.svg?invert_in_darkmode" align=middle width=38.824005pt height=22.46574pt/> has an inverse element <img src="svgs/a3d9b1b249a83ed17fc41dae0cd03536.svg?invert_in_darkmode" align=middle width=51.609525pt height=22.46574pt/> such that <img src="svgs/994bf83f590b888995ca453eb1ef6a8a.svg?invert_in_darkmode" align=middle width=152.299785pt height=24.6576pt/>
- The <img src="svgs/df33724455416439909c33a7db76b2bc.svg?invert_in_darkmode" align=middle width=12.78552pt height=19.17828pt/> operator may or may not be commutative (when it is commutative, we call the group "Abelian")

#### Additional Definitions
- A vector space of <img src="svgs/e293fa0f67d04b8f8108558c7ce4bde1.svg?invert_in_darkmode" align=middle width=20.98008pt height=22.46574pt/> is a group consisting of vectors with <img src="svgs/55a049b8f161ae7cfeb0197d75aff967.svg?invert_in_darkmode" align=middle width=9.867pt height=14.15535pt/>-dimensions over the field <img src="svgs/b8bc815b5e9d5177af01fd4d3d3c2f10.svg?invert_in_darkmode" align=middle width=12.853995pt height=22.46574pt/>.
- Every vector space <img src="svgs/e293fa0f67d04b8f8108558c7ce4bde1.svg?invert_in_darkmode" align=middle width=20.98008pt height=22.46574pt/> has a basis: a set of <img src="svgs/55a049b8f161ae7cfeb0197d75aff967.svg?invert_in_darkmode" align=middle width=9.867pt height=14.15535pt/> elements <img src="svgs/8b9af7f0c3dd172a35e9b89542c68313.svg?invert_in_darkmode" align=middle width=183.243555pt height=24.6576pt/> such that each element of the vector space can be expressed as a unique sum of basis elements: <img src="svgs/4e0c5913c22eb981f798b86b6327114e.svg?invert_in_darkmode" align=middle width=261.453555pt height=51.00447pt/>
- A standard basis is a basis of a vector space where each basis element is a vector of the form <img src="svgs/ab7126e4ad76a713043e3acadc024961.svg?invert_in_darkmode" align=middle width=76.69167pt height=24.6576pt/> or <img src="svgs/7ea33751c9927247cb3cdaa34e6e3db2.svg?invert_in_darkmode" align=middle width=76.69167pt height=24.6576pt/>, etc
- A group homomorphism is a function <img src="svgs/396ebc256c33dd601356a3467916d899.svg?invert_in_darkmode" align=middle width=74.86512pt height=22.83138pt/> from group <img src="svgs/b8bc815b5e9d5177af01fd4d3d3c2f10.svg?invert_in_darkmode" align=middle width=12.853995pt height=22.46574pt/> to group <img src="svgs/5201385589993766eea584cd3aa6fa13.svg?invert_in_darkmode" align=middle width=12.92478pt height=22.46574pt/> such that <img src="svgs/0ff4d02bb1960f671388dfe24832f757.svg?invert_in_darkmode" align=middle width=240.223005pt height=24.6576pt/>
- As corollaries to the above, for any group homomorphism <img src="svgs/190083ef7a1625fbc75f243cffb9c96d.svg?invert_in_darkmode" align=middle width=9.8175pt height=22.83138pt/>, the following are true: <img src="svgs/2c43d9f3a62f8128b9c6bb7789635316.svg?invert_in_darkmode" align=middle width=80.991405pt height=24.6576pt/> and <img src="svgs/9cd13c7a6ba13e3cf1c4fa8efca4fd6e.svg?invert_in_darkmode" align=middle width=174.538155pt height=24.6576pt/>
- The image of a set <img src="svgs/9b325b9e31e85137d1de765f43c0f8bc.svg?invert_in_darkmode" align=middle width=12.92478pt height=22.46574pt/> under <img src="svgs/190083ef7a1625fbc75f243cffb9c96d.svg?invert_in_darkmode" align=middle width=9.8175pt height=22.83138pt/> is <img src="svgs/2e140ce473466495ad522e51466335b6.svg?invert_in_darkmode" align=middle width=153.775215pt height=24.6576pt/>
- The preimage or inverse image of a set <img src="svgs/e257acd1ccbe7fcb654708f1a866bfe9.svg?invert_in_darkmode" align=middle width=11.027445pt height=22.46574pt/> under <img src="svgs/190083ef7a1625fbc75f243cffb9c96d.svg?invert_in_darkmode" align=middle width=9.8175pt height=22.83138pt/> is <img src="svgs/549ed9332e691a07b360f1de8c200533.svg?invert_in_darkmode" align=middle width=200.644455pt height=26.76201pt/>

### Applying Group Theory to the Problem

In order to make <img src="svgs/9b325b9e31e85137d1de765f43c0f8bc.svg?invert_in_darkmode" align=middle width=12.92478pt height=22.46574pt/> a group, we need to be able to answer the question *what does it mean to add two chessboard states together?* We can answer this by reinterpreting <img src="svgs/9b325b9e31e85137d1de765f43c0f8bc.svg?invert_in_darkmode" align=middle width=12.92478pt height=22.46574pt/> to be a set of chessboard state *deltas*. Note, this is a formality. There is essentially no difference between a group of states and a group of state deltas, except the latter motivates the intuition of what it means to add two group elements together. The identify element <img src="svgs/b8554c00e366606e9293d43bcd472ed8.svg?invert_in_darkmode" align=middle width=40.670025pt height=22.46574pt/> could be interpreted as the zero delta (when interpreting as chessboard state deltas) or as the state of all 64 coins being tails-side up (when interpreting as chessboard states). Conceptually, prisoner #1 must choose an element from a set of 64 chessboard state deltas, which is a subset of <img src="svgs/9b325b9e31e85137d1de765f43c0f8bc.svg?invert_in_darkmode" align=middle width=12.92478pt height=22.46574pt/>.

We will choose to make <img src="svgs/9b325b9e31e85137d1de765f43c0f8bc.svg?invert_in_darkmode" align=middle width=12.92478pt height=22.46574pt/> the vector space <img src="svgs/a3e5d1ccf05bfb09b34a944361a4dcfa.svg?invert_in_darkmode" align=middle width=53.287905pt height=26.76201pt/>. In other words, <img src="svgs/9b325b9e31e85137d1de765f43c0f8bc.svg?invert_in_darkmode" align=middle width=12.92478pt height=22.46574pt/> is the group of 64-bit bitvectors under the XOR operator. Incidentally, these groups are also Abelian, in case we happen to use commutativity below.

- Note the standard basis <img src="svgs/e501c3fbaf8b825ac927568e9ecd1beb.svg?invert_in_darkmode" align=middle width=48.13578pt height=22.46574pt/> is the set of valid moves that prisoner #1 can make, since each standard basis element is a delta that toggles exactly one coin.
- Since the field of the vector space is <img src="svgs/842a3ba6459f9c7d0b7724742b431bc1.svg?invert_in_darkmode" align=middle width=40.18278pt height=24.6576pt/>, we can simplify the definition of "basis" by removing the scalar multiplication: <img src="svgs/ddf99885c877e14e906750895a2c952e.svg?invert_in_darkmode" align=middle width=164.387355pt height=38.79018pt/> for some subset of basis elements. The intuition captured here is that every chessboard state can be arrived at with a series of valid moves (aka single coin toggles).

- Let <img src="svgs/163af9574bd73b91a67e73d0a25c7ad6.svg?invert_in_darkmode" align=middle width=54.197055pt height=22.46574pt/> be the move that prisoner #1 will make. Therefore, <img src="svgs/06e008cdad753b62dc068ef78d610d13.svg?invert_in_darkmode" align=middle width=91.797915pt height=22.46574pt/>
- Let <img src="svgs/190083ef7a1625fbc75f243cffb9c96d.svg?invert_in_darkmode" align=middle width=9.8175pt height=22.83138pt/> be a group homomorphism from <img src="svgs/87339665b58a744eea2d393d1988a61a.svg?invert_in_darkmode" align=middle width=73.03857pt height=22.83138pt/>
- We know that <img src="svgs/e257acd1ccbe7fcb654708f1a866bfe9.svg?invert_in_darkmode" align=middle width=11.027445pt height=22.46574pt/> is a set with 64 elements, but we have not yet decided how to make it into a group. That will come later.

Let's now solve for <img src="svgs/da6e5b842edb0a8ada1035cc77dd710f.svg?invert_in_darkmode" align=middle width=20.812605pt height=22.46574pt/>, the move prisoner #1 should make:

<p align="center"><img src="svgs/6b03f0cec4192f2fbee18c374770fe96.svg?invert_in_darkmode" align=middle width=78.379125pt height=16.438356pt/></p>

<p align="center"><img src="svgs/147dfe743e453dd6d9eb06b3c6dd2392.svg?invert_in_darkmode" align=middle width=119.28279pt height=16.438356pt/></p>

<p align="center"><img src="svgs/8d05933df29d6be39a199726f9844ec4.svg?invert_in_darkmode" align=middle width=141.885645pt height=16.438356pt/></p>

<p align="center"><img src="svgs/84551944d62d7c6c137a79d13abea3a0.svg?invert_in_darkmode" align=middle width=142.70751pt height=16.438356pt/></p>

We will now introduce the usage of the inverse image, <img src="svgs/8fac22af20becca270bc855e487906cb.svg?invert_in_darkmode" align=middle width=26.644035pt height=26.76201pt/>

<p align="center"><img src="svgs/0c2522212dfa8ffa37a2fb6a680506c9.svg?invert_in_darkmode" align=middle width=203.41695pt height=18.31236pt/></p>

Therefore:

<p align="center"><img src="svgs/dd429c45ce74c72091aa2141e3e7946a.svg?invert_in_darkmode" align=middle width=142.38972pt height=18.31236pt/></p>

The set on the right-hand side contains all deltas that can be applied to <img src="svgs/18c0a8d8429a250dc2f045fa67126640.svg?invert_in_darkmode" align=middle width=13.666455pt height=14.15535pt/> to make <img src="svgs/7e0b0f84da75b2556c33a223c49083e0.svg?invert_in_darkmode" align=middle width=66.714285pt height=24.6576pt/>. We need to guarantee that this set contains at least one legal move. In other words, it should contain at least one element <img src="svgs/4bdc8d9bcfb35e1c9bfb51fc69687dfc.svg?invert_in_darkmode" align=middle width=7.0548555pt height=22.83138pt/> in the standard basis <img src="svgs/760318998b5f14fb225b3862a8ff1901.svg?invert_in_darkmode" align=middle width=40.439355pt height=22.83138pt/>.

To do this, we must guarantee that each element <img src="svgs/2d8cca33f0ee74986943da285a93a659.svg?invert_in_darkmode" align=middle width=38.824005pt height=22.46574pt/> must have at least one standard basis element <img src="svgs/760318998b5f14fb225b3862a8ff1901.svg?invert_in_darkmode" align=middle width=40.439355pt height=22.83138pt/> such that <img src="svgs/60b2610f4905c8c159e74ff3539bb734.svg?invert_in_darkmode" align=middle width=59.28087pt height=24.6576pt/>. In other words <img src="svgs/71cb5ef0976705bb1329f4431a68cb9b.svg?invert_in_darkmode" align=middle width=65.188365pt height=24.6576pt/>. This is simple to guarantee. There are 64 elements in <img src="svgs/61e84f854bc6258d4108d08d4c4a0852.svg?invert_in_darkmode" align=middle width=13.293555pt height=22.46574pt/> (there are 64 valid moves prisoner #1 can make) and there are 64 elements in <img src="svgs/e257acd1ccbe7fcb654708f1a866bfe9.svg?invert_in_darkmode" align=middle width=11.027445pt height=22.46574pt/> (there are 64 squares on the chessboard). This is made even easier by the fact that every standard basis element toggles exactly one coin, so it has already "picked a square". We can then construct a straightforward one-to-one mapping between the two sets <img src="svgs/61e84f854bc6258d4108d08d4c4a0852.svg?invert_in_darkmode" align=middle width=13.293555pt height=22.46574pt/> and <img src="svgs/e257acd1ccbe7fcb654708f1a866bfe9.svg?invert_in_darkmode" align=middle width=11.027445pt height=22.46574pt/>. 

There are two main questions that remain:

1. We have defined the values of <img src="svgs/bc47eb54c896114e803bd8d0e087caf4.svg?invert_in_darkmode" align=middle width=29.65776pt height=24.6576pt/> where <img src="svgs/760318998b5f14fb225b3862a8ff1901.svg?invert_in_darkmode" align=middle width=40.439355pt height=22.83138pt/>. We must now define the remaining values of <img src="svgs/01b5d07b6b39ee917e3eb6119953d672.svg?invert_in_darkmode" align=middle width=29.716665pt height=24.6576pt/> where <img src="svgs/374ff7af31c1eefcd9d7c9f484d61d43.svg?invert_in_darkmode" align=middle width=40.12965pt height=22.46574pt/>.
1. We must still need to make <img src="svgs/e257acd1ccbe7fcb654708f1a866bfe9.svg?invert_in_darkmode" align=middle width=11.027445pt height=22.46574pt/> into a group. Without this, we cannot compute <img src="svgs/0ec35fad8786a167574244d347967252.svg?invert_in_darkmode" align=middle width=64.8879pt height=24.6576pt/>, for we have not defined the <img src="svgs/df33724455416439909c33a7db76b2bc.svg?invert_in_darkmode" align=middle width=12.78552pt height=19.17828pt/> operator yet.

And the answers:

1. This is straightforward. Each element <img src="svgs/374ff7af31c1eefcd9d7c9f484d61d43.svg?invert_in_darkmode" align=middle width=40.12965pt height=22.46574pt/> can be expressed as a sum of basis elements <img src="svgs/d2b3d983e09b253946f6fe3c72098eda.svg?invert_in_darkmode" align=middle width=45.91224pt height=22.83138pt/>, so <p align="center"><img src="svgs/7223690b61ae166c528c67916071d7e3.svg?invert_in_darkmode" align=middle width=67.221165pt height=36.65541pt/></p> therefore <p align="center"><img src="svgs/a8112cdbf2fc8e87d688a8b8c998f32e.svg?invert_in_darkmode" align=middle width=113.24874pt height=36.65541pt/></p> and finally <p align="center"><img src="svgs/3c93abebd3b56d1fbd29ce4f417a8734.svg?invert_in_darkmode" align=middle width=113.24874pt height=36.65541pt/></p> And we already have definitions of <img src="svgs/bc47eb54c896114e803bd8d0e087caf4.svg?invert_in_darkmode" align=middle width=29.65776pt height=24.6576pt/> for all <img src="svgs/760318998b5f14fb225b3862a8ff1901.svg?invert_in_darkmode" align=middle width=40.439355pt height=22.83138pt/>, so we are done here.
1. Now comes the interesting part. What do use as a group for <img src="svgs/e257acd1ccbe7fcb654708f1a866bfe9.svg?invert_in_darkmode" align=middle width=11.027445pt height=22.46574pt/>?

#### Choosing a group for <img src="svgs/e257acd1ccbe7fcb654708f1a866bfe9.svg?invert_in_darkmode" align=middle width=11.027445pt height=22.46574pt/>

Much of what we've discussed above is mere formalism. Now we get to the core of the problem: choosing a group for <img src="svgs/e257acd1ccbe7fcb654708f1a866bfe9.svg?invert_in_darkmode" align=middle width=11.027445pt height=22.46574pt/>.

The first intuition many seem to have when trying to solve this problem is to make <img src="svgs/e257acd1ccbe7fcb654708f1a866bfe9.svg?invert_in_darkmode" align=middle width=11.027445pt height=22.46574pt/> the additive group of integers mod 64, also known as <img src="svgs/0d9ce7c28dd16a4a3368542c3ca86a10.svg?invert_in_darkmode" align=middle width=46.57554pt height=24.6576pt/>. This is often chosen because it is a simple, well-known group. It seems promising at first, but you quickly run into problems when you discover that for almost all chessboard states <img src="svgs/374ff7af31c1eefcd9d7c9f484d61d43.svg?invert_in_darkmode" align=middle width=40.12965pt height=22.46574pt/>, not every square <img src="svgs/2d8cca33f0ee74986943da285a93a659.svg?invert_in_darkmode" align=middle width=38.824005pt height=22.46574pt/> is reachable with a single coin toggle.

The theoretical reason for this there are no valid nontrivial homomorphisms from the group <img src="svgs/a3e5d1ccf05bfb09b34a944361a4dcfa.svg?invert_in_darkmode" align=middle width=53.287905pt height=26.76201pt/> to <img src="svgs/0d9ce7c28dd16a4a3368542c3ca86a10.svg?invert_in_darkmode" align=middle width=46.57554pt height=24.6576pt/>, so we would not be able to construct <img src="svgs/190083ef7a1625fbc75f243cffb9c96d.svg?invert_in_darkmode" align=middle width=9.8175pt height=22.83138pt/>.

The main insight is in realizing that <img src="svgs/9b325b9e31e85137d1de765f43c0f8bc.svg?invert_in_darkmode" align=middle width=12.92478pt height=22.46574pt/> is *self-inverting*: <img src="svgs/17b1e895acf0f8e13f0ff6ca0d49cddf.svg?invert_in_darkmode" align=middle width=126.85101pt height=22.83138pt/>. In other words, x XOR x is always zero. If we apply <img src="svgs/190083ef7a1625fbc75f243cffb9c96d.svg?invert_in_darkmode" align=middle width=9.8175pt height=22.83138pt/>, then we see, realize that <img src="svgs/e257acd1ccbe7fcb654708f1a866bfe9.svg?invert_in_darkmode" align=middle width=11.027445pt height=22.46574pt/> must also be self-inverting:

<p align="center"><img src="svgs/a3badf8efdfa5afe3063a231fe2ff9ae.svg?invert_in_darkmode" align=middle width=172.05705pt height=16.438356pt/></p>

<p align="center"><img src="svgs/a216f6a14527bc196bee9cb1f1dff1c6.svg?invert_in_darkmode" align=middle width=172.05705pt height=16.438356pt/></p>

Since <img src="svgs/66c55ed40bd1f9f82c48c22b5665e5b4.svg?invert_in_darkmode" align=middle width=64.81959pt height=24.6576pt/>:

<p align="center"><img src="svgs/86ce8846af16bf5f531b68504cd11a1e.svg?invert_in_darkmode" align=middle width=126.728745pt height=12.7854045pt/></p>

<p align="center"><img src="svgs/f4992aa1b9b3699ae2ce779847e2d12a.svg?invert_in_darkmode" align=middle width=103.486515pt height=14.611872pt/></p>

<img src="svgs/0d9ce7c28dd16a4a3368542c3ca86a10.svg?invert_in_darkmode" align=middle width=46.57554pt height=24.6576pt/> is not self-inverting. For example, this would require that <img src="svgs/3f85da2e02a67cb0022399fe72188984.svg?invert_in_darkmode" align=middle width=51.141585pt height=21.18732pt/>, but in this group <img src="svgs/98706f43aab06e1c6a51059f936547cc.svg?invert_in_darkmode" align=middle width=59.36073pt height=21.18732pt/>. 

Which group has 64 elements and is self-inverting? The group <img src="svgs/7446d816a12be7b02fbd14ddf09acf47.svg?invert_in_darkmode" align=middle width=46.73526pt height=26.76201pt/>. In other words, <img src="svgs/e257acd1ccbe7fcb654708f1a866bfe9.svg?invert_in_darkmode" align=middle width=11.027445pt height=22.46574pt/> is 6-dimensional vector space over the field <img src="svgs/842a3ba6459f9c7d0b7724742b431bc1.svg?invert_in_darkmode" align=middle width=40.18278pt height=24.6576pt/>, also known as the group of 6-bit bitvectors under the XOR operator.

Recall:

<p align="center"><img src="svgs/33570e8d856faa4ead987d0c1be4e8de.svg?invert_in_darkmode" align=middle width=154.876425pt height=18.31236pt/></p>

<p align="center"><img src="svgs/f4992aa1b9b3699ae2ce779847e2d12a.svg?invert_in_darkmode" align=middle width=103.486515pt height=14.611872pt/></p>

We can make the simplification:

<p align="center"><img src="svgs/1c4614f2381217321b581ffe3365a0a4.svg?invert_in_darkmode" align=middle width=154.876425pt height=18.31236pt/></p>


