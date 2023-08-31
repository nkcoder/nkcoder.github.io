---
title: Configure Ruby development
date: 2021-08-28T20:49:23+08:00
categories:
- ruby
draft: false
---

## rbenv

```bash
# install rbenv, will also install ruby-build
$ brew install rbenv

# upgrade rbenv and ruby-build
$ brew upgrade rbenv ruby-build

# add `eval "$(rbenv init -)"` to your profile
$ echo 'eval "$(rbenv init -)"' >> ~/.zshrc

# also need to add rbenv to PATH, if you have separate homebrew
$ echo 'export PATH=~/homebrew/bin:$PATH' >> ~/.zshrc

# list latest stable versions
$ rbenv install -l

# list all local versions
$ rbenv install -L

# install a Ruby version
$ rbenv install 3.0.2

# set global version, will be set in ~/.rbenv/version
$ rbenv global 3.0.2

# set local version, or switch version
$ rbenv local system

# all versions install are located in `~/.rbenv/versions`, to remove a version, just remove the directory.
$ rm -rf ~/.rbenv/versions/3.0.2

# you can also remove a version by uninstall
$ rbenv uninstall 3.0.2
```

## bundler

```bash
# install Ruby gems
$ gem install bundler

# manage gem dependencies
$ bundler install

# check the location where gems are being installed
$ gem env
```

## Intellij Idea / RubyMine

Open a project from existing source and setup project SDK.

- File → New → Project from existing sources...
- Project Structure → Project → Project SDK: rbenv:3.0.2
- Module → Ruby SDK and Gems: rbenv:3.0.2

![intellij-project-structure-ruby](/images/ruby/intellij-project-structure-ruby.png)

![intellij-module-ruby](/images/ruby/intellij-module-ruby.png)

![ruby-mine-sdk-config](/images/ruby/ruby-mine-sdk-gems.png)