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
app.route('/posts/:id', function(data) {
  Post.findOne({
    id: data.id
  }).then(function(post) {
    $('#main').html(can.view.stache('<app-post post="{post}"></app-post>', {
      post: post
    }));
  });
});
```

Pro:

Easy to understand and set up

Con:

Route first development: You think about URL design instead of the
overall state of your application. Application state is tied
to route handlers.

## can.AppState

```js
export default new can.AppState({
  define: {
    post: {
      set: function(post) {
        if(post instanceof Post) {
          return post;
        }

        return Post.findOne({ id: id });
      },

      serialize: function() {
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
  <app-post post="{post}"></app-post>
</can-route>
```

Pro: Separate application state
Con: Route definition in template

## `<can-state>`

```html
<can-state route="">
  <a href="post/1">A blog post</a>
</can-route>
<can-state page="post">
  <app-post post="{id}"></app-post>
</can-route>
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
