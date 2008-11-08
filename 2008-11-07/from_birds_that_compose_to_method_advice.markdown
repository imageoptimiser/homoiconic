Aspect-Oriented Programming in Ruby Using Combinator Birds
---

In [Combinatory Logic](http://en.wikipedia.org/wiki/Combinatory_logic), the bluebird is one of the most important and fundamental combinators, because the bluebird *composes* two other combinators. Although this is usually discussed as part of [functional programming style](http://weblog.raganwald.com/2007/03/why-why-functional-programming-matters.html "Why Why Functional Programming Matters Matters"), it is just as valuable when writing object-oriented programs. In this post, we will develop an aspect-oriented programming module that adds before methods and after methods to Ruby programs, with the implementation inspired by the bluebird. 

> As explained in [Kestrels](http://github.com/raganwald/homoiconic/tree/master/2008-10-29/kestrel.markdown), the practice of nicknaming combinators after birds was established in Raymond Smullyan's amazing book [To Mock a Mockingbird](http://www.amazon.com/gp/product/0192801422?ie=UTF8&tag=raganwald001-20&linkCode=as2&camp=1789&creative=9325&creativeASIN=0192801422). In this book, Smullyan explains combinatory logic and derives a number of important results by presenting the various combinators as songbirds in a forest. Since the publication of the book more than twenty years ago, the names he gave the birds have become standard nicknames for the various combinators.


[![Eastern bluebird (c) 2008 Doug Greenberg, some rights reserved reserved](http://farm3.static.flickr.com/2376/2451392973_cd28956b14_o.jpg)](http://www.flickr.com/photos/dagberg/2451392973/ "Eastern bluebird (c) 2008 Doug Greenberg, some rights reserved")  

The bluebird is written `Bxyz = x(yz)`. In Ruby, we can express the bluebird like this:

	bluebird.call(proc1).call(proc2).call(value)
		=> proc1.call(proc2.call(value))

If this seems a little arcane, consider a simple Ruby expression `(x * 2) + 1`: This expression *composes* multiplication and addition. Composition is so pervasive in programming languages that it becomes part of the syntax, something we take for granted. We don't have to think about it until someone like Oliver Steele writes a library like [functional javascript](http://osteele.com/sources/javascript/functional/) that introduces a `compose` function, then we have to ask what it does.

Before we start using bluebirds, let's be clear about something. We wrote that `bluebird.call(proc1).call(proc2).call(value)` is equivalent to `proc1.call(proc2.call(value))`. We want to be very careful that we understand what is special about `proc1.call(proc2.call(value))`. How is it different from `proc1.call(proc2).call(value)`?

The answer is:

	proc1.call(proc2.call(value))
		=> puts value into proc2, then puts the result of that into proc1
	
	proc1.call(proc2).call(value)
		=> puts proc2 into proc1, getting a function out, then puts value into the new function
	
So with a bluebird you can chain functions together in series, while if you didn't have a bluebird all you could do is write functions that transform other functions. Not that there's anything wrong with that, we used that to great effect with [cardinals](http://github.com/raganwald/homoiconic/tree/master/2008-10-31/songs_of_the_cardinal.markdown) and [quirky birds](http://github.com/raganwald/homoiconic/tree/master/2008-11-04/quirky_birds_and_meta_syntactic_programming.markdown).

**giving methods advice**

We're not actually going to [Greenspun](http://en.wikipedia.org/wiki/Greenspun%27s_Tenth_Rule "Greenspun's Tenth Rule - Wikipedia, the free encyclopedia") an entire aspect-oriented layer on top of Ruby, but we will add a simple feature, we are going to add *before and after methods*. You already know what a normal method is. A before method simply specifies some behaviour you want executed before the method is called, while an after method specifies some behaviour you wnat executed after the method is called. In [Aspect-Oriented Programming](http://en.wikipedia.org/wiki/Aspect-oriented_programming ""), before and after methods are called "advice."

Ruby on Rails programmers are familiar with method advice. if you have ever written any of the following, you were using Rails' built-in aspect-oriented programming support:

	after_save
	validates_each
	alias_method_chain
	before_filter

These and other features of Rails implement method advice, albeit in a very specific way tuned to portions of the Rails framework. We're going to implement method advice in a module that you can use in any of your classes, on any method or methods you choose. We'll start with before methods. Here's the syntax we want:

	def something(parameter)
		# do stuff...
	end
	
	before :something do |parameter|
		# stuff to do BEFORE we do stuff...
	end
	
	before :something do |parameter|
		# stuff to do BEFORE stuff to do BEFORE we do stuff...
	end

As we can see, the before methods get chained together before the method. To keep this nice and clean, we are going to make them work just like composable functions: whatever our before method's block returns will be passed as a parameter up the chain. We also won't fool around with altering the order of before methods, we'll just take them as they come.

This is really simple, we are composing methods. To compare to the bluebird above, we are writing `before`, then the name of a method, then a function. I'll rewrite it like this:

	bluebird.call(something).call(stuff_to_do_before_we_do_stuff).call(value)
		=> something.call(stuff_to_do_before_we_do_stuff.call(value))

Now we can see that this newfangled aspect-oriented programming stuff was figured out nearly a century ago by people like Alonzo Church.

Okay, enough history, let's get started. First, we are not going to write any C, so there is no way to actually force the Ruby VM to call our before methods. So instead, we are going to have to rewrite our method. We'll use [a trick I found on Jay Fields' blog](http://blog.jayfields.com/2006/12/ruby-alias-method-alternative.html "Jay Fields' Thoughts: Ruby: Alias method alternative"):

	module NaiveBeforeMethods
  
	  module ClassMethods
    
	    def before(method_sym, &block)
	      old_method = self.instance_method(method_sym)
	      if old_method.arity == 0
	        define_method(method_sym) do
	          block.call
	          old_method.bind(self).call
	        end
	      else
	        define_method(method_sym) do |*params|
	          old_method.bind(self).call(*block.call(*params))
	        end
	      end
	    end
    
	  end
  
	  def self.included(receiver)
	    receiver.extend         ClassMethods
	  end
  
	end

	class SuperFoo

	  def one_parameter(x)
	    x + 1
	  end

	  def two_parameters(x, y)
	    x * y
	  end
  
	end

	class Foo < SuperFoo

	  include NaiveBeforeMethods

	  before :one_parameter do |x|
	    x * 2
	  end

	  before :two_parameters do |x, y|
	    [x + y, x - y]
	  end

	end

	Foo.new.one_parameter(5)
		=> 11
	
	Foo.new.two_parameters(3,1)
		=> 8

As you can see, we have a special case for methods with no parameters, and when we have a method with multiple parameters, our before method must answer an array of parameters. And the implementation relies on a "flock of bluebirds:" Our before methods and the underlying base method are composed with each other to define the method that is actually executed at run time.

> This could be even more useful if it supported methods with blocks. Adventurous readers may want to combine this code with the tricks in [cardinal.rb](http://github.com/raganwald/homoiconic/tree/master/2008-10-31/cardinal.rb) and see if they can build a version of `before` that supports methods that take blocks.

**the super keyword, perhaps you've heard of it?**

Of course, Ruby provides a means of 'decorating' methods like this by overriding a method and calling `super` within it. So we might have written:

	class Foo < SuperFoo

	  def one_parameter(x)
	    super(x * 2)
	  end

	  def two_parameters(x, y)
	    super(x + y, x - y)
	  end

	end
	
On a trivial example, the two techniques seem equivalent, so why bother with the extra baggage? The answer is that using `super` is a little low level. When you see a method definition in a language like Ruby, you don't know whether you are defining a new method, overriding an existing method with entirely new functionality, or "decorating" a method with before advice.

Using a before method signals exactly what you are trying to accomplish. And besides, before methods also work with methods defined in modules and the current class, not just superclasses. So in some ways they are even more flexible than Ruby's built-in meta-object protocol allows.

**the queer bird**

That looks handy. But we also want an _after method_, a way to compose methods in the other order. Good news, the queer bird combinator is exactly what we want. 


[![happy pride (c) 2008 penguincakes, some rights reserved reserved](http://farm4.static.flickr.com/3035/2891197379_556f528536.jpg)](http://www.flickr.com/photos/penguincakes/2891197379/ "happy pride (c) 2008 penguincakes, some rights reserved")  


Written `Qxyz = y(xz)`, the Ruby equivalent is:

	queer_bird.call(something).call(stuff_to_do_after_we_do_stuff).call(value)
		=> stuff_to_do_after_we_do_stuff.call(something.call(value))

Which is, of course:

	def something(parameter)
		# do stuff...
	end
	
	after :something do |parameter|
		# stuff to do AFTER we do stuff...
	end

We can copy, paste and modify our bluebird code for the before methods. But before you rush off to implement that, you might want to think about a few interesting "real world" requirements:

1. If you define before and after methods in any order, the final result should be that all of the before methods are run before the main method, then all of the after methods. This is not part of combinatory logic, but it's the standard behaviour people expect from before and after methods.
2. If you override the main method, the before and after methods should still work.

Here's one implementation meeting these requirements: [before\_and\_after\_advice.rb](http://github.com/raganwald/homoiconic/tree/master/2008-11-07/before_and_after_advice.rb "before_and_after_advice.rb"). As you can see, the basic pattern of composing methods is still there, however it is now surrounded by a lot of extra moving parts, which is why we looked at supporting just before methods first. If you are comfortable with the [na&iuml;ve implementation of before advice](http://github.com/raganwald/homoiconic/tree/master/2008-11-07/naive_before_advice.rb), the mechanism is easy to understand.

_Our aviary so far_: [Kestrels](http://github.com/raganwald/homoiconic/tree/master/2008-10-29/kestrel.markdown), [The Thrush](http://github.com/raganwald/homoiconic/tree/master/2008-10-30/thrush.markdown), [Songs of the Cardinal](http://github.com/raganwald/homoiconic/tree/master/2008-10-31/songs_of_the_cardinal.markdown), [Quirky Birds and Meta-Syntactic Programming](http://github.com/raganwald/homoiconic/tree/master/2008-11-04/quirky_birds_and_meta_syntactic_programming.markdown), and [From Birds that Compose to Meta-Object Protocols](http://github.com/raganwald/homoiconic/tree/master/2008-11-07/from_birds_that_compose_to_method_advice.markdown).

---

[homoiconic](http://github.com/raganwald/homoiconic/tree/master "Homoiconic on GitHub")
	
Subscribe here to [a constant stream of updates](http://github.com/feeds/raganwald/commits/homoiconic/master "Recent Commits to homoiconic"), or subscribe here to [new posts and daily links only](http://feeds.feedburner.com/raganwald "raganwald's rss feed").

<a href="http://feeds.feedburner.com/raganwald"><img src="http://feeds.feedburner.com/~fc/raganwald?bg=&amp;fg=&amp;anim=" height="26" width="88" style="border:0" alt="" align="top"/></a>