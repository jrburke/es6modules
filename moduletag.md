# `module`, a tag that should not need to exist

This is in relation to the `<module>` HTML tag proposal outlined in [the module tag explainer](https://github.com/dherman/web-modules/blob/master/module-tag/explainer.md).

The `module` tag should not need to exist. It is a result of core design decisions for ES6 modules that did not accurately weigh existing module use cases, and the actual use cases for modular code mean this tag is of marginal benefit.

These use cases are based on real world uses seen in the AMD and Node module communities.

## Core design issue

The core design issue is the lack of proper inlining of modules in the ES6 module proposal, and a belief that browser module use is different enough from other JS environments to need special add-ons.

However, in the Node community, it is common to want to reuse code written initially for Node in the browser, usually via inlined modules (complete with browser-friendly versions of Node core modules) in the browser.

In the AMD community, some users want to use their same module approach and configuration in the browser and Node. This could be to reuse the same module tree structure for server-based code, or to just unit test some of their modules via a command line Node test harness.

The key point is that module users want a consistent module platform to target, not special requirements that are only needed in some platforms, like the `module` tag and [asset packaging](https://github.com/w3ctag/packaging-on-the-web).

### Module inlining

A bit more on what is meant by module inlining: it is a way to list a few modules in one script.

From the AMD world:

```javascript
define('a', function() {
  return function() {};
});

define('b', function(require) {
  var a = require('a');
  return a() + 'b';
});
```

For ES6 perhaps something like:

```javascript
module 'a' {
  export default function() {};
}

module 'b' {
  import a from 'a';
  export default a() + 'b';
}
```

If inline modules like the above were supported natively in ES6, it removes some of the uses of the <module> tag, and goes toward creating a more common module base.

#### Function analogy

Modules are a way to reuse pieces of functionality. So are functions. Imagine a world where the JS language was originally designed where it did not have named function declarations, but instead you had to use a <function> tag to group functionality into one file:

```html
<function name="multiply" args="a,b">
  return a * b;
</function>

<function name="appstart">
  multiply(1, 2);
</function>
<script>appstart();</script>
```

I believe the same motivations are in play for modules. Modules just happen to use string identifiers instead of language identifiers to refer to other pieces of functionality that they use.

Inlining is just basic requirement of the environment.

[Asset packaging](https://github.com/w3ctag/packaging-on-the-web) is a fine thing to want in general for web delivery, but effective module use in the browser should not be blocked by needing that platform dependency. Plus, not all inlining is about bundled asset delivery, as even the module tag explainer shows.

## In practice

Even if you disagree with some of the reasoning on the core design issue, I do not believe the `module` tag helps with how modules are actually used in the browser, and it just creates more confusion for web developers that are not using modules.

### Areas of confusion

#### Linear vs Tree

Loading of scripts in HTML is as a linear process:

```html
<script src="a.js"></script>
<script src="b.js"></script>
<script src="c.js"></script>
```

The scripts are usually ordered by dependency requirements: c.js may depend on a.js, b.js.

However module loading is a tree process: the module is loaded, but then the tree of dependencies for that module are loaded underneath it.

These two models to not mesh well together:

```html
<script src="a.js"></script>
<module src="b.js"></module>
<script src="c.js"></script>
```

In this example, c.js depends on b.js. b.js creates a global, but uses import to reuse a module-based dependency.

It will be much cleaner to encourage any module use via ES6 Module Loader APIs, like `System.import` instead of allow for these types of mistakes to happen.

Side note: this is also why the ES6 Module Loader would do well to support something like [shim config](https://github.com/amdjs/amdjs-api/wiki/Common-Config#shim-): it is easier to fit a linear module with implicit dependencies into a tree model by allowing the specification of dependencies via a config like shim.

##### "module as a better script" misunderstanding

Developers will hear "`module` as a better `script`" and then try to just upgrade their existing script use to that pattern. However,
this will not work for many existing scripts that rely on just being able to do `var something = ''` to create their global export.

An example is [three.js](http://threejs.org/), which just uses `var THREE = ...` to set up its public export.

If a developer tries to update their HTML to use `<module>` instead of `<script>` tags because they hear it is better, their page will break.

##### Decision tree expansion

Since `module` is an HTML tag, it will show up for any HTML references, and be part of a web developer's decision tree now on including scripts, even if they do not plan to not use any module loading. And given the above, there is a good chance they will use it incorrectly.

However, if they only need to approach module use via using the module constructs in JS, I believe it will keep the choice complexity correctly scoped to people that want to get into module loading, and the in-language JS constructs will help further enforce the difference in loading and execution models.

##### module as a newscript

If the JS language around modules is so different that it needs a new HTML tag, why not change the language further then? Why not just call it `<newscript>` and change more of the language?

I do not want more language changes, but needing a new top level tag for a script opens the door for those discussions, which would just be more distraction from delivering modules.

### Real world use cases

#### Single page apps

The ES6 Module Loader will need to be configured to know where to load the modules from, by setting baseURL for the loader, and likely to set at least one paths config option. Developers do not like hosting their app code in the same directory as their library code.

The [volojs/create-template](https://github.com/volojs/create-template) project template for an AMD-based project is the simplest I can think of making a project like this.

It has one script tag reference to load a script that sets up the loader then loads the main app module.

Adapting to ES6 style:

```html
<script src="app.js"></script>
```

In app.js:

```javascript
System.baseUrl = 'lib';
// Set a paths config here for the 'app/' module ID prefix

System.import('app/main');
```

#### Multiple page apps

For multipage projects, they may want to reuse the same config that sets up baseURL and paths on multiple pages. For ES6, I see it looking like this:

```html
<script src="config.js"></script>
<script>System.import('page1');</script>
```

These uses do not need a new HTML tag to work.

#### Other example module uses

As shown in the "confusion" sections, I do not believe the linear model of script loading via HTML tags translates well to modular loading.

If the user is wanting the nicer encapsulation of modules, and if they need to reuse some scripts that are not modular, there will be less footguns by using something like a shim config for the module loader than using a module tag.

The other examples in the module tag explainer are satisfied with a proper inlinining syntax for modules in the JS language.





