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

  updateComponent() {
    // Awesome things to come
  }

  setState(partialNewState) {
    // Awesome things to come
  }

  //will be overridden
  render() {}
}

```

Exciting! We're going to implement the much used `setState` function ourselves! But, before
we're going finish the `Component` implementation we will mount a `vComponent` first. We will definitly
run into some problems, but these problems will let define the requirements for the `Component`!

## Mounting a Component with `mountVComponent`.

As you probably could guess we're going to define this function as `mountVComponent`. And it does 
a bit more work then the other mount functions. Let's see:

```javascript

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

  return dom;
}

```

The most important part is that we instantiate our `Component` with `new Component(props)`.
This in turn gives us access to the `render()` function. The render function returns the 
`vNodes` and we recurse again! **It's all about recursion yo!**

Now we need to update the `mount` function, so that it can handle `vComponents`. 

```javascript

function mount(input, parentDOMNode) {
  if (typeof input === 'string') {
    //we have a vText
    return mountVText(input, parentDOMNode);
  } 
  else if (typeof input.tag === 'function') {
    //we have a component
    return mountVComponent(input, parentDOMNode);
  }
  // for brevity make an else if statement. An
  // else would suffice. 
  else if (typeof input.tag === 'object') {
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
code we *should* have at this point. can be found [here](appendix/02-code_component_class.md)

Awesome! We've now introduced components and can also pass props. But, we're not 
utilizing the Components as we should, we introduced `Components` because we want a 
dynamic UI. Time to change that!


