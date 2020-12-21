## Single table inheritance

Rails makes it possible to have multiple models stored in the same table and map these rows to the correct models using a type column.

Active Record allows inheritance by storing the name of the class in a column that by default is named “type” (can be changed by overwriting Base.inheritance_column). This means that an inheritance looks like the below one.
```
class Company < ActiveRecord::Base; end
class Firm < Company; end
class Client < Company; end
class PriorityClient < Client; end
```


It should be noted that because the type column is an attribute on the record every new subclass will instantly be marked as dirty and the type column will be included in the list of changed attributes on the record. This is different from non Single Table Inheritance(STI) classes:


If a type column defined is not defined in table, single-table inheritance won't be triggered. In that case, it'll work just like normal subclasses with no special magic for differentiating between them or reloading the right type with find.

## References
* https://api.rubyonrails.org/classes/ActiveRecord/Inheritance.html
* https://www.youtube.com/watch?v=t8I4_8HcMPo
