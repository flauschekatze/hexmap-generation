# Hex Map Generation

#### Last Update: 2018-04-14 (Initial unpublished version without images)

I've been interested in map generation for a while. There are a lot of resources out there on computer algorithms to use, but when I recently began looking into pen and paper roleplaying games again, I discovered [hexcrawls](https://arsphantasia.wordpress.com/2014/02/20/hexcrawl-resources/). These are essentially roleplaying campaigns set around exploring and there are various systems out there to aid the game master in creating landscapes to explore using dice rolls on tables to determine various aspects.

Specifically, I was paging through [Scrawl](http://www.drivethrurpg.com/product/203380/Scrawl)'s hexcrawl [supplement](http://www.drivethrurpg.com/product/205282/Explore-to-the-Core-A-SCRAWL-supplement) and it suggested improvising the shape of the landmasses you explore. Since it's very much a pen and paper game, I figured it would be fun to try and come up with a method to generate the shape of islands (or continents) using only pen, paper and dice. In this particular case, as Scrawl uses only six-sided dice, I start with using only those dice.

#### Further Resources

If you wish to try these out without a computer, there are various kinds of free, PWYW or low-cost PDFs with hex graph paper available on DriveThruRPG; two examples are [Hex Grids](http://www.drivethrurpg.com/product/154178/Hex-Grids) from Crooked Staff Publishing and [Little Hexes](http://www.drivethrurpg.com/product/121264/Little-Hexes) from Peter Regan. Little Hexes contains two blank grids at the end as a sample for the printed pads of hex paper Peter offers at [Squarehex](https://squarehex.myshopify.com/).

If you prefer to follow along digitally, you could use the free demo of [Hexographer](http://www.hexographer.com/) or whatever else is most convenient for you. I will eventually publish the Perl scripts used to create the visualizations below. These use the magnificent [Text Mapper](https://campaignwiki.org/text-mapper) by Alex Schroeder. I highly recommend checking out his [RPG site](https://alexschroeder.ch/wiki/RPG) including various procedural generators.

## Island Shapes

The primary goal is to find an algorithm that is simple enough that it can be done without a computer, as little bookkeeping as possible and as few dice rolls as possible, while still making the resulting landmass interesting enough to actually explore.

Since my only aim is to come up with the shape, as the terrain will be filled in with random rolls while playing, I only differentiate between whether a hex is "filled in", i.e. a hex of unexplored land, or "not filled", i.e. an ocean hex.

### Compass and Orientation

Before you start, you'll need to pick which die result corresponds to which side of a hex, as well as which [hex grid orientation](https://gamedev.stackexchange.com/questions/49718/vertical-vs-horizontal-hex-grids-pros-and-cons) you prefer. In my implementations, I use flat side up hexes and start numbering the sides at the top (12 o'clock, if you will), clockwise:

```
   1
6 /-\ 2
5 \-/ 3
   4
```

If you want to go straight to the most refined algorithm I currently have, check Algorithm 5.

### Algorithm 1: Random Fill and Move

Start by picking a current hex, perhaps the centre hex of your grid. Fill it in.

Roll two dice. The first die is your movement die, the second die is your direction die.

If your movement die is __1__ or __2__, you'll be __moving__: The direction die tells you in which direction to move. Find the next empty hex in that direction, fill it in and consider it your new current hex.

If your movement die is __3__ or more, you'll be __filling__: The direction die tells you in which direction to fill a hex. Find the next empty hex in that direction and fill it in, without moving your current hex.

If you've now reached your target number of hexes, you are done. Otherwise, reroll the dice and repeat.

#### Results

This algorithm produces rather "spiky" islands with narrow "arms". There will be a bunch of backtracking to fully explore these landmasses.

`TODO: Example images, stats`

Let's try to curb the spikiness:

### Algorithm 2: Short Fill and Move

Start by picking a current hex, perhaps the centre hex of your grid. Fill it in.

Roll two dice. The first die is your movement die, the second die is your direction die.

If your movement die is __1__ or __2__, you'll be __moving__: The direction die tells you in which direction to move. Find the next empty hex in that direction, fill it in and consider it your new current hex.

If your movement die is __3__ or more, you'll be __filling__: The direction die tells you in which direction to fill a hex. *If the hex in that direction is empty, fill it in. If it is already filled, do nothing.*

If you've now reached your target number of hexes, you are done. Otherwise, reroll the dice and repeat.

#### Results

The spikiness is curbed a little, but the shapes are still not all that nice to explore.

`TODO: Example images, stats`

Next tweak: more predictable movement.

### Algorithm 3: Count and Move

Start by picking a current hex, perhaps the centre hex of your grid. Fill it in.

Roll two dice. The first die is your movement die, the second die is your direction die.

If your movement die is __*less than or equal to the number of filled neighbours*__, you'll be __moving__: The direction die tells you in which direction to move. Find the next empty hex in that direction, fill it in and consider it your new current hex.

If your movement die is __*greater than that*__, you'll be __filling__: The direction die tells you in which direction to fill a hex. If the hex in that direction is empty, fill it in. If it is already filled, do nothing.

If you've now reached your target number of hexes, you are done. Otherwise, reroll the dice and repeat.

#### Results

Again, an improvement, but not all that much. There are still occasional long, single-hex arms.

`TODO: Example images, stats`

Let's try to reduce the number of hexes with just a single neighbour.

### Algorithm 4: Count and Move On 2+

Start by picking a current hex, perhaps the centre hex of your grid. Fill it in.

Roll two dice. The first die is your movement die, the second die is your direction die.

If your movement die is __*1*__, you'll be __filling__, see below.

If your movement die is __*2 or more*__, but __less than or equal to the number of filled neighbours__, you'll be __moving__: The direction die tells you in which direction to move. Find the next empty hex in that direction, fill it in and consider it your new current hex.

If your movement die is __greater than that__, you'll be __filling__: The direction die tells you in which direction to fill a hex. If the hex in that direction is empty, fill it in. If it is already filled, do nothing.

If you've now reached your target number of hexes, you are done. Otherwise, reroll the dice and repeat.

#### Results

Movement seems to be improved now, but just filling immediate neighbours is not entirely ideal either.

`TODO: Example images, stats`

Let's try to reduce the number of hexes with just a single neighbour.

### Algorithm 5: Count and Move, Limit Range

Start by picking a current hex, perhaps the centre hex of your grid. Fill it in.

Roll two dice. The first die is your movement die, the second die is your direction die.

If your movement die is __*less than or equal to the number of filled neighbours*__, you'll be __moving__: The direction die tells you in which direction to move. Find the next empty hex in that direction. If the distance between it and the current hex is no more than the value of your movement die, fill it in and consider it your new current hex. Otherwise, do nothing.

If your movement die is __greater than that__, you'll be __filling__: The direction die tells you in which direction to fill a hex. If the hex in that direction is empty, fill it in. If it is already filled, do nothing.

If you've now reached your target number of hexes, you are done. Otherwise, reroll the dice and repeat.

#### Results

Overall, I am satisfied with these results; they seem to generate shapes that will be possible to explore with a minimal amount of backtracking, while still offering some variation.

`TODO: Example images, stats`




### Current TODO:

* Make gifs/images
* Clean & release Perl stuff
