---
title: "Running Rake Tasks in a Loop"
description: "Re-enabling rake tasks to make them run in a loop."
date: 2017-04-03T01:46:33+02:00
categories:
- tech
tags:
- ruby
- rails
cover:
    image: "posts/running-rake-tasks-in-a-loop/assets/ornacieux-1879-johan-barthold-jongkind.webp"
    alt: "Ornacieux (1879) - Johan Barthold Jongkind"
    relative: false
images: ["assets/ornacieux-1879-johan-barthold-jongkind.webp"]
---

Rake tasks in a loop, will only executed once if they are not "re-enabled".
Take a look at this example:

```ruby
namespace :yoksis do
  desc 'fetches all references'
  task :references do
    mapping = {
      get_instruction_language: 'UnitInstructionLanguage',
      get_instruction_type: 'UnitInstructionType'
    }

    mapping.each do |action, klass|
      Rake::Task['yoksis:reference'].invoke(action, klass)
    end
  end

  desc 'fetch an individual reference'
  task :reference, %i[soap_method klass] => [:environment] do |_, args|
    puts args[:soap_method]
    puts args[:klass]
  end
end
```

When you run the `yoksis:references` task, it will only print out
`{get_instruction_language: 'UnitInstructionLanguage'}` and will skip the
second item of the `mapping` hash.

Strange, isn't it? To overcome this issue, the rake task needs to be reenabled
during the iteration:

```ruby
Rake::Task['yoksis:reference'].reenable
```

So the previous task becomes:

```ruby
namespace :yoksis do
  desc 'fetches all references'
  task :references do
    mapping = {
      get_instruction_language: 'UnitInstructionLanguage',
      get_instruction_type: 'UnitInstructionType'
    }

    mapping.each do |action, klass|
      Rake::Task['yoksis:reference'].invoke(action, klass)
      Rake::Task['yoksis:reference'].reenable
    end
  end

  desc 'fetch an individual reference'
  task :reference, %i[soap_method klass] => [:environment] do |_, args|
    puts args[:soap_method]
    puts args[:klass]
  end
end
```

Cheers.
