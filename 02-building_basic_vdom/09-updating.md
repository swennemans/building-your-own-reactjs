# Updating

Now that we have *some* idea about an updating UI it's time to refine our implementation. We're going
to implement an `update` function that updates the UI accordingly. 
Again, it's a simplified version that **most definitly** will break when you try hard enough (probably even 
with not trying hard :smile: )

# An extremely expensive problem!

In [React's docs](https://facebook.github.io/react/docs/reconciliation.html) it is mentioned that updating 
the UI efficiently, by comparing trees, is an O(n3) problem. Where `n` is the number of elements in the tree. 
So if we would update 100 elements, there would be around 1 million comparisons. A tree with 100 elements
is actually very common. Ouch...

But the smart people working at Facebook solved this problem, by doing two assumptions:  

* Two elements of different types will produce different trees.
* The developer can hint at which child elements may be stable across different renders with a key prop.

The first assumption we will implement. The second one, being more complicated, we skip for now!
Later we will look at this. Hint: the solution has something to do something with the dreaded `key` property :smile:

# Patching 🤕 the `DOM` with the `update()` function 🕹

We're going to update the `DOM` based on the changes made in the `vDOM`. This process is also called
`patching`. 
Is it important to understand that we're aiming to update the `DOM` **only** where it has changed. 
In order to do so we have to find out where things have changed. The `vDOM`
is like our joystick that allows us to maneuver efficiently through the forrest of the `DOM`.. uhm. 

We're just comparing `vNodes` hehe :smile:

Our current `updateComponent` function in our `Component` class, currently looks like this:

```javascript

updateComponent() {
  const prevState = this.state;
  const prevRenderedElement = this._currentElement;

  if (this._pendingState !== prevState) {
    this.state = this._pendingState;
  }
  
  this._pendingState = null;
  const nextRenderedElement = this.render();
  this._currentElement = nextRenderedElement;

     
  mount(nextRenderedElement, this._parentNode);
}
```

As briefly mentioned we were `calling` mount for learning purposes. We can agree it certainly didn't 
do what we wanted. Time to do some revisiting. 

### `prevRenderedElement`, `nextRenderedElement` and `_currentElement`

Yes, `this._currentElement` on our `Component class` was sneaked in the code without explicitly 
mentioning it. Sorry 'bout that! But why do we need it? Well, as mentioned we want to **compare** `vNodes`. More specifically
we want to compare the `prevRenderedElement` and `nextRenderedElement`. Both are results of calling
the class' `render()` function. However, the `updateComponent()` function doesn't have access to
all previous results of the `render()` function. That's why we save the `nextRenderElement` on `this._currentElement`.
The next time `updateComponent()` is called, `this._currentElement` is the `previousRenderedElement`and so on!

Bot `prevRenderedElement` and `nextRenderedElement` will be`vNodes` which can be compared easily. 

Do you remember the shapes of the `vNodes`? Let's look at a simple example:

```javascript
const prevRenderedElement = { tag: 'div',
                      props: {
                        children: [],
                      },
                      className: 'my-class',
                      style: {
                        height: '100px',
                      },
                      dom: `HTMLElement reference`
                    }

const nextRenderedElement = { tag: 'div',
                      props: {
                        children: [],
                      },
                      className: 'my-class',
                      style: {
                        height: '300px',
                      },
                      dom: `HTMLElement reference`
                    }
```

We see that in the `nextRenderedElement` the style property is changed. We jus

> Note that React will actually collect the different changes to do something called batch updating. 

This actually sounds really simple right? Let's find out! We need to define an `update` function.

```javascript
index.js

...

function update(prevElement, nextElement) {
  //Implement the first assumption!
  if (prevElement.tag === nextElement.tag) {
    //Inspect the type. If the `tag` is a string
    //we have a `vElement`. (we should actually
    //made some helper functions for this ;))
    if (typeof prevElement.tag === 'string') {
      updateVElement(prevElement, nextElement);
    }
  } else {
    //Oh oh two elements of different types. We don't want to 
    //look further in the tree! We need to replace it!
  }
}
```

In our `update` function, we receive a `nextElement` and `prevElement`. First 
of all, we check if the `type` of the `vNode` has changed. This is the first assumption at work! If
the type `hasn't` changed we continue the **update** (or patching) process.

It's time to implement an `updateVElement` function:

```javascript
index.js

...

function updateVElement(prevElement, nextElement) {
  //get the native DOMnode information. 
  const dom = prevElement.dom;
  //store the native DOMnode information.
  //on our nextElement.  
  nextElement.dom = dom;
  
  const nextStyle = nextElement.style;
  if (prevElement.style !== nextStyle) {
    //The style has changed!
    Object.keys(nextStyle).forEach((s) => dom.style[s] = nextStyle[s])
  }
}
```

In the `updateVElement` function, we're only looking for `style` changes (we're not removing old styles). 
If the style has changed, we write the new styles to the native `DOMNode`;

```
class Component {

  ...

  //only update the updateComponent function
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
}

```
It's a contrived example, but hey who cares! 

### Time to disco 💃🎉💃

Let's redefine our application to see if things are working as expected: 

```javascript
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
    //background color stolen from: https://www.paulirish.com/2009/random-hex-color-code-snippets/
    return createElement('div', { style: { height: `${10 * this.state.counter}px`, background: '#'+Math.floor(Math.random()*16777215).toString(16) } }, [
      `the counter is ${this.state.counter}`
    ]);
  }
}

const root = document.body;

mount(createElement(App), root);

```

Is it working?? **YES** and **NO**. Yes our application is updating the height as we wanted 
and we could even make it a disco by lowering the interval. Pretty awesome 🕶

**BUT** our text is not updating... why is that?

Well, the text is defined in the `vElement` as a `child` and we don't handle the `children`, **yet**. 



