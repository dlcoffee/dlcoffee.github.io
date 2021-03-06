---
layout: post
title:  "call and apply"
date:   2017-06-23
categories: javascript
---

In JavaScript, functions are just objects. This means that you can attach functions to them,
just as you can with any other object. Some defaults that come along with being a function are
`.toString()`, `bind()`, `call()`, and `apply()`. We're going to tkae a closer look at two similar, but slightly different methods: `call()` and `apply()`.

### TL;DR
- both methods invoke the function caller, using a context passed in as the first parameter
- `call` takes a `this` context, and the arguments are passed with **commas**
- `apply` takes a `this` context, and the arguments are passed as an **Array**

### Why use either?

Using `call` or `apply` are ways to re-use some functionality.
They provide us a way to *borrow* functions, so we can avoid code duplication.
With either method, you can execute the function with a modified `this` context.
When a function is called normally, it's `this` value is set normally (either the containing
object or the global object).

{% highlight javascript %}
var obj = { value: 10 };

function addToThis(x) {
  return this.value + x; // `this` refers to the global object
};

console.log(addToThis(5)); // undefined, because `this.value` is undefined

// `call` or `apply` modifies the `this` context
console.log(addToThis.call(obj, 5)); // 15
console.log(addToThis.apply(obj, [5])); // 15
{% endhighlight %}

### Choosing one over the other

`call` is a good choice when the arguments for a function are known.

{% highlight javascript %}
var robert = {
  name: 'robert',
  greeting: function () {
    return 'my name is ' + this.name;
  }
};

var robot = {
  name: 'ROB',
}

console.log(robert.greeting()); // 'my name is robert'
console.log(robert.greeting.call(robot)) // 'my name is ROB'
{% endhighlight %}

`apply` is a good choice when the function has an unknown amount of arguments.
You can deconstruct the arguments yourself, and do whatever is necessary.

{% highlight javascript %}
var robert = {
  name: 'robert',
  greeting: function () {
    var foodsArray = [];
    var foodsStr;

    for (var i = 0; i < arguments.length; i++) {
      foodsArray.push(arguments[i]);
    }

    // insert oxford comma if necessary
    var length = foodsArray.length;
    if (length === 1) {
      foodsStr = foodsArray[0];
    } else if (length == 2) {
      foodsStr = foodsArray.join(' and ');
    } else if (length > 2) {
      foodsArray[length - 1] = 'and ' + foodsArray[length - 1];
      foodsStr = foodsArray.join(', ');
    }

    return 'my name is ' + this.name + ', and i like to eat ' + foodsStr;
  }
};

var robot = {
  name: 'ROB',
}

console.log(robert.greeting('pizza', 'hot dogs'));
// 'my name is robert, and i like to eat pizza and hot dogs'

console.log(robert.greeting.apply(robot, ['waffles', 'metal', 'humans']));
// 'my name is ROB, and i like to eat waffles, metal, and humans'
{% endhighlight %}

Note that I'm using a special JavaScript object here: `arguments`.
`arguments` gives us access to all parameters passed in, specified or not.
The `robert.greeting` method does not take any parameters, but we're still able to call it with a variable amount of parameters.
It's important to also note that that although `arguments` has a `length` method, it is **not** an Array.

Using this knowledge, we can be clever about our usage of `apply` and use it to our advantage on some built-in JavaScript functions.
Take `Math.max()`, for example. `Math.max()` takes arguments as comma separated values, and returns the max value.
If we gave it an array, we'd get `NaN`, because an Array can't be converted into a number for the function.
Instead, we can use `apply` in this form:

{% highlight javascript %}
var numbers = [5, 3, 12, 1];

console.log(Math.max(numbers)); // NaN

// this translates to:
// Math.max(numbers[0], numbers[1], ...);
console.log(Math.max.apply(null, numbers)); // 12

{% endhighlight %}

Here, we don't need to invoke `Math.max` with any specific `this`.
We just want to call the method using an array.
With `apply()`, we can do just that, and avoid writing our own version of `max()` for arrays.

##### References {#references}
- [http://adripofjavascript.com/blog/drips/invoking-javascript-functions-with-call-and-apply.html][drip]
- [https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Function/call][mdn-call]
- [https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Function/apply][mdn-apply]

[drip]: http://adripofjavascript.com/blog/drips/invoking-javascript-functions-with-call-and-apply.html
[mdn-call]: https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Function/call
[mdn-apply]: https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Function/apply
