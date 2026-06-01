# Ruby チートシート

## キーワード引数

メソッドに渡す引数に名前をつける書き方。

```ruby
# 普通の引数（順番が重要）
def greet(name, age)
end
greet("太郎", 20)

# キーワード引数（名前で判断するので順番自由）
def greet(name:, age:)
end
greet(name: "太郎", age: 20)
greet(age: 20, name: "太郎")  # 同じ
```

Railsの `redirect_to` や `render` で頻出：

```ruby
redirect_to @tweet, notice: "保存しました"
render :new, status: :unprocessable_entity
```

`notice:` や `status:` がキーワード引数。コロンは**キー名の後ろ**につく。

## 標準入力

```ruby
input = gets.chomp   # ユーザーの入力を受け取る

# gets       → 入力を文字列で受け取る（末尾に\nが含まれる）
# .chomp     → 末尾の\n（改行）を取り除く

# 数値として扱いたいときはto_iやto_fで変換
num = gets.chomp.to_i   # 整数に変換
num = gets.chomp.to_f   # 小数に変換（約17桁まで。桁数が多い実数はNG）

# ⚠️ 桁数が多い実数（小数第100位など）はto_fで丸められる
# 計算しない・そのまま出力するだけなら文字列のままでOK
n = gets.chomp
puts n
```

実際の使い方：

```ruby
print "名前を入力してください: "
name = gets.chomp
puts "こんにちは、#{name}さん！"
```

**複数の値をスペース区切りで受け取る場合（paizaなど）：**
(&:to_i)
```ruby
a, b = gets.chomp.split.map(&:to_i)
```

処理の流れ：

```
"3 5\n"  →  "3 5"  →  ["3","5"]  →  [3,5]  →  a=3, b=5
  gets     .chomp     .split    .map(&:to_i)   a,b=
```

- `.split` → スペースで分割して配列に。引数なしで連続する空白をまとめて処理（JSと違い`split(" ")`不要）
- `.map(&:to_i)` → 全要素を整数に変換（`{ |x| x.to_i }`の省略形）
- `a, b =` → 配列を複数変数に一括代入（JSにはない書き方）

**よくあるミス：splitの後にto_iはNG**

```ruby
gets.chomp.split.to_i   # エラー！splitは配列を返すので.to_iは使えない
```

`.to_i` は文字列に使うメソッド。splitすると配列になるので `.map(&:to_i)` を使う：

```
"3 5"  →  ["3","5"]  →  .to_i NG（配列には使えない）
         .map(&:to_i) OK（各要素に.to_iを適用）
```

**縦（複数行）: 1行に1つの値**

```ruby
a = gets.chomp.to_i
b = gets.chomp.to_i
```

**⚠️ 超頻出パターン：N行の入力をそのまま出力**

```ruby
n = gets.to_i      # ← .to_i 必須（文字列のままだと.timesが動かない）
n.times do
  puts gets.chomp  # ← .to_i 不要（文字列はそのまま出力）
end
```

よくあるミス：
- `n = gets.chomp` → 文字列のまま `.times` しようとしてエラー
- `gets.chomp.to_i` → 文字列入力を整数に変換してしまう（数値問題以外はNG）
- `numbers << gets.chomp` をループ内で `puts` → 配列全体が毎回出力される

**縦で行数が多い場合: n行まとめて配列に入れて後で使う（数値の場合）**

```ruby
n = gets.to_i
numbers = []
n.times do
  numbers << gets.chomp.to_i   # <<は配列に要素を追加する演算子
end
```

**縦と横の違い（paizaでよく出る）**

```
# 縦：2行目以降に1つずつ    # 横：2行目にスペース区切りで全部
3                            3
10                           10 20 30
20
30
```

```ruby
# 縦の場合                   # 横の場合
n = gets.to_i                n = gets.to_i
n.times { puts gets.chomp }  puts gets.chomp.split
```

**NとN個の値が同じ行にある場合**

```
3 10 20 30   ← 1行目にNと値が全部
```

```ruby
arr = gets.chomp.split.map(&:to_i)
numbers = arr[1..]   # インデックス1以降（N本体を除いた部分）

numbers.each do |n|
  puts n
end
```

- `arr[1..]` → JSの `slice(1)` に相当。インデックス0（N）を除いた残り
- `shift` でも同じことができる（`arr.shift` で先頭を削除）

**よくあるミス：Nを読み込まずに使う**

コード中の変数名は問題文の「N」と無関係。事前に `gets` で読み込まないとエラーになる：

```ruby
# NG：Nが未定義
(1..N).each { |i| puts gets.chomp }

# OK：先にNを読み込む
n = gets.to_i
n.times { puts gets.chomp }
```

## 変数・定数

```ruby
name = "Hitomi"   # 変数（var/let/$不要）
PI = 3.14         # 定数（大文字始まり）
```

## 文字列

```ruby
name = "Ruby"
puts "Hello, #{name}!"   # 式展開（ダブルクォートのみ有効）
puts 'Hello, #{name}'    # シングルクォートは展開しない
```

**puts vs print**

```ruby
puts "hello"   # 改行あり → "hello\n"
print "hello"  # 改行なし → "hello"
```

スペース区切りで横に並べたいときは `print` ＋ 式展開：

```ruby
(1..10).each { |n| print "#{n} " }   # => 1 2 3 4 5 6 7 8 9 10 
```

**⚠️ 整数と文字列は `+` でつなげない（JSと違う）**

```ruby
n = 5
print n + " "       # エラー！
print "#{n} "       # OK（式展開を使う）
print n.to_s + " "  # OK（to_sで文字列に変換）
```

## 配列

```ruby
fruits = ["apple", "banana", "cherry"]
fruits[0]         # => "apple"
fruits.length     # => 3
fruits.reverse    # => ["cherry", "banana", "apple"]
fruits.sample     # => ランダムに1つ取り出す
fruits << "date"  # => 末尾に要素を追加

# include?：含まれているか調べる（JSのincludes()と同じ）
fruits.include?("banana")   # => true
fruits.include?("mango")    # => false

# 配列の配列にも使える
wins = [["グー","チョキ"], ["チョキ","パー"], ["パー","グー"]]
wins.include?(["グー","チョキ"])   # => true
```

**⚠️ 配列は0始まり：「M番目」≠ `lines[m]`**

問題文の「M番目」は1始まり。Rubyの配列は0始まりなので1を引く：

```ruby
parts = gets.chomp.split.map(&:to_i)
n = parts[0]
m = parts[1]

lines = gets.chomp.split.map(&:to_i)
puts lines[m - 1]   # M番目 → インデックスは m-1
```

```
lines = [10, 20, 30, 40, 50]
          ↑   ↑   ↑   ↑   ↑
index:    0   1   2   3   4
1番目   2番目 ...
```

- M=1 → `lines[0]`（`lines[1]` ではない）
- よくあるミス：`lines[m]`（1個ずれる）、`lines[m+1]`（2個ずれる）

**応用：配列Bの各要素を使って配列Aから値を取り出す**

> 要素数 N の数列 A と要素数 Q の数列 B が与えられます。
> 各 i について、A の B_i 番目の値を出力してください。

```ruby
n = gets.to_i
parts1 = gets.chomp.split.map(&:to_i)
q = gets.to_i
parts2 = gets.chomp.split.map(&:to_i)

parts2.each do |b|
  puts parts1[b - 1]   # bは1始まりなのでインデックスはb-1
end
```

- `each do |b|` でparts2の要素が `b` に1個ずつ入る（`m += 1` は不要）
- `b` をそのままインデックスに使うとずれるので `b - 1`

## putsと配列

`puts` に配列を渡すと、各要素を1行ずつ自動で出力する：

```ruby
puts ["apple", "banana", "cherry"]
# apple
# banana
# cherry
```

これを使うと、スペース区切りの1行入力を各行に出力するのが1行で書ける：

```ruby
puts gets.chomp.split   # 1行読んで分割→各要素を改行して出力
```

`puts` vs `p` の違い：

```ruby
arr = [1, 2, 3]
puts arr   # 1行ずつ出力
# 1
# 2
# 3

p arr      # 配列のまま1行で出力
# [1, 2, 3]
```

## 文字列メソッド

```ruby
# 文字列の切り出し
n = "1234567"
n[0, 3]    # => "123"      先頭から3文字（[開始位置, 文字数]）
n[3..]     # => "4567"     インデックス3以降の全部

# 桁数が3の倍数でない数値のカンマ区切り
amari = n.length % 3      # 先頭グループの文字数（0なら端数なし）
# amari > 0 のとき → 先頭amari文字 + 残りを3桁ずつ
# amari == 0 のとき → 最初からscanでOK

# scan：パターンにマッチする部分を配列で返す
"123456789".scan(/.{3}/)   # => ["123", "456", "789"]
"abcdefghi".scan(/.{3}/)   # => ["abc", "def", "ghi"]
# /.{3}/ は「任意の文字3つ」の正規表現

# カンマ区切りフォーマット（けた数が3の倍数のとき）
n = gets.chomp
puts n.scan(/.{3}/).join(",")   # "123456789" → "123,456,789"
```

⚠️ `to_s(:delimited)` や `to_formatted_s(:delimited)` はRailsのActiveSupportのメソッド。素のRubyでは使えない。

```ruby
"3 5 8".split        # => ["3", "5", "8"]（引数なしで空白区切り）
"3 5 8".split(" ")   # => ["3", "5", "8"]（同じ結果）
"3,5,8".split(",")   # => ["3", "5", "8"]（区切り文字を指定）

# join：配列をスペース区切りの文字列に（splitの逆、JSと同じ）
["8", "1", "3"].join(' ')   # => "8 1 3"
["8", "1", "3"].join        # => "813"（区切りなし）

# 注意：splitは配列を返すので文字列と==比較できない
s = gets.chomp          # 1つの文字列を受け取るときはsplitしない
s == "paiza"            # OK
gets.chomp.split == "paiza"   # NG（配列と文字列の比較になる）
```

## ハッシュ（連想配列）

```ruby
person = { name: "Hitomi", age: 25 }
person[:name]     # => "Hitomi"（キーはシンボル）
```

## 条件分岐

```ruby
if score >= 90
  puts "A"
elsif score >= 70
  puts "B"
else
  puts "C"
end

# 後置if（1行）
puts "合格" if score >= 60
```

範囲条件は`&&`でつなぐ（`60 <= score < 80`はNG）：

```ruby
if score >= 80
  puts "優"
elsif 60 <= score && score < 80
  puts "良"
else
  puts "可"
end
```

## ループ

```ruby
# timesメソッド（0始まり）
10.times do |n|
  puts n + 1    # 1〜10を出力
end

# 範囲オブジェクト + each
(1..10).each do |n|
  puts n
end
```

**`|i|` とは（ブロック変数）**

`each` がコレクションから1個ずつ取り出して `|i|` に入れてくれる。`i` の名前は何でもよい。

```
(1..3).each do |i|
  puts i          # 1周目 i=1, 2周目 i=2, 3周目 i=3
end
```

JSで書くと：`[1, 2, 3].forEach((i) => console.log(i))` と同じ。

**⚠️ よくある誤解：出力する数字を `gets` で読む必要はない**

N を受け取って 1〜N を出力する問題：

```
入力: 2
出力:
1
2
```

```ruby
# NG：2回目のgetsは存在しない入力を読もうとする
n = gets.to_i
(1..n).each do |i|
  puts gets.to_i   # ← 入力はもう終わっている！
end

# OK：出力する数字はプログラムが自分で生成する
n = gets.to_i      # 入力は N の1行だけ
(1..n).each do |i|
  puts i           # i はプログラムが 1, 2, ... と作った数字
end
```

入力は「何個出力するか」という指示だけ。出力する数字は `gets` ではなくループが生成する。

```ruby

# 配列のeach
fruits.each do |fruit|
  puts fruit
end

# while
i = 0
while i < 3
  puts i
  i += 1
end

# loop do：無限ループ（breakで抜ける）
loop do
  input = gets.chomp
  break if input == "やめる"   # 条件を満たしたら抜ける
  puts "続けます"
end
```

## メソッド定義

```ruby
def greet(name)
  "Hello, #{name}!"   # returnは省略可（最後の式が戻り値）
end

puts greet("Hitomi")
```

`return`は省略するのが慣習。`if`全体も式として戻り値になる：

```ruby
def grade(score)
  if score >= 80
    "優"
  elsif score >= 60
    "良"
  else
    "可"
  end
end

puts grade(85)   # => 優
```

## メソッドの組み合わせ

メソッドの中から別のメソッドを呼び出せる：

```ruby
def double(n)
  n * 2
end

def double_all(arrays)
  arrays.each do |array|
    puts double(array)   # doubleメソッドを呼び出す
  end
end

double_all([1, 2, 3, 4, 5])
# => 2, 4, 6, 8, 10
```

## スコープ

`def`で作るメソッドのスコープは外と完全に隔離される（JSのクロージャとは逆）：

```ruby
sum = 0
def sum_array(arrays)
  sum += 1   # エラー！外のsumは見えない
end
```

`do ~ end`ブロックは外の変数を参照できる：

```ruby
sum = 0
[1, 2, 3].each do |n|
  sum += n   # OK（ブロックは外と繋がっている）
end
```

メソッド内で使う変数はメソッドの中で初期化する：

```ruby
def sum_array(arrays)
  sum = 0              # メソッド内で初期化
  arrays.each { |n| sum += n }
  sum                  # 戻り値（returnは省略）
end

puts sum_array([3, 8, 1, 5, 10, 2])   # => 29
```

## クラス

```ruby
class Dog
  def initialize(name)   # コンストラクタ（PHPの__construct）
    @name = name         # @はインスタンス変数
  end

  def bark
    "#{@name}がワンワン！"
  end

  def age_group(age)
    if age < 3
      "子犬"
    elsif age < 7
      "成犬"
    else
      "老犬"
    end      # ifのend
  end        # メソッドのend
end          # classのend
```

`end`はインデントに関係なく、**数が合っているか**が重要。インデントを揃えると数えやすくなる：

```
class        → end が1個必要
  def        → end が1個必要
    if       → end が1個必要
    end      # ifを閉じる
  end        # defを閉じる
end          # classを閉じる
```

## ゲッターメソッド

インスタンス変数はクラスの外から直接アクセスできない。ゲッターメソッドで取り出す：

```ruby
def name
  @name
end

def age
  @age
end
```

`attr_reader`を使うと1行で書ける（推奨）：

```ruby
class Cat
  attr_reader :name, :age   # def name / def age と同じ

  def initialize(name, age)
    @name = name
    @age = age
  end
end

cat = Cat.new("こうじ", 3)
puts cat.name   # => こうじ
puts cat.age    # => 3
```

## セッターメソッド

インスタンス変数を外から書き換えるメソッド。メソッド名に`=`をつける：

```ruby
def name=(new_name)
  @name = new_name
end

cat.name = "たろう"   # 書き換え
```

`attr_accessor`でゲッター＋セッターを1行で定義できる（実務でよく使う）：

```ruby
class Cat
  attr_accessor :name, :age   # 読み取り＋書き換え両方OK
  attr_reader :name           # 読み取りのみ
  attr_writer :name           # 書き換えのみ

  def initialize(name, age)
    @name = name
    @age = age
  end
end

cat = Cat.new("こうじ", 3)
cat.name = "たろう"    # セッター
puts cat.name          # => たろう（ゲッター）
```

## インスタンスの生成と使い方

```ruby
dog = Dog.new("ポチ")     # インスタンス生成
puts dog.bark             # メソッド呼び出し

# 配列に入れてeachで回せる
dogs = [Dog.new("ポチ"), Dog.new("タロ")]
dogs.each { |dog| puts dog.bark }
```

## よく使うイディオム（条件分岐×ループ）

```ruby
# 条件に合う要素だけ出力
(1..10).each { |n| puts n if n > 5 }

# 文字列の条件分岐
animals = ["dog", "cat", "bird"]
animals.each { |a| puts "ワンワン" if a == "dog" }

# 条件に合う要素をカウント
count = 0
[12, 7, 23, 5, 18].each { |n| count += 1 if n >= 10 }
puts "10以上は#{count}個"
```

## よく使うイディオム

```ruby
# 合計を求める
sum = 0
[3, 1, 4, 1, 5].each { |n| sum += n }

# 最大値・最小値・差（組み込みメソッドで一発）
input = gets.chomp.split.map(&:to_i)
puts input.max          # 最大値
puts input.min          # 最小値
puts input.max - input.min   # 最大値と最小値の差

# 最大値を求める（eachで自前実装する場合）
max = 0
[3, 1, 4, 1, 5].each { |n| max = n if n > max }

# 偶数だけ出力
(1..20).each { |n| puts n if n % 2 == 0 }

# FizzBuzz（条件の順番に注意）
(1..30).each do |n|
  if n % 15 == 0
    puts "FizzBuzz"
  elsif n % 3 == 0
    puts "Fizz"
  elsif n % 5 == 0
    puts "Buzz"
  else
    puts n
  end
end
```

## よくある問題パターン

**大きな数値をカンマ区切りで出力する（桁数が3の倍数とは限らない）**

> 大きな数値Nが入力されます。位の小さい方から3けたごとにカンマ区切りで出力してください。

```ruby
n = gets.chomp
amari = n.length % 3       # 先頭グループの文字数（0なら端数なし）
nums = n[amari..].scan(/.{3}/).join(',')  # 残りを3桁ずつ区切ってjoin

if amari > 0
  puts n[0, amari] + ',' + nums   # 先頭グループ + カンマ + 残り
else
  puts nums                        # 端数なしはそのままでOK
end
```

ポイント：
- `n.length % 3` で先頭グループの桁数がわかる（0なら端数なし）
- `n[0, amari]` → 先頭amari文字（文字列）
- `n[amari..]` → amari文字目以降（文字列）
- `scan(/.{3}/)` → 3文字ずつの配列に分割
- `amari == 0` のときは先頭グループ不要なので場合分けが必要



**入力された各数字を上限として1からの数列を出力する**

> 自然数NとN個の数列Mが与えられます。i行目には1以上M_i以下の数値を昇順で出力してください。

```
入力例:
4
2 4 3 1

出力例:
1 2
1 2 3 4
1 2 3
1
```

```ruby
n = gets.chomp.to_i
m = gets.chomp.split.map(&:to_i)   # [2, 4, 3, 1]

m.each do |limit|
  puts (1..limit).to_a.join(' ')
end
```

ポイント：
- `m.each` でMの各要素を1個ずつ取り出す
- 各要素を上限 `limit` として `(1..limit)` の範囲を出力
- `n` は `m.each` で自動的に回数が決まるので使わなくてOK

**i行目に1〜iの数列を出力する**

> 自然数Nが与えられます。i行目には1以上i以下の数値をスペース区切りで出力してください。

```ruby
n = gets.chomp.to_i
(1..n).each do |i|
  puts (1..i).to_a.join(' ')
end
```

ポイント：
- 外側のループは `(1..n).each` で1始まりにする（`n.times` は0始まりなので注意）
- `(1..i)` はRangeオブジェクト。`to_a` で配列に変換してから `join` する
- `arr` を使って溜める必要はない。各行をその場で生成して `puts` するだけ

よくある間違い：
- `n.times do |i|` → i が0始まりになり、1行目が `(1..0)` で空になる
- 配列をループまたいで使い回す → 前の行の要素が残って混ざる
- ループ変数を使い回す（外側と内側で同じ名前）→ どの値か追えなくなる

**九九表を出力する**

> 出力のi行j列目の数値がi*jになるように9行9列で出力してください。

```ruby
9.times do |i|
  row = []
  9.times do |j|
    row << (i+1) * (j+1)
  end
  puts row.join(' ')
end
```

ポイント：
- `i` `j` は0始まりなので `+1` して1〜9にする
- 内側ループで1行分を配列に溜める
- 内側ループが終わったら `join(' ')` して `puts`

よくある間違い：
- `x+1` → 値を捨てているだけ。変数を変えるには `x += 1`
- 内側でリセットすべき変数をリセットし忘れる

**9個の数値を3行3列で出力する**

> 9個の数値が半角スペース区切りで入力されます。3行3列の形式で出力してください。

```ruby
nums = gets.chomp.split

3.times do |i|
  puts nums[i*3, 3].join(' ')
end
```    puts row.join(' ')
end

ポイント：
- `nums[i*3, 3]` → i行目の3要素を取り出す（`[開始位置, 文字数]`）
  - i=0: nums[0,3] → 1〜3番目
  - i=1: nums[3,3] → 4〜6番目
  - i=2: nums[6,3] → 7〜9番目
- `.join(' ')` で空白区切りの1行に
- `puts` で改行して出力

よくある間違い：
- `nums.join(' ')` → 9個全部つながってしまう
- `puts nums[i*3, 3]` → 各要素が別々の行に出力される（joinが必要）
- ループを二重にする → 外側の `3.times` だけでOK

**配列に対して複数の操作を実行する（push/pop/print）**

> 数列 A に対し操作を Q 回行う。`0 x`=末尾に追加、`1`=末尾削除、`2`=スペース区切り出力。

```ruby
n, q = gets.chomp.split.map(&:to_i)
a = gets.chomp.split.map(&:to_i)

q.times do
  line = gets.chomp.split.map(&:to_i)   # ← lineもto_iで整数に
  if line[0] == 0
    a.push(line[1])    # line[1]が追加する値（line[1]であってpush(1)ではない）
  elsif line[0] == 1
    a.pop
  else
    puts a.join(' ')
  end
end
```

よくあるミス：
- `line[0] == "0"` → `line.map(&:to_i)` 済みなら整数 `0` と比較
- `a.push(1)` → 常に1を追加してしまう。`a.push(line[1])` が正しい
- `print a.join(' ')` → 改行なし。`puts` を使う

**2次元グリッド（迷路）の特定マスを調べる**

> H×Wの迷路が与えられる。r行c列のマスが「#」なら Yes、「.」なら No を出力。

```ruby
h, w, r, c = gets.chomp.split.map(&:to_i)

maze = []
h.times do
  maze << gets.chomp   # 1行を文字列のまま配列に追加
end

if maze[r - 1][c - 1] == "#"   # 1始まり → 0始まりに変換
  puts "Yes"
else
  puts "No"
end
```

- `maze[i]` → i行目の文字列
- `maze[i][j]` → i行目のj文字目（文字列は `[]` で1文字取り出せる）
- r・cは1始まりなので `-1` してインデックスに変換

**配列をB個ずつ分割して出力する**

> 数列AをB_1個、B_2個...で分割し、各グループをスペース区切りで1行ずつ出力する。

```ruby
n, m = gets.chomp.split.map(&:to_i)
a = gets.chomp.split.map(&:to_i)
b = gets.chomp.split.map(&:to_i)

pos = 0
b.each do |count|
  puts a[pos, count].join(' ')   # pos番目からcount個取り出してスペース区切りで出力
  pos += count                   # ポインタを進める
end
```

- `a[pos, count]` → pos番目からcount個まとめて取り出す（スライス）
- `a[0, 3]` → `[1, 2, 3]`、`a[3, 2]` → `[4, 5]` のように使う
- ポインタ `pos` で「どこまで読んだか」を管理する

## JS/PHPとの対比

| | Ruby | JS | PHP |
|---|---|---|---|
| 変数宣言 | `name =` | `let name =` | `$name =` |
| 文字列展開 | `"#{name}"` | `` `${name}` `` | `"$name"` |
| 連想配列 | `{ name: "x" }` | `{ name: "x" }` | `["name" => "x"]` |
| インスタンス変数 | `@name` | `this.name` | `$this->name` |
| インクリメント | `i += 1` | `i++` | `$i++` |
| else if | `elsif` | `else if` | `elseif` |
| ブロック終わり | `end` | `}` | `}` |
