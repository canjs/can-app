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
* Make routing / application state clearer
* Transitioning between pages
* Making bootstrapping more declarative

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
- Application bootstrap not easily testable

--

## Application routing

--

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
export default can.AppState.extend({
  define: {
    post: {
      serialize: false,
      get(id) {
        return Post.findOne({ id: this.attr('postId') });
      }
    }
  }
});
```

--

## AppState and routing

```javascript
const main = can.mustache('<app-main></app-main>');
const blog = can.mustache('<app-post post="{post}"></app-post>');
const appState = new AppState();

can.route.map(appState);

can.route(':page/:postId', { page: 'index' });

if(can.route.attr('page') === 'index') {
  $('#main').html(main(appState));
} else if(can.route.attr('page') === 'blog') {
  System.import('app/blog').then(() => $('#main').html(blog(appState)));
}
```

--

## `<can-route>`

```javascript
<can-route url="">
  <a href="{{makeUrl 'post' '1'}}">A blog post</a>
</can-route>
<can-route url="posts/:post">
  {{#if post.isPending}}
    <app-loader>Loading blog post</app-loader>
  {{else}}
    <app-blog post="{post.value}"></app-blog>
  {{/if}}
</can-route>
```

- __Good:__ Separate application state
- __Bad:__ Route definition in template

--

## Routes in appState

```javascript
// app/main.js
export default can.AppState.extend({
  define: {
    post: {
      serialize: false,
      get(id) {
        return Post.findOne({ id: this.attr('postId') });
      }
    }
  },
  routes: {
    ':page/:postId': {
      page: 'index'
    }
  }
});
```

--

## `<can-state>`

Component for matching properties, animating transitions and importing modules.

```javascript
<div class="container">
  <can-state page="index" can-animate-in="fadeIn" can-animate-out="slideRight">
    <a href="{{makeUrl 'blog' '1'}}">A blog post</a>
  </can-state>
  <can-state page="blog" can-import="app/blog"
      can-animate-in="slideLeft" can-animate-out="fadeOut">
    {{#if post.isPending}}
      <app-loader>Loading blog post</app-loader>
    {{else}}
      <app-blog post="{post.value}"></app-blog>
    {{/if}}
  </can-state>
</div>
```

--

## index.html

```javascript
<!DOCTYPE html>
<html>
  <head>
    <title>Bitovi Blog</title>
  </head>
  <body>
    <script src="node_modules/steal/steal.js" data-main="can"></script>
    <script type="text/stache" autorender>
      <h1>Blog</h1>
      <can-import from="app/main" class="container">
        <can-state page="index">
          <a href="{{makeUrl 'blog' '1'}}">A blog post</a>
        </can-state>
        <can-state page="blog" can-import="app/blog">
          <app-blog post="{post}"></app-blog>
        </can-state>
      </can-import>
    </script>
  </body>
</html>
```

--
## Animations

[can-animate](https://github.com/bitovi/can-animate)

```
<can-state route="">
  <home-manager can-animate-fade-in can-animate-slide-out></home-manager>
</can-state>

<can-state page="locator">
  <club-finder can-animate-slide-in can-animate-fade-out></club-finder>
</can-state>
```
--
### The good

- Fine grained control over how to transition each top level element.

### The bad

- Painful if you have a bunch of `can-state` tags.
--

## What our competitors do?

### Ember

[liquid-fire](http://ef4.github.io/liquid-fire/#/)

- Transitions are implemented within the view (`Template helpers`),
the 'kind of' transition is defined in the `Transition map` which maps
routes to transitions.

```
{{#liquid-outlet}} -> Transitions between routes.
{{#liquid-with}}   -> Transitions between models or contexts within a single route.
{{#liquid-bind}}   -> Updates to simple bound values.
{{#liquid-if}}     -> Switching between true and false branches in an #if statement.
{{#liquid-spacer}} -> Provides a smoothly growing/shrinking container that
                      animates whenever its contained DOM mutates.
```
--
### Transition map

```
export default function(){
  this.transition(
    this.fromRoute('people.index'),
    this.toRoute('people.detail'),
    this.use('toLeft'),
    this.reverse('toRight')
  );
};
```
--
### Defining custom transitions

```
import { animate, stop } from "liquid-fire";

export default function fade(oldView, insertNewView) {
  stop(oldView);
  return animate(oldView, {opacity: 0})
    .then(insertNewView)
    .then(function(newView){
      return animate(newView, {opacity: [1, 0]});
    });
}
```
--

### Angular

[ngAnimate](https://docs.angularjs.org/api/ngAnimate)

- Core directives provide hooks for css animations, basically it just
adds a `ng-enter` or `ng-leave` class when element is being either
added to or removed from the DOM.

```
<div ng-repeat="item in items" class="repeated-item">
  {{ item.id }}
</div>
```

```
.repeated-item.ng-enter, .repeated-item.ng-move { /* transition rules */ }
.repeated-item.ng-leave { /* transition rules */ }
```
--

### React

- Heavily inspired in `ngAnimate`, also relies on CSS classes.

```
var ReactCSSTransitionGroup = React.addons.CSSTransitionGroup;
var TodoList = React.createClass({
  render: function() {
    return (
      <div>
        <button onClick={this.handleAdd}>Add Item</button>
        <ReactCSSTransitionGroup transitionName="example">
          {items}
        </ReactCSSTransitionGroup>
      </div>
    );
  }
});
```

```
.example-enter { /* item added to the DOM */ }
.example-leave { /* item removed from the DOM */ }
```
--

## `can-transition`

- A 'syntax shortcut' to define the same animation behavior for a group of states.

```
<can-transition in="fade-in" out="slide-out">
  <can-state page="">
    <home-manager state="{.}"></home-manager>
  </can-state>

  <can-state page="locator">
    <club-finder></club-finder>
  </can-state>
</can-transition>
```
--

## `can-transition`

- Defining a custom target

```
<can-transition in="slide-in" out="slide-out" target=".container > app-*">
  <div class="container">
    {{#if foo}} <app-foo></app-foo> {{/if}}
    {{#if bar}} <app-bar></app-bar> {{/if}}
    {{#if baz}} <app-baz></app-baz> {{/if}}
  </div>
</can-transition>
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
