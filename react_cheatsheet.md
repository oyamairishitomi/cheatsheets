# React チートシート

## コンポーネント

関数を定義してJSXを返す。画面のパーツを作る単位。

```jsx
function MyComponent() {
  return <div>Hello</div>;
}
```

外から使えるようにするには `export default` をつける。1ファイルに1つだけ。

```jsx
export default function MyComponent() { ... }
```

他のコンポーネントの中で使うときは `<MyComponent />` と書く。

---

## JSX

JavaScriptの中にHTMLっぽく書ける記法。

- `class` → `className`（JavaScriptの予約語と被るため）
- `for` → `htmlFor`
- `return` で複数行返すときは `()` でくくる
- 複数要素を返すときは1つの親要素でくくる

```jsx
return (
  <div>
    <p>Hello</p>
    <p>World</p>
  </div>
);
```

変数や式を埋め込むときは `{}` で囲む。

```jsx
const name = 'Hitomi';
return <p>{name}</p>;
```

---

## Props

親コンポーネントから子コンポーネントに値を渡す仕組み。

**渡す側：**
```jsx
<Square value="X" />
```

**受け取る側：**
```jsx
function Square(props) {
  return <button>{props.value}</button>;
}
```

分割代入で書くとすっきりする：
```jsx
function Square({ value }) {
  return <button>{value}</button>;
}
```

---

## イベントハンドラ

JSXの要素に直接 `onClick` などを書く。DOMを直接触らない。

```jsx
function Square() {
  function handleClick() {
    alert('クリックされた！');
  }

  return <button onClick={handleClick}>X</button>;
}
```

`onClick={handleClick}` → クリック時に実行（OK）
`onClick={handleClick()}` → 括弧つきは今すぐ実行されるのでダメ

---

## State（useState）

コンポーネントの中で変化する値を管理する仕組み。
普通の変数を変えても画面は更新されない。stateを使うと更新される。

```jsx
import { useState } from 'react';

function Square() {
  const [value, setValue] = useState(null);
  //     ↑現在の値  ↑更新関数   ↑初期値

  function handleClick() {
    setValue('X'); // valueが'X'になって画面が再描画される
  }

  return <button onClick={handleClick}>{value}</button>;
}
```

- `setValue` はReactが用意した関数なので自分で定義しない
- 呼ぶだけで値が更新されて画面も自動で再描画される

---

## Lifting State Up（状態の引き上げ）

複数のコンポーネントで状態を共有したいとき、親コンポーネントにstateをまとめる。

```jsx
export default function Board() {
  const [squares, setSquares] = useState(Array(9).fill(null));
  // [null, null, null, null, null, null, null, null, null]

  return (
    <Square value={squares[0]} />
    // ...
  );
}
```

子は受け取るだけ、管理は親がやる。

---

## よくある間違い

| 間違い | 正しい書き方 |
|--------|-------------|
| `return` の後で改行 | `return (` で始める |
| `export default` を2つ書く | 1ファイルに1つだけ |
| `class=` と書く | `className=` |
| `document.getElementById()` でDOM操作 | `useState` + イベントハンドラ |
| `onClick={handleClick()}` | `onClick={handleClick}` |
| `props.value` と書いたのに `props` を引数に入れ忘れ | `function Square(props)` か `function Square({ value })` |
