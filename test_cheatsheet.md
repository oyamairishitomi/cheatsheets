# paiza JavaScript チートシート

## 入力の読み込み（テンプレート）

```js
process.stdin.resume();
process.stdin.setEncoding('utf8');
var lines = [];
var reader = require('readline').createInterface({ input: process.stdin });
reader.on('line', (line) => { lines.push(line.trim()); });
reader.on('close', () => {
  // ここに処理を書く
});
```

---

## 入力パターン別（ここが一番大事）

### パターンA：2行目以降に1行ずつデータが来る（c144, c148 タイプ）

```
3          ← lines[0]: データ数N
rock paper ← lines[1]: 1つ目
paper rock ← lines[2]: 2つ目
rock rock  ← lines[3]: 3つ目
```

```js
const N = parseInt(lines[0]);
for (let i = 1; i <= N; i++) {   // ← 1始まり、<= N でOK
  const data = lines[i].split(' ');
}
```

### パターンB：2行目に全データがスペース区切りで来る（c181 タイプ）

```
3              ← lines[0]: データ数N
hello hi hey   ← lines[1]: 全データ
```

```js
const N = parseInt(lines[0]);
const words = lines[1].split(' ');
for (let i = 0; i < N; i++) {    // ← 0始まり、< N でOK
  const word = words[i];
}
```

### パターンD：1行目に複数の値、2行目以降に1行ずつデータが来る（配列練習1タイプ）

```
5 3        ← lines[0]: N（要素数）と K（探す値）
10         ← lines[1]: 1つ目の要素
3          ← lines[2]: 2つ目
3          ← lines[3]: 3つ目
...
```

```js
const parts = lines[0].split(' ').map(Number);
const N = parts[0];
const K = parts[1];

const nums = [];
for (let i = 0; i < N; i++) {
  nums.push(Number(lines[i + 1]));  // ← i+1 が重要。lines[1]固定はNG
}

// K が何個含まれるか数える
let count = 0;
for (let j = 0; j < N; j++) {
  if (nums[j] === K) count++;
}
console.log(count);
```

### パターンC：1行目にまとめて数字が来る（c097 タイプ）

```
10 3 5   ← lines[0]: まとめて
```

```js
const parts = lines[0].split(' ');
const all  = parseInt(parts[0]);
const intX = parseInt(parts[1]);
const intY = parseInt(parts[2]);
```

---

## 最大値・最小値を求める（c181, Dpractice1）

```js
// 一番楽な方法（1行で書ける）
const nums = [3, 1, 4, 1, 5];
console.log(Math.min(...nums));              // 最小値: 1
console.log(Math.max(...nums));              // 最大値: 5
console.log(Math.max(...nums) - Math.min(...nums));  // 差: 4

// 各行が1つの数値の場合（Dpractice1タイプ）
console.log(Math.min(...lines.map(Number)));  // これだけでOK

// ループで求める場合（初期値に注意！）
let maxVal = -Infinity;  // 最大値の初期値は -Infinity
let minVal =  Infinity;  // 最小値の初期値は Infinity
for (let i = 0; i < nums.length; i++) {
  if (nums[i] > maxVal) maxVal = nums[i];
  if (nums[i] < minVal) minVal = nums[i];
}
```

## 余り・FizzBuzz系（c097）

```js
for (let i = 1; i <= N; i++) {
  if (i % x === 0 && i % y === 0) {
    console.log('AB');         // ← 文字列はクォートで囲む！変数名と区別する
  } else if (i % x === 0) {
    console.log('A');
  } else if (i % y === 0) {
    console.log('B');
  } else {
    console.log('N');
  }
}
```

## 切り捨て（c148）

```js
Math.floor(x / 2)  // 小数点以下切り捨て
```

---

## ループの種類

```js
const arr = ['apple', 'banana', 'cherry'];

// 従来の for（インデックスが必要なとき）
for (let i = 0; i < arr.length; i++) {
  console.log(arr[i]);
}

// for...of（要素だけ欲しいとき、シンプル）
for (const x of arr) {
  console.log(x);
}

// どちらも結果は同じ。インデックス不要なら for...of が楽
```

---

## オブジェクトとは

キーと値のペアでデータを管理するもの。出現回数カウントでよく使う。

```js
const count = {};           // 空のオブジェクト（メモ帳）
count['apple'] = 3;         // 'apple' に 3 を記録
count['banana'] = 2;
// → { apple: 3, banana: 2 }

count['apple']              // 3 を取り出す
```

**配列との違い**

```js
const arr = ['apple', 22];  // 番号で管理 → arr[0]
const obj = { name: 'apple', age: 22 };  // 名前で管理 → obj.name
```

**オブジェクトをループで回す**

```js
const count = { apple: 3, banana: 2, cherry: 1 };

// Object.entries → [['apple',3], ['banana',2], ['cherry',1]] に変換
for (const [word, n] of Object.entries(count)) {
  console.log(word, n);
}
// apple 3
// banana 2
// cherry 1
```

`[word, n]` は「配列の1番目を word、2番目を n に入れる」分割代入。

---

## 文字列の長さ

```js
"hello".length   // 5　← これだけでOK、.split('').length は不要
```

---

## 文字列の連結と繰り返し（c084）

```js
// 連結は + （. はPHPの書き方なのでNG）
'*' + lines[0] + '*'

// テンプレートリテラル（こっちが読みやすい）
`*${lines[0]}*`

// 繰り返しは .repeat()（* で掛け算はPythonの書き方なのでNG）
'*'.repeat(5)    // '*****'
'-'.repeat(10)   // '----------'
```

| 言語 | 連結 | 繰り返し |
|------|------|------|
| JavaScript | `'a' + 'b'` | `'a'.repeat(3)` |
| Python | `'a' + 'b'` | `'a' * 3` |
| PHP | `'a' . 'b'` | `str_repeat('a', 3)` |

---

## 挿入ソート

「手札に1枚ずつ加えて、そのたびに正しい位置に差し込む」イメージ。

```
入力例:
4          ← lines[0]: 要素数N
4 2 3 1    ← lines[1]: スペース区切りの数列
```

```js
const N = Number(lines[0]);
const parts = lines[1].split(' ').map(Number);

for (let i = 1; i <= N; i++) {
  let key = parts[i];   // 取り出す値
  let j = i - 1;        // keyより左の位置から比較開始

  while (j >= 0 && parts[j] > key) {
    parts[j + 1] = parts[j];  // keyより大きければ1つ右にずらす
    j--;
  }

  parts[j + 1] = key;  // 正しい位置に差し込む
}
console.log(parts.join(' '));
```

**よくあるバグ**

| ミス | 正しい書き方 |
|------|------|
| `while(i >= 0 ...)` | `while(j >= 0 ...)` ← ループ変数は `j` |
| `parts[j] = parts[j-1]` | `parts[j+1] = parts[j]` ← 右にずらす |
| `i=<N` | `i<=N` ← 向きに注意 |
| `for(i=1; ...)` | `for(let i=1; ...)` ← `let` を忘れずに |

---

## よくあるミスまとめ

| ミス | 原因 | 直し方 |
|------|------|--------|
| `i <= N` / `i < N` 間違える | パターンによって違う | 上の入力パターンで確認 |
| `for (let i = 1; i < N; i++)` で N-1個しか読まない | `lines[1]`〜`lines[N]` は N 個あるので `<=` が必要 | `i <= N` にする（1始まりで N 行読むときは `<=`）|
| `lines[0]` をそのまま使う | 文字列のまま | `parseInt(lines[0])` |
| 最小値の初期値を `0` にする | 更新されない | `Infinity` にする |
| 最大値の初期値を `0` にする | 負の数があると壊れる | `-Infinity` にする |
| `console.log(AB)` | 変数ABは未定義 | `console.log('AB')` と文字列にする |
| `split(' ')` した値をそのまま数値計算する | 文字列のまま | `.map(Number)` か `parseInt()` |
| `'*' * 5` で文字列を繰り返す | Pythonの書き方 | `'*'.repeat(5)` |
| `'a' . 'b'` で文字列を連結する | PHPの書き方 | `'a' + 'b'` |
| `split('')` と `split(' ')` を間違える | `''` は1文字ずつ、`' '` はスペース区切り | スペース区切りは必ず `' '` |
| `perseInt` / `parsInt` などのスペルミス | typo | `parseInt` （大文字のIに注意）|
| ループ変数を別のループ内で使う | `i` と `j` の混同 | ループの中では自分のループ変数を使う |
| `parseInt` を配列にかける | `split` の結果は配列 | `parts[0]`, `parts[1]` と添字で取り出してから `parseInt` |
| 「以上」の条件を `==` で書く | ちょうど同じときしか反応しない | `>=` を使う |
| `if (numbers[j < min])` | `]` の位置がずれている | `if (numbers[j] < min)` |
| ループ条件を `>` にする | 一度も回らない | `<` にする |
| `i`、`j`、`max` などを宣言なしで使う | 暗黙のグローバル変数になりバグの原因になる | 必ず `let` か `const` で宣言する |
| `j <= numbers.length` で配列をループ | 最後に `undefined` を参照する（`Number(undefined)` は `NaN`） | `j < numbers.length` にする（配列は `<` で止める）|
| `Infinity` を小文字で書く | 未定義エラーになる | 必ず大文字 `Infinity` |
| ループ内で `lines[1]` 固定にする | ループ変数 `i` を使い忘れ | `lines[i + 1]` と書く |
| `Number(amount).length` を使う | 数値に `.length` はなく `undefined` になる | `Number(amount)` だけでOK。`.length` は文字列・配列専用 |
| ループ内で `Number(lines[i])` を何度も呼ぶ | 同じ変換を繰り返している | `let num = Number(lines[i])` と変数に入れて `num` を使い回す |
| Yes/No をループ内の if-else で両方セットする | 一致しても次のループで `No` に上書きされる | `result = 'No'` を初期値にして、一致したら `result = 'Yes'; break;` |
| ループ内で `parts[i]` を使う | `parts` は `lines[0]` の分割なので要素数が少ない | 各行の値は `Number(lines[i])` で取る |
