```
autofocus:trueでページを読み込んだ後、クリックすることなく、カーソルが移動して入力待機状態を作るというメソッド

devise機能（ユーザー登録画面、ログイン画面作成のみ）で詰まったところ

1.sign_in画面の変更ができない。
- config/initializers/devise.rb
  config.scoped_views = trueを設定していなかったこと
- user_sessions/sessions/new.html.erbを作成し変更していた。
rails routes下記なのに。。。上記ならuser_session_sessionにならないとダメなのでは。
u
```
                                  Prefix Verb   URI Pattern                                                                                       Controller#Action
                        new_user_session GET    /users/sign_in(.:format)                                                                          devise/sessions#new
                            user_session POST   /users/sign_in(.:format)                                                                          devise/sessions#create
                    destroy_user_session DELETE /users/sign_out(.:format)                                                                         devise/sessions#destroy
```
参考にした方もきちんと下記のように記載いただいていたのに。。。。
app/views/"任意のフォルダ"/sessions/new.html.erb　というフォルダ、ファイルが生成されているので、こちらを編集します。

app/views/sessions/new.html.erbを編集しました。

2.NoMethodError in Devise::Registrations#new
controllerにてuser_nameを渡すメソッドになっていなかったために発生。
deviseだとデフォルトではemailとpassword認証のため変更が必要。
```
app/controllers/application_controller.rb
#変更前
class ApplicationController < ActionController::Base
  # Only allow modern browsers supporting webp images, web push, badges, import maps, CSS nesting, and CSS :has.
  allow_browser versions: :modern
end
```
```
#変更後
class ApplicationController < ActionController::Base
  before_action :configure_permitted_parameters, if: :devise_controller?
  # Only allow modern browsers supporting webp images, web push, badges, import maps, CSS nesting, and CSS :has.
  allow_browser versions: :modern

  protected
  def configure_permitted_parameters
    devise_parameter_sanitizer.permit(:sign_up, keys: [:user_name, :email])
  end
end
```

3.NoMethodError in Devise::Sessions#new
app/views/users/registrations/new.html.erb
app/views/users/sessions/new.html.erb
を編集しました。
```
#　変更前
<%= form_for(resource, as: resource_name, url: session_path(resource_name)) do |f| %>
```
```
#変更後
<%= form_with model: @user, url: user_session_path do |f| %>
```
form_forはRails5以前の書き方らしくRails5.1以降ではform_withにて記載するようにする。

参考サイト：
http://zenn.dev/ganmo3/articles/8ae23a01dd1206
https://qiita.com/39ppp/items/479b8d0c5dacbb2a195c
