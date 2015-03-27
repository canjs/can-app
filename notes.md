# can.Application

## Status

### The good

- Splitting your application into small components
  - Components
  - Demo page
  - Tests
  - Documentation
- NPM

### The bad

- How to compose those components into an application?
- Application state not a "first class citizen"

## Application routing

## The lazy way

```js
const renderer = can.view.stache('<app-post post="{post}"></app-post>');

router('/posts/:id', data => {
  Post.findOne({
    id: data.id
  }).then(post => $('#main').html(renderer({ post })));
});
```

## Why lazy?

__The good__

- Easy to understand and set up

__The bad__

- Route first development: You think about URL design instead of application state.
- Application state is tied to route handlers.

## can.AppState

A global application state object

```js
export default new can.AppState({
  define: {
    post: {
      set(id) {
        return Post.findOne({ id });
      },

      serialize() {
        return this.value.attr('id');
      }
    }
  }
});
```

## `<can-route>`

```html
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

## Route matching problems

```js
can.route(':page/:id', { page: 'index' });

if(can.route.attr('page') === 'index') {

} else if(can.route.attr('page') === 'blog') {

}
```

## `<can-state>`

```html
<div class="container">
  <can-state route="" can-animate-out="slideRight">
    <a href="post/1">A blog post</a>
  </can-route>
  <can-state page="post" can-animate-in="slideLeft" can-animate-out="slideRight">
    <app-post post="{id}"></app-post>
  </can-route>
</div>
```

## can.AppState

```js
import AppState from 'can/app/state';

export default new can.AppState({
  define: {
    filter: {
      type: 'string'
    },

    todos: {
      Value: Todo.List
    },

    displayTodos: {
      get: function() {
        return this.attr('todos').filter(function(todo) {
          return todo.attr('filter') === this.attr('filter');
        });
      }
    }
  },

  isBlogPost10: function() {
    return this.attr('page') === 'blog' && this.attr('post.id') === 10;
  },

  routes: {
    ':filter': {
      filter: 'all'
    }
  }
});
```

## Wiring it up

```html
<script src="steal.js" main="can"></script>
<script id="main" type="text/stache" autorender>
  <can-import from="app" />
  <todo-app todos="{displayTodos}" filter="{filter}"></todo-app>
</script>
```

## SamsClub


```html
<script src="steal.js" main="app!canjs"></script>
<script id="main" type="text/stache" autorender>
  <div class="container">
    <can-state page="blog" can-import="app/blog" can-animate-in="slideLeft" can-animate-out="slideRight">

    </can-state>
    <can-state isBlogPost10 animate-in="slideLeft" animate-out="slideRight">
    </can-state>
  </div>

  <can-state route="">
    <home-manager state="{.}"></home-manager>
  </can-state>
  <can-state page="locator">
    <club-finder account="{account}" club-id="{clubId}" active-tab="{locatorPage}"></club-finder>
  </can-state>
  <can-state page="club">
    <club-details account="{account}" club-id="{clubId}"></club-details>
  </can-state>
  <can-state page="clubpickup">
    <clubpickup-manager state="{.}"/>
  </can-state>
</script>
```
