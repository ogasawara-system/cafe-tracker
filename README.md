# 接客記録アプリ

カフェバーの接客パターンをiPadからタップで記録し、Google Sheetsにデータを蓄積するWebアプリです。

---

## 機能

- 接客のたびにワンタップで記録（TO/EI 4パターン）
- Google Sheetsにリアルタイム保存
- 月別・四半期・年別・全期間での構成比グラフ表示
- 本日の記録一覧・取り消し機能
- iPad Safari対応・ホーム画面追加でアプリ風に使用可能

## 接客パターン

| 略称 | 意味 |
|------|------|
| TO → EI | テイクアウト → イートイン（口コミ施策の転換） |
| EI → TO | イートイン → テイクアウト |
| TO → TO | テイクアウト → テイクアウト |
| EI → EI | イートイン → イートイン |

---

## セットアップ

### 1. Google スプレッドシートを作成

1. [sheets.google.com](https://sheets.google.com) で新規スプレッドシートを作成
2. シート名を `log` に変更（タブを右クリック → 名前変更）

### 2. Google Apps Script を設定

1. スプレッドシートのメニュー「拡張機能」→「Apps Script」を開く
2. 以下のコードを貼り付けて保存（Ctrl+S）

```javascript
const SHEET_NAME = 'log';
const HEADERS = ['timestamp', 'date', 'pattern', 'pattern_index'];

function doGet(e) {
  const result = handleRequest(e);
  return ContentService
    .createTextOutput(JSON.stringify(result))
    .setMimeType(ContentService.MimeType.JSON);
}

function handleRequest(e) {
  const params = e.parameter;
  const action = params.action;
  const ss = SpreadsheetApp.getActiveSpreadsheet();
  const sheet = ss.getSheetByName(SHEET_NAME);

  if (sheet.getLastRow() === 0) {
    sheet.appendRow(HEADERS);
  }

  if (action === 'add') {
    const now = new Date();
    const jst = new Date(now.getTime() + 9 * 60 * 60 * 1000);
    sheet.appendRow([jst.toISOString(), params.date, params.pattern, parseInt(params.pattern_index)]);
    return { status: 'ok' };

  } else if (action === 'get') {
    const data = sheet.getDataRange().getValues();
    const rows = data.slice(1).map(r => ({
      timestamp: r[0],
      date: r[1],
      pattern: r[2],
      pattern_index: r[3]
    }));
    return { status: 'ok', rows };

  } else if (action === 'delete_last') {
    const last = sheet.getLastRow();
    if (last > 1) sheet.deleteRow(last);
    return { status: 'ok' };

  } else {
    return { status: 'error', message: 'unknown action' };
  }
}
```

### 3. GAS をデプロイ

1. 右上「デプロイ」→「新しいデプロイ」
2. 種類：「ウェブアプリ」を選択
3. 実行ユーザー：「自分」
4. アクセスできるユーザー：「全員」
5. 「デプロイ」をクリック
6. 表示された **ウェブアプリURL** をコピーしておく

> ⚠️ 「Google hasn't verified this app」と表示された場合は「詳細」→「移動（安全でないページ）」→「許可」で進んでください。自分のスクリプトなので安全です。

### 4. アプリにURLを登録

1. アプリを開き「設定」タブを選択
2. コピーしたGAS URLを貼り付け
3. 「URLを保存して接続テスト」をタップ
4. 「接続成功」と表示されれば完了

---

## GitHub Pages での公開

```
https://あなたのユーザー名.github.io/cafe-tracker/
```

iPadのSafariで開き、**共有ボタン →「ホーム画面に追加」** でアプリとして使用できます。

---

## ファイル構成

```
cafe-tracker/
├── index.html   # アプリ本体
└── README.md    # このファイル
```

## データ構造（Google Sheets）

| 列 | 内容 | 例 |
|----|------|----|
| timestamp | 記録日時（JST） | 2025-05-01T13:24:00.000Z |
| date | 日付 | 2025-05-01 |
| pattern | パターン略称 | TO → EI |
| pattern_index | パターン番号（0〜3） | 0 |
