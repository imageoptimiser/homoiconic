You keep using that idiom. I do not think it means what you think it means.
---

From time to time `returning` seems to violate the so-called _principle of least surprise_:

	returning([]) do |arr|
	  arr += [:foo, :bar]
	end
	  => []
	
Let's work up from pure simplicity:

	begin
	  x = 1
	  y = x
	  x = x + 1
	  y
	end
	  => 1
	
	begin
	  x = []
	  y = x
	  x = x + [:foo, :bar]
	  y
	end
	  => []
	
	begin
	  x = []
	  y = x
	  x += [:foo, :bar]
	  y
	end
	  => []
	
Compare and contrast with:
	
	begin
	  x = []
	  y = x
	  x.concat [:foo, :bar]
	  y
	end
	  => [:foo, :bar]

Which leads to:

	returning([]) do |arr|
	  arr.concat [:foo, :bar]
	end
	  => [:foo, :bar]

There is similar behaviour with things like `returning('') do ...`, any time you have a value with constructive methods (methods that return copies of the object). Things like ActiveRecord instances almost never have methods with these semantics, so people almost never get this mixed up.

The principles behind this code are much more basic than `returning`, it's really a question of understanding values, references, and which Ruby methods are destructive and which are constructive. I note that it is not part of the Ruby culture to disambiguate them. For example, `!` is used in the standard library for destructive methods, but only when a constructive version of the same method already exists.

It is a matter of wonder why ordering arrays is handled with `sort` and `sort!`, but catenation is handled with `+` and `concat`.

---

Recent work:

* [JavaScript Allonge](http://leanpub.com/javascript-allonge), [CoffeeScript Ristretto](http://leanpub.com/coffeescript-ristretto), and my [other books](http://leanpub.com/u/raganwald).
* [Method Combinators](https://github.com/raganwald/method-combinators), a CoffeeScript/JavaScript library for writing method decorators, simply and easily.
* [Katy](http://github.com/raganwald/Katy), a library for writing fluent CoffeeScript and JavaScript using combinators.
* [jQuery Combinators](http://githiub.com/raganwald/jquery-combinators), what else? A jQuery plugin for writing your own fluent, jQuery-like code.  

---

[Reg Braithwaite](http://braythwayt.com) | [@raganwald](http://twitter.com/raganwald)