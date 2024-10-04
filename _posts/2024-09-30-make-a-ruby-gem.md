---
layout: post
title: Make A Ruby Gem
date: 2024-09-30 12:21 -0700
---

I will document my development process for writing a new [Ruby](https://www.ruby-lang.org/en/) gem here. There are various official guides for doing this, the most authoritative being the documentation at [RubyGems](https://guides.rubyGems.org/make-your-own-gem/) and at [Bundler](https://bundler.io/guides/creating_gem.html). These two guides have some differences. The RubyGems guide is simple and straightforward, and the Bundler guide is more full-featured, and more recent. RubyGems even refers to the Bundler guide, and so that is de-facto official. I have a few issues with that guide, however, and don't think it's entirely helpful as a reference. That guide has an extreme Test-Driven Development approach, which I find hard to follow (I am a fan of TDD in principle, but it often makes tutorials hard to read), and uses _RSpec_/_Aruba_/_Cucumber_ tooling, which I find excessive and cumbersome. I prefer a minimalist, _Minitest_-based approach. [The Minitest gem](https://github.com/minitest/minitest) is also built in to the `bundle gem` generator by default, which the Bundler guide oddly ignores. That guide is also written with an odd, humorous style, which glosses over some details and contextual information, which I again find confusing. I subscribe to the view that ["good code is boring"](https://news.ycombinator.com/item?id=2167539), and I would like this article to be similarly boring (in the good sense). I will try to fill in gaps in the other guides, and I will try to be more explicit and verbose regarding contextual information, workflow, and integrations with other code and tooling.

The code demonstrated here is available [on Github](https://github.com/brian-davis/urban-engine) for following along.

**_Set Up_**

I'm not going to start from absolutely nothing. I will assume this about your development environment:

- You have installed a package manager, version manager, and a version of Ruby 3, and have a properly configured shell environment. A good, up-to-date guide for all of this is at [GoRails](https://gorails.com/setup). That page should default to the correct guide for your operating system. I am using _macOS 13 (Ventura)_ on an _M1 (amd64)_ Mac computer, the Ventura _.zsh_ shell, the _asdf_ version manager, and _Ruby 3.2.2_. Beware of any documented gotchas integrating all of these elements (e.g., setting up a `~/.zshrc` file with the correct `PATH` and other directives). I also recommend adding `export EDITOR="code --wait"` to `.zshrc`

- You have [installed Git](https://git-scm.com/book/en/v2/Getting-Started-Installing-Git) and set up your local git credentials such as you might need for [pushing to Github](https://docs.github.com/en/authentication/keeping-your-account-and-data-secure/about-authentication-to-github) (or similar). You have set up a Github (or similar) account where you can build a remote repo for the new gem.

- You have set up a [RubyGems](https://guides.rubyGems.org) account if you want to distrubute via RubyGems.

- You have a code editor such as [VSCode](https://code.visualstudio.com/) and are familiar with basic Ruby coding, and with the command line.

Finally, think about the branding/naming of the new gem, and visit [RubyGems](https://rubygems.org/) and search for the name of your new gem, to verify it hasn't already been taken.

**_Initialize_**

Both _Bundler_ (`bundle`) and _RubyGems_ (`gem`) come standard with Ruby. Use `bundle` to initialize your gem:

```bash
$ bundle gem gem_demo
$ cd gem_demo
$ code .
```

`gem_demo` is the unimaginative name of this new gem demo. Note that I'm using an underscore name here, to demonstrate how snake_case/camelCase works out.

Update `gem_demo.gemspec`, to remove the `TODO` lines that `bundle` will complain about:

```bash
$ bundle install
The gemspec at ~/gem_demo/gem_demo.gemspec is not valid. Please fix this gemspec.
The validation error was '"FIXME" or "TODO" is not a summary'
```

Open this file, and replace the strings that say `TODO` with contact info, descriptions, and relevant URLs. These can be changed later. These can even be changed to empty strings for now, except for the URL strings, which can be changed to `"http://www.example.com"` if you don't have a repo or homepage already. The line for `metadata["allowed_push_host"]` _can_ be left as a `TODO`, or even commented out.

In my humble opinion, RSpec and Cucumber are more trouble than they are worth, and do not need to be installed. Minitest is a more lightweight, Ruby-esque standard testing library, and it comes already built in from the `bundle gem` generator we just ran. Only one minitest decorator gem, `minitest-reporters`, needs to be added, for colorized output. Because the built-in `minitest` gem already exists in `Gemfile`, (not in `gem_demo.gemspec`), we'll follow this pattern and use `Gemfile` for development dependencies instead of `add_development_dependency` calls in `gem_demo.gemspec`.

Run this:

```bash
$ bundle add minitest-reporters
```

which will insert a directive for this gem to the `Gemfile`, under the line already there for Minitest.

Now configure this new gem in `test/test_helper.rb`

```ruby
require 'minitest/reporters'
Minitest::Reporters.use! [Minitest::Reporters::DefaultReporter.new(:color => true)]
```

now run

```bash
$ rake test
```

There should be one green and one red test, because the generator sets you up with a red `TODO` kind of test. See `test/test_gem_demo.rb`.

**_Code_**

Examine `lib/gem_demo.rb`

This is the entrypoint of the actual code. Notice that a `module GemDemo` has been created for you (camelCase), which will be the namespace for your code. This is where you can `require` any namespaced classes or modules. Assume that you won't do much coding directly in this class, but mainly in other namespaced classes or modules. Create a new file `lib/gem_demo/hello_world.rb` file, and add:

```ruby
module GemDemo
  class HelloWorld
    class << self
      def call(name = "World")
        "Hello, #{name}!"
      end
    end
  end
end
```

This is our functional code, hopefully it is self-explanatory. In `lib/gem_demo.rb`, add

```ruby
require_relative "gem_demo/hello_world"
```

If this were a [Rails](https://guides.rubyonrails.org/) app, we could rely on `autoload_paths` and other Rails magic to find and load classes based on filenames, but here we just use `require` or `require_relative` directly. I like to use `require_relative` for files within this gem's project directory, and `require` for outside gem dependencies which we bring in. Now anything that requires the top-level `GemDemo` module, such as `test/test_helper.rb`, will also get the new `HelloWorld` class. Update `test/test_helper.rb` to use `require_relative` for consistency, instead of the generated code:

```ruby
# $LOAD_PATH.unshift File.expand_path("../lib", __dir__)
# require "gem_demo"
require_relative "../lib/gem_demo"
```

Note that `require_relative` also ensures that we are pulling from the working project, and not reverting to previously `gem install`-ed versions of the code which live in the gem cache (see below), as might happen with a `require "gem_demo"` call.

In `test/test_gem_demo.rb`, update the failing test:

```ruby
def test_it_does_something_useful
  assert_equal("Hello, World!", GemDemo::HelloWorld.call)
end
```

```bash
$ rake test
```

will now run green with 2 passing tests.

**_Dependencies_**

Now to add an example full gem dependency which is used by the internal logic of the gem, unlike the development dependencies which are just tooling.

[Text](https://github.com/threedaymonk/text) is an interesting gem that adds text comparison functionality. The [rubyGems page](https://rubyGems.org/gems/text/versions/1.3.1) for the gem says that v1.3 is the current version. We will add it to the `gem_demo.gemspec` file:

```ruby
  spec.add_dependency "text", "~> 1.3"
```

and run:

```bash
$ bundle install
```

Let's demo the functionality we're borrowing from this gem within an instance method in the HelloWorld class.

In `lib/gem_demo.rb`, add:

```ruby
require "text"
```

above any `require_relative` lines, and update `lib/gem_demo/hello_world.rb` like this:

```ruby
module GemDemo
  class HelloWorld
    class << self
      def call(name = "World")
        "Hello, #{name}!"
      end
    end

    attr_reader :name

    def initialize(name)
      @name = name
    end

    def name_distance(other_name)
      Text::Levenshtein.distance(name, other_name)
    end
  end
end
```

At this point, let's do some simple hands-on testing. The `bundle gem` command provided us with a handy executable out of the box (which is oddly not documented in the guide). Read the `bin/console` file to see how it works. It will start an _irb_ REPL, which is configured to require the gem code we just wrote. I like to use `require_relative` here as well:

```ruby
#!/usr/bin/env ruby
require "bundler/setup"
require_relative "../lib/gem_demo"

require "irb"
IRB.start(__FILE__)
```

If you were writing an ['application'](https://yehudakatz.com/2010/12/16/clarifying-the-roles-of-the-gemspec-and-gemfile/) which required _this gem_, you would use the `GemDemo::HelloWorld` class in _that_ application codebase, just like you would use it in this REPL.

From the project root directory, do this:

```bash
$ bin/console
```

or

```bash
$ ./bin/console
```

Then:

```ruby
>> aa = GemDemo::HelloWorld.new("Bob")
=> #<GemDemo::HelloWorld:0x00000001032f9c08 @name="Bob">
>> aa.name_distance("Rob")
=> 1
>> GemDemo::HelloWorld.call("Universe")
=> "Hello, Universe!"
>> exit
```

This is how the new class and its various methods can be used in a development environment (not just the test environment of the `/test` files).

That said, let's write some automated tests, and also refactor the test file structure a bit. Create `test/test_hello_world.rb`, cut the `test_it_does_something_useful` test from the first test file, then add this code to the new test file:

```ruby
# frozen_string_literal: true

require_relative "test_helper"

class TestHelloWorld < Minitest::Test
  def test_it_does_something_useful
    assert_equal("Hello, World!", GemDemo::HelloWorld.call)
  end

  def test_name_distance
    subject = GemDemo::HelloWorld.new("Jim")
    assert_equal(1, subject.name_distance("Tim"))
  end
end
```

```bash
$ rake test
```

will now run green with 3 passing tests.

Now is a good time to commit. Run:

```bash
$ git status
```

which should show some red, uncommitted code. Now:

```bash
$ git add -A
$ git commit -m "HelloWorld code"
```

**_Release Preparation_**

Before going through the CLI section of the Bundler guide, let's take this development environment concept one step further, because not every gem we make will actually need a CLI. We'll skip to the bottom of that guide and do something similar to the section _Releasing the Gem_ there. The `gem-release` development tool gem mentioned there is a good suggestion:

```bash
$ gem install gem-release
```

Notice that we are doing `gem install` rather than `bundle add`. The important thing is to understand this is being installed as a system-wide CLI development tool (namespaced under the current Ruby version gem cache created by the version manager). It is not a dependency of this specific `gem_demo` gem we are working on. We wouldn't want to have to run `bundle exec` with this tool, we just want it to be available anywhere. It provides a `gem bump` command for easy [semantic versioning](https://semver.org/). So let's use it now:

```bash
$ gem bump --version minor

Bumping gem_demo from version 0.1.0 to 0.2.0
Changing version in lib/gem_demo/version.rb from 0.1.0 to 0.2.0
Staging lib/gem_demo/version.rb

$ git add lib/gem_demo/version.rb

Creating commit

$ git commit -m "Bump gem_demo to 0.2.0"
[main e163eae] Bump gem_demo to 0.2.0
 1 file changed, 1 insertion(+), 1 deletion(-)
```

Notice that it has hooked into the namespaced `GemDemo::VERSION` constant with the version number in `lib/gem_demo/version.rb`, which is also referenced in the default test in `lib/gem_demo/version.rb` and in the `gem_demo.gemspec`. It has also automatically created a version bump commit which you can verify with:

```bash
$ git log
```

We're now going to do a _local_ development environment `gem install` with _this_ `gem_demo` code we've just made:

```bash
$ rake build
gem_demo 0.2.0 built to pkg/gem_demo-0.2.0.gem.

$ gem install pkg/gem_demo-0.2.0.gem --local
Successfully installed gem_demo-0.2.0
1 gem installed

$ gem list gem_demo

*** LOCAL GEMS ***

gem_demo (0.2.0)

$ gem info gem_demo

*** LOCAL GEMS ***

gem_demo (0.2.0)
    Author: Developer name
    Homepage: http://www.example.com
    License: MIT
    Installed at: ~/.asdf/installs/ruby/3.2.2/lib/ruby/gems/3.2.0

$ ls -lha ~/.asdf/installs/ruby/3.2.2/lib/ruby/gems/3.2.0/gems | grep gem_demo
...gem_demo-0.2.0
```

Note that I've used `rake build`, rather than `gem build`, which will also work, but will save the new package to the project root directory, rather than under `/pkg` like `rake build` will do.

The new code is now available system-wide, just like the _gem-release_ gem tool we previously installed. Our gem is not a CLI kind of tool (yet), so we won't use it like that. Instead `cd` back to the home directory, so you're out of the development context. This directory has nothing to do with the development directory we were using, other than (I assume) being within the same Ruby version specified by the version manager, and thus having access to the same cached gems via the `PATH` environment variable and shims.

```bash
$ cd ~
$ irb
```

Then:

```ruby
>> require "gem_demo"
=> true
>> GemDemo::HelloWorld.call("home directory")
=> "Hello, home directory!"
>> GemDemo::HelloWorld.new("John").name_distance("Paul")
=> 4
>> exit
```

Manually entering `require "gem_demo"` found the new gem in the local gem cache, and it can be used just as when we used the `bin/console` within the project directory, or just as if it were any other gem released by someone else and pulled down from rubyGems or Github via `bundle install` or via `gem install`.

**_Release_**

At this point, if this were a real gem, not a demo, we would be able to update the placeholder text in the .gemspec file with production URLs and contact info, and then run `rake release` which would push the code to those endpoints.

You can inspect what this rake task does at: the gem's `lib/bundler/gem_helper.rb` (Use `gem open bundler` to open a VSCode workspace in the install directory for the gem.)

```ruby
    def allowed_push_host
      @gemspec.metadata["allowed_push_host"] if @gemspec.respond_to?(:metadata)
    end

    def gem_push_host
      env_rubyGems_host = ENV["RUBYGEMS_HOST"]
      env_rubyGems_host = nil if
        env_rubyGems_host && env_rubyGems_host.empty?

      allowed_push_host || env_rubyGems_host || "rubyGems.org"
    end
```

It will look up the `allowed_push_host` set in the `gem_demo.gemspec` line, or will default to `"rubyGems.org"`. See the [RubyGems documentation](https://guides.rubyGems.org/publishing/) for more information on publishing. RubyGems is standard, so I recommend commenting out `metadata["allowed_push_host"]` in `gem_demo.gemspec`.

If you want to do a real release at this point, first update `gem_demo.gemspec` with the Github repo (substituting your own values):

```ruby
spec.metadata["source_code_uri"] = "https://github.com/user-name/repo-name"
```

And more importantly, add the repo as a `git remote`:

```bash
$ git remote add origin git@github.com:user-name/repo-name.git
```

The release push to RubyGems will fail if the `name` value in the `gem_demo.gemspec` isn't a new, unique gem name to RubyGems. Search there to make sure it has not already been taken. _NOTE_ that literal "gem_demo" has already been taken!

<img src="https://dev-blog-images-2024.s3.us-west-1.amazonaws.com/article-images/search2.jpeg" alt="gem search" width="400px">

Just look at all those "demo\_" gems! Remember to clean up after yourself with `$ gem yank` if you don't really want a demo gem to live on the internet forever.

in `gem_demo.gemspec`:

```ruby
spec.name = "totally_unique_gem_name"
```

This _can_ differ from the real name of the gem, but you would probably want to check uniqueness _first_ for any real, non-demo gem you make, for consistency. Make sure you have no uncommitted changes, then:

```
$ rake release
gem_demo 0.3.0 built to pkg/gem_demo-0.3.0.gem.
Tagged v0.3.0.
Pushed git commits and release tag.
Enter your RubyGems.org credentials.
Don't have an account yet? Create one at https://rubyGems.org/sign_up
   Email:   ...
Password:   ...

You have enabled multi-factor authentication. Please enter OTP code.
Code: ...
Signed in with API key: ....
Pushing gem to https://rubyGems.org...
Successfully registered gem: totally_unique_gem_name (0.3.0)
Pushed totally_unique_gem_name 0.3.0 to rubyGems.org
```

The push will create a new API key if necessary, which can be managed when logged in at the RubyGems site. You will be prompted for your rubyGems credentials. You have now released a versioned gem to the world. Remember to bump the version as needed while you develop features further. RubyGems will track each version of the gem separately, and you should think of each released version as being a kind of "final" version. You might have users in the real world who lock themselves into any previous version of your code.

If you would rather not release to RubyGems, you can simply not run `rake release`, and instead use only `git push origin main`, and instruct gem users to bundle or pull directly [from Github](https://bundler.io/guides/git.html) in the `README.md` in the project root directory.

**_CLI Development_**

Not every gem needs to be a Command-Line Interface (CLI) application. But this is a common use case. Normally, a CLI gem will be more like an _application_ and won't be the kind of gem you `require` into _another_ application (or REPL), like we've done so far. It would instead be a `gem install` kind of gem, like the `bump` command from `gem-release` that we saw earlier. It is conceptually different from a _library_ kind of gem, used in the `gemspec` or `Gemfile` in another application, or even in an application development context with `bundle exec demo_command`. However, there is no reason other than convention it can't work both ways. So we'll follow the lead of the Bundler guide and simply extend the functionality of the demo gem we've just built with some CLI functionality.

To do this, we'll pull in the [_thor_ gem](https://github.com/rails/thor), just as in the guide, because it is the de-facto standard. Rails uses it, as does Bundler itself. Read the code at the `gems/bundler-2.4.15/lib/bundler/gem_helper.rb` cited above to see some _thor_ coding in action. I have also used the similar [_commander_ gem](https://github.com/commander-rb/commander) [with success](https://github.com/brian-davis/gep_kml), but if _thor_ is good enough for Bundler, then it's good enough for demo purposes here. The best documentation for this gem is [the Github wiki](https://github.com/rails/thor/wiki/Method-Options).

Unlike the Bundler guide, we will _not_ be using _Aruba_ and/or _Cucumber_ for testing, but I will instead demonstrate a way to integrate CLI command testing with _Minitest_.

Find [the latest version](https://rubyGems.org/gems/thor) of `thor` and add it to `gem_demo.gemspec`, because this is a full dependency:

```ruby
spec.add_dependency "thor", "~> 1.2"
```

Then

```bash
$ bundle install
```

Make a new `cli/gem_demo_cli.rb` directory and file, and add this code:

```ruby
require "thor"
require_relative "../lib/gem_demo"

class GemDemoCLI < Thor
  # $ bin/gem_demo call George
  desc "call NAME", "Say hello to NAME"
  def call(name = "World")
    puts GemDemo::HelloWorld.call(name)
  end

  # $ bin/gem_demo name_distance Ringo --from Richard
  desc "name_distance NAME", "Show how distant two names are"
  method_option :from, aliases: "-f"
  def name_distance(name)
    hw = GemDemo::HelloWorld.new(name)
    puts hw.name_distance(options[:from])
  end

  # https://github.com/rails/thor/issues/244
  def self.exit_on_failure?
    true # 1
  end
end
```

Notice that I have added this to a new top-level `/cli` directory apart from the library files (not under `/lib` or `/lib/gem_demo`). I'm differing from the Bundler guide here. I think of the CLI as a separate little app that should be partitioned from the main logic of the library that lives in `lib`. Notice also the `require_relative` here which reaches across to require the `lib/gem_demo.rb` master file into itself. The CLI class is not nested under the `GemDemo` module, but is instead acting as a parallel wrapper. Notice that I `require "thor"` at the top of this file, and not from `lib/gem_demo`, because I do not need _thor_ CLI functionality in the main library. If we do use the internal `lib/gem_demo` logic as a `bundle` kind of gem included into another application, like we did with `bin/console`, we won't be loading all of the unnecessary `Thor` subclassed code that wouldn't work there anyway.

From the project root directory,

```bash
$ bin/console
```

and:

```ruby
>> GemDemoCLI
(irb):1:in `<main>': uninitialized constant GemDemoCLI (NameError)
	from bin/console:11:in `<main>'
```

This error is a good thing here in the console, and demonstrates the separation of concerns.

There are two example methods in the new `GemDemoCLI` class, which will both be made available (soon) from the command line. The purpose of this class which inherits from `Thor` is to parse command-line arguments and options, then to pass them into our library code, and to output results back to the shell. We do that with simple `puts` statements to print return values back to `STDOUT`. Note that the `aliases` option (e.g. `"-f"`) can only be in the format of a single dash and a single letter character, by convention, or else _thor_ will throw an error at runtime. Also note that the `call` method in this file duplicates the default argument `= "World"`, because _thor_ will throw an error when parsing an empty argument.

There is also a `self.exit_on_failure?` method which has been added to gracefully handle errors, as in the case where you enter a typo to the command line. So something like

```bash
$ bin/gem_demo nam_distance Bill --from Bob
```

will return a polite _thor_-generated error message, rather than trhowing a nasty exception and stacktrace. This is [an old gotcha](https://github.com/rails/thor/issues/244).

I've also added temporary documentation comments with examples of how these methods should be called from the command line, in development-mode. More on that below.

The final step to get this working is to make an executable file which will expose _this_ interface class to the shell. Make a `bin/gem_demo` (no extension) file alongside the `bin/console` and `bin/setup` executables which the generator has already provided. Theoretically, this could be named anything, but this is what your users will be typing into the command line, so it will effectively be the real name of your gem, so choose wisely and make it match the filenames and names throughout the code, for sanity's sake. I do not like the `exe` suggestion in the Bundler guide, and in my opinion it it better to put [_binstub_](https://devcenter.heroku.com/articles/ruby-binstub-shebang) files in `bin`. The [RubyGems guide](https://guides.rubyGems.org/make-your-own-gem/) says, in the _ADDING AN EXECUTABLE_ section:

> Adding an executable to a gem is a simple process. You just need to place the file in your gem’s bin directory, and then add it to the list of executables in the gemspec.

We will follow these guidelines instead of the Bundler guide.

The `gem_demo.gemspec` file should be updated like this:

```ruby
  # spec.bindir = "exe"
  # spec.executables = spec.files.grep(%r{\Aexe/}) { |f| File.basename(f) }
  spec.bindir = "bin"
  spec.executables << "gem_demo"
```

This is used by `gem install` to locate the relevant executable binstubs within the local gem cache when we are doing system-wide CLI development. I have changed the `bundle gem` generated boilerplate from an `"exe"` value to `"bin"`, and have replaced the line which globs _all_ executables in the `bin` directory with a one-by-one approach suggested by RubyGems. The `console` and `setup` binstubs provided by the generator do not need to be added to the `spec.executables` because they do not need to be exposed to the end-user as a system-wide install. They will still work for the developer when run from the project root as `./bin/console`.

Add this to the `bin/gem_demo` file:

```ruby
#!/usr/bin/env ruby

require_relative '../cli/gem_demo_cli'
GemDemoCLI.start
```

The binstub file is Ruby code, and we include a magic-comment _shebang_ at the first line. This will tell the system to use the default Ruby interpreter (as in `$ which ruby`) to run this code. This file then uses `require_relative` to find the nearby `/cli/gem_demo_cli.rb` just written, load it, and run it. Next, make the permissions on this file executable:

```bash
$ chmod a+x bin/gem_demo
```

Now, in the root directory of the project, try

```bash
$ bin/gem_demo
```

or

```bash
$ ./bin/gem_demo
```

This is executing the `bin/gem_demo` binstub file locally and relatively (no `PATH` lookup). Because this call is missing the required arguments (the method names defined in `lib/gem_demo_cli.rb`), the functionality inherited from `Thor` will display auto-generated help text, based on the `desc` calls above the methods.

```bash
Commands:
  gem_demo call NAME       # Say hello to NAME
  gem_demo help [COMMAND]  # Describe available commands or one specific command
  gem_demo name_distance   # Show how distant two names are
```

For demonstration purposes, we'll do some development-mode manual testing before writing automated tests. From the project root directory, run:

```bash
$ bin/gem_demo call Bob
Hello, Bob!
$ bin/gem_demo name_distance Tom --from Joe
2
```

Hopefully this works, and you have now written _a command_. Before further builds and releases, we'll write some real tests. These will look different as they are _end-to-end_ (aka _integration_, _feature_) tests which will black-box the `lib/gem_demo` internals, and instead call the CLI as a subprocess, just as if you were running the binstub commands from the command line. In a new file, `test/test_gem_demo_cli.rb` add this:

```ruby
# frozen_string_literal: true

require_relative "test_helper"

class TestGemDemoCLI < Minitest::Test
  CLI_EXECUTABLE = File.expand_path(File.join(__dir__, "../bin/gem_demo")).freeze

  def test_cli_call_default
    out, err = capture_subprocess_io do
      system("#{CLI_EXECUTABLE} call")
    end

    assert_equal("Hello, World!\n", out)
    assert_empty(err)
  end

  def test_cli_call_argument
    out, err = capture_subprocess_io do
      system("#{CLI_EXECUTABLE} call people")
    end

    assert_equal("Hello, people!\n", out)
    assert_empty(err)
  end

  def test_cli_name_distance
    out, err = capture_subprocess_io do
      system("#{CLI_EXECUTABLE} name_distance Will --from Bill")
    end

    assert_equal("1\n", out)
    assert_empty(err)
  end

  def test_cli_name_distance_alias
    out, err = capture_subprocess_io do
      system("#{CLI_EXECUTABLE} name_distance Tom -f Joe")
    end

    assert_equal("2\n", out)
    assert_empty(err)
  end
end
```

Note that I am making no reference to `GemDemoCLI` or to any `GemDemo` classes or internals here. The _Minitest_ gem has a handy `capture_subprocess` [helper method](https://www.rubydoc.info/gems/minitest/5.10.3/Minitest/Assertions#capture_io-instance_method) built in, which we can use to capture both `STDOUT` and `STDERR` from any `system` calls, which is one way to run a shell command as a subprocess from a ruby program. Note that the `err` value is the text captured from `STDERR`, not an exit code which you might inspect with `echo $?` at the command line. The string arguments to `system` interpolate the relatively-determined binstub file and use the same syntax as was documented above.

```bash
$ rake test
```

now runs green with 7 passing tests.

**_Re-Release_**

Now that this much has been done, it is a good time to re-commit, re-build, and re-release (locally).

```bash
$ git status
$ git add -A
$ git commit -m "CLI set up"
$ gem bump --version minor
$ gem uninstall gem_demo # 0.2.0
Successfully uninstalled gem_demo-0.2.0
$ rake build
gem_demo 0.3.0 built to pkg/gem_demo-0.3.0.gem.
$ gem install pkg/gem_demo-0.3.0.gem --local
Successfully installed gem_demo-0.3.0
1 gem installed
$ gem list gem_demo

*** LOCAL GEMS ***

gem_demo (0.3.0)
```

Notice that I was careful to uninstall the previous locally-installed version, to avoid clutter and confusion. The previous version's package still exists in the `/pkg` directory, so could be re-installed if needed. Also know that `gem` tracks the gem executables (the CLI code) separately from the gem library (the internal code you might `require` into a REPL or application codebase). If a previously-installed executable also needs to be removed. There will be a prompt:

```bash
Remove executables:
	gem_demo

in addition to the gem? [Yn]  Y
Removing gem_demo
```

Now let's do a final round of hands-on testing, running the commands based on the re-installed build in the gem cache, now using expanded `PATH` (not a manually-entered relative path).

```bash
$ cd ~
$ gem_demo call "friends"
Hello, friends!
$ irb
```

and:

```ruby
>> require "gem_demo"
=> true
>> GemDemo::HelloWorld.new("Alice").name_distance("Bob")
=> 5
>> exit
```

If `gem` is working nicely with the shell `PATH` and the version manager shims, the packaged binstub executable should be found automatically and it should just work anywhere. The `irb` REPL with `require "gem_demo"` functionality should still work with the library, as before.

Of course, if this were a real gem, not a demo, running `rake release` from the project directory should push the next version to the remote repositories, as before.

**_Documentation_**

One important consideration, which is not mentioned in the Bundler guide, is documentation. The RubyGems guide does have a section on _DOCUMENTING YOUR CODE_, which says:

> when you push a gem, RubyDoc.info generates YARDocs automatically from your gem

This is important. If you want your gem to be used by other developers, who might look up the nicely-formatted [rubydoc.info](https://rubydoc.info/) documentation, you should add some documentation comments. The guide reccomends using [_YARD_](https://rubydoc.info/gems/yard/file/docs/GettingStarted.md), so for the final section here, I will add some basic documentation according to the YARD guide.

Add the `yard` gem:

```bash
 $ gem install yard webrick rack
```

_webrick_ and _rack_ are dependencies for _yard_ which may not already be installed. Now add documentation comments above any original classes and methods. Here is the complete `lib/gem_demo.rb` file:

```ruby
# frozen_string_literal: true

require "text"

require_relative "gem_demo/version"
require_relative "gem_demo/hello_world"

# GemDemo is a demonstration of building a RubyGems library.
module GemDemo
  class Error < StandardError; end
end
```

And the complete `lib/gem_demo/hello_world.rb` file:

```ruby
module GemDemo
  # GemDemo::HelloWorld provides personal name functionality.
  class HelloWorld
    class << self

      # Compose a greeting message.
      #
      # @param name [String] the name to greet
      # @return [String] a greeting with the interpolated name.
      def call(name = "World")
        "Hello, #{name}!"
      end
    end

    # Controls the personal name.
    # @return [String] the personal name
    attr_reader :name

    def initialize(name)
      @name = name
    end

    # Compares the name to another name in terms
    # of number of changes necessary to make both equal.
    #
    # @param other_name [String] the name to compare
    # @return [Numeric] the number of changes required to make both names euqal.
    def name_distance(other_name)
      Text::Levenshtein.distance(name, other_name)
    end
  end
end
```

More detailed functionality is available in _YARD_'s own [documentation](https://rubydoc.info/gems/yard/file/README.md). As described there, we can generate documentation HTML from the project root directory with:

```bash
$ yardoc
Files:           4
Modules:         1 (    0 undocumented)
Classes:         3 (    2 undocumented)
Constants:       1 (    1 undocumented)
Attributes:      1 (    0 undocumented)
Methods:         6 (    2 undocumented)
 58.33% documented
```

The `/doc` directory, which is already registered in `.gitignore`, now has new files with the generated HTML. To access this, run a local server:

```bash
$ yard server
[2023-07-03 14:33:31] INFO  WEBrick 1.8.1
[2023-07-03 14:33:31] INFO  ruby 3.2.2 (2023-03-30) [arm64-darwin22]
[2023-07-03 14:33:31] INFO  WEBrick::HTTPServer#start: pid=16157 port=8808
```

and visit `localhost:8808` in a web browser.

<img src="https://dev-blog-images-2024.s3.us-west-1.amazonaws.com/article-images/yardoc.jpeg" alt="yardoc" width="400px">

This will preview the documentation which will be generated at [RubyDoc.info](https://rubydoc.info/).

Now we can commit:

```bash
$ git add -A
$ git commit -m 'documentation'
```

That should cover the essentials for setting up a new gem project. I hope this has been helpful in demonstrating not necessarily how to code Ruby itself, but how to use the standard library-building tools, _Rubygems_ and _Bundler_, for organizing code files, interacting with a local development environment, managing project versioning, and general workflow.
