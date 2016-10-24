The working code at the end of chapter [Component class](../02-building_basic_vdom/10-updating-children.md)

```javascript
index.js

function createVComponent(tag, props) {
  return {
    tag: tag,
    props: props,
    dom: null,
  }
}

function createVElement(tag, config, children = null) {
  const { className, style } = config;

  return {
    tag: tag,
    style: style,
    props: {
      children: children,
    },
    className: className,
    dom: null,
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


function mount(input, parentDOMNode) {
  if (typeof input === 'string' || typeof input === 'number') {
    //we have a vText
    return mountVText(input, parentDOMNode);
  } 
  else if (typeof input.tag === 'function') {
    //we have a component
    return mountVComponent(input, parentDOMNode);
  }
  // for brevity make an else if statement. An
  // else would suffice. 
  else if (typeof input.tag === 'string') {
    
    //we have a vElement
    return mountVElement(input, parentDOMNode)
  }
}

function mountVComponent(vComponent, parentDOMNode) {
  const { tag, props } = vComponent;
  
  // build a component instance. This uses the 
  // defined Component class. For brevity 
  // call it Component. Not needed ofcourse, new tag(props)
  // would do the same. 
  const Component = tag;
  const instance = new Component(props);

  // The instance or Component has a render() function 
  // that returns the user-defined vNode.
  const nextElement = instance.render();

  // the nextElement can be a vElement or a
  // vComponent. mountVComponent doenst't care. Let the mount()
  // handle that!
  const dom = mount(nextElement, parentDOMNode);
  //append the DOM we've created.
  parentDOMNode.appendChild(dom);

  return dom;
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

class Component {
  constructor(props) {
    this.props = props || {};
  }
  setState(partialNewState) {
    // Awesome things to come
  }
  //will be overridden
  render() {}
}

//get native DOM, For now let's use the body;
const root = document.body;
//create vElement
class App extends Component {
  render() {
    return createElement('div', { style: { height: '100px', background: 'red'} }, [
      createElement('h1', {}, [ this.props.message ])
    ]);
  }
}

mount(createElement(App, { message: 'Hello there!' }), root);

```
