title: can.App
output: index.html
theme: theme
controls: false
logo: theme/logo.png

--

# can.App

--

## Templating
### Stache / Mustache

#### The Good

* Elegantly solves building DOM from strings.

--

### Stache / Mustache
#### The Bad

1. Cannot be written directly in HTML.
  * Some uses aren't possible
    * Wrapping attributes: `{{#if true}}foo="bar"{{/if}}`
    * Can't do `<tbody>{{#each}}<tr></tr>{{/each}}</tbody>`
  * Flash of unstyled content
  * Cannot use in `<template>` (lose advantages that has)

--

### HTML-based Templates
#### The Good

* Can use directly in the DOM.

#### The Bad

* HTML is a bit limited for building DOM structure.
  * Everything is an Element, Attribute, or TextNode.

--

### `{{key}}`

* `{{key}}` - Angular
* `${key}` - ES6 string templating

--

### `{{#each}}`

#### Them
* `repeat="key"` - Angular / Polymer
  * `each="key"`
  * `for-each="key"`
* `repeat="k of key"` - Localized naming, introduces expressions

#### Us
* `<can-each>` - Easiest to implement
* `@each="key"`

--

### `{{#if}} / {{^if}}`

* `if="expr"` - Same problem as repeat
* `<can-if>` - Only for blocks, same problem as `{{#if}}`

#### Attributes

* `class="{{#if expr}}sidebar{{/if}}"`
* `class="{expr ? "sidebar" : ""}"` - ternary

--

### Exporting component's viewModel

[Issue #1362](https://github.com/bitovi/canjs/issues/1362)

* `can-scope="name"` - Current proposal
  * Get a specific property of the scope: `can-scope-*=""`, ex: `<todos can-scope-length="todosCount">`
* `ref="name"` - problem: because we pass data in attributes hard to add non `can-` attrs.
* `@bind="name"`
* `#name` - Angular: `<todos #todos />`
* `[length]="todosCount"` - Extract a specific property
* `[]="name"` - Full viewModel
* `[.]="name"` - full vm
* `[@viewModel]="name"` -extract full vm (`[@events]="todosEvents"`)

--
