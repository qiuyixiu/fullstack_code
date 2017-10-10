#### 配置Bootstrap
+ 安装Bootstrap、Devise、simple_form、carrierwave、jquery-rails


```
Gemfile
+  gem 'bootstrap-sass'
+  gem 'devise'
+  gem 'simple_form'
+  gem 'font-awesome-rails'
+  gem 'carrierwave'
+  gem 'mini_magick'
+  gem 'jquery-rails'


   group :development, :test do
```
`bundle install`

`rails g devise:install`

`rails g devise user`

`rake db:migrate`

`rails generate simple_form:install --bootstrap`

`rails g uploader attachment`

`rails g uploader image`

`mv app/assets/stylesheets/application.css app/assets/stylesheets/application.scss
`

```
app/assets/stylesheets/application.scss
/*
 * ...(一堆注解)
+ *= require font-awesome
 *= require_tree .
 *= require_self
 */

+ @import "bootstrap-sprockets";
+ @import "bootstrap";
+ @import "font-awesome";

```

```
.gitignore

...(略)
+ config/database.yml
+ public/uploads
```



+ 新增navbar、footer


`mkdir app/views/common`

`touch app/views/common/_navbar.html.erb`

```
app/views/common/_navbar.html.erb
<nav class="navbar navbar-default" role="navigation">
    <div class="container-fluid">
        <!-- Brand and toggle get grouped for better mobile display -->
        <div class="navbar-header">
            <a class="navbar-brand" href="/">JDstore </a>
        </div>

        <!-- Collect the nav links, forms, and other content for toggling -->
        <div class="collapse navbar-collapse" id="bs-example-navbar-collapse-1">
            <ul class="nav navbar-nav navbar-right">
                <li>
                  <% if !current_user %>
                    <li><%= link_to("注册", new_user_registration_path) %> </li>
                    <li><%= link_to(content_tag(:i, "登录", class: 'fa fa-sign-in'), new_user_session_path) %></li>
                  <% else %>
                    <li class="dropdown">
                      <a href="#" class="dropdown-toggle" data-toggle="dropdown">
                        Hi!, <%= current_user.email %>
                        <b class="caret"></b>
                      </a>
                      <ul class="dropdown-menu">
                        <li> <%= link_to(content_tag(:i, "退出", class: 'fa fa-sign-out'), destroy_user_session_path, method: :delete) %> </li>
                      </ul>
                    </li>
                  <% end %>
                </li>
            </ul>
        </div>
        <!-- /.navbar-collapse -->
    </div>
    <!-- /.container-fluid -->
</nav>
```

`touch app/views/common/_footer.html.erb`

```
app/views/common/_footer.html.erb
<footer class="container" style="margin-top: 100px;">
    <p class="text-center">Copyright ©2017 JDstore
        <br>Design by yourname

    </p>
</footer>
```

+ 调整application.html


```
app/views/layouts/application.html.erb
<!DOCTYPE html>
<html>
    <head>
        <title>Job Listing </title>
        <%= csrf_meta_tags %>

        <%= stylesheet_link_tag    'application', media: 'all', 'data-turbolinks-track': 'reload' %>
        <%= javascript_include_tag 'application', 'data-turbolinks-track': 'reload' %>
    </head>

    <body>

        <div class="container-fluid">
            <%= render "common/navbar" %>
            <%= render "common/flashes" %>
            <%= yield %>
        </div>

        <%= render "common/footer" %>

    </body>
</html>
```

+ 新增welcome#index

`rails g controller welcome`

`touch app/views/welcome/index.html.erb`

如何更改首页指向
```
config/routes.rb
Rails.application.routes.draw do
+  root 'welcome#index'
end
```

+ 增加flash功能

```
app/assets/javascripts/application.js
... (一堆注解)
+  //= require jquery
+  //= require jquery_ujs
  //= require turbolinks
+ //= require bootstrap
  //= require_tree .
```

`touch app/views/common/_flashes.html.erb`

```
app/views/common/_flashes.html.erb
<% if flash.any? %>
  <% user_facing_flashes.each do |key, value| %>
    <div class="alert alert-dismissable alert-<%= flash_class(key) %>">
      <button class="close" data-dismiss="alert">×</button>
      <%= value %>
    </div>
  <% end %>
<% end %>
```

`touch app/helpers/flashes_helper.rb`

```
app/helpers/flashes_helper.rb
module FlashesHelper
  FLASH_CLASSES = { alert: "danger", notice: "success", warning: "warning"}.freeze

  def flash_class(key)
    FLASH_CLASSES.fetch key.to_sym, key
  end

  def user_facing_flashes
    flash.to_hash.slice "alert", "notice","warning"
  end
end
```
