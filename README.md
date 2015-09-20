# Stamp Specification: Composables

## Introduction

The composables specification exists in order to define a standard format for composable factory functions (called **stamps**), and ensure compatibility between different stamp implementations.

## Status

This is a draft proposal. The specification may have breaking changes. It should not be considered ready for production use.

### Composable

**Composable** (aka **stamp**) is a composable factory function that returns object instances based on its **descriptor**.

```js
assert(typeof composable === 'function');

const newObject = composable();
```

It has method called `.compose()`:

```js
assert(typeof composable.compose === 'function');
```

When called the `.compose()` method creates new composable using the current composable as a base, composed with a list of *composables* passed as arguments:

```js
const combinedComposable = baseComposable.compose(composable1, composable2, composable3);
```

### Descriptor

**Composable descriptor** (or just **Descriptor**) is a meta data object with properties that contain the information necessary to create an object instance. Descriptor properties are attached to the `.compose()` method.



### Standalone `compose()` function (optional)

* `compose(...stampsOrDescriptors) => stamp` **Creates stamps.** Take any number of stamps or descriptors.
Return a new stamp that encapsulates combined behavior. If nothing is passed in, it returns an empty stamp.


## Implementation details

### Stamp

* `stamp(options) => instance` **Creates or mutates object instances.** Take an options object which may contain an `.instance` property. Return the mutated instance. If no instance is passed, it uses a new empty object as the instance. If present, the existing prototype of the instance must not be mutated. Instead, the behavior (methods) must be added to a new delegate prototype.
 * `.compose(...stampsOrDescriptors) => stamp` **Creates stamps.** A method exposed by all composables, identical to `compose()`, except it prepends `this` to the stamp parameters. Stamp descriptor properties are attached to the `.compose` method., e.g. `stamp.compose.properties`.


### The Stamp Descriptor

The names and definitions of the fixed properties that form the stamp descriptor.
The stamp descriptor properties are made available on each stamp as `stamp.compose.*`

* `methods` - A set of methods that will be added to the object's delegate prototype.
* `properties` - A set of properties that will be added to new object instances by assignment.
* `deepProperties` - A set of properties that will be added to new object instances by assignment with deep property merge.
* `initializers` - A set of functions that will run in sequence. Stamp details and arguments get passed to initializers.
* `staticProperties` - A set of static properties that will be copied by assignment to the stamp.
* `propertyDescriptors` - A set of [object property
descriptors](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Object/defineProperties) used for fine-grained control over object property behaviors.
* `staticPropertyDescriptors` - A set of [object property descriptors](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Object/defineProperties) to apply to the stamp.
* `configuration` - A set of options made available to the stamp and its initializers during object instance creation. Configuration properties get deep merged.

#### Composing descriptors

Descriptors are composed together to create new descriptors with the following rules:

* `methods` are copied by assignment: `descriptor.methods = _.assign({}, descriptor1.methods, descriptor2.methods)`
* `properties` are copied by assignment: `descriptor.properties = _.assign({}, descriptor1.properties, descriptor2.properties)`
* `deepProperties` are deep merged: `descriptor.deepProperties = _.merge({}, descriptor1.deepProperties, descriptor2.deepProperties)`
* `initializers` are appended: `descriptor.initializers = descriptor1.initializers.concat(descriptor2.initializers)`
* `staticProperties` are copied by assignment: `descriptor.staticProperties = _.assign({}, descriptor1.staticProperties, descriptor2.staticProperties)`
* `propertyDescriptors` are copied by assignment: `descriptor.propertyDescriptors = _.assign({}, descriptor1.propertyDescriptors, descriptor2.propertyDescriptors)`
* `staticPropertyDescriptors` are copied by assignment: `descriptor.propertyDescriptors = _.assign({}, descriptor1.propertyDescriptors, descriptor2.propertyDescriptors)`
* `configuration` are deep merged: `descriptor.configuration = _.merge({}, descriptor1.configuration, descriptor2.configuration)`


### Stamp `options` reserved keys

The following are reserved keys for the stamp options object:

```js
{
  instance // The object to be mutated by the stamp
}
```

### Initializer parameters

Initializers have the following signature:

```js
(options, instance, stamp) => Instance
```

* `options` The `options` argument passed into the stamp, containing propreties that may be used by initializers.
* `instance` The object instance being produced by the stamp. If the initializer returns a new object, it replaces the instance.
* `stamp` A reference to the stamp producing the instance.


-----

## Similarities with Promises (aka Thenables)

* *Thenable* ~ *Composable*.
* `.then` ~ `.compose`.
* *Promise* ~ *Stamp*.
* `new Promise(resolve, reject)` ~ `compose(...stampsOrDescriptors)`

-----
