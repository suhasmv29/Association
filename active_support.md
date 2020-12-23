# Active support

**blank? and present?**

The following values are considered to be blank in a Rails application:
* nil and false
* strings composed only of whitespace
* empty arrays and hashes
* any other object that responds to empty? and is empty

For example, this method from ActionController::HttpAuthentication::Token::ControllerMethods uses blank? for checking whether a token is present:
```
def authenticate(controller, &login_procedure)
  token, options = token_and_options(controller.request)
  unless token.blank?
    login_procedure.call(token, options)
  end
end
```
The method present? is equivalent to !blank?. This example is taken from ActionDispatch::Http::Cache::Response:
```
def set_conditional_cache_control!
  return if self["Cache-Control"].present?
  ...
end
```

**to_param**

All objects in Rails respond to the method to_param, which is meant to return something that represents them as values in a query string, or as URL fragments.

By default to_param just calls to_s:
```
7.to_param # => "7"
```

The return value of to_param should not be escaped:
```
"Tom & Jerry".to_param # => "Tom & Jerry"
```

Several classes in Rails overwrite this method.

For example nil, true, and false return themselves. Array#to_param calls to_param on the elements and joins the result with "/":
```
[0, true, String].to_param # => "0/true/String"
```

Notably, the Rails routing system calls to_param on models to get a value for the :id placeholder. ActiveRecord::Base#to_param returns the id of a model, but the method can redefined in models. For example, given
```
class User
  def to_param
    "#{id}-#{name.parameterize}"
  end
end
```

we get:
user_path(@user) # => "/users/357-john-smith"

**in?**

The predicate in? tests if an object is included in another object. An ArgumentError exception will be raised if the argument passed does not respond to include?.

Examples of in?:
```
1.in?([1,2])        # => true
"lo".in?("hello")   # => true
25.in?(30..50)      # => false
1.in?(1)            # => ArgumentError
```
**remove**

The method remove will remove all occurrences of the pattern:
```
"Hello World".remove(/Hello /) # => "World"
```
**squish**

The method squish strips leading and trailing whitespace, and substitutes runs of whitespace with a single space each:
```
" \n  foo\n\r \t bar \n".squish # => "foo bar"
```
**starts_with? and ends_with?**

Active Support defines 3rd person aliases of String#start_with? and String#end_with?:
```
"foo".starts_with?("f") # => true
"foo".ends_with?("o")   # => true
```
**at(position)**

The at method returns the character of the string at position position:
```
"hello".at(0)  # => "h"
"hello".at(4)  # => "o"
"hello".at(-1) # => "o"
"hello".at(10) # => nil
```
**pluralize**

The method pluralize returns the plural of its receiver:
```
"table".pluralize     # => "tables"
"ruby".pluralize      # => "rubies"
"equipment".pluralize # => "equipment"
```
**camelize**

The method camelize returns its receiver in camel case:
```
"product".camelize    # => "Product"
"admin_user".camelize # => "AdminUser"
```
**multiple_of?**

The method multiple_of? tests whether an integer is multiple of the argument:
```
2.multiple_of?(1) # => true
1.multiple_of?(2) # => false
```
**sum**

The method sum adds the elements of an enumerable:
```
[1, 2, 3].sum # => 6
(1..100).sum  # => 5050
```
**Durations**

Duration objects can be added to and subtracted from time objects:
```
now = Time.current
# => Tue, 22 Dec 2020 23:20:05 UTC +00:00
now + 1.year
# => Wed, 22 Dec 2021 23:21:11 UTC +00:00
now - 1.week
# => Tue, 15 Dec 2020 23:21:11 UTC +00:00
```

## References :

* https://guides.rubyonrails.org/active_support_core_extensions.html
* https://www.youtube.com/watch?v=SfbPePRc2tw
