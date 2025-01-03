---
layout: post
title: Problems with If Clauses
date: 2024-12-05 16:51 +0300
categories: [Development]
tags: [ruby]
image:
  path: /7exynu9pk/IMG_4750.JPG
---
Hello 👋

When I started learning Ruby programming, I loved using if statements because they were easy to understand and could be used everywhere. However, as I built bigger projects, I noticed a problem: my code became messy and hard to understand. I had too many nested if statements, and my logic was scattered all over the place.

I learned an important lesson: writing good if statements isn't just about getting the code to work. It's about writing code that other people (and future me) can easily read and update. I found better ways to organize my code by using techniques like guard clauses and breaking big problems into smaller pieces. Sometimes, I even redesigned my code using object-oriented programming principles.

These improvements helped me write cleaner, better-organized code. Looking back, learning to write better if statements has been one of the most valuable skills I've gained as a Ruby programmer.

## Problems with Overusing if Clauses:
### 1. Complexity:
- Nested `if` statements create hard-to-read, hard-to-maintain code.
- Each new `if` statement creates another decision branch, making code more complex.

### 2. Duplication:
- The same conditions often appear in multiple places, creating unnecessary redundancy.

### 3. Single Responsibility Principle Violation:
- Embedding business logic in conditionals often mixes different responsibilities.

### 4. Testing Challenges:
- Code with many branches requires extensive testing to cover all scenarios.

```ruby
def process_order(order)
  if order.status == "pending"
    if order.total > 100
      puts "Large order pending!"
    else
      puts "Regular order pending."
    end
  elsif order.status == "completed"
    if order.shipped
      puts "Order already shipped."
    else
      puts "Completed but not shipped yet."
    end
  else
    puts "Unknown status."
  end
end
```
This code is hard to follow, with nested if clauses and repeated logic.

## Best Practices to Handle if Clauses:

### 1. Use Guard Clauses:
Eliminate unnecessary nesting by returning early.

```ruby
def process_order(order)
  return puts "Unknown status." unless %w[pending completed].include?(order.status)

  if order.status == "pending"
    puts order.total > 100 ? "Large order pending!" : "Regular order pending."
  elsif order.status == "completed"
    puts order.shipped ? "Order already shipped." : "Completed but not shipped yet."
  end
end
```

### 2. Extract Logic into Methods:
Break down the logic into smaller, focused methods.

```ruby
def process_order(order)
  case order.status
  when "pending"
    handle_pending_order(order)
  when "completed"
    handle_completed_order(order)
  else
    puts "Unknown status."
  end
end

def handle_pending_order(order)
  puts order.total > 100 ? "Large order pending!" : "Regular order pending."
end

def handle_completed_order(order)
  puts order.shipped ? "Order already shipped." : "Completed but not shipped yet."
end
```

### 3. Use a Hash for Conditional Mapping:
Replace `if` or `case` logic with a hash for simple lookups.

```ruby
STATUS_MESSAGES = {
  "pending_large" => "Large order pending!",
  "pending_regular" => "Regular order pending.",
  "completed_shipped" => "Order already shipped.",
  "completed_not_shipped" => "Completed but not shipped yet."
}.freeze

def process_order(order)
  key = build_status_key(order)
  puts STATUS_MESSAGES[key] || "Unknown status."
end

def build_status_key(order)
  case order.status
  when "pending"
    order.total > 100 ? "pending_large" : "pending_regular"
  when "completed"
    order.shipped ? "completed_shipped" : "completed_not_shipped"
  end
end
```

### 4.Use Polymorphism:
Replace conditionals with object-oriented design when logic varies by type.

```ruby
class Order
  def process
    raise NotImplementedError
  end
end

class PendingOrder < Order
  def initialize(total)
    @total = total
  end

  def process
    puts @total > 100 ? "Large order pending!" : "Regular order pending."
  end
end

class CompletedOrder < Order
  def initialize(shipped)
    @shipped = shipped
  end

  def process
    puts @shipped ? "Order already shipped." : "Completed but not shipped yet."
  end
end

# Example Usage
order = PendingOrder.new(150)
order.process # => "Large order pending!"
```

## Conclusion:
While `if` clauses are essential in programming, their overuse can make code difficult to maintain. Here's how to write better code:

1. Implement guard clauses to keep logic simple and clear
2. Break down complex logic into smaller methods and leverage data structures like hashes
3. Apply object-oriented patterns such as polymorphism when handling complex conditional branches

By following these practices, you'll create Ruby code that's both cleaner and easier to maintain. 😊

❤️

