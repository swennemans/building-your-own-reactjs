# Recursion

As it turns out recursion is an extremely important factor in building (and updating) the `DOM`. In hindsight this is actually to be expected because we're working with a tree of data. 
 
In our previous section we've used a minimal version of our `vElement` only consisting of a `tag` and `className` property. The complete definition of our `vElement` is:

```
type VElement = {
  tag: string,
  props: {
    children: vNodeList || emtpy,
    etc.
  },
  className: string,
  style: string,
  events: ?Object,
  dom: ?Node
}
```

Because we're building a tree of data, one **prop**erty is of extra importance. Can you guess which one? It's the **children** property :)

> We've defined our children property on our props object because this is the convention that React uses. Where you can access the children using this.props.children. 

The children property can be empty or a `vNodeList`. A `vNodeList` is simply an array of `vNodes`. Arrays are pretty good at keeping order and are easy to iterate on. And that's exactly what were going to need when building a tree. 

Let's extend our application a with a header and a paragraph. 

```javascript
const myApp = createVElement('div', { className:'my-class' }, [
  createVElement('h1', { className:'my-header' }),
  createVElement('p', { className: 'my-paragraph'})
]);

```

The HTML equivelant would be:

```html
<div class="my-class">
  <h1 class="my-header"></h1>
  <p class="my-paragraph"></p>
</div>
```
The important thing to notice is: we now have multiple nodes and that some of these nodes are nested. How to handle this?
Well, the title probably already gave it away, recurse it!

While using recursion we can reuse our `mountVElement` function, because, well each child is an `vElement` that needs to be mounted. 
We just need to adjust our `mountVElement` function. We need to watch out for the `children` property, and if it exists
we need to **mount** the content.   

```javascript
function mountVElement(vElement, parentDOMnode) {
  const { className, tag, props } = vElement;
  
  //create a native DOM node
  const domNode = document.createElement(tag);
  
  //for later reference save the DOM node
  //on our vElement
  vElement.dom = domNode;

  if (props.children) {
    //Important: we pass the parentDOMnode 
    //again and again
     children.forEach((child) => 
      mountVElement(child, parentDOMNode)
    });
  }
  
  //add className to native node
  if (className !== undefined) {
    domNode.className = className;
  }

  //Append domNode to to our parentDOMNode
  parentDOMnode.appendChild(domNode);
}

```

Even with three nodes this part can be hard to wrap your head around how this works. 
There is a lot of mutating and recursing going on here. Personally I always have a hard
time understanding this.  
 In the table below the steps are (conceptually) laid out:

| Iteration | parentDOMNode | vElement | Resulting DOM |
| --- | --- | --- | --- |
| 1 | `body` | `div.my-class` | `body` |
| 2 | `.my-class` | `h1.my-header` | `body div.my-class > h1.my-header` |
| 3 | `.my-class` | `p.my-paragraph` | `body div.my-class > h1.my-header, p.my-paragraph` |



Hopefully with the above table you can reason about how the DOM is actually built from our `vElements`. 
If not try to place some `debugger` statements in places that look weird to you. 
Or just accept the fact that mutation and looping can be hard to understand :smile:. 

**Warning** If you would use`console.log` statement to debug, you will find that the printed `domNode`(s) 
are already mutated before you can inspect them (what is an insight in itself). 



## Building a complex DOM 🏯
Now we have made great progression! We this little bit of code we can even create a complex DOM with 
lots of nested nodes. 

Let's give it a shot!

```javascript

const myApp = createVElement('div', { className: 'my-class' }, [
  createVElement('h1', { className:'my-header' }),
  createVElement('div', { className:'my-container' }, [
    createVElement('div', { className:'my-sub-container' }, [
      createVElement('div', { className:'my-sub-sub-container' }, [
        createVElement('div', { className:'my-sub-sub-sub-container' }, [
          createVElement('div', { className:'my-sub-sub-sub-sub-container' }, [
            createVElement('div', { className:'my-sub-sub-sub-sub-sub-container' }, []),
            createVElement('h1', { className:'my-sub-sub-sub-sub-sub-header' }, [])
          ])
        ])
      ])
    ]),
  ])
]);

```

No problemo for our code! We can build a deeply nested `DOM` without any problems. Awesome! 
But  what is the role of our `vDOM` in all of this? Curious? Me too! Let's go to the next section 🛤



