# Mounting :love_hotel:

We have formal types for our `vNodes` methods needed to built each type of `vNode`. 
However, the `vNodes` are just simple JavaScript objects and we cannot magically merge
these into the native `DOM`. The browser needs actual **native** `nodes` in order to display HTML.

> The browser cannot display the Virtual DOM directly!

To create these native `nodes`, the `vNodes` need to be **mounted**.

If you've worked with React before you've encountered the lifecycle methods: `componentWillMount()` 
and/or `componentDidMount()`. These methods probably give you some hint what **mounting** could mean. 

It is the process where the **native** DOM `nodes` are created (from `vNode`'s) and inserted in 
the **native** DOM. 
<br>
## DOM and vDOM relationship :heart:

It is important to understand that there is a constant long-lasting relationship between the `DOM` and `vDOM`. 
Within this relationship I like to see the `vDOM` as the boss. 

> Actually this is not 100% accurate. In React.js we can use `refs` to manipulate the `DOM` directly. But in our implementation we
we will not implement refs (for now).

What does it mean the the `vDOM` is the boss? We can construct our `vDOM` with JavaScript and the `DOM` will follow suit. 
Meaning, the `DOM` will take the shape we define in our `vDOM`. And this is something we like very much, 
because we can use all the power that JavaScript has to offer. 

## From vDOM 🕹 to DOM 📺

At this stage we've already have discussed the ingredients to build a `DOM` from a `vDOM`. But
probably this is still way too vague. Don't worry! In the next secion we will build our own `vDOM` implementation. 
Epic times are almost here :v:.