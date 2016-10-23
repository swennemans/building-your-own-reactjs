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

Exciting! We've 