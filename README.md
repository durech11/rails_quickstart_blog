# Rails Quick Start Guide | Blog

## About
This app follows the quickstart guide from  
http://guides.rubyonrails.org/getting_started.html

## Key Concepts/Commands
`rails new -h` - see all command line options for rails application
`rails generate controller ControllerName controller_action`  



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
