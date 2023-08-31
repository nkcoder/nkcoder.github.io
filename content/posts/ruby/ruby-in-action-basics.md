---
title: 'Ruby in action: basics'
date: 2022-02-04T22:37:14+08:00
categories:
- ruby
draft: false
---

## pring vs puts vs p

```ruby
## print: no new line at the end
## puts: new line at the end
## p: print the argument as it is and returns the argument
print 'Good morning.'
puts 'Hello World!'
puts 'Come on.'
p 'Hello World!'
p ['one', 'two', 3]

## output:
# Good morning.Hello World!
# Come on.
# "Hello World!"
# ["one", "two", 3]
```

## string

```ruby
## single quote vs double quote, single quote doesn't support interpolation
first_name = 'Michael'
last_name = 'Dent'
puts "first name is #{first_name}, last name is #{last_name}"
puts 'first name is #{first_name}, last name is #{last_name}'

## output
# first name is Michael, last name is Dent
# first name is #{first_name}, last name is #{last_name}

## the methods will return a new copy
hello = 'Hello World!'
puts "upcase: #{hello.upcase}"
puts "capitalize: #{hello.capitalize}"
puts "original: #{hello}"

## output:
# HELLO WORLD!
# Hello world!
# Hello World!

## string escape
p 'welcome, #{first_name}'
p 'welcome, \'how are you?\''

## output
# "welcome, \#{first_name}"
# "welcome, 'how are you?'"

## other string methods
puts 'hello'.reverse
puts 'hello'.empty?
puts ''.empty?
puts 'hello'.nil?
puts 'Welcome to the ruby world'.sub('the ruby', 'the Scala')

## output
# olleh
# false
# true
# false
# Welcome to the Scala world
```

## numbers

```ruby
############# math operators ###############
## +, -, *, /, **, %
puts 10 + 3
puts 10 - 3
puts 10 * 3
puts 10 / 3
puts 10 ** 3
puts 10 % 3
puts 10 / 3.0
puts 10 * 0.3

## output
# 13
# 7
# 30
# 3
# 1000
# 1
# 3.3333333333333335
# 3.0


############# number/string conversion ###############
## to_s, to_i, to_f
puts "#{30.to_s + 'abc'}"
puts '50'.to_i + 100
puts '50'.to_f * 2

## output
# 30abc
# 150
# 100.0

############# the `times` method of Integer ###############
## `times`: Iterates the given block int times, passing in values from zero to int - 1.
10.times do |i|
  print i, ' '
end

puts

10.times { |i| print i, ' '}

## output
# 0 1 2 3 4 5 6 7 8 9
# 0 1 2 3 4 5 6 7 8 9
```

## variables

```ruby
## convention is to use under_score(_) between words
first_name = 'Michael'
puts first_name

## output:
# Michael

## variables names can be reused, and re-assigned
house_number = 501
house_number = house_number + 3
puts house_number

## output:
# 504
```

## operators

```ruby
############# comparison operators ###############
## ==, !=, >, >=, <, <=
## ===: equivalent to ==, but no recommend
## eql?: compares value and type
puts 10 == 10
puts 10 != 9
puts 'Michael' == 'michael'
puts 10 == 10.to_f
puts 10 === 10.to_f
puts 10.eql?(10.to_f)

## output
# true
# true
# false
# true
# true
# false

puts '-' * 30

############# assignment operators ###############
## =, +=, -=, *=, /=, %=, **=
num = 10
puts num += 10
puts num -= 5
puts num *= 2
puts num /= 3
puts num %= 3
puts num **= 5

## output
# 20
# 15
# 30
# 10
# 1
# 1
```

## get user inputs

```ruby
############# get user input with gets ###############
## similar to puts, gets also has a new line at the end
## `chomp` method will remove the new line at the end
## you need to convert the input to your expected type using conversion methods like `to_i`
print "What's your name: "
your_name = gets
puts "Hello, #{your_name}, how are you?"
print "What's your name again: "
your_name2 = gets.chomp
puts "Hello, #{your_name2}, how are you?"
print 'Enter a number: '
your_number = gets.chomp.to_i
puts "your input multiply 2 is #{your_number * 2}"

## output
# What's your name: Michael
# Hello, Michael
# , how are you?
# What's your name again: Michael
# Hello, Michael, how are you?
# Enter a number: 10
# your input multiply 2 is 20
```

## conditional statements

```ruby
# frozen_string_literal: true
require '../util'
############# if/else conditional statements ###############
## if...elsif...else
print_delimiter('if..elsif..else')
guess = 100
if (guess % 3).zero?
  puts 'your guess is dividable by 3'
elsif (guess % 5).zero?
  puts 'your guess is dividable by 5'
else
  puts 'bad guess'
end

## output
# your guess is dividable by 5

### you can use &&, || to combine conditions
print_delimiter('&&, ||')
first_condition = true
second_condition = false

if first_condition && second_condition
  puts 'both are true'
elsif first_condition || second_condition
  puts 'at least one is true'
else
  puts 'both are false'
end

## output
# at least one is true

### if and unless
# if and unless are opposite
print_delimiter('if and unless')
(1..5).each do |num|
  puts 'num is even' if num.even?
  puts 'num is odd' unless num.even?
end

## output
# num is odd
# num is even
# num is odd
# num is even
# num is odd
```

### References

- [ruby-for-beginners](http://ruby-for-beginners.rubymonstas.org/objects/classes.html)
- [The Complete Ruby on Rails Developer Course](https://www.udemy.com/course/the-complete-ruby-on-rails-developer-course/)
