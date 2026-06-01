# Rails アプリ作成の流れ（掲示板アプリ）

## 1. アプリを新規作成する

```bash
rails new bbs --database=sqlite3
cd bbs
bin/rails s  # localhost:3000 で起動確認
```

## 2. devise を導入する

`Gemfile` に追加：

```ruby
gem 'devise'
gem 'devise-i18n'
```

```bash
bundle install

bin/rails generate devise:install   # deviseの設定ファイルを生成（1回だけ）
bin/rails generate devise User      # Userモデルをdevise仕様で生成
bin/rails db:migrate                # DBに反映
```

- `devise:install` → deviseの動作設定ファイルを作る
- `devise User` → メール・パスワード・ログイン機能つきのUserモデルを作る

## 3. Postをscaffoldで作る

```bash
bin/rails generate scaffold Post title:string body:text
bin/rails db:migrate
```

`config/routes.rb` にrootを追加：

```ruby
Rails.application.routes.draw do
  devise_for :users
  root 'posts#index'
end
```

## 5. ログイン必須にする

`app/controllers/posts_controller.rb` のクラス直下に追加：

```ruby
before_action :authenticate_user!
```

- `authenticate_user!` → ログインしていなければ強制的にログインページへリダイレクト
- `authenticate_user?` → ログインしているか確認するだけ（真偽値を返す、リダイレクトしない）
- `!` はRubyの慣習で「強制・破壊的な処理」につける

## 6. 投稿を current_user に紐付ける

`posts_controller.rb` の `create` アクションを修正：

```ruby
# NG: user_id が空のまま作られる。フォームから user_id を改ざんされる危険もある
@post = Post.new(post_params)

# OK: ログイン中のユーザーの投稿として作る（user_id が自動セットされる）
@post = current_user.posts.build(post_params)
```

あわせて `post_params` から `:user_id` を削除する（許可しない）。

## 4. アソシエーションを設定する（User と Post の紐付け）

「この投稿は誰のもの？」を管理するために、PostテーブルにUserへの参照（`user_id`）を追加する。

```bash
bin/rails generate migration AddUserIdToPosts user:references
bin/rails db:migrate
```

モデルに関連を宣言する：

```ruby
# app/models/user.rb
has_many :posts, dependent: :destroy

# app/models/post.rb（自動で追加されている）
belongs_to :user
```

- `AddUserIdToPosts` → マイグレーション名。`Add〇〇To△△` の形でRailsが意図を解釈する
- `user:references` で `user_id` カラム＋外部キー制約が一度に作られる（`user:integer` より安全）
- アソシエーション＝テーブル同士の「関連付け」。書くことで `user.posts`（その人の投稿一覧）や `post.user`（投稿者）が使えるようになる
- `has_many` を書かないと `user.posts` が使えない
- `dependent: :destroy` を書かないとユーザー削除時に投稿が孤児になる

## 7. 投稿者のみ編集・削除できる認可

`posts_controller.rb` に追加：

```ruby
before_action :set_post, only: %i[ show edit update destroy ]
before_action :authenticate_user!
before_action :check_owner, only: %i[ edit update destroy ]  # ← 追加

private

def check_owner
  redirect_to @post unless @post.user == current_user
end
```

**ポイント：**
- `set_post` → `check_owner` の順に実行される。`@post` を取得してから誰のものか確認するため順番が重要
- `show` は誰でも見られるので `check_owner` に含めない
- `unless` は「〜でなければ」。投稿者でなければリダイレクト

**コントローラーの `private` について：**
- `private` 以下のメソッドはブラウザから直接 `/posts/check_owner` のようにアクセスできない
- アクション（`index`, `show` など）は `public` である必要があるが、内部処理用のメソッドは `private` に置く

## 8. Active Storage でアバター画像アップロード

### セットアップ

`Gemfile` に追加：

```ruby
gem 'active_storage_validations'  # content_type バリデーションに必要
```

```bash
bundle install
bin/rails active_storage:install  # マイグレーションファイルを生成
bin/rails db:migrate
```

### モデルに宣言する

```ruby
# app/models/user.rb
has_one_attached :avatar
validates :avatar, content_type: %i[jpg png gif], if: -> { avatar.attached? }
```

**ポイント：**
- `if: -> { avatar.attached? }` のラムダ `-> { }` が重要
- `if avatar.attached?` を直接書くとクラス読み込み時に1回だけ評価されて常にfalseになる
- ラムダにすることでバリデーション実行のたびに毎回評価される
- ラムダ＝「今は実行しないで、呼ばれたときに実行する処理のかたまり」

### 画像を表示する（リサイズあり）

```erb
<% if post.user.avatar.attached? %>
  <%= image_tag post.user.avatar.variant(resize_to_limit: [50, 50]) %>
<% end %>
```

- `each` ループの**中**に書く（外に書くと `post` が未定義でエラー）
- `attached?` で添付済みか確認してから表示する

### N+1 対策

一覧画面でアバターを表示する場合、コントローラの `index` アクションを修正：

```ruby
def index
  @posts = Post.all.includes(user: { avatar_attachment: :blob })
end
```

- `with_attached_avatar` は `has_one_attached :avatar` を書いたモデル（User）に自動で生えるメソッド。Post には存在しないので `Post.all.with_attached_avatar` はエラー
- Post 経由で User のアバターを取得する場合は `includes(user: { avatar_attachment: :blob })` を使う
- `includes(user: { avatar_attachment: :blob })` の意味：Post → user → avatar_attachment（添付レコード）→ blob（ファイルのメタ情報）の3段階をまとめて取得
- これをしないと投稿数分のSQLが発行される（N+1問題）

### devise の Strong Parameters

`app/controllers/application_controller.rb` に追加：

```ruby
class ApplicationController < ActionController::Base
  allow_browser versions: :modern      # モダンブラウザ以外を弾く（自動生成）
  stale_when_importmap_changes         # importmap変更時にキャッシュ無効化（自動生成）

  before_action :configure_permitted_parameters, if: :devise_controller?

  private

  def configure_permitted_parameters
    devise_parameter_sanitizer.permit(:sign_up, keys: [:avatar])
    devise_parameter_sanitizer.permit(:account_update, keys: [:avatar])
  end
end
```

**ポイント：**
- `devise_controller?`（`s` なし）。`devise_controllers?` はタイポ
- `private` はそれ以降に書いたものを全部 private にする。`allow_browser` などの設定メソッドは `private` より上に書く
- `avatar` などカスタムフィールドをここに追加しないとdeviseに弾かれる

## devise ビューの日本語化

`rails generate devise:views` で生成されたビューは英語テキストがハードコードされている。
`devise-i18n` のロケールファイルを生成しただけでは変わらず、ビュー側も `t()` に書き換えが必要。

```erb
<%# NG: 英語ハードコード %>
<h2>Edit User</h2>
<i>(leave blank if you don't want to change it)</i>
<%= f.submit "Update" %>

<%# OK: 翻訳キーを参照 %>
<h2><%= t('.title', resource: t('activerecord.models.user.other')) %></h2>
<i><%= t('.leave_blank_if_you_don_t_want_to_change_it') %></i>
<%= f.submit t('.update') %>
```

`devise/shared/_links.html.erb` も同様に英語ハードコードになっているので書き換える：

```erb
<%= link_to t('devise.shared.links.sign_up'), new_registration_path(resource_name) %>
<%= link_to t('devise.shared.links.forgot_your_password'), new_password_path(resource_name) %>
```

カスタムフィールドのラベルは `config/locales/devise.views.ja.yml` の
`activerecord.attributes.user` に追加する：

```yaml
postal_code: 郵便番号
address: 住所
introduction: 自己紹介
```

また `config/application.rb` に以下がないと何も反映されない：

```ruby
config.i18n.default_locale = :ja
```

## bundle install 後はサーバーを再起動する

`bundle install` でgemを追加した後、サーバーを起動したままだと新しいgemが読み込まれない。
`Ctrl+C` で止めてから `bin/rails s` で再起動する。
