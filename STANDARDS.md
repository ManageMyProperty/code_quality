# Standards

## Definitions
--------------------------------------

* "Business Logic" -> any logic which accesses or affects the data model.  (Ex. Logging in, adding a user, contacting a lead, etc.)

# Object Types & Purposes

## Controllers
--------------------------------------
URL mapping for the app, providing a user-facing interface for manipulating the app.

#### Responsibilities
1. Displaying content on given URLs
2. Strong parameter filtering
3. Handing off business logic to third party objects

#### Don'ts
* Write business logic in the controller

## Decorators
--------------------------------------
Dressing up the views for user consumption.

#### Responsibilities
1. View-related logic that pertains to a single record. (Ex. what email address to display, which avatar to show, etc)

## Helpers
--------------------------------------
Dressing up views for user consumption.  These are similar to decorators but will be used when operating on a collection, rather than a single object.

A common example would be where the you have a collection of objects, and you want to display them in a table; or if there are no objects, display a "no objects" message.

You'd use a helper like this:

```ruby
module ObjectNameHelper
  def objects_list(objects)
    if objects.size > 0
      # display the objects
    else
      # display a "no objects" message
    end
  end
end

# Then in the view:
<%= objects_list(objects) %>
```

## Extensions
--------------------------------------
**Flagged for reconsideration**  

These are monkey-patches of base classes, such as TrueClass and FalseClass to add a "yes" or "no functionality".

#### Example
```ruby
# extensions/booleans.rb
class TrueClass
  def yesno
    "Yes"
  end
end

class FalseClass
  def yesno
    "No"
  end
end
```

## Forms
--------------------------------------
Complex save operations that require more than just a single Active Record object or operation.

