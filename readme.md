# Ruby OOP

### Objectives
*After this lesson, students will be able to:*
- Define classes in Ruby
- Be comfortable with instance variables
  - And exposing them with `attr_reader`, `attr_writer`, `attr_accessor`
- Be able to inherit from superclasses and define subclasses

### Preparation
*Before this lesson, students should:*
- Be comfortable with the very basics of ruby
- Understand the basic concepts of OOP and inheritance (JS class lecture)


## OOP

Ruby is an **object-orientated** language.  Everything in Ruby is an object.  In other words everything is an instance of the [`Object` class](https://ruby-doc.org/core-2.4.1/Object.html).

```ruby
2.is_a?(Object) # => true
nil.is_a?(Object) # => true
true.is_a?(Object) # => true
''.is_a?(Object) # => true
:foo.is_a?(Object) # => true
[].is_a?(Object) # => true
{}.is_a?(Object) # => true
```

## Methods

Everything is an object and every object has methods.

```ruby
'some str'.methods
# => [..., :include?, :upcase, :downcase, :reverse, ...]
```

In Javascript everything is a property, where some properties happen to be functions.

In Ruby everything is a method.  When you call `thing.foo` you are calling a method called `foo` on `thing`.


## Classes

Every object is an instance of some [`Class`](https://ruby-doc.org/core-2.2.0/Class.html)

The `Array` class, for example, has methods `length`, `map`, `each`, etc.  This means we can call these methods on any instance of `Array`

```ruby
arr = ['Stacey', 'Lacey', 'Tracey']
arr.length # => 3
arr.map { |name| name.upcase }
```

> Check out all the `Array` methods by calling `Array.instance_methods` or `[].methods` (or looking at the docs).  Also check out `Enumerable.instance_methods`

## Defining a Class

```ruby
class Person
  def initialize(name, age)
    @name = name
    @age = age
  end

  def say_hi
    puts "Hi, name is #{@name} and I am #{@age} years old!"
  end
end

stacey = Person.new('Stacey', 25)
lacey = Person.new('Lacey', 26)
```

A few things here:

* We always use the `new` method to create a new instance.
* `initialize` is what gets called when creating the new instance.
  * We **never** call `initialize` in our code. We just define it
* We use the `@` sign to create _instance variables_ (aka _ivars_)
  * Instance variables can be referenced from any method inside of the class
  * Instance variables can **only** be referenced from methods inside of the class

```ruby
stacey.name
# => NoMethodError: undefined method `name' for Person
```

### `attr_reader`

If we want `name` to be accessible outside of the class we can simply create a method

```ruby
class Person
  # ...

  def name
    @name
  end
end

# ...

stacey.name # => "Stacey"
```

If we also want `age` to be accessible:

```ruby
class Person
  # ...

  def age
    @age
  end
end
```

Because this pattern is so common there is a very nice method, `attr_reader`, which we can use as a short-hand.

```ruby
class Person
  # ...
  def age
    @age
  end

  def name
    @name
  end
  # ...
```

Is the same as:

```ruby
class Person
  # ...
  attr_reader :age, :name
  # ...
end
```

Neat.

### `attr_writer`

Let's see if we can set the name

```ruby
stacey.name = 'Stacey Mae'
# => NoMethodError: undefined method `name='
```

To fix this, we can write a method:

```ruby
class Person
  def name=(new_name)
    @name = new_name
  end
# ...
```

Now we can do

```ruby
stacey.name = 'Stacey Mae'
stacey.name # => 'Stacey Mae'
```

Instead of having to define the method, we can just use `attr_writer`

```ruby
class Person
  attr_writer :name
# ...
```

And if we also wanted a writer for `age`

```ruby
class Person
  attr_writer :name, :age
# ...
```

To use our writer within the class:

```ruby
class Person
  # ...

  def have_birthday
    puts 'happy birthday'
    @age = @age + 1
  end
```

If we just did `age = age + 1` that would be setting a local variable.  We need to actually call our `age=` method to change the instance variable.

### `attr_accessor`

Now we have

```ruby
class Person
  attr_reader :name, :age
  attr_writer :name, :age

  # ...
```

We can merge together the first two lines with `attr_accessor`

```ruby
class Person
  attr_accessor :name, :age

  # ...
```

That one line creates 4 instance methods: `name`, `name=`, `age`, and `age=`.


## `private` methods

Every method by default is `public`. This means we can call it from in our outside our class.

`private` methods can only be called from inside the class.  They are usually helper methods or methods for implementation.

```ruby
class Person
  # ...

  def have_birthday
    puts 'happy birthday'
    increment_age
  end

  # ...

  private

  def increment_age
    @age = age + 1
  end
```

All we need is the `private` keyword.  Any method defined after that will be `private`.

```ruby
person.increment_age
# => NoMethodError: private method `increment_age' called for Person
```

This error is a good thing!  Our `increment_age` method is just used for implementation.  We do not want it to be called outside of the class.

## Inheritance

Cake!

```ruby
class Programmer < Person
end
```
That's it! We don't even need to write an `initialize` if it does all the same setup as `Person`!

Let's confirm a few things:

```ruby
person = Person.new('Stacey', 25)
programmer = Programmer.new('Tracey', 26)

person.is_a?(Person) # => true
person.is_a?(Programmer) # => false

programmer.is_a?(Person) # => true
programmer.is_a?(Programmer) # => true
```

### Re-defining initialize

We can do more work in `initialize`

```ruby
class Programmer < Person
  def initialize(name, age, options)
    super(name, age)
    @github = options[:github]
    @website = options[:website]
    @languages = options[:languages]
  end
end
```


### Adding new methods

Since `Programmer` inherits from `Person` we already have an `attr_accessor`s for `name` and `age`.  We can also add readers, writers, or accessors for other ivars too.

For example, if we want to be able to read and write `github` and `website` we would:

```ruby
class Programmer < Person
  attr_accessor :github, :website
  attr_reader :languages

  # ...
  private

  attr_writer :languages
```

And if we want to be able to publicly read `languages` but only privately write, we would:

```ruby
class Programmer < Person
  # ...
  attr_reader :languages

  # ...
  private

  attr_writer :languages
```

(This is only adding methods, it does not override or take away any methods)

We can add any new methods

```ruby
# ...
def build_resume
  "#{@name}, github: #{@github}, website: #{@website}, languages: #{@languages.join(', ')}"
end
# ...
```

`Programmer` now has a method `Programmer.build_resume`. (And we are not creating a `Person.build_resume`)


### Overriding methods

Of course we can also override methods

```ruby
# ...
  def say_hi
    super
    puts 'And I love to program!'
  end
# ...
```

We are using `super` to call `Person.say_hi`.  Notice we do not need parens if the superclass's method takes no args!

### More `private` methods

Let's refactor our `Programmer` using a `private` method

```ruby
class Programmer < Person
  # ...
  def initialize(name, age, options)
    super(name, age)
    setup(options)
  end
  # ... more public methods ...

  private

  def setup(options)
    @github = options[:github]
    @website = options[:website]
    @language = options[:languages]
  end

  # ... other private methods
```

This is nicer because we want to keep our `initialize` method small and readable.

## Class Methods

Like `static` methods in JS

```ruby
class Person
  # ...
  def self.about
    puts 'People are people. We are all human'
  end
  # ...
```

`self` works very similarly to `this` in javascript. It refers directly to its parent Class.

We call these methods the way you would expect

```ruby
Person.about # => 'People are people. We are all human'
Programmer.about # => 'People are people. We are all human'
person.about # => NoMethodError: undefined method `about' for Person
programmer.about # => NoMethodError: undefined method `about' for Person
```

## Resources

* [`Class` tutorial](http://ruby-for-beginners.rubymonstas.org/writing_classes/definition.html)
  - Almost everything in here is good. Click around to learn more than just classes!
* [Ruby in 20 Minutes](https://www.ruby-lang.org/en/documentation/quickstart/2/)

## Conclusion
- How are Ruby classes different from Javascript "classes"
- What are ivars used for? How do expose them outside the class?
- How do we create new instances of a class? What methods are called when we create them?
