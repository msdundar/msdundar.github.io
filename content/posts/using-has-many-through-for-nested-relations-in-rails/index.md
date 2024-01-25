---
title: "Using has_many :through for Nested Relations in Rails"
description: "Using 'has_many :through' for more effective nested many-to-many relations in Rails."
date: 2016-08-20T21:19:04+02:00
categories:
- tech
tags:
- rails
cover:
    image: "posts/using-has-many-through-for-nested-relations-in-rails/assets/the-towpath-1864-johan-barthold-jongkind.webp"
    alt: "The Towpath (1864) - Johan Barthold Jongkind"
    relative: false
images: ["assets/the-towpath-1864-johan-barthold-jongkind.webp"]
---

`has_many :through` is a useful association type of Rails. It's mostly popular and often used as a join model for
many-to-many relations.

However, `has_many :through` is more than a simple join model, because it conducts INNER JOIN(s) on related models. We
can also take the advantage of this behaviour on nested `has_many` relations. Lets imagine a scenario where we have a
nested `has_many` structure as follows:

```ruby
Country (has_many :regions)
  -> Region (has_many :cities)
      -> City (has_many :districts)
          -> District
```

Models and tables of the structure:

```ruby
class Country < ApplicationRecord
  has_many :regions
end

|  Country  |
|  :-----   |
|  name     |
```

```ruby
class Region < ApplicationRecord
  belongs_to :country
  has_many :cities
end

|  Region     |
|  :-----     |
|  name       |
|  country_id |
```

```ruby
class City < ApplicationRecord
  belongs_to :region
  has_many :districts
end

|  City      |
|  :-----    |
|  name      |
|  region_id |
```

```ruby
class District < ApplicationRecord
  belongs_to :city
end

|  District |
|  :-----   |
|  name     |
|  city_id  |
```

In this case, it's quite hard to reach records after more than one level of association. It's very possible to struggle
when reaching `districts` of a `country` and you will probably end up with complicated queries. First you have to scan
all regions of a specific country, then you have to scan cities of all that regions, and finally you will end up with
districts of the cities, and so on. With the current structure, we are unable to run simple queries like
`Country.first.districts`, firstly because we don't have a `country_id` in our `District` model, and secondly there is
no relationship between `Country` and `District`

Here where `has_many :through` comes in to the action! Lets update our models with `has_many :through` without
modifying the tables:

```ruby
class Country < ApplicationRecord
  has_many :regions
    has_many :cities, through: :regions
    has_many :districts, through: :cities
end
```

```ruby
class Region < ApplicationRecord
  belongs_to :country
  has_many :cities
  has_many :districts, through: :cities
end
```

```ruby
class City < ApplicationRecord
  belongs_to :region
  has_many :districts
end
```

```ruby
class District < ApplicationRecord
  belongs_to :city
end
```

And magic happens :tada: Notice that, we didn't add any foreign_key to our models! Just plain old Rails associations.
Now we are able to run any query between these models, such as:

```ruby
Country.first.districts
Region.first.districts
```

The opposite of this relation is also quite easy. We don't have a `belongs_to :through` simply because we don't need to.
You can reach the `country` of a `district` as follows:

```ruby
District.first.city.region.country
```

But how does Rails understand a relation between `district` and `country` since we don't have foreign_keys? Simple, as
I mentioned before - it conducts `INNER JOIN(s)`. Lets look at the SQL query run:

```ruby
country = Country.find_by(name: 'Turkey')
country.districts.to_sql

=>  "SELECT "districts".* FROM "districts" INNER JOIN "cities" ON "districts"."city_id" = "cities"."id" INNER JOIN "regions" ON "cities"."region_id" = "regions"."id" WHERE "regions"."country_id" = 209"
```

Cheers.
