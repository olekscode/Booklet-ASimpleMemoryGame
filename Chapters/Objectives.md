## Objectives of this book

Bloc is the new graphics library for Pharo. A graphics library implies several aspects such as coordinate systems, drawing shape, clipping, and event management. 

In this tutorial, you will build a memory game. Given a provided model of a game, we will focus on creating a UI for it.

### Memory game

Let us have a look at what we want to build with you: a simple Memory game. 
In a memory game, players need to find pairs of similar cards. In each round, 
a player turns over two cards at a time. If the two cards show the same symbol 
they are removed and the player gets a point. If not, they are both returned facedown. 

For example, Figure *@figmemoryExample0@* shows the game after the first selection 
of two cards. Facedown cards are represented with a cross and turned cards show their number. 
Figure *@figmemoryExample1@* shows the same game after a few 
rounds. While this game can be played by multiple players, in this tutorial we will 
build a game with just one player. 

![The game after the player has selected two cards: facedown cards are represented with a cross and turned cards with their number.](figures/memoryExample0.png width=60&label=figmemoryExample0)

Our goal is to have a working game with a model and a simple graphical user interface. 
In the end, the following code should be able to build, initialize, and launch the game:

```
game := MGGame withNumbers.
visual := MGGameElement new.
visual memoryGame: game.	

space := BlSpace new. 
space extent: 420@420.
space root addChild: visual.
space show 
```


- First, we create a game model and ask you to associate the numbers from 1 to 8 with the cards. By default, a game model has a size of 4 by 4, which fits eight pairs of numbered cards.
- Second, we create a graphical game element.
- Third, we assign the model of the game to the UI. 
- Finally, we create and display a graphical space in which we place the game UI. Note that this last sequence should be better packaged as a message to the `MGGameElement`.

 
 
![Another state of the memory game after the player has correctly matched two pairs.](figures/memoryExample1.png width=60&label=figmemoryExample1)

### Getting started


This tutorial is for Pharo 11.0 \(`https://pharo.org/download`\) running on the latest compatible Virtual machine. You can get them at the following address: `http://www.pharo.org/`

To load Bloc, execute the following snippet in a Pharo Playground:

```
[ Metacello new
	baseline: 'Bloc';
	repository: 'github://pharo-graphics/Bloc:master/src';
	onConflictUseIncoming;
	ignoreImage;
	load ]
		on: MCMergeOrLoadWarning
		do: [ :warning | warning load ]
```


### Loading the Memory Game

To make the demo easier to follow and help you if you get lost, we already made a full implementation of the game. You can load it using the following code:

```
Metacello new
    baseline: 'BlocMemoryTutorial';
    repository: 'github://pharo-graphics/Bloc-Memory-Tutorial/src';
    load
```

After you have loaded the MemoryTutorial project, you will get two new packages: `Bloc-Memory` and `Bloc-MemoryGame-Tests`. `Bloc-Memory-Tests` contains the full implementation of the game. 

You can browse a model of the game just executing the following code snippet:

```
MGGame withEmoji 
```

To get a working game just execute the following expression.

```
MGGameElement openWithNumber
```

Since we give you all the code, if you want to write it by your own, use a different prefix for the classes.

