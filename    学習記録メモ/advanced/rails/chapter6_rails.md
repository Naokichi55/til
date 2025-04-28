

namespaceを使うといいかも。
権限でグループ分けができる
ユーザー更新の制御
```
# app/policies/post_policy.rb

class PostPolicy < ApplicationPolicy
  def permitted_attributes
	  if user.admin? || user.owe_of?(post)
		  [:title, :body, :tag_list]
		else
		  [:tag_list]
		end
	end
end

```
controller.rbの設定
```
app/controllers/posts_controller.rb
  class PostController < ApplicationController
	  def update
		  @post = Post.find(params[:id])
			if @post.update(permitted_attributes(@post))
			  redirect_to @post
			else
			  render :edit
			end
		end
	end
```





⭐️HTTP 403を発行させる。

applocation_controller.rbに下記を記載する
```
config.action_dispatch.rescue_responses["Pundit::NotAuthorizedError"] = :forbidden
```


HTTP の 403 Forbidden クライアントエラーレスポンスステータスコードは、サーバーがリクエストを理解したものの、処理を拒否したことを示します。 このステータスは 401 と似ていますが、 403 Forbidden レスポンスが異なるのは、認証または再認証を行っても違いがないことです。 リクエストの失敗は、リソースに対するその権限の不足やアクションなどのアプリケーションロジックに関連したものです。
下記より
https://developer.mozilla.org/ja/docs/Web/HTTP/Status/403

403エラー表示画面について
下記ファイルを参照しました。

https://zenn.dev/yusuke_docha/articles/7b4b2f3f1bb203