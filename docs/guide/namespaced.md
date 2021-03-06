---
sidebarDepth: 1
---

# Namespaced Usage

The default export of `vue-types` exposes an ES6 class object that mimics React prop-type.

The class object exposes both [native](/guide/validators.html#native-validators) and [custom](/guide/validators.html#custom-validators) validators.

[[toc]]

## Native Validators

Native validators are exposed as static getters factories:

```js
import VueTypes from 'vue-types'

export default {
  props: {
    message: VueTypes.string.isRequired,
  },
}
```

The main difference between namespaced validators and those directly imported from the library is that the former come (usually) with a sensible default by design.

<div id="default-values">

| Validator | Default    | `validate()` method |
| --------- | ---------- | ------------------- |
| any       | -          | yes                 |
| func      | `() => {}` | yes                 |
| bool      | `true`     | yes                 |
| string    | `''`       | yes                 |
| number    | `0`        | yes                 |
| array     | `[]`       | yes                 |
| integer   | `0`        | -                   |
| symbol    | -          | -                   |
| object    | `{}`       | yes                 |

</div>

Examples:

```js
const numProp = VueTypes.number
// numProp === { type: Number, default : 0 }

const numPropCustom = VueTypes.number.def(10)
// numPropCustom ===  { type: Number, default : 10 }

const stringProp = VueTypes.string
// numPropCustom ===  { type: String, default : '' }
```

## Native Types Configuration

All native type validators (with the exception of `any` and `symbol`) come with a sensible default value. In order to customize or disable that value you can set the global option `VueTypes.sensibleDefaults`:

```js
//use VueTypes defaults (this is the "default" behavior)
VueTypes.sensibleDefaults = true

//disable all sensible defaults.
//Use .def(...) to set one on each prop
VueTypes.sensibleDefaults = false

//assign an object in order to specify custom defaults
VueTypes.sensibleDefaults = {
  string: 'mystringdefault',
  //...
}
```

Under the hood `VueTypes.sensibleDefaults` is a plain object implemented with custom getters/setters. That let's you play with it like you'd do with every other object.

For example you can remove some of the default values using [object rest spread](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Operators/Spread_syntax#Spread_in_object_literals) or [lodash.omit](https://lodash.com/docs/4.17.11#omit).

```js
console.log(VueTypes.bool.default)
// logs true

// copy every default value but boolean
const { bool, ...newDefaults } = VueTypes.sensibleDefaults

// or, with lodash
// const newDefaults = _.omit(VueTypes.sensibleDefaults, ['bool'])

VueTypes.sensibleDefaults = newDefaults

console.log(VueTypes.bool.default)
// logs undefined
```

::: tip
To unset the default value for an individual validator instance use `.def(undefined)`

```js
const type = VueTypes.string.def(undefined)
// { type: String }

const type2 = VueTypes.string
// still { type: String, default: '' }
```

:::

## Custom Validators

Custom validators are exposed as static methods. Refer to the [dedicated documentation](/guide/validators.html#custom-validators) for usage instructions.

```js
const arrayOfStrings = VueTypes.arrayOf(String)
```

## Utilities

The class object exposes some utility functions under the `.utils` property:

### `utils.validate(value, type)`

Checks a value against a type definition:

```js
VueTypes.utils.validate('John', VueTypes.string) //true

VueTypes.utils.validate('John', { type: String }) //true
```

::: warning

This utility won't check for `isRequired` flag, but will execute any provided custom validator function:

```js
const isJohn = {
  type: String,
  validator(value) {
    return value.length === 'John'
  },
}

VueTypes.utils.validate('John', isJohn) //true
VueTypes.utils.validate('Jane', isJohn) //false
```

:::

### `utils.toType(name, obj, validable = false)`

Will convert a plain object to a VueTypes validator object with `.def()` and `isRequired` modifiers:

```js
const password = {
  type: String,
  validator(value) {
    //very raw!
    return value.length > 10
  },
}

const passwordType = VueTypes.utils.toType('password', password)

export default {
  props: {
    password: passwordType.isRequired,
  },
}
```

If the last argument is `true` the resulting validator object will support the `.validate()` method as well:

```js
const password = {
  type: String,
  required: true
}

const passwordType = VueTypes.utils.toType('password', password)

export default {
  props: {
    // this password prop must include at least a digit
    password: passwordType.validate((v) => /[0-9]/test.(v)),
  },
}
```
