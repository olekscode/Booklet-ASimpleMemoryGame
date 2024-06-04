## Adding animations

In this chapter, we will add animations to the game. 
We will define animations and add them to the queue of card animation.
Then Bloc logic will execute the animation queue on the receiver.
This chapter will illustrate how animations can be composed or run in parallel.

### Card flipping animations

When the user flip the two cards we want to shrink them a bit to make them 
stand out. We use a simple scaling transformation.
An animation will modify the attributes (size, color, position...) of the element on which
it is applied.
Once the transformation is defined it is added to the animation queue of the 
receiver using the message `addAnimation:`. 

```
MGCardElement >> onFlippedFace
	| animation |
	animation := BlTransformAnimation scale: 0.85 @ 0.85.	
	animation
		absolute;
		easing: BlQuinticInterpolator default;
		duration: 0.3 seconds.
	self addAnimation: animation.
	self showFrontFace
```

We define another similar animation to put back the full size of flipped back card. 

```
MGCardElement >> onFlippedBack
	| animation |
	animation := BlTransformAnimation scale: 1@1.
	animation
		absolute;
		easing: BlEasing bounceOut;
		duration: 0.35 seconds.
	self addAnimation: animation.
	self showBackFace
```

We modify the `showCardFace` method to invoke the method performing the animations
prior to changing the visual of the cards. 

```
MGCardElement >> showCardFace
	card isFlipped
		ifTrue: [ self onFlippedFace ]
		ifFalse: [ self onFlippedBack ]
```

### Card disappearing animation
When two cards match we want them to enlarge a bit to get player's attention.
When a card disappears, we will compose several animations: one that will grow
the element, another that will change its opacity and in parallel its size. 

The opacity animation will slowly make the card transparent while the size will make 
it is smaller.

We replace the previous `onDisappear` method by the following one. 

```
MGCardElement >> onDisappear
	| vanish enlarge minimize disappear |
	enlarge := BlTransformAnimation scale: 1.15 @ 1.15.
	enlarge
		absolute;
		easing: BlEasing bounceOut;
		duration: 0.5 seconds.
	vanish := BlOpacityAnimation new
		          opacity: 0;
		          duration: 0.35 seconds.
	minimize := BlTransformAnimation scale: 0.01 @ 0.01.
	minimize
		absolute;
		easing: BlEasing linear;
		duration: 0.35 seconds.
	disappear := BlParallelAnimation withAll: { vanish . minimize }.

	self addAnimation: (BlSequentialAnimation withAll: { enlarge. disappear })
```

### Conclusion

This chapter presented how animations are simple to be defined in Bloc and how they are useful to enhance user experience. 
