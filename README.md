# virtual-dom documentation
This documentation is aimed at people who would like to work with virtual-dom directly, or gain a deeper understanding of how their virtual-dom based framework works. If you would rather be working at a higher level, you may find the [mercury framework](https://github.com/Raynos/mercury) a better place to start.

## Overview

virtual-dom consists of four main parts:

[vtree](https://github.com/Matt-Esch/virtual-dom/tree/master/vtree) - responsible for diffing two virtual representations DOM nodes  
[vdom](https://github.com/Matt-Esch/virtual-dom/tree/master/vdom) - responsible for taking the [patch](https://github.com/Matt-Esch/virtual-dom/blob/master/vdom/patch.js) genereated by [vtree/diff](https://github.com/Matt-Esch/virtual-dom/blob/master/vtree/diff.js) and using it to modify the rendered DOM  
[vnode](https://github.com/Matt-Esch/virtual-dom/tree/master/vnode) - virtual representation of dom elements  
[virtual-hyperscript](https://github.com/Matt-Esch/virtual-dom/tree/master/virtual-hyperscript) - an interface for generating VNodes from simple data structures

Newcomers should start by reading the VNode and VText documentation, as virtual nodes are central to the operation of virtual-dom. Hooks, Thunks, and Widgets are more advanced features, and you will find both documentation of their interfaces and several examples on their respective pages.

## Contents

[VNode](https://github.com/littleloops/virtual-dom-docs-wip/blob/master/vnode.md) - A representation of a DOM element

[VText](https://github.com/littleloops/virtual-dom-docs-wip/blob/master/vtext.md) - A representation of a text node

[h](https://github.com/littleloops/virtual-dom-docs-wip/blob/master/virtual-hyperscript.md) (virtual-hyperscript) - A concise way to generate VNode and VText

[Hooks](https://github.com/littleloops/virtual-dom-docs-wip/blob/master/hooks.md) - The mechanism for executing functions after a new node has been created

[Thunk](https://github.com/littleloops/virtual-dom-docs-wip/blob/master/thunk.md) - The mechanism for taking control of diffing a specific DOM sub-tree

[Widget](https://github.com/littleloops/virtual-dom-docs-wip/blob/master/widget.md) - The mechanism for taking control of node patching: DOM Element creation, updating, and removal.
