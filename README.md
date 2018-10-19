# rails-auto-docs


## Alternatives

### apipie-rails

https://github.com/Apipie/apipie-rails

#### Example

```ruby
# The simplest case: just load the paths from routes.rb
api!
def index
end

api :GET, '/users/:id'
param :id, :number
def index
end

# Most complex example
api :GET, "/users/:id", "Show user profile"
show false
error :code => 401, :desc => "Unauthorized"
error :code => 404, :desc => "Not Found", :meta => {:anything => "you can think of"}
param :session, String, :desc => "user is logged in", :required => true
param :regexp_param, /^[0-9]* years/, :desc => "regexp param"
param :array_param, [100, "one", "two", 1, 2], :desc => "array validator"
param :boolean_param, [true, false], :desc => "array validator with boolean"
param :proc_param, lambda { |val|
  val == "param value" ? true : "The only good value is 'param value'."
}, :desc => "proc validator"
param :param_with_metadata, String, :desc => "", :meta => [:your, :custom, :metadata]
returns :code => 200, :desc => "a successful response" do
   property :value1, String, :desc => "A string value"
   property :value2, Integer, :desc => "An integer value"
   property :value3, Hash, :desc => "An object" do
     property :enum1, ['v1', 'v2'], :desc => "One of 2 possible string values"
   end
end
description "method description"
formats ['json', 'jsonp', 'xml']
meta :message => "Some very important info"
example " 'user': {...} "
see "users#showme", "link description"
see :link => "users#update", :desc => "another link description"
def show
  #...
end
```

### rspec_api_documentation

https://github.com/zipmark/rspec_api_documentation

#### Example

```ruby
 # spec/acceptance/orders_spec.rb
  require 'rails_helper'
  require 'rspec_api_documentation/dsl'
  resource 'Orders' do
    explanation "Orders resource"
    
    header "Content-Type", "application/json"

    get '/orders' do
      # This is manual way to describe complex parameters
      parameter :one_level_array, type: :array, items: {type: :string, enum: ['string1', 'string2']}, default: ['string1']
      parameter :two_level_array, type: :array, items: {type: :array, items: {type: :string}}
      
      let(:one_level_array) { ['string1', 'string2'] }
      let(:two_level_array) { [['123', '234'], ['111']] }

      # This is automatic way
      # It's possible because we extract parameters definitions from the values
      parameter :one_level_arr, with_example: true
      parameter :two_level_arr, with_example: true

      let(:one_level_arr) { ['value1', 'value2'] }
      let(:two_level_arr) { [[5.1, 3.0], [1.0, 4.5]] }

      context '200' do
        example_request 'Getting a list of orders' do
          expect(status).to eq(200)
        end
      end
    end

    put '/orders/:id' do

      with_options scope: :data, with_example: true do
        parameter :name, 'The order name', required: true
        parameter :amount
        parameter :description, 'The order description'
      end

      context "200" do
        let(:id) { 1 }

        example 'Update an order' do
          request = {
            data: {
              name: 'order',
              amount: 1,
              description: 'fast order'
            }
          }
          
          # It's also possible to extract types of parameters when you pass data through `do_request` method.
          do_request(request)
          
          expected_response = {
            data: {
              name: 'order',
              amount: 1,
              description: 'fast order'
            }
          }
          expect(status).to eq(200)
          expect(response_body).to eq(expected_response)
        end
      end

      context "400" do
        let(:id) { "a" }

        example_request 'Invalid request' do
          expect(status).to eq(400)
        end
      end
      
      context "404" do
        let(:id) { 0 }
        
        example_request 'Order is not found' do
          expect(status).to eq(404)
        end
      end
    end
  end
  ```
  
  ### swagger-docs
  
  https://github.com/richhollis/swagger-docs
  
  ```ruby
  class Api::V1::UsersController < ApplicationController
  .....
  # POST /users
  swagger_api :create do
    summary "To create user"
    notes "Implementation notes, such as required params, example queries for apis are written here."
    param :form, "user[name]", :string, :required, "Name of user"
    param :form, "user[age]", :integer, :optional, "Age of user"
    param_list :form, "user[status]", :string, :required, "Status of user, can be active or inactive"
    response :success
    response :unprocessable_entity
    response :500, "Internal Error"
  end
  def create
    @user = User.new(user_params)
    if @user.save
      render json: @user, status: :created
    else
      render json: @user.errors, status: :unprocessable_entity
    end
  end
  .....
end
```

### rswag

https://github.com/domaindrivendev/rswag

```ruby
require 'swagger_helper'

describe 'Blogs API' do

  path '/blogs' do

    post 'Creates a blog' do
      tags 'Blogs'
      consumes 'application/json', 'application/xml'
      parameter name: :blog, in: :body, schema: {
        type: :object,
        properties: {
          title: { type: :string },
          content: { type: :string }
        },
        required: [ 'title', 'content' ]
      }

      response '201', 'blog created' do
        let(:blog) { { title: 'foo', content: 'bar' } }
        run_test!
      end

      response '422', 'invalid request' do
        let(:blog) { { title: 'foo' } }
        run_test!
      end
    end
  end
```


