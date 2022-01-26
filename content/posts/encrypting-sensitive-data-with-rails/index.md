---
title: "Encrypting Sensitive Data With Rails"
date: 2018-10-01T09:33:06+02:00
categories:
- tech
tags:
- rails
- security
- encryption
---

The most recent versions (5.1 and 5.2) of Ruby on Rails has shipped with a new
feature named as [encrypted credentials](https://edgeguides.rubyonrails.org/security.html#custom-credentials) which replaces the `secrets.yml`, and enables you to keep sensitive data in an
encrypted file named as `config/credentials.yml.enc`.

However, this feature only works with a single file that is
`config/credentials.yml.enc`. Recently we needed to add some data in our
repository, which we wanted to keep as encrypted, but that also didn't really
fit into the `credentials.yml.enc` conceptually.

> We were aware of many great tools available for encrypting files and
> decrypting them when needed, but we didn't want to implement another
> complexity level or some other dependency (such as a GEM or an infra tool) to
> the app. Therefore we aimed to solve this problem with minimum available
> tools, and preferably, with core capabilities of Ruby and Rails.

After taking a look to Rails source code, [roktas](https://github.com/roktas)
and I've decided to imitate the behaviour of encrypted credentials, and we ended
up writing a simple wrapper around the core `ActiveSupport` methods:

```ruby
# frozen_string_literal: true

module FileEncryptor
  DEFAULT_PARAMS = {
    env_key: 'RAILS_MASTER_KEY',
    key_path: Rails.root.join('config', 'master.key'),
    raise_if_missing_key: true
  }.freeze

  def self.encrypt(path)
    encryptor = ActiveSupport::EncryptedFile.new(
      merge_with_content_path(Rails.root.join('db', 'encrypted_data', path.split('/').last + '.enc'))
    )

    encryptor.write(File.read(Rails.root.join(path)))
  end

  def self.decrypt(path)
    encryptor = ActiveSupport::EncryptedFile.new(
      merge_with_content_path(Rails.root.join(path))
    )

    encryptor.read
  end

  def self.decrypt_lines(path)
    decrypt(path).split("\n")
  end

  def self.merge_with_content_path(value)
    DEFAULT_PARAMS.merge(
      content_path: value
    )
  end
end
```

Briefly, it wraps the `ActiveSupport::EncryptedFile` and behaves exactly the
same. You can use either an environment variable or the `master.key` for
decrypting encrypted files, and it works with absolute and relative paths.
Here are some examples:

Encrypt a file:

```ruby
FileEncryptor.encrypt('db/static_data/users.csv')
```

Decrypt an encrypted file as a whole:

```ruby
FileEncryptor.decrypt('db/encrypted_data/users.csv.enc')
```

Decrypt an encrypted file as an array:

```ruby
FileEncryptor.decrypt_lines('db/encrypted_data/users.csv.enc')
```

`decrypt_lines` method allows you to iterate encrypted records one by one:

```ruby
users = FileEncryptor.decrypt_lines('/lib/important_files/users.csv.enc')

users.each do |user|
  User.create(...)
end
```

No magic, no dependencies, no GEMs, no packages. Just native capabilities of
Ruby on Rails.

Cheers.
