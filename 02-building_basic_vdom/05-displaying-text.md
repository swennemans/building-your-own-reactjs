# Displaying Text 📖

Now that we can actually add styles, the next step is to add some text to display. 
In the native DOM a `textNode` is a different type than a `HTMLElement`, 
hence the different methods to create each. In our `vDOM` we follow suit. 

A `vText` node is formally defined as: `type VText = string || number`. So, it's
just a `string` or `number` :smile:

## Refactor the our code

Currently our `mountVElement` is doing everything related to mounting. This made
sense because we were only mounting `vElements` . But there is a new player in town, 
the `vText` yo, and that will force us to rethink our code. 

We now have two players, `vElement` and `vText`. We want to mount the `vText` 
and give it it's own dedicated function, `mountVText`. The function `mountVElement` 
is currently doing work which we can extract into a new function.  We call this function `mount`. 
The purpose of the `mount` function is to choose the approporiate function to 
call when we're recursing. This is easily shown in our new code:

```javascript

function mount(input, parentDOMNode) {
  //Hmmm lets see what input is. 
  if (typeof input === 'string' || typeof input === 'number') {
    //we have a vText
    mountVText(input, parentDOMNode);
  } else {
    //we have a vElement
    mountVElement(input, parentDOMNode)
  }
}

function mountVText(vText, parentDOMNode) {
  // Oeeh we received a vText with it's associated parentDOMNode.
  // we can set it's textContent to the vText value. 
  // https://developer.mozilla.org/en-US/docs/Web/API/Node/textContent
  parentDOMNode.textContent = vText;
}

function mountVElement(vElement, parentDOMNode) {
  const { className, tag, props, style } = vElement;

  const domNode = document.createElement(tag);
  vElement.dom = domNode;
  if (props.children) {
     // Oeh, we have children. Pass it back to our mount
     // function and let it determine what type it is.
     props.children.forEach(child => mount(child, domNode));
  }

  if (className !== undefined) {
    domNode.className = className;
  }
  
  if (style !== undefined) {
    Object.keys(style).forEach(sKey => domNode.style[sKey] = style[sKey]);
  }

  parentDOMNode.appendChild(domNode);

  return domNode;
}
```

Awesome everything is in place. 
Let's redefine anew app where we can test our new implementations:
```javascript
index.js

...

const root = document.body;

const myApp = createVElement('div', { 
  style: { height: '100px', background: 'red'},
  className: 'my-class' }, 
    [ createVElement('h1', { className:'my-header' }, ['Hello!']),
      createVElement('div', { className:'my-container' }, [
        createVElement('p', {}, ['A container with some nice paragraph'])])
    ]
);
mount(myApp, root);
```

## Awesome, we can read stuff now! 

You can see that we didn't define a `createVText` function. 
We *could* do it, but this would just return it's value (a string or a number). 

Also, it might look weird that we only need to change the textContent of it's
parentDOMNode, but again this is recursion at work. The function will receive an h1 `HTMLElement` 
and add text to it, first we only have `<h1></h1>`, then in the next iteration: `<h1>My Text</h1>`. 

Very cool! We now have a minimal `vDOM` implementation! We can add styles, classNames and text. 
However, it is static. How would we be able to add state and add some dynamics??

Let's find out!