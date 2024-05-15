## Graphical alternatives

While in the previous chapters, we used a PNG for the back of the card, in this chapter 
we show alternative solutions: (1) draw using a low-level API such as the one proposed by the Alexandrie canvas or (2) compose elementary `BlElement`s.


### Using the Alexandrie Canvas

Alexandrie is a Cairo-based optimized canvas. It is the default canvas supported by Bloc.
Every element is ultimately drawn with Alexandrie and Cairo. Alexandrie abstracts the Cairo interface for low-level operations. 

Now you can also draw the visual aspect of a BlElement directly using Cairo operations. 
For this, it is enough to override the method `aeDrawOn:`.

We give an example of a `ClockElement`. 
It shows that the developer should use a cairo context provided by a factory and configure such context to draw elementary paths or other properties such as borders.


```
ClockElement >> aeDrawOn: aeCanvas
	"draw clock tick on frame"
	super aeDrawOn: aeCanvas.
	aeCanvas setOutskirtsCentered.
	0 to: 11 do: [ :items |
		| target |
		target := (items * Float pi / 6) cos @ (items * Float pi / 6) sin.
		items % 3 == 0
			ifTrue: [	aeCanvas pathFactory: [ :cairoContext |
					cairoContext
						moveTo: center;
						relativeMoveTo: target * 115;
						relativeLineTo: target * 35;
						closePath ].
				aeCanvas setBorderBlock: [
					aeCanvas
						setSourceColor: Color black;
						setBorderWidth: 8 ] ]
			ifFalse: [ aeCanvas pathFactory: [ :cairoContext |
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
```

### Using simple BlElements

We can also compose `BlElement`s and place them as children of the main element. 
As such they form a tree of objects. 
Here is for example the same `ClockElement`, this time defined using elementary `BlElement`s.

The `initClockFrame` method is invoked when initializing the object.
Developers use normal element API such as geometry and border to define the visual aspect.

```
lClock >> initClockFrame
	"draw small lines around clock frame"
	0 to: 11 do: [ :items |
		| target |
		target := (items * Float pi / 6) cos @ (items * Float pi / 6) sin.
		items % 3 == 0
			ifTrue: [ self addChild: (BlElement new
					geometry: (BlLineGeometry from: center + (target * 115) to: center + (target * 150));
					outskirts: BlOutskirts centered;
					border: (BlBorder paint: Color black width: 8)) ]
			ifFalse: [ self addChild: (BlElement new
					geometry: (BlLineGeometry from: center + (target * 125) to: center + (target * 150));
					outskirts: BlOutskirts centered;
					border: (BlBorder paint: Color black width: 6)) ] ]
```

### Using elements to add a cross

We can apply the same technique that presented in previous section
to define the backside of our card. We start by drawing a line.
To draw a line we should use the `BlLineGeometry`. 
At the end, we create two lines and therefore two elements with a line geometry that are added as children of the `MGCardElement`.

Bloc uses parent-child relations between its elements thus leaving us with trees of elements where each node is an element, connected to a single parent and with zero to many children.

A line is defined between two points given as parameters of the `from:to:` message of the `BlLineGeometry` class. 
Lines created using `BlLineGeometry` are a bit special because they are considered as "open geometries" meaning we don't define their color with the usual `background:` message like any other `BlElement`. Instead, we define a border for our line and give this border the color we wanted (here we chose light green), we also define the thickness of our line with the border's width.

Another particularity of open geometries is that they don't fit well with default outskirts this is why we redefine them to be centered 

```
MGCardElement >> buildFirstLine
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
The message `when:do:` is used here to wait for the line parent to be drawn for the line to be defined, otherwise the `line parent extent` will be `0@0` and our line will not be displayed. 


### Full cross

Now we can add the second line to build a full cross. 

```
MGCardElement >> buildSecondLine

	| line |
	line := BlElement new
		        border: (BlBorder paint: Color lightGreen width: 3);
			geometry: BlLineGeometry new;
		        outskirts: BlOutskirts centered.
	line when: BlElementLayoutComputedEvent do: [ :e |
		line geometry from: 0 @ line parent height to: line parent width @ 0 ].
	^ line
```

We redefine the back side element using the two lines. 
```
MGCardElement >> buildBackSide

	backElement := BlElement new
		addChildren: { self buildFirstLine .  self buildSecondLine };
		constraintsDo: [ :c |
			c horizontal matchParent.
			c vertical matchParent ]
```

Then we make sure that we invoke the `buildBackSide` method in the card creation method `card:`.
 

```
MGCardElement >> card: aCard
	card := aCard.
	self fillUpFrontElement.
	self buildBackSide.
	card isFlipped
		ifTrue: [ self showFrontFace ]
		ifFalse: [ self showBackFace ]
```

Once this method is defined, refresh the inspector and you should get a card as in Figure *@crossedfigOneLine@*.

![Using crossed back side cards.](figures/crossedmemoryExample0.png width=60&label=crossedfigOneLine)

The backside is then an `BlElement` holding both lines, we tell this element to match its parent using constraints, meaning the element size will scale according to the parent size, this also makes our lines defined to the correct points. 

### Conclusion

We show that you can have different ways to define a graphical representation of your element. 


