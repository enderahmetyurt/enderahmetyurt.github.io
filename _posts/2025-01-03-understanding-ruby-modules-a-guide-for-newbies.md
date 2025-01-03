---
layout: post
title: "A Beginner's Guide to Ruby Modules"
date: 2025-01-03 15:49 +0300
categories: [Development]
tags: [ruby]
image:
  path: /7exynu9pk/IMG_7248 (1).jpg
---
Hello 👋

I have been writing Ruby since 2010. At the beginning of my Ruby adventure I didn't have no idea what modules in Ruby is. Ruby is a beginner-friendly programming language but after some time, you want to have more detailed information about this language. Modules are one of Ruby's best features. I didn't have this kind of guide when I was trying to learn Ruby but you will have. This guide will show you what modules are and how to use them.

## What Is a Ruby Module?
A module in Ruby is a collection of methods, constants, and other module or class definitions. Modules are similar to classes, but they cannot be instantiated or inherited from. Instead, modules are used to:

1. **Organize Code:** Group related methods and constants together.

2. **Mixin Functionality:** Add shared behavior to multiple classes.

3. **Avoid Name Collisions:** Create namespaces to prevent conflicts in larger projects.

## Define
Defining a module is straightforward. Use the module keyword followed by the module name (written in PascalCase):

```ruby
module Greeting
  def say_hello
    "Hello!"
  end
end
```

Here, we've defined a `Greeting` module with a `say_hello` method. But how do we use it?

## Using Modules as Mixins
Modules can be included in classes to add methods without duplication. This is called a **"mixin."**

```ruby
class Person
  include Greeting
end

person = Person.new
puts person.say_hello  # Outputs: "Hello!"
```

By including the `Greeting` module in the `Person` class, we make the `say_hello` method available to instances of `Person`.

If you want the methods to be available as class methods instead of instance methods, use `extend`:

```ruby
class Robot
  extend Greeting
end

puts Robot.say_hello  # Outputs: "Hello!"
```

## Creating Namespaces with Modules
Modules can be used to group related classes and methods to avoid naming conflicts.

```ruby
module Animals
  class Dog
    def speak
      "Woof!"
    end
  end

  class Cat
    def speak
      "Meow!"
    end
  end
end

# Access classes inside the module
dog = Animals::Dog.new
puts dog.speak  # Outputs: "Woof!"
```

Using the `Animals::Dog` syntax ensures clarity and prevents conflicts with other `Dog` classes in the codebase.

## Including vs. Extending
It's essential to understand the difference between `include` and `extend`:

- `include`: Adds module methods as instance methods to the class.
- `extend`: Adds module methods as class methods.

You can combine both in a single module using `self.included`:

```ruby
module Utility
  def self.included(base)
    base.extend(ClassMethods)
  end

  def instance_method
    "I'm an instance method."
  end

  module ClassMethods
    def class_method
      "I'm a class method."
    end
  end
end

class Tool
  include Utility
end

tool = Tool.new
puts tool.instance_method       # Outputs: "I'm an instance method."
puts Tool.class_method          # Outputs: "I'm a class method."
```

## Real World Example: Logging Module
Okay! Everything looks fine until here but how can we use it in real life. Let's suppose that you want to add logging functionality to multiple classes.

```ruby
module Logger
  def log(message)
    puts "[LOG] #{message}"
  end
end

class Order
  include Logger

  def process
    log("Processing order...")
  end
end

class Payment
  include Logger

  def charge
    log("Charging payment...")
  end
end

order = Order.new
order.process  # Outputs: [LOG] Processing order...

payment = Payment.new
payment.charge  # Outputs: [LOG] Charging payment...
```

## Conclusion
Modules are helpful tools in Ruby. It lets you organize your code better. They make it easier to reuse code and keep it organized. When you learn to use modules well, your Ruby programs will be easier to read and update. Try using modules in your code - you'll see how useful they can be. Good luck with your coding!

❤️