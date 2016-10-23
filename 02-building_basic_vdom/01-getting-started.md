# Building our own Virtual DOM

In this section we will create our own Virtual DOM. However, this will be the most 
simplified version there is :smile:. 

Please note that from now one we assume that we implement a `vDOM` in order to
create a native `DOM`. Of course we can create a `vDOM` without mapping it to the native `DOM`
but that would make it pretty useless. But it would be a waste of both our times to keep repeating
this. 

#### Why?

React.js is pretty complex and I personally found the the whole `vDOM` implementation
also a bit hard to wrap my head around. My hope is that if we first build a very simple
implementation of a `vDOM` it is easier to understand when combining it with other concepts. 

#### Quick recap

In the previous sections we've learned some of the basics of the vDOM. 
The vDOM is constructed out of `vNodes` and determines the shape of the DOM. The vDOM cannot be displayed by the browser directly, the vDOM must be **mounted**. 

## Let's build! :rocket:

In our first version we will create a vDOM with a whopping number of one node! 
We will the methods that will create, *mount* and render our `vElement`

#### Getting started
If you want to program along. I've prepared a simple project which 
you can clone from: `https://github.com/swennemans/building-react-js`. 

After you've cloned the repo, you need to install the dependencies
and start the development server. 

```
npm install
npm run start //starts webpack-dev-server at localhost:4000
```

## The `createElement` method

If you use React.js without JSX you have to use the `React.createElement()` method every time 
you want to create an `ReactElement`. 
In React and using just JavaScript the equivalent of the JSX code `<div></div>` would be 
`React.createElement('div')`. In our implementation we will leave JSX for what it is,
and use JavaScript only. Therefore we need to define a similair method as React's `createElement()` 
method. At this stage we ar only interested in creating `vElement`'s and thus 
we define the method as `createVElement`.  The implementation is very minimal, 
and we only allow a `tag` and `className`. 

```javascript
index.js

//create a vElement
function createVElement(tag, config) {
  const { className } = config;

  return {
    tag: tag,
    className: className
  }
}
```
 
Awesome, now we can create `vElement`'s. But on their own their not particulary useful.
Actually they are **useless**. 

We need to **mount** them in the `DOM` to make them **usefull**! Let's define the method we want to use. We want to mount a `vElement` 
let's call this method `mountVElement`, not very creative I know.  

```javascript
index.js

function mountVElement(vElement, DOM) {
  const { tag, className } = vElement;
  
  //create a native DOM node
  const domNode = document.createElement(tag);

  //for later reference save the DOM node
  //on our vElement
  vElement.dom = domNode;

  //add className to native node
  if (className !== undefined) {
    domNode.className = className;
  }
  
  
  //Append domNode to the DOM
  DOM.appendChild(domNode)
}
```
 
Now we have a function that allows use to mount our `VElement` in the DOM. 
As you can see nothing is magical is happening. The `vElement` is basicly a 
blueprint for the `domNode`. It's important to note that we attaching a native
`node` to our `vElement` in the line: `vElement.dom = domNode;`. It's sort of like
putting a beating heart in `vElement`. We will look at this in more detail later, but there
is already a big hint, think a bit how we're adding a className to the `domNode`.

 
 
### Create our first app!

At this stage we can create an app. Well it's not actually an app because it's static. But we
call it that anyways. It's the most simplistic app ever. 

```javascript
index.js

...

//get native DOM, For now let's use the body;
const root = document.body;
//create vElement
const myApp = createVElement('div', { className: 'my-class' });
//mount in DOM

mountVElement(myApp, root);

```
If you inspect the `DOM` in your developer tools you should see a `div` element with the class `my-class`. 

## AWESOME! :smile: You've just build a Virtual Dom. 

Granted the implementation is extremely simple. Nontheless is a great starting point. At this point
we're very motivated and are already looking further. How could we build complexer applications? An application
with any x number of `nodes`? And how would we create this application in an efficient manner? It turns out
**recursion** is our friend!

