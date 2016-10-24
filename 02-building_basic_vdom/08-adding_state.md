# Adding state!

Okido, now it's getting exciting. We're going to add `state`! Woop Woop. 

> In this section we're going to implement the option to update state. However, we will keep it
extremely simple and with a lot of missing functionalities. The point is to give you an overal idea
of how it *could* work. The goal of this series is to understand how React.js works. If we would fully
implement state updates with corresponding patch/diff algorithms *here* it would take too much time. 
You're here because you want to learn more about React.js and not see my code :smile:

The public API for updating state in React, as you probably know, is: `this.setState(partialNewState)`.
We could naively and wrongly implement the `setState` function as:

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

Technically we're updating the local state, but that isn't enough is it? We want to reflect
the state updates in the UI.

## What does it even mean 🤔?

Let's stop and think about the what we're trying to accomplish. Our main goal is to reflect
the state updates in the UI. Just how would we do that?

> For now we leave **efficient** re-rendering for what it is. 

Let's implement another naive version and learn some things along the way:

```javascript
function mountVComponent(vComponent, parentDOMNode) {
  const { tag, props } = vComponent;

  const Component = tag;
  const instance = new Component(props);

  const currentElement = instance.render();
  //create a reference of our currenElement
  //on our component instance.
  instance._currentElement = initialVNode;
  //create a reference to the passed
  //DOMNode. We might need it.
  instance._parentNode = parentDOMNode; 

  const dom = mount(currentElement, parentDOMNode);

  parentDOMNode.appendChild(dom);
}

```

And update the `Component class`:


```javascript

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
      const nextElement = this.render();
      this._currentElement = nextElement;

      //get it in the native DOM
      mount(nextElement, this._parentNode);

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

Let's take a look at the new `updateComponent()` function. If needed we update the local state. Then we will call
the `render()` function to retrieve the `vNodes`. The `vNodes` may or may not been affected by the state change. We
now have a `nextElement` and `prevElement`.

Both of these element's are `vNodes` and we know which shapes they can have. For learning purposes we 
call the `mount` function with our `newElement`.


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
