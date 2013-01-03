# Code Quality Standards
================================

## Available Object “Types”

### 1. Concerns
Type: Module

Concerns are a module of shared code that you mix in to your Active Record classes.  The business logic ends up in the Active Record module through including the module, its just separated in the filesystem.

See this post from DHH for more details: http://37signals.com/svn/posts/3372-put-chubby-models-on-a-diet-with-concerns

### 2. Service Objects
Type: Class

Use a service object when any of the following is true:

* The action is complex (e.g. closing the books at the end of an accounting period)
* The action reaches across multiple models (e.g. an e-commerce purchase using Order, CreditCard and Customer objects)
* The action interacts with an external service (e.g. posting to social networks)
* The action is not a core concern of the underlying model (e.g. sweeping up outdated data after a certain time period).
* There are multiple ways of performing the action (e.g. authenticating with an access token or password). This is the Gang of Four Strategy pattern.

**Note:** We have a similar object, but not identical.  “Actors” perform complex tasks, but aren’t as standardized as these service objects.  

### 3. Form Objects
Type: Class
Includes ActiveRecord validations, uses Virtus attributes

A form that handles multiple objects.  Implements a #save method to behave like an ActiveRecord model.  Example from Lead Connector: the signup form, adding/editing an employee.

Note:  We have 1 form object, the signup form.  This could be refactored though to use the cleaner pattern in the Code Climate quality article.  We also could probably afford to use one for the person form, so that we don’t depend on callbacks to update Gatekeeper, etc.

### 4. Query Objects
Type: Class

Used for complex queries.  Could be helpful for queries like the filtering the lead page, or searching for leads.  This keeps the controller from becoming polluted.

### 5. Draper Decorators
Type: Class
Source: Draper gem

Adds view-specific functionality to an ActiveRecord model in an object oriented fashion, rather than calling specific functional methods on a collection or object (default Rails helpers).

### 6. Rails Helpers
Type: Module
Source: Rails

These are collections of methods, basically functional.  We’re currently using them but they are redundant in light of Draper Decorators.  They should be refactored.


