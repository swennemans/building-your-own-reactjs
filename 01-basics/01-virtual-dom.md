#The virtual DOM 🕹

React is not inheritly tied to it's Virtual DOM (vDOM) but React.js (currently) is. 
 The `vDOM` is an (important) implementation detail to make React.js declarable, predictable and performant.

> If you're interested in why React.js is using a vDOM I can recommend [this video](https://www.youtube.com/watch?v=-DX3vJiqxm4) of Pete Hunt.

The vDOM is an in-memory JavaScript representation of the (native) `DOM` tree.
the `DOM` tree is built out of `nodes`. The `vDOM` beign a representation of the `DOM` is built out of
`virtual nodes` (vNodes).
This does **not** means that the `vDOM` a replacement of the `DOM`. They need to co-exist and work together :couple:

## Formal definitions 🤓

In order to understand what these `vNodes` are, let's get a little formal! Don't worry it's just describing the `vNodes`. 

In our own implementation the `vNodes` can be of either two types: virtual elements (vElement) or virtual text elements (vText). 
Which can be mapped to respectively the DOM's `HTMLElement` and `TextNode`. 

>The formal type definitions of our custom implementation differs a little bit from React’s one. The reason being that it will lead to (hopefully) some easier to understand code.
Our vElement and vText are defined as.

Without further ado I present our definitions:
```javascript
type VNode = vElement | vText
type VNodeList = VNode[]

type VElement = {
  tag: string,
  props: {
    children: vNodeList || emtpy,
    etc.
  },
  className: string,
  style: string,
  events: ?Object,
  dom: ?Node,
}

type VText = {
  text: string,
  dom: ?Node,
}

```

Hopefully this all is starting to make a little sense. If not, don't worry we will get our hands dirty really soon! We will be 
building our own `vDOM` implementation and shit, epic times ahead :v: :surfer:️!

##Constructing the vDOM :office:

Remember that the `DOM` is a tree of nodes. The `vDOM` is a tree of `vNodes`. 
 To built a (v)DOM tree we need to create (v)Nodes.
We can create native `node`s with two methods that you've probably seen before: `document.createElement` and `document.createTextNode`. 
 To built our `vNodes` we need to define two similar methods. We can create two functions which we conveniently call 
 `createVElement` and `createVText` which respectively built and return a `vElement` and `vText` (see the formal definitions above :smile:).

In order to understand the mapping between the DOM and (our) vDOM we can compare them based on the types of `node` and how we could create
these. 
<br>
#### The DOM
|Node|Method|
|---|---|
| HTMLElement |	`document.createElement` |
| TextNode |	`document.createTextNode` |

<br>
#### The VDOM
|Node|Method|
|---|---|
| vElement |`createVElement` |
| vText | `createVText` |

## And now..

We already know that the `vDom` is **not** a replacement of the `DOM`. They have to work together. How? Let's find
out in the next section. 