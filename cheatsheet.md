# シッタカブッター 開発チートシート

## Artisanコマンド
```bash
php artisan make:migration create_users_table  # マイグレーション作成
php artisan make:request RegisterRequest       # Requestクラス作成
php artisan make:controller UserController     # コントローラー作成
php artisan make:model Term                    # モデル作成
```
すべてプロジェクトルート（`/srv/projects/laravel/shittaka/`）で実行。

---

## よくやるミス

### functionキーワード忘れ
```php
public showLogin(){         // ❌
public function showLogin(){ // ✅
```

### view()のクォート忘れ
```php
return view(register.register);   // ❌
return view('register.register'); // ✅
```

### withErrors()は複数形
```php
->withError([...])  // ❌
->withErrors([...]) // ✅
```しといて

### メソッドの引数に$requestを書く
```php
public function login(){                    // ❌ $requestが使えない
public function login(LoginRequest $request){ // ✅
```

### User::create()の書き方
```php
User::create('users')->([...]) // ❌
User::create([...])            // ✅
```

---

## モデル・DB

### モデルに書く3つのこと
```php
class Term extends Model
{
    // 1. $fillable: create()やupdate()で一括保存できるカラム
    protected $fillable = ['user_id', 'name', 'description'];
    // created_at は timestamps() が自動管理するので不要

    // 2. casts(): 保存・取得時の自動型変換
    protected function casts(): array
    {
        return ['password' => 'hashed'];
    }

    // 3. リレーション: テーブル間の関係
    public function user()
    {
        return $this->belongsTo(User::class); // TermはUserに属する
    }
}
```

### リレーションの書き方
```php
// 親側（User）: 1人が複数のTermを持つ
public function terms()
{
    return $this->hasMany(Term::class);
}

// 子側（Term）: 1つのTermは1人のUserに属する
public function user()
{
    return $this->belongsTo(User::class);
}
```

### パスワードハッシュはモデルに任せる
Userモデルのcastsに `'password' => 'hashed'` があれば `User::create()` 時に自動ハッシュ。`Hash::make()` は不要。

### DB::table()よりEloquentを使う
```php
DB::table('users')->insert([...]) // 動くが非推奨
User::create([...])               // ✅ モデルを使う
```

### テーブル名は複数形、モデル名は単数形
```php
DB::table('user')      // ❌
DB::table('users')     // ✅ テーブルは複数形

use App\Models\Users;  // ❌
use App\Models\User;   // ✅ モデルは単数形
```

### マイグレーションのカラム定義
型を指定してカラム名を文字列で渡す：
```php
$table->name();        // ❌ そんなメソッドはない
$table->description(); // ❌

$table->string('name');      // ✅
$table->text('description'); // ✅
```

### 外部キー（他テーブルとの紐付け）
参照先テーブル名の**単数形** + `_id`：
```php
$table->foreign(user_id)->id();          // ❌
$table->foreignId('user_id')->constrained(); // ✅ usersテーブルのidと紐付け
```
`users`テーブル → `user_id`（`users_id` ではない）

---

## バリデーション

### 入力チェック = Requestクラス
```
ブラウザ → Request（入力チェック） → Controller（処理）
```
チェックを通過したデータだけControllerに届く。`php artisan make:request XxxRequest` で作成。

### rules()とmessages()は別メソッド
```php
public function rules(): array
{
    return [
        'email' => ['required', 'email'],
        'password' => ['required'],
    ];
}

public function messages(): array
{
    return [
        'email.required' => 'メールアドレスを入力してください',
        'email.email'    => 'メールアドレスの形式で入力してください',
        'password.required' => 'パスワードを入力してください',
    ];
}
```

### チェックボックスの同意確認は `accepted`
```php
'consent' => ['accepted'], // チェックなしはここで弾く、DBには保存しない
```

---

## ルーティング

### 名前付きルートは定義してから使う
```php
route('front.index') // ❌ 定義してないとエラー
redirect('/login')   // ✅ 直接パスで書くのが確実
```

### authミドルウェアはloginという名前のルートが必須
`auth` ミドルウェアは未ログイン時にLaravel内部で `redirect()->route('login')` を呼ぶ決まりになっている。
`->name('login')` がないと500エラーになる：
```php
Route::get('/login', [UserController::class, 'showLogin'])->name('login'); // ✅ 必須
```

### getとpostの使い分け
- `Route::get` → 画面表示（URLを開く）
- `Route::post` → フォーム送信（処理する）

### `/`と`.`の使い分け
`/`（スラッシュ）→ URLのパス
```php
return redirect('/login');  // ブラウザのURL
Route::get('/login', ...);  // ルート定義
```
`.`（ドット）→ bladeファイルのパス（`view()` の中だけ）
```php
return view('login.login');       // resources/views/login/login.blade.php
return view('register.register'); // resources/views/register/register.blade.php
```

---

## CSRFトークン

LaravelがPOSTリクエストを正規のサイトからか確認するセキュリティ機能。書き忘れると**419エラー**になる。

**formタグの中**
```html
<form method="POST">
  @csrf
</form>
```

**JavaScriptのfetchの場合**
headに追加：
```html
<meta name="csrf-token" content="{{ csrf_token() }}">
```
fetchのheadersに追加：
```javascript
'X-CSRF-TOKEN': document.querySelector('meta[name="csrf-token"]').content
```

---

## 外部API連携（HTTPクライアント）

LaravelからOpenAIなど外部サーバーにリクエストを送る機能。

```
ブラウザ → Laravelサーバー  （普通のリクエスト）
Laravelサーバー → OpenAI API （これがHTTPクライアント）
```

```php
use Illuminate\Support\Facades\Http; // importが必要

$response = Http::withHeaders([
    'Authorization' => 'Bearer ' . env('OPENAI_API_KEY'), // 認証情報をヘッダーにセット
])->attach(
    'file',                                  // 受け取り側のフィールド名
    file_get_contents($file->getRealPath()), // 添付ファイルの中身
    'audio.webm'                             // ファイル名
)->post('https://api.openai.com/v1/audio/transcriptions', [
    'model' => 'whisper-1', // 使用モデル
    'language' => 'ja',     // 言語指定
]);

return $response->json('text'); // レスポンスのJSONから値を取り出す

// GPT APIはネストが深いのでドット区切りで辿る
// {"choices": [{"message": {"content": "解説テキスト"}}]}
return $response->json('choices.0.message.content');
```

### 用語メモ
- **ヘッダー**：リクエストの伝票。本文とは別に認証情報などを添付する
- **`Bearer`**：APIキー認証の決まり文句。`Authorization: Bearer APIキー` の形が規約
- **`attach`**：ファイルを添付する（メールの添付ファイルと同じイメージ）
- **`post`**：データを送りつけて結果を返してもらう。formタグの `method="POST"` と同じ意味

---

## import

使わないimportが残っていてもエラーにはならないが、使うfacadeやモデルのimportが**ない**とエラーになる。

### パターンは3種類
```php
use App\Models\クラス名;                   // モデル
use Illuminate\Support\Facades\クラス名;  // Facade（Laravelの機能）
use App\フォルダ名\クラス名;               // 自作クラス（RequestやService）
```

### 具体例
```php
use App\Models\Term;                        // モデル
use Illuminate\Support\Facades\Auth;        // Facade
use App\Http\Requests\WhisperRequest;       // 自作Request
use App\Services\GetMeanService;            // 自作Service
```

フォルダ構成がそのままパスになる。