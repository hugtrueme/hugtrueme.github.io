---
layout: post
title:  "Learn to make your own gem in 10 minutes"
date:   2018-10-16 23:00:00 +0800
categories: Ruby
tags: Ruby Gem
---


#### First, we should know what does the ‘require’ do?

When you require a gem, really you’re just placing that gem’s lib directory onto your $LOAD_PATH.

Tip: Passing -r to irb will automatically require a library when irb is loaded.

```
% irb -rap
2.4.4 :001 > ap $LOAD_PATH
[
    [0] "/Users/joeychung/.rvm/gems/ruby-2.4.4@global/gems/did_you_mean-1.1.0/lib",
    [1] "/Users/joeychung/.rvm/gems/ruby-2.4.4/gems/awesome_print-1.8.0/lib",
    [2] "/Users/joeychung/.rvm/rubies/ruby-2.4.4/lib/ruby/site_ruby/2.4.0",
    [3] "/Users/joeychung/.rvm/rubies/ruby-2.4.4/lib/ruby/site_ruby/2.4.0/x86_64-darwin17",
    [4] "/Users/joeychung/.rvm/rubies/ruby-2.4.4/lib/ruby/site_ruby",
    [5] "/Users/joeychung/.rvm/rubies/ruby-2.4.4/lib/ruby/vendor_ruby/2.4.0",
    [6] "/Users/joeychung/.rvm/rubies/ruby-2.4.4/lib/ruby/vendor_ruby/2.4.0/x86_64-darwin17",
    [7] "/Users/joeychung/.rvm/rubies/ruby-2.4.4/lib/ruby/vendor_ruby",
    [8] "/Users/joeychung/.rvm/rubies/ruby-2.4.4/lib/ruby/2.4.0",
    [9] "/Users/joeychung/.rvm/rubies/ruby-2.4.4/lib/ruby/2.4.0/x86_64-darwin17"
]
 => nil
```

#### What does a gem directory look like?

You are able to create a gem if you have the 3 things in a directory.

1. A **lib** directory.
2. Name a Ruby file the same as your gem name.
3. A gemspec file, which contains information about the gem, like version number, author’s email and name etc...

```
% tree /Users/joeychung/.rvm/gems/ruby-2.4.4/gems/cancancan-2.2.0/
/Users/joeychung/.rvm/gems/ruby-2.4.4/gems/cancancan-2.2.0/
├── cancancan.gemspec
├── init.rb
└── lib
    ├── cancan
    │   ├── ability
    │   │   ├── actions.rb
    │   │   └── rules.rb
    │   ├── ability.rb
    │   ├── conditions_matcher.rb
    │   ├── controller_additions.rb
    │   ├── controller_resource.rb
    │   ├── controller_resource_builder.rb
    │   ├── controller_resource_finder.rb
    │   ├── controller_resource_loader.rb
    │   ├── controller_resource_name_finder.rb
    │   ├── controller_resource_sanitizer.rb
    │   ├── exceptions.rb
    │   ├── matchers.rb
    │   ├── model_adapters
    │   │   ├── abstract_adapter.rb
    │   │   ├── active_record_4_adapter.rb
    │   │   ├── active_record_5_adapter.rb
    │   │   ├── active_record_adapter.rb
    │   │   ├── can_can
    │   │   │   └── model_adapters
    │   │   │       └── active_record_adapter
    │   │   │           └── joins.rb
    │   │   └── default_adapter.rb
    │   ├── model_additions.rb
    │   ├── rule.rb
    │   └── version.rb
    ├── cancan.rb
    ├── cancancan.rb
    └── generators
        └── cancan
            └── ability
                ├── USAGE
                ├── ability_generator.rb
                └── templates
                    └── ability.rb

11 directories, 29 files
```

#### Make your own gem

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

Test it in irb.
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