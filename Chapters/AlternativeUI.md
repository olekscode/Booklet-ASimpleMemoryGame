## Alternative graphical elements


While in the previous chapters we used a PNG for the back of the card, in this chapter 
we show alternative solutions: (1) that we can also draw using a low-level API such as the one proposed by the Alexandrie canvas or (2) compose elementary `BlElement`s.

### Using Alexandrie Canvas

```
Clock >> aeDrawOn: aeCanvas [
	"draw clock tick on frame"

	super aeDrawOn: aeCanvas.

	aeCanvas setOutskirtsCentered.

	0 to: 11 do: [ :items |
		| target |
		target := (items * Float pi / 6) cos @ (items * Float pi / 6) sin.

		items % 3 == 0
			ifTrue: [
				aeCanvas pathFactory: [ :cairoContext |
					cairoContext
						moveTo: center;
						relativeMoveTo: target * 115;
						relativeLineTo: target * 35;
						closePath ].

				aeCanvas setBorderBlock: [
					aeCanvas
						setSourceColor: Color black;
						setBorderWidth: 8 ] ]
			ifFalse: [
				aeCanvas pathFactory: [ :cairoContext |
					cairoContext
						moveTo: center;
						relativeMoveTo: target * 125;
						relativeLineTo: target * 25;
						closePath ].

				aeCanvas setBorderBlock: [
					aeCanvas
						setSourceColor: Color black;
						setBorderWidth: 6 ] ].
		aeCanvas drawFigure ]
]
```


### Using Element to add a cross

Now we are ready to define the backside of our card. We will start by drawing a line. To draw a line we should use the `BlLineGeometry`. At the end, we will create two lines and therefore two elements with a line geometry that will be added as children of the card Element.

Bloc uses parent-child relations between its elements thus leaving us with trees of elements where each node is an element, connected to a single parent and with zero to many children

A line is obviously defined between two points, we then need to give two points as parameters of the `from:to:` message from the `BlLineGeometry` class. 
Lines created using BlLineGeometry are a bit special as considered as "open geometries" meaning we don't define their color with the usual `background:` message like any other `BlElement`. Instead we define a border for our line and give this border the color we wanted (here we chose light green), we also define the thickness of our line with the border's width.
Another particularity of open geometries is that they don't fit well with default outskirts in the current verision of Bloc, this is why we redefine them to be centered 

```
MGCardElement >> initializeFirstLine

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
MGCardElement >> drawBackSide

	self addChild: self initializeFirstLine.
	^ self
```

Once this method is defined, refresh the inspector and you should get a card as in Figure *@figOneLine@*.

![A rounded card with half of the cross.](figures/CardOneLine.png width=60&label=figOneLine)

### Full cross


Now we can add the second line to build a full cross. 

```
MGCardElement >> initializeSecondLine

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
MGCardElement >> initializeBackSide

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

