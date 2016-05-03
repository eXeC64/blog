+++
date = "2016-05-03T23:58:00Z"
title = "On Electronic Vote Counting"
+++

## Background

In a couple of days time, there's going to be a Mayoral election in London.
[This video](https://www.youtube.com/watch?v=XdHCK3pJFGA) details how
electronic vote counting is being deployed in the London Mayoral elections.
Naturally I'm concerned about the possibility voting fraud and manipulation
of the result of the election.

It's important for me to clarify that I'm opposed to *electronig voting*.
That is, people voting over the internet.  This is on grounds of security. Tom
Scott has [an excellent video](https://www.youtube.com/watch?v=w3_0x6oaDmI)
discussing why electronic voting is a bad idea.

However, electronically counting votes isn't so inherently bad, as long as the
votes are cast in a traditional secure way, i.e. pencil and paper. It does
have some obvious security concerns though.

What if the author of the counting software is malicious, and writes the software
to miscount in favour of a particular candidate or political party? It only takes
one backdoor, uncaught security breach, or disgruntled employee for such a slip
to occur.

Perhaps as assurance we're given a copy of the source code that we can independently
vet. But even that isn't foolproof, as the [underhanded C contest](http://www.underhanded-c.org/)
demonstrates repeatedly. Even if we are certain the source code is above board,
how can we verify the software on the counting devices corresponds to the source
code? We could install it ourselves, but then everyone must trust us too. This
is a vicious cycle, that goes on ad infinitum. Just read [Reflections On Trusting Trust](https://www.win.tue.nl/~aeb/linux/hh/thompson/trust.html)
if you want to be thoroughly freaked out.

There's simply no way around it, we simply cannot *trust* the vote counting
machine wholesale. It must be treated as potentially malicious. So is there a
way we can employ this technology in a safe, trustworthy, and verifiable way?

I think so, yes. The following is a scheme I've come up with allows for the
efficiency of electronic vote counting, whilst maintaining a very high confidence
that no manipulation of the outcome has taken place. (Without hand counting
every ballot it's impossible to be 100% certain, but we can get very close
without much manual labour)

## Step 1 - Boxing

Separate all the ballots into equal, or near equal sized chunks, henceforth 
called *boxes*. These should contain on the order of a few hundred ballots.
Enough to hand-count reasonably quickly, but not so few that there are too
many boxes.

## Step 2 - Initial Count

Have the vote counting machine count each box's votes separately, giving a
count for that box. Ideally use several independent machines that have no
knowledge of the total number of boxes, what order they are scanned in, or what
any other machine is counting or outputting. A hard copy of this count shall be
made and attached to the box, with the machine used also noted.

## Step 3 - Basic Verification

Every box should now have an initial count. Have invested parties (such as candidates)
or their agents select 15% to 25% of the boxes at random.

If you have only one machine, have it recount those boxes, and verify the same
count is given for each box. This protects against less sophisticated
algorithms for manipulating the count, such as randomly changing an
unfavourable vote to be a favourable one.

If you have two or more machines, split each box into two piles. Have a separate
machine count each pile, then sum the results. Verify the new count is the same
as the initial count. This challenges the machines to lie consistently, whilst
making it more difficult to recognise and regurgitate a previous fraudulent count.
It also protects against deterministic manipulation of the vote count, that is
the same authentic count being deliberately miscounted the same way every time.

At this point, you have only a very rough assurance that the votes have not
been tampered with, but fraud could still be easily missed by bad luck.
Increasing the amount of votes re-counted electronically can increase
confidence, but is still susceptible to fraud by an intelligent attacker.

## Step 4 - Statistical Verification

Create a spreadsheet, and insert a row for every box. Each row should have
columns showing the count for each candidate, and columns showing the
percentage of votes received by each candidate in that box.

Calculate the standard deviation of each percentage column. With sufficiently
large boxes, the distribution of votes within a box should be very similar across
all the boxes. The standard deviation should therefore be low (barring factors
such as certain boxes containing votes from an area with a strong bias for
one candidate). A high standard deviation ought to be treated with suspicion,
and boxes with unusual counts hand counted.

Now, take each percentage column and plot a histogram of its values, with each
bin representing a range of one percent. This histogram ought to have a normal
distribution. Outliers must be hand counted.

Statistic discrepancies in the result should now have been identified, with
boxes containing unlikely counts, or counts that are uncharacteristic of the
election as a whole, being verified. This protects against an attacker manipulating
many votes in only a few boxes, in the hope of not being discovered by sheer
luck alone. Instead, to avoid showing up in statistical analysis, the attacker
must now manipulate far fewer votes per box, but in many more boxes, dramatically
increasing the chance of detection by spot checks.

## Step 5 - Spot Checks

The final, and most labour intensive part of the verification procedure.
Invested parties or their agents again select boxes at random, this time to be
counted by hand. The more boxes that are re-counted, the more secure the result
is. It is recommended that boxes from across the entire normal distribution of
percentage votes for each candidate be checked.

These hand counts are essential, as they prevent a sophisticated attacker from
merely translating the normal distribution along its x axis, disguising the
manipulation. Such a shift requires many boxes to be manipulated, else the
normal distribution will appear sharp, and uneven. Randomly hand counting
boxes from various points along this distribution make such a shift impossible
to disguise without a very high chance of detection by this step, or any of the
previous ones.

## Conclusion

There you have it. My big scheme for employing technology to streamline
elections without compromising the security and trustworthiness of the count.
I'm pretty sure of this technique, and as far as I can tell, it would easily
catch both an attacker manipulating many votes in few boxes, and an attacker
manipulating few votes in many boxes.

Can you pick any holes in this approach? How could you rig an election by ensuring
a majority for your candidate? Remember, the counting machines don't know how
many ballots they'll be counting in total, but they need to ensure that your
candidate gets more votes than any other candidate.

If you figure out a way, I'd love to know about it.
