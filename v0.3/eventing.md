# Eventing

When developing a famo.us app, the developer must be able to encode interactions between the different parts of the app. 
In our previous example, we designed an instance of view, BackButton, as separate from the Header so that we could 
encapsulate its functionality. However, the header must know when the BackButton has been clicked, and we can manage this
with events. Famo.us has its own eventing library as a consequence of how it lays out HTML. Because of the limitations of
the browser's support for nested preserve-3d in CSS, the DOM is mostly flat in famo.us. This prevents us from using the
DOM's eventing API.

As a best practice, 'children' emit events that are heard by their parents, and parents call methods on their children, as
they have references to their children as objects.  Instances of View contain two EventEmitter objects, called \_eventInput
and \_eventOutput. Once you pipe a child's events to it's parent, they can be emitted by the child's \_eventOutput and heard
on the parent's \_eventInput.

```js
var parent = new View();
var child = new View();

child.pipe(parent);

parent._eventInput.on('foo', function(data) {
    alert(JSON.stringify(data));
});
child._eventOutput.emit('foo', {bar: 'baz'}); // will cause an alert
```

While parent and child are instances of View, not EventEmitter, there is some sugar put into View to allow this syntax.
The following six lines produce the same result, and are used interchangeably for different symantic effect.
If for some reason you were to create your own objects with EventEmitter members that are not instances of View, this
knowledge could be handy.

```js
child.pipe(parent);
child._eventOutput.pipe(parent);
child._eventOutput.pipe(parent._eventInput);
parent.subscribe(child);
parent._eventInput.subscribe(child);
parent._eventInput.subscribe(child._eventOutput);
```

While the \_eventInput and \_eventOutput are available as properties of any view object, they are preceeded by the 
underscore to denote they are private members of those objects, and should only be accessed in their class definition.
The previous example, written for simplicity, is more correctly written:

```js
function Parent() {
    View.call(this);
    this._eventInput.on('foo', function(data) {
        alert(JSON.stringify(data));
    });
    this.child = new Child();
    this.child.stopTheWorld();
}
function Child() {
    View.call(this);
}
// set up the prototype chain correctly for Parent and Child to inherit View...
Child.prototype.stopTheWorld = function stopTheWorld() {
    this._eventOutput.emit('foo', {bar: 'baz'}); // will cause an alert
};
```

In a more practical example, as we set up in the previous lesson, views.md, when some content changes in the app,
the back button should appear to allow the user to navigate back. This can be set up rather easily. The most correct
implementation is that events travel from children, up through their parent, until they reach a common ancestor of
their target, then that ancestor calls a method on the uncle of the originating child until the target gets its method
called.  Thus events flow up and method calls flow down the render tree.

Tweet (!)-> Content (!)-> App (.)-> Header (.)-> BackButton

When a tweet is clicked, it emits a click event to its parent, Content. The Content can then change what it renders.
More importantly, it will emit an event to App, saying that the content changed, with enough data that the App knows
to show the back button. It will then call a method on Header to show the BackButton and update the title. The Header
will then call a method on BackButton to show the surface in the view. Then once the BackButton is clicked, the process
will happen again in reverse:

BackButton (!)-> Header (!)-> App (.)-> Content (.)-> Tweet

Some implementation of this might look like:
```js
//header.js

function Header() {
    // ...
    this.backButton = new BackButton();
    this._eventInput.on('backButtonClicked', function() {
        this._eventOutput.emit('navigateBack');
    });
}

// ...

Header.prototype.changeState(options) {
    this.titleLabel.setContent(options.content);
    if (options.showBackButton) {
        this.backButton.flyInFromLeft();
    }
};

//backButton.js

function BackButton() {
    //...
    this.buttonSurface = new Surface(this.options.button);
    this.buttonSurface.on('click', function() {
        this._eventOutput.emit('backButtonClicked');
    });
}
```

