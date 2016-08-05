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

## Key Rails Methods & Helpers
`form_for` - form builder for templates in views. See example in `app/views/articles/new.html.erb`.  
`link_to` - creates a hyperlink based on text to display and where to go. See example in `app/views/welcome/index.html.erb`  
`pluralize` - a rails helper that takes a number and a string as its arguments. If the number is greater than one, the string will be automatically pluralized.

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
* We need to get a listing of all the articles. To do so, we need to check the routes again. `rails routes`. Look for **articles GET***
```
       Prefix Verb   URI Pattern                  Controller#Action
welcome_index GET    /welcome/index(.:format)     welcome#index
     articles GET    /articles(.:format)          articles#index
              POST   /articles(.:format)          articles#create
  new_article GET    /articles/new(.:format)      articles#new
 edit_article GET    /articles/:id/edit(.:format) articles#edit
      article GET    /articles/:id(.:format)      articles#show
              PATCH  /articles/:id(.:format)      articles#update
              PUT    /articles/:id(.:format)      articles#update
              DELETE /articles/:id(.:format)      articles#destroy
         root GET    /                            welcome#index
```
* Add the corresponding index action (`articles#index`) for that route (`articles GET`) in the `ArticlesController`. This corresponding action is the `def index`... see below:
```ruby
class ArticlesController < ApplicationController
  def index     # add this method for the index action
    @articles = Article.all
  end
 
  def show
    @article = Article.find(params[:id])
  end
 
  def new
  end
  # more code below ...
  ```

  * Add the view for this action in `app/views/articles/index.html.erb`

### Adding Links
* Since we can now *create, show, and list* articles, we can add some links for navigation. Edit `app/views/welcome/index.html`:
```
<h1>Hello, Rails!</h1>
<%= link_to 'My Blog', controller: 'articles' %>
```

### Adding Some Validation
* Edit `app/models/article.rb`
```ruby
class Article < ApplicationRecord
  validates :title, presence: true, length: { minumum: 5 }
end
```
* These changes will ensure that all articles have a title that is at least 5 characters long. Rails can validate a variety of conditions in a model, including the presence or uniqueness of columns, their format, and the existence of associated objects. See [Active Record Validations](http://guides.rubyonrails.org/active_record_validations.html)

* Update `ArticlesController`
```ruby
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
      render 'new'
    end
  end

  private
    def article_params
      params.require(:article).permit(:title, :text)
    end
end
```

* Make sure the user cannot submit an article without a title. Modify `app/views/articles/new.html.erb`

### Updating Articles
* Add an `edit` action to the `ArticlesController`. This is generally placed between the `new` and `create` actions.
```ruby
def new
    @article = Article.new
  end

  def edit                                  # add the edit action
    @article = Article.find(params[:id])
  end

  def create
    @article = Article.new(article_params)

    if @article.save
      redirect_to @article
    else
      render 'new'
    end
  end
  ```

* Create `app/views/articles/edit.html.erb`
```ruby
<h1>Editing Article</h1>

<%= form_for :article, url: article_path(@article), method: :patch do |f| %>

  <% if @article.errors.any? %>
    <div id="error_explanation">
      <h2>
        <%= pluralize(@article.errors.count, "error") %>
        prohibited this article from being saved:
      </h2>
      <ul>
        <% @article.errors.full_messages.each do |msg| %>
          <li><%= msg %></li>
        <% end %>
      </ul>
    </div>
  <% end %>

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

<%= link_to 'Back', articles_path %>
```

* The `method: :patch` option tells Rails that we want this form to be submitted via the `PATCH HTTP` method which is used to **update** resources according to the `REST` protocol.

* Create an `update` action in `app/controllers/articles_controller.rb`. Add it between the `create` action and `private method`

* Show a link to the `edit` action in the list of all the articles. Add it to `app/views/articles/index.html`

* Add the `edit` action to the `app/views/articles/show.html.erb` as well...

### Using Partials to clean up duplication in views
* make a new file `app/views/articles/_form.html.erb`
* edit the following files: `new.html.erb`, `edit.html.erb` and use the `render `form` helper.

* Add the destroy action in the `articles_controller`
* Add the `Destroy` link to `app/views/articles/index.html.erb`

### 6. Adding a Second Model
