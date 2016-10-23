# Updating

Now that we have *some* idea about an updating UI it's time to refine our implementation. We're going
to implement an `update` function that updates the UI accordingly. 
Again, it's a simplified version that most definitly will break when you try hard enough. 

# A extremely expensive problem!

In [React's docs](https://facebook.github.io/react/docs/reconciliation.html) it is mentioned that updating 
the UI efficiently is an O(n3) problem. Where `n` is the number of elements in the tree. 
So if we would update 100 elements, there would be around 1 million comparisons! 
To solve this problem efficiently React.js does two assumptions: 

* Two elements of different types will produce different trees
* The developer can hint at which child elements may be stable across different renders with a key prop.

The first assumption is taking into account, the second one we skip. 

# Patching the `DOM` with the `update()` function 🤕

We're going to update the `DOM` based on the changes made in the `vDOM`. Another term that
is for this process is `patching`. Is it important to understand that we're aiming
to update the `DOM` **only** where it has changed. In order to do so, we have
to do some sort of comparison. Let's see how. 

Our current `updateComponent` function in our `Component` class, currently looks like this:

```javascript

updateComponent() {
  const prevState = this.state;
  const prevElement = this._currentElement;

  if (this._pendingState !== prevState) {
    this.state = this._pendingState;
  }
  
  this._pendingState = null;
  const nextElement = this.render();
  this._currentElement = nextElement;

     
  mount(nextElement, this._parentNode);
}
```

As briefly mentioned we were `calling` mount for learning purposes, to see what would
happen. We can agree it certainly didn't do what we wanted. And we have to revisit this. 

We need to do some sort of comparison. More specific we want to
compare the `currentElement` with the `nextElement`. Both are `vNodes` which we can
compare easily. For example: 

```javascript
{
  tag: 'div',

}
```



