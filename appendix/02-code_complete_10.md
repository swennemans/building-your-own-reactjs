The code after chapter [Updating children](../02-building_basic_vdom/10-component-class.md)

```javascript
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

  const Component = tag;
  const instance = new Component(props);

  const nextRenderedElement = instance.render();
  //create a reference of our currenElement
  //on our component instance.
  instance._currentElement = nextRenderedElement;
  //create a reference to the passed
  //DOMNode. We might need it.
  instance._parentNode = parentDOMNode; 


  const dom = mount(nextRenderedElement, parentDOMNode);
  
  //save the instance for later!
  vComponent._instance = instance;
  vComponent.dom = dom;

  parentDOMNode.appendChild(dom);
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
    if (!Array.isArray(props.children)) {
      mount(props.children, domNode)
    } else {
      props.children.forEach(child => mount(child, domNode));
    }
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

function updateVElement(prevElement, nextElement) {
  const dom = prevElement.dom;
  nextElement.dom = dom;

  if (nextElement.props.children) {
    updateChildren(prevElement.props.children, nextElement.props.children, dom);
  }
  
  if (prevElement.style !== nextElement.style) {
    Object.keys(nextElement.style).forEach((s) => dom.style[s] = nextElement.style[s])
  }
}

function updateVText(prevText, nextText, parentDOM) {
  if (prevText !== nextText) {
    parentDOM.firstChild.nodeValue = nextText;
  }
}

function updateChildren(prevChildren, nextChildren, parentDOMNode) {
  if (!Array.isArray(nextChildren)) {
    nextChildren = [nextChildren];
  }
  if (!Array.isArray(prevChildren)) {
    prevChildren = [prevChildren];
  }

  for (let i = 0; i < nextChildren.length; i++) {
    //We're skipping a lot of cases here. Like what if
    //the children array have different lenghts? Then we 
    //should replace smartly etc. :)
    const nextChild = nextChildren[i];
    const prevChild = prevChildren[i];


    //Check if the vNode is a vText
    if (typeof nextChild === 'string' && typeof prevChild === 'string') {
      //We're taking a shortcut here. It would cleaner to
      //let the `update` function handle it, but we would to add some extra
      //logic because we don't have a `tag` property. 
      updateVText(prevChild, nextChild, parentDOMNode);
      continue;
    } else {
      update(prevChild, nextChild);
    }
  }
}


function updateVComponent(prevComponent, nextComponent) {
  
  //get the instance. This is Component. It also 
  //holds the props and _currentElement; 
  const { _instance } = prevComponent;
  const { _currentElement } = _instance;

  //get the new and old props!
  const prevProps = prevComponent.props;
  const nextProps = nextComponent.props;

  //Time for the big swap!
  nextComponent.dom = prevComponent.dom;
  nextComponent._instance = _instance;
  nextComponent._instance.props = nextProps;

  if (_instance.shouldComponentUpdate()) {
    const prevRenderedElement = _currentElement;
    const nextRenderedElement = _instance.render();
    //finaly save the nextRenderedElement for the next iteration!
    nextComponent._instance._currentElement = nextRenderedElement;
    //call update 
    update(prevRenderedElement, nextRenderedElement, _instance._parentNode);
  }
}

class Component {
  constructor(props) {
    this.props = props || {};
    this.state = {};

    this._currentElement = null;
    this._pendingState = null;
    this._parentNode = null;
  }

  //add this in existing code
  shouldComponentUpdate() {
    return true;
  }

  updateComponent() {
    const prevState = this.state;
    const prevElement = this._currentElement;

    if (this._pendingState !== prevState) {
      this.state = this._pendingState;
    }
    
    this._pendingState = null;
    const nextElement = this.render();
    this._currentElement = nextElement;

      
    update(prevElement, nextElement, this._parentNode);
  }

  setState(partialNewState) {
    // I know this looks weired. Why don't pass state to updateComponent()
    // function, I agree. 
    // We're just getting a little familiair with putting data on instances. 
    // seomthing that React uses heavily :)
    this._pendingState = Object.assign({}, this.state, partialNewState);
    this.updateComponent();
  }
  //will be overridden
  render() {}
}

function update(prevElement, nextElement) {
  //Implement the first assumption!
  if (prevElement.tag === nextElement.tag) {
    //Inspect the type. If the `tag` is a string
    //we have a `vElement`. (we should actually
    //made some helper functions for this ;))
    if (typeof prevElement.tag === 'string') {
      updateVElement(prevElement, nextElement);
    } else if (typeof prevElement.tag === 'function') {
      updateVComponent(prevElement, nextElement);
    }
  } else {
    //Oh oh two elements of different types. We don't want to 
    //look further in the tree! We need to replace it!
  }
}


//get native DOM, For now let's use the body;
const root = document.body;
//create vElement
class NestedApp extends Component {
  constructor(props) {
    super(props);
  }

  shouldComponentUpdate() {
    return this.props.counter % 2;
  }
  
  render() {
    return createElement('h1', { style: { color: '#'+Math.floor(Math.random()*16777215).toString(16) } }, `The count from parent is: ${this.props.counter}`)
  }
}

class App extends Component {
  constructor() {
    super();
    this.state = {
      counter: 1
    }
    setInterval(() => {
      this.setState({ counter: this.state.counter + 1 })
    }, 500);
  }

  render() {
    const { counter } = this.state;
    //background color stolen from: https://www.paulirish.com/2009/random-hex-color-code-snippets/
    return createElement('div', { style: { height: `${10 * counter}px`, background: '#'+Math.floor(Math.random()*16777215).toString(16) } }, [
      `the counter is ${counter}`,
      createElement('h1', { style: { color: '#'+Math.floor(Math.random()*16777215).toString(16) } }, `${'BOOM! '.repeat(counter)}`),
      createElement(NestedApp, { counter: counter})
    ]);
  }
}

mount(createElement(App, { message: 'Hello there!' }), root);
```