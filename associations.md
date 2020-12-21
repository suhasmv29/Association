# The Types of Associations

Rails supports six types of associations.

* belongs_to 
 *    has_one
 *   has_many
 *   has_many :through
  *  has_one :through
  * has_and_belongs_to_many

Associations are implemented using macro-style calls, so that features can be declartively added to models. For example, by declaring that one model belongs_to another, Rails will be insruced to maintain Primary Key-Foreign Key information between instances of the two models, and number of utility methods added to the model will alos be known.

**2.1 The belongs_to Association**

A belongs_to association sets up a connection with another model, such that each instance of the declaring model "belongs to" one instance of the other model. 
For example, if an application includes authors and books, and each book can be assigned to exactly one author.
```
class Book < ApplicationRecord
  belongs_to :author
end
```

The corresponding migration might look like below.
```
class CreateBooks < ActiveRecord::Migration[6.0]
  def change
    create_table :authors do |t|
      t.string :name
      t.timestamps
    end

    create_table :books do |t|
      t.belongs_to :author
      t.datetime :published_at
      t.timestamps
    end
  end
end
```

When used alone, belongs_to produces a one-directional one-to-one connection. Therefore each book in the above example "knows" its author, but the authors don't know about their books. 
To setup a bi-directional association, belongs_to is used in  combination with a has_one or has_many on the other model.

belongs_to does not ensure reference consistency, so depending on the use case, a database-level foreign key constraint on the reference column needs to be added like below.
```
create_table :books do |t|
  t.belongs_to :author, foreign_key: true
  # ...
end
```

**2.2 The has_one Association**

A has_one association indicates that one or the other model has a reference to this model. That model can be fetched through this association.

For example, if each supplier in an application has only one account, the supplier model would be like the one below.
```
class Supplier < ApplicationRecord
  has_one :account
end
```

The main difference from belongs_to is that the link column supplier_id is located in the other table.


The corresponding migration might look like this:
```
class CreateSuppliers < ActiveRecord::Migration[6.0]
  def change
    create_table :suppliers do |t|
      t.string :name
      t.timestamps
    end

    create_table :accounts do |t|
      t.belongs_to :supplier
      t.string :account_number
      t.timestamps
    end
  end
end
```

Depending on the use case, there might be a need to create a unique index and/or a foreign key constraint on the supplier column for the accounts table. 
In this case, the column definition might look like the below one.
```
create_table :accounts do |t|
  t.belongs_to :supplier, index: { unique: true }, foreign_key: true
  # ...
end
```

This relation can be bi-directional when used in combination with belongs_to on the other model.

**2.3 The has_many Association**

A has_many association is similar to has_one, but indicates a one-to-many connection with another model. 
This will be often found in association on the "other side" of a belongs_to association. 
This indicates that each instance of the model has zero or more instances of another model. 
For example, in an application containing authors and books, the author model could be declared like the below one.
```
class Author < ApplicationRecord
  has_many :books
end
```

The name of the other model is pluralized when declaring a has_many association.

The corresponding migration might look like the one below.
```
class CreateAuthors < ActiveRecord::Migration[6.0]
  def change
    create_table :authors do |t|
      t.string :name
      t.timestamps
    end

    create_table :books do |t|
      t.belongs_to :author
      t.datetime :published_at
      t.timestamps
    end
  end
end
```

Depending on the use case, it's usually a good idea to create a non-unique index and optionally a foreign key constraint on the author column for the books table.
```
create_table :books do |t|
  t.belongs_to :author, index: true, foreign_key: true
  # ...
end
```

**2.4 The has_many :through Association**

A has_many :through association is often used to set up a many-to-many connection with another model. This association indicates that the declaring model can be matched with zero or more instances of another model by proceeding through a third model. 
For example, consider a medical practice where patients make appointments to see physicians. The relevant association declarations are as follows.
```
class Physician < ApplicationRecord
  has_many :appointments
  has_many :patients, through: :appointments
end

class Appointment < ApplicationRecord
  belongs_to :physician
  belongs_to :patient
end

class Patient < ApplicationRecord
  has_many :appointments
  has_many :physicians, through: :appointments
end
```

The corresponding migration might look like below.
```
class CreateAppointments < ActiveRecord::Migration[6.0]
  def change
    create_table :physicians do |t|
      t.string :name
      t.timestamps
    end

    create_table :patients do |t|
      t.string :name
      t.timestamps
    end

    create_table :appointments do |t|
      t.belongs_to :physician
      t.belongs_to :patient
      t.datetime :appointment_date
      t.timestamps
    end
  end
end
```

The collection of join models can be managed via the has_many association methods. 
For example, 
```
physician.patients = patients
```
Then new join models are automatically created for the newly associated objects. 
If some that existed previously are now missing, then their join rows are automatically deleted.

Automatic deletion of join models is direct, no destroy callbacks are triggered.

The has_many :through association is also useful for setting up "shortcuts" through nested has_many associations. 
For example, if a document has many sections, and a section has many paragraphs.
To get a simple collection of all paragraphs in the document,
```
class Document < ApplicationRecord
  has_many :sections
  has_many :paragraphs, through: :sections
end

class Section < ApplicationRecord
  belongs_to :document
  has_many :paragraphs
end

class Paragraph < ApplicationRecord
  belongs_to :section
end

```
**2.5 The has_one :through Association**

A has_one :through association sets up a one-to-one connection with another model. This association indicates that the declaring model can be matched with one instance of another model by proceeding through a third model. For example, if each supplier has one account, and each account is associated with one account history, then the supplier model could be as follows.
```
class Supplier < ApplicationRecord
  has_one :account
  has_one :account_history, through: :account
end

class Account < ApplicationRecord
  belongs_to :supplier
  has_one :account_history
end

class AccountHistory < ApplicationRecord
  belongs_to :account
end
```

The corresponding migration might look like the one below.
```
class CreateAccountHistories < ActiveRecord::Migration[6.0]
  def change
    create_table :suppliers do |t|
      t.string :name
      t.timestamps
    end

    create_table :accounts do |t|
      t.belongs_to :supplier
      t.string :account_number
      t.timestamps
    end

    create_table :account_histories do |t|
      t.belongs_to :account
      t.integer :credit_rating
      t.timestamps
    end
  end
end
```

**2.6 The has_and_belongs_to_many Association**

A has_and_belongs_to_many association creates a direct many-to-many connection with another model, with no intervening model. This association indicates that each instance of the declaring model refers to zero or more instances of another model. 
For example, if an application includes assemblies and parts, with each assembly having many parts and each part appearing in many assemblies, model declaration woul be as follows
```
class Assembly < ApplicationRecord
  has_and_belongs_to_many :parts
end

class Part < ApplicationRecord
  has_and_belongs_to_many :assemblies
end
```

The corresponding migration might look like the one below.
```
class CreateAssembliesAndParts < ActiveRecord::Migration[6.0]
  def change
    create_table :assemblies do |t|
      t.string :name
      t.timestamps
    end

    create_table :parts do |t|
      t.string :part_number
      t.timestamps
    end

    create_table :assemblies_parts, id: false do |t|
      t.belongs_to :assembly
      t.belongs_to :part
    end
  end
end
```

## References :

* https://guides.rubyonrails.org/association_basics.html
