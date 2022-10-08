# Rails Tutorial

- Ruby 3.0.2
- Rails 7.0.1

---

This tutorial will guide you through the steps to get a minimal Rails application up and running without using the `rails new` command.

- First, create a new directory for the app and move into it:

```
mkdir my-app && cd $_
```

If you try running the `rails s` command now, you should see the help text for the Rails gem output in the terminal:

```
Usage:
  rails new APP_PATH [options]
```

This is because the `rails s` command expects certain files to exist within the directory it is run in in order to run. These files are usually created when you run the `rails new` command, or when you clone a repo from GitHub, etc. Since the current directory is empty, the `rails s` command cannot run.

Obviously, to get around this problem, you need to add the missing files. However, as mentioned above, instead of generating them using the `rails new` command, you will add them manually one-by-one.

The first file is `bin/rails` (that is, a file called `rails` inside a directory called `bin`).

- First, create the `bin` directory:

```
mkdir bin
```

- Next, create a file called `rails` inside that directory:

```
touch bin/rails
```

---

**NOTE**

Since you will be creating a lot of directories and files, it will probably be easier to do so using a code editor such as Sublime Text or VS Code.

---

- Add the following contents to `bin/rails`:

```
#!/usr/bin/env ruby
# frozen_string_literal: true

APP_PATH = File.expand_path("../config/application", __dir__)

require_relative "../config/boot"

require "rails/commands"
```

- Finally, change the permissions for the file:

```
chmod +x bin/rails
```

---

**NOTE**

Now that you have added the first file, you can try running the `rails s` command again and use the output to figure out what the next step might be. You should follow this iterative approach until you have the `rails s` command working properly.

---


If you run the `rails s` command now, you should see the following error:

```
cannot load such file -- my-app/config/boot (LoadError)
```

This is because this file is required on line 6 in `bin/rails`.

- Create `config/boot.rb` with the following contents:

```
# frozen_string_literal: true

ENV["BUNDLE_GEMFILE"] ||= File.expand_path("../Gemfile", __dir__)

require "bundler/setup"
require "bootsnap/setup"
```

---

If you run the `rails s` command now, you should see the following error:

```
my-app/Gemfile not found
```

This is because this file is specified on line 3 in `config/boot.rb` and is referenced while booting the app.

- Create `Gemfile` with the following contents:

```
# frozen_string_literal: true

source "https://rubygems.org"

git_source(:github) { |repo| "https://github.com/#{repo}.git" }

ruby "3.0.2"
```

- Run `bundle install` to generate `Gemfile.lock`.

---

If you run the `rails s` command now, you should see the following error:

```
cannot load such file -- bootsnap/setup (LoadError)
```

This is because `bootsnap/setup` is required on line 6 in `config/boot.rb`.

- Add the following to `Gemfile`, and run `bundle install`:

```
gem "bootsnap", require: false
```

---

If you run the `rails s` command now, you should see the following error:

```
cannot load such file -- rails/commands (LoadError)
```

This is because `rails/commands` is required on line 8 in `bin/rails`.

- Add the following to `Gemfile`, and run `bundle install`:

```
gem "rails", "~> 7.0.1"
```

---

**NOTE**

When you ran the `rails s` command, the `tmp/cache` directory was generated automatically.

---

If you run the `rails s` command now, you should see the following error:

```
cannot load such file -- my-app/config/application (LoadError)
```

This is because this file is specified on line 4 in `bin/rails` and is referenced while booting the app.

- Create `config/application.rb` with the following contents:

```
# frozen_string_literal: true

require_relative "boot"

require "rails"

Bundler.require(*Rails.groups)

module MyApp
  class Application < Rails::Application
    config.load_defaults 7.0
  end
end
```

---

If you run the `rails s` command now, you should see the following error:

```
Could not find server ''.
```

This is because you have not installed an application server to run the Rails application. By default, Rails uses Puma.

- Add the following to `Gemfile`, and run `bundle install`:

```
gem "puma", "~> 5.0"
```

---

If you run the `rails s` command now, you should see the following error:

```
configuration config.ru not found
```

This is because Rails is built on top of Rack, and all Rack applications require `config.ru`. This file is automatically picked up by Puma when you run the `rails s` command.

- Create `config.ru` with the following contents:

```
# frozen_string_literal: true

require_relative "config/environment"

run Rails.application

Rails.application.load_server
```

---

**NOTE**

When you ran the `rails s` command, the `tmp/pids` and `tmp/sockets` directories were generated automatically.

---

If you run the `rails s` command now, you should see the following error:

```
cannot load such file -- my-app/config/environment (LoadError)
```

This is because this file is required on line 3 in `config.ru`.

- Create `config/environment.rb` with the following contents:

```
# frozen_string_literal: true

require_relative "application"

Rails.application.initialize!
```

---

If you run the `rails s` command now, the server will start, but when you visit `localhost:3000`, you should see the following error in the terminal:

```
Puma caught this error: uninitialized constant ActionController::Base
```

This is because Rails is a modular library, and you must specify the modules (called "railties" or "engines") you want to use. By default, Rails includes all the modules.

In this case, you need to specify that you want to use the ActionController module, which will give you the ability to define routes, controllers and views.

- Add the following to `config/application.rb` (under `require "rails"`):

```
require "action_controller/railtie"
```

---

**NOTE**

When you ran the `rails s` command, the `log` directory was automatically generated.

---

**GREEN**

If you run the `rails s` command now, you should see the Rails welcome index page.

You have now added the missing files required to run a minimal Rails application.

In the next steps, you will add a basic home page.

---

**NOTE**

When running the `rails s` command, you should see the following warning in the terminal:

```
config.eager_load is set to nil. Please update your config/environments/*.rb files accordingly:

  * development - set it to false
  * test - set it to false (unless you use a tool that preloads your test environment)
  * production - set it to true
```

Rails follows the concept of "convention over configuration". Therefore, sensible default values are used for most of the application configuration, which is specified on line 13 in `config/application.rb`. In each environment (`development`, `test` and `production`), you can override the default values by defining new values in an environment-specific configuration file. (These should be put in the `config/environments` directory, as seen in the warning message.)

Since the `eager_load` configuration value is not set to `nil` by default, you should specify an appropriate value for each environment.

- Create `config/environments/development.rb` with the following contents:

```
# frozen_string_literal: true

require "active_support/core_ext/integer/time"

Rails.application.configure do
  config.eager_load = false
end
```

- Create `config/environments/test.rb` with the following contents:

```
# frozen_string_literal: true

require "active_support/core_ext/integer/time"

Rails.application.configure do
  config.eager_load = ENV["CI"].present?
end
```

- Create `config/environments/production.rb` with the following contents:

```
# frozen_string_literal: true

require "active_support/core_ext/integer/time"

Rails.application.configure do
  config.eager_load = true
end
```

When you create a new Rails application using the `rails new` command, there are many other configuration values that are set for you that help improve the performance of the application. Since this tutorial is taking a minimal approach, the default values will be used unless otherwise specified. Nevertheless, it would be a good idea to explore the recommended values generated by Rails.

---

**REFACTOR**

As mentioned above, you will now add a basic home page.

- Create `config/routes.rb` with the following contents:

```
# frozen_string_literal: true

Rails.application.routes.draw do
  get "/", to: "pages#home", as: :home
end
```
---

If you run the `rails s` command now, and visit `localhost:3000`, you should see the following error:

```
ActionController::RoutingError (uninitialized constant PagesController)
```

This is because the `PagesController` is referenced on line 4 in `config/routes.rb`.

Before creating that file, it would be a good idea to create a parent controller that all other controllers inherit from. By default, Rails uses `ApplicationController`.

- Create `app/controllers/application_controller.rb` with the following contents:

```
# frozen_string_literal: true

class ApplicationController < ActionController::Base
end
```

- Create `app/controllers/pages_controller.rb` with the following contents:

```
# frozen_string_literal: true

class PagesController < ApplicationController
  def home
  end
end
```

---

If you run the `rails s` command now, and visit `localhost:3000`, you should see the following error:

```
ActionController::MissingExactTemplate (PagesController#home is missing a template for request formats: text/html)
```

This is because, unless specified otherwise, Rails expects to render a HTML page that has the same name as the action.

Again, before creating that file, it would be a good idea to create a layout that all views are rendered within. By default, Rails uses the layout that matches the name of the controller. Since all controllers inherit from `ApplicationController`, it will try to find a layout called `application.html.erb`.

- Create `app/views/layouts/application.html.erb` with the following contents:

```
<!DOCTYPE html>
<html>
  <head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1">
    <title>My App</title>
    <%= csrf_meta_tags %>
    <%= csp_meta_tag %>
  </head>
  <body>
    <%= yield %>
  </body>
</html>
```

- Create `app/views/pages/home.html.erb` with the following contents:

```
<h1>Home</h1>
```

The contents of `pages/home.html.erb` will be inserted into `layout/application.html.erb` on line 11 (replacing `<%= yield %>`).

---

**GREEN**

If you run the `rails s` command now, and visit `localhost:3000`, you should see the home page.

In the next steps, you will add some basic tests to verify what you have done so far. Moving forward, these tests can be used as part of a TDD approach.

---

**REFACTOR**

Rails comes with Minitest already installed, and by default, the basic configuration for the test suite is specified in `test/test_helper.rb`, which is required at the top of all your test files.

- Create `test/test_helper.rb` with the following contents:

```
# frozen_string_literal: true

ENV["RAILS_ENV"] ||= "test"

require_relative "../config/environment"

require "rails/test_help"

class ActiveSupport::TestCase
  parallelize workers: :number_of_processors
end
```

Any files inside the `test` directory that end in `_test.rb` will be run automatically whenever you run the `rails t` command.

- Create `test/controllers/pages_controller_test.rb` with the following contents:

```
# frozen_string_literal: true

require "test_helper"

class PagesControllerTest < ActionDispatch::IntegrationTest
  test "get home" do
    get home_url
    assert_response :success
    assert_select "title", text: "My App"
    assert_select "h1", text: "Home"
  end
end
```

The test above tests that visiting the home url (specified in `config/routes.rb` using `as:`):
1. is successful (i.e. returns a HTTP status code of `200`)
2. the title in the top of the browser is "My App"
3. there is a H1 on the page with the text "Home"

---

**GREEN**

If you run the `rails t` command now, all tests should pass:

```
1 runs, 1 assertions, 0 failures, 0 errors, 0 skips
```

---

**REFACTOR**

You can now update the views to use `I18n` translation values rather than hard-coded strings. (Although this is not strictly necessary, it is a good example of how to refactor your code while integrating the `I18n` library.)

- Update `test/controllers/pages_controller_test.rb` with the following contents:

```
test "should get home" do
  get home_url
  assert_response :success
  assert_select "title", text: I18n.t("application_name")
  assert_select "h1", text: I18n.t("home")
end
```

---

If you run the `rails t` command now, you should see the following error:

```
translation missing: en.application_name
```

This is because you have not specified the translation value for the key `en.application_name`.

- Create `config/locales/en.yml` with the following contents:

```
en:
  application_name: My Website
  home: Welcome
```

---

If you run the `rails t` command now, you should see the following error:

```
<My Website> expected but was
<My App>
```

This is because the title is still set as `My App` on line 6 in `app/views/layouts/application.html.erb`.

- Update `app/views/layouts/application.html.erb` with the following contents:

```
<title><%= t("application_name") %></title>
```

---

If you run the `rails t` command now, you should see the following error:

```
<Welcome> expected but was
<Home>
```

This is because the heading text is still set as `Home` on line 1 in `app/views/pages/home.html.erb`.

- Update `app/views/pages/home.html.erb` with the following contents:

```
<h1><%= t("home") %></h1>
```

---

**GREEN**

If you run the `rails t` command now, all tests should pass:

```
1 runs, 3 assertions, 0 failures, 0 errors, 0 skips
```

---

**REFACTOR**

You can now add a an API endpoint that will return a message in JSON format.

- Create `test/controllers/api/v1/hello_controller_test.rb` with the following contents:

```
# frozen_string_literal: true

require "test_helper"

class API::V1::HelloControllerTest < ActionDispatch::IntegrationTest
  test "should get index" do
    get api_v1_hello_url
    assert_response :success
    assert_equal "Hello, World!", @response.parsed_body["message"]
  end
end
```

The test above tests that visiting the endpoint (called `api_v1_hello_url`):
1. is successful (i.e. returns a HTTP status code of `200`)
2. the returned JSON data contains a key called `message` with the value `Hello, World!`

---

If you run the `rails t` command now, you should see the following error:

```
uninitialized constant API (NameError)
```

This is because the module `API` that is used as the namespace for the API controllers has not been defined.

By default, Rails does not recognise the word API (all uppercase). So before creating this file, you must add an initializer that specifies that the word API is an acronym.

- Create `config/initializers/inflections.rb` with the following contents:

```
# frozen_string_literal: true

ActiveSupport::Inflector.inflections(:en) do |inflect|
  inflect.acronym "API"
end
```

Again, it is a good idea to have a parent controller that all API controllers inherit from.

- Create `app/controllers/api/v1/application_controller.rb` with the following contents:

```
# frozen_string_literal: true

class API::V1::ApplicationController < ActionController::API
end
```

Notice that `ApplicationController` inherits from `ActionController::Base` while `API::V1::ApplicationController` inherits from `ActionController::API`.

`ActionController::Base` adds CSRF checks as well as automatic rendering of HTML pages, which are useful for HTML controllers. Since these are not needed for API endpoints, the API controllers can inherit from `ActionController::API`.

---

If you run the `rails t` command now, you should see the following error:

```
NameError: undefined local variable or method `api_v1_hello_url'
```

This is because you have not defined a route with this name.

- Update `config/routes.rb` with the following contents:

```
namespace :api do
  namespace :v1 do
    get "/", to: "hello#index", as: :hello
  end
end
```

---

If you run the `rails t` command now, you should see the following error:

```
Expected response to be a <2XX: success>, but was a <404: Not Found>
```

This is because the `API::V1::HelloController` is referenced on line 8 in `config/routes.rb`.

- Create `app/controllers/api/v1/hello_controller.rb` with the following contents:

```
# frozen_string_literal: true

class API::V1::HelloController < API::V1::ApplicationController
  def index
  end
end
```

---

If you run the `rails t` command now, you should see the following error:

```
Expected: "Hello, World!"
  Actual: nil
```

This is because the `index` action returns nil on line 4 in `app/controllers/api/v1/hello_controller.rb`.

- Update `app/controllers/api/v1/hello_controller.rb` with the following contents:

```
render json: { message: "Hello, World!" }
```

---

**GREEN**

If you run the `rails t` command now, all tests should pass:

```
2 runs, 5 assertions, 0 failures, 0 errors, 0 skips
```

---
