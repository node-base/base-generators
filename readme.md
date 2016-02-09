# base-generators [![NPM version](https://img.shields.io/npm/v/base-generators.svg)](https://www.npmjs.com/package/base-generators) [![Build Status](https://img.shields.io/travis/jonschlinkert/base-generators.svg)](https://travis-ci.org/jonschlinkert/base-generators)

> Adds project-generator support to your `base` application.

You might also be interested in [base-task](https://github.com/node-base/base-task).

- [Install](#install)
- [Usage](#usage)
- [Examples](#examples)
  * [Tasks](#tasks)
  * [Generators](#generators)
  * [Sub-generators](#sub-generators)
- [In the wild](#in-the-wild)
- [API](#api)
- [Related projects](#related-projects)
- [Running tests](#running-tests)
- [Contributing](#contributing)
- [Author](#author)
- [License](#license)

_(TOC generated by [verb](https://github.com/verbose/verb))_

## Install

Install with [npm](https://www.npmjs.com/):

```sh
$ npm i base-generators --save
```

## Usage

```js
var generators = require('base-generators');
var Base = require('base');

// register the plugin before instantiating, to make 
// sure the plugin is called on all Generator instances 
Base.use(generators());
var base = new Base();
```

## Examples

All examples assume the following code is defined:

```js
var Base = require('base');
var generators = require('base-generators');

Base.use(generators());
var base = new Base();
```

### Tasks

Tasks are exactly the same as [gulp](http://gulpjs.com) tasks, and are powered by [bach](https://github.com/phated/bach) and [composer](https://github.com/jonschlinkert/composer).

**Register a task:**

```js
base.task('default', function(cb) {
  // do stuff
  cb();
});
```

**Run a task:**

```js
base.build('default', function(err) {
  if (err) throw err;
});
```

### Generators

> I heard you liked tasks, so I put some tasks in your tasks.

**What's a generator?**

Generators are functions that are registered by name, and are used to encapsulate and organize code, [tasks](#tasks), other generators, or [sub-generators](#sub-generators), in a sharable, publishable and easily re-usable way.

In case it helps, here are some [live examples](#in-the-wild).

**Register a generator:**

```js
base.register('foo', function(app, base) {
  // `app` is the generator's "private" instance
  // `base` is a "shared" instance, accessible by all generators
});
```

**Get a generator:**

```js
var foo = base.generator('foo');
```

**Register tasks in a generator:**

```js
base.register('foo', function(app, base) {
  app.task('default', function() {});
  app.task('one', function() {});
  app.task('two', function() {});
});
```

**Run a generator's tasks:**

The `.generate` method simply calls the `.build` method on a specific generator.

To run a generator's tasks, pass the generator name as the first argument, and optionally define one or more tasks as the second argument. _(If no tasks are defined, the `default` task is run.)_

```js
// run the "default" task on generator "foo"
base.generate('foo', function(err) {
  if (err) throw err;
  console.log('done!');
});

// or specify tasks
base.generate('foo', ['default'], function() {);
base.generate('foo', ['one', 'two'], function() {);
```

Alternatively, you can call `.build` on the generator directly:

```js
// run the "default" task on generator "foo"
base.generator('foo')
  .build('default', function(err) {
    if (err) throw err;
  });
```

### Sub-generators

Sub-generators are just generators that are registered on (or invoked within) another generator instance.

**Register sub-generators:**

Register generators `one`, `two`, and `three` on generator `foo`:

```js
base.register('foo', function(app, base) {
  app.register('one', function() {});
  app.register('two', function() {});
  app.register('three', function() {});
});
```

**Get a sub-generator:**

Use dot-notation to get a sub-generator:

```js
var one = base.generator('foo.one');
```

Sub-generators may be nested to any level. In reality, you probably won't write code like the following example, but this only illustrates the point that generators are extremely composable, and can be built on top of or with other generators.

```js
base.register('a', function(a, base) {
  // do stuff
  a.register('b', function(b) {
    // do stuff
    b.register('c', function(c) {
      // do stuff
      c.register('d', function(d) {
        // do stuff
        d.register('e', function(e) {
          // arbitrary task
          e.task('default', function(cb) {
            console.log('e > default!');
            cb();
          });
        });
      });
    });
  });
});

base.getGenerator('a.b.c.d.e')
  .build(function(err) {
    if (err) throw err;
    // 'e > default!'
  });
```

**Register tasks on sub-generators:**

```js
base.register('foo', function(app, base) {
  app.register('one', function(one) {
    one.task('default', function() {});
    one.task('a', function() {});
    one.task('b', function() {});
    one.task('c', function() {});
  });

  app.register('two', function(two) {
    two.task('default', function() {});
  });

  app.register('three', function(three) {
    three.task('default', function() {});
  });
});
```

**Run a sub-generator's tasks**

```js
// run the `default` task from sub-generator `foo.one`
base.generate('foo.one', function(err) {
  if (err) throw err;
  console.log('done!');
});
```

**Run multiple tasks on a sub-generator:**

```js
// run tasks `a`, `b` and `c` on sub-generator `foo.one`
base.generate('foo.one', ['a', 'b', 'c'], function(err) {
  if (err) throw err;
  console.log('done!');
});
```

## In the wild

Checked off as they're added:

* [ ] generate: adds a CLI, template rendering, fs methods and generator convenience-methods to base-generators
* [ ] assemble: site generation
* [ ] verb: documentation generation
* [ ] update: renames generators to "updaters", which are used to keep your project up-to-date

## API

### [.resolve](index.js#L74)

Attempts to find a generator with the given `name` and `options`.

**Params**

* `name` **{String}**: Can be a module name or filepath to a module that is locally or globally installed.
* `options` **{Object}**
* `returns` **{Object}**: Returns the filepath to the generator, if found.

**Example**

```js
// resolve by generator "alias"
app.resolve('foo');

// resolve by generator name
app.resolve('generate-foo');

// resolve by filepath
app.resolve('./a/b/c/');
```

Register configfile with the given `name` and `options`.

**Params**

* `name` **{String}**
* `configfile` **{String}**
* `options` **{Object}**
* `returns` **{object}**

### [.register](index.js#L130)

Register generator `name` with the given `fn`.

**Params**

* `name` **{String}**
* `fn` **{Function}**: Generator function or instance
* `returns` **{Object}**: Returns the generator instance.

**Example**

```js
base.register('foo', function(app, base) {
  // "app" is a private instance created for the generator
  // "base" is a shared instance
});
```

### [.generator](index.js#L154)

Register generator `name` with the given `fn`, or get generator `name` if only one argument is passed. This method calls the `.getGenerator` method but goes one step further: if `name` is not already registered, it will try to resolve and register the generator before returning it (or `undefined` if unsuccessful).

**Params**

* `name` **{String}**
* `fn` **{Function|Object}**: Generator function or instance. When `fn` is defined, this method is just a proxy for the `.register` method.
* `returns` **{Object}**: Returns the generator instance or undefined if not resolved.

**Example**

```js
base.generator('foo', function(app, base) {
  // "app" is a private instance created for the generator
  // "base" is a shared instance
});
```

### [.hasGenerator](index.js#L186)

Return true if the given `name` exists on the `generators` object. Dot-notation may be used to check for sub-generators.

**Params**

* `name` **{String}**
* `returns` **{Boolean}**: Returns true if the generator exists.

**Example**

```js
base.register('foo', function(app) {
  app.register('bar', function() {});
});

base.hasGenerator('foo');
//=> true
base.hasGenerator('bar');
//=> false
base.hasGenerator('foo.bar');
//=> true
```

### [.getGenerator](index.js#L210)

Get generator `name` from `app.generators`. Dot-notation may be used to get a sub-generator.

**Params**

* `name` **{String}**: Generator name.
* `returns` **{Object|undefined}**: Returns the generator instance or undefined.

**Example**

```js
var foo = app.getGenerator('foo');

// get a sub-generator
var baz = app.getGenerator('foo.bar.baz');
```

### [.globalGenerator](index.js#L243)

Search for globally installed generator `name`. If found, then generator
is registered and returned, otherwise `undefined` is returned.

**Params**

* `name` **{String}**
* `returns` **{Object|undefined}**

### [.findGenerator](index.js#L270)

Find generator `name`, by first searching the cache,
then searching the cache of the `base` generator,
and last searching for a globally installed generator.

**Params**

* `name` **{String}**
* `fn` **{Function}**: Optionally supply a rename function.
* `returns` **{Object|undefined}**: Returns the generator instance if found, or undefined.

### [.invoke](index.js#L301)

Invoke the given generator in the context of (`this`) the current instance.

**Params**

* `app` **{String|Object}**: Generator name or instance.
* `returns` **{Object}**: Returns the instance for chaining.

**Example**

```js
base.invoke('foo');
```

### [.extendWith](index.js#L338)

Alias for `.invoke`, Extend the current generator instance with the settings of other generators.

**Params**

* `app` **{String|Object}**
* `returns` **{Object}**: Returns the instance for chaining.

**Example**

```js
var foo = base.generator('foo');
base.extendWith(foo);
// or
base.extendWith('foo');
// or
base.extendWith(['foo', 'bar', 'baz']);
```

### [.generate](index.js#L376)

Run a `generator` and `tasks`, calling the given `callback` function upon completion.

**Params**

* `name` **{String}**
* `tasks` **{String|Array}**
* `cb` **{Function}**: Callback function that exposes `err` as the only parameter.

**Events**

* `emits`: `generate` with the generator `name` and the array of `tasks` that are queued to run.

**Example**

```js
// run tasks `bar` and `baz` on generator `foo`
base.generate('foo', ['bar', 'baz'], function(err) {
  if (err) throw err;
});

// or use shorthand
base.generate('foo:bar,baz', function(err) {
  if (err) throw err;
});

// run the `default` task on generator `foo`
base.generate('foo', function(err) {
  if (err) throw err;
});

// run the `default` task on the `default` generator, if defined
base.generate(function(err) {
  if (err) throw err;
});
```

### [.generateEach](index.js#L444)

Iterate over an array of generators and tasks, calling [generate](#generate) on each.

**Params**

* `tasks` **{String|Array}**: Array of generators and tasks to run.
* `cb` **{Function}**: Callback function that exposes `err` as the only parameter.

**Example**

```js
// run tasks `a` and `b` on generator `foo`,
// and tasks `c` and `d` on generator `bar`
base.generateEach(['foo:a,b', 'bar:c,d'], function(err) {
  if (err) throw err;
});
```

### [.alias](index.js#L468)

Create a generator alias from the given `name`.

**Params**

* `name` **{String}**
* `options` **{Object}**
* `returns` **{String}**: Returns the alias.

### [.fullname](index.js#L488)

Create a generator's full name from the given `alias`.

**Params**

* `alias` **{String}**
* `options` **{Object}**
* `returns` **{String}**: Returns the full name.

### [.configfile](index.js#L503)

Getter/setter for defining the `configfile` name to use for lookups.
By default `configfile` is set to `generator.js`.

### [.modulename](index.js#L521)

Getter/setter for defining the `modulename` name to use for lookups.
By default `modulename` is set to `generate`.

### [.base](index.js#L548)

Getter/setter for the `base` (or shared) instance of `generate`.

When a generator is registered, the current instance (`this`) is
passed as the "parent" instance to the generator. The `base` getter
ensures that the `base` instance is always the _first instance_.

**Example**

```js
app.register('foo', function(app, base) {
  // "app" is a private instance created for "foo"
  // "base" is a shared instance, also accessible using `app.base`
});
```

Return true if the given `name` exists on the
`app.tasks` object.

**Params**

* `name` **{String}**

## Related projects

* [bach](https://www.npmjs.com/package/bach): Compose your async functions with elegance | [homepage](https://github.com/gulpjs/bach#readme)
* [base](https://www.npmjs.com/package/base): base is the foundation for creating modular, unit testable and highly pluggable node.js applications, starting… [more](https://www.npmjs.com/package/base) | [homepage](https://github.com/node-base/base)
* [base-fs](https://www.npmjs.com/package/base-fs): base-methods plugin that adds vinyl-fs methods to your 'base' application for working with the file… [more](https://www.npmjs.com/package/base-fs) | [homepage](https://github.com/jonschlinkert/base-fs)
* [base-pipeline](https://www.npmjs.com/package/base-pipeline): base-methods plugin that adds pipeline and plugin methods for dynamically composing streaming plugin pipelines. | [homepage](https://github.com/jonschlinkert/base-pipeline)
* [base-plugins](https://www.npmjs.com/package/base-plugins): Upgrade's plugin support in base-methods to allow plugins to be called any time after init. | [homepage](https://github.com/jonschlinkert/base-plugins)
* [base-task](https://www.npmjs.com/package/base-task): base plugin that provides a very thin wrapper around [https://github.com/doowb/composer](https://github.com/doowb/composer) for adding task methods to… [more](https://www.npmjs.com/package/base-task) | [homepage](https://github.com/node-base/base-task)
* [composer](https://www.npmjs.com/package/composer): API-first task runner with three methods: task, run and watch. | [homepage](https://github.com/doowb/composer)
* [gulp](https://www.npmjs.com/package/gulp): The streaming build system | [homepage](http://gulpjs.com)

## Running tests

Install dev dependencies:

```sh
$ npm i -d && npm test
```

## Contributing

Pull requests and stars are always welcome. For bugs and feature requests, [please create an issue](https://github.com/jonschlinkert/base-generators/issues/new).

## Author

**Jon Schlinkert**

* [github/jonschlinkert](https://github.com/jonschlinkert)
* [twitter/jonschlinkert](http://twitter.com/jonschlinkert)

## License

Copyright © 2016 [Jon Schlinkert](https://github.com/jonschlinkert)
Released under the [MIT license](https://github.com/jonschlinkert/base-generators/blob/master/LICENSE).

***

_This file was generated by [verb](https://github.com/verbose/verb), v0.9.0, on February 09, 2016._