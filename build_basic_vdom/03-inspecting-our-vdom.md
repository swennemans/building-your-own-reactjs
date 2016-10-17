# Inspecting our vDOM 🕵

In the last section we've mostly focussed on building the `DOM`. Now it's time to give our `vDOM` some love. 
Let's expose our `myApp` variable globally so that we can play with it. 

```javascript
index.js
...

global.myGlobalApp = myApp

```

In our console we can inspect our `myGlobalApp` variable, and see that it's an Object with the by us defined properties. 
You will see that the props object holds an array of of `vElements` that may also hold an array of `vElement`s etcetera. 
our `vDOM` is thus also build as a tree, just as the `DOM`. Cool, exactly what we wanted :grin:.

## Manipulation 

Even though we're skipping some fundamentals (like actually displaying something useful in the browser). 
I'd like to point our focus on the `dom` property that every `vElement` holds. 

If you inspect this property you will find that each of this property holds an reference to an actual `DOM` `node`. 
It is basicly a small `DOM` snippet which we can manipulate. 

```javascript
myGlobalApp.dom // returns "my-class"

//let's change some inline styles!
myGlobalApp.dom.style = 'height: 100px; width: 100px; background-color: red';

//DOM is updated
```

Try it! What happens? Did you expect this?

In some sense this is a little scary. We've all heard (and experienced) the great dangers of mutability. 
But the `DOM` will do whatever it damn well pleases and we have to work with that. And we do, it's
actual pretty cool right? Our `vDOM` holds `vNodes` that have reference to the actual specific `DOM node`. This also
means that we can do our magic on the `vDOM` and **only** update the relevant `DOM node`. It feels as if
we have reclaimed the power and made the `vDOM` boss (again) 💪

