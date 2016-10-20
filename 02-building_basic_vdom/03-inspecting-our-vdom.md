# Inspecting our vDOM 🕵

In the last section we've mostly focussed on building the `DOM`. Now it's time to give our `vDOM` some love. 
Let's expose our `myApp` variable globally so that we can play with it. 

```javascript
index.js
...

global.myGlobalApp = myApp

```

In our console we can inspect our `myGlobalApp` variable, and see that it's an `object` with the, by us, 
defined properties. You can see that the props object holds an array of of `vElements` 
that may also hold an array of `vElement`s etcetera.  Our `vDOM` is, as you can see, build as a tree, just as the `DOM`. 
Cool, exactly what we wanted :grin:

## Manipulation 

Even though we're skipping some fundamentals (like actually displaying something useful in the browser). 
I'd like to point our focus on the `dom` property that every `vElement` holds. 

If you inspect this property you will find that each of this property holds an reference to an actual `DOM` `node`. 
It is basicly a small `DOM` snippet which we can manipulate. Do you remember when I said
that this property is sort of like a beating heart? It's aliveee: 

```javascript
myGlobalApp.dom // returns "my-class"

//let's try something weird:
myGlobalApp.dom.style = 'height: 100px; width: 100px; background-color: red';

```
Try it! What happens? Did you expect this?

In some sense this is a little scary. We've all experienced and heard about the great dangers of mutability. 
But the `DOM` will do whatever it damn well pleases and we have to work with that. And we do, it's
actual pretty cool right? Our `vDOM` holds `vNodes` that have reference to the actual specific `DOM node`. This also
means that we can do our magic on the `vDOM` and **only** update the relevant `DOM node`. It feels as if
we have reclaimed the power and made the `vDOM` boss (again) 💪
