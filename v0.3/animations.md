# Implementing Animations with Modifiers and Transitionables

The power of famo.us lies in its abstraction of the 16-element transform matrix which is applied to surfaces by the
Engine. The framework gives you helper libraries like Transform, Modifier, and Transitionable to create powerful
animations.

To implement the BackButton flying in from the left side of the screen, the simplest method would be to use a
StateModifier. StateModifier is a great class to use when you know where something will be or how it will be positioned
at any given time. In BackButton:

```js
function BackButton(options) {
    View.call(this, options); 

    this.translationsModifier = new StateModifier();
    this.backButtonSurface = new Surface({
        classes: ['back-button']
        content: 'back'
    });

    this.translationsModifier.add(this.backButtonSurface);
    this.add(this.translationsModifier);
}

// some more code here ...

BackButton.prototype.flyInFromLeft = function flyInFromLeft(offset, transition, callback) {
    offset = offset || 1000; // the amount of pixels offset the back button surface will start
    this.translationsModifier.setTransform(Transform.translate(-offset, 0, 0));

    transition = transition || {duration: 800};
    this.translationsModifier.setTransform(Transform.translate(0, 0, 0), transition, callback);
};
```

However, a more powerful implementation is to build the animation with the more atomic Transitionable class.
A transitionable should only be thought of as a container of a value with tweening capabilities. The simplest
transitionable contains a number and tweens to another number.

```js
var position = new Transitionable(0);
position.set(1, {duration: 800}, function() { alert('I got there'); });
```

Transitionables also give you an optional callback, like StateModifier, once the animation is completed. This is
unneccessary though. The real power of transitionables is setting and retrieving their values every frame. This allows
the developer to control animations with real-time data. Say, for instance, that you want a surface to have a rotation
based on a value delivered via a socket.io-flavored websocket. You might set up that interaction like this:

```js
var angle = new Transitionable(0);

// establish the socket connection, ...

socket.on('update angle', function(theta) {
    angle.set(theta);
});

// And in your famo.us code ...

rotationModifier.transformFrom(function() {
    return Transform.rotate(0, angle.get(), 0);
});
```

Note that because require.js and browserify module loaders can export singleton objects via their exports and require
statements, one Transitionable could be the export of a data-source file that is then required into other source
files. In that way several objects could implement some interaction based on the value of a transitionable singleton.

```js
// dataSource.js

    module.exports = new Transitionable(0);

// foo.js

    var data = require('/path/to/dataSource');
    
    // read the value
    doSomethingWith(data.get());

    // change the world
    data.set(1);
```

