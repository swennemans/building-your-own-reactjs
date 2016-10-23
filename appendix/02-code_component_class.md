The working code at the end of chapter [Component class](../02-building_basic_vdom/07-component-class.md)

```javascript
index.js

function createVElement(tag, config, children = null) {
  const { style, className } = config;
  return {
    tag: tag,
    props: {
      children: children,
    },
    className: className,
    style: style,
 }
}

function createVComponent(tag, props) {
  return {
    tag: tag,
    props: props,
  }
}

function createElement(tag, config, children) {
  // If the tag is a function. We have a component!
  // we will see later why. 
  if (typeof tag === 'function') {
    //of course we could do some checks here if the props are 
    //valid or not.
    const vNode = createVComponent(tag, config);
    return vNode;
  }

  //Add children on our props object, just as in React. Where
  //we can acces it using this.props.children;
   
  const vNode = createVElement(tag, config, children);
  return vNode;
}

function mountVComponent(vComponent, parentDOMNode) {
  const { tag, props } = vComponent;
  
  // build a component instance. This uses the 
  // defined Component class. For brevity 
  // call it Component. Not needed ofcourse;
  const Component = tag;
  const instance = new Component(props);

  // The instance or Component has a render() function 
  // that returns the user-defined vNode.
  const initialVNode = instance.render();

  // the initialVNode can be a vElement or a
  // vComponent. mountVComponent doenst't care. Let the mount()
  // handle that!
  const dom = mount(initialVNode, parentDOMNode);

  // save the DOM reference!
  vComponent.dom = dom;
  // save the instance. 
  vComponent._instance = instance;

  //append the DOM we've created.
  parentDOMNode.appendChild(dom);
}



function mount(input, parentDOMNode) {
  if (typeof input === 'string') {
    //we have a vText
    return mountVText(input, parentDOMNode);
  } 
  else if (typeof input.tag === 'function') {
    //we have a component
    return mountVComponent(input, parentDOMNode);
  }
  else {
    //we have a vElement
    return mountVElement(input, parentDOMNode)
  }
}

function mountVText(vText, parentDOMNode) {
  parentDOMNode.textContent = vText;
  return vText;
}

function mountVElement(vElement, parentDOMNode) {
  const { className, tag, style, props } = vElement;
  const domNode = document.createElement(tag);

  vElement.dom = domNode;

  if (props.children) {
     //we call our new function mount!
     props.children.forEach((child) => mount(child, domNode));
  }
  
  if (className !== undefined) {
    domNode.className = className;
  }
  
  if (style !== undefined) {
    Object.keys(style).forEach((sKey) => domNode.style[sKey] = style[sKey]);
  }

  parentDOMNode.appendChild(domNode);

  return domNode;
}

class Component {
  constructor(props) {
    this.props = props || {};
  }
  updateComponent() {
    // Awesome things to come
  }
  setState(partialNewState) {
    // Awesome things to come
  }
  //will be overridden
  render() {}
}

//Our app
class App extends Component {
  render() {
    return createElement('div', { style: { height: '100px', background: 'red'} }, [
      createElement('h1', {}, [ this.props.message ])
    ]);
  }
}

const root = document.body;

mount(createElement(App, { message: 'Hello there!' }), root);
```
