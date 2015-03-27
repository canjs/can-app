title: SPA
output: index.html
theme: theme
controls: false
logo: theme/logo.png

--

# Single Page Applications

--

## Problems to Solve

* Bootstrapping Application
* Make routing easier
* Transitioning between pages
* Allow Components to be Written into DOM

--

# can.Application

--

## Status

#### The good

- Splitting your application into small components
  - Components
  - Demo page
  - Tests
  - Documentation
- NPM

#### The bad

- How to compose those components into an application?
- Application state not a "first class citizen"

--

## Application routing

## The lazy way

```javascript
const renderer = can.view.stache('<app-post post="{post}"></app-post>');

router('/posts/:id', data => {
  Post.findOne({
    id: data.id
  }).then(post => $('#main').html(renderer({ post })));
});
```

--

## Why lazy?

### The good

- Easy to understand and set up

### The bad

- Not bidirectional
- Route first development: You think about URL design instead of application state.
- Application state (and rendering) is tied to route handlers.

--

## can.AppState

A global application state object

```javascript
export default new can.AppState({
  define: {
    post: {
      set(id) {
        return Post.findOne({ id });
      },

      serialize() {
        return this.attr('post').value.attr('id');
      }
    }
  }
});
```

--

## `<can-route>`

```
<can-route url="">
  <a href="post/1">A blog post</a>
</can-route>
<can-route url="posts/:id">
  {{#if post.isPending}}
    <app-loader>Loading blog post</app-loader>
  {{else}}
    <app-post post="{post.value}"></app-post>
  {{/if}}
</can-route>
```

Pro: Separate application state
Con: Route definition in template

--

## Route matching problems

```javascript
can.route(':page/:id', { page: 'index' });

if(can.route.attr('page') === 'index') {

} else if(can.route.attr('page') === 'blog') {

}
```

--

## `<can-state>`

```
<div class="container">
  <can-state route="" can-animate-out="slideRight">
    <a href="post/1">A blog post</a>
  </can-route>
  <can-state page="post" can-animate-in="slideLeft" can-animate-out="slideRight">
    <app-post post="{id}"></app-post>
  </can-route>
</div>
```

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

## Implementing Templates


