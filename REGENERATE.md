# 週次再生成タスク指示書

このファイルは、週1回のスケジュール実行で `docs/data/latest.json` を更新するための作業指示です。
**このファイルの内容をそのままエージェントに渡せば再生成できる**ように書いてあります。

---

## タスク

PubMed を検索し、`C:\Users\kahoo\Downloads\pubmed-journal-club\docs\data\latest.json` を
最新版に置き換え、**GitHub に push する**。
`docs/index.html` は変更しない（データ層だけを差し替える設計）。

push しないとスマートフォンから見えるデータが更新されないので、**push まで必ず完了させること。**

---

## 手順

### 1. 各薬剤について esearch を実行

NCBI E-utilities を使う。ベース URL:
`https://eutils.ncbi.nlm.nih.gov/entrez/eutils/`

**アバロパラチド**

```
esearch.fcgi?db=pubmed&retmode=json&retmax=60&sort=date&term=abaloparatide[tiab] AND (<骨代謝ホワイトリスト>)
```

**アセトアミノフェン**

```
esearch.fcgi?db=pubmed&retmode=json&retmax=80&sort=date&term=(acetaminophen[tiab] OR paracetamol[tiab]) AND (<鎮痛ホワイトリスト>)
```

ホワイトリストは下記の雑誌名を `"雑誌略名"[ta] OR ...` の形で連結する。

### 2. esummary で雑誌名・出版日・publication type を取得

```
esummary.fcgi?db=pubmed&retmode=json&id=<PMIDをカンマ区切り・25件ずつ>
```

### 3. 除外基準でふるいにかける

以下は**除外**する。

- 動物実験・in vitro・基礎研究のみのもの（title/abstract に mice, rats, in vitro 等）
- 症例報告（`Case Reports`）
- Letter / Comment / Editorial / News / Erratum
- 対象薬剤が主題でなく単に言及されているだけのもの

残ったものから**出版日の新しい順に各薬剤 10 編**を採用する。

### 4. efetch で抄録を取得

```
efetch.fcgi?db=pubmed&rettype=abstract&retmode=text&id=<PMIDをカンマ区切り・5件ずつ>
```

5 件ずつに分けるのは、要約時の情報欠落を防ぐため。

### 5. JSON を生成して上書き

`docs/data/latest.json` を下記スキーマで書き出す。`generatedAt` は実行日（YYYY-MM-DD）。

---

## 雑誌ホワイトリスト（IF 5 以上・JCR 2024 概数）

IF は有料ライセンスのため API 取得できない。以下のリストで代替する。
**年 1 回、JCR 更新時に見直すこと。**

### 骨代謝・内分泌（アバロパラチド用）

| 雑誌略名 | IF概数 |
|---|---|
| BMJ | 93 |
| N Engl J Med | 78 |
| Lancet | 88 |
| JAMA | 55 |
| Nat Rev Rheumatol | 28 |
| Nat Rev Endocrinol | 27 |
| Ann Intern Med | 20 |
| Lancet Diabetes Endocrinol | 20 |
| Endocr Rev | 20 |
| Drugs | 13 |
| Ann Rheum Dis | 12 |
| Annu Rev Med | 11 |
| Bone Res | 11 |
| JAMA Netw Open | 10 |
| Lancet Rheumatol | 10 |
| Cochrane Database Syst Rev | 9 |
| Lancet Healthy Longev | 9 |
| EClinicalMedicine | 9 |
| Nat Commun | 15 |
| Mayo Clin Proc | 7 |
| J Bone Miner Res | 6 |
| J Clin Endocrinol Metab | 5.0 ← **境界域** |

### 鎮痛・救急・集中治療・小児（アセトアミノフェン用）

| 雑誌略名 | IF概数 |
|---|---|
| BMJ / N Engl J Med / Lancet / JAMA | 上に同じ |
| Lancet Child Adolesc Health | 19 |
| J Hepatol | 26 |
| Intensive Care Med | 27 |
| JAMA Pediatr | 24 |
| JAMA Intern Med | 22 |
| JAMA Neurol | 20 |
| Gastroenterology | 25 |
| Circulation | 35 |
| JAMA Surg | 16 |
| Hepatology | 12 |
| Br J Anaesth | 9 |
| Anesthesiology | 9 |
| Crit Care | 9 |
| Anaesthesia | 8 |
| Crit Care Med | 7.7 |
| Ann Surg | 7.5 |
| Neurology | 7.7 |
| Pediatrics | 6.2 |
| Pain | 5.9 |
| BMC Med | 7 |
| Lancet Reg Health Eur | 10 |
| Ann Emerg Med | 5.0 ← **境界域** |

---

## JSON スキーマ

```jsonc
{
  "generatedAt": "YYYY-MM-DD",       // 実行日
  "source": "PubMed (NCBI E-utilities: esearch / esummary / efetch)",
  "ifBasis": "JCR 2024 概数",
  "notes": ["注意書きの配列（既存のものを踏襲）"],
  "drugs": [
    {
      "id": "abaloparatide",          // 変更しない（localStorage のキーになる）
      "labelJa": "アバロパラチド",
      "labelEn": "Abaloparatide",
      "query": "abaloparatide[tiab]",
      "theme": "a",                   // "a"=緑 / "b"=茶。変更しない
      "note": "領域の概況。争点が変わったら更新する",
      "articles": [
        {
          "pmid": "42461643",
          "title": "英語原題",
          "titleJa": "日本語訳題",
          "authors": "First A, et al.",
          "journal": "JAMA",          // 略名
          "if": "≈55",                // 文字列。ホワイトリストの値を使う
          "date": "2026-07-16",       // YYYY-MM-DD / YYYY-MM / YYYY のいずれか
          "design": "多施設RCT",       // 研究デザインを日本語で
          "n": "653",                 // 症例数。なければ null
          "doi": "10.1001/jama.2026.11895",
          "hasAbstract": true,        // 抄録が PubMed になければ false
          "summary": "抄録原文に基づく日本語要約。数値を必ず含める",
          "point": "抄読会での論点・議論のとっかかり"
        }
      ]
    },
    { "id": "acetaminophen", "theme": "b", "...": "同様" }
  ]
}
```

---

## 要約を書くときの厳守事項

これは医師が抄読会で使う資料である。**エビデンスの乏しい記述をしてはならない。**

1. **数値は抄録原文にあるものだけを書く。** 推測・補完・丸めによる創作は禁止。
   信頼区間・p 値・症例数は原文の表記をそのまま使う。
2. **抄録が PubMed に無い場合**（Research Letter、Clinical Review など）は
   `hasAbstract: false` とし、`summary` には
   「抄録が無いため本文確認が必要」である旨を明記する。**内容を想像で埋めない。**
3. **研究デザインを `design` に必ず明記する。** RCT の事後解析・二次解析は
   「RCT事後解析」と書き、一次解析と混同させない。
4. **代替エンドポイント（骨密度、動脈管閉鎖など）と臨床エンドポイント
   （骨折、死亡）を区別して書く。**
5. `point` には、その論文の**限界**（交絡、非ランダム化、探索的エンドポイント、
   早期中止など）を最低 1 つ含める。
6. 観察研究に因果を示す語（「〜が〜を引き起こす」）を使わない。「〜と関連していた」を使う。

---

## 実行上の注意

- **レート制限**：APIキーなしで 3 req/秒。連続で叩くと 429 が返る。
  無料の NCBI API キーを取得したら `&api_key=...` を付けて 10 req/秒に上げられる。
- **日付順の意味**：`sort=date` は Entrez への登載日順であり、
  epub ahead of print との間に数日〜数週のラグがある。
- **書き換え前に必ずバックアップ**：`docs/data/latest.json` を
  `archive/latest-YYYY-MM-DD.json` にコピーしてから上書きする。
  （`archive/` はリポジトリ直下。`docs/` の外に置くことで配信対象から外している）
- **検証**：書き出した JSON が `JSON.parse` できること、
  各 drug の `articles` が 10 件あることを確認してから終了する。
- 該当論文が 10 件に満たない場合は、**件数を水増しせず、実際の件数のまま**出力し、
  `notes` にその旨を追記する。

### 最後に必ず push する

```bash
git -C "C:\Users\kahoo\Downloads\pubmed-journal-club" add -A
git -C "C:\Users\kahoo\Downloads\pubmed-journal-club" commit -m "data: 文献データを YYYY-MM-DD 版に更新"
git -C "C:\Users\kahoo\Downloads\pubmed-journal-club" push
```

push 後、GitHub Pages のビルドに 1〜2 分かかる。
アプリ側は Service Worker が `data/` を network-first で扱うため、
利用者が「更新」ボタンを押せば新しいデータを取得する。
