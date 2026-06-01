# JS チートシート

## fetch の基本形

```javascript
fetch('送り先URL', {
  method: 'POST',
  headers: {
    'X-CSRF-TOKEN': document.querySelector('meta[name="csrf-token"]').content,
  },
  body: 送るデータ,
})
.then(res => res.json())
.then(data => console.log(data));
```

### JSONを送る場合
```javascript
headers: {
  'Content-Type': 'application/json',
  'X-CSRF-TOKEN': ...,
},
body: JSON.stringify({ key: value }),
```

### ファイル（FormData）を送る場合
```javascript
const formData = new FormData();
formData.append('フィールド名', ファイルデータ, 'ファイル名');

fetch(url, {
  method: 'POST',
  headers: {
    'X-CSRF-TOKEN': ...,
    // Content-Type は書かない（ブラウザが自動でセット）
  },
  body: formData,
})
```

---

## MediaRecorder（音声録音）

```javascript
let mediaRecorder;
let audioChunks = [];

// 開始
const stream = await navigator.mediaDevices.getUserMedia({ audio: true });
mediaRecorder = new MediaRecorder(stream);
audioChunks = [];

// 録音中：チャンクを溜める
mediaRecorder.ondataavailable = (event) => {
  if (event.data.size > 0) {
    audioChunks.push(event.data);
  }
};

// 停止後：Blobにまとめて使う
mediaRecorder.onstop = () => {
  const audioBlob = new Blob(audioChunks, { type: 'audio/webm' });
  // ここでfetch送信など
};

mediaRecorder.start();  // 録音開始

// 停止（stop()を呼ぶとonstopが発火する）
mediaRecorder.stop();
```

---

## async / await

```javascript
// asyncをつけた関数の中でawaitが使える
someButton.addEventListener('click', async () => {
  const result = await 時間のかかる処理();  // 完了を待ってから次へ
  console.log(result);
});
```

- `await` なしだと処理の完了を待たずに次の行に進んでしまう
- `navigator.mediaDevices.getUserMedia()` など非同期APIで必要

---

## Blob と FormData

```javascript
// Blob：バイナリデータをまとめるオブジェクト
const blob = new Blob([データの配列], { type: 'MIMEタイプ' });

// FormData：ファイルアップロード用のデータ形式
const formData = new FormData();
formData.append('キー名', 値);
formData.append('キー名', blob, 'ファイル名.webm');  // ファイルの場合
```
