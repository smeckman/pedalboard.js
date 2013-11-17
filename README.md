Pedalboard.js
====
####Checkout [pedals.io](http://pedals.io)! It's pedalboard.js packaged as a product you can really use.

Introduction
------

Ever wanted to have your pedal stack in the cloud, available anywhere you go without any hardware? Ever wanted to manage your sound as easily as browsing a web site? Ever wanted to share the perfect sound you created with your friends without the hassle? You loved the overdrive on that effects software but wanted to tweak its EQ a little bit?

Pedalboard.js is a ground-breaking, first of its kind, novel open-source JavaScript framework for developing audio effects and applying them to sound sources, and it's particularly good at guitar effects.

The API and all the abstraction is built around the concept of guitar effects — pedals and stomp boxes, pots and switches.

You design your pedal with the powerful Webkit audio API, attach pots and switches to it, style it via CSS3 and voila.

Bring multiple pedals together to create a pedalboard, easily adjust their settings and routing. Prepare as many pedalboards as you'd like, e.g. for your favorite styles. Easily switch pedalboards for a completely different sound.

Finally, a complete guitar effects stack, completely customizable, in your hands.

**Checkout the [demo](http://dashersw.github.com/pedalboard.js/demo). Works best with the latest Chrome and Safari. Make sure to try the real-time line-in feature**

###Motivation

Guitar effects software on the market are very nice indeed, but they somehow lack few important concepts — ease of use, portability, sharing stuff, customization, extensibility, etc.

What's worse, now that the freemium model is very popular, what looks like an easy initial buy turns into a small fortune when you'd like to buy the effects you wanted.

Pedalboard.js is an attempt to address these issues. With the help of the powerful audio API in WebKit, Pedalboard.js gives you a fairly-easy-to-build-upon abstraction and foundation.

------
Technology
======
Audio
------

The technology that made Pedalboard.js available is Webkit's implementation of the W3C audio API. Although in its final draft version (which means it's more or less stable — at least in concept and context, but subject to change) the API is quite extensive, easy to use, extendable and has very nice approaches.

Right now, the audio API is only available in the latest builds of WebKit-based browsers — Chrome and Safari. Mozilla is said to support it, with IE to follow suit.

###Nodes

Audio API has the "node" concept at its core. Everything is a node and each node can connect to other nodes. A node can be a *source* (*array* or stream or generated by an *oscillator* or even arbitrary JS functions) or an *effect* or an *output*; or yet again, an *analyzer* to poke at what's going on inside.

Effects are the most versatile offering of the API. You can create *gain*, *wave shaping*, *convolution* or *filter* effects, all of which have various settings you can tweak. With this much opportunities, it's quite easy to build the perfect tone you're after.

###State of the art

Currently, the AudioContext implementation in Webkit supports streaming audio from the inputs of your computer—line-in or microphone in only the nightly builds of Chrome Canary and soon will be a part of everyday Chrome and later Safari. In this context, the native implementation supports pre-recorded or generated sound buffers, as well as real-time input on Chrome Canary. This enables Pedalboard.js to fully function as a guitar effects stack. Experimentally, Pedalboard.js implements streaming from your input through a Flash proxy, too; but that's deprecated as of now, since the native version is already on the stage. Due to the hard-coded buffer size of 2048 samples in Flash, there's a latency of about 50ms, an audible delay. Unfortunately, there are no other workarounds to increase Flash stream quality; but as the native implementation rolls out to a browser near you, expect a reasonable buffer size and 0-latency input.

JavaScript
------

At its core, Pedalboard.js uses tartJS, an open source JavaScript framework developed at Tart New Media. tartJS is a library built upon Google Closure Library, which is another great library developed  by Google, Inc. Besides a great approach to object oriented programming and a vast amount of classes, Closure has a set of build tools that perfectly analyzes (read: lints) your code, and compiles and builds it.

Closure Library allows modular, object oriented JavaScript at its best, with tons of utility classes for DOM manipulation, visual effects, components, mathematics, arrays, objects, etc.

Closure Compiler not only minifies your code, but obfuscates it maximally, rewrites it to the letter to squeeze out the last bit of performance and file size. An entire web application can be under 30kb gzipped, which is quite insane.

------
Concepts
======

Pedalboard.js follows an object oriented coding style, with an analogy to the real world objects and concepts.

Classes are modeled after real components of pedals and pedal boards. You configure each component as you configure a real pedal stack.

You use your pedal board on stage, connect your guitar to it, and connect it to an output, like an amp or a mixer. Pedalboard.js follows the same basic ideas.

Stage
------

At the base is the Stage. Stage is where everything lives — what you see, the floor, the place that hosts your pedal board. The Stage has one input, an instance of an Input class — FileInput (for pre-recorded sounds) or StreamInput (for line-in). It also has one output, an instance of an Output class, which is your speakers. Stage also hosts a pedal board, an instance of a Board class. Of course, you can swap the board currently on stage with another one. Stage also has its own AudioContext instance and everything inside the Stage shares this context.

Stage is a manager class, in classical terminology.

Usage:

```js
var stage = new pb.Stage();
```

Board
------

Board is analogous to a pedal board, where you put and organize your pedals. It has an InputBuffer instance, an OutputBuffer instance and as many Pedal instances as you'd like. A Stage can hold only one Board at a single given time, but you can have many Boards and can swap them at your leisure. A stage connects its Input to the InputBuffer of the Board and its Output to the OutputBuffer of the Board.

Usage:

```js
var stage = new pb.Stage();
var board1 = new pb.Board(stage.getContext());
var board2 = new pb.Board(stage.getContext());

stage.setBoard(board1);
stage.setBoard(board2);
```

Pedal
------

Pedal component is where the magic happens. Given an AudioContext, Pedal implements its effects in an effects chain. Pedal instances also have an InputBuffer and an OutputBuffer for brevity. The effects lie in between these two, so when two pedals are cascaded, the output buffer of the first will be connected to the input buffer of the latter. This lets a pedal provide a clean and expectable interface to the outer world — the other pedals or the pedal board.

One should be able to tweak a pedals parameters easily, and may want to turn it off at some point.

Pedalboard.js offers two abstractions for this; Pot and Switch.

Usage:

```js
// initialize the stage and get the context
var stage = new pb.Stage();
var ctx = stage.getContext();

// initialize the board and pedals
var board = new pb.Board(ctx);
var od = new pb.stomp.Overdrive(ctx);
var reverb = new pb.stomp.Reverb(ctx);
var vol = new pb.stomp.Volume(ctx);

// add pedals to board
board.addPedals([od, reverb]);
board.addPedalsAt(1, vol);

// tweak pedal settings
od.setDrive(0.7);
od.setLevel(0.7);
reverb.setLevel(0.3);
vol.setLevel(0.2);

// set the board on stage and start playing!
stage.setBoard(board);
```

Pot
------

Pot component serves as a means to tweak a Pedal's parameters, such as gain, level, distortion, delay feedback, etc. It's analogous to a potentiometer, i.e. variable resistor.

LinearPot and LogPot are two classes that inherit from the Pot class, and provide two different potentiometer implementations. A pot has a value, and a multiplier. Value reflects a pedal's parameter value, such as gain. Multiplier is kind of like the resistance of a resistor. Optionally, you can define minimum, maximum and default values.

The Pot's setValue method requires a windowed range between 0 and 1, representing pot rotation. Values larger than 1 will be interpreted as 1 and lower than 0 will be interpreted as 0.

In an application where your parameter is required to be between 0 and 10, multiplier should be 10. When you need your parameter to have a range between 0 and 1000, your multiplier should be 1000. Optionally, you can just leave the multiplier as 1 and play with minimum and maximum values to achieve the same effect.

The only difference between a LinearPot and a LogPot is their algorithm of sweeping through their values. While LinearPot will just reflect setValue input times the multiplier, LogPot will calculate as log(setValue input) times 10 times multiplier.

Usage:

```js
pb.stomp.Overdrive.prototype.createPots = function() {
    var handler = goog.bind(this.model.setDrive, this.model);

    this.volumePot = new pb.pot.Linear(this.model.gain.gain, 'volume', 0.1); // this will automatically set the gain on change.
	this.drivePot = new pb.pot.Log(handler, 'drive', 200); // called with a function as the argument, this will call that function with its new value.
	this.pots = [this.drivePot, this.volumePot];

	this.drivePot.addEventListener('valueChanged', function(e) {
		this.model.setDrive(e.newValue); // we can explicitly listen to the change event and do things after a change.
	}, this);
};

// Later on, you can manually set a pot's value like
drivePot.setValue(0.3);
```

Switch
------

Switch class models a foot switch, used to toggle pedals or individual effects of pedals. A switch has an array of 3-node arrays and toggles between them when pressed. In Pedalboard.js, one defines any number of 3-node arrays and the connection will be handled by the switch.

The input should always be the middle node in a 3-node array. When *on*, a switch will connect the input to the first node. When *off*, a switch will connect the input to the last node. This is in fact modeled after a 3PDT switch, and functions the same.

Two classes, Toggle and Momentary, extend Switch and implement two different toggle mechanisms. Toggle is the standard switch, you press it once to turn it on, and it stays on until you press it again; it then turns off. Momentary is a little bit different, it's on as long as you press it and turns off as soon as you release it.

Led
------

Led component models an LED light commonly used in pedals to signal if it's on. A Box can have many LEDs, but one LED is standard in the Box component, that is connected to the Switch and signals if the pedal is bypassed.

The Led component requires a Switch instance to operate on and binds its events automatically. It will be on as long as the Switch is active.

Usage:

```js
var bypassSwitch = new pb.footswitch.Toggle();
var led = new pb.Led(this.bypassSwitch);
```
