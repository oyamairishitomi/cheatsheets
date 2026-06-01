# Rails Twitterクローン チートシート

今日から金曜まで、FBCで学んだことを骨身に染み込ませるための記録。

---

## セットアップの順番（重要）

Userモデルが必要なアプリは **Deviseを最初に入れる**。後から入れると `users` テーブルの作成順序でマイグレーションエラーになる。

```
1. rails new
2. Gemfile に gem 'devise' を追加 → bundle install
3. rails g devise:install
4. rails g devise User → rails db:migrate
5. その後に他のモデルを作る
```

---

## Day 1 — セットアップ・モデル設計

### アプリ作成

```bash
rails new twitter_clone
cd twitter_clone
```

### モデル生成

```bash
rails g model Tweet body:text user:references
rails db:migrate
```

- `id` / `created_at` / `updated_at` は自動生成されるので指定不要
- `user:references` → `user_id` カラム＋インデックス＋外部キー制約をまとめて生成

生成されるマイグレーション：

```ruby
t.references :user, null: false, foreign_key: true
```

| 部分 | 意味 |
|---|---|
| `t.references :user` | `user_id` カラム＋インデックスを自動生成 |
| `null: false` | ユーザーなしで保存不可（NOT NULL制約） |
| `foreign_key: true` | 存在しない `user_id` はDBレベルで弾く |

### モデルのアソシエーション

```ruby
# app/models/user.rb
has_many :tweets, dependent: :destroy
# → ユーザー削除時にそのユーザーのツイートも一緒に削除される

# app/models/tweet.rb（rails g model で自動生成済み）
belongs_to :user
```

`dependent:` を書かないとユーザーだけ消えてツイートがDBに残り続ける（孤児レコード）。

| `dependent:` オプション | 意味 |
|---|---|
| `:destroy` | 子レコードも一緒に削除 |
| `:nullify` | 子レコードのuser_idをnullにする（消さない） |

### ルーティング

```ruby
# config/routes.rb
resources :tweets
root "tweets#index"
```

`resources :tweets` 1行でCRUDの7ルートが生成される：

| Prefix | Verb | URL | Controller#Action |
|---|---|---|---|
| `tweets` | GET | `/tweets` | `tweets#index` |
| | POST | `/tweets` | `tweets#create` |
| `new_tweet` | GET | `/tweets/new` | `tweets#new` |
| `edit_tweet` | GET | `/tweets/:id/edit` | `tweets#edit` |
| `tweet` | GET | `/tweets/:id` | `tweets#show` |
| | PATCH/PUT | `/tweets/:id` | `tweets#update` |
| | DELETE | `/tweets/:id` | `tweets#destroy` |

Prefixに `_path` をつけるとそのURLになる（例: `tweets_path` → `/tweets`）。

### コントローラー生成

```bash
rails g controller Tweets index show new edit create update destroy --skip-routes
```

- `resources` を使うときはルートが既にあるので `--skip-routes` をつける
- `resources` のアクションは常にこの7つ固定：`index show new edit create update destroy`

### カラム名変更マイグレーション

```bash
rails g migration RenameXxxToYyyInTableName
```

生成されたファイルに `rename_column` を書く：

```ruby
def change
  rename_column :tweets, :tweet, :body
  #              テーブル名  変更前  変更後
end
```

書いたら `rails db:migrate` を実行。

現在のカラム名は `db/schema.rb` で確認できる。マイグレーション後の最新状態が書いてある。

---

## コントローラーの実装

### 各アクションの役割

| アクション | やること |
|---|---|
| `index` | `@tweets = Tweet.all` で全件取得 |
| `show` | `@tweet = Tweet.find(params[:id])` で1件取得 |
| `new` | `@tweet = Tweet.new` で空オブジェクトを用意（フォーム表示用） |
| `edit` | `@tweet = Tweet.find(params[:id])` で1件取得（showと同じ） |
| `create` | フォームデータを保存。成功→redirect、失敗→render |
| `update` | フォームデータで更新。成功→redirect、失敗→render |
| `destroy` | 削除してredirect |

### Strong Parameters

フォームから来たデータをそのまま使うと危険なので、許可するカラムを明示する：

```ruby
private

def tweet_params
  params.require(:tweet).permit(:body)
end
```

- `require(:tweet)` → `params` の中の `:tweet` キーを取り出す（なければエラー）
- `permit(:body)` → `:body` だけ許可する（それ以外は弾く）

`form_with model: @tweet` で生成したフォームは `tweet[body]` という名前でデータを送るので、`params = { tweet: { body: "..." } }` という構造になる。

### create アクションの書き方

**Deviseなし（user_idを気にしないとき）**

```ruby
def create
  @tweet = Tweet.new(tweet_params)
  if @tweet.save
    redirect_to tweets_path
  else
    render :new, status: :unprocessable_entity
  end
end
```

`user_id null: false` の制約があると保存が失敗するので、Userと紐づけるアプリでは使えない。

**Deviseあり（ログイン中ユーザーのツイートとして保存）**

```ruby
def create
  @tweet = current_user.tweets.build(tweet_params)
  if @tweet.save
    redirect_to tweets_path
  else
    render :new, status: :unprocessable_entity
  end
end
```

`current_user.tweets.build` にすることで `user_id` が自動セットされる。Deviseを使うアプリでは基本こちらを使う。

### render と redirect_to の違い

| | 何をするか | `@tweet` は |
|---|---|---|
| `render :new` | `new.html.erb` をそのまま描画 | 残る（エラー情報も残る） |
| `redirect_to tweets_path` | ブラウザに「`/tweets` へ行け」と命令（新しいリクエスト） | 消える |

バリデーション失敗時は `render :new` を使う。`redirect_to` にすると `@tweet` のエラー情報が消えてしまう。

`status: :unprocessable_entity` はHTTPステータス422。Rails 7以降ではTurboが正しく動くために必須。

### `_path` vs `_url`

| 書き方 | 返す値 | 使いどころ |
|---|---|---|
| `tweets_path` | `/tweets`（相対URL） | 基本これを使う |
| `tweets_url` | `http://localhost:3000/tweets`（絶対URL） | メール本文など |
| `@tweet`（オブジェクト） | Railsが `/tweets/1` に自動変換 | show・update・destroyで便利 |

---

## 今日の疑問 Q&A

**Q. `new()` のかっこの中にメソッドを入れられる？**
はい。`Tweet.new(tweet_params)` は `tweet_params` の戻り値がそのまま引数になる。

**Q. `params.require` と `params.permit` の違いは？**
- `require(:tweet)` → `params` の中に `:tweet` キーがなければエラー（必須チェック）
- `permit(:body)` → `:body` だけ通す。それ以外は弾く（許可リスト）

**Q. `params` って何もの？未定義なのになぜ使える？**
`TweetsController` → `ApplicationController` → `ActionController::Base` と継承している。`params` は `ActionController::Base` が提供するメソッドなので、コントローラー内ならどこでも使える。

**Q. `params` はURLだけを見るの？**
URLだけじゃない。リクエスト全体からデータを集めたもの：

| 種類 | 例 | `params` に入るもの |
|---|---|---|
| URLのパラメータ | `/tweets/3` | `params[:id] = "3"` |
| クエリ文字列 | `/tweets?page=2` | `params[:page] = "2"` |
| フォームのデータ | POSTで送信 | `params[:tweet][:body] = "..."` |

**Q. `params[:id]` の `:id` はどこから来るの？**
`routes.rb` の `resources :tweets` が `/tweets/:id` というルートを生成する。`/tweets/3` にアクセスすると Railsが自動で `params[:id] = "3"` をセットする。

**Q. なぜフォームのデータが `params[:tweet][:body]` という構造になるの？**
`form_with model: @tweet` を使うと、Railsがフォームフィールドを `tweet[body]` という名前で生成する。それがサーバーに届くと `{ tweet: { body: "..." } }` という構造になる。

**Q. `render :new` には `_path` をつけない？**
`render` はURLではなくビューファイル名を指定する。`:new` = `new.html.erb` のこと。`_path` はURL用（`redirect_to` と一緒に使う）。

**Q. `status: :unprocessable_entity` とは？**
HTTPステータスコード422。「処理できないリクエスト」を意味する。Rails 7以降ではTurboが正しく動くために必要。

**Q. `form_with do |f|` の `do |f|` って何？**
Rubyではメソッドに「処理のかたまり（ブロック）」を渡せる。`|f|` はそのブロックが受け取る変数。

```ruby
# each → コレクションを繰り返す（f の中身が1件ずつ変わる）
@tweets.each do |tweet|
  tweet.body
end

# form_with → 繰り返しではない。フォームオブジェクトを受け取るだけ
form_with model: @tweet do |f|
  f.text_area :body   # f.xxx でフォームの部品を作る
  f.submit
end
```

`each` は「繰り返す」、`form_with` は「フォームオブジェクトを渡す」。書き方は同じだが意味が違う。

**Q. `current_user` はどこから来るの？**
Deviseを入れると自動で使えるようになるヘルパーメソッド。ログイン中のUserオブジェクトを返す。ログアウト中は `nil`。Laravelの `Auth::user()` と同じ。

**Q. `Tweet.new(tweet_params)` と `current_user.tweets.build(tweet_params)` の違いは？**
- `Tweet.new` → `user_id` がセットされない → `null: false` 制約で保存失敗
- `current_user.tweets.build` → `user_id = current_user.id` が自動でセットされる

関連（has_many）を通して `build` すると、親の id が自動で入る。

**Q. `new` と `build` の違いは？**
ほぼ同じだが、`build` は関連を通して呼ぶときに使う慣習。`current_user.tweets.build` のように関連経由で呼ぶと `user_id` が自動セットされる。

**Q. gemを追加したらサーバーを再起動しないといけない？**
はい。`bundle install` しただけではgemは読み込まれない。サーバーを再起動して初めて反映される。
