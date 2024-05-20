## Adding a board view

In the previous chapter, we defined all the card visualizations. We are now ready to define the game board visualization.
Basically, we will define a new `BlElement` subclass and set its layout.

Here is a typical scenario to create the game: we create a model and its view and we assign the model as the view's model.

```
game := MGGame withNumbers.
board := MGGameElement new.
board memoryGame: game. 
```


### The GameElement class

Let us define the class `MGGameElement` that will represent the game board. 
As for the `MGCardElement`, it inherits from the `BlElement` class. 
The instance variable `memoryGame` holds a reference to the game model.

```
BlElement << #MGGameElement
	slots: { #memoryGame };
	package: 'Bloc-MemoryGame'
```


We define the `memoryGame:` setter method. We will extend it to create
all the card elements shortly. 

```
MGGameElement >> memoryGame: aGameModel
	memoryGame := aGameModel
```


```
MGGameElement >> memoryGame
	^ memoryGame
```


During the object initialization, we set the layout (i.e., how sub-elements are placed inside their container).
Here we define the layout to be a grid layout with a little extra space around the card element and we set it as horizontal.

```
MGGameElement >> initialize
	super initialize.
 	self background: Color veryLightGray.
	self layout: (BlGridLayout horizontal cellSpacing: 20).
```


### Creating cards


When a model is set for a board game, we use the model information to perform the following actions: 
- we set the number of columns of the layout and
- we create all the card elements paying attention to set their respective model. 



```
MGGameElement >> memoryGame: aGameModel
	memoryGame := aGameModel.
	memoryGame availableCards
		do: [ :aCard | self addChild: (MGCardElement new card: aCard) ]
```

Note in particular that we add all the card graphical elements as children of the board game using the message `addChild:`.



![A first board - not really working.](figures/BoardStarted.png width=60&label=figBoardStarted)

```
game := MGGame withNumbers.
board := MGGameElement new.
board memoryGame: game. 
board
```

When we inspect the previous code snippet, we obtain a situation similar to the one of Figure *@figBoardStarted@*.
It shows that only a small part of the game is displayed. This is due to the fact that the board game element did not adapt to its children. 


### Updating the container to its children

A layout is responsible for the layout of the children of a container but not of the container itself. 
For this, we should use constraints. 

```
MGGameElement >> initialize
	super initialize.
	self background: Color veryLightGray.
	self layout: (BlGridLayout horizontal cellSpacing: 20).
	self
		constraintsDo: [ :aLayoutConstraints | 
			aLayoutConstraints horizontal matchParent.
			aLayoutConstraints vertical matchParent ]
```


Now when we refresh our view we should get a situation close to the one presented in Figure*@figBoardOneRow@*, i.e., having 
just one row. Indeed we never mentioned to the layout that it should layout its children into a grid, wrapping after four.

![Displaying a row.](figures/BoardOneRow.png width=60&label=figBoardOneRow)


### Getting all the children displayed


We modify the `memoryGame:` method to set the number of columns 
that the layout should handle. 

```
MGGameElement >> memoryGame: aGameModel
	memoryGame := aGameModel.
	self layout columnCount: memoryGame gridSize.
	memoryGame availableCards
		do: [ :aCard | self addChild: (MGCardElement new card: aCard) ]
```


Once the layout is set with the correct information we obtain a full board as shown in Figure *@figBoardFull2@*.

![Displaying a full board.](figures/BoardFull.png width=60&label=figBoardFull2)


Before adding interaction let's define a method `openWithNumber` that will open our game element with a given model.

```
MGGameElement class >> openWithNumber
	| aGameElement space |
	aGameElement := MGGameElement new
		memoryGame: MGGame withNumbers;
		yourself.
	space := BlSpace new.
	space root addChild: aGameElement.
	space root whenLayoutedDoOnce: [ space extent: 420 @ 420 ].
	space show
```

We are now ready to add interaction to the game. 


### Conclusion

The board is now ready to display the full game but the user interaction is still missing. 
This is what we will investigate in the next chapter.




