# Displaying Text

Now that we can actually add styles, the next step is to add some text. In the native DOM
a `textNode` is a different type than a `HTMLElement`, hence the different methods to create each. 
In our `vDOM` we follow suit. 

We can formally define a `vText` node as: `type VText = string || number`.


## Breaking-up our code

Currently our `mountVElement` is doing everything related to mounting. This made
sense because we were only mounting `vElements` . But there is a new player in town
and that will force us to rethink our code. 

We now have two players, `vElement` and `vText`. We want to mount the `vText` and give it it's function, `mountVText`, and now everybody is happy I guess. But, not really
`mountVElement` is currently doing too much work which we can extract into a new function. 
We call this function `mount`. The purpose of the `mount` function is to choose the approporiate function to
call when we're recursing.  
 
Let's see some code:

```javascript

function mount(input, parentDOMNode) {
  //Hmmm lets see what input is. 
  if (typeof input === 'string' || typeof input === 'number') {
    //we have a vText
    mountVText(input, DOMNode);
  } else {
    //we have a vElement
    mountVElement(input, DOMNode)
  }
}

function mountVText(vText, parentDOMNode) {
  // Oeeh we received a vText with it's associated parentDOMNode.
  // we can set it's textContent to it. 
  // https://developer.mozilla.org/en-US/docs/Web/API/Node/textContent
  DOMNode.textContent = vText;
}

function mountVElement(vElement, parentDOMNode) {
  const { className, tag, props } = vElement;
  
  const domNode = document.createElement(tag);

  vElement.dom = domNode;

  if (props.children) {
     // Oeh, we have children. Pass it back to our mount
     // function and let it determine what type it is.
     children.forEach((child) => mount(child, domNode));
  }
  
  if (className !== undefined) {
    domNode.className = className;
  }
  
  if (style !== undefined) {
    Object.keys(style).forEach((sKey) => domNode.style[sKey] = style[sKey]);
  }

  parentDOMNode.appendChild(domNode);
}
```

Awesome everything is in place. 

```javascript
//get native DOM, For now let's use the body;
const root = document.body;

const myApp = createVElement('div', { 
  style: { height: '100px', background: 'red'},
  className: 'my-class' }, 
    [ createVElement('h1', { className:'my-header' }, ['Hello!']),
      createVElement('div', { className:'my-container' }, [
        createVElement('p', {}, ['A container with some nice paragraph'])
    ]);
]);

mount(myApp, root);

```
If you look closely you'll see that we didn't define a `createVText` function. Why?   

