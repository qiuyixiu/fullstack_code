```
rails g model job title:string description:text
rake db:migrate
rails g controller jobs
rails g controller admin::jobs
```

```
config/routes.rb
Rails.application.routes.draw do
  devise_for :users
+  namespace :admin do
+    resources :jobs
+  end

+   resources :jobs
+   root 'jobs#index'
end
```

```
app/controllers/jobs_controller.rb
class JobsController < ApplicationController

  def index
    @jobs = Job.all
  end

  def show
    @job = Job.find(params[:id])
  end
end
```

```
app/controllers/admin/jobs_controller.rb

class Admin::JobsController < ApplicationController
  before_action :authenticate_user!, only: [:new, :create, :update, :edit, :destroy]
  before_action :require_is_admin


  def show
    @job = Job.find(params[:id])
  end

  def index
    @jobs = Job.all
  end

  def new
    @job = Job.new
  end

  def create
    @job = Job.new(job_params)

    if @job.save
      redirect_to admin_jobs_path
    else
      render :new
    end
  end

  def edit
    @job = Job.find(params[:id])
  end

  def update
    @job = Job.find(params[:id])
    if @job.update(job_params)
      redirect_to admin_jobs_path
    else
      render :edit
    end
  end

  def destroy
    @job = Job.find(params[:id])

    @job.destroy

    redirect_to admin_jobs_path
  end

  private

  def job_params
    params.require(:job).permit(:title, :description)
  end
end
```

`touch app/views/jobs/show.html.erb`

`touch app/views/jobs/index.html.erb`

`touch app/views/admin/jobs/index.html.erb`

`touch app/views/admin/jobs/new.html.erb`

`touch app/views/admin/jobs/show.html.erb`

`touch app/views/admin/jobs/edit.html.erb`

`touch app/views/admin/jobs/_form.html.erb`

```
app/views/jobs/show.html.erb

<h1> <%= @job.title %> </h1>

<p>
  <%= simple_format(@job.description) %>
</p>
```

```
app/views/jobs/index.html.erb
<table class="table table-bordered">
  <% @jobs.each do |job| %>
  <tr>
    <td>
      <%= link_to(job.title, job_path(job)) %>
    </td>
    <td>
      <%= job.created_at %>
    </td>
  </tr>
  <% end %>
</table>
```

```
app/views/admin/jobs/index.html.erb
<div class="pull-right">
  <%= link_to("Add a job", new_admin_job_path, :class => "btn btn-default" ) %>
</div>

<br><br>

<table class="table table-bordered">
  <% @jobs.each do |job| %>
  <tr>
    <td>
      <%= link_to(job.title, admin_job_path(job)) %>
    </td>
    <td>
      <%= link_to("Edit", edit_admin_job_path(job)) %>
      <%= link_to("Destroy", admin_job_path(job), :method => :delete, :data => { :confirm => "Are you sure?" }) %>
    </td>
    <td>
      <%= job.created_at %>
    </td>
  </tr>
  <% end %>
</table>
```

```
app/views/admin/jobs/new.html.erb

<h1>Add a job</h1>

<%= render "form" %>
```

```
app/views/admin/jobs/show.html.erb
<h1> <%= @job.title %> </h1>

<p>
  <%= simple_format(@job.description) %>

</p>
```

```
app/views/admin/jobs/edit.html.erb
<h1> Edit a job </h1>
<%= render "form" %>

```

```
app/views/admin/jobs/_form.html.erb
<%= simple_form_for [:admin, @job] do |f| %>
    <%= f.input :title %>
    <%= f.input :description %>
    <%= f.submit "Submit" %>
<% end %>
```

+ jobs必须要有标题

```
app/models/job.rb
class Job < ApplicationRecord
  validates :title, presence: true
end
```
+ 设定admin

`rails g migration add_is_admin_to_user`

```
db/migrate/xxxxx.rb
class AddIsAdminToUser < ActiveRecord::Migration[5.0]
  def change
    add_column :users, :is_admin, :boolean, default: false
  end
end
```

```
app/models/user.rb
+ def admin?
+    is_admin
+  end

```

`rails console`
```
u = User.first
u.is_admin = true
u.save
exit
```

```
app/controllers/application_controller.rb
class ApplicationController < ActionController::Base


+  def require_is_admin
+    if !current_user.admin?
+      flash[:alert] = 'You are not admin'
+      redirect_to root_path
+    end
+  end
end
```

+ 调整导航栏


```
app/views/common/_navbar.html.erb

<ul class="dropdown-menu">
+ <% if current_user.admin? %>
+ <li> <%= link_to("Admin Panel", admin_jobs_path) %> </li>
+ <% end %>
  <li> <%= link_to("登出", destroy_user_session_path, method: :delete) %> </li>
</ul>
```

+ 新增字段

`rails g migration add_more_detail_to_job`

```
db/migrate/XXX_add_more_detail_to_job.rb
class AddMoreDetailToJob < ActiveRecord::Migration[5.0]
  def change
    add_column :jobs, :wage_upper_bound, :integer
    add_column :jobs, :wage_lower_bound, :integer
    add_column :jobs, :contact_email, :string
  end
end
```

`rake db:migrate`

```
app/views/admin/jobs/_form.html.erb
<%= simple_form_for [:admin, @job] do |f| %>
    <%= f.input :title %>
    <%= f.input :description %>
+    <%= f.input :wage_lower_bound, :label => "薪资下限"  %>
+    <%= f.input :wage_upper_bound, :label => "薪资上限"  %>
+    <%= f.input :contact_email  %>
    <%= f.submit "Submit" %>
<% end %>

```

```
app/controllers/admin/jobs_controller.rb
  def job_params
    params.require(:job).permit(:title, :description, :wage_upper_bound, :wage_lower_bound, :contact_email)
  end
```
+ 薪水不能为空，且至少要大于0

```
app/models/job.rb
+  validates :wage_upper_bound, presence: true
+  validates :wage_lower_bound, presence: true
+  validates :wage_lower_bound, numericality: { greater_than: 0}

```

```
app/views/jobs/index.html.erb

 <div class="dropdown clearfix pull-right">
    <button class="btn btn-default dropdown-toggle" type="button" id="dropdownMenuDivider" data-toggle="dropdown" aria-haspopup="true" aria-expanded="true">
      排序
        <span class="caret"></span>
    </button>
    <ul class="dropdown-menu" aria-labelledby="dropdownMenuDivider">
        <li>
          <%= link_to("按照薪资下限排序", jobs_path(:order => "by_lower_bound")) %>
        </li>
        <li>
            <%= link_to("按照薪资上限排序", jobs_path(:order => "by_upper_bound")) %>

        </li>
        <li>
          <%= link_to("按照发表时间排序", jobs_path ) %>

        </li>
    </ul>
</div>
<!---以上全加入---!>
<table class="table table-boldered">
    <thead>
        <tr>
            <td>职缺</td>                
+            <td>薪资下限</td>
+            <td>薪资上限</td>
+            <td>刊登时间</td>
        </tr>

    </thead>
    <tbody>
        <% @jobs.each do |job| %>
        <tr>
            <td>
                <%= link_to(job.title, job_path(job)) %>
            </td>
+            <td>
+                <%= job.wage_lower_bound %>
+            </td>
+            <td>
+                <%= job.wage_upper_bound %>
+            </td>
+            <td>
+                <%= job.created_at %>
+            </td>
        </tr>
        <% end %>
    </tbody>

</table>

```

+ 新增is_hidden

`rails g migration add_is_hidden_to_job`
```
db/migrate/(一串数字)_add_is_hidden_to_job.rb
class AddIsHiddenToJob < ActiveRecord::Migration[5.0]
  def change
    add_column :jobs, :is_hidden, :boolean, default: true
  end
end

```
`rake db:migrate`

```
app/views/admin/jobs/_form.html.erb
<%= simple_form_for [:admin, @job] do |f| %>
    <%= f.input :title %>
    <%= f.input :description %>
    <%= f.input :wage_lower_bound, :label => "薪资下限"  %>
    <%= f.input :wage_upper_bound, :label => "薪资上限"  %>
    <%= f.input :contact_email  %>
+   <%= f.input :is_hidden %>
    <%= f.submit "Submit" %>
<% end %>
```

```
app/models/job.rb

class Job < ApplicationRecord
+  def publish!
+    self.is_hidden = false
+    self.save
+  end

+  def hide!
+    self.is_hidden = true
+    self.save
+  end

+ scope :published, -> { where(is_hidden: false) }
+ scope :recent, -> { order('created_at DESC') }
end  

```

```
app/controllers/admin/jobs_controller.rb

+ def publish
+  @job = Job.find(params[:id])
+  @job.publish!
+  redirect_to admin_jobs_path
+ end

+ def hide
+  @job = Job.find(params[:id])
+  @job.hide!
+  redirect_to admin_jobs_path
+ end

  private

  def job_params
    params.require(:job).permit(:title, :description, :wage_upper_bound, :wage_lower_bound, :contact_email, :is_hidden)
  end
```

```
app/controllers/jobs_controller.rb

def index
    @jobs = case params[:order]
            when 'by_lower_bound'
              Job.published.order('wage_lower_bound DESC')
            when 'by_upper_bound'
              Job.published.order('wage_upper_bound DESC')
            else
              Job.published.recent
            end
end

def show
  @job = Job.find(params[:id])

  if @job.is_hidden
    flash[:warning] = "This Job already archived"
    redirect_to root_path
  end
end
```

```
app/views/admin/jobs/index.html.erb

<td>
  <%= link_to("Edit", edit_admin_job_path(job)) %>
  <%= link_to("Destroy", admin_job_path(job), :method => :delete, :data => { :confirm => "Are you sure?" }) %>


  <% if job.is_hidden %>

    <%= link_to("Publish", publish_admin_job_path(job) , :method => :post, :class => "btn btn-xs btn-default") %>
  <% else %>
    <%= link_to("Hide", hide_admin_job_path(job), :method => :post,  :class => "btn btn-xs btn-default") %>
  <% end %>
</td>

```

```
config/routes.rb

namespace :admin do
  resources :jobs do
    member do
      post :publish
      post :hide
    end
  end
end
```

+ 区分前后台

```
app/views/admin/jobs/index.html.erb
  <td>
+    <%= render_job_status(job) %>

     <%= link_to(job.title, admin_job_path(job)) %>
  </td>

```

```
app/helpers/jobs_helper.rb

module JobsHelper

  def render_job_status(job)
    if job.is_hidden
      content_tag(:span, "", :class => "fa fa-lock")
    else
      content_tag(:span, "", :class => "fa fa-globe")
    end
  end
end

```

+ 增加admin layouts

`touch app/views/layouts/admin.html.erb`

```
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
            <div class="row">
              <div class="col-md-2">
                <ul class="nav nav-pills nav-stacked" style="max-width: 300px; padding-top:20px;">
                  <li class="active"><%= link_to("Jobs", admin_jobs_path ) %> </li>

                </ul>
              </div>

              <div class="col-md-10">
                <%= yield %>
              </div>
           </div>
        </div>

        <%= render "common/footer" %>

    </body>
</html>
```

```
app/controllers/admin/jobs_controller.rb

class Admin::JobsController < ApplicationController
  before_action :authenticate_user!, only: [:new, :create, :update, :edit, :destroy]
  before_action :require_is_admin
+  layout "admin"

```

+ 新增简历


`rails g model resume job_id:integer user_id:integer content:text`
`rake db:migrate`
`rails g controller resumes`
`rails g controller admin::resumes`

```
app/models/resume.rb

class Resume < ApplicationRecord

+  belongs_to :user
+  belongs_to :job
+  validates :content, presence: true
+  mount_uploader :attachment, AttachmentUploader

end
```

```
app/models/job.rb

class Job < ApplicationRecord
+  has_many :resumes
end
```

```
app/models/user.rb
class User < ApplicationRecord
+  has_many :resumes
end
```

```
config/routes.rb
Rails.application.routes.draw do
  devise_for :users

  namespace :admin do
    resources :jobs do
      member do
        post :publish
        post :hide
      end
+     resources :resumes
    end
  end

-  resources :jobs
+  resources :jobs do
+    resources :resumes
+  end

  root 'jobs#index'
end

```

```
app/views/jobs/show.html.erb

<div class="text-center">
  <%= link_to("投交履历", new_job_resume_path(@job), :style => "font-size: 30px; text-decoration:underline;") %>
</div>
```

```
app/controllers/resumes_controller.rb

class ResumesController < ApplicationController
  before_action :authenticate_user!

  def new
    @job = Job.find(params[:job_id])
    @resume = Resume.new
  end

  def create
    @job = Job.find(params[:job_id])
    @resume = Resume.new(resume_params)
    @resume.job = @job
    @resume.user = current_user

    if @resume.save
      flash[:notice] = "成功提交履历"
      redirect_to job_path(@job)
    else
      render :new
    end
  end

  private

  def resume_params
    params.require(:resume).permit(:content, :attachment)
  end
end
```

`touch app/views/resumes/new.html.erb`

```
app/views/resumes/new.html.erb

<h3> 投交履历到 <%= @job.title %> </h3>

<hr>

<%= simple_form_for [@job, @resume] do |f| %>
  <%= f.input :content %>
  <%= f.input :attachment %>
  <%= f.submit "送出" %>
<% end %>
```

`touch app/views/admin/resumes/index.html.erb`

`touch app/views/admin/resumes/_resume.html.erb`

```
app/views/admin/resumes/index.html.erb

<h2> <%= @job.title %> - 履历列表 </h2>

<hr>
<%= render :partial => "resume", :collection => @resumes, :as => :resume %>

```

```
app/views/admin/resumes/_resume.html.erb

<div class="panel panel-default">
    <div class="panel-heading">
        <div class="row">
            <div class="col-md-12">
                <h3 class="panel-title">
                    <%= resume.user.email %>
                </h3>
            </div>

        </div>
    </div>
    <div class="panel-body">
        <div class="trix-content">
            <p> <%= simple_format(resume.content) %> </p>
        </div>

        <hr>

        <% if resume.attachment.present? %>
          <i class="fa fa-paperclip"> </i>
          <%= link_to("Download Resume", resume.attachment_url) %>
        <% else %>
           No Attachment
        <% end %>
    </div>

</div>
```

`rails g migration add_attachment_to_resume`

```
class AddAttachmentToResume < ActiveRecord::Migration[5.0]
  def change
    add_column :resumes, :attachment, :string
  end
end
```

`rake db:migrate`


```
.gitignore
+ public/uploads
```

+ 调整管理后台

```
app/controllers/admin/resumes_controller.rb
class Admin::ResumesController < ApplicationController
  before_action :authenticate_user!
  before_action :require_is_admin

  layout 'admin'

  def index
    @job = Job.find(params[:job_id])
+    @resumes = @job.resumes.order('created_at DESC')
  end
end
```

```
app/views/admin/jobs/index.html.erb

<table class="table table-boldered">
    <thead>
        <tr>
            <td>
                职缺
            </td>

            <td>
              收到履历数量
            </td>
            <td>
                薪资下限
            </td>
            <td>
                薪资上限
            </td>
            <td>
                管理选项
            </td>
            <td>
                刊登时间
            </td>
        </tr>
    </thead>

    <% @jobs.each do |job| %>
    <tr>

        <td>
            <%= render_job_status(job) %>

            <%= link_to(job.title, admin_job_path(job)) %>
        </td>
        <td> <%= link_to(job.resumes.count, admin_job_resumes_path(job)) %> </td>
        <td> <%= job.wage_lower_bound %> </td>
        <td> <%= job.wage_upper_bound %> </td>
        <td>
            <%= link_to("Edit", edit_admin_job_path(job)) %>
            |
            <%= link_to("Destroy", admin_job_path(job), :method => :delete, :data => { :confirm => "Are you sure?" }) %>

            |
            <% if job.is_hidden %>

            <%= link_to("Publish", publish_admin_job_path(job) , :method => :post, :class => "btn btn-xs btn-default") %>
        <% else %>
            <%= link_to("Hide", hide_admin_job_path(job), :method => :post,  :class => "btn btn-xs btn-default") %>
            <% end %>
        </td>
        <td>
            <%= job.created_at %>
        </td>
    </tr>
    <% end %>
</table>
```
