---
layout: post
title: "Ruby OpenStruct"
excerpt: "OpenStruct: a datastructure, simliar to a hash, that allows the definition of arbitrary attributes with their accompanying values."
date:       2019-10-22 23:05:45 -0400
permalink: ruby-ostruct
---

_**OpenStruct**: a data structure, similar to a hash, that allows the definition of arbitrary attributes with their accompanying values. This is accomplished using Ruby's metaprogramming to define methods on the class itself_.

OpenStruct is part of the Ruby stdlib, included with a "require 'ostruct'"

Unlike structs that create subclasses of themselves, OpenStructs create instances.

```ruby
  person = OpenStruct.new(name:'Bill', age: 30)

  # <OpenStruct name="Bill", age=30>
```

#### Creation of OpenStructs

Internally, the openstruct uses a hash to store attributes and values. Using key/value pairs, as in the example above, is the most common way to assign attributes in it's creation. The syntax used is only missing the {} of the hash.

OpenStructs can also be created from a Struct:

```ruby
  Person = Struct.new(:name, :age)
  bill = Person.new('Bill', 30)

  OpenStruct.new(bill)

  # <OpenStruct name="Bill", age=30>
```

or from another OpenStruct:

```ruby
  bill1 = OpenStruct.new(name:'Bill', age:30)
  bill2 = OpenStruct.new(bill1)

  # <OpenStruct name="Bill", age=30>
```

#### Accessing and Creating Attributes

In similar fashion to Structs, attributes can be both accessed and assigned in a few ways.

The following uses the standard '.' notation of an accessor method call.

```ruby
  person.name # "Bill"
  person.age # 30

  person.name = 'Ben' # "Ben"
  person.age = 50 # 50
```

The hash lookup using symbols or strings can also be used.

```ruby
  person['name'] # "Bill"
  person[:age] # 30

  person[:name] = 'Ben' # "Ben"
  person['age'] = 50 # 50
```

Using the string hash lookup makes it possible to use keys you would not be able to use as method names (that include spaces, other symbols, etc).

```ruby
  person['This is my name'] = 'Bogart' # "Bogart"
  
  person
  #<OpenStruct This is my name="Bogart">
```

The last way to create/access an attribute is by calling the #send method.

```ruby
  person.send('name') # "Bill"

  person.send('age', 50) # 50

  person.send('weight', 165) # 165

  person

  #<OpenStruct name="Bill", age=50, weight=165>
```

##### When the attributes don't exist

What happens if an attribute doesn't exist? Well, if you're simply trying to access the value, you will get 'nil' just as you would using a hash. If you're trying to set a value that doesn't exist, though, Ruby will call OpenStruct#method_missing which will search it's own hash table for the attribute name. If it doesn't find it, it will add it to the table and call #define_singleton_method on the class, which adds setter/getter methods to the instance of OpenStruct.

Lazy loading is employed to save memory and speed up the access to the attribute. Accessor methods are not defined until they are needed.

```ruby
  person = OpenStruct.new(name: 'Bill', age: 30)

  person.methods(false) # []
  person['name'] # "Bill"
  person.methods(false) # []
  
  person['weight'] = 165
  person.methods(false) # [:weight=, :weight]
  
  person.name # "Bill"

  person.methods(false) # [:name=, :name, :weight=, :weight]
  
```

After creation, the methods are not defined until they are called directly using the accessor methods or a new attribute is created. This process is what gives OpenStruct the overhead, making is significantly slower than other data structures such as Struct or Hash.


#### Deleting Attributes

The only way to completely remove an attribute from the data structure is by calling the #delete_field method.

```ruby
  person = OpenStruct.new(name: 'Bill', age: 30, legs: 4)

  person.legs = nil
  # nil

  person
  #<OpenStruct name="Bill", age=30, legs=nil>

  person.delete_field(:legs)
  # nil

  person
  #<OpenStruct name="Bill", age=30>
```

As in the example, setting an attribute to nil does nothing but change it's value.

#### Equality

Unlike classes, OpenStructs use the attributes or hash table to decide equality. If an object is both an openstruct and has the same key/values, it is considered equal, even though the objectID's are different.

```ruby
  person1 = OpenStruct.new(name:'Bill') # <OpenStruct name="Bill">
  person2 = OpenStruct.new(name:'Bill') # <OpenStruct name="Bill">

  person1.object_id == person2.object_id # false

  person1 == person2 # true
```

#### Iteration

OpenStruct does not include the Enumerable module like Struct, but #each_pair is defined to iterate over it's attributes/values.

```ruby
  person = OpenStruct.new(name:'Bill', age:30)

  person.each_pair do |key, value|
    puts ":#{key} => #{value}"
  end

  # :name => Bill
  # :age => 30

  #<OpenStruct name="Bill", age=30>
```

#### Conclusion

Although I do find this class to have a lot of interesting and useful features, I'm not sure I would use it over a hash. Especially in Ruby 2.6, the hash is now very fast, even quicker than the Struct! I'm sure it has it's uses and I'm glad I to learn about it in case that need arises in the future.