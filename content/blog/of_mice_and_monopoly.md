+++
date = "2016-04-08T19:52:26Z"
title = "Of Mice and Monopoly"

+++

## Introduction

I've been playing a bit too much [Monopoly](https://en.wikipedia.org/wiki/Monopoly_(game\))
lately. So as an aid to my procrastination from completing the dissertation for
my Computer Science degree, I decided to employ a little bit of maths and
finance theory to try and figure out which properties offer the most value for
money, and which are duds.

The result of my labour is [this scruffy little spreadsheet here](https://docs.google.com/spreadsheets/d/1t8OPPOxSrS0p2dl08SWcF4LhF3hy3p6S-8nR5ESp4Us/edit?usp=sharing).
As a Brit, I'm using the London property names and prices. The abbreviations
I used in the spreadsheet may be a little bit cryptic, so bear with me while I
explain them.

 * **S. Inc.** - Single income. The rent charged for this property without a set.
 * **S. Rate** - Single rate. The proportion of the purchase cost each rental gives.
 * **B. Inc.** - Base income. The rent charged for this property when the entire set is owned.
 * **B. Rate** - Base rate. The proportion of the purchase cost each rental gives when the entire set is owned.
 * **D. Cost** - Developed cost. How much it costs to purchase and build a hotel on this property.
 * **D. Income** - Developed income. The rent charged for this property with a hotel on it.
 * **D. Rate** - Developed rate. The proportion of the developed cost each rental gives on this property.
 * **Obs. Cost** - Obstruction cost. How much it costs to purchase, then immediately mortgage this property, obstructing someone else from building a set.

## Residential Lots

Now that we're more comfortable with the spreadsheet, let's break down the numbers
a little bit.

As we can see, the cheaper the property, the worse its single and basic rates are.
Buying Old Kent Road may not cost much, but you're going to get a pittance in
rental income unless you're able to build a set and develop it. In fact, someone
would have to land on Old Kent Road thirty times before you make your money back.
Not a good investment. On the other end of the spectrum is Mayfair, which costs
a whopping £400, but provides £50 as its base rental. You only need to collect
rent eight times to make your money back, making it a far sounder investment.
All the properties inbetween follow the same linear trend, so I won't comment
on them further.

Where things get interesting is when properties are developed. The cost of
constructing houses is fixed for each side of the board, ranging from £50 to
£200, in increments of £50, with hotels costing the same. The cost of improving
a single lot therefore varies from £250 to £1,000, and the cost of improving
a set of three lots from £750 to £3,000. However, the rental income varies
independently, leading to different rates of return from different sets.

The most efficient set is the light blues: The Angel Islington, Euston Road, and
Pentonville Road. This set costs only £1,070 to fully develop, with a total
rental yield from its lots of £1,700. This produces an expected rate of return
of 158%, the highest in the game.

N.B. This rate of return metric is somewhat arbitrary. It's not useful for
telling us how long it will take to make our money back, but it's good enough
for comparing the different investments.

The light blues are also suited to an early game investment due to their low
base cost and cheap housing. If you're a fan of anecdotal evidence, In the last
game I played these were the first set I purchased, and were extremely lucrative,
allowing me to rapidly expand and eventually win the game by a landslide.

The next best set are the oranges, for their similarly high rate of return. If
you can't get the oranges, the pinks can substitute, costing £120 less to develop,
but yielding £500 less in total. The oranges are notably more expensive than the
blues to develop, setting you back £2,060, over £500 more than each player's
initial balance. Oranges make a great investment when you already have a set
like the browns or light blues, but I'd be reluctant to invest in them heavily
early in the game as you're liable to run short on cash very quickly.

Investments to avoid would be the reds, yellows, and greens. These have among
the lowest developed rental yields releative to cost, and require a lot of cash
to become worthwhile. Purchase at most one of each of these. If you run short
of cash you can always mortgage is to reclaim half its printed value, and it
insures you against that set being developed by another player. Not a bad bet
in the long run.

## Stations

The stations in Monopoly are an excellent investment. Owning just one of these
offers an immediate rate of 12.5%, the same as the highest rate for a single
lot on the board. This only improves with each additional station, making its
way up to a 25% return. Not only do these provide a cheap source of regular
income (you'd be surprised how often people land on stations), but they also
provide safe landing sports around the board.

## Utilities

The utilities are a unique pair of properties on the Monopoly board, due to the
way their rent is calculated. If one utility is owned, the rent is the number
on the dice multiplied by four. If both utilities are owned, the rent is the
number on the dice multiplied by ten. This makes our calculations a little more
complex, but not insurmountable.

The most likely roll with two six sided dice is seven. The most likely being
two and twelve. By summing the product of the probability of each roll and the
rent yielded by the roll we get the average income. For one utility this is £28,
and for both utilities it's £70. Against the cost, this gives us a rate of 18.67%
for one utility and 23.33% for two.

While this may look similar to the train stations it's really only about half
as good, since half as many places on the board pay out compared to the stations.

## Bonus: My Super Sneaky ~~Secret~~ Strategy

This is a strategy that works in larger games with four or more players. It
also requires verbal agreements made in the game to be binding. That is, if I
give you something now in exchange for something else later, you can't back out
later.

The key to this strategy is subverting as many of the cash flows in the game to
flow towards you, but not away. By entering agreements where you never have to
pay rent on another player's set but they don't have the same luxury you set
yourself up for a guaranteed win. Here's how the strategy works.

1. Obtain one property of as many sets as possible. Block players from getting
   monopolies without your co-operation.
2. Attempt to build a monopoly of your own, if you can't that's okay. The key
   is making the board safe for yourself and then starting to siphon cash off
   the other players.
3. When a player has 2/3 properties in a set, offer to give them (or sell them
   very cheaply) the last property of the set. The strategy here is to absolutely
   **require** that they agree to give you immunity from rent for that set. When
   there are many other players and that's the only way for the player to make
   their set they'll agree to this surprisingly easily.
4. Repeat step 3 with as many other players' as possible. Set the board up so
   everyone pays each other rent, but you pay rent on as little of the board as
   possible.
5. Build up your own set. If you don't have one, negotiate cleverly. Perhaps
   offer one player the same deal in reverse. Even if they can't pay you rent,
   they can pay other players rent, and the other players can pay you rent.
   The money will find its way to you.
6. Profit. People are paying you rent. You're not paying them rent.

This sounds like it would never work, but of the 4+ player games I've played
where verbal agreements are binding, this has worked about 75% of the time. Of
course, if you're trying it for the third time against the same players they
probably won't play ball.

Your mileage may vary.
