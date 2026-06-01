or# paiza よく出るパターン集（未経験編）

---

## 1. 出現回数カウント（頻出度★★★）

文字や単語が何回出てくるか数える問題。

### ステップ1：カウント用オブジェクトを作る

```js
const count = {};
// count は空の「メモ帳」。{ apple: 3, banana: 2 } のように記録していく
```

### ステップ2：1つずつ見てカウントする

```js
const items = ['apple', 'banana', 'apple', 'cherry', 'banana', 'apple'];

for (const w of items) {
  count[w] = (count[w] || 0) + 1;
}
// count → { apple: 3, banana: 2, cherry: 1 }
```

`(count[w] || 0) + 1` の意味：
- `count[w]` がまだない（undefined）→ `0 + 1 = 1` にする
- `count[w]` がすでにある → その数に `+1` する

### ステップ3：一番多い要素を探す

```js
let maxWord = '';
let maxCount = 0;

for (const [word, n] of Object.entries(count)) {
  // Object.entries(count) → [['apple',3], ['banana',2], ['cherry',1]] に変換
  // [word, n] で「単語」と「回数」を同時に受け取る
  if (n > maxCount) {
    maxCount = n;
    maxWord = word;
  }
}

console.log(maxWord);  // 'apple'
```

### 入力が複数行のとき（paiza形式）

```
6        ← lines[0]
apple    ← lines[1]
banana   ← lines[2]
apple    ← lines[3]
...
```

```js
const N = parseInt(lines[0]);
const items = lines.slice(1, N + 1);  // lines[1]〜lines[N] を配列にする

const count = {};
for (const w of items) {
  count[w] = (count[w] || 0) + 1;
}

let maxWord = '';
let maxCount = 0;
for (const [word, n] of Object.entries(count)) {
  if (n > maxCount) {
    maxCount = n;
    maxWord = word;
  }
}

console.log(maxWord);
```

---

## 2. ソート（頻出度★★★）

```js
const nums = [3, 1, 4, 1, 5, 9, 2];

// 昇順（小→大）
nums.sort((a, b) => a - b);   // [1, 1, 2, 3, 4, 5, 9]

// 降順（大→小）
nums.sort((a, b) => b - a);   // [9, 5, 4, 3, 2, 1, 1]

// 文字列のソート（デフォルト）
const strs = ['banana', 'apple', 'cherry'];
strs.sort();                   // ['apple', 'banana', 'cherry']
```

⚠️ `nums.sort()` だけだと文字列順になるので数値には必ず `(a, b) => a - b` を書く

---

## 3. 配列の便利メソッド（頻出度★★★）

```js
const nums = [1, 2, 3, 4, 5];

// 条件に合う要素だけ取り出す
nums.filter(x => x % 2 === 0);    // [2, 4]

// 全要素を変換する
nums.map(x => x * 2);             // [2, 4, 6, 8, 10]

// 合計を求める
nums.reduce((sum, x) => sum + x, 0);  // 15

// 条件を満たす要素が存在するか
nums.some(x => x > 4);            // true

// 全要素が条件を満たすか
nums.every(x => x > 0);           // true
```

---

## 4. 文字列操作（頻出度★★★）

```js
const s = 'Hello World';

s.includes('World');        // true  （含むか）
s.startsWith('Hello');      // true  （始まりか）
s.endsWith('World');        // true  （終わりか）
s.indexOf('o');             // 4     （最初の位置）
s.slice(0, 5);              // 'Hello'（切り出し）
s.replace('World', 'paiza'); // 'Hello paiza'
s.toLowerCase();            // 'hello world'
s.toUpperCase();            // 'HELLO WORLD'
s.split('');                // ['H','e','l','l','o',' ','W','o','r','l','d']

// 文字列を逆にする
s.split('').reverse().join('');  // 'dlroW olleH'

// 回文チェック（よく出る！）
function isPalindrome(str) {
  return str === str.split('').reverse().join('');
}
```

---

## 5. 文字コード（頻出度★★）

アルファベットを数値として扱う問題で使う。

```js
'a'.charCodeAt(0);          // 97
'z'.charCodeAt(0);          // 122
'A'.charCodeAt(0);          // 65

String.fromCharCode(97);    // 'a'
String.fromCharCode(65);    // 'A'

// a=0, b=1, ... として番号を得る
const pos = 'c'.charCodeAt(0) - 'a'.charCodeAt(0);  // 2

// 大文字・小文字を判定
const c = 'A';
if (c >= 'A' && c <= 'Z') console.log('大文字');
if (c >= 'a' && c <= 'z') console.log('小文字');
```

---

## 6. 2次元配列 / グリッド（頻出度★★）

迷路や盤面の問題。

```
3 4        ← 行数H, 列数W
..#.
.#..
....
```

```js
const [H, W] = lines[0].split(' ').map(Number);
const grid = [];
for (let i = 0; i < H; i++) {
  grid.push(lines[i + 1].split(''));
}

// grid[行][列] でアクセス
grid[0][2];   // '#'

// 全セルを走査
for (let r = 0; r < H; r++) {
  for (let c = 0; c < W; c++) {
    console.log(grid[r][c]);
  }
}
```

---

## 7. 累積和（頻出度★★）

「i番目からj番目までの合計」を何度も聞かれる問題。

```js
const nums = [1, 2, 3, 4, 5];

// 累積和を作る
const prefix = [0];
for (const n of nums) {
  prefix.push(prefix[prefix.length - 1] + n);
}
// prefix → [0, 1, 3, 6, 10, 15]

// i〜j（0始まり、両端含む）の合計
const sum = (i, j) => prefix[j + 1] - prefix[i];
sum(1, 3);   // 2+3+4 = 9
```

---

## 8. 素数判定（頻出度★）

```js
function isPrime(n) {
  if (n < 2) return false;
  for (let i = 2; i <= Math.sqrt(n); i++) {
    if (n % i === 0) return false;
  }
  return true;
}

isPrime(7);   // true
isPrime(12);  // false
```

---

## 9. 最大公約数 / 最小公倍数（頻出度★）

```js
// 最大公約数（GCD）
function gcd(a, b) {
  return b === 0 ? a : gcd(b, a % b);
}

// 最小公倍数（LCM）
function lcm(a, b) {
  return (a / gcd(a, b)) * b;
}

gcd(12, 8);   // 4
lcm(4, 6);    // 12
```

---

## 10. 面積・長さの計算（重なりあり）（c099）

「足してから引く」パターン。重なりを考慮するとき、最大値から差分を引く方が楽。

```
←D→
┌────┐
│    ├──┐   2枚目は d_2 だけ重なる → D - d_2 だけ右に伸びる
│    │  │
└────┴──┘
```

```js
// ❌ 重なり分を直接足そうとする（混乱しやすい）
let length = 0;
for (...) { length += (D - d); }

// ✅ まず最大値を作って、引く
let totalWidth = D * N;          // 重なりゼロと仮定した最大幅
for (let i = 1; i < N; i++) {
  totalWidth -= parseInt(lines[i]);  // 重なり分を引く
}
const area = D * totalWidth;
```

**考え方の手順**
1. 重なりがないと仮定して最大値を求める
2. 重なっている分を引く

---

## 11. 入力をまとめて処理するイディオム

```js
// 複数行の数値をまとめて配列にする
const nums = lines.slice(1).map(Number);

// 2列のデータを同時に読む
for (let i = 1; i <= N; i++) {
  const [a, b] = lines[i].split(' ').map(Number);
}

// 分割代入で複数の値を一気に取得
const [x, y, z] = lines[0].split(' ').map(Number);
```
