---
layout: post
title:  "define_method"
date:   2017-04-20
categories: ruby metaprogramming
---

There are multiple ways of defining methods in Ruby. The most popular is by using a block, via `def` and `end`. One cool thing about Ruby is its **metaprogramming** (being able to write code that writes code) capabilities. As an example, take a look at this `Baby` class definition.

### A Baby example
{% highlight ruby linenos %}
class Baby

  CRYING = 0
  POOPING = 1
  SLEEPING = 2

  attr_writer :status

  def initialize
    @status = CRYING
  end

  def sleeping?
    status == SLEEPING
  end

  def pooping?
    status == POOPING
  end

  def crying?
    status == CRYING
  end

end

baby = Baby.new
baby.crying?
#=> true

baby.sleeping?
#=> false

baby.status = 2
baby.crying?
#=> false

baby.sleeping?
#=> true
{% endhighlight %}

The defined methods are all very similar, and it'd be great if we could DRY it up somehow. Luckily, we can! We can use **metaprogramming** to write some code that will generate our `sleeping?`, `pooping?`, and `crying?` methods.

### dynamically defining methods
We can use `define_method` to dynamically define methods in Ruby. `define_method` is used by passing a name of the method we want to define, and a block that will be the body for that method. Our `Baby` class can be refactored into:

{% highlight ruby linenos %}
class Baby

  CRYING = 0
  POOPING = 1
  SLEEPING = 2

  attr_writer :status

  def initialize
    @status = CRYING
  end

  [:crying, :pooping, :sleeping].each do |status|
    define_method "#{status}?" do
      status == Babe.const_get(status.upcase)
    end
  end

end

baby = Baby.new
baby.crying?
#=> true

baby.sleeping?
#=> false
{% endhighlight %}

Isn't that much nicer? We got rid of redundancy and made it a lot easier to write more status methods, such as `laughing?` or `hiccuping?`.

### being mindful
A downside to using `define_method` is that it creates a closure. Objects in the closure are not garbage collected, so be mindful when using dynamically creating methods using `define_method` and creating objects. For more on benchmarking, take a look at tenderlove's blog post [here][two].

##### References {#references}
- [http://rohitrox.github.io/2013/07/02/ruby-dynamic-methods/][one]
- [https://tenderlovemaking.com/2013/03/03/dynamic_method_definitions.html][two]
- [http://rubylearning.com/blog/2010/11/23/dont-know-metaprogramming-in-ruby/][three]

[one]: http://rohitrox.github.io/2013/07/02/ruby-dynamic-methods/
[two]: https://tenderlovemaking.com/2013/03/03/dynamic_method_definitions.html
[three]: http://rubylearning.com/blog/2010/11/23/dont-know-metaprogramming-in-ruby/
