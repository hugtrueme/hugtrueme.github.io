---
layout: post
title:  "速覽 Active Storage"
date:   2020-01-12 10:16:00 +0800
categories: Rails
tags: ActiveStroage
---

#### 初始設定
```
# Create two table: active_storage_blobs, active_storage_attachments

$ rails active_storage:install
```

#### 設定存放檔案的地方
```
#config/storage.yml

local:
  service: Disk
  root: <%= Rails.root.join("storage") %>

test:
  service: Disk
  root: <%= Rails.root.join("tmp/storage") %>

amazon:
  service: S3
  access_key_id: ""
  secret_access_key: ""
```

開發環境使用本機資料夾
```
# config/environments/development.rb

config.active_storage.service = :local
```

正式環境使用 S3
```
# config/environments/production.rb

config.active_storage.service = :amazon
```

欲使用 S3 的話，記得要安裝 gem
```
# Gemfile

# For Amazon S3
gem "aws-sdk-s3", require: false
```


#### 設定 model 與檔案的關聯

一對一關聯
```
# app/models/user.rb

class User < ApplicaitonRecord
  has_on_attached :avatar # 頭像圖檔
end
```
```
class SignupController < ApplicationController
  def create
    user = User.create!(user_params)
    session[:user_id] = user.id
    redirect_to root_path
  end

  private
    def user_params
      params.require(:user).permit(:email_address, :password, :avatar)
    end
end
```
```
# 將 user 與新的 avatar 關聯起來
Current.user.avatar.attach(params[:avatar])

# 檢查 avatar 存在否？
Current.user.avatar.attached?
```

一對多關聯
```
# app/models/message.rb

class Message < ApplicationRecord
  # 注意要加 s
  has_many_attached :images
end
```

```
class MessagesController < ApplicationController
  def create
    message = Message.create!(message_params)
    redirect_to message
  end

  private
    def message_params
      # 注意此處 images 參數為陣列
      params.require(:message).permit(:title, :content, images: [])
    end
end
```

```
@message.images.attach(params[:images])
@message.images.attached?
```

#### 刪除檔案
```
# 直接刪除檔案，若會拖慢網頁回應，可改用 purge_later 經由 Active Job 刪除關聯檔案
user.avatar.purge

# 經由 Active Job 刪除關聯檔案
user.avatar.purge_later
```

#### 產生檔案超連結
```
# 在 controller 內 redirect 用（？？）
url_for(user.avatar)

# 提供下載連結
rails_blob_path(user.avatar, disposition: "attachment”)

# 從除了 controller/view 以外的地方（例如：Job 內）
Rails.application.routes.url_helpers.rails_blob_path(user.avatar, only_path: true)
```

#### 從 Server 內處理檔案
```
# 取得二進位物件

binary = user.avatar.download

# 對檔案掃毒
class VirusScanner
  include ActiveStorage::Downloading

  attr_reader :blob


  def initialize(blob)
    @blob = blob
  end


  def scan
    # 預設會把檔案下載到 Dir.tmpdir 的路徑
    download_blob_to_tempfile do |file|
      system 'scan_virus', file.path
    end
  end

  private

  # 如果想要下載到別的地方請 override AactiveStorage::Downloading#tempdir
  def tmpdir
    ‘/path/to/tmp’
  end
end
```

#### 取得圖檔的縮圖
```
# Gemfile

gem ‘mini_magick’

# 當瀏覽器觸碰到這個 URL，Active Storage 才會轉換出你要的縮圖，
# 再將瀏覽器導過去（Lazy式的行為）
<%= image_tag user.avatar.variant(resize: "100x100") %>
```

預覽檔案
```
# Active Storage 支援預覽 videos 與 PDF 文件。

<ul>
  <% @message.files.each do |file| %>
    <li>
      <%= image_tag file.preview(resize: "100x100>") %>
    </li>
  <% end %>
</ul>
```

#### 前端的設定

asset popeline
```
# app/assets/javascripts/application.js

//= require activestorage
```

若使用 npm package
```
import * as ActiveStorage from "activestorage"

ActiveStorage.start()
```

在 form.html.erb 內
```
<%= form.file_field :avatar, multiple: false, direct_upload: true %>
```

若想在圖檔上傳前，先從瀏覽器端預覽圖檔
```
<script src="https://ajax.googleapis.com/ajax/libs/jquery/2.1.1/jquery.min.js"></script>

<form runat="server">
  <input type='file' id="imgInp" />
  <img id="blah" src="#" alt="your image" />
</form>

<script>
function readURL(input) {
  if (input.files && input.files[0]) {
    var reader = new FileReader();

    reader.onload = function(e) {
      $('#blah').attr('src', e.target.result);
    }

    reader.readAsDataURL(input.files[0]);
  }
}

$("#imgInp").change(function() {
  readURL(this);
});
<script>
```

#### 如使用外部貯存服務，記得要處理 CORS 組態

各家服務的 [CORS] 文件
* [S3]
* [Google Cloud Storage]
* [Azure Storage]

```
# S3 CROS 組態
<?xml version="1.0" encoding="UTF-8"?>
<CORSConfiguration xmlns="http://s3.amazonaws.com/doc/2006-03-01/">
<CORSRule>
    <AllowedOrigin>https://www.example.com</AllowedOrigin>
    <AllowedMethod>PUT</AllowedMethod>
    <AllowedHeader>Origin</AllowedHeader>
    <AllowedHeader>Content-Type</AllowedHeader>
    <AllowedHeader>Content-MD5</AllowedHeader>
    <AllowedHeader>Content-Disposition</AllowedHeader>
    <MaxAgeSeconds>3600</MaxAgeSeconds>
</CORSRule>
</CORSConfiguration>
```
```
# Google Cloud Storage CORS 組態
[
  {
    "origin": ["https://www.example.com"],
    "method": ["PUT"],
    "responseHeader": ["Origin", "Content-Type", "Content-MD5", "Content-Disposition"],
    "maxAgeSeconds": 3600
  }
]
```
```
# Azure Storage CORS 組態
<Cors>
  <CorsRule>
    <AllowedOrigins>https://www.example.com</AllowedOrigins>
    <AllowedMethods>PUT</AllowedMethods>
    <AllowedHeaders>Origin, Content-Type, Content-MD5, x-ms-blob-content-disposition, x-ms-blob-type</AllowedHeaders>
    <MaxAgeInSeconds>3600</MaxAgeInSeconds>
  </CorsRule>
<Cors>
```

[CORS]:https://developer.mozilla.org/zh-TW/docs/Web/HTTP/CORS
[S3]:https://docs.aws.amazon.com/AmazonS3/latest/dev/cors.html#how-do-i-enable-cors
[Google Cloud Storage]:https://cloud.google.com/storage/docs/configuring-cors
[Azure Storage]:https://docs.microsoft.com/en-us/rest/api/storageservices/cross-origin-resource-sharing--cors--support-for-the-azure-storage-services
