# Thunk
Thunks allow the user to take control of the diff'ing process for a specific dom tree, usually to avoid doing calculations you know are unneeded, such as diff'ing a tree you know hasn't changed. Most Thunks will employ memoization of some sort.

## Thunk Interface
A Thunk needs to be an object with two keys

**type**   
must be the string "Thunk"

**render**   
Function that returns a VNode, Widget, or VText.

## Render Method Arguments
When diff is run, the render method is passed a single argument.

**previous**  
The previous VNode, Thunk, Widget, or VText that the Thunk is being diffed against

## Simple Example
As long as the Thunk object has the required interface, it will be treated as a Thunk. Here's an unchanging Thunk.

```javascript
var exampleNode = new VNode("div", null, [new VText("I don't get diffed")])
var unchangingThunk = { type: "Thunk", render: function(previous) { return exampleNode } }
```

Whenever you diff `unchangingThunk` against any other Thunk whose render function returns a reference to `exampleNode`, diff will stop comparing when it sees both `Thunk.render` methods returned the same reference. We never waste time comparing the child VText.

## Full Example
Let's explore how Thunks work by imagining a situation in which we have a static list of books and authors. The user can change the background color of the table, but otherwise the listing will stay the same.

```javascript
diff = require("vtree/diff")
patch = require("vdom/patch")
VNode = require("vtree/vnode")
VText = require("vtree/vtext")
createElement = require("vdom/create-element")
// The current UI state
var state = {
  bg: "#CCC"
}

// Stores previous and memoized values
var memos = {
  bookTable: {}
}

// For making td columns with a text node inside
var makeColumn = function(text) {
  var textNode = new VText(text)
  return new VNode("td", { className: "col" }, [textNode])
}

// For making table rows with book title and author columns
var makeBookRow = function(title, author) {
  return new VNode("tr", { className: "listing" }, [makeColumn(title), makeColumn(author)])
}

// Add some books
var rows = []
rows.push(makeBookRow("Surface Detail", "Iain M. Banks"))
rows.push(makeBookRow("The Quantum Thief", "Hannu Rajaniemi"))
rows.push(makeBookRow("Lock In", "John Scalzi"))

// For making the surrounding table
var makeBookTable = function(bg) {
  var bodyNode = new VNode('tbody', null, rows)
  return new VNode('table', { style: { width: "300px", backgroundColor: bg }}, [bodyNode])
}

// getBookTableThunk memoizes and returns a Thunk object
// Takes bg - the background color
var getBookTableThunk = function(bg) {
  if (!memos.bookTable[bg]) {
    // Here we memoize the Thunk, so whenever the background color changes to a 
    // color already shown, we won't need to build the VNode again
    memos.bookTable[bg] = { type: "Thunk", render: function () { return makeBookTable.call(null, bg) } }
  }
  return memos.bookTable[bg]
}

// A simple function to diff your thunks, and patch the dom
var update = function(nextNode) {
  var patches = diff(memos.currentNode, nextNode)
  patch(memos.rootNode, patches)
  memos.currentNode = nextNode
}

// Create a div container around our Thunk
// This will cause a memo to be created for the current state.bg value - #CCC
containerNode = new VNode("div", { id: "container" }, [getBookTableThunk(state.bg)])

var rootNode = createElement(containerNode)
memos.currentNode = containerNode
memos.rootNode = rootNode

document.body.appendChild(rootNode)

// Schedule some updates
setTimeout(function() { 
    // Here we change the surrounding container, but the Thunk will stay the same. 
    // And because state.bg hasn't changed, none of the Book Table's children (such as the table rows) 
    // will be compared against each other. diff will see that 
    var changedContainer = new VNode("div", 
      { id: "container", style: { border: "2px solid black", width: "300px" }}, 
      [getBookTableThunk(state.bg)])

    update(changedContainer)
  }, 
  1000)

setTimeout(function() { 
    // Now we change state.bg to a new value, our thunk will generate a new VNode, which
    // will be patched as we would expect. Now the background color of the table will be
    // white instead of grey.
    state.bg = "#FFF"
    var changedContainer = new VNode("div", 
      { id: "container", style: { border: "2px solid red", width: "300px" }}, 
      [getBookTableThunk(state.bg)])

    update(changedContainer)
  }, 
  2000)
```

### Thunks Inside Thunks
Did you notice another bit of optimization we could do in the above code? The book list isn't changeable by state, so there is no reason to build it or diff it more than once. Fortunately, Thunks can be nested, so we can edit the `makeBookTable` function and memoize `bodyNode`.

```javascript
var makeBookTable = function(bg) {
  if (!memos.bookBodyThunk) {
    var bodyNode = new VNode('tbody', null, rows)
    memos.bookBodyThunk = { type: "Thunk", render: function(previous) { return bodyNode } }
  }
  return new VNode('table', { style: { width: "300px", backgroundColor: bg }}, [memos.bookBodyThunk])
}

```

Now even when the parent thunk is re-rendered, diff won't waste time comparing all the table rows holding our static book information.

## Other Resources
Raynos has created a library for making Thunks at [vdom-thunk](https://github.com/Raynos/vdom-thunk).
