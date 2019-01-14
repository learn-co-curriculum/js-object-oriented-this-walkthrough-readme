# JavaScript This Walkthrough

## Objectives

- Understand how `this` depends on the invocation of a method

## Deep Dive into `this`

JavaScript's `this` keyword can sometimes be unpredictable. Now that we are
going deeper into object oriented code, we should discuss it with the detail
that it deserves. In brief, the value of `this` changes based on where it is.

- Outside of any function, `this` refers to the [global object][global]. In
  web browsers, this is the [window][window]

- Inside an object method, `this` refers to the object that received the method
  call

- Inside a standalone function, `this` will either default to the
  [global object][global] _or_ be `undefined`

Let's make sure we properly define the word _method_. A JavaScript method is a
property on an object that points to a function. So just like we can write `let person = {name: 'bob'}`, whereby the `name` property points to a value of
`'bob'`, we can also write the following:

```javascript
let person = {
	greet: function() {
		console.log('hello');
	}
};

typeof person.greet;
// 'function'
```

And have the `greet` property point to a function. Because this `greet` property
points to a function, we call the property a method. Ok, so because the `greet`
property is a method, we know that when we call `person.greet()` the object
receiving the method call equals `person`. Let's modify our `greet` function and
see that:

```javascript
let person = {
	greet: function() {
		return this;
	}
};

typeof person.greet;
// 'function'
person.greet() == person;
// true
```

We see that `person.greet()` returns `this`, which refers to the object that
received the method call.

#### Watching `this` Change

Ok, hang on tight, because we are about to see something weird. Let's modify the
code above slightly:

```javascript
let person = {
	greet: function() {
		return this;
	}
};

person.greet() == person;
// true

let greetFn = person.greet;

greetFn() == person;
// false
greetFn();
// window

person.greet() == person;
// true
```

Ok, so let's walk through the code above. We set our `greet` property on the
`person` object to point to the same function, and see that when we call
`person.greet()`, `this` refers to the `person` object.

The tricky part happens when we write `let greetFn = person.greet`. What this
does is assign a local variable `greetFn` to refer to the function on the
`person` object. However, this local variable **does not** refer to the `person`
object at all. It has zero knowledge of the `person` object. It _only_ points to
the function. Then we invoke the function by calling `greetFn()`. In doing so,
we are not calling the property on the object, we are simply invoking the
function.

When the function is invoked, `this` refers to `window`, our global object.

#### Functions Within Functions

Another occurrence where `this` becomes the global object is when we invoke
functions within functions. Let's change our code to look like the following:

```javascript
let person = {
	greet: function() {
		function otherFunction() {
			return this;
		}
		return otherFunction();
	}
};
```

Ok, let's walk through what the above code is doing. Our object, `person` has a
property `greet` that contains to a function. Now when this method, `greet`, is
called, it first declares and then invokes the `otherFunction` function. The
`otherFunction` returns `this`, which is then returned by the `greet` method. So
what does `this` equal? Well, note that `this` **is not** referenced directly
inside a method. `otherFunction` is not a _method_, just a function.

```javascript
let person = {
	greet: function() {
		// this == person
		function otherFunction() {
			// this == window
			return this;
		}

		// otherFunction is invoked, returning the functions value of `this`, which is window
		return otherFunction();
	}
};

person.greet();
// window
```

Remember, a method is a property that points to a function. If we call the
`person.greet` property, it invokes an anonymous function. That function _is_ in
the context of `person`. The function it _contains_, `otherFunction`, is
**not**.

With the exception of arrow functions, every time a function is invoked, `this`
will be defined based on the context that it is in. Inside `otherFunction`,
`this` defaults to the global object.

#### Callbacks

The previous example may seem unnecessarily complicated, but we actually include
functions in other functions whenever we use callbacks. In fact, many out of the
box JavaScript functions are passed callbacks. So if we did something like the
following:

```js
[1, 2, 3].filter(function(element) {
	console.log(this);
	return element % 2 == 0;
});
// window
// window
// window
```

Each time our callback function is invoked, `this` is global. Do you see why?
Our function is invoked by the `filter` method. Filter is a _method_. We can
invoke it as a property on our array. However, the callback function is not
called as a property on an object, and thus when we reference `this` from inside
the callback function, `this` will refer the window, or global object.

## Exceptions

#### Arrow Functions

Arrow functions do not have their own `this`. Instead, an arrow function uses
whatever `this` is defined within the scope it is in. So, for instance, if we
_rewrote_ our last `person` object:

```javascript
let person = {
	greet: function() {
		// this == person
		const otherFunction = () => {
			// this == person still, due to the arrow function
			return this;
		};

		// otherFunction is invoked, returning the functions value of `this`, which is person
		return otherFunction();
	}
};

person.greet();
// { greet: [Function: greet] }
```

The arrow function assigned to `otherFunction` (inside of `greet`) uses `this`
from the `person` object. `otherFunction` does not have its own `this`, so it
defaults to using `this` from the scope it is in. The result is that `this`
will now refer to `person`.

#### Classes

Methods defined within [JavaScript classes][classes] behave similarly. If we
were to rewrite our `person` object into a class:

```js
class Person {
	constructor(name) {
		this.name;
	}

	greet() {
		function innerFunction() {
			return this;
		}
		return innerFunction();
	}
}

let sally = new Person('Sally');
sally.greet();
// undefined
```

Inside `innerFunction`, `this` has defaulted back to undefined, as it does with
normal functions. Once again, though, with an _arrow_ function, `this` will not
be redefined, so we could rewrite the code as follows:

```js
class Person {
	constructor(name) {
		this.name;
	}

	greet() {
		const innerFunction = () => {
			return this;
		};
		return innerFunction();
	}
}

let sally = new Person('Sally');
sally.greet();
// Person {}
```

Now, the `this` inside of `innerFunction` will be based on the context the arrow
function is in: the `greet()` method. Inside the `greet()` method context,
`this` will refer to the object it is in: `Person`.

## Summary

The above lesson displayed how `this` changes depending on how a function is
called and where that function is. We can make an addendum to our original
rules about `this`:

- Outside of any function, `this` refers to the [global object][global]. In
  web browsers, this is the [window][window]

- Inside an object method, `this` refers to the object that received the method
  call

- Inside a standalone function, **even one inside a method**, `this` will
  default to the [global object][global]

- When using [strict mode][strict] in a standalone function, as we do inside
  classes, `this` will be undefined

- Arrow functions don't define their own `this` like standard functions do.

We saw that even if a function was originally declared as a property on an
object, if we do not reference the function as a method, `this` will default to
the global object.

We also saw how callback methods are a specific application of an inner function
being called, and therefore `this` again defaults to the global object.

`this` has some specific behaviors that can sometimes lead to unexpected
results. In later lessons, we will look at some built in JavaScript methods that
actually allow us to control what `this` refers to, ensuring that when we fire a
function, it will always be in the right context.

[global]: https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Operators/this#Global_Context
[window]: https://developer.mozilla.org/en-US/docs/Web/API/Window
[strict]: https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Strict_mode
[classes]: https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Classes#Strict_mode
