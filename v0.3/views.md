# View Hierarchy

With Famo.us v0.3 your code will be mostly broken into classes that extend View, and files that contain that class,
which are required in to their containing classes.  This helps to keep the code modular.

The scene graph can be visualized as a tree of nodes. I use a star* to denote a context, a circle() to denote an
instance of View (or a subclass of View), a triangleA to denote a Modifier, a rectangle[] to denote a Surface, and
curly braces{} to denote a collection of sub-trees.

This graph is then decorated with pipes, showing event flow. Events only flow up or horizontally. Parents, because
they have references to their children, should not emit events that are heard by their children.

To create a standard iOS-style app there are several layout considerations. My demo usually focuses on a twitter-style
app with a Header with settings and back buttons, a content pane with scrolling content and a footer with other nav
buttons.

To create this, the simplified scene graph would look something like:

                                                        * Context
                                                        |
                                                        () App - in charge of laying out content and shuttling message
                                                        |
                                                        () headerFooterLayout 
                                                       /|\
                                                      / | \
               Header () ----------------------------+  |  +-------------------------------() Footer
                      |                                 |                                  |
                HFL-x ()                                |                                  () SequentialLayout-x
                     /|\                                |                                  |
   Back Button RC   / | \  Settings RenderController    |                         { [], [], [], [] } Buttons
          () ------+  |  +------()                      |
          |           |         |                       |
          []          []        []                      |
   Back Button       Title    Settings Button           () Content
                                                        |
                                                        () ScrollView 
                                                        |
                                                { [], [], [], [] } Tweets

User-defined Views: App, Header, Content, Footer
User-defined Surfaces: BackButton, Title, SettingsButton, Tweet, FooterButton

In this rather simple diagram there are no instances of Modifier. This is because we have only used pre-composed
widgets and Views like HeaderFooterLayout, ScrollView, and SequentialLayout, which use specs(internal workings) or
Modifiers in their internal methods.

Note the RenderControllers used in the Header sub-tree. There will be instances where you want to show content
(a subtree of the scene graph) and some where you want to remove, or hide the content. You have two options when
removing content:
- Using CSS, either display: none; or opacity: 0;
- Using a RenderController, which can show and hide famo.us sub-trees programatically

If we want a more visually appealing interface, we will need to make heavier use of View and Modifier.

For example, if we want the back button to fly onto the screen from the left whenever the user makes a selection
on the page, changing the state of the app, and making a "back" route available, they should use Modifiers to move
the subtree (in this case a surface).  This might make the Back button subtree look like:

                                                       /
                                                      () backButtonRenderController
                                                      |
                                                      A  translationModifier
                                                      |
                                                      [] backButton

As it stands, the Header is the closest ancestor which subclasses View, so any logic that we might employ
to move the back button must be implemented in that class.  This is less than ideal, especially in the case of content,
which manages several surfaces, and any modifiers that we might put above them in the scene graph.  To fix this dilemma,
we can use another subclass of View, in this case, BackButton which will manage a RenderController instance, a Modifier
instance, and an instance of Surface.  Think of an instance of View as a wrapper for the subtree, encapsulating its 
behavior.  We did the same thing at the hightest level of the scene graph, attaching App to the context.  App subclasses
View, giving us a layer of abstraction between the main context and the surface leaf nodes. The updated BackButton sub-
tree might look like:
                                                        /
                                                       () backButtonRenderController
                                                       |
                                                       () BackButton
                                                       |
                                                       A  translationsModifier
                                                       |
                                                       [] backButtonSurface

If the change seems small, it is, at least to the JS runtime in question. However, these layers of abstraction allow
the developer to focus on one level of the scene graph at a time, or one link at a time, greatly encapsulating the
complexity of a sizeable app.

