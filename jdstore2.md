#### 上架后台CRUD及前台

`rails g controller admin::products`
`rails g controller products`

```
config/routes.rb

Rails.application.routes.draw do
+ namespace :admin do
+   resources :products
+ end

- root 'welcome#index'
+ root 'products#index'

+ resources :products

...(略)
end
```

`rails g model product title:string description:text quantity:integer price:integer`

`rake db:migrate`


```
app/controllers/admin/products_controller.rb

class Admin::ProductsController < ApplicationController

  before_action :authenticate_user!
  before_action :admin_required
  layout "admin"

  def index
    @products = Product.all
  end

  def new
    @product = Product.new
  end

  def edit
    @product = Product.find(params[:id])
  end

  def update
    @product = Product.find(params[:id])

    if @product.update(product_params)
      redirect_to admin_products_path
    else
      render :edit
    end
  end

  def create
    @product = Product.new(product_params)

    if @product.save
      redirect_to admin_products_path
    else
      render :new
    end
  end

  private

  def product_params
    params.require(:product).permit(:title, :description, :quantity, :price, :image)
  end
end
```

```
app/controllers/products_controller.rb

class ProductsController < ApplicationController
+ def index
+   @products = Product.all
+ end

+ def show
+   @product = Product.find(params[:id])
+ end
end
```


`touch app/views/admin/products/new.html.erb`

`touch app/views/admin/products/index.html.erb`

`touch app/views/admin/products/edit.html.erb`

`touch app/views/admin/products/show.html.erb`

`touch app/views/admin/products/_form.html.erb`

`touch app/views/products/index.html.erb`

`touch app/views/products/show.html.erb`


```
app/views/admin/products/new.html.erb

<h2> New Product </h2>
<%= render "form" %>
```

```
app/views/admin/products/index.html.erb

<ul>
  <% @products.each do |product| %>

    <li> <%= link_to(product.title, admin_product_path(product)) %> </li>
  <% end %>
</ul>

```

```
app/views/admin/products/edit.html.erb
<h2> Edit Product </h2>
+ <% if @product.image.present? %>
+   <span>目前商品图</span> <br>
+   <%= image_tag(@product.image.thumb.url) %>
+ <% end %>
<%= render "form" %>
```

```
app/views/admin/jobs/_form.html.erb
<%= simple_form_for [:admin, @job] do |f| %>
    <%= f.input :title %>
    <%= f.input :description %>
    <%= f.input :quantity %>
    <%= f.input :price %>
    <%= f.input :image, as: :file %>
    <%= f.submit "提交", data: { disable_with: "提交中..." } %>
<% end %>
```

```
app/views/products/index.html.erb

<h2> Product List </h2>
<div class="pull-right" style="padding-bottom: 20px;">
  <%= link_to("新增产品", new_admin_product_path, class: "btn btn-primary btn-sm") %>
</div>
<table class="table table-bordered">
  <thead>
    <tr>
      <th>#</th>
      <th width="220">Product Pic</th>
      <th>Name</th>
      <th>Price</th>
      <th width="100"> Options</th>
    </tr>
  </thead>
  <tbody>
    <% @products.each do |product| %>
      <tr>
        <td>
          <%= product.id %>
        </td>
        <td>
          <%= link_to product_path(product) do %>
            <% if product.image.present? %>
              <%= image_tag(product.image.thumb.url, class: "thumbnail") %>
            <% else %>
              <%= image_tag("http://placehold.it/200x200&text=No Pic", class: "thumbnail") %>
            <% end %>
          <% end %>
        </td>
        <td>
          <%= product.title %>
        </td>
        <td>
          <%= product.price %>
        </td>
        <td>
          <%= link_to("Edit", edit_admin_product_path(product)) %>
        </td>
      </tr>
    <% end %>
  </tbody>
</table>
```

```
app/views/products/show.html.erb

<div class="row">
  <div class="col-md-6">
    <% if @product.image.present? %>
      <%= image_tag(@product.image.medium.url, class: "thumbnail") %>
    <% else %>
      <%= image_tag("http://placehold.it/400x400&text=No Pic", class: "thumbnail") %>
    <% end %>
  </div>
  <div class="col-md-6">
    <h2> <%= @product.title %> </h2>
    <div style="height:100px;">
      <p>
        <%= @product.description %>
      </p>
    </div>
    <div> 数量 : <%= @product.quantity %> </div>
    <div class="product-price"> ￥ <%= @product.price %> </div>
    <div class="pull-right">
      <%= link_to("加入购物车", "#", :class => "btn btn-danger btn-lg") %>
    </div>
  </div>
</div>
```

```
app/views/common/_navbar.html.erb

...(略)
    <!-- Collect the nav links, forms, and other content for toggling -->
    <div class="collapse navbar-collapse" id="bs-example-navbar-collapse-1">
+     <ul class="nav navbar-nav">
+       <li class="active">
+         <%= link_to("Products", products_path) %>
+       </li>
+     </ul>
...(略)

<li class="dropdown">
  <a href="#" class="dropdown-toggle" data-toggle="dropdown">
    Hi!, <%= current_user.email %>
    <b class="caret"></b>
  </a>
  <ul class="dropdown-menu">
+             <% if current_user.admin? %>
+               <li>
+                 <%= link_to("Admin 选单", admin_products_path ) %>
+               </li>
+             <% end %>
    <li>
      <%= link_to(content_tag(:i, '登出', class: 'fa fa-sign-out'), destroy_user_session_path, method: :delete) %>
    </li>
  </ul>
</li>
...(略)
```


#### admin功能
```
app/controllers/application_controller.rb

class ApplicationController < ActionController::Base
  # Prevent CSRF attacks by raising an exception.

  # For APIs, you may want to use :null_session instead.

  protect_from_forgery with: :exception

+ def admin_required
+   if !current_user.admin?
+     redirect_to "/", alert: "You are not admin."
+   end
+ end
end
```

```
app/models/user.rb

class User < ApplicationRecord
  # Include default devise modules. Others available are:

  # :confirmable, :lockable, :timeoutable and :omniauthable

  devise :database_authenticatable, :registerable,
         :recoverable, :rememberable, :trackable, :validatable

+ def admin?
+   is_admin
+ end
end
```

`rails g migration add_is_admin_to_user`

```
db/migrate/xxx(一堆数字)_add_is_admin_to_user.rb

class AddIsAdminToUser < ActiveRecord::Migration[5.0]
  def change
+   add_column :users, :is_admin, :boolean, default: false
  end
end
```

`rake db:migrate`

```
rails c

u = User.new(email: "admin@test.com", password: "123456", password_confirmation: "123456")
u.save
u.is_admin = true
u.save
```

`touch app/views/layouts/admin.html.erb`

```
app/views/layouts/admin.html.erb

<!DOCTYPE html>
<html>
<head>
  <title>JDstore 后台</title>
  <%= stylesheet_link_tag    'application', media: 'all', 'data-turbolinks-track' => true %>
  <%= javascript_include_tag 'application', 'data-turbolinks-track' => true %>
  <%= csrf_meta_tags %>
</head>
<body>
  <div class="container">
    <%= render "common/navbar" %>
    <div class="row">
      <div class="col-md-2">
        <ul class="nav nav-pills nav-stacked" style="max-width: 300px;">
          <li> <%= link_to("Products", admin_products_path) %> </li>
        </ul>
      </div>
      <div class="col-md-10">
        <%= yield %>
      </div>
    </div>
  </div>
</body>
</html>
```

#### 上传图片功能
`rails g migration add_image_to_product`

```
db/migrate/XXX(一堆数字)_add_image_to_product.rb

class AddImageToProduct < ActiveRecord::Migration[5.0]
  def change
+   add_column :products, :image, :string
  end
end
```

`rake db:migrate`

```
app/models/product.rb

class Product < ApplicationRecord
+ mount_uploader :image, ImageUploader
end
```

```
app/uploaders/image_uploader.rb

class ImageUploader < CarrierWave::Uploader::Base

  # Include RMagick or MiniMagick support:

  # include CarrierWave::RMagick

- # include CarrierWave::MiniMagick

+ include CarrierWave::MiniMagick

  # Choose what kind of storage to use for this uploader:

  storage :file
  # storage :fog


  # Override the directory where uploaded files will be stored.

  # This is a sensible default for uploaders that are meant to be mounted:

  def store_dir
    "uploads/#{model.class.to_s.underscore}/#{mounted_as}/#{model.id}"
  end

+ process resize_to_fit: [800, 800]

+ version :thumb do
+   process resize_to_fill: [200,200]
+ end

+ version :medium do
+   process resize_to_fill: [400,400]
+ end
...(略)
```

#### 购物车功能
```
app/views/products/show.html.erb

-  <%= link_to("加入购物车", "#", :class => "btn btn-primary btn-lg btn-danger") %>
+  <%= link_to("加入购物车", add_to_cart_product_path(@product), :method => :post, :class => "btn btn-primary btn-lg btn-danger") %>
```

```
app/controllers/products_controller.rb

+ def add_to_cart
+  @product = Product.find(params[:id])
+  if !current_cart.products.include?(@product)
     current_cart.add_product_to_cart(@product)
+    flash[:notice] = "你已成功将 #{@product.title} 加入购物车"
+  else
+    flash[:warning] = "你的购物车内已有此物品"
+  end
+  redirect_to :back
+ end
```

```
config/routes.rb

root 'products#index'
devise_for :users

  namespace :admin do
   resources :products
  end

-  resources :products
+  resources :products do
+    member do
+      post :add_to_cart
+    end
+  end

+ resources :carts do
+   collection do
+     delete :clean
+   end
+ end

+ resources :cart_items

end
```

`rails g model cart`

`rails g model cart_item`

`rails g controller carts`

`rails g controller cart_items`

`rails g controller orders`

```
db/migrate/XXX(一堆数字)_create_cart_items.rb

class CreateCartItems < ActiveRecord::Migration[5.0]
  def change
    create_table :cart_items do |t|
+      t.integer :cart_id
+      t.integer :product_id
+      t.integer :quantity, default: 1
      t.timestamps
    end
  end
end
```

`rake db:migrate`

```
app/models/cart.rb

class Cart < ApplicationRecord
+  has_many :cart_items
+  has_many :products, through: :cart_items, source: :product

+  def add_product_to_cart(product)
+    ci = cart_items.build
+    ci.product = product
+    ci.quantity = 1
+    ci.save
+  end

+  def total_price
+    sum = 0
+    cart_items.each do |cart_item|
+      if cart_item.product.price.present?
+        sum += cart_item.quantity * cart_item.product.price
+      end
+    end
+    sum
+  end

+ def clean!
+   cart_items.destroy_all
+ end
end
```

```
app/models/cart_item.rb

class CartItem < ApplicationRecord
+  belongs_to :cart
+  belongs_to :product
end
```

```
app/controllers/carts_controller.rb

class CartsController < ApplicationController
+ def clean
+   current_cart.clean!
+   flash[:warning] = "已清空购物车"
+   redirect_to carts_path
+ end
end
```

```
app/controllers/cart_items_controller.rb

class CartItemsController < ApplicationController
+  before_action :authenticate_user!

+  def destroy
+    @cart = current_cart
+    @cart_item = @cart.cart_items.find_by(product_id: params[:id])
+    @product = @cart_item.product
+    @cart_item.destroy

+    flash[:warning] = "成功将 #{@product.title} 从购物车删除!"
+    redirect_to :back
+  end

+  def update
+    @cart = current_cart
+    @cart_item = @cart.cart_items.find_by(product_id: params[:id])
+    if @cart_item.product.quantity >= cart_item_params[:quantity].to_i
+      @cart_item.update(cart_item_params)
+      flash[:notice] = "成功变更数量"
+    else
+      flash[:warning] = "数量不足以加入购物车"
+    end

+    redirect_to carts_path
+  end

+  private

+  def cart_item_params
+    params.require(:cart_item).permit(:quantity)
+  end
end
```

```
app/controllers/orders_controller.rb

class OrdersController < ApplicationController
+  before_action :authenticate_user!, only: [:create]

+  def create
+    @order = Order.new(order_params)
+    @order.user = current_user
+    @order.total = current_cart.total_price

+    if @order.save
+      redirect_to order_path(@order)
+    else
+      render 'carts/checkout'
+    end
+  end

+  private

+  def order_params
+    params.require(:order).permit(:billing_name, :billing_address, :shipping_name, :shipping_address)
+  end
end

```

```
app/views/common/_navbar.html.erb

<ul class="nav navbar-nav navbar-right">
+ <li>
+    <%= link_to "carts_path" do  %>
+       购物车 <i class="fa fa-shopping-cart"> </i> ( <%= current_cart.products.count %> )
+    <% end %>
+ </li>

  <% if !current_user %>
```

`touch app/views/carts/index.html.erb`

```
app/views/carts/index.html.erb

<div class="row">
  <div class="col-md-12">

    <%= link_to("清空购物车", clean_carts_path ,
              method: :delete , class: "pull-right",
              style: "text-decoration: underline;",
              data: { confirm: "你确定要清空整个购物车吗？"} )%>

    <h2> 购物车 </h2>

    <table class="table table-bordered">
      <thead>
        <tr>
          <th colspan="2">商品资讯</th>
          <th>单价</th>
          <th>数量</th>
          <th>操作选项 </th>
        </tr>
      </thead>
      <tbody>

        <% current_cart.cart_items.each do |cart_item| %>
          <tr>
            <td>
              <%= link_to product_path(cart_item.product) do %>
                <% if cart_item.product.image.present? %>
                  <%= image_tag(cart_item.product.image.thumb.url, class: "thumbnail") %>
                <% else %>
                  <%= image_tag("http://placehold.it/200x200&text=No Pic", class: "thumbnail") %>
                <% end %>
              <% end %>
            </td>
            <td>
              <%= link_to(cart_item.product.title, product_path(cart_item.product)) %>
            </td>
            <td>
              <%= cart_item.product.price %>
            </td>
            <td>
              <%= form_for cart_item, url: cart_item_path(cart_item.product_id) do |f| %>
                <%= f.select :quantity, 1..cart_item.product.quantity %>
                <%= f.submit "更新", data: { disable_with: "Submiting..." } %>
              <% end %>
            </td>
            <td>
              <%= link_to cart_item_path(cart_item.product_id), method: :delete do %>
                <i class="fa fa-trash"></i>
              <% end %>
            </td>
          </tr>
          </tr>
        <% end %>

      </tbody>
    </table>

    <br>

    <div class="total clearfix">
      <span class="pull-right">
         <span>
           总计 <%= render_cart_total_price(current_cart) %> RMB
         </span>
      </span>
    </div>

    <hr>

    <div class="checkout clearfix">
      <%= link_to("确认结账", "#", method: :post, class: "btn btn-lg btn-danger pull-right") %>
    </div>
  </div>
</div>
```

```
app/views/products/show.html.erb
...(略)
    <div class="product-price"> ¥ <%= @product.price %> </div>

    <div class="pull-right">
+     <% if @product.quantity > 0 %>
        <%= link_to("加入购物车", add_to_cart_product_path(@product), method: :post,
                    class: "btn btn-lg btn-danger") %>
+     <% else %>
+       已销售一空，无法购买
+     <% end %>
    </div>
...(略)
```

```
app/helpers/carts_helper.rb

module CartsHelper
+  def render_cart_total_price(cart)
+    sum = 0
+    cart.cart_items.each do |cart_item|
+      if cart_item.product.price.present?
+        sum += cart_item.quantity * cart_item.product.price
+      end
+    end
+    sum
+  end
end
```

```
app/controllers/application_controller.rb

class ApplicationController < ActionController::Base

 # ...略



+  helper_method :current_cart

+  def current_cart
+    @current_cart ||= find_cart
+  end

+  private

+  def find_cart
+    cart = Cart.find_by(id: session[:cart_id])
+    if cart.blank?
+      cart = Cart.create
+    end
+    session[:cart_id] = cart.id
+    return cart
+  end
end
```


#### 结帐页功能
```
app/views/carts/index.html.erb

...(略)
    <div class="checkout clearfix">
－      <%= link_to("确认结账", "#", method: :post, class: "btn btn-lg btn-danger pull-right") %>
＋      <%= link_to("确认结账", checkout_carts_path, method: :post, class: "btn btn-lg btn-danger pull-right") %>
    </div>
...(略)
```

```
config/routes.rb

...(略)
  resources :carts do
    collection do
      delete :clean
+      post :checkout
    end
  end

+  resources :orders  
...(略)
```

`rails g model order`

```
db/migrate/XXX(一堆数字)_create_orders.rb

class CreateOrders < ActiveRecord::Migration[5.0]
  def change
    create_table :orders do |t|
+      t.integer :total, default: 0
+      t.integer :user_id
+      t.string :billing_name
+      t.string :billing_address
+      t.string :shipping_name
+      t.string :shipping_address
      t.timestamps
    end
  end
end
```

`rake db:migrate`

```
app/models/order.rb

class Order < ApplicationRecord
+  belongs_to :user

+  validates :billing_name, presence: true
+  validates :billing_address, presence: true
+  validates :shipping_name, presence: true
+  validates :shipping_address, presence: true

end
```

```
app/controllers/carts_controller.rb

class CartsController < ApplicationController
...(略)
+  def checkout
+    @order = Order.new
+  end

    private
...(略)
end
```

```
app/controllers/orders_controller.rb

class OrdersController < ApplicationController
+  before_action :authenticate_user!, only: [:create]

+  def create
+    @order = Order.new(order_params)
+    @order.user = current_user
+    @order.total = current_cart.total_price

+    if @order.save
+      redirect_to order_path(@order)
+    else
+      render 'carts/checkout'
+    end
+  end

+  private

+  def order_params
+    params.require(:order).permit(:billing_name, :billing_address, :shipping_name, :shipping_address)
+  end
end
```

```
app/models/user.rb

class User < ApplicationRecord
+  has_many :orders
...(略)  
```

`touch app/views/carts/checkout.html.erb`

```
app/views/carts/checkout.html.erb

<div class="row">
   <div class="col-md-12">

    <h2> 购物明细 </h2>

    <table class="table table-bordered">
      <thead>
        <tr>
          <th width="80%">商品明细</th>
          <th>单价</th>
          <th>数量</th>
        </tr>
      </thead>
      <tbody>

        <% current_cart.cart_items.each do |cart_item| %>
          <tr>
            <td>
              <%= link_to(cart_item.product.title, product_path(cart_item.product)) %>
            </td>
            <td>
              <%= cart_item.product.price %>
            </td>

            <td>
              <%= cart_item.quantity %>
            </td>

          </tr>
        <% end %>

      </tbody>
    </table>

    <div class="total clearfix">
      <span class="pull-right">
        总计 <%= render_cart_total_price(current_cart) %> CNY
      </span>
    </div>

    <hr>

    <h2> 订单资讯 </h2>

    <div class="order-form">

      <%= simple_form_for @order do |f| %>



          <legend> 订购人</legend>
          <div class="form-group col-lg-6">
            <%= f.input :billing_name  %>
          </div>
          <div class="form-group col-lg-6">
            <%= f.input :billing_address  %>
          </div>

          <legend> 收货人</legend>
          <div class="form-group col-lg-6">
           <%= f.input :shipping_name  %>
          </div>
          <div class="form-group col-lg-6">
            <%= f.input :shipping_address  %>
          </div>

        <div class="checkout">
          <%= f.submit "生成订单", class: "btn btn-lg btn-danger pull-right",
                       data: { disable_with: "Submitting..." } %>
        </div>
      <% end %>

    </div>
  </div>
</div>
```

#### 购买明细功能
`rails g model product_list`

```
db/migrate/XXX(一堆数字)_create_product_lists.rb

class CreateProductLists < ActiveRecord::Migration[5.0]
  def change
    create_table :product_lists do |t|
+      t.integer :order_id
+      t.string  :product_name
+      t.integer :product_price
+      t.integer :quantity
      t.timestamps
    end
  end
end
```

`rake db:migrate`

```
app/models/order.rb

class Order < ApplicationRecord
...(略)

+  has_many :product_lists
end
```

```
app/models/product_list.rb

class ProductList < ApplicationRecord
+  belongs_to :order
end
```

```
app/controllers/orders_controller.rb

class OrdersController < ApplicationController
  before_action :authenticate_user!, only: [:create]

  def create
    @order = Order.new(order_params)
    @order.user = current_user
    @order.total = current_cart.total_price

    if @order.save

+      current_cart.cart_items.each do |cart_item|
+        product_list = ProductList.new
+        product_list.order = @order
+        product_list.product_name = cart_item.product.title
+        product_list.product_price = cart_item.product.price
+        product_list.quantity = cart_item.quantity
+        product_list.save
+      end

      redirect_to order_path(@order)
    else
      render 'carts/checkout'
    end
  end

+  def show
+    @order = Order.find(params[:id])
+    @product_lists = @order.product_lists
+  end

  private

  def order_params
    params.require(:order).permit(:billing_name, :billing_address, :shipping_name, :shipping_address)
  end
end
```

`touch app/views/orders/show.html.erb`

```
app/views/orders/show.html.erb

<div class="row">
  <div class="col-md-12">

    <h2> 订单明细 </h2>

    <table class="table table-bordered">
      <thead>
        <tr>
          <th width="80%">商品明细</th>
          <th>单价</th>
        </tr>
      </thead>
      <tbody>

        <% @product_lists.each do |product_list| %>
          <tr>
            <td>
              <%= product_list.product_name %>
            </td>
            <td>
              <%= product_list.product_price %>
            </td>
          </tr>
        <% end %>

      </tbody>
    </table>

    <div class="total clearfix">
      <span class="pull-right">
        总计 <%= @order.total %> CNY
      </span>
    </div>

     <hr>

     <h2> 寄送资讯 </h2>

     <table class="table table-striped table-bordered">
      <tbody>
        <tr>
          <td>
            订购人
          </td>
        </tr>
        <tr>
          <td>
            <%= @order.billing_name %> - <%= @order.billing_address %>
          </td>
        </tr>
        <tr>
          <td>
            收件人
          </td>
        </tr>
        <tr>
          <td>
            <%= @order.shipping_name %> - <%= @order.shipping_address %>
          </td>
        </tr>
      </tbody>
    </table>
  </div>
</div>
```

#### 网址无序功能
`rails g migration add_token_to_order`

```
db/migrate/XXX(一堆数字)_add_token_to_order.rb

class AddTokenToOrder < ActiveRecord::Migration[5.0]
  def change
+    add_column :orders, :token, :string
  end
end
```

`rake db:migrate`

```
app/models/order.rb

class Order < ApplicationRecord
+  before_create :generate_token

+  def generate_token
+    self.token = SecureRandom.uuid
+  end
...(略)
end
```

```
app/controllers/orders_controller.rb

...(略)
  def create
    @order = Order.new(order_params)
    @order.user = current_user
    @order.total = current_cart.total_price

    if @order.save
      current_cart.cart_items.each do |cart_item|
        product_list = ProductList.new
        product_list.order = @order
        product_list.product_name = cart_item.product.title
        product_list.product_price = cart_item.product.price
        product_list.quantity = cart_item.quantity
        product_list.save
      end
-     redirect_to order_path(@order)
+     redirect_to order_path(@order.token)
    else
      render 'carts/checkout'
    end
  end

  def show
-   @order = Order.find(params[:id])
+   @order = Order.find_by_token(params[:id])
    @product_lists = @order.product_lists
  end
...(略)
```
