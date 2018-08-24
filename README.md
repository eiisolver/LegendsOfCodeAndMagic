# LegendsOfCodeAndMagic
This is a report of the 4 week AI contest "Legends of Code and Magic" hosted by www.codingame.com during the summer of 2018.

The contest was a simplification of the card game Hearthstone, a two-player card game. Participants write a program called "bot" that plays
against the bots of other participants, and the best bots win a t-shirt and lots of honour.

During the summer vacation I had studied Alpha Zero's reinforcement learning algorithm and I had played around
with a toy open source implementation of it. I had also bought a brand new laptop with a GPU so I was really in
the mood for some machine learning when the competition started.

An Alpha Zero-like implementation on codingame is not feasible, since the bot may contain a max of 100 K source code;
a typical alpha zero neural network would take many megabytes to store all the weights.

Therefore my plan was to create a "Alpha One" implementation. The Zero in Alpha Zero stands for zero domain knowledge.
My son used to be good at Hearthstone, so together we studied the Legends of Code and Magic game and we extracted
a list of 20+ features that seemed important to judge who is going to win.

The features include own and opponent attack/defense/mana/number of cards on board/in hand, number of lethals/wards/guards on board,
etc etc. So every position can be converted to a list of floats (one for every feature), let's call it the fingerprint of the position.
The fingerprint is going to be fed into a neural network which has one input signal for every feature + one output signal, which is the
probability that we are going to win.

I ended up with a neural network with 37 inputs (features) and one hidden layer of 30 neurons. Two hidden layers did not give better results.

I started with a simple algorithm to select which move to play: simulate every possible move we can play, feed the resulting
position in the neural network, and play the move that returns the highest probability to win according to the net.

The learning algorithm is also simple:

1. start with a randomly initialized neural network
2. self-play 10 000 games
3. record the fingerprint of every position, along with the result of the game. For example if blue won a game, all positions during the game
   after a blue move are saved with result 1 and all positions after a red move are saved with result 0.
4. create a csv file that includes all the fingerprints + game results and use it to train the neural network, so the game result is used
   as the learning target. I used Keras, a python based library which is easy to use.
5. export the learned weights from python to the AI bot's source code (which was written in C++)
6. goto 2.

The cool thing is that from nothing, the bot became stronger and stronger, and displayed "emergent behaviour". I do not play Hearthstone at all, so
I was amazed to see evaluation "78% win chance" in a positition that I thought was hopeless, and indeed a few moves later the tables
were turned and my bot won. I do not think I would have been able to create a nearly as good bot by hand-tuning the feature parameters.

However, since the neural network is small, the bot stops improving after
some iterations. I added a few more features and this made the bot a little bit stronger. I also added some simple opponent simulation to avoid
very bad moves where we play an expensive card that is immediately killed by a cheap lethal card. But I did not know how to evaluate "half
finished" situations where the opponent kills one of my cards and the opponent card is still left on the board, but cannot be used in the
rest of that move.

So the simulation capabilities of my bot are limited compared to most other well performing bots. The simulation code is also very
simple: I have a loop "while (time left): perform a completely random move, calculate fingerprint, feed it to the network and return the score,
if it is higher than current max: check opponent moves". If there are not many moves possible, identical moves are tested over and over again,
and if there are many moves possible possibly only a fraction of those will be evaluated.

I never bothered to improve this non-optimal simulation code since I did not find noticeable improvements when I tested with many times more
allowed computing time.

# Drafting

The draft phase is a phase consisting of 30 moves where you select one out of three presented cards. It turned out that it is very important to
choose wisely.

I started simple. As part of the self play, also win-statistics for every card were maintained (different statistics for red and blue).
When drafting we just select the card with the highest historic win rate during self-play.

Then I tried to build a neural network similar to the one used during searching: using around 20 features that characterize the cards we have already
selected I tried to predict the expected win-rate, so my hope was that I could just feed this network with the 3 different possibilities and pick the
card with the highest score.

Unfortunately I was not able to make this work, and I gave up. The network never really stabilized and could return totally different scores for
very similar card selections, and it played considerably worse than the win rate selection strategy. In hind-sight I should have tried harder.

Next thing I tried was to simulate card usage based on the currently selected mana curve. For example if we only select cards with very high cost,
it is likely that we will loose because we will not be able to play the first rounds, and vice versa, if we only select cards with low cost,
there is a risk that we will not be able to play enough cards during later rounds and loose later on.

For each of the three cards from which we much choose I
simulated 1000 simplified games where in each game get random cards and we try to maximize mana usage, i.e. first turn we try to use up 1 mana, second turn as close as 2 mana
as possible, etc until 12 turns. The result of the simulation is a histogram that gives info like "if we pick this card, on average, we will be
able to use 55% of all available mana on turn 1, 68% of all available mana on turn 2, 87% mana on turn 3, etc.

When we choose between two cards with nearly identical win-rates, we give bonus for the one that results in the best overal mana usage.

Unfortunately also this approach did not work, in fact I don't know why.

On the very last day I was at least able to improve a little bit by attempting to achieve an average cost of selected cards between 3.5 and 4
(did not have time to fiddle with exact parameters): for example if we have until now selected cheap cards, expensive cards get a bonus that is added to the historic win rate score. This gave a small improvement.

# Conclusion

It was a really fun contest, one of the best so far at codingame, and I learned a lot. I had no idea what result to expect from the reinforcement learning experiment
and was very pleased to make it to the legend league.
As usual it was a pleasure to use the codingame.com platform, I can really recommend to participate in one of its competitions.
