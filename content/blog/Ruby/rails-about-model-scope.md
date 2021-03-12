---
title: '【Rails】Model（モデル）のscope（スコープ）ってなんだろう？を解決する'
date: 2020-09-15 0:26:32
category: 'Ruby'
draft: false
---


## モデルにおけるscopeってなんですか？

Railsを学習していて次のようなコードを発見しました。「モデルのscopeってなんだろう？」ということで調べて記事にしました。

```ruby
scope :name, ->(name) { where(name: name) }
```

<br>
<br>




ひとまずRailsガイドで調べてみました。[Rails ガイド Active Record - 14.スコープ](https://railsguides.jp/active_record_querying.html#%E3%82%B9%E3%82%B3%E3%83%BC%E3%83%97)



> スコープを設定することで、関連オブジェクトやモデルへのメソッド呼び出しとして参照される、よく使用されるクエリを指定することができます。スコープでは`where`、`joins`、`includes`など、これまでに登場したすべてのメソッドを使用できます。どのスコープメソッドも、常に`ActiveRecord::Relation`オブジェクトを返します。このオブジェクトに対して、別のスコープを含む他のメソッド呼び出しを行うこともできます。

<br>

Railsガイドを読むとなんとなく分かると思いますが、次のようなArticleモデル内に「公開済み」となっている記事をwhereを用いて取得するクラスメソッドがあるとします。

```ruby
class Article < ApplicationRecord

  def self.published
    where(published: true)
  end
end
```

<br>

上のようなクラスメソッドを`scope`メソッドを用いることで次のように書き換えることができます。書き換えているだけでクラスメソッドの定義と完全に同じなので`scope`を利用するか、クラスメソッドで定義するかは好みの問題のようです。

```ruby
class Article < ApplicationRecord
  scope :published, -> { where(published: true) }
end
```

<br>

また、次のようにすることでスコープをスコープ内でチェインさせることも出来るみたいです。

```ruby
class Article < ApplicationRecord
  scope :published, -> { where(published: true) }
  scope :published, -> { published.where("comments_count > 0") }
end
```



上で定義したスコープを呼び出すためには、クラスにて次のようにすることで呼び出しができるようです。

```ruby
Article.published
```



また、関連付けされているクラスからスコープを呼び出すことも出来ます。

```ruby
category = Category.first
category.articles.publiished
```



ここまでは、Railsガイドをそのまま読んだだけなので、実際にRailsでプロジェクトを作成して`scope`を使ってみます。


<br>
<br>
<br>


## 実際に`scope`を使ってみる

まずは次のようなテーブルとデータを作成しました。

<br>
<br>


#### Users

| id  | name     |
|:--- |:-------- |
| 1   | 山田太郎 |



<br>
<br>

#### Posts

| id  | title                    | body                               | published | user_id |
|:--- |:------------------------ | ---------------------------------- | --------- | ------- |
| 1   | 山田太郎の記事1          | これは山田太郎の初めての記事です。 | true      | 1       |
| 2   | 山田太郎の記事2          | これは山田太郎の2回目の記事です。  | true      | 1       |
| 3   | 山田太郎の非公開記事 | これは山田太郎の非公開記事です。   | false     | 1       |



<br>
<br>
<br>

公開済みとなっているブログ記事のみを取得して一覧表示する想定なので、Postモデルで`scope`を設定します。



app/models/post.rb

```ruby
class Post < ApplicationRecord
  belongs_to :user

  # 公開済みとなっている記事を取得
  scope :published, -> { where(published: true) }
end

```


<br>
<br>


それではコントローラ側で取得したブログ記事を返します。今回はRailsをAPIモードで作成しているため、json形式でブログ記事を返します。

app/controllers/posts_controller.rb

```ruby
class PostsController < ApplicationController
  def index
    # モデル側で定義したpublishedを呼び出す
    posts = Post.published
    render json: { data: posts }
  end
end

```

<br>
<br>


postmanなどで正常に公開済みの記事がレスポンスとして返ってきているか来ているか確認してみます。



```json
{
    "data": [
        {
            "id": 1,
            "title": "山田太郎の記事1",
            "body": "これは山田太郎の初めての記事です。",
            "user_id": 1,
            "created_at": "2020-09-14T13:23:52.206Z",
            "updated_at": "2020-09-14T13:23:52.206Z",
            "published": true
        },
        {
            "id": 2,
            "title": "山田太郎の記事2",
            "body": "これは山田太郎の二回目の記事です。",
            "user_id": 1,
            "created_at": "2020-09-14T13:23:52.216Z",
            "updated_at": "2020-09-14T13:23:52.216Z",
            "published": true
        }
    ]
}
```






無事に`published: true`（公開済み）のブログ記事が返ってきました。



<br>




<br>

### 降順への並び替えを`scope`で同時にやってみる　



次はレスポンスとして返ってきているブログ記事一覧を`created_at`で降順にしてみます。

app/models/post.rb

```ruby
class Post < ApplicationRecord
  belongs_to :user

  scope :published, -> { where(published: true).order(created_at: :DESC) }
end
```

<br>


`order`を追記し`created_at: :DESC`で降順にして取得するようにして結果を確認してみます。



```json
{
    "data": [
        {
            "id": 2,
            "title": "山田太郎の記事2",
            "body": "これは山田太郎の二回目の記事です。",
            "user_id": 1,
            "created_at": "2020-09-14T13:23:52.216Z",
            "updated_at": "2020-09-14T13:23:52.216Z",
            "published": true
        },
        {
            "id": 1,
            "title": "山田太郎の記事1",
            "body": "これは山田太郎の初めての記事です。",
            "user_id": 1,
            "created_at": "2020-09-14T13:23:52.206Z",
            "updated_at": "2020-09-14T13:23:52.206Z",
            "published": true
        }
    ]
}
```

<br>
<br>

seedでデータを作成しているため、作成日時がほとんど同じなので分かりづらいと思うので、新たにデータを追加してみます。



```bash
$ rails c
> user = User.first # => 山田太郎
> Post.create(title: "降順チェック", body: "降順で取得できるかチェックします。", published: true, user_id: user.id)
```

<br>

新たに公開済みのブログ記事として作成できたので、確認してみます。



```json
{
    "data": [
        {
            "id": 3,
            "title": "降順チェック",
            "body": "降順で取得できるかチェックします。",
            "user_id": 1,
            "created_at": "2020-09-14T14:23:27.174Z",
            "updated_at": "2020-09-14T14:23:27.174Z",
            "published": true
        },
        {
            "id": 2,
            "title": "山田太郎の記事2",
            "body": "これは山田太郎の二回目の記事です。",
            "user_id": 1,
            "created_at": "2020-09-14T13:23:52.216Z",
            "updated_at": "2020-09-14T13:23:52.216Z",
            "published": true
        },
        {
            "id": 1,
            "title": "山田太郎の記事1",
            "body": "これは山田太郎の初めての記事です。",
            "user_id": 1,
            "created_at": "2020-09-14T13:23:52.206Z",
            "updated_at": "2020-09-14T13:23:52.206Z",
            "published": true
        }
    ]
}
```



降順で表示されていることが分かります。


<br>
<br>



## クラスメソッドと`scope`で違いはないのか確認する

### スコープ

```ruby
scope :published, -> { where(published: true).order(created_at: :DESC) }

# スコープで取得したときのSQL文
SELECT `posts`.* FROM `posts` WHERE `posts`.`published` = TRUE ORDER BY `posts`.`created_at` DESC
```

<br>


### クラスメソッド

```ruby
def self.published
  where(published: true).order(created_at: :DESC)
end

# クラスメソッドで取得したときのSQL文
SELECT `posts`.* FROM `posts` WHERE `posts`.`published` = TRUE ORDER BY `posts`.`created_at` DESC
```



同じSQL文が実行されているようです。

<br>
<br>



## スコープで引数を渡す

見つけたコードが次のような引数を渡して取得するコードだったので、引数を渡す方法も確認してみたいと思います。

```ruby
scope :name, ->(name) { where(name: name) }
```


<br>
<br>



### スコープの引数で受け取った文字列で部分一致検索をする

次のようなRequestを受け取ったとき、パラメータを引数で受け取り検索するスコープを定義します。

```json
{
  "title": "山田"
}
```


<br>
<br>


`scope`で`search`メソッドを定義して引数を受け取り、Postsテーブルの`title`カラムと部分一致するデータを取得します。

app/models/post.rb

```ruby
class Post < ApplicationRecord
  belongs_to :user

  scope :search, ->(title) { where("title LIKE?", "%#{title}%") }
end

```





ストロングパラメータを設定して、モデル側で作成した`search`メソッドに渡しています。

app/controllers/posts_controller.rb

```ruby
class PostsController < ApplicationController
  def index
    # 引数として受けとったパラメータを渡す
    posts = Post.search(post_params[:title])

    render json: { data: posts }
  end

  private

  # ストロングパラメータを追記
  def post_params
    params.require(:post).permit(:title)
  end
end

```


<br>
<br>

jsonで送信した文字列で部分一致検索出来ているか確認します。



- 結果

```json
{
    "data": [
        {
            "id": 1,
            "title": "山田太郎の記事1",
            "body": "これは山田太郎の初めての記事です。",
            "user_id": 1,
            "created_at": "2020-09-14T13:23:52.206Z",
            "updated_at": "2020-09-14T13:23:52.206Z",
            "published": true
        },
        {
            "id": 2,
            "title": "山田太郎の記事2",
            "body": "これは山田太郎の二回目の記事です。",
            "user_id": 1,
            "created_at": "2020-09-14T13:23:52.216Z",
            "updated_at": "2020-09-14T13:23:52.216Z",
            "published": true
        },
        {
            "id": 3,
            "title": "山田太郎の非公開記事",
            "body": "これは山田太郎の非公開記事です。",
            "user_id": 1,
            "created_at": "2020-09-14T13:23:52.227Z",
            "updated_at": "2020-09-14T13:23:52.227Z",
            "published": false
        }
    ]
}
```





無事`title`に「山田」と含まれているブログ記事のみ取得することができました。


<br>
<br>




## まとめ

- モデルにおける`scope`を用いることでクラスメソッドを書き換えることができる
- スコープで引数を渡すことができる
- `scope`で定義したスコープメソッドとクラスメソッドが実行するSQL文が同じ
- スコープメソッドは常に`ActiveRecord::Relation`を返す