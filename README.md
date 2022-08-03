# Project to learn about UUID and Soft Delete

## Reference: 
http://linqueta.com.br/rails/uuid/active_record/2019/05/24/rails_uuid_primary_key/
<hr/>

### Steps:
1. Add config to primary key is not more sequentials, but to be a "hash" (https://en.wikipedia.org/wiki/Universally_unique_identifier);

`config/application.rb`
```
module Paranoia
  class Application < Rails::Application
    # ...
    config.generators do |g|
      g.orm :active_record, primary_key_type: :uuid
    end
  end
end
```

2. Add extensions to Postgresql build UUID in database;

`$ rails g migration enable_uuid_extension_and_pgcrypto`

```
class EnableUuidExtensionAndPgcrypto < ActiveRecord::Migration[5.2]
  def change
    enable_extension 'uuid-ossp'
    enable_extension 'pgcrypto'
  end
end
```
run `rails db:migrate` and your schema will appear like:
```
 ActiveRecord::Schema.define(version: 2019_05_21_005059) do
  enable_extension "pgcrypto"
  enable_extension "plpgsql"
  enable_extension "uuid-ossp"
  # ...
 end  
```

3. Create your models
`rails g model Author name:string`
`rails g model Book title:string author:references`

When a model has reference, depending on your version, we may be missing an identifier in the new migration. But in the latest versions of Postgresql the `type: :uuid` is already being inserted in the migration.

```
class CreateBook < ActiveRecord::Migration[5.2]
  def change
    create_table :book_uuids, id: :uuid do |t|
      t.string :title
      t.references :author, foreign_key: true, index: true, type: :uuid

      t.timestamps
    end
  end
end
```

### Problem
Maybe your last data created not be the last in ActiveRecord because the new ID (UUID). Because is a string, so if you call `Author.last` maybe the last data is not called by ActiveRecord, so we can to solve this issue:

Directly in model:
```
class Author < ApplicationRecord
  default_scope { order(:created_at) }
end
```
Or to become a default to all models:
`models/application_record`

```
class ApplicationRecord < ActiveRecord::Base
  primary_abstract_class

  default_scope { order(:created_at) }
end
```
