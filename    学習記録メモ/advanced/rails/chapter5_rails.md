`scope`
モデルに対してよく使うクエリーを簡単に呼び出すことができるようにする仕組み
クエリーとはDBへの指示。DBのレコードを検索するための操作

実際のコード
```
scope: by_category → {(category_id)(where(category_id: category_id))}
```
scope ActiveRecordのscopeを定義
:by_category  scopeの名前
{ }内に
(category_id)  scopeの引数
(where (category_id: :category_id))  category_idへのクエリー SQLのwhere句にてcategory_idを絞り込んでいる。



app/models/article.rb
```
  scope :viewable, -> { published.where('published_at < ?', Time.current) }
  scope :new_arrivals, -> { viewable.order(published_at: :desc) }
  scope :by_category, ->(category_id) { where(category_id: category_id) }
  scope :by_author, ->(author_id) { where(author_id: author_id) }
  scope :by_tag, ->(tag_id) { joins(:article_tags).merge(ArticleTag.where(tag_id: tag_id))} #{where(tag_id: tag_id) }よりjoins(:articles_tags).merge(ArticleTagを追加
  scope :body_contain, ->(word) { joins(:sentences).merge(where('sentences.body LIKE ?', "%#{word}%")) } #追加でコード記載。joins(sentences).mergeを記載。意味を調べる
  scope :title_contain, ->(word) { where('title LIKE ?', "%#{word}%") }
  scope :past_published, -> { where('published_at <= ?', Time.current) }
```




