Circular
===

    AdvancedOpenWater:~ raganwald$ sudo gem install twitter

...

    AdvancedOpenWater:~ raganwald$ irb
    irb(main):001:0> require 'rubygems'
    => true
    irb(main):002:0> require 'twitter'
    => true
    irb(main):003:0> twit = Twitter::Base.new('raganwald', 'already_changed_back')
    => #<Twitter::Base:0x1141f6c @api_host="twitter.com", @proxy_host=nil, @config={:email=>"raganwald", :password=>"already_changed_back"}, @proxy_port=nil>
    irb(main):004:0> twit.update 'http://github.com/raganwald/homoiconic/blob/master/2009-03-03/circular.md'
    => #<Twitter::Status:0x11356cc @truncated=false, @source="web", @favorited=false,
      @text="http://tinyurl.com/agrcgh", @created_at="Tue Mar 03 20:33:19 +0000 2009", @in_reply_to_user_id="",
      @user=#<Twitter::User:0x1125178 @location="Toronto", @name="Reg Braithwaite", @protected=false,
      @profile_image_url="http://s3.amazonaws.com/twitter_production/profile_images/79244481/Picture_1_normal.png",
      @followers_count="412", @screen_name="raganwald", @url="http://braythwayt.com", @description="",
      @id="18137723">, @in_reply_to_status_id="", @id="1275406032">

---

Recent work:

* [JavaScript Allonge](http://leanpub.com/javascript-allonge), [CoffeeScript Ristretto](http://leanpub.com/coffeescript-ristretto), and my [other books](http://leanpub.com/u/raganwald).
* [Method Combinators](https://github.com/raganwald/method-combinators), a CoffeeScript/JavaScript library for writing method decorators, simply and easily.
* [Katy](http://github.com/raganwald/Katy), a library for writing fluent CoffeeScript and JavaScript using combinators.
* [jQuery Combinators](http://githiub.com/raganwald/jquery-combinators), what else? A jQuery plugin for writing your own fluent, jQuery-like code.  

---

[Reg Braithwaite](http://braythwayt.com) | [@raganwald](http://twitter.com/raganwald)
