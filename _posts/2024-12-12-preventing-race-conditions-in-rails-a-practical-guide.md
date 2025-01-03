---
layout: post
title: 'Preventing Race Conditions in Rails: A Practical Guide'
date: 2024-12-12 17:15 +0300
categories: [Development]
tags: [ruby on rails]
image:
  path: /7exynu9pk/IMG_4577.JPG
---
Hello 👋

A race condition happens when two or more parts of a program try to change the same information at the same time. Think of it like two people trying to edit the same document simultaneously - it can lead to mistakes and unexpected results. This often happens in programs that run multiple tasks at once, where you can't predict which task will finish first.

## Common Scenarios of Race Conditions in Rails

### 1. Updating the Same Record
When multiple requests try to update the same database record at the same time, the last operation may overwrite the others without accounting for changes made by other operations.

```ruby
# Two requests fetching and updating the same record
user = User.find(1)
user.balance += 100
user.save
```

If another process modifies the `balance` simultaneously, the final value may be incorrect.

### 2. Concurrent Record Creation
If two or more processes attempt to create records with the same unique constraints (e.g., email or username), it might lead to duplication or errors.

```ruby
# Concurrent requests creating users with the same email
User.create(email: "test@example.com")
```

### 3. Background Jobs and Web Requests
A background job and a web request might access and modify the same resource, causing inconsistencies.

## How to Prevent Race Conditions in Rails

### 1. Database Transactions
Use ActiveRecord transactions to group related operations into an atomic unit. If any operation in the transaction fails, the entire transaction is rolled back.

```ruby
ActiveRecord::Base.transaction do
  user = User.find(1)
  user.balance += 100
  user.save!
end
```

### 2. Database-Level Locks
- Use database locks to ensure only one process modifies a resource at a time.
- **Optimistic Locking:** Rails provides built-in support for optimistic locking using the `lock_version` column. This approach checks if the record was modified by another process before saving.

```ruby
class User < ApplicationRecord
  # Add lock_version column in the table
end

user = User.find(1)
user.balance += 100
user.save! # Fails if lock_version has changed
```

- **Pessimistic Locking:** Locks the database row to prevent other processes from modifying it until the lock is released.

```ruby
user = User.lock.find(1)
user.balance += 100
user.save!
```

### 3. Uniqueness Constraints
Add database-level constraints (e.g., unique indexes) to prevent duplicate records.

```ruby
add_index :users, :email, unique: true
```

### 4. Retry Logic
Implement retries for operations that may fail due to race conditions.

```ruby
begin
  user = User.find(1)
  user.balance += 100
  user.save!
rescue ActiveRecord::StaleObjectError
  retry
end
```

### 5. Background Job Idempotency
Ensure background jobs are idempotent so that running the job multiple times produces the same result.

### 6. Custom Locking Mechanisms
Use Redis locks (e.g., with the `redlock` gem) or similar mechanisms for distributed systems.

```ruby
require 'redlock'

redlock = Redlock::Client.new([redis_url])
redlock.lock("user:#{user_id}:lock", 2000) do
  user = User.find(user_id)
  user.balance += 100
  user.save!
end
```

## Tools for Detecting Race Conditions
- **Testing**: Write tests that simulate concurrent requests to uncover race conditions.
- **Concurrency Gems**: Use tools like `rspec-racecar` or `parallel_tests` for testing concurrency.
- **Database Logging**: Enable detailed database query logs to identify potential race conditions.

By understanding the scenarios where race conditions can occur and implementing appropriate safeguards, you can build more robust and reliable Rails applications.

❤️