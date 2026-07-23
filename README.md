# 抄読会 PubMed ダイジェスト

アバロパラチド／アセトアミノフェンの **Impact Factor 5 以上の臨床文献**を
新しい順に各 10 編、日本語要約と抄読ポイント付きで表示するスマートフォン Web アプリ。

プルダウンで薬剤を切り替え、「両方を表示」では 2 領域に振り分けて表示します。

## 構成

```
pubmed-journal-club/
├─ docs/                   ← GitHub Pages の配信ディレクトリ
│  ├─ index.html           アプリ本体（フレームワーク不使用・単一ファイル）
│  ├─ data/
│  │  └─ latest.json       文献データ。週次でこれだけを差し替える
│  ├─ manifest.webmanifest PWA 設定（ホーム画面追加用）
│  ├─ sw.js                Service Worker（シェルはオフライン、データは network-first）
│  └─ icons/               アプリアイコン
├─ archive/                過去の latest.json（配信対象外）
├─ REGENERATE.md           週次再生成タスクの指示書
└─ README.md
```

**データとビューを分離**してあります。週次更新で書き換えるのは `data/latest.json` だけで、
`index.html` は触りません。

## 動かし方

### ローカル確認

`file://` で直接開くと `fetch()` がブラウザに拒否されてデータを読めません。
簡易サーバー経由で開いてください。

```bash
python -m http.server 8140 --directory docs
```

`http://localhost:8140/` を開く。

### スマートフォンで使う

GitHub Pages の公開 URL をスマホのブラウザで開き、「ホーム画面に追加」。
PWA として全画面で起動します。

Service Worker により、**一度開けば圏外でも直前のデータを閲覧できます**。
複数人での閲覧はサーバー側に何も持たないため、人数制限はありません。

## 更新の流れ（案C：定期再生成）

1. **毎週月曜 7 時**、スケジュールタスク `weekly-journal-club-pubmed` が `REGENERATE.md` の手順を実行
2. PubMed を検索 → ホワイトリストで IF フィルタ → 日本語要約を生成
3. `docs/data/latest.json` を上書き（旧版は `archive/` に退避）
4. **GitHub に push** → Pages が 1〜2 分で再ビルド
5. 利用者がアプリの「更新」ボタンを押すとキャッシュを無視して再取得する

スケジュールタスクは **Claude アプリが起動しているときに動きます。**
月曜朝に閉じていた場合は、次回起動時に実行されます。

タップした瞬間に PubMed を叩くわけではないので、**API のレート制限や
API キーの管理が一切発生しない**のが、この方式の利点です。

## 既知の制約

- **IF フィルタはホワイトリスト方式。** JCR は有料ライセンスで公開 API が無いため、
  雑誌名のリストで代替しています。リスト外の新興誌の良質論文は抽出されません。
  年 1 回、JCR 更新時に `REGENERATE.md` のリストを見直してください。
- **IF は概数**です。`J Clin Endocrinol Metab`（約 5.0）、`Ann Emerg Med`（約 5.0）は境界域。
- **`sort=date` は Entrez 登載日順**で、epub ahead of print との間にラグがあります。
- 抄録が PubMed に無い文献（Research Letter、Clinical Review 等）は
  カードに「抄録なし・本文確認要」と表示されます。**数値の引用前に必ず原著を確認してください。**

## 薬剤を追加したいとき

`data/latest.json` の `drugs` 配列にオブジェクトを 1 つ足すだけで、
プルダウンの選択肢とセクションが自動的に増えます。`index.html` の変更は不要です。

`theme` は `"a"`（緑）と `"b"`（茶）が用意されています。
3 剤目以降は `index.html` の CSS に `.sec-c` を追加してください。
