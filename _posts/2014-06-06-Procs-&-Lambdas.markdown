---
layout: post
title:  "Procs & Lambdas"
date:   2014-06-06 20:21:00
---

I wrote my first ‘Hello world!’ in Ruby about three years ago and it was love at first site. Since then I have tried to learn all its in’s and out’s. It has been a while since I first heard about procs and lambdas, but I have always avoided using them until recently.

A few months ago I decided to dive a bit deeper into JavaScript, which seemed like another useful language to know. The first thing that I really liked about JavaScript was the ability to pass a function as an argument to another function. At that moment I thought how nice it would be to be able to pass Ruby methods as arguments to other methods. That is when my light bulb went on, Ruby methods do accept blocks –- like JavaScript anonymous functions -- and they also accept variables that store snippets of functionality, yes... procs and lambdas. Now if you are like me and you are trying to stay away from black magic like these two little fellas right here, then please keep reading. Hopefully I can help you overcome that fear and see the beauty of using them.


##What are *procs* and *lambdas*


There are more similarities than differences between the two, maybe because both *procs* and *lambdas* are instances of the class Proc. Here are a few ways you can instantiate them, I encourage you to try the following in **irb**:


Procs:
{% highlight ruby %}
>> proc { puts "I am a proc"}
=> #<Proc:0x007fe5d4096e58@(irb):16>
>> Proc.new { puts "I am a proc as well" }
=> #<Proc:0x007fe5d408d100@(irb):17>
{% endhighlight %}


Lambdas:
{% highlight ruby %}
>> lambda { puts "I am a lambda"}
=> #<Proc:0x007fe5d4088d58@(irb):18 (lambda)>
>> -> { puts "I am a lambda as well"}
=> #<Proc:0x007fe5d4082b38@(irb):19 (lambda)>
  {% endhighlight %}


Both `#proc` and `#lambda` methods are syntactic sugar for `Proc.new` with the appropriate parameters. A proc object stores a code block as it’s body and can execute it with the help of the `#call` instance method like this:

{% highlight ruby %}
>> proc { puts "I am a proc"}.call
I am a proc
=> nil     # method #puts returns nil
{% endhighlight %}


Even though blocks can be used without being stored as procs, most of the methods will still use them as procs and we can see that in the next example:







{% highlight ruby %}
>>def method_with_block(&block)
>>    block.call
>>end

>>method_with_block { puts "I think I am a block"}
I think I am a block
=> nil

{% endhighlight %}


So the block was called inside the method… or was it? Let’s try calling the block outside the method.


{% highlight ruby %}
>> { puts "I think I am a block"}.call
SyntaxError: (irb):37: syntax error, unexpected tSTRING_BEG,... long error
{% endhighlight %}


So what exactly happened in our previous method? Well, the **&** symbol in front of `&block` is again syntactic sugar built into Ruby, that creates an instance of the `class Proc` if the object is not one already. So a `proc` is an object, therefore it can be assigned to a variable or passed as an argument just like any other object.



##How can we use them?


Let’s say that we have a few methods that have different outputs, but except a few lines of code, they look very similar.  Sounds like repetition, doesn’t it? Even though this is not the most practical example, let’s say that we have a container of different objects that all have a weight attribute in grams.

{% highlight ruby %}
class TV
  def weight
    15000
  end
end

class Bed
  def weight
    50000
  end
end

class Mattress
  def weight
    25000
  end
end
{% endhighlight %}



Now let’s say that we have a container with several objects of different weights and that we want a method that would get all the weights of the objects in grams:


{% highlight ruby %}
#container
container = [ TV.new, Bed.new, Mattress.new]

#method
def get_weights_from(container)
  container.map(&:weight)
end

>>get_weight_from(container)
=> [15000, 50000, 25000]
{% endhighlight %}


What if we would want it in Kilograms or pounds, should we have a different method for each one of them? Or maybe… we could have our method accept a block that would be passed to the map method. It could look something like this:


{% highlight ruby %}
def get_weights_from(container, &block)
  container.map(&block)
end

in_kg = proc { |x| (x.weight / 1000.0).round(2) }
in_pounds = proc { |x| (x.weight / 453.5).round(2) }
in_ounces = proc { |x| (x.weight / 28.3).round(2) }

>> get_weights_from(container, &in_pounds)
=> [33.08, 110.25, 55.13]
>> get_weights_from(container, &in_kg)
=> [15.0, 50.0, 25.0]
>> get_weights_from(container, &in_ounces)
=> [530.04, 1766.78, 883.39]
>> get_weights_from(container, &:weight)
=> [15000, 50000, 25000]  # just calling the weight method on the objects
{% endhighlight %}



This may not be the best example on how to use the procs but hopefully you get a feeling on how they work. I think that it reads really nicely. Here we noticed that a proc can accept a variable, but it is important to know that they can accept several variables. Here is another way to call a proc with several variables.

{% highlight ruby %}
pr = proc { |x, y, z|  puts "x = #{x}; y = #{y}; z = #{z}" }
pr.call(1, 2, 3)
x = 1; y = 2; z = 3
=> nil
{% endhighlight %}


##Variables, procs & scope


Unlike the methods in Ruby, variable scope inside a proc is dependent on the environment the proc is created in.

{% highlight ruby %}
>>a = 3
>>pr = proc { puts "a = #{a}"}
>> pr.call
a = 3
=> nil
>>a  = 5
>> pr.call
a = 5
=>nil
{% endhighlight %}


As we saw here the proc remembers the context of its scope. But also if the proc is taken out of it’s context it will remember the state of the context it was created in. As David A. Black says “Creating a closure is like packing a suitcase: wherever you open the suitcase, it contains what you have put in when you packed it” (414).

{% highlight ruby %}
def proc_creator(x)
  proc { puts "x = '#{x}'"}
end

p3 = proc_creator(3)
p4 = proc_creator(4)

>> p4.call
x = '4'
>> p3.call
x = '3'
{% endhighlight %}


##Differences between *proc* and *lambda*


Besides the difference in initialization I could find two more differences between the two.

1. Unlike procs,  if a lambda expects an argument, it will throw an error when it is called without one:

{% highlight ruby %}
pr = proc { |x| puts "x = '#{x}'"}
lb  = lambda  { |x|  puts "x = '#{x}'"}
>> pr.call
x = ''
=> nil
>> lb.call
ArgumentError: wrong number of arguments (0 for 1)
{% endhighlight %}


2. When a lambda returns, it passes the control back to the method it is part of. When a proc returns it returns all the way out of the method it is part of.


{% highlight ruby %}
def return_demo
  puts "control in the begining of the method"
  lb = lambda{return puts "returning inside the lambda"}
  lb.call
  puts "control back inside the method"
  pr = proc {return puts "returning inside the proc"}
  pr.call
  puts "this will not be printed out"
end

>> return_demo
control in the begining of the method
returning inside the lambda
control back inside the method
returning inside the proc
=> nil
{% endhighlight %}


As you can see the last statement was never printed out. These are the two major differences that I know off between the two.



To better understand how to use them I really encourage you to try out all these samples in your **irb**, and why not try and come up with new examples. Hopefully this was useful to you and that from now on when you see several methods with very slight differences you will at least give a thought to what you have read today.


##Bibliography:

Black, David A. *The Well-grounded Rubyist*. Greenwich, CT.: Manning, 2009. Print.
