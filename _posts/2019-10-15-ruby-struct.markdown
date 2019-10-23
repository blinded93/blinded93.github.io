---
layout: post
title: "Ruby Struct"
date:       2019-10-15 22:31:45 -0400
permalink: ruby-struct
---

_**Struct**: a conveient way to bundle a number of attributes together using accessor methods, without having to write an explicitly class_.

As my previous post mentioned the use of Structs, I thought it to be an excelent follow-up topic. Structs offer a concise, perfomant and easy to understand syntax as an alternative data structure.

#### Creation of Structs

There are several ways to create structs in Ruby. The first is by including a name as the first argument to it's #new method. The symbols after are the structure's attributes.

```ruby
  Structs.new('Person', :name, :age)
```

This method creates the structure within the scope of the struct class, adding it's name to the list of constants under Struct. Referencing is done like so:

```ruby
  Struct::Person.new
```

The second way is to assign directly to a constant:

```ruby
  Person = Struct.new(:name, :age)
```

In this way, we can use it just as we would a class name:
```ruby
  Person.new
```

The last way,  and how I was introduced to Structs, is by creating a normal class and inheriting from Struct.new:

```ruby
  class Person < Struct.new(:name, :age)
  end
```
Although this does make for a very concise class definition, there are side effects to be aware of.

1. Arguments are no longer required. Without passing the proper number of arguments, the remaining attributes will be assigned a 'nil' value. If any of the attributes are meant to be an object and are not assigned correctly, calling a method on this member will cause an error.

2. Attributes are always public. Without the encapsulation of classes, these attributes are treated more like a hash.

3. Given the same attributes, two instances of the class will be equal. Object ID is no longer how equality is tested.

4. When Structs are subclassed, they leave an unused annonymous class, this pattern is suggested against in the documentation.

When we call Struct.new, we are not creating an instance of the Struct class. The method defined on classes used to set aside memory for new objects, #allocate, is not a part of Struct, so no objects can be created. Instead, the inherited #new method is overridden by Struct, creating subclasses instead of objects. This allows both access to Struct methods and internals as well as including the mechanisms of classes, like class reopening, inheritance and mixins.

So creating a struct:

```ruby
  Person = Struct.new(:name, :age)
```

Gives the same basic behavior as a class.

```ruby
  class Person
    attr_accessor :name, :age
    
    def initialize(name, age)
      @name = name
      @age = age
    end
  end

```

We wont stop there, what if we need some kind of logic within this structure class? Methods can be defined in two ways, reopening the class:

```ruby
  class Person
    def introduce
      "My name is #{name} and I am #{age} years old."
    end
  end
```

Or if you would rather define them when creating the structure, it can be done through a block passed to Struct.new. It will be evaluated in the context of the struct class, passing the created class as a parameter:

```ruby
  Person = Struct.new(:name, :age) do
    def introduce
      "My name is #{name} and I am #{age} years old."
    end
  end
```

#### Accessing Attributes

Objects created from a struct class can access and update attributes in a number of ways. As structs utilize accessor methods similarly to class objects, they can be accessed using the dot notation of a method call.

```ruby
  person = Person.new('Bill', 30)
  person.name # "Bill"
  person.age # "34"
```

If you remember from earlier the 'things to be aware of when inheriting from Struct.new' section, all of the attributes are public. Instead of being encapsulated as in a class, they can be accessed like a hash.

```ruby
  person[:name] # "Bill"
  person['age'] # "34"
```

Even more, because the created structure class contains an immutable LIST of attributes, we can access them just as we would an array, in the order in which they were defined.

```ruby
  person[0] # "Bill"
  person[1] # "34"
```

#### Iteration

Because the struct class includes the enumerable module, the created objects have access to all of the iteration methods such as `#each` and `#each_pair`. These attributes are able to be iterated over like a hash or array.

```ruby
  person.each { | value| puts value }
  # "Bill"
  # "34"
```

#### Use Case

One very useful way of using a struct is inside of a class, holding internal data such as an address in our person class:

```ruby
  class Person
    attr_accessor :name, :age, :address

    Address = Struct.new(:street, :city, :state, :zip)

    def initialize(name, age, address)
      @name = name
      @age = age
      @address = Address.new(address[:street], address[:city], address[:state], address[:zip])
    end
  end

  bill = Person.new("Bill", 30, { street: '123 Happy Ln', city: 'Seattle', state: 'WA', zip: 98122 })
  # <Person: 0x0000... @name='Bill'......>
  bill.address
  # <struct Person::Address street='123 Happy Ln' city.....>
  bill.address.each { |v| puts v }
  # 123 Happy Ln
  # Seattle
  # WA
  # 98122
```
This is a much more descriptive way of assigning internal data. It is easy to see what is going on by looking at the class and easy to access and manipulate the data.

Lastly, in previous versions of Ruby, attributes had to be assigned in the order in which they were defined, called positional arguments. With the introduction of Ruby 2.5 came keyword arguments. Key/value pairs can be given in any order by adding 'keyword_init: true' as the last argument of Struct.new.

#### Conclusion

To summarize, creating a structure type or class is like creating a blueprint that contains an immutable list of attributes. Once this structure class is made, creating new objects from this and saving it's memory reference is akin to creating instances of that blueprint.