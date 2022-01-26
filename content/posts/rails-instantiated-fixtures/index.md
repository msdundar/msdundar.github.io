---
title: "Rails Instantiated Fixtures"
description: "Auto-instantiated fixtures in Ruby on Rails."
date: 2017-05-03T18:52:34+02:00
categories:
- tech
tags:
- rails
cover:
    image: "posts/rails-instantiated-fixtures/assets/notre-dame-vue-du-quai-de-la-tournelle-1852-johan-barthold-jongkind.jpg"
    alt: "Notre-Dame vue du quai de la Tournelle (1852) - Johan Barthold Jongkind"
    relative: false
images: ["assets/notre-dame-vue-du-quai-de-la-tournelle-1852-johan-barthold-jongkind.jpg"]
---

Here is a sample Ruby on Rails fixture named as `newsletter`:

```yml
newsletter:
  name: foo
  message: bar
  first_name: foo
  last_name: bar
```

There are two popular ways to use this fixture in your Rails tests. The first one is directly calling the name of
fixture file, followed by a symbol stating the name of any individual fixture:

```ruby
class NewsletterTest < ActiveSupport::TestCase
  test 'a sample test' do
    assert newsletters(:newsletter).valid?
  end
end
```

And the other one is assigning fixtures to an instance variable in the `setup` block:

```ruby
class NewsletterTest < ActiveSupport::TestCase
  setup do
    @newsletter = newsletters(:tenant_newsletter)
  end

  test 'a sample test' do
    assert @newsletter.valid?
  end
end
```

When you would like to stick with instance variables, there is an easier way, _instantiated fixtures_:

```ruby
class NewsletterTest < ActiveSupport::TestCase
  self.use_instantiated_fixtures = true

  test 'a sample test' do
    assert @newsletter.valid?
  end
 end
```

`instantiated_fixtures` automatically creates instance variables for each fixture item, and they become accessible as
instance variables with the same name. Imagine a more populated fixture like this:

```
city:
    name: Samsun
country:
    name: Turkey
region:
    name: Black Sea
```

when you enable `instantiated_fixtures` you will automatically have three different instance variables named as
`@city`, `@country` and `@region`. These instance variables will immediately be accessible from anywhere in your
test file.

This behaviour might come in handy for some cases but please be aware of the risk of **conflicting variables**! Since
`instantiated_fixtures` will give you an instance variable for each item, you must make sure that you don't use these
variable names anywhere else in your test files, in order to prevent possible conflicts and confusions.

Cheers.
