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
we're going finish the `Component`, let's try to mount a `vComponent` first. We will definitly
run into some problems, but these problems will let define the requirements for the `Component`!

## Mounting a Component with `mountVComponent`.

