# Building our own Virtual DOM

In this section we will create our own Virtual DOM. However, this will be the most 
simplified version there is :smile:. 

Please note that from now one we assume that we implement a `vDOM` in order to
create a native `DOM`. Of course we can create a `vDOM` without mapping it to the native `DOM`
but that would make it pretty useless. But it would be a waste of both our times to keep repeating
this. 

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
you create an `ReactElement`. 
In React using vanilla JavaScript the equivalent of the JSX code `<div></div>` would be 
`React.createElement('div')`. In our implementation we will leave JSX for what it is,
and use JavaScript only. Therefore we need to define a similair method.  
For now we only create `vElement`'s and thus we define the method as `createVElement`. 
The implementation is very minimal, and we only allow a `tag` and `className`. 

```javascript
index.js

//create a vElement
function createVElement(tag, className) {
 return {
   tag: tag,
   className: className
 }
}
```
 
 Awesome, now we can create `vElement`'s. But on their own their not particulary useful.
 Actually they are useless. We need to **mount** them in the `DOM`! 

```javascript
index.js

function mountVElement(vElement, DOM) {
  const { className, tag } = vElement;
  
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
 As you can see nothing is magical  is happening. The `vElement` 
 is basicly a blueprint for the `domNode`. Another thing to notice is that in the 
 `mountVElement` we're altering the `domNode` by adding a className on it. 
 Keep in mind that the `domNode` is mutable.

>In later steps we will use this mounting phase to attach inline styles, click handlers etcetera. 
 
At this stage we can create an app (that consists of one `node`):

```javascript
index.js

...

//get native DOM, For now let's use the body;
const root = document.body;
//create vElement
const myApp = createVElement('div', 'my-class');
//mount in DOM

mountVElement(myApp, root);

```
If you inspect the `DOM` in your developer tools you should see a `div` element with the class `my-class`. 

## AWESOME! :smile: You've just build a Virtual Dom. 

Granted the implementation is extremely simple. Nontheless is a great starting point. At this point
we're very motivated and are already looking further. How could we complexer applications? An application
with an x number of `nodes`? And how would we create this application in an efficient manner? It turns out
**recursion** is our friend!

