---
title: 'Ruby in action: advanced'
date: 2022-02-06T10:33:57+08:00
categories:
- ruby
draft: false
---

## objects

```ruby
## objects are instances of classes
puts 'hello'.class
puts 'hello'.is_a?(String)

## output
# String
# true
```

## class instances

```ruby
# frozen_string_literal: true

### A class is defined using the keyword class, a name, and the keyword end.
#
# Class names must start with an uppercase letter, and should use CamelCase. Variable and methods names should use snake_case.
# The method new is defined on every class, and returns a new instance of the class.
class Person

end

person = Person.new
puts person
puts person.is_a?(Person)

## output
# #<Person:0x00007fb5de100f18>
# true

### instance methods
# Methods that are available on classes are called class methods, for example,`Person.new`,
# and methods that are available on instances are called instance methods, for example, `person.is_a?`.
#
# Instance methods are defined inside the class body.
#
# `initialize` is a special instance method, which is called under the hood when the object is created using the class
# method `new`, and all the arguments that you pass to `new` will be passed to `initialize`.
#
# `self` is a keyword that means the object itself
class Dog
  def initialize(name);
  end

  def bark(times)
    'bark, ' * times
  end

  def bark_3_times
    bark = bark(3)
    puts bark
    puts self.bark(2)
  end
end

puts Dog.new('Kate').bark(3)
puts Dog.new('Helen').bark_3_times

## output
# bark, bark, bark,
# bark, bark, bark,
# bark, bark,
```

## class attributes

```ruby
############# class: getter/setter/to_s ###############
## method `initialize` will be called when you create a new instance of the class by `.new()`
## `@` means it is an instance variable
class Hat
  def initialize(color)
    @color = color
  end

  ## getter
  def color
    @color
  end

  ## setter, the convention is append `=` to the variable name
  def color=(new_color)
    @color = new_color
  end

  ## override the default to_s method
  def to_s
    "This is a hat and the color is: #{color}"
  end
end

hat = Hat.new('red')
puts hat
puts hat.inspect
puts hat.color
hat.color = 'green'
puts hat.color

## output
# This is a hat and the color is: red
# #<Hat:0x00007f9e0d0708e8 @color="red">
# red
# green

############# class: attr_accessor ###############
## attr_reader: only getter
## attr_writer: only setter
## attr_accessor: both getter and setter
class Hat2
  attr_accessor :color, :size

  def initialize(color, size)
    @color = color
    @size = size
  end

  def to_s
    "This is a hat and the color is: #{@color}, the size is #{size}"
  end
end

hat2 = Hat2.new('blue', 10)
puts hat2
puts hat2.to_s
puts hat2.color
puts hat2.size

## output
# This is a hat and the color is: blue, the size is 10
# This is a hat and the color is: blue, the size is 10
# blue
# 10
```

## require a library

```ruby
### we use the method `require` to load libraries manually.
#
# Normally require statements should be placed at the very top of the file, so it is easy to see what libraries a
# particular piece of code (class) uses.
require 'digest'
class RequireDemo
  def encrypt(password)
    Digest::SHA2.hexdigest(password)
  end
end

puts RequireDemo.new.encrypt("my-password")

## output
# 6fa2288c361becce3e30ba4c41be7d8ba01e3580566f7acc76a7f99994474c46
```

## modules

```ruby
### modules vs classes
# modules are similar to classes: can hold methods, but:
# modules don't have a `new` method, so they cannot be instantiated, which means we cannot create objects from a module.
# modules are used to share methods between classes: you can `include` modules into classes and this makes the methods of
# modules available to the class.
#
module Encryption
  require 'digest'

  def encrypt(password)
    Digest::SHA2.hexdigest(password)
  end
end

class Account
  include Encryption

  def initialize(name, password)
    @name = name
    @password = password
  end

  attr_reader :name, :password

  def to_s
    encrypted_password = encrypt(@password)
    "name is #{@name}, password is: #{encrypted_password}"
  end
end

account = Account.new('Jack', 'password1')
puts account

## output
# name is Jack, password is: 0b14d501a594442a01c6859541bcb3e8164d183d32937b851835442f69d5c94e
```

## private methods

```ruby
### The keyword `private` tells Ruby that all methods defind from now on are supposed to be private.
# Private methods can only be called inside the object.
#
class PrivateDemo
  private

  def greet
    'Hello, world!'
  end

  public

  def say_hi
    'Hi, how are you?'
  end

end

puts PrivateDemo.new.say_hi
puts PrivateDemo.new.greet

## output
# Hi, how are you?
# private method `greet' called for #<PrivateDemo:0x00007fa035844fb8> (NoMethodError)
```

### References

- [ruby-for-beginners](http://ruby-for-beginners.rubymonstas.org/objects/classes.html)
- [The Complete Ruby on Rails Developer Course]()

