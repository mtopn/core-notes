- [1. Omise Core](#1-omise-core)
  - [1.1. API Versioning](#11-api-versioning)
  - [1.2. Opening PRs (pull request) for development](#12-opening-prs-pull-request-for-development)
    - [1.2.1. Code linting (rubocup)](#121-code-linting-rubocup)
    - [1.2.2. Production](#122-production)
    - [1.2.3. Staging](#123-staging)
  - [1.3. Account](#13-account)
  - [1.4. Policies](#14-policies)
  - [1.5. Serializer](#15-serializer)
    - [1.5.1. Custom serializer](#151-custom-serializer)
  - [1.6. Registration](#16-registration)
  - [1.7. Errors](#17-errors)
    - [1.7.1. Generic Omise Errors](#171-generic-omise-errors)
  - [1.8. Tips for development (local)](#18-tips-for-development-local)
- [2. Ruby and Ruby on Rails](#2-ruby-and-ruby-on-rails)
  - [2.1. Ruby](#21-ruby)
    - [2.1.1. Array](#211-array)
    - [2.1.2. Lambda function](#212-lambda-function)
      - [2.1.2.1. ActiveRecord scopes](#2121-activerecord-scopes)
      - [2.1.2.2. Lambda vs Proc](#2122-lambda-vs-proc)
      - [2.1.2.3. Lambda use cases](#2123-lambda-use-cases)
    - [2.1.3. Ruby core](#213-ruby-core)
    - [2.1.4. Ruby method](#214-ruby-method)
      - [2.1.4.1. Method arguments](#2141-method-arguments)
    - [2.1.5. Class attribute, property, getter, and setter](#215-class-attribute-property-getter-and-setter)
  - [2.2. Ruby on Rails](#22-ruby-on-rails)
    - [2.2.1. Best practices](#221-best-practices)
      - [2.2.1.1. Fat Model, Skinny Controller](#2211-fat-model-skinny-controller)
      - [2.2.1.2. Eager Loading](#2212-eager-loading)
      - [2.2.1.3. Model vs Database validation](#2213-model-vs-database-validation)
      - [2.2.1.4. Prevent SQL injection](#2214-prevent-sql-injection)
      - [2.2.1.5. Enums](#2215-enums)
      - [2.2.1.6. Use Time.current instead of Time.now](#2216-use-timecurrent-instead-of-timenow)
      - [2.2.1.7. Concerns, Services and Helpers](#2217-concerns-services-and-helpers)
        - [2.2.1.7.1. Concerns](#22171-concerns)
        - [2.2.1.7.2. Services](#22172-services)
        - [2.2.1.7.3. Helpers](#22173-helpers)
      - [2.2.1.8. Helpful Gems](#2218-helpful-gems)
    - [2.2.2. App convention](#222-app-convention)
      - [2.2.2.1. Auto loading](#2221-auto-loading)
    - [2.2.3. View](#223-view)
    - [2.2.4. Router](#224-router)
    - [2.2.5. Controller](#225-controller)
      - [2.2.5.1. Conventions](#2251-conventions)
    - [2.2.6. Model](#226-model)
      - [2.2.6.1. Database migration](#2261-database-migration)
      - [2.2.6.2. Create new entity](#2262-create-new-entity)
        - [2.2.6.2.1. Using a form_builder](#22621-using-a-form_builder)
        - [2.2.6.2.2. Using strong parameters](#22622-using-strong-parameters)
        - [2.2.6.2.3. Validations and displaying error message](#22623-validations-and-displaying-error-message)
      - [2.2.6.3. Update an entity](#2263-update-an-entity)
      - [2.2.6.4. Model with reference](#2264-model-with-reference)
    - [2.2.7. Concerns](#227-concerns)
    - [2.2.8. Active Record (ORM)](#228-active-record-orm)
      - [2.2.8.1. Pluck](#2281-pluck)
    - [2.2.9. Meta programming](#229-meta-programming)
      - [2.2.9.1. Instance variable set](#2291-instance-variable-set)

# 1. Omise Core

## 1.1. API Versioning

1. Naming convention is following alphabetic moons.
2. [https://opn-ooo.atlassian.net/wiki/spaces/EN/pages/579733167/API+versioning](https://opn-ooo.atlassian.net/wiki/spaces/EN/pages/579733167/API+versioning)

## 1.2. Opening PRs (pull request) for development

### 1.2.1. Code linting (rubocup)

1. `rubocop` is a ruby linter and applied in the CI pipeline.

```shell
rubocop -a
```

### 1.2.2. Production

1. Require marking with `category:[level]` and `docs:ok` label for the PR to pass `policy-bot: master`.

### 1.2.3. Staging

1. Development are mostly building on top and branch from `master` branch.
2. When creating a PR for `staging`, branch out from latest staging and merge latest `HEAD` from `master`.
3. Cherry pick commits of works on `master` branch.
4. Push the branch and open PR against `staging`.

```shell
# branch from staging
git checkout -b your-branch-staging
# get latest header from master branch
git merge -s ours origin/master
# cherry-pick commits from development on master branch
git cherry-pick A^..B
```

## 1.3. Account

1. An `account` works as a mixin and includes with various properties and entities and defined at `/app/models/account.rb`
2. `account.closure_node` is related to `sub_merchants`.
3. The account is given in the authentication layer and assigned as an inherited variable `@account` that can be used in all the child classes.
4. `app/controllers/concerns/authenticable.rb`

## 1.4. Policies

1. A team can invited the other user (account) to join as different role (e.g. `admin` and `technical manager`)
2. `policies` method can be applied to allow private endpoint to work as the invited team.
3. Besides adding `policies`, we need to add/update the route in `app/models/team_membership.rb` and its test

```ruby
# allow invited role as admin and technical manager to work on index method
class SomeController
  policies do
    allow :index, only: ["membership.admin", "membership.technical"]
  end

  def index
    # some action
  end
end
```

## 1.5. Serializer

### 1.5.1. Custom serializer

1. Update `app/routes.rb`

```ruby
OmiseCore::Application.routes.draw do
  namespace :api, path: nil, defaults: { format: :json }, constraints: { subdomain: /^api(-\w+)*$/ } do
    resources :some_entities, only: %i[index show] do
    end
  end
end
```

2. Assign serializer in controller

```ruby
class SomeEntityController < API::BaseController
  def index
    some_data = SomeEntity.all

    render json: serialize(some_data,
      as: :some_entity_list_serializer,
      to: :json,
      location: api_some_entities_path)
  end
end
```

3. Register new serializer in `config/initializer/api_version.rb`.

```ruby
# new serializer/service may be register in new version(s) of API
Omise::API.versions.root "Aegaeon", "2014-07-27" do
  define :account,            serializer: true, service: true
  define :charge,             serializer: true, service: true
end
```

4. Create new custom serializer
   1. Named with API versioning (e.g. `Aegaeon`).
   2. Serializers are inherited from `Omise::API::Serializer` (`lib/omise/api/serializer.rb`).
   3. `initialize` with `Omise::API::Schema::Helpers.instance_variable_set` (`lib/omise/api/schema.rb`)
   4. `scoped_list` method is required and has instance variable named by serializer. (e.g. `@some_entity_list`)

```ruby
class SomeEntityListSerializer < Omise::API::Serializer
  def scoped_list
    #dynamic naming from instance_variable_set
    @some_entity_list
  end
end
```

## 1.6. Registration

1. When a team submits to enable `live` account, it starts a `registration` process.
2. `sidekiq` must be running to make full process of the `registration` service.
3. If not, the submitted `registration` may not be listed in `admin` mode of a PSP.
4. The other workaround is to visit `/admin/ekycs/:registration_uid`. The `registration_uid` can be found in `updated_registrations` table.

## 1.7. Errors

### 1.7.1. Generic Omise Errors

1. `lib/omise/api/errors.rb`
2. Error details for response [https://docs.opn.ooo/api-errors](https://docs.opn.ooo/api-errors)

```ruby
class SomeController
  def method
    Omise::API::Errors::[ErrorType], 'error message', if is_error
  end
end
```

```json
// 403 forbidden response
{
  "object": "error",
  "location": "https://www.omise.co/api-errors#not-authorized",
  "code": "not_authorized",
  "message": "error message"
}
```

---

## 1.8. Tips for development (local)

1. When developing locally, we can comment out `config/initializers/postgres_patches.rb` to show exact line of execution from the code in ruby console.

# 2. Ruby and Ruby on Rails

## 2.1. Ruby

### 2.1.1. Array

1. `<<` can be used to append a single item to an array.
2. To add multiple items once, we can use `Array#push` method instead.
3. `Array#push` method is useful when receiving multiple arguments, but will push the argument directly if it's an array.
4. To append all items individually into another array, we can use `Array#concat` instead.
5. The other workaround is to use `Array#each` method and append items from the code block.

```ruby
# append item to a ruby array
array = [1,2,3]
array << 4
# [1,2,3,4]

array.push 5
# [1,2,3,4,5]

array.concat [6]
# [1,2,3,4,5,6]

source_array = [7]
array.each { |n| array << n }
# [1,2,3,4,5,6,7]
```

### 2.1.2. Lambda function

1. Lambda function are considered anonymous functions which can be kept in a variable and passed as an argument.
2. A lambda function must be called with `Lambda#call` method or the other syntax. It doesn't work directly as regular ruby functions.
3. Lambda functions can also accept argument(s).
4. To declare a lambda function, we can use either `lambda` keyword or `->` (dash rocket) symbol.

```ruby
# lambda without argument
my_lambda = -> { puts "hello" }
my_lambda = lambda { puts "hello" }

# syntax calling lambda
my_lambda.call
my_lambda.()
my_lambda.[]
my_lambda.===

# lambda accepting argument
my_lambda_with_args = -> (v) { puts "hello "+v }

# lambda works as passed in arguments
double_it = lambda { |num| num * 2 }
triple_it = lambda { |num| num * 3 }
half_it  = lambda { |num| num / 2 }

value = 5

lambda_pipeline = [double_it, triple_it, half_it]

lambda_pipeline.each do |lmb|
  value = lmb.call(value)
end

puts value
# 5 * 2 * 3 / 2 = 15
```

5. A lambda has an execution context, represented in Ruby by a Binding object. This is the environment in which your code executes, including, among other things, local variables. This means that lambdas can be closures which allow the code in the function to access these captured local variables.

```ruby
# lambda context
def build_lambda
  output = "output from function"
  return lambda { puts output }
end

output = "output from top level"

my_lambda = build_lambda

my_lambda.call
# output from function
```

#### 2.1.2.1. ActiveRecord scopes

1. ActiveRecord scopes, used in Rails applications, are commonplace to see a lambda function.
2. If the value is not evaluated at run time, when the code executes, the function can go very wrong.
3. If this wasn’t a `lambda`, then `1.week.ago` would be evaluated when the class is loaded, rather than when the lambda runs.

```ruby
# lambda in active record in RoR
scope :new_posts, lambda { where("created_at > ?", 1.week.ago) }
```

#### 2.1.2.2. Lambda vs Proc

1. `Lambda`s enforce argument count. A lambda must be called with exactly number of arguments as it declares.
2. `Procs`, on the other hand, accept any number of arguments. If they are passed too few arguments, the unpassed arguments are set to a value of nil. If they are passed too many arguments, the extraneous arguments are dropped silently.
3. However, `lambda`s accept splat operator `*` to receive multiple arguments.
4. The behavior of `return` statement is different
   1. The return statement in a `lambda` function stops the lambda and returns control to the calling code.
   2. The return statement in a `Proc`, in contrast, returns from both the Proc and the calling code.

#### 2.1.2.3. Lambda use cases

1. ActiveRecord scopes
   1. As mentioned previously, ActiveRecord scopes are a common use of Ruby lambdas.
2. Callbacks
   1. Lambdas are great choices for simple callbacks.
   2. You can define them right before you use them or inline, without the cognitive overhead of an object or the namespace pollution of a function.
   3. In the Rails codebase, lambdas are used to capture success or failures in tests.
3. Dynamic mapping
   1. If you want to map over a collection, but the map function changes based on the input, you can use a lambda to encapsulate the changing logic.
4. Faux hash
   1. Because a lambda can be called with the syntax: `my_lambda["argument"]`, you can create a hash-like read-only object which returns values from a key based on code.

```ruby
# lambda faux hash
def build_lambda(restricted_values)
  my_hash = {}

  my_lambda = lambda do |key|
    if restricted_values.include? key
      return "n/a"
    else
      return key + key
    end
  end

  my_lambda
end

my_multiplying_hash_like_object = build_lambda(['hi'])

puts my_multiplying_hash_like_object["hi"]
# n/a
puts my_multiplying_hash_like_object["bye"]
# byebye
```

### 2.1.3. Ruby core

1. `Comparable` module can be included in a class to make its instances comparable.

### 2.1.4. Ruby method

1. `return` keyword can only be used in a ruby method.

#### 2.1.4.1. Method arguments

1. [https://www.rubyguides.com/2018/06/rubys-method-arguments/](https://www.rubyguides.com/2018/06/rubys-method-arguments/)
2. Function/method arguments can be named and become mandatory and shiftable.
3. Single asterisk `*` turns multiple arguments into an array.
4. Double asterisk `**` suggests the function/method only accepts hash for the argument and keeps it optional.
5. required -> optional -> variable -> keyword

```ruby
def testing(a, b = 1, *c, d: 1, **x)
  p a,b,c,d,x
end

testing('a', 'b', 'c', 'd', 'e', d: 2, x: 1)

# "a"
# "b"
# ["c", "d", "e"]
# 2
# {:x=>1}
```

### 2.1.5. Class attribute, property, getter, and setter

1. Ruby class has `initialize` method to have custom initializing behavior.
2. To access a property of an object, we need to setup either or both getter and setter for the property.
3. However, setting getters and setters directly is a tedious task.
4. We can use these keywords to help creator helpers `attr_reader`, `attr_writer`, `attr_accessor`

```ruby
class Book
  def initialize(title, author)
    @title = title
    @author = author
  end

  ## Getter methods
  def title
    return @title
  end
  def author
    return @author
  end

  ## Setter methods
  def title=(new_title)
    @title = new_title
  end
  def author=(new_author)
    @author = new_author
  end
end
```

4. In this case, we can use Ruby's `attr` class helper methods.
5. Using `attr_reader`

```ruby
class Book
  attr_reader :title, :author # <-- Getter methods

  def initialize(title, author)
    @title = title
    @author = author
  end

  ## Setter methods
  def title=(new_title)
    @title = new_title
  end
  def author=(new_author)
    @author = new_author
  end
end
```

6. Using `attr_writer`

```ruby
class Book
  attr_reader :title, :author # <-- Getter methods
  attr_writer :title, :author # <-- Setter methods

  def initialize(title, author)
    @title = title
    @author = author
  end

  ## Setter methods
  def title=(new_title)
    @title = new_title
  end
  def author=(new_author)
    @author = new_author
  end
end
```

7. Using `attr_accessor`

```ruby
class Book
  attr_accessor :title, :author # <-- Creates both getter and setter method

  def initialize(title, author)
    @title = title
    @author = author
  end
end
```

## 2.2. Ruby on Rails

### 2.2.1. Best practices

1. References
   1. [https://medium.com/@ishtri51/ruby-on-rails-best-practices-d7692f3c6d4d](https://medium.com/@ishtri51/ruby-on-rails-best-practices-d7692f3c6d4d)

#### 2.2.1.1. Fat Model, Skinny Controller

1. It basically means placing most of the business logic, data manipulation, and validations within the models (fat models) while keeping the controllers focused on handling request/response and routing (skinny controllers).

#### 2.2.1.2. Eager Loading

1. `N+1` Problem is an infamous problem faced by many ORM using frameworks including rails.
2. It arises when an application makes `N` additional queries to the database for a collection of records, where `N` is the number of initial records are retrieved.
3. In Rails, eager loading can be accomplished using the `includes` method or the `preload` method.
4. Note that `includes` and `preload` have slightly different behaviors.
   1. `includes` performs a left outer join to load the associated data, which can help avoid the `N+1` query problem.
   2. On the other hand, `preload` performs separate queries for each association and then associates the data in memory.

#### 2.2.1.3. Model vs Database validation

1. `ActiveRecord` includes [validation](https://guides.rubyonrails.org/active_record_validations.html) features so data offered to the database can be validated before being accepted and persisted.
2. Data-related validation (such as field constraints) should be done in the database.
3. Any validation that is following some business rules (such as email validation) should be done in the model.

#### 2.2.1.4. Prevent SQL injection

1. SQLi (SQL Injection) is a type of security vulnerability that occurs when an application fails to properly sanitize or validate user-supplied input before incorporating it into SQL queries.
2. Use dynamic attribute based finder

```ruby
# Bad Practice
User.where("name = '#{params[:name]}'") # SQL Injection!

# Good Practice
User.find_by_name(name) # dynamic finder
```

#### 2.2.1.5. Enums

1. An `enum` in Rails is a data type that represents a set of named values. The values of an `enum` are stored as integers in the database, but they can be accessed by name in Ruby code. This makes it easier to work with `enum`s in your Rails application.

```ruby
class AddStatusToOrders < ActiveRecord::Migration[6.0]
  def change
    add_column :orders, :status, :integer, default: 0
  end
end

# Model: Order.rb
class Order < ApplicationRecord
  enum status: {
    placed: 0,
    packed: 1,
    shipped: 2,
    delivered: 3
  }
end

# work with status
order = Order.create(status: :placed)

# Check the status of the order
order.status == :placed
# or
order.placed?

# Change the status of the order
order.status = :packed
# or
order.packed!

# Scope
order.shipped # use instead Order.where(status: 'shipped')
```

2. You can use `_prefix` (or `suffix`) in enum definition, so that all the helpers can be prefixed(or suffixed) by the column name.

```ruby
# Model: Order.rb
enum status: {
    placed: 0,
    packed: 1,
    shipped: 2
}, _prefix: true

order.status_placed? # status == 'placed'
order.status_packed! # update(status: :packed)
order.status_shipped # User.where(status: :shipped)
```

#### 2.2.1.6. Use Time.current instead of Time.now

1. keep in mind that `Time.now` returns a Time object representing the current time in the default system time zone. To get the current time in application’s configured time zone you have to use `Time.zone.now`.
2. Unlike `Time.now`, which uses the default system time zone, `Time.current` takes into account the time zone configured for the Rails application.

```ruby
# config/application.rb
config.time_zone = "Africa/Nairobi"

# Fetch current time in default system's zone
current_time = Time.now # 2023-07-13 10:57:05.751843358 +0530

# Fetch current time in application set time zone
current_time = Time.zone.now # Thu, 13 Jul 2023 08:27:07.607994466 EAT +03:00
# or better
current_time = Time.current # Thu, 13 Jul 2023 08:27:10.015227438 EAT +03:00
```

#### 2.2.1.7. Concerns, Services and Helpers

##### 2.2.1.7.1. Concerns

1. In Rails, `Concerns` are modules that encapsulate reusable code and behavior that can be included in multiple classes or modules. It extends `ActiveSupport::Concern` module.
2. They help keep models focused and avoid excessive code duplication. `Concerns` are implemented using Ruby modules and included in classes using the `include` keyword.

```ruby
# app/models/concerns/trashable.rb
module Trashable
  extend ActiveSupport::ConConcerncern

  included do
    scope :existing, -> { where(trashed: false) }
    scope :trashed, -> { where(trashed: true) }
  end

  def trash
    update_attribute :trashed, true
  end
end

# Model song.rb
class Song < ApplicationRecord
  include Trashable

  has_many :authors

  # ...
end

# Model album.rb
class Album < ApplicationRecord
  include Trashable

  has_many :authors

  def featured_authors
    authors.where(featured: true)
  end

  # ...
end
```

##### 2.2.1.7.2. Services

1. As your application grows, you may begin to see domain/business logic littered across the models and the controller. Such logics do not belong to either the controller or the model, so they make the code difficult to re-use and maintain.
2. Service objects are plain old Ruby objects (PORO’s) that do one thing .They encapsulate a set of business logic, moving it out of models and controllers and into a more focused setting.

```ruby
# app/services/create_user_service.rb

class CreateUserService
  attr_reader :name, :email, :pass

  def initialize(name, email, pass)
    @name = name
    @email = email
    @pass = pass
  end

  def call
    prev_user = User.find_by email: @email
    raise StandardError, 'User already exists' if prev_user.present?

    User.create(name: @name, email: @email, pass: @pass)
  end
end


# Controller: users_controller.rb

class UserController < ApplicationController
  def create
    begin
      ActiveRecord::Base.transaction do
        # Create User
        user = CreateUserService.new(params[:name], params[:email], params[:pass]).call
        render json: { user: user }
      end
    rescue StandardError => e
      render json: { error: e }
    end
  end
end
```

##### 2.2.1.7.3. Helpers

1. A helper is a method that is used in your Rails **views** to share reusable code. Helper methods are defined within modules called `Helper Module` and are automatically made available to the corresponding views.

```ruby
# app/helpers/users_helper.rb

module UsersHelper
  def full_name(user)
    "#{user.first_name} #{user.last_name}"
  end
end

# View: users/show.html.erb

<h1><%= full_name(@user) %></h1>
```

#### 2.2.1.8. Helpful Gems

- `oj`: Library for both parsing and generating JSON with a ton of options.
- `rack-mini-profiler`: Middleware that provides performance profiling and diagnostics for Rack-based applications.
- `pry`: Debug APIs in real time.
- `rubocop`: Static code analyzer and code formatter enforcing Ruby code style conventions.
- `bullet`: Helps detecting and alerts N+1 database query issues in Rails applications.
- `devise`: A flexible and secure authentication solution for Ruby on Rails.
- `cancancan`: Authorization library that restricts user access based on user roles and permissions.
- `annotate`: Automatically adds schema information as comments to your models and specs.
- `discard`: Soft-delete implementation for ActiveRecord models.
- `sidekiq`: Simple and efficient background job processing for Ruby.
- `friendlyId`: Gem for creating human-readable URLs by using slugs for ActiveRecord models.
- `paperclip`: Gem for handling file attachments in Rails applications.

### 2.2.2. App convention

#### 2.2.2.1. Auto loading

1. Rails applications do not use require to load application code.
2. You only need require calls for two use cases:
   1. To load files under the `lib` directory.
   2. To load gem dependencies that have `require: false` in the Gemfile.

### 2.2.3. View

1. For UIs built from similar code, we can create `partials` to share these UI components in between views.
2. `partials` are named with a prefix with underscore `_`.
3. A partial can be used with keyword `render` in the other view files.

```erb
# app/views/articles/_form.html.erb
<%= form_with model: article do |form| %>
  <div>
    <%= form.label :title %><br>
    <%= form.text_field :title %>
    <% article.errors.full_messages_for(:title).each do |message| %>
      <div><%= message %></div>
    <% end %>
  </div>

  <div>
    <%= form.label :body %><br>
    <%= form.text_area :body %><br>
    <% article.errors.full_messages_for(:body).each do |message| %>
      <div><%= message %></div>
    <% end %>
  </div>

  <div>
    <%= form.submit %>
  </div>
<% end %>
```

4. A partial's filename must be prefixed with an underscore, e.g. `_form.html.erb`. But when rendering, it is referenced without the underscore, e.g. render "`form`".

```erb
# app/views/articles/edit.html.erb
<h1>Edit Article</h1>

<%= render "form", article: @article %>
```

### 2.2.4. Router

1. We can change router default to show content of a specific route.

```ruby
# config/routes.rb
Rails.application.routes.draw do
  root "articles#index"

  get "/articles", to: "articles#index"
end
```

2. Though we may create different API endpoints to `CRUD` resources, it can be very tedious to work on each of them with `RESTful` pattern.
3. We can use `resources` keyword in router which creates several endpoints by following RESTful styles to implement `CRUD`.

```bash
# list all routes
bin/rails routes

# table of routes
      Prefix Verb   URI Pattern                  Controller#Action
        root GET    /                            articles#index
    articles GET    /articles(.:format)          articles#index
 new_article GET    /articles/new(.:format)      articles#new
     article GET    /articles/:id(.:format)      articles#show
             POST   /articles(.:format)          articles#create
edit_article GET    /articles/:id/edit(.:format) articles#edit
             PATCH  /articles/:id(.:format)      articles#update
             DELETE /articles/:id(.:format)      articles#destroy
```

4. Besides, `resources` method also set up URL and path helper methods that we can use to keep the code from depending on a specific route configuration.
5. The values in the "`Prefix`" column above plus a suffix of `_url` or `_path` form the names of these helpers.
6. For example, the `article_path` helper returns "`/articles/#{article.id}`" when given an article.
7. In view file, we can use the helper function to generate links for anchor tag `<a>`.

```erb
<!-- app/views/articles/index.html.erb -->

<h1>Articles</h1>

<ul>
  <% @articles.each do |article| %>
    <li>
      <a href="<%= article_path(article) %>">
        <%= article.title %>
      </a>
    </li>
  <% end %>
</ul>
```

8. On the other hand, we can use `link_to` shorthand in `erb` file to generate a link.
9. The `link_to` helper renders a link with its 1st argument as the link's text and its 2nd argument as the link's destination.
10. If we pass a model object as the 2nd argument, link_to will call the appropriate path helper to convert the object to a path.

```erb
<!-- app/views/articles/index.html.erb -->

<h1>Articles</h1>

<ul>
  <% @articles.each do |article| %>
    <li>
      <%= link_to article.title, article %>
    </li>
  <% end %>
</ul>
```

### 2.2.5. Controller

1. Generating `controller`

```bash
bin/rails generate controller Articles index --skip-routes

# created along with article_controller.rb
create  app/controllers/articles_controller.rb
invoke  erb
create    app/views/articles
create    app/views/articles/index.html.erb
invoke  test_unit
create    test/controllers/articles_controller_test.rb
invoke  helper
create    app/helpers/articles_helper.rb
invoke    test_unit
```

#### 2.2.5.1. Conventions

1. When an action (method) of a controller class is empty, rails will automatically render a view that matches the name of the controller.
2. In the following class, the `index` method will try to render `app/views/articles/index.html.erb` by default.

```ruby
# config/routes.rb
Rails.application.routes.draw do
  get "/articles", to: "articles#index"
end

# articles_controller.rb
class ArticlesController < ApplicationController
  def index
  end
end
```

### 2.2.6. Model

1. A `model` is a Ruby class that is used to represent data.
2. Additionally, models can interact with the application's database through a feature of Rails called `Active Record`.
3. When generating a `model`, several other files are created.

```bash
bin/rails generate model Article title:string body:text

# created along with models/article.rb
invoke  active_record
create    db/migrate/<timestamp>_create_articles.rb
create    app/models/article.rb
invoke    test_unit
create      test/models/article_test.rb
create      test/fixtures/articles.yml
```

#### 2.2.6.1. Database migration

1. Migrations are used to alter the structure of an application's database.
2. In Rails applications, migrations are written in Ruby so that they can be database-agnostic.

```ruby
# migration file after creating a new model
# db/migrate/<timestamp>_create_articles.rb
class CreateArticles < ActiveRecord::Migration[7.1]
  def change
    create_table :articles do |t|
      t.string :title
      t.text :body

      t.timestamps
    end
  end
end
```

3. By having the migration file, we can run `bin/rails db:migrate` to update database schema.
4. After migrating changes to database, we can run `bin/rails console` to work on `irb` and work with database with the models we created.
5. We can use some generic model methods such as `Model#find` and `Model#all`.
6. Note that `Model#all` returns an `ActiveRecord::Relation` object which is an array of entities with some other useful methods.

#### 2.2.6.2. Create new entity

##### 2.2.6.2.1. Using a form_builder

1. In controller, we may have a `new` and `create` method to create a new entity.
   1. `new` for user to request and fill a form to create an entity.
   2. `create` is the endpoint to be called to create an entity with the data from the `new` form.
2. `redirect_to` will cause the browser to make a new request, whereas render renders the specified view for the current request.
3. It is important to use `redirect_to` after mutating the database or application state.
4. Otherwise, if the user refreshes the page, the browser will make the same request, and the mutation will be repeated.

```ruby
# app/controllers/articles_controller.rb
class ArticlesController < ApplicationController
  def index
    @articles = Article.all
  end

  def show
    @article = Article.find(params[:id])
  end

  def new
    @article = Article.new
  end

  def create
    @article = Article.new(title: "...", body: "...")

    if @article.save
      # redirect to make a new request and
      # prevent the same mutation
      redirect_to @article
    else
      render :new, status: :unprocessable_entity
    end
  end
end
```

5. To generate a form for `new` method to collect value to create a new entity, we can use `form builder`.
6. The `form_with` helper method instantiates a form builder.

```erb
# app/views/articles/new.html.erb
<h1>New Article</h1>

<%= form_with model: @article do |form| %>
  <div>
    <%= form.label :title %><br>
    <%= form.text_field :title %>
  </div>

  <div>
    <%= form.label :body %><br>
    <%= form.text_area :body %>
  </div>

  <div>
    <%= form.submit %>
  </div>
<% end %>
```

##### 2.2.6.2.2. Using strong parameters

7. Submitted form data is put into the `params` Hash, alongside captured route parameters.
8. Thus, the create action can access the submitted title via `params[:article][:title]` and the submitted body via `params[:article][:body]`.
9. Though we can pass a single Hash that contains the values, we must still specify what values are allowed in that Hash.
10. Otherwise, a malicious user could potentially submit extra form fields and overwrite private data.
11. In such cases, we can create a new private method `article_params` in the controller.

```ruby
# app/controllers/articles_controller.rb
class ArticlesController < ApplicationController
  def index
    @articles = Article.all
  end

  def show
    @article = Article.find(params[:id])
  end

  def new
    @article = Article.new
  end

  def create
    @article = Article.new(article_params)

    if @article.save
      redirect_to @article
    else
      render :new, status: :unprocessable_entity
    end
  end

  private
    # strong parameters
    def article_params
      params.require(:article).permit(:title, :body)
    end
end
```

##### 2.2.6.2.3. Validations and displaying error message

1. Besides setting up process for creating an entity on `view` and `controller`, we can set validation process on `model`.
2. Validations are rules that are checked before a model object is saved.
3. If any of the checks fail, the save will be aborted, and appropriate error messages will be added to the errors attribute of the model object.
4. `ActiveRecord` automatically defines model attributes for every table column, so you don't have to declare those attributes in your model file.

```ruby
# app/models/article.rb
# ActiveRecord defines models attribute and includes all column from the table
class Article < ApplicationRecord
  validates :title, presence: true
  validates :body, presence: true, length: { minimum: 10 }
end
```

1. Thus in the collecting form `new.html.erb` with `ArticlesController#new` method, we can refer the error form the model object.
2. The `full_messages_for` method returns an array of user-friendly error messages for a specified attribute. If there are no errors for that attribute, the array will be empty.

```erb
# app/views/articles/new.html.erb
<h1>New Article</h1>

<%= form_with model: @article do |form| %>
  <div>
    <%= form.label :title %><br>
    <%= form.text_field :title %>
    <% @article.errors.full_messages_for(:title).each do |message| %>
      <div><%= message %></div>
    <% end %>
  </div>

  <div>
    <%= form.label :body %><br>
    <%= form.text_area :body %><br>
    <% @article.errors.full_messages_for(:body).each do |message| %>
      <div><%= message %></div>
    <% end %>
  </div>

  <div>
    <%= form.submit %>
  </div>
<% end %>
```

#### 2.2.6.3. Update an entity

1. The implementation to edit/update an existing entity is quite similar to create a new one.
2. From the previous example, we can have 2 more methods `edit` and `update` in `ArticlesController`.

```ruby
# app/controllers/articles_controller.rb
class ArticlesController < ApplicationController
  def index
    @articles = Article.all
  end

  def show
    @article = Article.find(params[:id])
  end

  def new
    @article = Article.new
  end

  def create
    @article = Article.new(article_params)

    if @article.save
      redirect_to @article
    else
      render :new, status: :unprocessable_entity
    end
  end

  def edit
    @article = Article.find(params[:id])
  end

  def update
    @article = Article.find(params[:id])

    if @article.update(article_params)
      redirect_to @article
    else
      render :edit, status: :unprocessable_entity
    end
  end

  private
    def article_params
      params.require(:article).permit(:title, :body)
    end
end
```

#### 2.2.6.4. Model with reference

1. After creating `Article` entity, we can have `Comment` which belongs to each article.

```bash
bin/rails generate model Comment commenter:string body:text article:references
```

2. By giving references, `Comment` will have an attribute `belongs_to`.
3. The `:references` keyword used in the shell command is a special data type for models.
4. It creates a new column on your database table with the provided model name appended with an `_id` that can hold integer values.
5. In this case, we set up an one to many relationship. That is an `Article` may have one or more `Comment`s.

```ruby
# app/models/comment.rb
class Comment < ApplicationRecord
  belongs_to :article
end

# app/models/article.rb
class Article < ApplicationRecord
  has_many :comments

  validates :title, presence: true
  validates :body, presence: true, length: { minimum: 10 }
end
```

6. For the API routes, since `Comment` must be attached to an `Article`.
7. This creates `Comment`s as a nested resource within `Article`s. This is another part of capturing the hierarchical relationship that exists between `Article`s and `Comment`s.

```ruby
# config/routes.rb
Rails.application.routes.draw do
  root "articles#index"

  resources :articles do
    resources :comments
  end
end
```

8. To allow users adding `Comment` to an `Article`, we can update `app/views/articles/show.html.erb`.
9. This adds a form on the `Article` show page that creates a new comment by calling the `CommentsController` `create` action.
10. The `form_with` call here uses an array, which will build a nested route, such as `/articles/1/comments`.

```erb
# app/views/articles/show.html.erb
<h1><%= @article.title %></h1>

<p><%= @article.body %></p>

<ul>
  <li><%= link_to "Edit", edit_article_path(@article) %></li>
  <li><%= link_to "Destroy", article_path(@article), data: {
                    turbo_method: :delete,
                    turbo_confirm: "Are you sure?"
                  } %></li>
</ul>

<h2>Add a comment:</h2>
<%= form_with model: [ @article, @article.comments.build ] do |form| %>
  <p>
    <%= form.label :commenter %><br>
    <%= form.text_field :commenter %>
  </p>
  <p>
    <%= form.label :body %><br>
    <%= form.text_area :body %>
  </p>
  <p>
    <%= form.submit %>
  </p>
<% end %>
```

9.

### 2.2.7. Concerns

1. A `concern` is a module that you extract to split the implementation of a class or module in coherent chunks, instead of having one big class body. The API surface is the same one, they just help organize the code
2. [https://www.akshaykhot.com/how-rails-concerns-work-and-how-to-use-them/](https://www.akshaykhot.com/how-rails-concerns-work-and-how-to-use-them/)

### 2.2.8. Active Record (ORM)

#### 2.2.8.1. Pluck

1. [https://apidock.com/rails/ActiveRecord/Calculations/pluck](https://apidock.com/rails/ActiveRecord/Calculations/pluck)
2. `Record.pluck` works as a shortcut to load only the selected attributes of the entity.
3. Pluck returns an Array of attribute values type-casted to match the plucked column names, if they can be deduced.
4. Plucking an SQL fragment returns String values by default.

```ruby
# Use pluck
Person.pluck(:name)

# NOT
Person.all.map(&:name)

Person.pluck(:name)
# SELECT people.name FROM people
# => ['David', 'Jeremy', 'Jose']

Person.pluck(:id, :name)
# SELECT people.id, people.name FROM people
# => [[1, 'David'], [2, 'Jeremy'], [3, 'Jose']]

Person.distinct.pluck(:role)
# SELECT DISTINCT role FROM people
# => ['admin', 'member', 'guest']

Person.where(age: 21).limit(5).pluck(:id)
# SELECT people.id FROM people WHERE people.age = 21 LIMIT 5
# => [2, 3]

Person.pluck('DATEDIFF(updated_at, created_at)')
# SELECT DATEDIFF(updated_at, created_at) FROM people
# => ['0', '27761', '173']
```

### 2.2.9. Meta programming

#### 2.2.9.1. Instance variable set

1. Imagine that you have a class constructor with too many arguments. You want to assign each argument to an instance variable of that class.
2. Use the `Object#instance_variable_set` method provided by Ruby to initialize instance variables.
   1. `instance_variable_set` sets the instance variable named by symbol to the given object.
   2. This may circumvent the encapsulation intended by the author of the class, so it should be used with care.
   3. The variable does not have to exist prior to this call.
   4. If the instance variable name is passed as a string, that string is converted to a symbol.

```ruby
class Person
  def initialize(name, age, address, data, ...)
    @name = name
    @age = age
    @address = address
    @data = data

    # remaining assignments
  end
end

# refactored using instance_variable_set
class Person
  def initialize(assigns)
    assign(assigns)
  end

  private
    def assign(attributes)
      attributes.each do |key, value|
        instance_variable_set("@#{key}", value)
      end
    end
end

person = Person.new(name: 'John Doe', age: 31)
puts person.instance_variable_get("@name")  # John Doe
```
