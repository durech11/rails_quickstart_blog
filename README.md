# Rails Quick Start Guide | Blog

## About
This app follows the quickstart guide from  
http://guides.rubyonrails.org/getting_started.html

## Key Rails Commands
`rails new -h` - see all command line options for rails application
`rails routes` - display defined routes in application
`rails generate controller ControllerName controller_action` - generate controller    
`rails generate model ModelName attribute:type` - generate model  
`rails db:migrate` - run the migration *locally*
`rails db:migrate RAILS_ENV=production` - run the migration in production  

## Key Rails Methods
`form_for` - form builder for templates in views. See example in `app/views/articles/new.html.erb`

## Key Concepts/Definitions
**routes** - connects incoming requests to controllers and actions
**resource** - a collection of similar objects (articles, people, animals). Can create, read, update, and destroy items in a resource; CRUD operations. Rails have a `reources` method is used to declare a standard REST resource.
**controller** - a class that inherit from `ApplicationController`. Inside this controller class, we define methods that will become actions for this controller. These actions perform CRUD operations.  
**migrations** - Ruby classes that are designed to make it easy to create and modify database tables. Migration filenames include a timestamp to ensure that they are processed in the order that they were created. Located in `db/migrate/YYYYMMDDHHMMSS*.rb`

## Setup

### Create the Blog Application
`rails new blog`

### Say 'Hello'
* Need to create at least a controller and a view
`bin/rails generate controller Welcome index`  
will generate a controller called *Welcome* with an action called *index*
* Edit the view in `app/views/welcome/index.html.erb`, replace all code 
with `<h1>Hello, Rails!</h1>`
* Change the home page in the routes. Open `config/routes.rb`. Add:
`root 'welcome#index'`

### Add Resources (Articles)
* Edit `config/routes` by adding `resources :articles` (note: plural form in articles).
* Generate Articles controller  
`bin/rails generate controller Articles`.
* Inside the `app/controllers/articles_controller.rb`, define a new method
```ruby
class ArticlesController < ApplicationController
  def new # add this new method
  end
end
```
* Since there is no associated view yet for the articles controller when we visit `localhoset/articles/new`, we need to manually create it. Make a new **html.erb** file in `app/views/articles/` called `new.html.erb`. Add this 
```html
<h1>New Article</h1>
```

### Form Builder
* Use the form_for method from *Form Builder* in `app/views/articles/new.html.erb`
```ruby
<h1>New Article</h1>

<%= form_for :article do |f| %>
  <p>
    <%= f.label :title %><br>
    <%= f.text_field :title %>
  </p>

  <p>
    <%= f.label :text %><br>
    <%= f.text_area :text %>
  </p>

  <p>
    <%= f.submit %>
  </p>
<% end %>
```
* The form needs the correct URL to point to. Edit the `form_for` line to show:
`<%= form_for :article, url: articles_path do |f| %>`
* The `articles_path` helper tells Rails to point the form to the URI Patern associated with the `articles` prefix; and the form will (by default) send a `POST` request to that route.

### Creating Articles
* In order for the submit button in the form to work, we need to create a `create` action in the `ArticlesController`. Edit `app/controllers/application_controller.rb`  
```ruby
class ArticlesController < ApplicationController
  def new
  end

  def create                                # add this create method
    render plain: params[:article].inspect  # add this render params
  end
end
```

* Create the Article model
`bin/rails generate model Article title:string text:text`
This generates a model named *Article* with a *title attribute* of type string, and a *text attribute* of type text.

* The generated model also created a *database migration* file inside the `db/migrate` directory. 

* Run the database migration: `bin/rails db:migrate`

* In order to save the data to the database, we need to change the `ArticlesController` create method. We also define a private method to whitelist the article_params for security.
```ruby
class ArticlesController < ApplicationController
  def new
  end

  def create
    @article = Article.new(article_params)

    @article.save
    redirect_to @article
  end

  private
    def article_params
      params.require(:article).permit(:title, :text)
    end
end
  ```
* Create a new file: `app/views/articles/show.html.erb`
```ruby
<p>
  <strong>Title:</strong>
  <%= @article.title %>
</p>
 
<p>
  <strong>Text:</strong>
  <%= @article.text %>
</p>
```
### Listing All Articles
