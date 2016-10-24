# Adding state!

Okido, now it's getting exciting. We're going to add `state`! Woop Woop 🚨🚨🚨

> In this section we're going to implement the option to update state. However, we will keep it
extremely simple and with lots of missing functionalities. The point is to give you an overal idea
of how it *could* work. The goal of this series is to understand how React.js works. If we would fully
implement state updates with corresponding patch/diff algorithms *here* it would take too much time. 
You're here because you want to learn more about React.js and not see my code :smile:


OK then, sorry for the small intervention. The public API for updating state in React, 
as you probably know, is: `this.setState(partialNewState)`. We could naively and wrongly 
implement the `setState` function as:

```javascript
class Component {
  constructor(props) {
    this.props = props || {};
    this.state = {};

    this._pendingState = null;
  }
  updateComponent() {
    // Awesome things to come
  }
  setState(partialNewState) {
    // Awesome things to come
    const newState = Object.assign({}, this.state, partialNewState);
    this.state = newState;
  }
  //will be overridden
  render() {}
}

```

Technically we're updating the local state, but that isn't enought, bummer! We want to reflect
the state updates in the UI.

## What does it even mean 🤔?

Our main goal is to reflect the state updates in the UI. Just how would we do that?

> For now we leave **efficient** re-rendering for what it is. 

Let's implement another naive version and learn some things along the way:

```javascript
index.js
...

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
  
  //save the instance for later references.
  vComponent._instance = instance;
  vComponent.dom = dom;

  parentDOMNode.appendChild(dom);
}

```

And update the `Component class`:


```javascript
index.js

...

class Component {
  constructor(props) {
    this.props = props || {};
    this.state = {};

    this._currentElement = null;
    this._pendingState = null;
    this._parentNode = null;
  }

  updateComponent() {
      const prevState = this.state;
      const prevElement = this._currentElement;

      if (this._pendingState !== prevState) {
        this.state = this._pendingState;
      }
      //reset _pendingState
      this._pendingState = null;
      const nextRenderedElement = this.render();
      this._currentElement = nextRenderedElement;

      //get it in the native DOM
      mount(nextRenderedElement, this._parentNode);

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
```

Let's take a look at the new `updateComponent()` function. 
If the state is updated, we update the local state. Then we will call the `render()` function to retrieve 
the `nextRenderedElement`. This element consists of one or more `vNode(s)` and may or not 
may not been affected by the state change. 
We now have a `nextRenderedElement` and `prevRenderedElement`. We've nameed them like this. because it are the
results of calling `render()`.  

Both of these elements are a `vNode` and we know which shapes they can have. 

For learning purposes we call the `mount` function with our `nextRenderedElement`. Let's see
what happens!


```javascript
class App extends Component {
  constructor() {
    super();
    this.state = {
      counter: 1
    }

    setInterval(() => {
      this.setState({ counter: this.state.counter + 1})
    }, 1000);
  }

  render() {
    return createElement('div', { style: { height: '100px', background: 'red'} }, [
      createElement('h1', {}, [ this.state.counter ])
    ]);
  }
}
```


What do you think that will happen? Yes, we have an updating UI **but** but the updates are just appended to 
the exsting DOM, this creates a long list... **NOT** exactly what we're looking for!
