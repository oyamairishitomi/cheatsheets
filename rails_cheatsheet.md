# Rails チートシート

## Railsの哲学

| 原則 | 意味 |
|------|------|
| DRY（Don't Repeat Yourself） | 同じコードを繰り返さず1箇所にまとめる |
| Convention over Configuration | Railsがルール（命名・フォルダ構成など）を決めてくれるので設定を自分で書かなくていい |

## アプリ作成

```bash
rails new アプリ名   # 新しいRailsアプリを作成
```

## 主なフォルダ構成

| フォルダ | 役割 |
|------|------|
| `app/` | 一番よく触う場所。MVC（models/views/controllers）が入っている |
| `config/` | ルーティング・DB設定など |
| `db/` | マイグレーションファイル・schema.rb |
| `bin/` | `rails`などの実行ファイル |

## MVCのリクエスト流れ

```
ブラウザ
  ↓ アクセス
routes.rb → どのControllerを呼ぶか判断
  ↓
Controller → Modelにデータを問い合わせる
  ↓
Model → DBからデータを取得して返す
  ↓
Controller → @変数にセットしてViewに渡す
  ↓
View → HTMLを生成してブラウザに返す
```

## routes.rb の書き方

```ruby
resources :posts          # CRUD用の7つのルートを自動生成
root "posts#index"        # / にアクセスしたときに呼ぶController#Action
```

```bash
rails routes              # 定義されているルート一覧を確認
```

ルート一覧の見方：

| 列 | 意味 |
|------|------|
| Prefix | path helperの名前（`new_post` → `new_post_path`） |
| Verb | HTTPメソッド（GET/POST/PATCH/DELETE） |
| URI Pattern | URLのパターン |
| Controller#Action | 呼ばれるコントローラとメソッド |

### path helper（Prefixの使い方）

```erb
<%= link_to "新規作成", new_post_path %>
<%# → /posts/new に変換される %>
```

URLをベタ書きしないことでroutes.rbを変えるだけで全体に反映される（DRY）。

## パーシャル（部品テンプレート）

ファイル名が `_` で始まるViewファイル。繰り返し使う部品を切り出すためのもの。

```erb
<%# 呼び出し側（index.html.erb など）%>
<%= render post %>          # _post.html.erb を描画する
```

- `_post.html.erb` → `render post` で呼ばれる（命名規則で自動的に対応）
- 単体でブラウザに表示されることはない

## 新しいリポジトリをクローンしたとき

```bash
bundle install       # gem をインストール（これをしないと rails コマンドが使えない）
rails db:reset       # DB を作り直してシードデータも投入
rails s              # サーバー起動
```

## Ruby バージョン関連

```bash
rbenv install 3.2.2  # .ruby-version に指定されたバージョンをインストール
rbenv versions       # インストール済みバージョン一覧
```

## DB 操作

```bash
rails db:reset       # drop → create → migrate → seed をまとめて実行
rails db:migrate     # マイグレーションだけ実行
rails db:seed        # シードデータだけ投入
rails db:rollback    # 直前のマイグレーションを取り消す
```

## コード品質チェック

```bash
bundle exec rubocop              # Ruby のスタイルチェック
bin/erblint --lint-all           # ERB のスタイルチェック（bundle exec erb_lint は動かない）
bundle exec rubocop -A           # 自動修正（全部 Correctable なら一括で直る）
```

erblint で「Missing a trailing newline at the end of the file.」が出たら、該当ファイルの末尾に改行を追加する：

```bash
# 複数ファイルまとめて末尾改行を追加する
for f in ファイル1 ファイル2 ...; do echo "" >> $f; done
```

## ブランチ操作（プラクティスの手順）

```bash
git checkout -b 03-pagination origin/03-pagination  # スタート地点を作る
git checkout -b my-pagination                        # 提出用ブランチを作る
git push origin my-pagination                        # push
```

## i18nのビューでよく使うパターン

### バリデーションエラーのメッセージ

```erb
<h2><%= t('views.common.validation_error', errors: i18n_error_count(report.errors.count), name: Report.model_name.human.downcase) %></h2>
```

分解するとこう：

| 部分 | 意味 |
|---|---|
| `t('views.common.validation_error', ...)` | `config/locales/ja.yml` の翻訳文字列を呼び出す |
| `report.errors.count` | バリデーションエラーの件数（数字） |
| `i18n_error_count(...)` | 件数を「1件」「2件」と日本語にするヘルパー |
| `Report.model_name.human` | i18nで定義したモデルの日本語名（例：「日報」） |
| `.downcase` | 小文字化（英語用。日本語には影響なし） |

**`.human` とは？**

「人間が読める形式」という意味。機械向けの `:report` や `"Report"` ではなく、
i18nで定義した「日報」という人間向けの文字列を返す。「人類」という意味ではない。

`config/locales/ja.yml` の中身はこんな感じ：
```yaml
validation_error: "%{errors}があるため、この%{name}は保存できませんでした"
```
`%{errors}` と `%{name}` に引数が埋め込まれて「1件のエラーがあるため、この日報は保存できませんでした」になる。

## i18n（日本語化）の設定

`config/application.rb` に追加：

```ruby
config.i18n.default_locale = :ja
```

devise の日本語訳が反映されない場合もこれが抜けていることが多い。
ロケールファイル（`config/locales/*.ja.yml`）が存在していてもこの設定がないと英語のまま。

devise のビューで翻訳キーを使う例：

```erb
<h2><%= t('.title', resource: t('activerecord.models.user.other')) %></h2>
<i><%= t('.leave_blank_if_you_don_t_want_to_change_it') %></i>
<%= f.submit t('.update') %>
```

カスタムフィールド（例: postal_code）のラベルを日本語にするには
`config/locales/devise.views.ja.yml` の `activerecord.attributes.user` に追加：

```yaml
postal_code: 郵便番号
address: 住所
introduction: 自己紹介
```

## devise の Strong Parameters

`ApplicationController` に追加する：

```ruby
before_action :configure_permitted_parameters, if: :devise_controller?

private

def configure_permitted_parameters
  keys = %i[name postal_code address self_introduction avatar]
  devise_parameter_sanitizer.permit(:sign_up, keys:)
  devise_parameter_sanitizer.permit(:account_update, keys:)
end
```

- `%i[...]` はシンボルの配列リテラル（`[:name, :postal_code, ...]` と同じ）
- `:sign_up` → 新規登録時、`:account_update` → プロフィール編集時に許可するパラメータ
- ファイルアップロード（`avatar` など）もここに追加しないと弾かれる

## devise の日本語化

```bash
# Gemfile に追加
gem 'devise-i18n'

# bundle install 後にロケールファイルを生成
bundle install
rails g devise:i18n:locale ja
```

生成された `config/locales/devise.views.ja.yml` で日本語訳が反映される。
デフォルトの訳文でよければ追加作業不要。

## devise のログアウトリンク

```erb
<%= button_to 'ログアウト', destroy_user_session_path, method: :delete %>
```

`link_to` ではなく `button_to` を使う（DELETE メソッドが必要なため）。

ログイン済みのときだけ表示したい場合は `user_signed_in?` で囲む：

```erb
<% if user_signed_in? %>
  <%= button_to 'プロフィール編集', edit_user_registration_path, method: :get %>
  <%= button_to 'ログアウト', destroy_user_session_path, method: :delete %>
<% end %>
```

## よく使うコマンド

```bash
rails s              # サーバー起動（localhost:3000）
rails c              # コンソール起動
rails routes         # ルーティング一覧
rails g migration    # マイグレーションファイル生成
```

## モデル生成（rails g model）

```bash
rails g model Tweet body:text user:references
```

### カラム型の指定

| 書き方 | 生成されるもの |
|---|---|
| `body:text` | text 型のカラム |
| `user:references` | `user_id` カラム＋インデックス＋外部キー制約 |

`id` と `created_at` / `updated_at` は自動生成されるため指定不要。

### `user:references` が生成するマイグレーション

```ruby
t.references :user, null: false, foreign_key: true
```

| 部分 | 意味 |
|---|---|
| `t.references :user` | `user_id` カラム＋インデックスを自動生成 |
| `null: false` | ユーザーなしで保存不可（NOT NULL制約） |
| `foreign_key: true` | 存在しない `user_id` はDBレベルで弾く |

`user_id:integer` と書くより `user:references` の方が制約とインデックスが揃うので推奨。

---

## モデルの関連付け（アソシエーション）

### has_many / belongs_to

「1対多」の関係を定義する。

```ruby
# app/models/user.rb
has_many :reports, dependent: :destroy
# → 1人のユーザーは複数の日報を持てる
# → ユーザー削除時に日報も一緒に削除される

# app/models/report.rb（scaffold が自動生成）
belongs_to :user
# → この日報はどのユーザーのものか
```

`has_many` を書くと `user.reports` という書き方ができるようになる。書かないとエラー。

### dependent: :destroy

親レコードを削除したとき、紐づく子レコードも一緒に削除する設定。
書かないと子レコードだけDBに残り続ける（孤児レコード）。

### コントローラで current_user を使う

`user_id` をフォームから受け取ると誰でも任意の `user_id` を送れてしまう（危険）。
代わりに `current_user` からレコードを作る：

```ruby
# NG: フォームの user_id をそのまま使う
@report = Report.new(report_params)

# OK: ログイン中のユーザーの日報として作る（user_id が自動セットされる）
@report = current_user.reports.build(report_params)
```

あわせて `report_params` から `:user_id` を削除する。

### 投稿者のみ編集・削除できるようにする

`before_action` を使って「アクションの前に権限チェックを実行する」パターン。

```ruby
before_action :set_report, only: %i[show edit update destroy]
before_action :check_owner, only: %i[edit update destroy]

private

def check_owner
  redirect_to @report unless @report.user == current_user
end
```

**ポイント：**
- `set_report` → `check_owner` の順に実行される。`set_report` で `@report` を取得してから `check_owner` で誰のものか確認するため、順番が重要
- `check_owner` は `private` に置く（ブラウザから直接アクセスされるべきでないため）
- `show` は誰でも見られるので `check_owner` に含めない
- `unless` は「〜でなければ」。`@report.user == current_user` でなければリダイレクト

### scaffoldのJSONレスポンスを削除する

scaffoldは `respond_to` でHTML/JSON両方に対応したコードを生成するが、JSONを使わない場合は削除してシンプルにする。

```ruby
# scaffold が生成するコード（削除前）
def create
  @report = current_user.reports.build(report_params)
  respond_to do |format|
    if @report.save
      format.html { redirect_to @report, notice: '...' }
      format.json { render :show, status: :created, location: @report }
    else
      format.html { render :new, status: :unprocessable_entity }
      format.json { render json: @report.errors, status: :unprocessable_entity }
    end
  end
end

# シンプルにした後
def create
  @report = current_user.reports.build(report_params)
  if @report.save
    redirect_to @report, notice: '日報を作成しました。'
  else
    render :new, status: :unprocessable_entity
  end
end

def update
  if @report.update(report_params)
    redirect_to @report, notice: '日報を更新しました。'
  else
    render :edit, status: :unprocessable_entity
  end
end

def destroy
  @report.destroy!
  redirect_to reports_path, status: :see_other, notice: '日報を削除しました。'
end
```

### render と redirect_to の違い

| | 何をするか | `@report` は |
|---|---|---|
| `render :new` | そのままnewのビューを描画 | 残る（エラー情報も残る） |
| `redirect_to reports_new_path` | ブラウザに「newへ行け」と命令 | 消える |

バリデーションエラー時に `redirect_to` を使うと `@report` が消えてエラーメッセージを表示できない。だから `render` を使う。

**render の書き方**
```ruby
render :new                                     # new.html.erb を表示
render :edit                                    # edit.html.erb を表示
render :new, status: :unprocessable_entity      # ステータスコードも指定（Rails 7以降は必須）
```

**redirect_to の書き方**
```ruby
redirect_to @report                             # 詳細ページ（show）へ
redirect_to reports_path                        # 一覧ページへ
redirect_to reports_path, status: :see_other, notice: '削除しました。'
```

- `notice:` → 画面上部に一瞬表示されるフラッシュメッセージ
- `status: :see_other` → 削除後のリダイレクトで必要なHTTPステータスコード（303）
- `status: :unprocessable_entity` → バリデーションエラー時のステータスコード（422）

### content_for と yield

ページごとにタイトルなどをレイアウトに差し込む仕組み。

**ビュー側（各ページ）：**
```erb
<% title = t('views.common.title_index', name: '日報一覧') %>
<% content_for :title, title %>
```
「`:title` という名前でこの内容を用意しておく」という宣言。

**レイアウト側（`app/views/layouts/application.html.erb`）：**
```erb
<title><%= content_for(:title) || "Books App" %></title>
```
「`:title` があればそれを、なければ "Books App" をタブに表示する」という受け取り側。

→ 各ページで `content_for :title` を書くとブラウザのタブのタイトルが変わる。書かないとデフォルトの "Books App" が表示される。

### form_with の書き方

```
form_with( model: [commentable, Comment.new] )
│          │       └─ 配列でネストしたURLを生成
│          └─ URLとHTTPメソッドを自動決定するオプション
└─ Railsのフォームヘルパーメソッド
```

`model:` にオブジェクトを1つ渡すと通常のURL、配列で渡すとネストしたURLを生成する。
内部では `polymorphic_path` が使われている。

```erb
<%# 通常（単一モデル） %>
<%= form_with model: @report do |form| %>
  <%# → POST /reports %>

<%# ネスト（ポリモーフィック） %>
<%= form_with model: [commentable, Comment.new] do |form| %>
  <%# commentable = Book(id:1) → POST /books/1/comments %>
  <%# commentable = Report(id:3) → POST /reports/3/comments %>
```

### link_to の書き方

```erb
link_to テキスト, リンク先
```

第1引数がテキスト、第2引数がリンク先。順番は固定。逆に書くとURLが画面に表示される。

```erb
<%= link_to '日報一覧', reports_path %>
<%= link_to '詳細', report %>   # Railsが /reports/1 に変換してくれる

<%# i18nを使う場合 %>
<%= link_to t('views.common.show', name: Report.model_name.human.downcase), report %>
```

- `t('views.common.show', name: "日報")` → `ja.yml` の翻訳文字列に `name` を埋め込む
- `report`（モデルオブジェクト）をリンク先に渡すと Rails が自動でURLに変換する

## 多対多の関連付け（has_many :through）

### 中間テーブルのマイグレーション

```bash
rails generate migration CreateReportMentions mentioning_report_id:integer mentioned_report_id:integer
```

マイグレーションファイルに以下を追加する：

```ruby
# 外部キー制約（参照先が削除されたら一緒に消す）
add_foreign_key :report_mentions, :reports, column: :mentioning_report_id, on_delete: :cascade
add_foreign_key :report_mentions, :reports, column: :mentioned_report_id, on_delete: :cascade

# ユニーク制約（同じ組み合わせの重複防止）
add_index :report_mentions, [:mentioning_report_id, :mentioned_report_id], unique: true

# 検索用インデックス
add_index :report_mentions, :mentioned_report_id
```

### 中間テーブルのモデル

カラム名からモデルが自動推測できないときは `class_name:` で明示する：

```ruby
class ReportMention < ApplicationRecord
  belongs_to :mentioning_report, class_name: 'Report'
  belongs_to :mentioned_report, class_name: 'Report'
end
```

### has_many :through の書き方

`report.mentioned_reports`（この日報に言及している日報たち）と
`report.mentioning_reports`（この日報が言及している日報たち）を定義する例：

```ruby
class Report < ApplicationRecord
  # 言及元（この日報に言及している日報たち）
  has_many :mentioned_report_mentions, class_name: 'ReportMention', foreign_key: :mentioned_report_id, dependent: :destroy, inverse_of: :mentioned_report
  has_many :mentioned_reports, through: :mentioned_report_mentions, source: :mentioning_report

  # 言及先（この日報が言及している日報たち）
  has_many :mentioning_report_mentions, class_name: 'ReportMention', foreign_key: :mentioning_report_id, dependent: :destroy, inverse_of: :mentioning_report
  has_many :mentioning_reports, through: :mentioning_report_mentions, source: :mentioned_report
end
```

各オプションの意味：

| オプション | 意味 |
|---|---|
| `class_name:` | 関連先のモデル名（カラム名から推測できないとき） |
| `foreign_key:` | どのカラムで絞り込むか |
| `dependent: :destroy` | この日報が消えたら関連レコードも消す |
| `inverse_of:` | 反対側の `belongs_to` と対応関係を明示（パフォーマンス改善） |
| `through:` | 経由する関連名 |
| `source:` | 経由先でどの関連を使うか |

**`mentioned` と `mentioning` の対称性：**

| `mentioned_reports`（言及元） | `mentioning_reports`（言及先） |
|---|---|
| `foreign_key: :mentioned_report_id` | `foreign_key: :mentioning_report_id` |
| `inverse_of: :mentioned_report` | `inverse_of: :mentioning_report` |
| `source: :mentioning_report` | `source: :mentioned_report` |

`source:` は「取得したいもの」なので `through` の関連とは **逆側** になる点に注意。

### モデルのバリデーション

DBのユニーク制約に加えて、モデルにもバリデーションを書く：

```ruby
validates :mentioning_report_id, uniqueness: { scope: :mentioned_report_id }
```

---

## トランザクション

「全部成功するか、全部なかったことにする」仕組み。
複数のDB保存をセットで行うときに使う。

```ruby
ActiveRecord::Base.transaction do
  @report.save!
  ReportMention.create!(...)
end
redirect_to @report, notice: '...'
rescue ActiveRecord::RecordInvalid
  render :new, status: :unprocessable_entity
```

| メソッド | 失敗したとき |
|---|---|
| `save` | `false` を返す |
| `save!` | 例外（`RecordInvalid`）を投げる → トランザクションでロールバック |

`rescue` の位置は `def` メソッドの中（`begin` 不要）。
`rescue` の後は例外クラス名、`render` の後はビュー名：

```ruby
rescue ActiveRecord::RecordInvalid   # ← 例外クラス
  render :new, status: :unprocessable_entity  # ← ビュー名
```

---

## マイグレーションのタイミング

`rails g` で `db/migrate/` に新しいファイルが生成されたら **すぐ** `rails db:migrate` する。
ファイルが増えた = migrateが必要なタイミング。

## ERB コメントの注意点

```erb
<%# これはコメント %>
```

**罠：コメントの中に ERB タグを入れてはいけない**

```erb
<%# <h3><%= t('.title') %></h3> %>  ← NG！
```

`%>` が最初に現れた時点でコメントが閉じてしまい、残りの ` %></h3>` が画面に表示される。
コメントアウトしたい場合は行ごと削除するか、HTML コメントを使う：

```erb
<!-- <h3>...</h3> -->
```

## ERB タグの使い分け

```erb
<% if condition %>   # 制御構文（出力しない）
  <%= value %>       # 値を出力する
<% end %>
```

- `<%=` は値をHTMLに出力する。`if` や `end` に使うと余計な空白が出る
- `<%` は出力しない。制御構文（if / end / each など）はこちらを使う

## bin/rails vs rails

`bin/rails` はそのプロジェクト専用の Rails を使うことを保証する（推奨）。  
`rails` だとグローバルにインストールされた gem が使われ、バージョンがズレる可能性がある。

```bash
bin/rails db:migrate   # ◎ 推奨
rails db:migrate       # △ 環境によっては動かないこともある
```

## Active Storage（画像アップロード）

Rails 標準のファイルアップロード機能。外部 gem 不要。

### セットアップ

```bash
# Gemfile の image_processing gem のコメントを外して bundle install
bundle install

# Active Storage 用テーブルのマイグレーションファイルを生成
bin/rails active_storage:install

# マイグレーションを実行（active_storage_blobs などのテーブルが作られる）
bin/rails db:migrate
```

### 生成されるテーブル

| テーブル | 役割 |
|---|---|
| `active_storage_blobs` | ファイルのメタ情報（ファイル名・サイズ・MIMEタイプなど）を保存 |
| `active_storage_attachments` | モデルと blob を紐付ける中間テーブル |
| `active_storage_variant_records` | リサイズなど加工した画像のキャッシュ情報を保存 |

実ファイルは `storage/` ディレクトリに保存される。

### 必要な gem

```ruby
# Gemfile
gem 'image_processing', '~> 1.2'       # variant でのリサイズに必要
gem 'active_storage_validations'        # content_type バリデーションに必要
```

`content_type:` バリデーションは Rails 標準の `validates` だけでは使えない。
`active_storage_validations` gem を追加しないと `Unknown validator: 'ContentTypeValidator'` エラーになる。

### モデルに宣言する

```ruby
class User < ApplicationRecord
  devise ...
  has_one_attached :avatar
  validates :avatar, content_type: %i[jpg png gif], if: -> { avatar.attached? }
end
```

- `if: -> { ... }` はラムダ。クラス読み込み時ではなくバリデーション実行時に毎回評価される
- `if avatar.attached?` をクラス直下に書くのは NG（クラス読み込み時に1回だけ評価されてしまう）

### フォームにファイル選択を追加する

```erb
<div class="field">
  <%= f.label :avatar %><br />
  <%= f.file_field :avatar %>
</div>
```

`f` はフォームオブジェクト。パーシャルに `f: f` で渡されていればそのまま使える。

### 画像を表示する（リサイズあり）

```erb
<% if user.avatar.attached? %>
  <%= image_tag user.avatar.variant(resize_to_limit: [320, 320]) %>
<% end %>
```

- `attached?` で添付済みか確認してから表示する
- `resize_to_limit` はアスペクト比を保ったまま指定サイズ以内に収める（歪まない）
- `resize_to_fill` は指定サイズにトリミングする
- サイズ指定は数値だけ。`[320px, 320px]` は NG、`[320, 320]` が正しい
- `f.file_field` でファイル選択フィールドを追加する（`file_area` は存在しない）

### N+1 対策

一覧画面でアバターを表示する場合、コントローラで `with_attached_avatar` を使う：

```ruby
def index
  @users = User.order(:id).with_attached_avatar.page(params[:page])
end
```

テキストカラムと違い、画像は別テーブル（`active_storage_attachments`）にあるため、
事前にまとめて取得しないとユーザー数分のSQLが発行される（N+1問題）。

### libvips のインストール

`image_processing` gem はシステムに `libvips` が必要。入っていないと `LoadError` になる：

```bash
sudo apt-get install -y libvips
```
