---
layout: post
title: Adding SessionTestHelper to Existing Rails Project
date: 2025-03-05 15:51 +0300
categories: [Development]
tags: [rails,test]
image:
  path: https://images.unsplash.com/photo-1517581177682-a085bb7ffb15?q=80&w=1172&auto=format&fit=crop&ixlib=rb-4.0.3&ixid=M3wxMjA3fDB8MHxwaG90by1wYWdlfHx8fGVufDB8fHx8fA%3D%3D
---
Hello 👋

With Rails 8, built-in authentication has been introduced. Although it seems like a small and simple step, for a developer like me who uses Devise in every project, even for basic authentication, it is a significant initiative. I am not going to talk about the Rails Authentication Generator. Instead, I'd like to focus on a small update related to testing in it.

I was very happy to use a small authentication generator for my project ([bloud.me](https://bloud.me)), but at that time, the documentation was not sufficient. I found [a blog post](https://blog.corsego.com/rails-8-authentication) about adding Rails Authentication to my tests.

I had a small method to authenticate a user, which I used in my tests whenever I needed to sign in a user.

```ruby
# test/test_helper.rb
def sign_in(user, password: "password")
  post session_url, params: { email_address: user.email_address, password: }
end
```

```ruby
class HomeControllerTest < ActionDispatch::IntegrationTest
  ....
  sign_in(user, password: "password")
  ...
end
```

There was a problem—I couldn't run system tests. I needed another solution, but I couldn't find or create one. So, I ended up removing all system tests. To be honest, I don’t really enjoy writing system tests, but that’s a topic for another blog post 😬.

One week ago, [a PR](https://github.com/rails/rails/pull/53708/files) was merged into the main branch of the Rails project. I am using Rails main in my project and always keep it up to date. So, I decided to update my tests according to that PR.

First of all, I needed to add `SessionTestHelper` to the `test_helpers` folder.

```ruby
# test/test_helpers/session_test_helper.rb
module SessionTestHelper
  def sign_in_as(user)
    Current.session = user.sessions.create!

    ActionDispatch::TestRequest.create.cookie_jar.tap do |cookie_jar|
      cookie_jar.signed[:session_id] = Current.session.id
      cookies[:session_id] = cookie_jar[:session_id]
    end
  end

  def sign_out
    Current.session&.destroy!
    cookies.delete(:session_id)
  end
end
```

Update `test_helper.rb`

```ruby
# test/test_helper.rb

...
require_relative "test_helpers/session_test_helper"
...

module ActiveSupport
  class TestCase
  include SessionTestHelper
  ...
end
```

I removed all my old testing helpers from the codebase and added a new one.

```ruby
require "test_helper"

class HomeControllerTest < ActionDispatch::IntegrationTest
  test "..." do
    user = users(:one)
    sign_in_as(user)
    ...
  end
end
```

Run the test again and see the test is pass.

❤️