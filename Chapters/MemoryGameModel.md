## Game model insights

Before starting with the actual graphical elements, we first need a model for our game.
This game model will be used as the Model in the typical Model View architecture.
On the one hand, the model does not communicate directly with the graphical elements;
all communication is done via announcements. On the other hand, the graphic elements 
communicate directly with the model.

In the remainder of this chapter, we describe the game model in detail. If you want to move directly to
building graphical elements using Bloc, this model is fully defined in the package.


### Reviewing the card model

Let us start with the card model: a card is an object holding a symbol to be displayed, a state representing whether it is flipped or not, and an announcer that emits state changes. This object could also be a subclass of Model which already provides announcer management. 

```
Object << #MGCard
	slots: { #symbol . #flipped . #announcer};
	tag: 'Model';
	package: 'Bloc-Memory'
```



After creating the class we define an `initialize` method to set the card as not flipped, together with several accessors:

```
MGCard >> initialize
	super initialize.
	flipped := false
```

```
MGCard >> symbol: aCharacter
	symbol := aCharacter
```

```
MGCard >> symbol
	^ symbol
```

```
MGCard >> isFlipped
	^ flipped
```

```
MGCard >> announcer
	^ announcer ifNil: [ announcer := Announcer new ]
```


### Card simple operations


Next we need two methods to flip a card and make it disappear when it is no longer needed in the game.

```
MGCard >> flip
	flipped := flipped not.
	self notifyFlipped
```


```
MGCard >> disappear
	self notifyDisappear
```


### Adding notification

The notification is implemented as follows in the `notifyFlipped` and `notifyDisappear` methods. 
They simply announce events of type `MGCardFlippedAnnouncement` and `MGCardDisappearAnnouncement`. 
The graphical elements have to register subscriptions to these announcements as we will see later.

```
MGCard >> notifyFlipped
	self announcer announce: MGCardFlippedAnnouncement new
```


```
MGCard >> notifyDisappear
	self announcer announce: MGCardDisappearAnnouncement new
```


Here, `MGCardFlippedAnnouncement` and `MGCardDisappearAnnouncement` are subclasses of `Announcement`.

```
Announcement << #MGCardFlippedAnnouncement
	tag: 'Events';
	package: 'Bloc-Memory'
```


```
Announcement << #MGCardDisappearAnnouncement
	tag: 'Events';
	package: 'Bloc-Memory'
```


We add one final method to print a card more nicely and we are done with the card model!

```
MGCard >> printOn: aStream
	aStream
		nextPutAll: 'Card';
		nextPut: Character space;
		nextPut: $(;
		nextPut: self symbol;
		nextPut: $)
```

In addition, we can extend the inspector to display information in the inspector.


```
MGCard >> inspectCard: aBuilder
	<inspectorPresentationOrder: 1 title: 'card'>
	| builder |
	builder := StSimpleInspectorBuilder on: aBuilder.
	builder key: 'card:' value: self symbol.
	^ builder table
```


### Reviewing the game model

The game model is simple: it keeps track of all the available cards and all the cards currently selected by the player. 

```
Object << #MGGame
	slots: { #availableCards . #chosenCards};
	tag: 'Model';
	package: 'Bloc-MemoryGame'
```


The `initialize` method sets up two collections for the cards.

```
MGGame >> initialize
	super initialize.
	availableCards := OrderedCollection new.
	chosenCards := OrderedCollection new
```

```
MGGame >> availableCards
	^ availableCards
```

The `chosenCards` collection will hold at max two cards in this version of the game. 

```
MGGame >> chosenCards
	^ chosenCards
```


### Grid size and card number

For now, we'll hardcode the size of the grid and the number of cards that need to be matched by a player.
Later this could be turned into an instance variable and be configured.

```
MGGame >> gridSize
	"Return grid size"
	^ 4
```

The method `matchesCount` indicates that two identical cards are needed to match. 

```
MGGame >> matchesCount
	"How many chosen cards should match for them to disappear"
	^ 2
```

```
MGGame >> cardsCount
	"Return how many cards there should be depending on grid size"
	^ self gridSize * self gridSize
```



### Initialization

To initialize the game with cards, we add an `initializeForSymbols:` method. 
This method creates a list of cards from a list of characters and shuffles it. 
We also add an assertion in this method to verify that the caller provided enough characters to fill up the game board.

```
MGGame >> initializeForSymbols: aCollectionOfCharacters

	aCollectionOfCharacters size = (self cardsCount / self matchesCount)
		ifFalse: [ self error: 'Amount of characters must be equal to possible all combinations' ].

	aCollectionOfCharacters do: [ :aSymbol |
		1 to: self matchesCount do: [ :i |
		availableCards add: (MGCard new symbol: aSymbol) ] ].
	availableCards := availableCards shuffled
```


### Game logic

Next, we define the method `chooseCard:`. It will be called when a user selects a card. 
This method is the most complex method of the model and implements the main
logic of the game. 

- First, the method makes sure that the chosen card is not already selected.
This could happen if the view uses animations that give the player the chance to click on a card more than once.
- Next, the card is flipped by sending it the message `flip`. 
- Finally, depending on the actual state of the game, the step is complete and the selected cards are either removed or flipped back.

```
MGGame >> chooseCard: aCard
	(self chosenCards includes: aCard) 
		ifTrue: [ ^ self ].
	self chosenCards add: aCard.
	aCard flip.
	self shouldCompleteStep
		ifTrue: [ ^ self completeStep ].
	self shouldResetStep
		ifTrue: [ self resetStep ]
```


##### Completed. 
The current step is completed if the player selects the right amount of cards and they all show the same symbol.
In this case, all selected cards receive the message `disappear` and are removed from the list of selected cards.

```
MGGame >> shouldCompleteStep
	^ self chosenCards size = self matchesCount 
		and: [ self chosenCardMatch ]
```

```
MGGame >> chosenCardMatch
	| firstCard |
	firstCard := self chosenCards first.
	^ self chosenCards allSatisfy: [ :aCard | 
		aCard isFlipped and: [ firstCard symbol = aCard symbol ] ]
```
Note that the logic of chosenCardMatch looks more complex than expected but it works with matches that require more than two cards. 

```
MGGame >> completeStep
	self chosenCards 
		do: [ :aCard | aCard disappear ];
		removeAll.
```


##### Reset.

The current step should be reset if the player selects a third card. This will happen when a player already
selected two cards that do not match and clicks on a third one. In this situation, the two initial cards will be
flipped back. The list of selected cards will only contain the third card.

```
MGGame >> shouldResetStep 
	^ self chosenCards size > self matchesCount
```

```
MGGame >> resetStep
	| lastCard |
	lastCard := self chosenCards  last.
	self chosenCards 
		allButLastDo: [ :aCard | aCard flip ];
		removeAll;
		add: lastCard
```



### Ready 


We are now ready to start building the game view.
