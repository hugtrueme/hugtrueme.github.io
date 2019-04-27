---
layout: post
title:  "Learn to make your own Ruby gem in 10 minutes"
date:   2019-03-16 22:40:00 +0800
categories: Ruby
tags: Ruby Gem
---

You are able to create a gem if you have the 3 things in a directory.

1. A **lib** directory.
2. Name a Ruby file the same as your gem name.
3. A gemspec file, which contains information about the gem, like version number, author’s email and name etc...


This is an example created in my local, a gem named hello and its directory.
```
% tree
.
├── hello.gemspec
└── lib
    └── hello.rb

1 directory, 2 files
```

This is the content of the gemspec file.
```
% cat hello.gemspec
Gem::Specification.new do |s|
  s.name = 'hello'
  s.version = '0.0.0'
  s.date = '2019-03-12'
  s.summary = "Hello!"
  s.description = "A simple hello world gem"
  s.authors = ["Joey Chung"]
  s.email = “hugtruem@gmail.com"
  s.files = ["lib/hello.rb"]
  s.homepage = ‘https://hugtrueme.github.com'
  s.license = 'MIT'
end
```

Put your ruby code in 'lib/hello.rb'.
```
% cat lib/hello.rb
class Hello
  def self.hi
    puts 'Hello World!'
  end
end
```

Build the gem.
```
% gem build hello.gemspec
  Successfully built RubyGem
  Name: hello
  Version: 0.0.0
  File: hello-0.0.0.gem
```

Install the gem.
```
% gem install ./hello-0.0.0.gem
Successfully installed hello-0.0.0
Parsing documentation for hello-0.0.0
Installing ri documentation for hello-0.0.0
Done installing documentation for hello after 0 seconds
1 gem installed

% gem list hello

*** LOCAL GEMS ***

hello (0.0.0)
```

Test it in irb, you will see the hello gem directory in your $LOAD_PATH after you require it. (Tip: When you require a gem, really you’re just placing that gem’s lib directory onto your $LOAD_PATH.)
```
% irb -rap
2.4.4 :001 > require 'hello'
 => true
2.4.4 :002 > ap $LOAD_PATH
[
    [ 0] "/Users/joeychung/.rvm/gems/ruby-2.4.4@global/gems/did_you_mean-1.1.0/lib",
    [ 1] "/Users/joeychung/.rvm/gems/ruby-2.4.4/gems/awesome_print-1.8.0/lib",
    [ 2] "/Users/joeychung/.rvm/gems/ruby-2.4.4/gems/hello-0.0.0/lib",
    [ 3] "/Users/joeychung/.rvm/rubies/ruby-2.4.4/lib/ruby/site_ruby/2.4.0",
    [ 4] "/Users/joeychung/.rvm/rubies/ruby-2.4.4/lib/ruby/site_ruby/2.4.0/x86_64-darwin17",
    [ 5] "/Users/joeychung/.rvm/rubies/ruby-2.4.4/lib/ruby/site_ruby",
    [ 6] "/Users/joeychung/.rvm/rubies/ruby-2.4.4/lib/ruby/vendor_ruby/2.4.0",
    [ 7] "/Users/joeychung/.rvm/rubies/ruby-2.4.4/lib/ruby/vendor_ruby/2.4.0/x86_64-darwin17",
    [ 8] "/Users/joeychung/.rvm/rubies/ruby-2.4.4/lib/ruby/vendor_ruby",
    [ 9] "/Users/joeychung/.rvm/rubies/ruby-2.4.4/lib/ruby/2.4.0",
    [10] "/Users/joeychung/.rvm/rubies/ruby-2.4.4/lib/ruby/2.4.0/x86_64-darwin17"
]
 => nil
2.4.4 :003 > Hello.hi
Hello World!
 => nil
```

Uninstall it.
```
% gem uninstall hello
Successfully uninstalled hello-0.0.0
```