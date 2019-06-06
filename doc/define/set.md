@property can-define-object/define/set set
@parent can-define-object/object.behaviors

@description

Specify what happens when a property value is set.

@signature `set( [newVal], [current] )`


> NOTE: Instead of using `set` to set the values of other properties, use the [can-define-object/define/value] behavior.

A set function defines the behavior of what happens when a value is set on an
instance. It is typically used to:

 - Add or update other properties as side effects
 - Coerce the set value into an appropriate action

The behavior of the setter depends on the number of arguments specified. This means that a setter like:

```js
{
	prop: {
		set() {}
	}
}
```

behaves differently than:

```js
{
	prop: {
		set( newVal ) {}
	}
}
```

@param {*} [newVal] The [can-define-object/define/type type function] coerced value the user intends to set on the
instance.

@param {*} current The current value of the property.

@return {*|undefined} If a non-undefined value is returned, that value is set as
the attribute value.


If an `undefined` value is returned, the behavior depends on the number of
arguments the setter declares:

 - If the setter _does not_ specify the `newValue` argument, the property value is set to the type converted value.
 - If the setter specifies the `newValue` argument only, the attribute value will be set to `undefined`.
 - If the setter specifies both `newValue` and `resolve`, the value of the property will not be
   updated until `resolve` is called.


@body

## Use

A property's `set` function can be used to customize the behavior of when an attribute value is set.  Let's see some common cases:

#### Side effects

The following makes setting a `page` property update the `offset`:


```js
import { DefineObject } from "can/everything";

class Pages extends DefineObject {
  static define = {
    limit: 5,
    offset: 0,

    page: {
      set( newVal ) {
        this.offset =  ( parseInt( newVal ) - 1 ) * this.limit;
      }
    }
  };
}

const book = new Pages();
book.page = 10;
console.log( book.offset ); //-> 45
```
@codepen

The following makes changing `makeId` un-define the `modelId` property:

```js
import { DefineObject } from "can/everything";

class Car extends DefineObject {
  static define = {
    modelId: Number,
    makeId: {
      set( newVal ) {
        // Check if we are changing.
        if(newValue !== this.makeId) {
            this.modelId = undefined;
        }
        // Must return value to set as we have a `newValue` argument.
        return newValue;
      }
    }
  };
}

const myCar = new Car({ makeId: "GMC", modelId: "Jimmy" });
console.log( myCar.modelId ); //-> "Jimmy"
myCar.makeId = "Chevrolet";
console.log( myCar.modelId ); //-> undefined
```
@codepen


## Behavior depends on the number of arguments.

When a setter returns `undefined`, its behavior changes depending on the number of arguments.

With 0 arguments, the original set value is set on the attribute.

```js
import { DefineObject } from "can/everything";

class MyMap extends DefineObject {
  static define = {
    prop: {
      set() {

      }
    }
  };
}

const map = new MyMap( { prop: "foo" } );

console.log( map.prop ); //-> "foo"
```
@codepen

With 1 argument, an `undefined` return value will set the property to `undefined`.  

```js
import { DefineObject } from "can/everything";

class MyMap extends DefineObject {
  static define = {
    prop: {
      set( newVal ) {

      }
    }
  };
}

const map = new MyMap( { prop: "foo" } );

console.log( map.prop ); //-> undefined
```
@codepen

## Side effects

A set function provides a useful hook for performing side effect logic as a certain property is being changed.

In the example below, Paginator DefineObject includes a `page` property, which derives its value entirely from other properties (limit and offset).  If something tries to set the `page` directly, the set method will set the value of `offset`:

```js
import { DefineObject } from "can/everything";

class Paginate extends DefineObject {
  static define = {
    limit: Number,
    offset: Number,

    page: {
      set( newVal ) {
        this.offset = ( parseInt( newVal ) - 1 ) * this.limit;
      },
      get() {
        return Math.floor( this.offset / this.limit ) + 1;
      }
    }
  };
}

const p = new Paginate( { limit: 10, offset: 20 } );

console.log( p.offset ); //-> 20
console.log( p.page ); //-> 2
```
@codepen

## Merging

By default, if a value returned from a setter is an object the effect will be to replace the property with the new object completely.

```js
import { DefineObject } from "can/everything";

class Contact extends DefineObject {
  static define = {
    info: {
      set( newVal ) {
        return newVal;
      }
    }
  };
}

const alice = new Contact( {
	info: { name: "Alice Liddell", email: "alice@liddell.com" }
} );

const info = alice.info;

alice.info = { name: "Allison Wonderland", phone: "888-888-8888" };

console.log( info === alice.info ); // -> false
```
@codepen

In contrast, you can merge properties with:

```js
import { DefineObject } from "can/everything";

class Contact extends DefineObject {
  static define = {
    info: {
      set( newVal ) {
        if ( this.info ) {
          return this.info.set( newVal );
        } else {
          return newVal;
        }
      }
    }
  };
}

const alice = new Contact( {
	info: { name: "Alice Liddell", email: "alice@liddell.com" }
} );

const info = alice.info;

alice.info = { name: "Allison Wonderland", phone: "888-888-8888" };

console.log( info === alice.info ); // -> true
```
@codepen

## Batched Changes

By default, calls to `set` methods are wrapped in a call to [can-queues.batch.start] and [can-queues.batch.stop], so if a set method has side effects that set more than one property, all these sets are wrapped in a single batch for better performance.