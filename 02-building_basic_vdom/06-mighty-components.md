# The Mighty Component 🏋

Till know we've kept a strict mapping between our `vDOM` and `DOM`, meaning we've used
`vElements` and `vTexts`. Even though it's pretty cool that we can construct our UI this way, we've
kept it static. How would we update our UI and keep state? The answer is `Components`.
Components are like `vNodes` but with superpowers!  

Well, sort of because you will see that **we** selectably define these extra powers. The practical difference
with other `vNodes` is that instead of directly returning it's contents, we explicitly define a function, 
the (all familiair) `render` function. 
This **opens all kinds of opportunities** We can save stuff and do
work before and after we call `render()`. Hint: we're do you think the `lifecycle` methods are called? :smile:

## Creating a Component

Just as any other `vElement` the (mighty) Component also needs to be created. We can define
a `createVComponent` function. 

> Please note that React.js does this differently. It has the concept
of internal components. These types of components handle the 
mounting, updating etc. 

```javascript
index.js

...

function createVComponent(tag, props) {
  return {
    component: tag,
    props: config,
  }
}

```  
You could have some questions here like what does the component property hold, and why are there no children??
We're going to find out soon! But first we need to take a small detour:

## Our own `createElement` function

Do you remember that we first had a `mountVElement` function that handled all the mounting? We refactored
our code (out of necissity) and created a `mount()` function. 
We're at a similair point now. We need a function that will select the appropriate `create` function, either the `createVElement`
or `createVComponent` function. 

We're going to call this function, just as React does, `createElement`. Let add this function to our code:

```javascript
index.js
...

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


```

Most of the code looks straightforward. However the `typeof tag === 'function`' could raises
some questions. In React.js and using JSX you could write. 

```javascript
class App extends React.Component {
  render() {
    return (
      <div>Foo</div>
    );
  }
}

ReactDOM.render(<App/>, document.getElementById('root'));
```

If we don't use `JSX`, our `App` would be instantiated with `React.createElement(App)`. 

We see that App is extending `React.Component` and it is a `Class`. 
For the `createElement` function to make sense, let's inspect what a `Class` actually is:

```javascript
class Component { 
  render() {}
}

typeof Component === 'function' // true!
```

Ahaa it's a function. That explains our `createVElement` code. 


## Instantiating our Components
Just as in React we want to instantiate our `Components` as:

```javascript
class App extends Component {
  constructor() {
    super();
    this.state = {
      message: 'Were creating a Component!'
    }
  }

  render() {
    return (
      createVElement('div', { className: 'foo'}, 'bar')
    );
  }
}
```

Hopefully the `Class` keyword doesn't scare you. We will keep it simple! 

> We *could* define it in one of the other ways JavaScript allows it, but we want to 
keep the API close to React's one and, above all, keep it simple!


### Quick recap

Till now we briefly discussed why we need a `Component` and that we would like to instantiate it
using it the same as in React.js. 
 We've refactored our code so that it can call the appropriate function on creation. 

However, we didn't discuss how our `Component class` looks like and how to mount it. Let's continue...🚂
