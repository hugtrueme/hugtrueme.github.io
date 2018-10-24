---
layout: post
title:  "使用 CanCanCan 搭建 Rails 權限管理系統"
date:   2018-10-16 23:00:00 +0800
categories: Rails
tags: Rails CanCanCan Devise
---

一間新創公司的網路服務，在從零開始，並一路成長的過程中，在程式開發上一定會遇到的其中一站是：**權限管理系統**，而它又可分為兩個層面：

1. 處理使用者的註冊、登入、忘記密碼
2. 限制使用者能夠操作哪些功能、看到哪些資料

幸運的是，在 Ruby on Rails 的生態系中，已經有前人優雅地解決了上面的兩個問題，並且將解法打包成 gem 供大家使用。第一項「處理使用者註冊」已有大家熟知的 [Devise] 可以使用，本文章係針對第二項「限制使用者可以操作哪些功能、看到哪些資料」來分享如何使用 [CanCanCan] 來建立一個基於角色（Role based）權限管理機制 。


<br>
#### **沒有權限機制的蠻荒時代**
<br>
在一個**沒有**權限管理機制的系統裡面，為了要限制只有管理員（_admin_）可以刪除文章，您可能會這樣做：

```
<% if current_user.role == :admin %>
  <%= link_to 'Destroy',
              article,
              method: :delete,
              data: { confirm: 'Are you sure?' } %>
<% end %>
```

如果有一天，行銷人員（_marketing_）也要可以刪除文章的話，您可能會修改如下：

```
<% if current_user.role == :admin ||
      current_user.role == :marketing %>
```

如果有一天，客服人員（_service_）也要可以刪除文章，然後你已經在 10 個 view 內都有寫下這句判斷式的話，很可能你就會有 10 個地方需要修改。而且你還沒有把握，是不是曾有同事在另外 3 個你不知道的地方，也寫下了這個判斷式，會成為漏網之魚。

這些都是造成 Bug 的來源，通常你很難立即察覺到。就算當初有寫測試，追查得到所有地點，但你仍然要改 13 個地方，也是很浪費時間。
如果今天換一種寫法：

```
<% if can? :destroy %>
  ...
<% end %>
```

讓 `can?` 這個 method 替你判斷當前的 **_User_** 是否具有 _:destroy_ 的權限，而未來權限條件有所變化時，你也只要修改一個地方，而不是 13 個。如此的美好境界，就是使用 [CanCanCan] 這個 gem 能夠帶來的效果，以下就開始帶領大家看看它是怎麼做到的。

<br>
#### **基於角色的權限系統（Role Based Authorization）**
<br>

_(以下的實作範例皆假設各位的專案已安裝 [Devise])_

首先，我們替 **_User_** 定義角色，本篇文章將會模擬一個新聞發布系統，所以我們定義管理員（_admin_） 與 作者（_author_）這兩種角色：

```
# app/models/user.rb
class User < ActiveRecord::Base
  ROLES = %i[admin author]
end
```

這裡為了講解上的方便，筆者使用類別常數直接定義，您可以依照自身的任務需求，將其定義為 **_Role_** model。

接著我們替資料庫內的 _users_ 表格加入 _role_ 欄位：

```
rails generate migration add_role_to_users role:string
rake db:migrate
```

接下來，為了示範方便，我們要在註冊畫面上便捷地提供一個能替 **_User_** 設定 Role 的 selector（當然，真實世界中的網站不會把這樣的功能放在這邊）：

![Role select](/assets/role_select.png)



為了要能夠修改 devise 提供的註冊畫面，我們得先把負責註冊的 controller 跟 view 產生出來，請執行：

```
$ rails g devise:controllers users
$ rails g devise:views users -v registrations
```

告訴 router 要使用客製的 **_Users::RegistrationsController_**：

```
# config/routes.rb
Rails.application.routes.draw do
  devise_for :users,
             controllers: { registrations: 'users/registrations' }
  root 'welcome#index'
end
```

修改 **_Users::RegistrationsController_** ，讓它在 _create_ action 的時候，可以替 **_User_** 儲存 _role_ 欄位，所以我們要先放行 _role_ 這個參數：

```
# app/controllers/users/registrations_controller.rb
before_action :configure_sign_up_params, only: [:create]
def configure_sign_up_params
  # 放行 [:role]
  devise_parameter_sanitizer.permit(:sign_up, keys: [:role])
end
```

然後修改註冊畫面的 _new.html.erb_：

```
# app/views/users/registrations/new.html.erb
<div class="field">
  <%= f.collection_select(:role,
                          User::ROLES,
                          :to_s,
                          lambda { |i| i.to_s.humanize }) %>
</div>
```

至此為止，您再回到註冊畫面就會看到 role selector 已經運作起來了。接下來，筆者要示範如何實際運用 [CanCanCan] 做權限控管了。


一個新聞發布網站，理應有一個 **_Article_** model，這裡我們先偷懶直接使用 scaffold 幫我們把所有事情做完：

```
$ rails g scaffold Article title:string content:text user:references
$ rails db:migrate
```

然後把 user 登入後的畫面導向文章列表 _articles#new_：

```
# config/routes.rb
authenticated :user do
  root to: 'articles#index', as: :authenticated_root
end
```

然後要做一件重要的事情，在 _articles#new_ 的畫面上不應該出現 user 這個欄位，而應該在 _articles#create_ 內直接將當前 user 的 id 寫入新文章內，請修改 _articles_controller.rb_ 如后：

```
# app/controller/articles_controller.rb
def create
  # 在此塞入 user_id
  @article = Article(article_params.merge!(user_id: current_user.id)
  ...
end

def article_params
  # 不再從外界接收 user_id
  params.require(:article).permit(:title, :content)
end
```

移除 **新增/編輯** 文章時出現的 user 欄位：

```
# app/view/articles/_form.html.erb
- <div class="field">
-   <%= form.label :user_id %>
-   <%= form.text_field :user_id,
-                       id: :article_user_id %>
- </div>
```

把原本會顯示 user id 的地方通通改為顯示 user name，並暫以 user 的 email 帳號作為 user name 的值：

```
# app/models/user.rb
def name
  email.gsub(/\A\w+[^@]/).first
end
```

```
# app/views/articles/index.html.erb
- <td><%= article.user %></td>
+ <td><%= article.user.name %></td>
```

```
# app/views/articles/show.html.erb
- <%= @article.user %>
+ <%= @article.user.name %>
```

限定只有登入過的 user 才可以使用 **_ArticlesController_**：

```
# app/controller/articles_controller.rb
before_action :authenticate_user!
```

接下來我們替網站建立一個 Dashboard，就是一般登入之後的後台管理畫面，通常可以用來觀看網站流量等數據。

建立 **_DashboardController_**：
```
#app/controllers/dashboard_controller.rb
class DashboardController < ApplicationController

  # 告訴 devise，此 DashboardController 的所有 action
  # 都必須要登入後的 user 才能使用
  before_action :authenticate_user!ƒ

  def index
  end
end
```

```
#app/views/dashboard/index.html.erb
<h1>Dashboard#index</h1>
<p>Find me in app/views/dashboard/index.html.erb></p>
```

在 _application.html.erb_ 加入 Dashboard 的連結，且只有登入後才看得見：
```
...
<body>
  <% if user_signed_in? %>
    <p><%= "Hello #{current_user.email}!" %></p>
    <%= link_to('Logout', destroy_user_session_path, method: :delete) %>
    <%= link_to('Dashboard', dashboard_index_path) %>
  <% else %>
    ...
  <% end %>

  ...
</body>
...
```

設定 _router.rb_
```
Rails.application.routes.draw do
  get 'dashboard/index'
  ...
end
```

接下來，就是重頭戲 **CanCanCan** 了。

<br>
#### **安裝 CanCanCan**
<br>

修改 Gemfile 後 `$ bundle install` ：

```
# Gemfile
gem 'cancancan', '~> 2.0'
```

然後執行：
```
$ rails g cancan:ability
```

CanCanCan 會幫我們產生一個 **_ability.rb_**，讓我們把所有權限的規則寫在裡面**集中管理**：

```
# app/models/ability.rb
class Ability
  include CanCan::Ability

  def initialize(user)
    # Define abilities for the passed in user here. For example:

    user ||= User.new # guest user (not logged in)

    if user.role == 'admin'

      # 只有 admin role 可以操作所有功能
      can :manage, :all

    else

      # 可以閱讀每個人的文章
      can :read, Article

      # 可以 update 自己的文章
      can :update, Article, user_id: user.id

      # 只有 markting role 可以拜訪 Dashboard
      can :index, DashboardController if user.role == 'marketing'
    end
  end
end
```

然後增加一個 _marketing_ 角色，他將可以拜訪 Dashboard：

```
class User < ApplicationRecord
  ROLES = %i[admin author marketing]
  ...
end
```

在 view 裡面使用 can? 檢查當前登入的 user 是否具有查看 Dashboard 的權限。如果沒有，就不要顯示 link：

```
# app/views/layouts/application.html.erb
...
<body>
  <% if user_signed_in? %>
    ...
    <%= link_to('Dashboard', dashboard_index_path) if can? :index, DashboardController %>
    ...
  <% else %>
    ...
  <% end %>
  ...
</body>
```

先在 **_dashboard#index_** 內加上 **authorize!**，確保只有具備權限的 user 可以使用這個 action。至於 **authorize!** 與 **can?** 的差別是：前者會 raise exception，後者不會。

```
class DashboardController < ApplicationController
  before_action :authenticate_user!
  def index
    authorize! :index, DashboardController
  end
end
```

還記得 **ability.rb** 內的這一行嗎？
```
can :update, Article, user_id: user.id
```
其最後一個參數 `user_id: user.id` 是條件，意思是：只有當 **_Article_** 的 _user_id_ 是 _user.id_（當前登入的使用者的 id） 的時候，才會具有 **_:update_** 的權限。這意味著只有 user 自己可以 update 自己建立的文章。配合這個設定，我們將可在 _app/views/articles/index.html.erb_ 內決定是否顯示 **edit** 與 **destroy** 的 link：

```
# app/views/articles/index.html.erb
...
<tbody>
  <% @articles.each do |article| %>
    <tr>
      ...
      <td><%= link_to 'Show', article %></td>
      <% if can? :update, article %>
        <td><%= link_to 'Edit', edit_article_path(article) %></td>
        <td><%= link_to 'Destroy',
                        article,
                        method: :delete,
                        data: { confirm: 'Are you sure?' } %></td>
      <% end %>
    </tr>
  <% end %>
</tbody>
...
```

您也可以在 _articles#index_ 內限制只會顯示當前使用者有權限看到的文章：

```
class ArticlesController < ApplicationController
  load_and_authorize_resource # 告訴 CanCanCan 替所有 action 加上 authorize!
  ...
  def index
    @articles = Article.accessible_by(current_ability)
  end
  ...
end
```

以上就完成了一個具體而微的權限系統，本範例完整的 Rails 專案 repo 在此：[canx3 on Github]


<br>
### **結語**

就筆者參與專案的經驗來說，像 **CanCanCan** 這樣的權限管理機制必須儘早引入，以避免權限判斷的程式碼四散於各處，所有的權限判斷式定義都要在 **app/models/ability.rb** 內做，外部只能使用 `can?` 與 `authorize!`，否則會引起日後維護上的大麻煩，並且要在 code review 的時候也要注意同事們的程式碼是否有符合這些原則。

類似於 CanCanCan 的還有一個叫做 [Pundit] 的 gem ，用途是一樣的，但使用概念上有些不同，有興趣的話請自行研究看看囉。

[canx3 on Github]:https://github.com/hugtrueme/canx3
[Pundit]:https://github.com/varvet/pundit
[CanCanCan]:https://github.com/CanCanCommunity/cancancan
[Devise]:https://github.com/plataformatec/devise
