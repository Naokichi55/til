パンくず設定について
⭕️ドリンク、お酒>ワイン＞赤ワイン
❌

⭐️関連性のないもの、わかりにくくなるので注意



```
crumb :root do
  link "HOme", root_path
end
```

## 参考URL
***
### 公式ドキュメント

### 技術記事
rakeタスク
https://qiita.com/mmaumtjgj/items/8384b6a26c97965bf047

wheneverでcronを設定
https://qiita.com/mmaumtjgj/items/19e866f31541abb6c614

### 学習めも、

##### 下記一時おき

@article = Article.new(article_params)
    #下書きと公開機能を追加するために下記修正
    if params[:draft].present?
      @article.status = @article.state = :draft
    elsif 
      @port.status = published
    else 
      @port.status = publish_waite
    end