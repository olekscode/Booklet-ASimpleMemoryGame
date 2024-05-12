## Adding game interactions

In this chapter, we will add interaction to the game. We want to flip the cards by clicking on them. 
Bloc supports such situations using two mechanisms: on one hand, event listeners handle events
and on the other hand, the communication between the model and view is managed via the registration to announcements sent by the model.

### Adding events and event listeners

In Bloc, there is of course plenty of Events and we will focus on `BlClickEvent`. 
We can also say that events are easily managed through event handlers 

Now we should add an event handler to each card because we want to know which card will be clicked and pass this information to the game model.


```
MGGameElement >> initialize
	super initialize.
	self initializeBackElement.
	self initializeFrontElement.
	self size: self cardExtent.
	self layout: BlLinearLayout new alignCenter.
	self background: self backgroundPaint.
	self geometry: (BlRoundedRectangleGeometry cornerRadius: 12).
	self card: (MGCard new symbol: $a).
	self addEventHandlerOn: BlClickEvent do: [ :anEvent | self click ]
```

We can easily see that whenever our card Element will receive a click Event, we will send the `click` message to this element



### Defining a click method

Now we can specialize the `click` method as follows: 
- We tell the model we just chose this card.
- We switch the card visual according to its card state.


```
MGCardElement >> click
	self parent memoryGame chooseCard: self card.
	self showCardFace
```


It means that the memory game model is changed but the cards don't flip back after mistaking the symbols. Indeed this is normal. We never made sure that visual elements were listening to model changes except for when we click on it.  This is what we will do in the following . 


### Alternate design 

Alternatively to use `self addEventHandlerOn: BlClickEvent do: [ :anEvent | self click ]`, we can define a specific event listener and reuse it over all cards. In this case we can reuse the 444 event handler for all card elements. It allows us to reduce overall memory consumption and improve game initialization time.


```
BlEventListener << #MGCardEventListener
	slots: { #memoryGame };
	package: 'Bloc-Memory'
```

```
MGCardEventListener >> clickEvent: anEvent
	memoryGame chooseCard: anEvent currentTarget card
```

```
MGGameElement >> memoryGame: aGame

	| aCardEventListener |
	game := aGame.
	aCardEventListener :=
		MGCardEventListener new
			memoryGame: aGame;
			yourself.

	self layout columnCount: game gridSize.

	game availableCards do: [ :aCard | 
		| cardElement |
		cardElement :=
			MGCardElement new
				card: aCard;
				addEventHandler: aCardEventListener;
				yourself.
		self addChild: cardElement ]
```

### Connecting the model to the UI

We show how the domain communicates with the user interface: the domain emits notifications
using announcements but it does not refer to the UI elements. It is the visual elements that should register to the notifications and react accordingly. We can prepare the message that will tell our elements to disappear we both cards match, otherwise we just tell our cards to flip back and draw their backside.

```
MGCardElement >> onDisappear
	"nothing for now"
```

Now we can modify the setter so that when a card model is set to a card graphical element, we register to the notifications emitted by the model. 
In the following methods, we make sure that on notifications we invoke the method just defined. 

```
MGCardElement >>card: aCard
	card := aCard.
	self fillUpFrontElement.
	self showCardFace.
	
	card announcer
		when: MGCardFlippedAnnouncement
		send: #showCardFace to: self;
		when: MGCardDisappearAnnouncement
		send: #onDisappear to: self
```

### Handling disappear

There are two ways to implement the disappearance of a card:
Either setting the opacity of the element to 0
(Note that the element is still present and receives events.)

```
MGCardElement >> onDisappear
	self opacity: 0
```

Or changing the visibility as follows:

```
MGCardElement >> disappear
	self visibility: BlVisibility hidden
```

Note that in the latter case, the element no longer receives events. It is used for layout. 

![Selecting two cards that are not in pair.](figures/BoardMissedPair.png width=50&label=figBoardMissedPair)


### Reminder on missed pair

Remember that when the player selects two cards that are not a pair, we present the two cards as shown in Figure *@figBoardMissedPair@*.
Now clicking on another card will flip back the previous cards. 
In addition, a card raises a notification when flipped in either direction. 

```
MGCard >> flip
	flipped := flipped not.
	self notifyFlipped
```


In the method `resetStep` we see that all the previous cards are flipped (toggled).

```
MGGame >> resetStep
	| lastCard |
	lastCard := self chosenCards  last.
	self chosenCards 
		allButLastDo: [ :aCard | aCard flip ];
		removeAll;
		add: lastCard
```

### Conclusion

At this stage, you are done for the simple interaction. Future versions of this document will explain how to add animations.
