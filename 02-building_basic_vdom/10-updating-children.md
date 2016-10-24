# Children.. 🚸🚸

Children are **very** important and at the same time troublemakers, just as in real life would my parenting friends say 👪. 
React created a seperate class to handle them: `ReactMultiChildren`. But we don't need that in our lifes yet. 
We're just practicing :smile:

We know that our `children` live in `this.props.children`. Soooo, in our `updateVElement` function we can look for them. And 
implement logic to handle those brats: 

```javascript
index.js
...

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

```

We have an `updateChildren` function that is called when the current `vElement` has `children`. The `updateChildren` 
is very naivly implemented. For example it doesn't take into account the possibility that `vNode` can be added or removed (the arrays
would have different lenghts). But we won't worry about that. 

We also take a quick shortcut to check if we're dealing with a `vText`. If so, we directly call the `updateVText` function. Of course it 
would be nicer to call the `update` function directly to handle the different types, but *we aint got no time for that*. 

The only thing remaining to be implemented is the `updateVText` function:

```javascript
index.js
...

function updateVText(prevText, nextText, parentDOM) {
  if (prevText !== nextText) {
    parentDOM.firstChild.nodeValue = nextText;
  }
}

```

Easy does it! Let's build a proper application now: 

```javascript

class NestedApp extends Component {
  constructor(props) {
    super(props);
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
    }, 2500);
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

```

OOH YEAHHHH ITS TIME TO **PREMATURELY** CELEBRATE YOO.

![](../appendix/disco.gif)


We've **almost** made it. Everythang seems to work nicely. However, there is one thing left to implement. We can pass `Components` as child to other
`Components` which is awesome! It also renders the passed props. However, if the props are updated this is not represented in the UI. 

If you look closely you will see that the render function of the `NestedApp` isn't called after the initial render. We need to fix that!

## Updating the mighty Components!

As we've found out when we were writing the code for mounting `Components`, `Components` are behaving differently then our `vElements` and `vTexts`. And 
that is a good thing because it creates all kinds of opportunities, but it **does** mean we have to think a little bit harder about the implementation
of.... the `updateVComponent` function. 

**What do we actually want?** Mentally, I like to split `Components` in two. The `render()` function
and everything else. 

The eventual goal is to call the `update` function, with an `prevElement` and `nextElement`. The render function
will give us the `nextElement`. The `prevElement` we can grab from `this._currentElement`, we were smart and saved
this one on `mountVComponent` . Actually, better names would be: `nextRenderedElement` and `prevRenderedElement`, because
it a results from calling `render()`.

However, before we're calling `render()` we can do some housekeeping. We swap the `_instance`, `dom` and `props` from our
previous to next `vComponent`, and update the `_instance` property with the `_currentElement`. 

**Then** we call `render()`, and call `update` with the `prevRenderedElement`and `nextRenderedElemen`. 

```javascript
index.js

...

function updateVComponent(prevElement, nextElement) {
  
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

  const prevRenderedElement = _currentElement;
  const nextRenderedElement = _instance.render();
  
  //finaly save the nextRenderedElement for the next iteration!
  nextComponent._instance._currentElement = nextRenderedElement;

  //call update 
  update(prevRenderedElement, nextRenderedElement, _instance._parentNode);
}

```

Tell me about it, this can be massively confusing :smile: I would suggest to put some `debugger` statements
between the different lines and see what is doing what. 
Remember, we're doing a lot preparing for the next iteration and next and next and next... **Recursion is very
important**!

> 💡 If we would refactor our `updateComponent` function on our Component class, we could
reuse logic here. 

Time to take our new code for a testdrive. Let's redefine our new application:

*Please work 🙏🙏🙏*

```javascript

class NestedApp extends Component {
  constructor(props) {
    super(props);
  }
  render() {
    return createElement('h1', { style: { color: '#'+Math.floor(Math.random()*16777215).toString(16) } }, [
      `The count from parent is: ${this.props.counter}`, 
      createElement('div', {}, `Some text ${this.props.counter}`)
    ])
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
    }, 100);
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


const root = document.body;

mount(createElement(App), root);

```

# AWESOME! ✋👏✋👏✋👏 (high fives) The app is rerendering, props are updated, we can nest components. We should be proud!

## ShouldComponentUpdate?

We haven't really discussed lifecycle methods yet. But at this stage it would be nice to add the `shouldComponentUpdate`
lifecylce method. 

We need to update our `Component class` a bit, so that it will return true as default: 

```javascript
Class Component {
  ...

  //add this in existing code
  shouldComponentUpdate() {
    return true;
  }
}
```

Now we can update our `updateVComponent` function: 

```javascript
function updateVComponent(prevComponent, nextComponent) {
  ...

  if (_instance.shouldComponentUpdate()) {
    const prevRenderedElement = _currentElement;
    const nextRenderedElement = _instance.render();
    //finaly save the nextRenderedElement for the next iteration!
    nextComponent._instance._currentElement = nextRenderedElement;
    //call update 
    update(prevRenderedElement, nextRenderedElement, _instance._parentNode);
  }

  //do nothing!
}

```

As you can see, we call the `shouldComponentUpdate()` function before we want to start rendering. If 
the function returns false, we do nothing and don't rerender. Let give it a try:

```javascript
index.js

class NestedApp extends Component {
  constructor(props) {
    super(props);
  }
  
  shouldComponentUpdate() {
    return this.props.counter % 2;
  }

  render() {
    console.log('OK IM RERENDERING!');

    return createElement('h1', { style: { color: '#'+Math.floor(Math.random()*16777215).toString(16) } }, [
      `The count from parent is: ${this.props.counter}`, 
      createElement('div', {}, `Some text ${this.props.counter}`)
    ])
  }
}

```

# ANDDDDD there you go! Now we can build all efficient apps and shit! 👏👏






