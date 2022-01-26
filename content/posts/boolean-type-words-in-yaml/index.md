---
title: "Boolean Type Words in YAML"
description: "A brief summary of boolean type words in YAML."
date: 2016-06-07T09:10:16+02:00
categories:
- tech
tags:
- yaml
cover:
    image: "posts/boolean-type-words-in-yaml/assets/environs-de-grenoble-1873-johan-barthold-jongkind.jpg"
    alt: "Environs de Grenoble (1873) - Johan Barthold Jongkind"
    relative: false
images: ["assets/environs-de-grenoble-1873-johan-barthold-jongkind.jpg"]
---

YAML is a widely used data serialization language. In almost any software
project, or during any dev-ops tasks, you can come across with YAML.
For example Ruby on Rails uses YAML for fixtures, configuration files and
localization. CI/CD tools such as CircleCI and Travis also use YAML for
configuration. If you ever experienced a strange behaviour with YAML, you may
have used one of the many reserved words of YAML. YAML reserves some words
such as 'yes', 'no', 'y', 'n', 'off', 'on', etc. for boolean type. For example:

```yaml
yes:
	turkish: evet
  english: yes
no:
	turkish: hayır
	english: no
```

will be interpreted as:

```yaml
true:
	turkish: evet
  english: true
false:
	turkish: hayır
	english: false
```

actually [YAML has a long list of reserved words](http://yaml.org/type/bool.html)
for the boolean type:

```
y|Y|yes|Yes|YES|n|N|no|No|NO
|true|True|TRUE|false|False|FALSE
|on|On|ON|off|Off|OFF
```

In order to prevent YAML from interpreting these words as a boolean, you need to
wrap them in single or double quotes as follows:

```yaml
'yes':
	'turkish': 'evet'
    'german': 'ja'
    'english': 'yes'
'no':
	'turkish': 'hayır'
    'german': 'nein'
	'english': 'no'
```

[YAML also interprets](http://yaml.org/type/null.html) some words to `null`, so
wrap them inside quotes too:

```
~ # (canonical)
|null|Null|NULL # (English)
| # (Empty)
```

Cheers.
