# Components

In many situations, you want to encapsulate behaviour and markup into a single reusable *component*, which can be dropped in to Ractive applications.

First, you need to **create the component** with [Ractive.extend()](Ractive.extend().md), which allows you to define a template, initialisation behaviour, default data, and so on. (If you're not familiar with Ractive.extend() you should read that page first.)

## Initialisation Options

### beforeInit *`Function`*

beforeInit will be passed all the data that will initialize the particular component. This is where you should modify the component with dynamic logic.
This could include setting/changing partials, adding decorators, adding other subcomponents, or anything you would use to initialise a normal Ractive instance. This is great for dynamic templating, an example can be found below.


### init *`Function`*

Init is called after the component has been initialised. This means that `this` will reference that component instance. This allows you to observe on, or listen on events/keypaths for this particular component.

### isolated *`Boolean`*

Isolated is defaulted to false. This applies to your template scope being isolated to only the data that is passed in.
For example if you wanted to access a particular variable on your parent data as if it existed on your component then setting isolated false would allow this. However if you wanted your template scoped to only data you give it so that everything was very module then set isolated to true.


Setting isolated to false

```js


var Component = Ractive.extend({
  isolated: false,
  template: '#component'

});

var ractive = new Ractive({
  el: 'body',
  template: '#template',
  data: {
    nonIsolatedSetting: 'This is a non isolated setting, get it anywhere'
  }
});

```

```html 

<script id="template" type="text/ractive">
  <div>
    <Component>
  </div>
</script>

<script id="component" type="text/ractive">

  {{nonIsolatedString}}

</script>

```

The result of this would be

```html
  <div>
    This is a non isolated setting, get it anywhere
  </div>
```


But if we used the same example and set isolated to true
```js

var Component = Ractive.extend({
  isolated: true,
  template: '#template'
})

```

Then the resulting output would be

```html
  <div>
  </div>
```

Very complex example for such a simple idea but this could drastically effect you.


init is called after the extended compoent has been initialized 

```js
var MyWidget = Ractive.extend({
  template: '<div on-click="activate">{{message}}</p>',
  init: function () {
    this.on( 'activate', function () {
      alert( 'Activating!' );
    });
  },
  data: {
    message: 'No message specified, using the default'
  }
});
```

Then, you need to **register the component** by adding it to `Ractive.components`:

```js
Ractive.components.widget = MyWidget;

// or, if you don't want to make it globally available,
// you can use it with only a single Ractive instance
var ractive = new Ractive({
  el: 'body',
  template: mainTemplate,
  components: {
    widget: MyWidget
  }
});
```

Finally, you **use the component** by referring to it within another template:

```html
<div class='application'>
  <h1>I've got a widget</h1>
  <widget message='Click to activate!'/>
</div>
```

Using the 'widget' component like this is more or less equivalent to doing the following:

```js
new MyWidget({
  el: document.querySelector( '.application' ),
  append: true, // so the component doesn't nuke the <h1>
  data: {
    message: 'Click to activate!'
  }
});
```

## Binding

Instead of passing a static message ('Click to activate!') through, we could have passed a dynamic one:

```html
<widget message='{{foo}}'/>
```

In this case, the value of `message` within the component would be bound to the value of `foo` in the parent Ractive instance. When the value of parent-`foo` changes, the value of child-`message` also changes, and vice-versa.

### Data Context
A new data context gets created for the component, so any parameters don't pollute the primary data (bound values still update):

```js
var data = {
    name: 'Colors',
    colors: ['red', 'blue', 'yellow'],
    style: 'tint'
}
Ractive.components.widget = Ractive.extend({})
var ractive = new Ractive({ 
    template: '<widget items='{{colors}}' option1='A' option2='{{style}}'/>',
    data: data 
})

function logData(){
    console.log('main', JSON.stringify(ractive.data))
    console.log('widget', JSON.stringify(ractive.findComponent('widget').data))
}

logData()
// main {"name":"Colors","colors":["red","blue","yellow"],"style":"tint"}
// widget {"items":["red","blue","yellow"],"option1":"A","option2":"tint"}

ractive.set('colors.1', 'green')

logData()
// main {"name":"Colors","colors":["red","green","yellow"],"style":"tint"}
// widget {"items":["red","green","yellow"],"option1":"A","option2":"tint"} 
```


## Events

You can subscribe to events that emanate from a component in the normal way. For example, our widget template contains a `<div>` element with an `on-click="activate"` directive. We're listening for that event inside the MyWidget `init` method, but we can also listen for it like so:

```html
<widget on-activate="doSomething"/>
```
```js
ractive.on( 'doSomething', function () {
  alert( 'Widget activated!' );
});
```

(The same goes for any events that are fired programmatically by the component, rather than as a result of user interaction.)

## The `{{>content}}` Directive 

Any content in the template calling the component:

```html
<list items='{{gunas}}''>
    {{name}} ({{qualities}})
</list>

<list items='{{fates}}'>
    "{{.}}"
</list>
``` 
is exposed in the component as a partial named "content":

```html
<!-- template for "list" component -->
<ul>
    {{#items}}
    <li>{{>content}}</li>
    {{/}}
</ul>
```

A partial, or component, can be used as well:
```html
<list items='{{gunas}}''>
    {{>partialA}})
</list>

<list items='{{fates}}''>
    <widget value='{{.}}'/> 
</list>
```

Currently, the content markup resolves in the component's context. In the second example above the `value` parameter for `<widget>` is the individual list item of `{{fates}}`. And in the first case, the partial would need to be defined globally (`Ractive.partials`) or on the component, as it won't be found if it is defined, for example, inline in the outer context:

```html
<list items="{{fates}}">
    {{>partialB}}
</list>
<!-- {{>partialB}} -->
    //won't be found by list component,
    //because this markup is handed off 
    //and rendered inside component
<!-- {{/partial}} -->
```
Be aware that the context scope for resolution may change or be expanded in a future release to control whether it resolves in the calling or component context. 

## Pseudo Dynamic Templating

In some cases you want to render a different template depending on the data that is passed in.

### Form Element Component
```js
var formElement = Ractive.extend({
  template: '{{>element}}',
  setTemplate: function(options) {
    options.partials.element = options.data.template;
  },
  beforeInit: function(options) {
    this.setTemplate(options)
  }
});
```

### Form Template
```html
{{#items}}
<formElement value="{{value}}" template="{{template}}"/>
{{/items}}
```
### Ractive Form
```js
var ractive = new Ractive({
  el: 'body',
  template: formTemplate,
  components: {
    formElement: formElement
  }
  data: {
    items: [
        {
          template: '<h1>{{value}}</h1>',
          value: 'This is a title'
        },
        {
          template: '<input type="text" value="{{value}}" style="display:block; clear: both;" />',
          value: 'Input Value'
        },
        {
          template: '<textarea value="{{value}}"></textarea>',
          value: 'Textarea Value'
        }
    ]
  }

});

ractive.observe('items.*.value', function(newValue, oldValue, keypath) {
  console.log(keypath + ' ' + newValue);
});
```
## Caveat

Components are a stable feature, but still relatively new - certain aspects have yet to be implemented. Pardon the dust! There is a [long-running GitHub issue](https://github.com/RactiveJS/Ractive/issues/74), which may be of interest.
