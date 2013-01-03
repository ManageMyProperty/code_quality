# Code Quality Standards
================================

## Available Object “Types”

### 1. Concerns
Type: Module

Concerns are a module of shared code that you mix in to your Active Record classes.  The business logic ends up in the Active Record module through including the module, its just separated in the filesystem.

See this post from DHH [for more details](http://37signals.com/svn/posts/3372-put-chubby-models-on-a-diet-with-concerns).

##### Example
```ruby
module Taggable
  extend ActiveSupport::Concern

  included do
    has_many :taggings, as: :taggable, dependent: :destroy
    has_many :tags, through: :taggings 
  end

  def tag_names
    tags.map(&:name)
  end
end
```

##### Usage
```ruby
# in the Active Record model
class Lead < ActiveRecord::Base
  include Taggable
end

# in some other context
lead = Lead.find(5)
lead.tag_names # => the tag names from Taggable
```

### 2. Service Objects
Type: Class

Use a service object when any of the following is true:

* The action is complex (e.g. closing the books at the end of an accounting period)
* The action reaches across multiple models (e.g. an e-commerce purchase using Order, CreditCard and Customer objects)
* The action interacts with an external service (e.g. posting to social networks)
* The action is not a core concern of the underlying model (e.g. sweeping up outdated data after a certain time period).
* There are multiple ways of performing the action (e.g. authenticating with an access token or password). This is the Gang of Four Strategy pattern.

**Note:** We have a similar object, but not identical.  “Actors” perform complex tasks, but aren’t as standardized as these service objects.

##### Example
```ruby
class UserAuthenticator
  def initialize(user)
    @user = user
  end

  def authenticate(unencrypted_password)
    return false unless @user

    if BCrypt::Password.new(@user.password_digest) == unencrypted_password
      @user
    else
      false
    end
  end
end
```

##### Usage
```ruby
class SessionsController < ApplicationController
  def create
    user = User.where(email: params[:email]).first

    if UserAuthenticator.new(user).authenticate(params[:password])
      self.current_user = user
      redirect_to dashboard_path
    else
      flash[:alert] = "Login failed."
      render "new"
    end
  end
end
```

### 3. Form Objects
Type: Class  
Includes ActiveRecord validations, uses Virtus attributes

A form that handles multiple objects.  Implements a #save method to behave like an ActiveRecord model.  Example from Lead Connector: the signup form, adding/editing an employee.

Note:  We have 1 form object, the signup form.  This could be refactored though to use the cleaner pattern in the Code Climate quality article.  We also could probably afford to use one for the person form, so that we don’t depend on callbacks to update Gatekeeper, etc.

##### Example
```ruby
class Signup
  include Virtus

  extend ActiveModel::Naming
  include ActiveModel::Conversion
  include ActiveModel::Validations

  attr_reader :user
  attr_reader :company

  attribute :name, String
  attribute :company_name, String
  attribute :email, String

  validates :email, presence: true
  # … more validations …

  # Forms are never themselves persisted
  def persisted?
    false
  end

  def save
    if valid?
      persist!
      true
    else
      false
    end
  end

private

  def persist!
    @company = Company.create!(name: company_name)
    @user = @company.users.create!(name: name, email: email)
  end
end
```

##### Usage
```ruby
class SignupsController < ApplicationController
  def create
    @signup = Signup.new(params[:signup])

    if @signup.save
      redirect_to dashboard_path
    else
      render "new"
    end
  end
end
```

### 4. Query Objects
Type: Class

Used for complex queries.  Could be helpful for queries like the filtering the lead page, or searching for leads.  This keeps the controller from becoming polluted.

##### Example
```ruby
class AbandonedTrialQuery
  def initialize(relation = Account.scoped)
    @relation = relation
  end

  def find_each(&block)
    relation.
      where(plan: nil, invites_count: 0).
      find_each(&block)
  end
end
```
##### Usage
```ruby
AbandonedTrialQuery.new.find_each do |account|
  account.send_offer_for_support
end
```

### 5. Draper Decorators
Type: Class  
Source: Draper gem

Adds view-specific functionality to an ActiveRecord model in an object oriented fashion, rather than calling specific functional methods on a collection or object (default Rails helpers).

##### Example
```ruby
class LeadDecorator < Draper::Decorator
  include Draper::LazyHelpers

  def email_address
    # display real or masked email here, if present
  end
end
```

##### Usage
```ruby
# in leads/show
<%= @lead.email_address %>
```

### 6. Rails Helpers
Type: Module
Source: Rails

These are collections of methods, basically functional.  We’re currently using them but they are redundant in light of Draper Decorators.  They should be refactored.

##### Example
```ruby
module LeadsHelper
  def lead_email_address(lead)
    # display real or masked email here, if present
  end
end
```

##### Usage
```ruby
# in leads/show
<%= lead_email_address(@lead) %>
```

### 7. Value Objects
Type: Class  

From the [Code Climate quality article](https://www.evernote.com/shard/s19/sh/f4225911-686e-4806-aaf6-58ace1111fce/de6c472b4cd35cd8e9ec4805ed6d3e14):
> Value Objects are simple objects whose equality is dependent on their value rather than an identity. They are usually immutable. Date, URI, and Pathname are examples from Ruby’s standard library, but your application can (and almost certainly should) define domain-specific Value Objects as well. Extracting them from ActiveRecords is low hanging refactoring fruit.

These could be useful to us for the following types of things:
* Statistics
* Contact Information
* Letter scores on company performance

##### Example
```ruby
class Rating
  include Comparable

  def self.from_cost(cost)
    if cost <= 2
      new("A")
    elsif cost <= 4
      new("B")
    elsif cost <= 8
      new("C")
    elsif cost <= 16
      new("D")
    else
      new("F")
    end
  end

  def initialize(letter)
    @letter = letter
  end

  def better_than?(other)
    self > other
  end

  def <=>(other)
    other.to_s <=> to_s
  end

  def hash
    @letter.hash
  end

  def eql?(other)
    to_s == other.to_s
  end

  def to_s
    @letter.to_s
  end
end
```

##### Usage
```ruby
class ConstantSnapshot < ActiveRecord::Base
  # …

  def rating
    @rating ||= Rating.from_cost(cost)
  end
end
```

### 8. Policy Objects

> Sometimes complex read operations might deserve their own objects. In these cases I reach for a Policy Object. This allows you to keep tangential logic, like which users are considered active for analytics purposes, out of your core domain objects.

This could be useful for our analytics on users.  We could extract the logic for how we consider a company/user activated into these types of objects.

##### Example
```ruby
class ActiveUserPolicy
  def initialize(user)
    @user = user
  end

  def active?
    @user.email_confirmed? &&
    @user.last_login_at > 14.days.ago
  end
end
```

> This Policy Object encapsulates one business rule, that a user is considered active if they have a confirmed email address and have logged in within the last two weeks. You can also use Policy Objects for a group of business rules like an Authorizer that regulates which data a user can access.

> Policy Objects are similar to Service Objects, but I use the term “Service Object” for write operations and “Policy Object” for reads. They are also similar to Query Objects, but Query Objects focus on executing SQL to return a result set, whereas Policy Objects operate on domain models already loaded into memory.