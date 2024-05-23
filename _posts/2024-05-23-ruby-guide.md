---
toc: false
layout: post
description: Easy steps to fix a lot of weird ruby and gems errors on M1 macs
categories: [fixes, week_16]
title: Ruby Fixes for M1 macs
hide: false
permalink: /rubyFixes
---

# Introduction 
Essentially the issue to my understanding is mac has a base ruby version which isn't compatiable for our uses. This isn't bad, but the problem arises when you want to switch to a different version of ruby. You need to change the path which your system uses for ruby. The way I did this was using Chruby. Chruby is a ruby manager which allows you to change which version of ruby you want. It's kind of overkill for our purposes, but it was the only thing I could get to work. 

There are multiple prominent ruby managers and all of them will probably work. This is how to set up Chruby, but if you would like to look on your own for rvm or rbenv or something else entirely then you definitely can. Chruby is the simplest to use and understand. 

# What do you do

## Step 1: Install brew and upgrade brew
First you need to install brew. Homebrew (brew) is a package manager which basically means it helps you install anything and everything that you need. If you don't have it installed go to this [link](https://brew.sh/) and run the command under install. 

If you have brew then you need to make sure your brew is up to date. Run the following:
```shell
$ brew outdated #This command checks if your packages are outdated. If any of your packages are outdated, run the next two commands.
$ brew update 
$ brew upgrade #These upgrade all of your brew dependencies
```
Restart your terminal (close and reopen)
## Step 2: Install ruby manager and installer
Now you can actually install the ruby manager and installer. Run the follwing:
```shell
$ brew install chruby ruby-install
```

## Step 3: Install ruby
Install whatever version of ruby you want. For this project you should use 2.7.7. Run the following:
```shell
$ ruby-install 2.7.7
```

If it fails with it could be because:
- Something is wrong with your development setup
- You tried to install Ruby even though Homebrew reported warnings/errors

Read the error message and try to fix it. If you can't send a message on slack and mention Toby and I can look at it. 

## Step 4: cd into your directory
You should know how to do this. cd into whatever directory has the local server you want to run. This will probably be the repository that you copied from Mr. Mortensen.


## Step 5: Configure your shell
My understanding of this is right now your computer wants to use the built in ruby version so we need to change it so that it uses the correct path. to do that run these three commands (don't worry when it doesn't say anything after you run the command, nothing is supposed to happen in terminal):
```shell
$ echo ". $(brew --prefix)/opt/chruby/share/chruby/chruby.sh" >> ~/.zshrc
$ echo ". $(brew --prefix)/opt/chruby/share/chruby/auto.sh" >> ~/.zshrc
$ echo "chruby ruby-2.7.7" >> ~/.zshrc
```

If you get an error saying that the file path can't be found then restart from upgrading your homebrew. If you get any other error you could also try that or again put it on slack.

## Step 6: Check if it worked
You should be using the correct ruby version now. Inside of the directory that you used cd to enter earlier run:
```shell
$ ruby -v
```
If it says 2.7.7 then it worked and you can move on. 

### If it doesn't say 2.7.7 run:
```shell
$ chruby 2.7.7
```

### If that says chruby not found or something along those lines then run:
```shell
$ echo "source ~/.bashrc" >> ~/.bash_profile
```

Then restart the terminal and run:
```shell
$ chruby 2.7.7 
```

If the error persists run:
```shell
$ . /usr/local/opt/chruby/share/chruby/chruby.sh
$ . ~/.bashrc
```

Then restart the terminal and run:
```shell
$ chruby 2.7.7 
```

If you get a different error (NOT the chruby not found error) or the error still doesn't go away try restarting from the beginning or send it on slack. 

## Step 7: Check your Gemfile
Your gemfile is a file in the root of your project that you copied from Mr. Mortensen. Open it in vscode and look for it and check if it looks like this:
```gem
source "https://rubygems.org"

gem "github-pages", group: :jekyll_plugins
```
Delete anything extra that you have.
## Step 8: Install Jekyll if you haven't already
I didn't need to do this but some of you might. It doesn't hurt to run it anyway. Run:
```shell
$ gem insall jekyll
```
## Step 9: Install bundler if you haven't already
These instructions are also given on the bundler website. Run:
```shell
$ bundle install
$ git add Gemfile Gemfile.lock
```

## Step 10: Run your server
Now you should be ready to run your server. Run:
```shell
$ bundle exec jekyll serve
```
It will take a very long time the first time you run it. 

## Persisting errors
If you try to run your server and you still get any errors, the first thing you should do is check that you're using the correct ruby version in the correct directory (ruby -v) If you are then try all of these steps again from the beginning. If you still are getting errors send a message on slack and I can try to help you. 

In general however it is a good skill to be able to figure out some of the errors you get on your own. Read the error message, do what it says if there is a quick fix. Google error messages try to work through stuff if you think you'll be able to. 