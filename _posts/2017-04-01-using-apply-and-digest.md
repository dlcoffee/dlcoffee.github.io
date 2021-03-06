---
layout: post
title:  "Using $apply() and $digest()"
date:   2017-04-01
categories: angularjs javascript
---

Understanding Angular's `$apply` and `$digest` methods require a bit of background on how the browser renders the DOM. The browser has an event loop that will tell it how to display a page.

The browser's event loop initiates when any of the three events occur:
  1. User interaction, e.g. clicking a page
  2. A timer event, e.g. a timeout
  3. A network response

When an event happens, its callback will execute, and we enter the JavaScript context. During this step, the JavaScript code can modify the DOM structure.

Once the callback finishes executing, we leave the JavaScript context, and the browser re-renders the view based on the changes.

### TL;DR
- `scope.$digest()` will fire off all watchers on the current scope and child scopes
- `scope.$apply(fn)` will execute `fn`, and kick off a full `$digest` loop

### Angular's $digest loop

Angular's JavaScript context modifies the original event loop by providing its own event loop, called the `$digest` loop, still within the browser's JavaScript context. This is how you benefit from data-binding, exception handling, etc from Angular.

The `$digest` loop is made up of two smaller loops:
1. one for processing the `evalAsync` queue
2. one for detecting changes in the `$watch` list

The `$evalAsync` queue is a queue used to schedule work that happens before browser rendering. The `$watch` list is a list of expressions that may have changed as a result of the work done during the `$evalAsync` queue. 

As Angular goes through the `$watch` list, it uses each watcher function to decide whether the value has changed or not. This process is called *dirty checking*.

The loop will keep iterating until it does not have anything left in the `$evalAsync` queue and no changes are detected in the `$watch` list.

Once Angular reaches the last watcher, it's not quite done. It will run through the loop again to see if any values have been changed due to later evaluations. It will do this up to 10 times. If it finds that there have been no changes, the `$digest` loop is complete. If it finds that values are still changing, and it has looped 10 times, Angular will throw an error.

### $apply()
`$apply()` is a method on `scope` that takes a `stimulusFn`, which is a function that you want to execute in an angular context. When you call `scope.$apply(stimulusFn)`, you're telling Angular to kick off a `$digest` loop.

The pseudocode for `$apply()` is:

{% highlight javascript %}
function $apply(expr) {
  try {
    return $eval(expr);
  } catch (e) {
    $exceptionHandler(e);
  } finally {
    $root.$digest();
  }
}
{% endhighlight %}

If you're trying to run code outside of an Angular context that is updating something in an Angular context, you might want to use call `$apply()`. For example, if you wanted to use `setTimeout`, you would call `$apply()` so that values are updated in the Angular context.

{% highlight javascript %}
var callback = function() {
  // update a value in Angular context
};

setTimeout(function() {
  scope.apply(callback);
}, 500);
{% endhighlight %}

Generally, you do not need to call `$apply()` manually. The example above could be achieve the same effect by calling Angular's `$timeout` instead of `setTimeout`.


### $digest()
When you call `$digest()`, you're telling Angular to run a `$digest` on its current scope and all of its children. Other scopes are not affected. If you do not need to check every single watcher, `$digest()` might increase your performance over `$apply()`, which calls `$digest()` from the `$rootScope`.

One thing to note is that `$digest()` does provide exception handling in the way `$apply()` does.


##### References {#references}
- [https://code.angularjs.org/1.1.5/docs/guide/concepts#runtime][runtime]

[runtime]: https://code.angularjs.org/1.1.5/docs/guide/concepts#runtime
