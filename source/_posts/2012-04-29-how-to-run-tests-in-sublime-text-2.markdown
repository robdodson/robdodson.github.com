---
layout: post
title: "How to Run Tests in Sublime Text 2"
date: 2012-04-29 09:26
comments: true
categories: 
---

If you're lazy like me then you love to automate as much of your process as possible. Running tests from within your IDE is one of those tasks that screams for a keyboard shortcut. In full featured tools like RubyMine or Eclipse this is usually pretty straightforward. However many developers in the Ruby community seem to prefer more lightweight tools like TextMate, Vim and Sublime. Today we'll look at how to setup RubyTest in Sublime Text 2 so we can easily run tests with just a few hotkeys. The tests will be written in [Rspec](http://rspec.info/) which should be familiar to most Rubyists.


###How to setup RubyTest in Sublime Text 2
If you haven't installed the [Sublime Package Manager](http://wbond.net/sublime_packages/package_control) go ahead and do that now. The package manager is a wonderful tool that lets us install and update plugins from within Sublime. After you have that installed you can hit `cmd-shift-p` to open up the Command Palette. Type the following `Install Package` and press Enter. This will bring up a list of available packages that we can install. You can also type `Discover Packages` which will take you to a page listing each plugin with a brief description.

Go ahead and type `RubyTest` and press Enter. You will see a progress indicator at the bottom of your editor. Once it's finished you should be able to click on `Tools > RubyTest`.


###Running tests in Sublime Text 2
I've put together [a sample project which you can download](https://github.com/robdodson/testing_demo). It has a Gemfile and a pair of contrived Rspec tests. Put it someplace where you can easily get to it using the command line. In order for RubyTest to work you have to open your projects from the CLI. This has to do with how certain load paths get setup and it's probably the cause of about 95% of RubyTest issues. Follow [these instructions](http://www.sublimetext.com/docs/2/osx_command_line.html) if you don't already have the `subl` command setup in your terminal.

`cd` into the project folder and do a quick `bundle install`. When that's finished try to run
```
bundle exec rspec spec/
```
If you see test results then you know that rspec is good to go. Use `subl .` to open the project. You should see a list of files on your left hand sidebar. Navigate to the `spec/robot_spec.rb` file. If you're lucky you should be able to hit `cmd-shift-t` to run all of the tests within the spec. If succesful you'll see a console window that looks like this.

{% img center /images/ruby_test_console.png Ruby Test Console %}

Don't worry if it doesn't work the first time. Sublime and RubyTest are *very* finicky. Close Sublime and try to open it again from the command line.It might help to close any other Sublime projects you already have open or even quit the program entirely using `cmd-q`. You'll have to experiment a bit to get it all working.

Hopefully after all that you've got your tests showing up green. Now you can integrate RubyTest into your workflow and save all that alt-tabbing back and forth between the command line!