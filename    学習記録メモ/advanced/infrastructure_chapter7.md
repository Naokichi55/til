
` /var/www/Naokichi55/deploy_sample



```
# MySQL. Versions 5.5.8 and up are supported.
#
# Install the MySQL driver
#   gem install mysql2
#
# Ensure the MySQL gem is defined in your Gemfile
#   gem "mysql2"
#
# And be sure to use new-style password hashing:
#   https://dev.mysql.com/doc/refman/5.7/en/password-hashing.html
#
default: &default
  adapter: mysql2
  encoding: utf8mb4
  pool: <%= ENV.fetch("RAILS_MAX_THREADS") { 5 } %>
  username: root
  password: password
  host: db


development:
  <<: *default
  database: v3_basic_rails_basic_development

# Warning: The database defined as "test" will be erased and
# re-generated from your development database when you run "rake".
# Do not set this db to the same as development or production.
test:
  <<: *default
  database: v3_basic_rails_basic_test

# As with config/credentials.yml, you never want to store sensitive information,
# like your database password, in your source code. If your source code is
# ever seen by anyone, they now have access to your database.
#
# Instead, provide the password or a full connection URL as an environment
# variable when you boot the app. For example:
#
#   DATABASE_URL="mysql2://myuser:mypass@localhost/somedatabase"
#
# If the connection URL is provided in the special DATABASE_URL environment
# variable, Rails will automatically merge its configuration values on top of
# the values provided in this file. Alternatively, you can specify a connection
# URL environment variable explicitly:
#
#   production:
#     url: <%= ENV["MY_APP_DATABASE_URL"] %>
#
# Read https://guides.rubyonrails.org/configuring.html#configuring-a-database
# for a full overview on how database connection configuration can be specified.
#
production:
  <<: *default
  database: v3_basic_rails_basic_production
  username: v3_basic_rails_basic
  password: <%= ENV["V3_BASIC_RAILS_BASIC_DATABASE_PASSWORD"] %>
```
↓
default: &default
  adapter: mysql2
  encoding: utf8mb4
  charset: utf8mb4
  collation: utf8mb4_general_ci
  username: admin
  password: Naoyuki0618
  host: database-1.ch6uwiqceji4.ap-northeast-1.rds.amazonaws.com
  pool: 5
  timeout: 5000

production:
  <<: *default
  database: runteq_production
```


# つまづいたところ！！
1. port22 閉鎖
そのためssh接続ができない！！
解決策
インスタンスの内のインバウンドルールにて、ssh port22の設定をしていなかった

２.

```
vim config/database.yml

default: &default　←記載漏れ
  adapter: mysql2　
	#adapter: mysql2 ←重複記載
  encoding: utf8mb4 ←utf8mbとの記載「4」がない
  charset: utf8mb4
  collation: utf8mb4_general_ci
  username: admin
  password: Naoyuki0618
  host: database-1.ch6uwiqceji4.ap-northeast-1.rds.amazonaws.com
  pool: 5
  timeout: 5000

production:
  <<: *default
  database: runteq_production_v1

```

３.

```
[ec2-user@ip-172-31-47-1 v3_deploy_sample_app]$ cd /etc/nginx/conf.d
[ec2-user@ip-172-31-47-1 conf.d]$ sudo vi runteq-infra.conf

#sudo vi runteq-infra.conf

error_log  /var/www/v3_deploy_sample_app/log/nginx.error.log; ←「3_deploy_sample_app」となっていた
access_log /var/www/v3_deploy_sample_app/log/nginx.access.log;←「3_deploy_sample_app」となっていた

client_max_body_size 2G;
upstream runteq-app {
  server unix:///var/www/v3_deploy_sample_app/tmp/sockets/puma.sock fail_timeout=0;
}
server {
  listen 80;
  server_name 18.182.154.198; # 作成したEC2の ElasticIPアドレス。← http://18.182.154.198となっていた
  keepalive_timeout 5;
  root /var/www/v3_deploy_sample_app/public;
  try_files $uri/index.html $uri.html $uri @app;
  location @app {
    proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
    proxy_set_header Host $http_host;
    proxy_redirect off;
    proxy_pass http://runteq-app;
  }

  error_page 500 502 503 504 /500.html;
  location = /500.html {
      root /var/www/v3_deploy_sample_app/public;
  }
}
```

3 Rails 起動後18.182.154.198にアクセルすると下記エラーのため表示されない
The page you were looking for doesn't exist.
You may have mistyped the address or the page may have moved.
If you are the application owner check the logs for more information.

やったこと
 cat runteq-infra.conf　で状態確認

 systemctl status nginx.serviceで nginxの確認（ActiveなのでOK）railsの表示の時点でOKだがねんのため確認
● nginx.service - The nginx HTTP and reverse proxy server
   Loaded: loaded (/usr/lib/systemd/system/nginx.service; disabled; vendor preset: disabled)
   Active: active (running) since 土 2025-03-15 07:47:01 UTC; 58s ago
  Process: 4007 ExecStart=/usr/sbin/nginx (code=exited, status=0/SUCCESS)
  Process: 4002 ExecStartPre=/usr/sbin/nginx -t (code=exited, status=0/SUCCESS)
  Process: 4001 ExecStartPre=/usr/bin/rm -f /run/nginx.pid (code=exited, status=0/SUCCESS)
 Main PID: 4009 (nginx)
   CGroup: /system.slice/nginx.service
           ├─4009 nginx: master process /usr/sbin/nginx
           └─4010 nginx: worker process

tail -f log/production.logにてログの確認

[ec2-user@ip-172-31-47-1 v3_deploy_sample_app]$ tail -f log/production.log

):
[cd0639d8-5305-43e2-8bf5-ac091ea5d7a2]     3: <h1>Posts</h1>
[cd0639d8-5305-43e2-8bf5-ac091ea5d7a2]     4: 
[cd0639d8-5305-43e2-8bf5-ac091ea5d7a2]     5: <div id="posts">
[cd0639d8-5305-43e2-8bf5-ac091ea5d7a2]     6:   <% @posts.each do |post| %>
[cd0639d8-5305-43e2-8bf5-ac091ea5d7a2]     7:     <%= render post %>
[cd0639d8-5305-43e2-8bf5-ac091ea5d7a2]     8:     <p>
[cd0639d8-5305-43e2-8bf5-ac091ea5d7a2]     9:       <%= link_to "Show this post", post %>
[cd0639d8-5305-43e2-8bf5-ac091ea5d7a2]   
[cd0639d8-5305-43e2-8bf5-ac091ea5d7a2] app/views/posts/index.html.erb:6
^C  


vim log/production.logでログの確認
```

[ec2-user@ip-172-31-47-1 v3_deploy_sample_app]$ vim log/production.log
[ec2-user@ip-172-31-47-1 v3_deploy_sample_app]$ 
[ec2-user@ip-172-31-47-1 v3_deploy_sample_app]$ 
[ec2-user@ip-172-31-47-1 v3_deploy_sample_app]$ vim config/database.yml
[ec2-user@ip-172-31-47-1 v3_deploy_sample_app]$ 
[ec2-user@ip-172-31-47-1 v3_deploy_sample_app]$ 
[ec2-user@ip-172-31-47-1 v3_deploy_sample_app]$ vim config/database.yml
[ec2-user@ip-172-31-47-1 v3_deploy_sample_app]$ vim config/database.yml
[ec2-user@ip-172-31-47-1 v3_deploy_sample_app]$ 
[ec2-user@ip-172-31-47-1 v3_deploy_sample_app]$ 
[ec2-user@ip-172-31-47-1 v3_deploy_sample_app]$ vim log/production.log
[ec2-user@ip-172-31-47-1 v3_deploy_sample_app]$ 
[ec2-user@ip-172-31-47-1 v3_deploy_sample_app]$ 
[ec2-user@ip-172-31-47-1 v3_deploy_sample_app]$ tail -f log/production.log



ElasticIPのアドレスにアクセスできない
Rails

```
[cd0639d8-5305-43e2-8bf5-ac091ea5d7a2]     3: <h1>Posts</h1>
[cd0639d8-5305-43e2-8bf5-ac091ea5d7a2]     4: 
[cd0639d8-5305-43e2-8bf5-ac091ea5d7a2]     5: <div id="posts">
[cd0639d8-5305-43e2-8bf5-ac091ea5d7a2]     6:   <% @posts.each do |post| %>
[cd0639d8-5305-43e2-8bf5-ac091ea5d7a2]     7:     <%= render post %>
[cd0639d8-5305-43e2-8bf5-ac091ea5d7a2]     8:     <p>
[cd0639d8-5305-43e2-8bf5-ac091ea5d7a2]     9:       <%= link_to "Show this post", post %>
[cd0639d8-5305-43e2-8bf5-ac091ea5d7a2]   
[cd0639d8-5305-43e2-8bf5-ac091ea5d7a2] app/views/posts/index.html.erb:6


^C  
[ec2-user@ip-172-31-47-1 v3_deploy_sample_app]$ vim log/production.log
[ec2-user@ip-172-31-47-1 v3_deploy_sample_app]$ 
[ec2-user@ip-172-31-47-1 v3_deploy_sample_app]$ 
[ec2-user@ip-172-31-47-1 v3_deploy_sample_app]$ vim config/database.yml
[ec2-user@ip-172-31-47-1 v3_deploy_sample_app]$ 
[ec2-user@ip-172-31-47-1 v3_deploy_sample_app]$ 
[ec2-user@ip-172-31-47-1 v3_deploy_sample_app]$ vim config/database.yml
[ec2-user@ip-172-31-47-1 v3_deploy_sample_app]$ vim config/database.yml
[ec2-user@ip-172-31-47-1 v3_deploy_sample_app]$ 
[ec2-user@ip-172-31-47-1 v3_deploy_sample_app]$ 
[ec2-user@ip-172-31-47-1 v3_deploy_sample_app]$ vim log/production.log
[ec2-user@ip-172-31-47-1 v3_deploy_sample_app]$ 
[ec2-user@ip-172-31-47-1 v3_deploy_sample_app]$ 
[ec2-user@ip-172-31-47-1 v3_deploy_sample_app]$ tail -f log/production.log
I, [2025-03-15T12:28:14.719158 #5503]  INFO -- : Migrating to CreatePosts (20230831110140)
I, [2025-03-15T12:29:37.403689 #5614]  INFO -- : [36d0cd8f-ca5b-48d7-abf1-faede4766a94] Started GET "/posts" for 220.153.101.204 at 2025-03-15 12:29:37 +0000
I, [2025-03-15T12:29:37.409077 #5614]  INFO -- : [36d0cd8f-ca5b-48d7-abf1-faede4766a94] Processing by PostsController#index as HTML
I, [2025-03-15T12:29:37.442939 #5614]  INFO -- : [36d0cd8f-ca5b-48d7-abf1-faede4766a94]   Rendered posts/index.html.erb within layouts/application (Duration: 30.0ms | Allocations: 1824)
I, [2025-03-15T12:29:37.445604 #5614]  INFO -- : [36d0cd8f-ca5b-48d7-abf1-faede4766a94]   Rendered layout layouts/application.html.erb (Duration: 32.7ms | Allocations: 2583)
I, [2025-03-15T12:29:37.445864 #5614]  INFO -- : [36d0cd8f-ca5b-48d7-abf1-faede4766a94] Completed 200 OK in 37ms (Views: 34.4ms | ActiveRecord: 2.1ms | Allocations: 4140)
I, [2025-03-15T12:39:19.146037 #5614]  INFO -- : [caa1aefb-2d11-42fa-b85d-bf8cc4cb9f54] Started GET "/" for 54.196.247.85 at 2025-03-15 12:39:19 +0000
F, [2025-03-15T12:39:19.146743 #5614] FATAL -- : [caa1aefb-2d11-42fa-b85d-bf8cc4cb9f54]   
[caa1aefb-2d11-42fa-b85d-bf8cc4cb9f54] ActionController::RoutingError (No route matches [GET] "/"):
[caa1aefb-2d11-42fa-b85d-bf8cc4cb9f54]   
^C      
[ec2-user@ip-172-31-47-1 v3_deploy_sample_app]$ vim log/production.log
[ec2-user@ip-172-31-47-1 v3_deploy_sample_app]$ 
[ec2-user@ip-172-31-47-1 v3_deploy_sample_app]$ vim log/production.log

[423eac27-aa54-4a9d-807f-574a7f023119]
I, [2025-03-15T11:30:21.668242 #4921]  INFO -- : [5af9cdf0-b066-49b3-995c-311f96c63ae4] Started GET "/" for 87.236.176.161 at 2025-03-15 11:30:21 +0000
F, [2025-03-15T11:30:21.668553 #4921] FATAL -- : [5af9cdf0-b066-49b3-995c-311f96c63ae4]
[5af9cdf0-b066-49b3-995c-311f96c63ae4] ActionController::RoutingError (No route matches [GET] "/"):
[5af9cdf0-b066-49b3-995c-311f96c63ae4]
I, [2025-03-15T11:48:44.409861 #4921]  INFO -- : [a534f3f6-daa5-4a3b-baf5-f85973e286cf] Started GET "/.env" for 196.251.86.78 at 2025-03-15 11:48:44 +0000
F, [2025-03-15T11:48:44.410226 #4921] FATAL -- : [a534f3f6-daa5-4a3b-baf5-f85973e286cf]
[a534f3f6-daa5-4a3b-baf5-f85973e286cf] ActionController::RoutingError (No route matches [GET] "/.env"):
[a534f3f6-daa5-4a3b-baf5-f85973e286cf]
I, [2025-03-15T11:54:40.548621 #4921]  INFO -- : [5000d11e-e3a0-4db2-9d86-bd4ffdd663a6] Started GET "/posts" for 220.153.101.204 at 2025-03-15 11:54:40 +0000
I, [2025-03-15T11:54:40.549238 #4921]  INFO -- : [5000d11e-e3a0-4db2-9d86-bd4ffdd663a6] Processing by PostsController#index as HTML
I, [2025-03-15T11:56:40.561454 #4921]  INFO -- : [5000d11e-e3a0-4db2-9d86-bd4ffdd663a6]   Rendered posts/index.html.erb within layouts/application (Duration: 120011.8ms | Allocations: 498)
I, [2025-03-15T11:56:40.561552 #4921]  INFO -- : [5000d11e-e3a0-4db2-9d86-bd4ffdd663a6]   Rendered layout layouts/application.html.erb (Duration: 120011.9ms | Allocations: 529)
I, [2025-03-15T11:56:40.561668 #4921]  INFO -- : [5000d11e-e3a0-4db2-9d86-bd4ffdd663a6] Completed 500 Internal Server Error in 120012ms (Allocations: 742)
F, [2025-03-15T11:56:40.562229 #4921] FATAL -- : [5000d11e-e3a0-4db2-9d86-bd4ffdd663a6]
[5000d11e-e3a0-4db2-9d86-bd4ffdd663a6] ActionView::Template::Error (There is an issue connecting with your hostname: database-1.ch6uwiqceji4.ap-northeast-1.rds.amazonaws.com.

Please check your database configuration and ensure there is a valid connection to your database.
):
[5000d11e-e3a0-4db2-9d86-bd4ffdd663a6]     3: <h1>Posts</h1>
[5000d11e-e3a0-4db2-9d86-bd4ffdd663a6]     4:
[5000d11e-e3a0-4db2-9d86-bd4ffdd663a6]     5: <div id="posts">
[5000d11e-e3a0-4db2-9d86-bd4ffdd663a6]     6:   <% @posts.each do |post| %>
[5000d11e-e3a0-4db2-9d86-bd4ffdd663a6]     7:     <%= render post %>
[5000d11e-e3a0-4db2-9d86-bd4ffdd663a6]     8:     <p>
[5000d11e-e3a0-4db2-9d86-bd4ffdd663a6]     9:       <%= link_to "Show this post", post %>
[5000d11e-e3a0-4db2-9d86-bd4ffdd663a6]
[5000d11e-e3a0-4db2-9d86-bd4ffdd663a6] app/views/posts/index.html.erb:6
I, [2025-03-15T12:21:07.252506 #5409]  INFO -- : [d3e8c4c9-34b3-4603-a4b6-0bf83bb28e27] Started GET "/posts" for 220.153.101.204 at 2025-03-15 12:21:07 +0000
I, [2025-03-15T12:21:07.257409 #5409]  INFO -- : [d3e8c4c9-34b3-4603-a4b6-0bf83bb28e27] Processing by PostsController#index as HTML
I, [2025-03-15T12:21:16.560094 #5409]  INFO -- : [e7b69694-7050-4eab-bd1a-1bd7cad1daf9] Started GET "/posts" for 220.153.101.204 at 2025-03-15 12:21:16 +0000
I, [2025-03-15T12:21:16.560747 #5409]  INFO -- : [e7b69694-7050-4eab-bd1a-1bd7cad1daf9] Processing by PostsController#index as HTML
I, [2025-03-15T12:21:38.904553 #5409]  INFO -- : [d3e8c4c9-34b3-4603-a4b6-0bf83bb28e27]   Rendered posts/index.html.erb within layouts/application (Duration: 31643.7ms | Allocations: 2123)
I, [2025-03-15T12:21:38.904646 #5409]  INFO -- : [d3e8c4c9-34b3-4603-a4b6-0bf83bb28e27]   Rendered layout layouts/application.html.erb (Duration: 31643.9ms | Allocations: 2181)
I, [2025-03-15T12:21:38.904862 #5409]  INFO -- : [d3e8c4c9-34b3-4603-a4b6-0bf83bb28e27] Completed 500 Internal Server Error in 31647ms (Allocations: 3673)
F, [2025-03-15T12:21:38.906533 #5409] FATAL -- : [d3e8c4c9-34b3-4603-a4b6-0bf83bb28e27]
[d3e8c4c9-34b3-4603-a4b6-0bf83bb28e27] ActionView::Template::Error (We could not find your database: runteq_production_v1. Which can be found in the database configuration file located at config/database.yml.


I, [2025-03-15T12:21:48.025348 #5409]  INFO -- : [e7b69694-7050-4eab-bd1a-1bd7cad1daf9] Completed 500 Internal Server Error in 31465ms (Allocations: 1942)
F, [2025-03-15T12:21:48.025887 #5409] FATAL -- : [e7b69694-7050-4eab-bd1a-1bd7cad1daf9]
[e7b69694-7050-4eab-bd1a-1bd7cad1daf9] ActionView::Template::Error (We could not find your database: runteq_production_v1. Which can be found in the database configuration file located at config/database.yml.

To resolve this issue:

- Did you create the database for this app, or delete it? You may need to create your database.
- Has the database name changed? Check your database.yml config has the correct database name.

To create your database, run:

        bin/rails db:create
):
[e7b69694-7050-4eab-bd1a-1bd7cad1daf9]     3: <h1>Posts</h1>
[e7b69694-7050-4eab-bd1a-1bd7cad1daf9]     4:
[e7b69694-7050-4eab-bd1a-1bd7cad1daf9]     5: <div id="posts">
[e7b69694-7050-4eab-bd1a-1bd7cad1daf9]     6:   <% @posts.each do |post| %>
[e7b69694-7050-4eab-bd1a-1bd7cad1daf9]     7:     <%= render post %>
[e7b69694-7050-4eab-bd1a-1bd7cad1daf9]     8:     <p>
[e7b69694-7050-4eab-bd1a-1bd7cad1daf9]     9:       <%= link_to "Show this post", post %>
[e7b69694-7050-4eab-bd1a-1bd7cad1daf9]
[e7b69694-7050-4eab-bd1a-1bd7cad1daf9] app/views/posts/index.html.erb:6
I, [2025-03-15T12:21:48.075283 #5409]  INFO -- : [cd0639d8-5305-43e2-8bf5-ac091ea5d7a2] Started GET "/posts" for 220.153.101.204 at 2025-03-15 12:21:48 +0000
I, [2025-03-15T12:21:48.075856 #5409]  INFO -- : [cd0639d8-5305-43e2-8bf5-ac091ea5d7a2] Processing by PostsController#index as HTML
I, [2025-03-15T12:21:48.093434 #5409]  INFO -- : [cd0639d8-5305-43e2-8bf5-ac091ea5d7a2]   Rendered posts/index.html.erb within layouts/application (Duration: 17.2ms | Allocations: 273)
I, [2025-03-15T12:21:48.093490 #5409]  INFO -- : [cd0639d8-5305-43e2-8bf5-ac091ea5d7a2]   Rendered layout layouts/application.html.erb (Duration: 17.3ms | Allocations: 304)
I, [2025-03-15T12:21:48.093580 #5409]  INFO -- : [cd0639d8-5305-43e2-8bf5-ac091ea5d7a2] Completed 500 Internal Server Error in 18ms (Allocations: 494)
F, [2025-03-15T12:21:48.094096 #5409] FATAL -- : [cd0639d8-5305-43e2-8bf5-ac091ea5d7a2]
[cd0639d8-5305-43e2-8bf5-ac091ea5d7a2] ActionView::Template::Error (We could not find your database: runteq_production_v1. Which can be found in the database configuration file located at config/database.yml.

To resolve this issue:

- Did you create the database for this app, or delete it? You may need to create your database.
- Has the database name changed? Check your database.yml config has the correct database name.

To create your database, run:

        bin/rails db:create
):
[cd0639d8-5305-43e2-8bf5-ac091ea5d7a2]     3: <h1>Posts</h1>
[cd0639d8-5305-43e2-8bf5-ac091ea5d7a2]     4:
[cd0639d8-5305-43e2-8bf5-ac091ea5d7a2]     5: <div id="posts">
[cd0639d8-5305-43e2-8bf5-ac091ea5d7a2]     6:   <% @posts.each do |post| %>
[cd0639d8-5305-43e2-8bf5-ac091ea5d7a2]     7:     <%= render post %>
[cd0639d8-5305-43e2-8bf5-ac091ea5d7a2]     8:     <p>
[cd0639d8-5305-43e2-8bf5-ac091ea5d7a2]     9:       <%= link_to "Show this post", post %>
[cd0639d8-5305-43e2-8bf5-ac091ea5d7a2]
[cd0639d8-5305-43e2-8bf5-ac091ea5d7a2] app/views/posts/index.html.erb:6
I, [2025-03-15T12:23:26.177873 #5409]  INFO -- : [0f13324f-0d46-4310-a500-fd2d961f85bf] Started GET "/" for 196.216.56.126 at 2025-03-15 12:23:26 +0000
F, [2025-03-15T12:23:26.178914 #5409] FATAL -- : [0f13324f-0d46-4310-a500-fd2d961f85bf]
[0f13324f-0d46-4310-a500-fd2d961f85bf] ActionController::RoutingError (No route matches [GET] "/"):
[0f13324f-0d46-4310-a500-fd2d961f85bf]
I, [2025-03-15T12:28:14.719158 #5503]  INFO -- : Migrating to CreatePosts (20230831110140)
I, [2025-03-15T12:29:37.403689 #5614]  INFO -- : [36d0cd8f-ca5b-48d7-abf1-faede4766a94] Started GET "/posts" for 220.153.101.204 at 2025-03-15 12:29:37 +0000
I, [2025-03-15T12:29:37.409077 #5614]  INFO -- : [36d0cd8f-ca5b-48d7-abf1-faede4766a94] Processing by PostsController#index as HTML
I, [2025-03-15T12:29:37.442939 #5614]  INFO -- : [36d0cd8f-ca5b-48d7-abf1-faede4766a94]   Rendered posts/index.html.erb within layouts/application (Duration: 30.0ms | Allocations: 1824)
I, [2025-03-15T12:29:37.445604 #5614]  INFO -- : [36d0cd8f-ca5b-48d7-abf1-faede4766a94]   Rendered layout layouts/application.html.erb (Duration: 32.7ms | Allocations: 2583)
I, [2025-03-15T12:29:37.445864 #5614]  INFO -- : [36d0cd8f-ca5b-48d7-abf1-faede4766a94] Completed 200 OK in 37ms (Views: 34.4ms | ActiveRecord: 2.1ms | Allocations: 4140)
I, [2025-03-15T12:39:19.146037 #5614]  INFO -- : [caa1aefb-2d11-42fa-b85d-bf8cc4cb9f54] Started GET "/" for 54.196.247.85 at 2025-03-15 12:39:19 +0000
F, [2025-03-15T12:39:19.146743 #5614] FATAL -- : [caa1aefb-2d11-42fa-b85d-bf8cc4cb9f54]
[caa1aefb-2d11-42fa-b85d-bf8cc4cb9f54] ActionController::RoutingError (No route matches [GET] "/"):
```
Please check your database configuration and ensure there is a valid connection to your database.よりデータベースがおかしいのではと思ったので
DBのインスタンスを確認したが結局分からず。

解決策
 rails db:migrate RAILS_ENV=production
 を行うと治った。

 編集でDBを変更したこともあったので、
 # DBを作成
[ec2-user@ip... `アプリ名`]$ rails db:create RAILS_ENV=production
# テーブル作成
[ec2-user@ip... `アプリ名`]$ rails db:migrate RAILS_ENV=production
をやると
$ rails db:migrate RAILS_ENV=production
にてテーブルが作成されるのを確認した。


アプリ名:v3_deploy_sample_app
ユーザー名
admin
password
Naoyuki0618