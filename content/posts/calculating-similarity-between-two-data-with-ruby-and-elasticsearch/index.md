---
title: "Calculating Similarity Between Two Data With Ruby and Elasticsearch"
date: 2015-12-08T19:04:50+02:00
categories:
- tech
tags:
- ruby
- elasticsearch
---

I recently had to find similar data located in a dataset, in order to find potential duplicate records:

```
"John Doe 123456789"
"John Foe 123123123"
```

After considering a couple of options, I've decided to continue with Elasticsearch, as it was already integrated in the
project I was working on. The Ruby client of Elasticsearch provided a useful function on search results,
`records.each_with_hit`, that I could abuse for this situation:

```ruby
file = File.open("some_file_path", "w")

User.all.each do |u|
  r = User.search "*#{u.full_name}*"

  file.write("Searching for: #{u.id_number}-#{u.full_name}\n")

  r.records.each_with_hit {|r, hit|}.map{|k, v| "#{k.id_number}-#{k.full_name}:  #{v._score}"}.each do |y|
    unless (u.id_number == y.split("-")[0])
      if y.split(": ")[1].to_f > 1.2
        file.write("                           #{y}\n")
      end
    end
  end
  file.write("***********************************\n\n")
end
```

The `1.2` value in the script can be adjusted according to your needs. It will basically represent how close two values
are. You might choose to increase it, if you want your results to cover a wider range of records, or decrease it
if you are only interested in values looking very similar. Here is a sample output for the script:

```
Searching for: 11156069590-JOHN DOE WHITE
  33304078456-JOHN DOE BROWN: 2.3747075
  22256325768-JOHN DOE BLACK: 2.2594523
  33315234436-JOHN DOE BLUE: 2.229414
  33378500384-JOHN DOE RED: 2.146143
  11171764882-JOHN DOE GRAY: 2.146143
  55577374544-JOHN DOE PURPLE: 2.0586686

Searching for: 00027522389-JOHANNA DOE
  00022785363-JOHANNA DOE: 1.4142135
```
 
Cheers.
