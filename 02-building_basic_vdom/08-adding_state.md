# Adding state!

Okido, now it's getting exciting. We're going to add `state`! Woop Woop. The public API 
for updating state , as you probably know, is: `this.setState(partialNewState)`. Of
course we could naively implement the `setState` function as:

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

Technically we're updating the state, but there will be no updates shown in the UI. Bummer 😞

Let's dive in the problem 🤓

## What does it even mean 🤔?

Let's stop and think about the what we're trying to accomplish. Our main goal is to reflect
the state updates in the UI. Just how would we do that?

> For now we leave **efficient** re-rendering for what it is. 

Let's implement a naive version:


```javascript
function mountVComponent(vComponent, parentDOMNode) {
  const { tag, props } = vComponent;

  const Component = tag;
  const instance = new Component(props);

  const currentElement = instance.render();

  const dom = mount(currentElement, parentDOMNode);

  // save references
  // vComponent.dom = dom; 
  // vComponent._instance = instance;

  //append the DOM we've created.
  parentDOMNode.appendChild(dom);
}

```

And update the `Component class`:


```javascript
class Component {
  constructor(props) {
    this.props = props || {};
    this.state = {};

    this._pendingState = null;
    this._parentNode = null;
  }

  updateComponent() {
    const prevState = this.state;

    if (this._pendingState !== prevState) {
      this.state = this._pendingState;
    }
    //reset _pendingState
    this._pendingState = null;
    const newElement = this.render();

    //get it in the native DOM
    mount(newElement, this._parentNode);

  }

  setState(partialNewState) {
    // Awesome things to come
    const newState = Object.assign({}, this.state, partialNewState);
    this._pendingState = newState;
    this.updateComponent();
  }
  //will be overridden
  render() {}
}
```

Let's take a look at the new `updateComponent()` function. If needed we update the local state. Then we will call
the `render()` function to retrieve the `vNodes`. These `vNode` may or may not been affected by the state change. 

> We can do at least one obvious optimization here. Why do we continue if the state hasn't changed? Another question:
 do you see any potential for adding lifecycle methods? We will get there :)

Let's define our application with updated state: 

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


What do you think that will happen? Yes, we have an updating UI **but** but it's just appended to 
the exsting DOM creating a list our elements. **Not** exactly what we're looking for! But it's actually
a really good start!

