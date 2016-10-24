# Component class

The `Component` class will be the bread and butter of our dynamic UI. Let's write an minimal 
implementation: 

```javascript
index.js

...

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

```

Exciting! We're going to implement the much used `setState` function ourselves! 
But, before we're going finish the `Component class` implementation we will we look at mounting first. 
We will definitly run into some problems, but these problems will 
let us define the requirements for the `Component class` better!

## Mounting a Component with `mountVComponent`.

As you probably could have guessed we're going to define 
this function as `mountVComponent`. And it does a bit more work then 
the other mount functions. Let's see:

```javascript
index.js
...

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
  const currentElement = instance.render();

  // the currentElement can be a vElement or a
  // vComponent. mountVComponent doenst't care. Let the mount()
  // handle that!
  const dom = mount(currentElement, parentDOMNode);

  //save the instance for later
  //references! 
  vComponent._instance = instance;
  vComponent.dom = dom;

  //append the DOM we've created.
  parentDOMNode.appendChild(dom);

  return dom;
}

```

> React.js has a `mountComponent` function on it's ReactCompositeComponent or ReactDOMComponent.

The most important part is that we instantiate our `Component` with `new Component(props)`.
This creates a `Component class`, that has a `render()` function. 
The render function returns the `vNodes` and, we recurse again! 
**It's all about recursion yo!**

Now we need to update the `mount` function, so that it can handle `vComponents`. 

```javascript
index.js
...

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
  // else would suffice :). 
  else if (typeof input.tag === 'string') {
    //we have a vElement
    return mountVElement(input, parentDOMNode)
  }
}
```

And we can define and render our new application as: 

```javascript
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

> If the code is not working, or If I accidently skipped parts, please let me know. The
code we *should* have at this point. can be found [here](../appendix/02-code_component_class.md)

Awesome! We've now introduced `Components`!. We've built a `Component Class`
which we can easily extend. We're making moves yoo!
But, we're not  utilizing the Components as we should, we introduced them for their powers.. 

Let's take a look at **state**.



