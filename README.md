# Project to learn about UUID and Soft Delete

## UIID - Reference: 
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

### Problem:
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
## PARANOIA (Soft Delete) - References: 
https://github.com/rubysherpas/paranoia
https://gorails.com/episodes/soft-delete-with-paranoia (Video)
<hr/>

### Steps:
1. After install `gem paranoia` and run `bundle install`, we'll create a migration
`rails generate migration add_deleted_at_to_authors deleted_at:datetime:index`
If don't appear like bellow, just add the lines =D
```
class AddDeletedAtToAuthors < ActiveRecord::Migration[7.0]
  def change
    add_column :authors, :deleted_at, :datetime
    add_index :authors, :deleted_at
  end
end
```
Another way to add this collumn is insert the collumn `t.datetime :deleted_at, index: true`, in the migration to create add the follow line:
```
class CreateBooks < ActiveRecord::Migration[7.0]
  def change
    create_table :books, id: :uuid do |t|
      t.string :title
      t.references :author, null: false, foreign_key: true, type: :uuid
      
      t.datetime :deleted_at, index: true

      t.timestamps
    end
  end
end
```
### Feature
After these settings, we can use all the methods that gem paranoia offer to us. To know more about the methods, visit [Paranoia Documentation](https://github.com/rubysherpas/paranoia#usage), but before...
1. We need add a helper from Paranoia to lib works:
```
class Book < ApplicationRecord
  acts_as_paranoid
  
  belongs_to :author
end
```
For further informations, visit the [documentation](https://github.com/rubysherpas/paranoia).
