# Extending our vDOM with properties

In the last section we've manually edited our `vDOM` by manipulating a
the attached `domNode`. 
 Of course in practice we would declare this properties beforehand. These properties
 could be among other things be: inline styles, classname(s) or events. Because this is
just our practice implementation we won't be implemeting all. We've already added classNames 
and to keep it simple we will add inline styles :smile:

## Adding inline styles 🍭

Let`s start with the following code:

```javascript
index.js

...

const myApp = createVElement('div', {
  style: { background: 'red', height: '100px' }}, [
  createVElement('h1', {
    style: { color: 'white' }
    })
  ])
```

Now it's time to add some red and height in our app! Compared to our previous code we've omitted 
the `className` property and added our styles. Thus, we have to update our `createVElement` function:

```javascript
index.js
...
//create a vElement
function createVElement(tag, config) {
  const { className, style } = config;

  return {
    tag: tag,
    style: style,
    className: className,
  }
}
```

Easy-peasy. Of course we're adding inline styles for the native `domNode` so
we have to attach it in the `mount` phase. Let's edit the `mountVElement` function. 

```javascript
index.js

...

function mountVElement(vElement, DOM) {
  const { tag, className, style } = vElement;
  
  const domNode = document.createElement(tag);

  vElement.dom = domNode;
  
  if (className !== undefined) {
    domNode.className = className;
  }

  //Iterate over the properties in the style
  //object. And add the property and it's value to our domNode. 
  if (style !== undefined) {
    Object.keys(style).forEach((sKey) => domNode[sKey] = style[sKey]);
  }
  
  DOM.appendChild(domNode)
}
```

Ofcourse the implementation is very naive. We skipping some important parts 
like accounting for properties where an `-` is needed, like `background-color` and 
for integer values where `px` should be added. But hey, we're still practicing!

