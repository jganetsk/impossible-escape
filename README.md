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

Prisoner #1 sees board state <img src="svgs/18c0a8d8429a250dc2f045fa67126640.svg?invert_in_darkmode" align=middle width=13.666455pt height=14.15535pt/>. Then, prisoner #1 computes <img src="svgs/1948750e43d256dbb893cb3234c97208.svg?invert_in_darkmode" align=middle width=53.949885pt height=19.17828pt/> and toggles that coin to make the board state <img src="svgs/671706266bd1ac19ca4c7552f0ab84fe.svg?invert_in_darkmode" align=middle width=102.13665pt height=24.6576pt/> which is equal to <img src="svgs/a427362f81f2b0712802d9f2af34017e.svg?invert_in_darkmode" align=middle width=19.37034pt height=14.15535pt/>.

## The Long Explanation

> Feynman's Algorithm:
>
> 1. Write down the problem.
> 2. Think very hard.
> 3. Write down the solution.

I find it insufficient to merely describe the solution, I am motivated to describe the methodology used to arrive at it. The point of algebra (both in its elementary form and its various more abstract forms) is to provide a means to express the constraints of a problem such that the solution flows natrually from the constraints. Instead of anticipating a logical leap to an answer, you should continue to refine the expression of constraints until the answer has no choice but to reveal itself to you.

### Set-up

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

### Applying Group Theory to the Problem

I have included a [review of group theory in the appendix](#review-of-group-theory).

In order to make <img src="svgs/9b325b9e31e85137d1de765f43c0f8bc.svg?invert_in_darkmode" align=middle width=12.92478pt height=22.46574pt/> a group, we need to be able to answer the question *what does it mean to add two chessboard states together?* We can answer this by reinterpreting <img src="svgs/9b325b9e31e85137d1de765f43c0f8bc.svg?invert_in_darkmode" align=middle width=12.92478pt height=22.46574pt/> to be a set of chessboard state *deltas*. Note, this is a formality. There is essentially no difference between a group of states and a group of state deltas, except the latter motivates the intuition of what it means to add two group elements together. 

We will choose to make <img src="svgs/9b325b9e31e85137d1de765f43c0f8bc.svg?invert_in_darkmode" align=middle width=12.92478pt height=22.46574pt/> the vector space <img src="svgs/a3e5d1ccf05bfb09b34a944361a4dcfa.svg?invert_in_darkmode" align=middle width=53.287905pt height=26.76201pt/>, alternately represented as the set of all 64-bit bitvectors under the XOR operator. To review, in the field <img src="svgs/842a3ba6459f9c7d0b7724742b431bc1.svg?invert_in_darkmode" align=middle width=40.18278pt height=24.6576pt/>, the following identities are true:

<p align="center"><img src="svgs/914d951fd808f6ac4d9224a71917dfab.svg?invert_in_darkmode" align=middle width=66.666435pt height=11.9634735pt/></p>

<p align="center"><img src="svgs/d251a66c1c2cafb3f5a10636b44adf29.svg?invert_in_darkmode" align=middle width=66.666435pt height=11.9634735pt/></p>

<p align="center"><img src="svgs/b74195ef05c1c949fa7d0c319e95b0ec.svg?invert_in_darkmode" align=middle width=66.666435pt height=11.9634735pt/></p>

<p align="center"><img src="svgs/be7a276c453c7754a3b248e3c76026b0.svg?invert_in_darkmode" align=middle width=66.666435pt height=11.9634735pt/></p>

The meaning of <img src="svgs/29632a9bf827ce0200454dd32fc3be82.svg?invert_in_darkmode" align=middle width=8.219277pt height=21.18732pt/> vs <img src="svgs/034d0a6be0424bffe9a6e7ac9236c0f5.svg?invert_in_darkmode" align=middle width=8.219277pt height=21.18732pt/> depends on whether we are interpreting the group as chessboard states or chessboard state deltas:

|Interpretation of <img src="svgs/9b325b9e31e85137d1de765f43c0f8bc.svg?invert_in_darkmode" align=middle width=12.92478pt height=22.46574pt/>|Meaning of <img src="svgs/29632a9bf827ce0200454dd32fc3be82.svg?invert_in_darkmode" align=middle width=8.219277pt height=21.18732pt/>|Meaning of <img src="svgs/034d0a6be0424bffe9a6e7ac9236c0f5.svg?invert_in_darkmode" align=middle width=8.219277pt height=21.18732pt/>|
|-|-|-|
|Chessboard states|Tails|Heads|
|Chessboard state deltas|Do not toggle the coin|Toggle the coin|

- The group above happens to be Abelian. This means addition is commutative, or <img src="svgs/09db8256a22e8ce6c8ff1007bf2f00ed.svg?invert_in_darkmode" align=middle width=98.18853pt height=19.17828pt/>
- Note the standard basis <img src="svgs/e501c3fbaf8b825ac927568e9ecd1beb.svg?invert_in_darkmode" align=middle width=48.13578pt height=22.46574pt/>, consisting of vectors such as <img src="svgs/ab7126e4ad76a713043e3acadc024961.svg?invert_in_darkmode" align=middle width=76.69167pt height=24.6576pt/>, is the set of valid moves that prisoner #1 can make, since each standard basis element is a delta that toggles exactly one coin.
- Since the field of the vector space is <img src="svgs/842a3ba6459f9c7d0b7724742b431bc1.svg?invert_in_darkmode" align=middle width=40.18278pt height=24.6576pt/>, we can simplify the definition of "basis" by removing the scalar multiplication: <img src="svgs/ddf99885c877e14e906750895a2c952e.svg?invert_in_darkmode" align=middle width=164.387355pt height=38.79018pt/> for some subset of basis elements. The intuition captured here is that every chessboard state can be arrived at by starting with an all-tails board and proceeding with a series of valid moves (aka single coin toggles).

- Let <img src="svgs/163af9574bd73b91a67e73d0a25c7ad6.svg?invert_in_darkmode" align=middle width=54.197055pt height=22.46574pt/> be the move that prisoner #1 will make. Therefore, <img src="svgs/06e008cdad753b62dc068ef78d610d13.svg?invert_in_darkmode" align=middle width=91.797915pt height=22.46574pt/>
- Let <img src="svgs/190083ef7a1625fbc75f243cffb9c96d.svg?invert_in_darkmode" align=middle width=9.8175pt height=22.83138pt/> be a group homomorphism from <img src="svgs/87339665b58a744eea2d393d1988a61a.svg?invert_in_darkmode" align=middle width=73.03857pt height=22.83138pt/>. In particular, we want to guarantee that <img src="svgs/190083ef7a1625fbc75f243cffb9c96d.svg?invert_in_darkmode" align=middle width=9.8175pt height=22.83138pt/> is surjective, aka "onto", aka <img src="svgs/66c55ed40bd1f9f82c48c22b5665e5b4.svg?invert_in_darkmode" align=middle width=64.81959pt height=24.6576pt/>. Otherwise, we would not be able to encode all possible magic squares.
- We know that <img src="svgs/e257acd1ccbe7fcb654708f1a866bfe9.svg?invert_in_darkmode" align=middle width=11.027445pt height=22.46574pt/> is a set with 64 elements, but we have not yet decided how to make it into a group. That will come later.

Let's now solve for <img src="svgs/da6e5b842edb0a8ada1035cc77dd710f.svg?invert_in_darkmode" align=middle width=20.812605pt height=22.46574pt/>, the move prisoner #1 should make:

<p align="center"><img src="svgs/6b03f0cec4192f2fbee18c374770fe96.svg?invert_in_darkmode" align=middle width=78.379125pt height=16.438356pt/></p>

<p align="center"><img src="svgs/147dfe743e453dd6d9eb06b3c6dd2392.svg?invert_in_darkmode" align=middle width=119.28279pt height=16.438356pt/></p>

<p align="center"><img src="svgs/8d05933df29d6be39a199726f9844ec4.svg?invert_in_darkmode" align=middle width=141.885645pt height=16.438356pt/></p>

<p align="center"><img src="svgs/84551944d62d7c6c137a79d13abea3a0.svg?invert_in_darkmode" align=middle width=142.70751pt height=16.438356pt/></p>

We will now introduce the usage of the inverse image, <img src="svgs/8fac22af20becca270bc855e487906cb.svg?invert_in_darkmode" align=middle width=26.644035pt height=26.76201pt/>

<p align="center"><img src="svgs/9022ce9f2fa0117e8284a6edaa3da604.svg?invert_in_darkmode" align=middle width=215.90415pt height=18.31236pt/></p>

Therefore:

<p align="center"><img src="svgs/33570e8d856faa4ead987d0c1be4e8de.svg?invert_in_darkmode" align=middle width=154.876425pt height=18.31236pt/></p>

The set on the right-hand side contains all deltas that can be applied to <img src="svgs/18c0a8d8429a250dc2f045fa67126640.svg?invert_in_darkmode" align=middle width=13.666455pt height=14.15535pt/> to make <img src="svgs/ea91a6e64281baeada5eb2450dfe7641.svg?invert_in_darkmode" align=middle width=78.379125pt height=24.6576pt/>. We need to guarantee that this set contains at least one legal move. 

To do this, we must guarantee that each element <img src="svgs/2d8cca33f0ee74986943da285a93a659.svg?invert_in_darkmode" align=middle width=38.824005pt height=22.46574pt/> must have at least one standard basis element <img src="svgs/760318998b5f14fb225b3862a8ff1901.svg?invert_in_darkmode" align=middle width=40.439355pt height=22.83138pt/> such that <img src="svgs/60b2610f4905c8c159e74ff3539bb734.svg?invert_in_darkmode" align=middle width=59.28087pt height=24.6576pt/>. In other words <img src="svgs/71cb5ef0976705bb1329f4431a68cb9b.svg?invert_in_darkmode" align=middle width=65.188365pt height=24.6576pt/>. This is straightforward. There are 64 elements in <img src="svgs/61e84f854bc6258d4108d08d4c4a0852.svg?invert_in_darkmode" align=middle width=13.293555pt height=22.46574pt/> (there are 64 possible valid moves prisoner #1 can make) and there are 64 elements in <img src="svgs/e257acd1ccbe7fcb654708f1a866bfe9.svg?invert_in_darkmode" align=middle width=11.027445pt height=22.46574pt/> (there are 64 squares on the chessboard). This is made even easier by the fact that every standard basis element toggles exactly one coin, so it has already "picked a square". We can then construct a straightforward one-to-one mapping between the two sets <img src="svgs/61e84f854bc6258d4108d08d4c4a0852.svg?invert_in_darkmode" align=middle width=13.293555pt height=22.46574pt/> and <img src="svgs/e257acd1ccbe7fcb654708f1a866bfe9.svg?invert_in_darkmode" align=middle width=11.027445pt height=22.46574pt/>. 

There are two main questions that remain:

1. We have defined the values of <img src="svgs/bc47eb54c896114e803bd8d0e087caf4.svg?invert_in_darkmode" align=middle width=29.65776pt height=24.6576pt/> where <img src="svgs/760318998b5f14fb225b3862a8ff1901.svg?invert_in_darkmode" align=middle width=40.439355pt height=22.83138pt/>. We must now define the remaining values of <img src="svgs/01b5d07b6b39ee917e3eb6119953d672.svg?invert_in_darkmode" align=middle width=29.716665pt height=24.6576pt/> where <img src="svgs/374ff7af31c1eefcd9d7c9f484d61d43.svg?invert_in_darkmode" align=middle width=40.12965pt height=22.46574pt/>.
1. We still need to make <img src="svgs/e257acd1ccbe7fcb654708f1a866bfe9.svg?invert_in_darkmode" align=middle width=11.027445pt height=22.46574pt/> into a group. Without this, we cannot compute <img src="svgs/7a0eaaa44412e500c7d1d75eb7d0df1b.svg?invert_in_darkmode" align=middle width=77.374605pt height=24.6576pt/>, for we have not defined the <img src="svgs/df33724455416439909c33a7db76b2bc.svg?invert_in_darkmode" align=middle width=12.78552pt height=19.17828pt/> operator yet.

And the answers:

1. Each element <img src="svgs/374ff7af31c1eefcd9d7c9f484d61d43.svg?invert_in_darkmode" align=middle width=40.12965pt height=22.46574pt/> can be expressed as a sum of basis elements <img src="svgs/d2b3d983e09b253946f6fe3c72098eda.svg?invert_in_darkmode" align=middle width=45.91224pt height=22.83138pt/>, so <p align="center"><img src="svgs/7223690b61ae166c528c67916071d7e3.svg?invert_in_darkmode" align=middle width=67.221165pt height=36.65541pt/></p> <p align="center"><img src="svgs/a8112cdbf2fc8e87d688a8b8c998f32e.svg?invert_in_darkmode" align=middle width=113.24874pt height=36.65541pt/></p> <p align="center"><img src="svgs/3c93abebd3b56d1fbd29ce4f417a8734.svg?invert_in_darkmode" align=middle width=113.24874pt height=36.65541pt/></p> And we already have definitions of <img src="svgs/bc47eb54c896114e803bd8d0e087caf4.svg?invert_in_darkmode" align=middle width=29.65776pt height=24.6576pt/> for all <img src="svgs/760318998b5f14fb225b3862a8ff1901.svg?invert_in_darkmode" align=middle width=40.439355pt height=22.83138pt/>, so we are done here.
1. Now comes the interesting part. What do use as a group for <img src="svgs/e257acd1ccbe7fcb654708f1a866bfe9.svg?invert_in_darkmode" align=middle width=11.027445pt height=22.46574pt/>?

#### Choosing a group for <img src="svgs/e257acd1ccbe7fcb654708f1a866bfe9.svg?invert_in_darkmode" align=middle width=11.027445pt height=22.46574pt/>

Much of what we've discussed above is mere formalism. Now we get to the core of the problem: choosing a group for <img src="svgs/e257acd1ccbe7fcb654708f1a866bfe9.svg?invert_in_darkmode" align=middle width=11.027445pt height=22.46574pt/>.

The first intuition many seem to have when trying to solve this problem is to make <img src="svgs/e257acd1ccbe7fcb654708f1a866bfe9.svg?invert_in_darkmode" align=middle width=11.027445pt height=22.46574pt/> the additive group of integers mod 64, also known as <img src="svgs/0d9ce7c28dd16a4a3368542c3ca86a10.svg?invert_in_darkmode" align=middle width=46.57554pt height=24.6576pt/>. This is often chosen because it is a simple, well-known group. It seems promising at first, but you quickly run into problems when you discover that for almost all chessboard states <img src="svgs/374ff7af31c1eefcd9d7c9f484d61d43.svg?invert_in_darkmode" align=middle width=40.12965pt height=22.46574pt/>, not every square <img src="svgs/2d8cca33f0ee74986943da285a93a659.svg?invert_in_darkmode" align=middle width=38.824005pt height=22.46574pt/> is reachable with a single coin toggle.

The main insight is in realizing that <img src="svgs/9b325b9e31e85137d1de765f43c0f8bc.svg?invert_in_darkmode" align=middle width=12.92478pt height=22.46574pt/> is *self-inverting*: <img src="svgs/17b1e895acf0f8e13f0ff6ca0d49cddf.svg?invert_in_darkmode" align=middle width=126.85101pt height=22.83138pt/>. In other words, x XOR x is always zero. When we apply <img src="svgs/190083ef7a1625fbc75f243cffb9c96d.svg?invert_in_darkmode" align=middle width=9.8175pt height=22.83138pt/>, we see realize that <img src="svgs/e257acd1ccbe7fcb654708f1a866bfe9.svg?invert_in_darkmode" align=middle width=11.027445pt height=22.46574pt/> must also be self-inverting:

<p align="center"><img src="svgs/cc3be83ab2dbafae183ef4d508302b5b.svg?invert_in_darkmode" align=middle width=126.85101pt height=12.7854045pt/></p>

<p align="center"><img src="svgs/a3badf8efdfa5afe3063a231fe2ff9ae.svg?invert_in_darkmode" align=middle width=172.05705pt height=16.438356pt/></p>

<p align="center"><img src="svgs/a216f6a14527bc196bee9cb1f1dff1c6.svg?invert_in_darkmode" align=middle width=172.05705pt height=16.438356pt/></p>

Since <img src="svgs/66c55ed40bd1f9f82c48c22b5665e5b4.svg?invert_in_darkmode" align=middle width=64.81959pt height=24.6576pt/>:

<p align="center"><img src="svgs/86ce8846af16bf5f531b68504cd11a1e.svg?invert_in_darkmode" align=middle width=126.728745pt height=12.7854045pt/></p>

<p align="center"><img src="svgs/a06ed81cafc6f1f8c27d2733a9122134.svg?invert_in_darkmode" align=middle width=111.768855pt height=12.7854045pt/></p>

<img src="svgs/0d9ce7c28dd16a4a3368542c3ca86a10.svg?invert_in_darkmode" align=middle width=46.57554pt height=24.6576pt/> is not self-inverting. For example, this would require that <img src="svgs/3f85da2e02a67cb0022399fe72188984.svg?invert_in_darkmode" align=middle width=51.141585pt height=21.18732pt/>, but in that group <img src="svgs/98706f43aab06e1c6a51059f936547cc.svg?invert_in_darkmode" align=middle width=59.36073pt height=21.18732pt/>. Therefore, we cannot construct a homomorphism <img src="svgs/190083ef7a1625fbc75f243cffb9c96d.svg?invert_in_darkmode" align=middle width=9.8175pt height=22.83138pt/> such that <img src="svgs/f8459734707b8539adaad1017520b325.svg?invert_in_darkmode" align=middle width=100.367685pt height=24.6576pt/>. It is impossible.

Which group has 64 elements and is self-inverting? The group <img src="svgs/7446d816a12be7b02fbd14ddf09acf47.svg?invert_in_darkmode" align=middle width=46.73526pt height=26.76201pt/>, or the set of 6-bit bitvectors under XOR. It turns out, <img src="svgs/e257acd1ccbe7fcb654708f1a866bfe9.svg?invert_in_darkmode" align=middle width=11.027445pt height=22.46574pt/> must be the "same kind of group" as <img src="svgs/9b325b9e31e85137d1de765f43c0f8bc.svg?invert_in_darkmode" align=middle width=12.92478pt height=22.46574pt/>, also a vector space over <img src="svgs/842a3ba6459f9c7d0b7724742b431bc1.svg?invert_in_darkmode" align=middle width=40.18278pt height=24.6576pt/>.

You may not yet be convinced that <img src="svgs/190083ef7a1625fbc75f243cffb9c96d.svg?invert_in_darkmode" align=middle width=9.8175pt height=22.83138pt/> is a homomorphism. At this point, we can actually represent <img src="svgs/190083ef7a1625fbc75f243cffb9c96d.svg?invert_in_darkmode" align=middle width=9.8175pt height=22.83138pt/> as matrix multiplication of a <img src="svgs/301cd3bb428dd419c0d5297a8dbb1151.svg?invert_in_darkmode" align=middle width=44.748825pt height=21.18732pt/> vector with a <img src="svgs/388a92a7f9b8de7a7e5ad99b56aeed0b.svg?invert_in_darkmode" align=middle width=44.748825pt height=21.18732pt/> matrix, resulting in a <img src="svgs/98179e552bb4479228248cc851cd0063.svg?invert_in_darkmode" align=middle width=36.52968pt height=21.18732pt/> vector. The matrix would look like this:

<p align="center"><img src="svgs/38ea13ee7b483d20c1941d7116fab965.svg?invert_in_darkmode" align=middle width=247.9455pt height=118.357305pt/></p>

Matrix multiplication is a group homomorphism.

#### A Small Simplification

Recall:

<p align="center"><img src="svgs/33570e8d856faa4ead987d0c1be4e8de.svg?invert_in_darkmode" align=middle width=154.876425pt height=18.31236pt/></p>

<p align="center"><img src="svgs/a06ed81cafc6f1f8c27d2733a9122134.svg?invert_in_darkmode" align=middle width=111.768855pt height=12.7854045pt/></p>

We can make this minor simplification:

<p align="center"><img src="svgs/1c4614f2381217321b581ffe3365a0a4.svg?invert_in_darkmode" align=middle width=154.876425pt height=18.31236pt/></p>

### Mapping the Solution to Code

We will now translate the math above into the C++ code in the first section.

- Wherever we see <img src="svgs/df33724455416439909c33a7db76b2bc.svg?invert_in_darkmode" align=middle width=12.78552pt height=19.17828pt/>, we replace it with `^`, the bitwise XOR operator.
- Wherever we see <img src="svgs/31b5b986fb2493a1e9ff48a4d00d0409.svg?invert_in_darkmode" align=middle width=62.732505pt height=24.6576pt/> where <img src="svgs/3e18a4a28fdee1744e5e3f79d13b9ff6.svg?invert_in_darkmode" align=middle width=7.113876pt height=14.15535pt/> is the current board state, we replace it with `ComputeXOROfHeads()`.
- Wherever we see <img src="svgs/e362928854defc86f84307c22a81cf37.svg?invert_in_darkmode" align=middle width=63.04221pt height=24.6576pt/>, we replace it with a `uint6` representing the 6-bit encoding of the square on the chessboard.
- The implementation of `ComputeXOROfHeads` corresponds to <img src="svgs/737ef05c15b685a42179349e1d1984f9.svg?invert_in_darkmode" align=middle width=113.24874pt height=38.79018pt/>
- `GetMagicSquareIndex() ^ ComputeXOROfHeads()` corresponds to <img src="svgs/9faed75638c2be7d18cffbd1ce43a9af.svg?invert_in_darkmode" align=middle width=77.374605pt height=24.6576pt/>

## The Number of Squares on the Board Must Be a Power of 2

### The Short Explanation

- Imagine a 9x9 board. There will be 81 squares on it. We need 7 bits to represent the square indices, 0 to 80 inclusive. However, the full range representable by 7 bits is 0 to 127 inclusive. This means some board states encode values of 81 to 127. It is not guaranteed that, from every possible chessboard state, a single coin toggle can be used to cover the full range of possible desired board states [0, 80].
- Another explanation [[YouTube video](https://www.youtube.com/watch?v=wTJI_WuZSwE)] looks at the problem from the standpoint of graph coloring. Imagine a graph arranged like an <img src="svgs/55a049b8f161ae7cfeb0197d75aff967.svg?invert_in_darkmode" align=middle width=9.867pt height=14.15535pt/>-dimensional hypercube, where n is the number of squares on the chessboard. There are <img src="svgs/55a049b8f161ae7cfeb0197d75aff967.svg?invert_in_darkmode" align=middle width=9.867pt height=14.15535pt/> colors. We need to assign each node a color such that all <img src="svgs/55a049b8f161ae7cfeb0197d75aff967.svg?invert_in_darkmode" align=middle width=9.867pt height=14.15535pt/> colors are reachable from one of its direct neighbors. The number of nodes will always be a power of 2 (due to the number of possible chessboard states being <img src="svgs/f8f25e4580c418a51dc556db0d8d2b93.svg?invert_in_darkmode" align=middle width=16.34523pt height=21.8394pt/>). If the number of colors is not also a power of 2, it will not divide evenly. Some colors will be represented more than others. The video has a more detailed explanation.

### The Long Explanation

Remember above we said that, in order to guarantee the existence of a legal move in <img src="svgs/bb2340bc819a3e831b8cbfd2391d67c6.svg?invert_in_darkmode" align=middle width=113.972925pt height=26.76201pt/>, we must guarantee <img src="svgs/71cb5ef0976705bb1329f4431a68cb9b.svg?invert_in_darkmode" align=middle width=65.188365pt height=24.6576pt/>. Or in other words, <img src="svgs/7c824f3c569b0a60602ee689772d69f3.svg?invert_in_darkmode" align=middle width=86.035455pt height=24.6576pt/>.


## Appendix

### Review of Group Theory

#### What is a Group?
- A group <img src="svgs/5201385589993766eea584cd3aa6fa13.svg?invert_in_darkmode" align=middle width=12.92478pt height=22.46574pt/> is the tuple <img src="svgs/68c91478b407d56f3e1870f1c19cc33f.svg?invert_in_darkmode" align=middle width=57.95097pt height=24.6576pt/>.
- It is a set <img src="svgs/e257acd1ccbe7fcb654708f1a866bfe9.svg?invert_in_darkmode" align=middle width=11.027445pt height=22.46574pt/> with an associative binary operator <img src="svgs/df33724455416439909c33a7db76b2bc.svg?invert_in_darkmode" align=middle width=12.78552pt height=19.17828pt/>
- The set contains an identity element <img src="svgs/24f21c4f924a49dc5f3d6cd7b1cb4e89.svg?invert_in_darkmode" align=middle width=38.77269pt height=22.46574pt/> such that <img src="svgs/4fef05c55c708a7fa54121562a8332a3.svg?invert_in_darkmode" align=middle width=126.72891pt height=22.83138pt/>
- Each element <img src="svgs/2d8cca33f0ee74986943da285a93a659.svg?invert_in_darkmode" align=middle width=38.824005pt height=22.46574pt/> has an inverse element <img src="svgs/a3d9b1b249a83ed17fc41dae0cd03536.svg?invert_in_darkmode" align=middle width=51.609525pt height=22.46574pt/> such that <img src="svgs/994bf83f590b888995ca453eb1ef6a8a.svg?invert_in_darkmode" align=middle width=152.299785pt height=24.6576pt/>
- The <img src="svgs/df33724455416439909c33a7db76b2bc.svg?invert_in_darkmode" align=middle width=12.78552pt height=19.17828pt/> operator may or may not be commutative (when it is commutative, we call the group "Abelian")

#### Additional Definitions
- A vector space <img src="svgs/e293fa0f67d04b8f8108558c7ce4bde1.svg?invert_in_darkmode" align=middle width=20.98008pt height=22.46574pt/> is a group consisting of vectors with <img src="svgs/55a049b8f161ae7cfeb0197d75aff967.svg?invert_in_darkmode" align=middle width=9.867pt height=14.15535pt/>-dimensions over the field <img src="svgs/b8bc815b5e9d5177af01fd4d3d3c2f10.svg?invert_in_darkmode" align=middle width=12.853995pt height=22.46574pt/>.
- Every vector space <img src="svgs/e293fa0f67d04b8f8108558c7ce4bde1.svg?invert_in_darkmode" align=middle width=20.98008pt height=22.46574pt/> has a basis: a set of <img src="svgs/55a049b8f161ae7cfeb0197d75aff967.svg?invert_in_darkmode" align=middle width=9.867pt height=14.15535pt/> elements <img src="svgs/8b9af7f0c3dd172a35e9b89542c68313.svg?invert_in_darkmode" align=middle width=183.243555pt height=24.6576pt/> such that each element of the vector space can be expressed as a unique sum of basis elements: <img src="svgs/c515b96ce425844f6946565df36c24a7.svg?invert_in_darkmode" align=middle width=155.467785pt height=51.00447pt/> where <img src="svgs/23e262c50c8071736ca95434dbb50260.svg?invert_in_darkmode" align=middle width=97.857375pt height=22.83138pt/>
- A standard basis is a basis of a vector space where each basis element is a vector of the form <img src="svgs/ab7126e4ad76a713043e3acadc024961.svg?invert_in_darkmode" align=middle width=76.69167pt height=24.6576pt/> or <img src="svgs/7ea33751c9927247cb3cdaa34e6e3db2.svg?invert_in_darkmode" align=middle width=76.69167pt height=24.6576pt/>, etc
- A group homomorphism is a function <img src="svgs/396ebc256c33dd601356a3467916d899.svg?invert_in_darkmode" align=middle width=74.86512pt height=22.83138pt/> from group <img src="svgs/b8bc815b5e9d5177af01fd4d3d3c2f10.svg?invert_in_darkmode" align=middle width=12.853995pt height=22.46574pt/> to group <img src="svgs/5201385589993766eea584cd3aa6fa13.svg?invert_in_darkmode" align=middle width=12.92478pt height=22.46574pt/> such that <img src="svgs/0ff4d02bb1960f671388dfe24832f757.svg?invert_in_darkmode" align=middle width=240.223005pt height=24.6576pt/>
- As corollaries to the above, for any group homomorphism <img src="svgs/190083ef7a1625fbc75f243cffb9c96d.svg?invert_in_darkmode" align=middle width=9.8175pt height=22.83138pt/>, the following are true: <img src="svgs/2c43d9f3a62f8128b9c6bb7789635316.svg?invert_in_darkmode" align=middle width=80.991405pt height=24.6576pt/> and <img src="svgs/9cd13c7a6ba13e3cf1c4fa8efca4fd6e.svg?invert_in_darkmode" align=middle width=174.538155pt height=24.6576pt/>
- The image of a set <img src="svgs/9b325b9e31e85137d1de765f43c0f8bc.svg?invert_in_darkmode" align=middle width=12.92478pt height=22.46574pt/> under <img src="svgs/190083ef7a1625fbc75f243cffb9c96d.svg?invert_in_darkmode" align=middle width=9.8175pt height=22.83138pt/> is <img src="svgs/2e140ce473466495ad522e51466335b6.svg?invert_in_darkmode" align=middle width=153.775215pt height=24.6576pt/>
- The preimage or inverse image of a set <img src="svgs/e257acd1ccbe7fcb654708f1a866bfe9.svg?invert_in_darkmode" align=middle width=11.027445pt height=22.46574pt/> under <img src="svgs/190083ef7a1625fbc75f243cffb9c96d.svg?invert_in_darkmode" align=middle width=9.8175pt height=22.83138pt/> is <img src="svgs/549ed9332e691a07b360f1de8c200533.svg?invert_in_darkmode" align=middle width=200.644455pt height=26.76201pt/>
