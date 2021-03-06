# Mustaches


## What is Mustache?

[Mustache](http://mustache.github.com) is one of the most popular templating languages. It's a very lightweight, readable syntax with a comprehensive specification - which means that implementations (such as Ractive) can test that they're doing things correctly.

Other templating languages borrow liberally from Mustache. [Handlebars](http://handlebarsjs.com) has a mustache-like syntax, as does [Angular](http://angularjs.org).

## What are mustaches?

Within this documentation, and within Ractive's code, 'mustache' means two things - a snippet of a template which uses mustache delimiters, such as `{{name}}`, and the object within our [parallel DOM](parallel DOM.md) that is responsible for listening to data changes and updating the (real) DOM.

We say that the `{{name}}` mustache has a *[reference](references)* of `name`. When it gets rendered, and we create the object whose job it is to represent `name` in the DOM, we attempt to *resolve the reference according to the current context stack*. For example if we're in the `user` context, and `user` has a property of `name`, `name` will resolve to a [keypath](keypaths.md) of `user.name`.

As soon as the mustache knows what its keypath is (which may not be at render time, if data has not yet been set), it registers itself as a *[depedant](dependants.md)* of the keypath. Then, whenever data changes, Ractive scans the dependency graph to see which mustaches need to update, and notifies them accordingly.

As well as simple *interpolators* like `{{name}}`, the other mustaches types - sections, partials, and even delimiter changes - are supported. Consult the [tutorials](http://learn.ractivejs.org) to learn about these.


## Extensions

Ractive is 99% backwards-compatible with Mustache, but adds five additional features (array index references, object iteration, restricted references, ancestor references and expressions) which are detailed below.


### Array Index references

Index references are a way of determining where we are within a list section. It's best explained with an example:

```html
{{#items:i}}
  <!-- within here, {{i}} refers to the current index -->
  <p>Item {{i}}: {{content}}</p>
{{/items}}
```

If you then set `items` to `[{content: 'zero'}, {content: 'one'}, {content: 'two'}]`, the result would be

```html
<p>Item 0: zero</p>
<p>Item 1: one</p>
<p>Item 2: two</p>
```

This is particularly useful when you need to respond to user interaction. For example you could add a `data-index='{{i}}'` attribute, then easily find which item a user clicked on.

### Object Iteration

Mustache can also iterate over objects, rather than array. The syntax is the same as for Array indices. Given the following ractive:

```javascript
ractive = new Ractive({
  el: container,
  template: template,
  data: {
    users: {
      'joe@example.com': { name: 'Joe' },
      'jane@example.com': { name: 'Jane' },
      'mary@example.com': { name: 'Mary' }
    }
  }
});
```

We can iterate over the users object with the following:

```html
<ul>
  {{#users:email}}
    <li>{{email}}: {{name}}</li>
  {{/users}}
</ul>

<!-- becomes... -->
<ul>
  <li>joe@example.com: Joe</li>
  <li>jane@example.com: Jane</li>
  <li>mary@example.com: Mary</li>
</ul>
```

### Restricted references

Normally, references are resolved according to a specific algorithm, which involves *moving up the context stack* until a property matching the reference is found. In the vast majority of cases this is exactly what you want, but occasionally (for example when dealing with [recursive partials](parials.md#recursive-partials)) it is useful to be able to specify that a property must exist *in the current context*.

To restrict a reference to the current context, prefix it with a `.`, e.g. `{{#.bar}}`:

```html
{{#foo}}
  {{#bar}}This section will render, because it will resolve to 'bar'{{/bar}}
  {{#.bar}}This section will NOT render, because it resolves to 'foo.bar'{{/.bar}}
{{/foo}}
```

```js
ractive = new Ractive({
  el: myContainer,
  template: myTemplate,
  data: { bar: true, foo: {} }
});
```


### Ancestor references

Very occasionally, you might need to refer explicitly to a property higher up in the tree to avoid naming conflicts. You can do that by prefixing references with `../` (or `../../`, or `../../../`...):

```html
<h1>Blog posts by {{name}}</p>

<ul class='blog-posts'>
  {{#posts}}
    <li><a href='{{ slugify(../../name) }}/{{ slugify(name) }}'>{{name}}</a></li>
  {{/posts}}
</ul>
```

```js
var ractive = new Ractive({
  el: document.body,
  template: myTemplate,
  data: {
    name: 'Rich',
    posts: [
      { name: 'This is a blog post' },
      { name: 'And so is this' }
    ],
    slugify: function ( str ) {
      var slug;
      /* SOME CODE HAPPENS */
      return slug;
    }
  }
});
```

In this example, some damn fool decided it would be a good idea to give posts a 'name' property as well as authors. Which means that in the `href` attribute, it wouldn't be possible to refer to the root-level `name` property, because it would always resolve to the `name` property of the current post - and that's no good, because the root-level `name` property is used to construct the URL.

By using `../../name` instead of `name`, we're saying 'go up one level (to `posts`), then up one more (to the root), and *then* look for the `name` property'.


### Expressions

Expressions are a big topic, so they have a [page of their own](Expressions.md). But this section is about explaining the difference between vanilla Mustache and Ractive Mustache, so they deserve a mention here.

Expressions look like any normal mustache. For example this expression converts `num` to a percentage:

```html
<p>{{ num * 100 }}%</p>
```

The neat part is that this expression will recognise it has a dependency on whatever keypath `num` resolves to, and will re-evaluate whenever the value of `num` changes.

Mustache fans may bristle at expressions - after all, the whole point is that mustache templates are *logic-less*, right? But what that really means is that the logic is *embedded in the syntax* (what are conditionals and iterators if not forms of logic?) rather than being language dependent. Expressions just allow you to add a little more, and insodoing make complex tasks simple.


## Footnote

*Ractive implements the Mustache specification as closely as possible. 100% compliance is impossible, because it's unlike other templating libraries - rather than turning a string into a string, Ractive turns a string into DOM, which has to be restringified so we can test compliance. Some things, like lambdas, get lost in translation - it's unavoidable, and unimportant.
