# VNode

A VNode is a representation of a dom node. Most end users will probably have their VNodes generated through a templating language (such as [virtual-hyperscript](https://github.com/Raynos/virtual-hyperscript/)), but understanding the VNode interface can be useful.

In virtual-dom, VNodes are turned from virtual nodes into real nodes with the createElement function. You can read the code at [vdom/create-element](https://github.com/Matt-Esch/vdom/blob/master/create-element.js)

## Arguments

**tagName**
A string, e.g. 'div', 'span', 'article'

**properties** *(optional)*
An object that maps property names to their values, e.g. `{ id: "foo", className: "bar" }`

**children** *(optional)*
An array of any combination of VNodes, VText, Widgets, or Thunks

**key** *(optional)* 
An identifying string used to differentiate this node from others. Used internally by vtree/diff to do node reordering.

**namespace** *(optional*) A string specifying the namespace uri to associate with the element. VNodes created with a namespace will use [Document.createElementNS](https://developer.mozilla.org/en-US/docs/Web/API/document.createElementNS)

### A full example
```javascript
var VNode = require("vtree/vnode")
var createElement = require("vdom/create-element")

var Hook = function(){}
Hook.prototype.hook = function(elem, key, previousValue) {
  console.log("Hello from " + elem.id + "!\nMy key was: " + key)
}

var tagName = "div"
var style = "width: 100px; height: 100px; background-color: #FF0000;"
var attributes = {"class": "red box", style: style }
var key = "my-unique-red-box"
var namespace = "http://www.w3.org/1999/xhtml"
var properties = {
  attributes: attributes,
  id: "my-red-box",
  "a hook can have any property key": new Hook()
}
var childNode = new VNode("div", {id: "red-box-child"})

RedBoxNode = new VNode(tagName, properties, [childNode], key, namespace)
RedBoxElem = createElement(RedBoxNode)
document.body.appendChild(RedBoxElem)

// Will result in html that looks like
// <div class="red box" style="width: 100px; height: 100px; background-color: #FF0000;" id="my-red-box">
//   <div id="red-box-child"></div>
// </div>
```


### properties
Keys in properties have several special cases. 
#### properties.attributes
Everything in the `properties.attributes` object will be set on the rendered dom node using `Element.setAttribute(key, value)`, and removed using `Element.removeAttribute`. Refer to [MDN HTML Attributes](https://developer.mozilla.org/en-US/docs/Web/HTML/Attributes) for available attributes. 

#### Hook Objects
Any key whose value is an object with an inherited key called "hook" is considered a hook. Hooks are used to run functions at render time. Refer the the [hook documentation](https://github.com/littleloops/virtual-dom-docs-wip/blob/master/README.md) for more information.

#### Other Objects
Any key in `properties` that is an object, but whose value isn't a hook and isn't in the attributes key, will set the rendered elements property to the given object.

```javascript
createElement(new VNode('div', { style: { width: "100px", height: "100px"}}))
// When added to the dom, the resulting element will look like <div style="width: 100px; height: 100px"><div>
```

#### Other Values
Most attributes can be set using properties. During element creation, keys and values in the `properties.attributes` get set using `Element.setAttribute(key, value)`, whereas keys other than `attributes` present in properties get set using `Element[key] = value`.

```javascript
foo = new VNode('div', { id: 'foo'})
// will render the same as
foo = new VNode('div', { attributes: { id: 'foo' }})
```

This can, however, cause some unexpected behavior. We have documented some common differences below.

### properties.style vs properties.attributes.style
`properties.style` expects an object, while `properties.attributes.style` expects a string.

```javascript
// using attributes
redBox = new VNode('div', { attributes: { style: "width: 100px; height: 100px; background-color: #FF0000;" }})
// using a property
anotherRedBox = new VNode('div', { style: { width: "100px", height: "100px", backgroundColor: "#FF0000" }})
```

### properties.className vs properties.attributes.class
Set the class attribute value using `className` key in properties, and the `class` key in attributes.

```javascript
redBox = new VNode('div', { className: "red box" })
// will render the same as
anotherRedBox = new VNode('div', { attributes: { "class": "red box" } })
```

### Custom attributes (data-\*)
Custom attributes won't be rendered if set directly in properties. Set them in properties.attributes.

```javascript
doThis = new VNode('div', { attributes: { "data-example": "I will render" } })
notThis = new VNode('div', { "data-example": "I will not render" })
```

### ARIA attributes
Like custom attributes, ARIA attributes also must be set in `properties.attributes`.

```javascript
ariaExample = new VNode('div', { attributes: { "aria-checked": "true" } })
```

### properties.value, properties.defaultValue, and properties.attributes.value
If an input element is reset, its value will be returned to its value set in `properties.attributes.value`. If you've set the value using `properties.value`, this will not happen. However, you may set `properties.defaultValue` to get the desired result.
