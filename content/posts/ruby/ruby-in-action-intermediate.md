---
title: 'Ruby in action: intermediate'
date: 2022-02-05T15:04:15+08:00
categories:
- ruby
draft: false
---

## arrays

```ruby
require '../util'
# frozen_string_literal: true

############# arrays ###############
## array can hold different data type
data = ['Daniel', 'Michael', 10, [1, 2, 3, 4], 11.4, true]

# index start from zero, first element(s)
puts "first item: #{data[0]}"
puts "first item: #{data.first}"
puts "first 2 items: #{data.first(2)}"

# last element(s)
puts "last element: #{data[data.length - 1]}"
puts "last element: #{data.last}"
puts "last 2 elements: #{data.last(2)}"

# multiple array
puts "multiple array item: #{data[3][0]}"

puts "the length of the array is #{data.length}"

## output
# first item: Daniel
# first item: Daniel
# first 2 items: ["Daniel", "Michael"]
# last element: true
# last element: true
# last 2 elements: [11.4, true]
# multiple array item: 1
# the length of the array is 6

### bracket and fetch
print_delimiter('get element in the array')
## negative index counts from the end
puts "last item: #{data[-1]}"
## just return empty, no error raised
puts "out of index returns empty: #{data[data.length]}"
puts "last item by fetch: #{data.fetch(-1)}"
## if you uncomment the below, error will raise: in `fetch': index 6 outside of array bounds: -6...6
# puts data.fetch(data.length)
## the second is the default value if the index is invalid
puts "fetch use default value: #{data.fetch(data.length, 100)}"
## the block will be executed if the index is invalid
puts "fetch use block: #{data.fetch(data.length) { |i| "invalid index #{i}" }}"

## output
# last item: true
# out of index returns empty:
# last item by fetch: true
# fetch use default value: 100
# fetch use block: invalid index 6

### iterate array elements, each is the preferred one.
print_delimiter('iterate array elements')
arr = ['hello', 'world', 'peeps']
arr.each { |i| print i }
puts
for i in arr
  print i
end
puts

### other useful methods: include?, empty?, append, prepend, push, pop, filter, join, split
print_delimiter('other useful methods')

x = (1..5).to_a
puts "is empty?: #{x.empty?}"
puts "include?: #{x.include?(6)}"
x.append(5, 6, 7)
puts "uniq elements: #{x.uniq}"
x.prepend(8)
x.push(9)
y = x.pop
puts "push and pop: #{x}, #{y}"
puts "filter even: #{x.filter { |e| e.even? }}"
puts "filter odd: #{x.filter(&:odd?)}"
## filter is an alias of select
puts "select even: #{x.select(&:even?)}"
z = x.join('-')
puts "after join: #{z}"
puts "split: #{z.split('-')}"

## output
# is empty?: false
# include?: false
# uniq elements: [1, 2, 3, 4, 5, 6, 7]
# push and pop: [8, 1, 2, 3, 4, 5, 5, 6, 7], 9
# filter even: [8, 2, 4, 6]
# filter odd: [1, 3, 5, 5, 7]
# select even: [8, 2, 4, 6]
# after join: 8-1-2-3-4-5-5-6-7
# split: ["8", "1", "2", "3", "4", "5", "5", "6", "7"]
```

## loops

```ruby
# frozen_string_literal: true
require '../util'
############# while loops ###############
print_delimiter('while loop')
index = 1
while index < 5
  puts "index is #{index}"
  index += 1
end

## output
# index is 1
# index is 2
# index is 3
# index is 4

############# each loops ###############
## if there is only one line in the loop body, prefer {}
print_delimiter('each loop')
(1..5).each { |i| puts "index is #{i}" }

## output
# index is 1
# index is 2
# index is 3
# index is 4
# index is 5

## for multi-line body, prefer do...end
names = ['Daniel', 'Michael', 'Colin']
# names = %w[Daniel Michael Colin]
names.each do |name|
  puts "The name is #{name}"
  puts "Welcome #{name}"
end

## output
# The name is Daniel
# Welcome Daniel
# The name is Michael
# Welcome Michael
# The name is Colin
# Welcome Colin

############# loop ###############
# `loop` is equivalent to `while true`
print_delimiter('loop keyword')
loop_index = 1
max_loop = 5
loop do
  puts "current loop index: #{loop_index}"
  loop_index += 1
  break if loop_index == max_loop
end
```

## hashes

```ruby
# frozen_string_literal: true

require '../util'

############# hashes ###############
## hash can hold any data type, return nil if the key not exist in the hash
print_delimiter('access hash elements')
dict = { 'one' => 1, 2 => 'two', 'three' => [1, 2, 3], 4 => true, 'five' => { 'hello' => 'world' } }
puts "the hash is: #{dict}"
puts "the value of an element: #{dict['one']}"
puts "the value of an element: #{dict.fetch(2)}"
puts "the value of not exist key: #{dict['no-exist']}"

## output
# the hash is: {"one"=>1, 2=>"two", "three"=>[1, 2, 3], 4=>true, "five"=>{"hello"=>"world"}}
# the value of an element: 1
# the value of an element: two
# the value of not exist key:

### symbols as hash keys
print_delimiter('symbols as hash keys')
symbol_hash = { name: 'Jonathon', country: 'AU', 'age': 20 }
puts "value is: #{symbol_hash[:name]}"
symbol_hash.each { |k, _| puts "type of key: #{k.class}" }
symbol_hash.each { |k, v| puts v if k.is_a?(Symbol) }

## output
# value is: Jonathon
# type of key: Symbol
# type of key: Symbol
# type of key: Symbol
# Jonathon
# AU
# 20

### useful methods of hash
# some methods are equivalent, prefer key? over has_key?, prefer value? over has_value?
print_delimiter('useful methods')
dict2 = { 'Michael' => 'AU', 'Daniel' => 'US' }
puts "keys: #{dict2.keys}"
puts "values: #{dict2.values}"
puts "length: #{dict2.length}"
puts "has value?: #{dict2.value?('AU')}"
puts "has value?: #{dict2.has_value?('AU')}"
puts "has key?: #{dict2.key?('Anderson')}"
puts "has key?: #{dict2.has_key?('Daniel')}"
puts "has key?: #{dict2.include?('Daniel')}"

## output
# keys: ["Michael", "Daniel"]
# values: ["AU", "US"]
# length: 2
# has value?: true
# has value?: true
# has key?: false
# has key?: true
# has key?: true

### each, select
print_delimiter('each, select')
dict3 = { 'name': 'Michael', 'age': 18, 'favorite_sport': 'basketball' }
dict3.each { |k, v| puts "key is #{k}, value is #{v}" }
## if an argument is unnecessary, use _ or prepend _ to the argument
dict_by_select = dict3.select { |_, v| v.is_a?(Integer) }
dict_by_select2 = dict3.select { |_k, v| v.is_a?(Integer) }
puts "dict_by_select: #{dict_by_select}"
puts "dict_by_select2: #{dict_by_select2}"

## output
# key is name, value is Michael
# key is age, value is 18
# key is favorite_sport, value is basketball
# dict_by_select: {:age=>18}
# dict_by_select2: {:age=>18}
```

## methods

```ruby
############# methods ###############
## a method must be first defined before it can be referenced.
## if the method doesn't have parameters, the parentheses are redundant.
def say_hi
  puts 'Hi!'
end

say_hi

## output
# Hi!

## ruby methods will return the result of the last statement, so the `return` keyword is redundant.
## the parameters must match exactly
def hello_message(first_name, last_name)
  "Hello, #{first_name}, #{last_name}"
end

message = hello_message('Michael', 'Ken')
same_message = hello_message 'Michael', 'Ken'
puts message
puts same_message

## output
# Hello, Michael, Ken

## special methods: class, methods
puts 'Hello'.class
puts 10.class
puts 10.0.class
p 'Hello'.methods

## output
# String
# Integer
# Float
# [:unicode_normalize, :unicode_normalize!, :ascii_only? ...
```

### References

- [ruby-for-beginners](http://ruby-for-beginners.rubymonstas.org/objects/classes.html)
- [The Complete Ruby on Rails Developer Course](https://www.udemy.com/course/the-complete-ruby-on-rails-developer-course/)
