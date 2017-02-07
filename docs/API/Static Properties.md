# Ractive.adaptors

_`(Object<string, Object>)`_

The registry of globally available [adaptors](../Plugin Authoring/Adaptors.md).

---

# Ractive.components

_`(Object<string, Function>)`_

The registry of globally available [component definitions](../Plugin Authoring/Components.md).

---

# Ractive.DEBUG

_`(boolean)`_

Tells Ractive if it's in debug mode or not. When set to `true`, non-fatal errors are logged. When set to `false`, non-fatal errors are suppressed. By default, this is set to `true`.

---

# Ractive.DEBUG_PROMISES

_`(boolean)`_

Tells Ractive to log errors thrown inside promises. When set to `true`, errors thrown in promises are logged. When set to `false`, errors inside promises are suppressed. By default, this is set to `true`.

---

# Ractive.decorators

_`(Object<string, Function>)`_

The registry of globally available [decorators](../Plugin Authoring/Decorators.md).

---

# Ractive.defaults

_`(Object<string, any>)`_

The defaults for [initialisation options](../API/Initialization Options.md) with the exception of [plugin registries](../Integrations/Plugins.md).

```js
// Change the default mustache delimiters to [[ ]] globally
Ractive.defaults.delimiters = [ '[[', ']]' ];

// Future instances now use [[ ]]
ractive1 = new Ractive({
    template: 'hello [[world]]'
});
```

Configurations set on the instance override the ones present in `Ractive.defaults`.

```js
Ractive.defaults.delimiters = [ '[[', ']]' ];

// Uses the delimiters specified above
new Ractive({
	template: 'hello [[world]]'
});

// Uses the delimiters specified in the init options
new Ractive({
	template: 'hello //world\\',
	delimiters: [ '//', '\\' ]
});
```

Defaults can also be specified a subclass of Ractive.

```js
var MyRactive = Ractive.extend();

MyRactive.defaults.el = document.body;
```

---

# Ractive.easing

_`(Object<string, Function>)`_

The global registry of [easing functions](../Plugin Authoring/Easings.md).

The easing functions are used by the [`ractive.animate`](../API/Instance Methods.md#ractive.animate) method and by [transitions](../Plugin Authoring/Transitions.md). Four are included by default: `linear`, `easeIn`, `easeOut` and `easeInOut`.

---

# Ractive.events

_`(Object<string, Function>)`_

The global registry of [events](../Plugin Authoring/Events.md).

---

# Ractive.interpolators

_`(Object<string, Function>)`_

A key-value hash of interpolators use by [`ractive.animate()`](../API/Instance Methods.md#ractive.animate()).

---

# Ractive.length

TODO

---

# Ractive.magic

TODO

---

# Ractive.name

TODO

---

# Ractive.partials

_`(Object<string, string|Object|Function>)`_

The global registry of [partial templates](../Plugin Authoring/Partials.md).

Like [templates](../Under The Hood/Templates/Templates.md), partials are [parsed](../Under The Hood/Templates/Parsing.md) at the point of use. The parsed output is cached and utilized for future use.

---

# Ractive.Promise

_`(Function)`_

A spec-compliant Promise implementation.

---

# Ractive.svg

_`(boolean)`_

Indicates if the browser supports SVG.

---

# Ractive.transitions

_`(Object<string, Function>)`_

The global registry of [transition functions](../Plugin Authoring/Transitions.md).

---

# Ractive.VERSION

The version of the currently loaded Ractive.
