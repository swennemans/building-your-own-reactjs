# Updating

Now that we have *some* idea about an updating UI it's time to refine our implementation. We're going
to implement an `update` function that updates the UI accordingly. Again, it's a simplified version
that most definitly will break when you try hard enough. 

# A very expensive problem!

In [React's docs](https://facebook.github.io/react/docs/reconciliation.html) it is mentioned that updating 
the UI efficiently is an O(n3) problem. Where `n` is the number of elements in the tree. 
So if we would update 100 elements, there would be around 1 million comparisons! 
To solve this problem efficiently React.js uses two assumptions: 

1) Two elements of different types will produce different trees
2) The developer can hint at which child elements may be stable across different renders with a key prop.

The first assumption we will take into account, the second don't. 