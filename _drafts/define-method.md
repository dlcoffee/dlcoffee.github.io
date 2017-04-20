---
layout: post
title:  "define_method"
categories: ruby
---

The usual way of defining methods is by using a block, via `def` and `end`. Sometimes, you define a bunch of methods that are very similar. Something about metaprogramming. As a dynamic language, Ruby parses and compiles your code at runtime.

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
baby.pooping?
#=> false
{% endhighlight %}

The defined methods are all very similar, and it'd be great if we could DRY it up somehow. Luckily, we can! As a dynamic language, Ruby allows us to use **metaprogramming** (writing code that writes code).


### dynamically defining methods
We can use `define_method` to dynamically define methods in Ruby. We use `define_method` by passing it the name of the method we want to define, and a block that will be the body for that method. The code above can be refactored into:

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

Isn't that much nicer?

### downside
A downside to using `define_method` is that it creates a closure. Objects in the closure are not garbage collected, so be careful when using it.

##### References {#references}
- [http://rohitrox.github.io/2013/07/02/ruby-dynamic-methods/][a]
- [https://tenderlovemaking.com/2013/03/03/dynamic_method_definitions.html][tenderlove]
- [http://rubylearning.com/blog/2010/11/23/dont-know-metaprogramming-in-ruby/][learning]

[a]: http://rohitrox.github.io/2013/07/02/ruby-dynamic-methods/
[tenderlove]: https://tenderlovemaking.com/2013/03/03/dynamic_method_definitions.html
[learning]: http://rubylearning.com/blog/2010/11/23/dont-know-metaprogramming-in-ruby/
