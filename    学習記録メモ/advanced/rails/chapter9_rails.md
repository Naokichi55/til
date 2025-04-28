***
Active Storageにてリンク削除を個別にする方法

基礎
railsにて渡したパラメータを削除させたい。
現状は削除ボタンをクリックすると全て削除してしまう。
それを防ぐためにまずはコントローラーに個別のパラメーターを渡す方法ことが可能かで考えてる。
```
html.erb
<%= link_to("削除", "/users/#{user.id}/destroy", {method: "post"})%>

controller.rb
def destroy
 user User.find_by(id: params[:id])
 user.destroy
 redirect_to("/users/index")
end


今回だと送るパラメーターは下記になる


少し調べてみた結果
controller側でのdestory アクションを分け、その分けアクションに各アクションを飛ばしてやればいいのではと感じました。
現状destoryアクション内で条件分岐をしようとしてしていましたが、
link_toで送るアクション自体を変更すればいいのでは考えました。
参考にした記事は下記
https://stackoverflow.com/questions/49515529/rails-5-2-active-storage-purging-deleting-attachments

views側は下記
```
<%= link_to, 'Remove', delete_image_attachment_collections_url(image.
signed_id) 
                method: :delete,
                data: { confirm: 'Are you sure? } %>
```
controller側で定義する。
```
def delete_image_attacment
  @image = ActiveStorage::Blob.find_signed(params[:id])
  @image.purge
end
```
routes.rbはこうなる
```
resources :collections do
  member do
    delete :delete_image_attachment
  end
end
```




***
`maetadata` メソッド
 自身が持つメタデータ（他から任意の値を設定してもよい）を返す
***
***
case文
値と一致するか調べる場合に便利。

例)

coin == 0

if coin == 0
 puts "金がねーーー"
elsif coin == 100
 puts "100円もある"
elsif coin == 10
 puts "うまい棒買える。え今は11円ガーン"
else
 puts "金にとらわれる人生は嫌だ。"
end

coin == 0

case coin
  when 0 then
	  puts puts "金がねーーー"
	when 100
	  puts  "100円もある"
	when 10
	  puts "うまい棒変える。えっ今は11円ガーン"
	else
	   puts "金にとらわれる人生は嫌だ。"
end
***

```
class Site < ApplicationRecord
  has_many_attached :og_image
  has_many_attached :favicon
  has_many_attached :many_image

  validates :name, presence: true, length: { maximum: 100 }
  validates :subtitle, length: { maximum: 100 }
  validates :description, length: { maximum: 400 }
  validates :og_image, attachment: { purge: true, content_type: %r{\Aimage/(png|jpeg)\Z}, maximum: 524_288_000 }
  validates :favicon, attachment: { purge: true, content_type: %r{\Aimage/png\Z}, maximum: 524_288_000 }
end
```
has_many_attached は複数型にする必要があるため、変更。

***
```
<header class="jumbotron jumbotron-fluid"><div class="container"><h1><a href="/">Blog</a></h1><p class="lead">Very awesome!</p></div></header>

```

```
<div class="form-group file optional site_main_images"><label class="control-label file optional" for="site_main_images">Main images</label><input multiple="multiple" class="file optional" type="file" name="site[main_images][]" id="site_main_images"><p class="help-block">JPEG/PNG (1200x400)</p></div>

```
<div class="main_images_box"><div class="main_image"><img src="https://runteq-advanced-curriculum-db9c3f1a62da.herokuapp.com/rails/active_storage/representations/redirect/eyJfcmFpbHMiOnsibWVzc2FnZSI6IkJBaHBCZz09IiwiZXhwIjpudWxsLCJwdXIiOiJibG9iX2lkIn19--48513866ddf44cf33759b46818b19b8fa7077c3d/eyJfcmFpbHMiOnsibWVzc2FnZSI6IkJBaDdCem9MWm05eWJXRjBTU0lJY0c1bkJqb0dSVlE2QzNKbGMybDZaVWtpRERNd01IZ3hNREFHT3daVSIsImV4cCI6bnVsbCwicHVyIjoidmFyaWF0aW9uIn19--d24c51359cda7b8a7f33ad53875c025ba0d643cf/fruit_apple_illust_150.png"><a class="btn btn-danger" rel="nofollow" data-method="delete" href="/admin/site/attachments/1">削除</a></div></div>

<a class="btn btn-danger" rel="nofollow" data-method="delete" href="/admin/site/attachments/2">削除</a>



<img src="https://runteq-advanced-curriculum-db9c3f1a62da.herokuapp.com/rails/active_storage/representations/redirect/eyJfcmFpbHMiOnsibWVzc2FnZSI6IkJBaHBDQT09IiwiZXhwIjpudWxsLCJwdXIiOiJibG9iX2lkIn19--0230d848114df431fecee5b67f9f1ffa13f363be/eyJfcmFpbHMiOnsibWVzc2FnZSI6IkJBaDdCem9MWm05eWJXRjBTU0lJY0c1bkJqb0dSVlE2QzNKbGMybDZaVWtpQ2pNeWVETXlCanNHVkE9PSIsImV4cCI6bnVsbCwicHVyIjoidmFyaWF0aW9uIn19--3222c98f3b0398293ca28b266f8ac811bc6b19e9/fruit_apple_illust_150.png">



```
<img src="https://runteq-advanced-curriculum-db9c3f1a62da.herokuapp.com/rails/active_storage/representations/redirect/eyJfcmFpbHMiOnsibWVzc2FnZSI6IkJBaHBDQT09IiwiZXhwIjpudWxsLCJwdXIiOiJibG9iX2lkIn19--0230d848114df431fecee5b67f9f1ffa13f363be/eyJfcmFpbHMiOnsibWVzc2FnZSI6IkJBaDdCem9MWm05eWJXRjBTU0lJY0c1bkJqb0dSVlE2QzNKbGMybDZaVWtpQ2pNeWVETXlCanNHVkE9PSIsImV4cCI6bnVsbCwicHVyIjoidmFyaWF0aW9uIn19--3222c98f3b0398293ca28b266f8ac811bc6b19e9/fruit_apple_illust_150.png">

```


<header>
  <div class="swiper-container swiper-container-initialized swiper-container-horizontal">
	  <div class="swiper-wrapper" id="swiper-wrapper-ea9b8dadae423961" aria-live="off" style="transition: all; transform: translate3d(-1584px, 0px, 0px);"><img class="swiper-slide swiper-slide-duplicate swiper-slide-next swiper-slide-duplicate-prev" src="/images/cover.jpg" data-swiper-slide-index="0" role="group" aria-label="1 / 3" style="width: 792px;"><img class="swiper-slide swiper-slide-duplicate-active swiper-slide-prev swiper-slide-duplicate-next" src="/images/cover.jpg" data-swiper-slide-index="0" role="group" aria-label="2 / 3" style="width: 792px;"><img class="swiper-slide swiper-slide-duplicate swiper-slide-active swiper-slide-duplicate-prev" src="/images/cover.jpg" data-swiper-slide-index="0" role="group" aria-label="3 / 3" style="width: 792px;"></div><span class="swiper-notification" aria-live="assertive" aria-atomic="true"></span></div><div class="container blog-title"><h1><a href="/">Blog</a></h1><p class="lead">Very awesome!</p></div></header>
```

```
/ - if @site.main_images.attached?
      /   = image_tag @site.main_images_url('1200x400')
      /   br
      /   br
      / - if @site.main_images.attached?
      /   - if @site.main_images.each do |main_image|
      /     - if main_image.representable?
      /       = main_image.representation (resize_to_limit: [1200, 400])
```