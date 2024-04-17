## Alternative graphical elements


While in previous chapter we used a Png for the back of the card, in this chapter 
we show that we can also compose multiple BlElements


### Adding a cross

Now we are ready to define the backside of our card. We will start by drawing a line. To draw a line we should use the `BlLineGeometry`. At the end, we will create two lines and therefore two elements with a line geometry that will be added as children of the card Element.

Bloc uses parent-child relations between its elements thus leaving us with trees of elements where each node is an element, connected to a single parent and with zero to many children

A line is obviously defined between two points, we then need to give two points as parameters of the `from:to:` message from the `BlLineGeometry` class. 
Lines created using BlLineGeometry are a bit special as considered as "open geometries" meaning we don't define their color with the usual `background:` message like any other `BlElement`. Instead we define a border for our line and give this border the color we wanted (here we chose light green), we also define the thickness of our line with the border's width.
Another particularity of open geometries is that they don't fit well with default outskirts in the current verision of Bloc, this is why we redefine them to be centered 

```
MgdRawCardElement >> initializeFirstLine

	| line |
	line := BlElement new
		        border: (BlBorder paint: Color lightGreen width: 3);
			geometry: BlLineGeometry new;
		        outskirts: BlOutskirts centered.
	line
		when: BlElementLayoutComputedEvent
		do: [ :e | line geometry from: 0 @ 0 to: line parent extent ].
	^ line
```
The message `when:do:` is used here to wait for the line parent to be drawn for the line to be defined, otherwise the `line parent extent` will be 0@0 and our line will not be displayed. 

We can redefine `drawBackSide` and add the line we just created.

```
MgdRawCardElement >> drawBackSide

	self addChild: self initializeFirstLine.
	^ self
```

Once this method is defined, refresh the inspector and you should get a card as in Figure *@figOneLine@*.

![A rounded card with half of the cross.](figures/CardOneLine.png width=60&label=figOneLine)

### Full cross


Now we can add the second line to build a full cross. We will add another instance variable holding our back side so that the lines are created only once during initialization. Our solution is defined as follows: 

```
BlElement << #MgdRawCardElement
	slots: { #card #backSide };
	package: 'Bloc-MemoryGame-Demo-Elements'
```
```
MgdRawCardElement >> backSide: aBlElement
	backside := aBlElement
```

Before creating the getter of backside, we will create the method that will be responsible for adding the two lines together as a cross `initializeBackSide`. Let's start by creating the second line the same way we created the first one.


```
MgdRawCardElement >> initializeSecondLine

	| line |
	line := BlElement new
		        border: (BlBorder paint: Color lightGreen width: 3);
			geometry: BlLineGeometry new;
		        outskirts: BlOutskirts centered.
	line when: BlElementLayoutComputedEvent do: [ :e |
		line geometry from: 0 @ line parent height to: line parent width @ 0 ].
	^ line
```
```
MgdRawCardElement >> initializeBackSide

	| firstLine secondLine cross |
	firstLine := self initializeFirstLine.
	secondLine := self initializeSecondLine.

	cross := BlElement new
		         addChildren: {
				         firstLine.
				         secondLine };
		         constraintsDo: [ :c |
			         c horizontal matchParent.
			         c vertical matchParent ].

	^ cross
```

Our backside is then an Element holding both lines, we tell this element to match its parent using constraints, meaning the element size will scale according to the parent size, this also makes our lines defined to the correct points. 

We can now define the backSide getter but with a little twist, using lazy initialization. This will create our element only when accessed the first time and not right after the initialization of the card element. This concept is very useful in certain situations and is great to know in case you need it, we define it as such :


```
MgdRawCardElement >> backSide
	^ backSide ifNil: [ self initializeBackSide ]
```

We can finally redefine `drawBackSide` and call it in our initialization to draw our backside when the card is created. 

```
MgdRawCardElement >> drawBackSide
	self removeChildren.
	self addChild: self backSide
```

```
MgdRawCardElement >> initialize

	super initialize.
	self size: 80 @ 80.
	self background: self backgroundPaint.
	self geometry:
		(BlRoundedRectangleGeometry cornerRadius: self cornerRadius).
	self card: (MgdCardModel new symbol: $a).
	self drawCardElement.
```


![A card with a complete backside.](figures/CardCross.png width=60&label=figCardCross)

Now our backside is fully implemented and when you refresh your view, you should get the card 
as shown in Figure *@figCardCross@*. 


### Flipped side 

Now we are ready to develop the flipped side of the card. To see if we should change the card model you can use the inspector to get the card element and send it the message `card flip` or directly 
recreate a new card  as follows: 

```
| cardElement | 
cardElement := MgdRawCardElement new.
cardElement card flip.
cardElement
```


You should get an inspector in the situation shown in Figure *@figCardForFlip@*.
Now we are ready to implement the flipped side. 

![A flipped card without any visuals.](figures/CardForFlip.png width=60&label=figCardForFlip)

Let us redefine `drawFlippedSide` as follows: 
- First, we create a text element that holds the symbol of the card, we also give properties to this text by changing the font of the text but also its size and its color.
- Then we add the text element as a child of our card element

We will add an instance variable 'flippedSide' to our `MgdRawCardElement` class so that we create the text only once during the initialization. We don't forget about getter and setter.

```
BlElement << #MgdRawCardElement
	slots: { #card #backSide #flippedSide };
	package: 'Bloc-MemoryGame-Demo-Elements'
```
```
MgdRawCardElement >> flippedSide

	^ flippedSide ifNil: [ self initializeFlippedSide ]
```

We can now create the method that will create the text for the flipped side, this method will be called during initialization.

```
MgdRawCardElement >> initializeFlippedSide

	| elt |
	elt := BlTextElement new text: self card symbol asRopedText.
	elt text fontName: 'Source Sans Pro'.
	elt text fontSize: 50.
	elt text foreground: Color white.
	^ elt
```

Now we can redefine `drawFlippedSide` to add our text element as a child of our card element

```
MgdRawCardElement >> drawFlippedSide

	self removeChildren.
	self addChild: self flippedSide
```


When we refresh the display we can see the letter 'a' appear but it is positioned in the top left corner of our element, just as shown in Figure *@figCardNotCentered@*

![Not centered letter.](figures/CardNotCentered.png width=60&label=figCardNotCentered)

Let's change that !
We will have to use constraints on our text element to tell him to get aligned in the center of its parent. To achieve this goal, we need to define a layout on the parent which is our card Element. We will use a Frame Layout that acts just like a real frame, meaning the text element will be able to get centered into the frame of its parent. Let's add the frame layout in the initialization of the card Element.

```
MgdRawCardElement >> initialize

	super initialize.
	self size: 80 @ 80.
	self background: self backgroundPaint.
	self geometry:
		(BlRoundedRectangleGeometry cornerRadius: self cornerRadius).
	self card: (MgdCardModel new symbol: $a).
	self drawCardElement.
	self layout: BlFrameLayout new.
```

We can now add the constraints to the text element.

```
MgdRawCardElement >> initializeFlippedSide

	| elt |
	elt := BlTextElement new text: self card symbol asRopedText.
	elt text fontName: 'Source Sans Pro'.
	elt text fontSize: 50.
	elt text foreground: Color white.
	elt constraintsDo: [ :c |
		c frame horizontal alignCenter.
		c frame vertical alignCenter ].
	^ elt
```


With this definition, we get a centered letter as shown in Figure *@figCardCentered@*.

![Centered letter.](figures/CardCentered.png width=60&label=figCardCentered)

Now we are ready to work on the board game.
