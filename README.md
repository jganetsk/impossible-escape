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
- The image of a set <img src="/tex/9b325b9e31e85137d1de765f43c0f8bc.svg?invert_in_darkmode&sanitize=true" align=middle width=12.92464304999999pt height=22.465723500000017pt/> under <img src="/tex/190083ef7a1625fbc75f243cffb9c96d.svg?invert_in_darkmode&sanitize=true" align=middle width=9.81741584999999pt height=22.831056599999986pt/> is <img src="/tex/1b84ce00a7faec46de95d1ba0df5af9b.svg?invert_in_darkmode&sanitize=true" align=middle width=435.00455955pt height=24.65753399999998pt/>S<img src="/tex/11096e6b179f614c40bc560f0747929d.svg?invert_in_darkmode&sanitize=true" align=middle width=43.36020644999999pt height=22.831056599999986pt/>f<img src="/tex/253d3d381ac81d846b9a66f189190e3d.svg?invert_in_darkmode&sanitize=true" align=middle width=13.36870589999999pt height=21.68300969999999pt/>f^{-1}[S] = \{c \in C \mid f(c) \in S\}<img src="/tex/6eba57e0dbf75da82decf1c821773dfd.svg?invert_in_darkmode&sanitize=true" align=middle width=282.6720468pt height=78.90410879999997pt/>C<img src="/tex/3f83b43ad51b74f7507eb82fc5c5df6d.svg?invert_in_darkmode&sanitize=true" align=middle width=749.5517732999999pt height=45.84475500000001pt/>C<img src="/tex/065486305714b3da3d66390f05fc6b57.svg?invert_in_darkmode&sanitize=true" align=middle width=1794.1551144000002pt height=22.831056599999986pt/>e \in C<img src="/tex/6a047f30c40c9060a2d66da6905f6ae1.svg?invert_in_darkmode&sanitize=true" align=middle width=1072.3280188499998pt height=47.671232400000015pt/>C<img src="/tex/fc0b782bba4ff9017cf2d6ed0f766dba.svg?invert_in_darkmode&sanitize=true" align=middle width=160.822299pt height=39.45205439999997pt/>C<img src="/tex/963c311a889c3ff574c5213536ffc70e.svg?invert_in_darkmode&sanitize=true" align=middle width=107.59737944999999pt height=22.831056599999986pt/>\{0, 1\}^{64}<img src="/tex/7f3a10a2f880f9819d4e00026566f3c4.svg?invert_in_darkmode&sanitize=true" align=middle width=110.73094064999998pt height=22.831056599999986pt/>C<img src="/tex/100d333f42afef5fb55642b5b191a2dd.svg?invert_in_darkmode&sanitize=true" align=middle width=351.16458135pt height=22.831056599999986pt/>+<img src="/tex/0d1744e7dd1d07e48366a44debda2288.svg?invert_in_darkmode&sanitize=true" align=middle width=680.36176395pt height=39.45205440000001pt/>B \subset C<img src="/tex/654590c07c32cb78c8820f69aaa84e16.svg?invert_in_darkmode&sanitize=true" align=middle width=342.52354289999994pt height=22.831056599999986pt/>\Delta c \in B<img src="/tex/896a25f180fb76a7b39ac70bd30d0ed5.svg?invert_in_darkmode&sanitize=true" align=middle width=331.3382853pt height=22.831056599999986pt/>c_{1} = c_{0} + \Delta c<img src="/tex/6244aaec562ec171c68123565b314f7a.svg?invert_in_darkmode&sanitize=true" align=middle width=12.785434199999989pt height=19.1781018pt/>f<img src="/tex/ab3c1fdca4b9ee061831d9d5b896ed0f.svg?invert_in_darkmode&sanitize=true" align=middle width=219.75949379999997pt height=22.831056599999986pt/>f : C \rightarrow S<img src="/tex/77f6c35afd48c0368b0b9c95ebad0798.svg?invert_in_darkmode&sanitize=true" align=middle width=107.40142875pt height=22.831056599999986pt/>S<img src="/tex/94be2488a2067d4a4bba3ff0411f0a3c.svg?invert_in_darkmode&sanitize=true" align=middle width=663.07980585pt height=39.45205440000001pt/>\Delta c<img src="/tex/7fd9fbe3336e8735826e8d8c769c0f23.svg?invert_in_darkmode&sanitize=true" align=middle width=237.39712425pt height=22.831056599999986pt/><img src="/tex/7e0b0f84da75b2556c33a223c49083e0.svg?invert_in_darkmode&sanitize=true" align=middle width=66.71422064999999pt height=24.65753399999998pt/><img src="/tex/51709c221bb606c7f0a6193f462db8dd.svg?invert_in_darkmode&sanitize=true" align=middle width=8.21920935pt height=14.15524440000002pt/><img src="/tex/0d26bc3e8e559ae5911309fb08c1ea5d.svg?invert_in_darkmode&sanitize=true" align=middle width=107.61788729999999pt height=24.65753399999998pt/><img src="/tex/51709c221bb606c7f0a6193f462db8dd.svg?invert_in_darkmode&sanitize=true" align=middle width=8.21920935pt height=14.15524440000002pt/><img src="/tex/ebeb39ee0f8e17053e5a5becba54ea53.svg?invert_in_darkmode&sanitize=true" align=middle width=130.22073404999998pt height=24.65753399999998pt/><img src="/tex/51709c221bb606c7f0a6193f462db8dd.svg?invert_in_darkmode&sanitize=true" align=middle width=8.21920935pt height=14.15524440000002pt/><img src="/tex/cd5e65e226a57ede5778ac786566a8f3.svg?invert_in_darkmode&sanitize=true" align=middle width=130.22073404999998pt height=24.65753399999998pt/><img src="/tex/8d6549f6680cbdfd61513d1bca28ed10.svg?invert_in_darkmode&sanitize=true" align=middle width=372.4210215pt height=45.84475499999998pt/>f^{-1}<img src="/tex/51709c221bb606c7f0a6193f462db8dd.svg?invert_in_darkmode&sanitize=true" align=middle width=8.21920935pt height=14.15524440000002pt/><img src="/tex/78dfe4810c82fd05270a2b693cd07f70.svg?invert_in_darkmode&sanitize=true" align=middle width=203.41737074999995pt height=26.76175259999998pt/><img src="/tex/b83e5c715c63cb4b01e2135a74723374.svg?invert_in_darkmode&sanitize=true" align=middle width=61.73537369999999pt height=39.45205439999997pt/><img src="/tex/d215f22aad063453311ffca7e3eec72c.svg?invert_in_darkmode&sanitize=true" align=middle width=142.3897134pt height=26.76175259999998pt/><img src="/tex/128b0026c3a2a413efc710169d689eaa.svg?invert_in_darkmode&sanitize=true" align=middle width=507.71815874999993pt height=45.84475499999998pt/>c_{0}<img src="/tex/5bcae781a01b6c3a2e19d92324afc028.svg?invert_in_darkmode&sanitize=true" align=middle width=53.75590274999998pt height=22.831056599999986pt/>f(c_{1}) = s<img src="/tex/c9ff754018e33be3354dfea711df3e6d.svg?invert_in_darkmode&sanitize=true" align=middle width=806.21928585pt height=22.831056599999986pt/>b<img src="/tex/f26d67404f324ccb97ecc2673e527b47.svg?invert_in_darkmode&sanitize=true" align=middle width=141.28123184999998pt height=22.831056599999986pt/>b \in B<img src="/tex/990b55f7244a3d876319d4abe7b2bd03.svg?invert_in_darkmode&sanitize=true" align=middle width=324.33173519999997pt height=22.831056599999986pt/>f<img src="/tex/6df56e8d65ccc860037cef9f485f058d.svg?invert_in_darkmode&sanitize=true" align=middle width=169.11133634999996pt height=22.831056599999986pt/>B<img src="/tex/bc2615e3b80a9f6c7c09c962202eb24e.svg?invert_in_darkmode&sanitize=true" align=middle width=151.00651334999998pt height=24.65753399999998pt/>S<img src="/tex/0fc6905abc0968eab9c6d9da46949347.svg?invert_in_darkmode&sanitize=true" align=middle width=106.16471745pt height=22.831056599999986pt/>f[B] = S<img src="/tex/790afbf3a698101afc5b405969e4958b.svg?invert_in_darkmode&sanitize=true" align=middle width=361.1511684pt height=22.831056599999986pt/>B<img src="/tex/203f7a0131559e930abbf7f28cdd6923.svg?invert_in_darkmode&sanitize=true" align=middle width=598.1799978pt height=24.65753399999998pt/>S<img src="/tex/7d2579bd0fab34069b6a682d8ea8bd29.svg?invert_in_darkmode&sanitize=true" align=middle width=1440.5838777pt height=118.35616320000003pt/>f(b)<img src="/tex/1da9fcafafb5c088ab6ddc5e21359621.svg?invert_in_darkmode&sanitize=true" align=middle width=44.86319144999999pt height=22.831056599999986pt/>b \in B<img src="/tex/ab0dc231c72f5133d9707d79c2963fa5.svg?invert_in_darkmode&sanitize=true" align=middle width=313.00305629999997pt height=22.831056599999986pt/>f(c)<img src="/tex/1da9fcafafb5c088ab6ddc5e21359621.svg?invert_in_darkmode&sanitize=true" align=middle width=44.86319144999999pt height=22.831056599999986pt/>c \in C<img src="/tex/db94ec000e14af77faa2ce5234019a45.svg?invert_in_darkmode&sanitize=true" align=middle width=197.54751389999998pt height=22.831056599999986pt/>S<img src="/tex/d17c3cf5d047e5e4ee9ee7b29a68cd0f.svg?invert_in_darkmode&sanitize=true" align=middle width=313.00862505pt height=22.831056599999986pt/>s - f(c_{0})<img src="/tex/2858d4a12acab5bda197e7c4d7c3df43.svg?invert_in_darkmode&sanitize=true" align=middle width=191.8016265pt height=22.831056599999986pt/>+<img src="/tex/706639e3e9b6901d49dd349a65741961.svg?invert_in_darkmode&sanitize=true" align=middle width=284.20164465pt height=85.29681270000002pt/>c \in C<img src="/tex/f1097b2d4641caf272e8d6795187061e.svg?invert_in_darkmode&sanitize=true" align=middle width=299.34748964999994pt height=22.831056599999986pt/>b_{i} \in B, c = \displaystyle\sum_{i} b_{i}<img src="/tex/d9159700c729f6206a12add992acfd07.svg?invert_in_darkmode&sanitize=true" align=middle width=82.42043699999998pt height=22.831056599999986pt/><img src="/tex/52efd2ba4358e4a5fadb98ea787072fb.svg?invert_in_darkmode&sanitize=true" align=middle width=113.2487202pt height=38.79006780000001pt/><img src="/tex/1075dfca734a87fdb1ef8bba5da4f905.svg?invert_in_darkmode&sanitize=true" align=middle width=77.51484344999999pt height=22.831056599999986pt/><img src="/tex/737ef05c15b685a42179349e1d1984f9.svg?invert_in_darkmode&sanitize=true" align=middle width=113.2487202pt height=38.79006780000001pt/>$
1. Hello
