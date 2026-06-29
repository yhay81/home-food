# home-food

家の食材、冷蔵庫・冷凍庫・常温在庫、日々の食事、買い物をAIエージェントが継続管理するための台帳リポジトリです。

ユーザーはレシート、冷蔵庫画像、料理写真、食事メモ、外部レシピを送ります。エージェントは `AGENTS.md` に従って記録を更新し、現在の在庫、今日のおすすめ料理、追加購入すべきもの、期限が近いもの、健康・食費の観点の提案を返します。

## 構成

```text
.
├── AGENTS.md
├── data
│   ├── events/events.jsonl
│   ├── expense-categories.json
│   ├── inbox/README.md
│   ├── inventory/current.yml
│   ├── meals/log.yml
│   ├── profile.yml
│   ├── shopping/current.yml
│   └── sources/catalog.yml
├── docs
│   ├── expense-categorization.md
│   ├── recommendation-policy.md
│   └── recording-policy.md
└── templates
    ├── fridge-observation.md
    ├── inventory-item.yml
    ├── meal-log.md
    └── receipt.md
```

## 使い方

1. エージェントにレシート、冷蔵庫画像、食事内容、買い物予定、レシピURLなどを渡します。
2. 必要なら原本を `data/inbox/` に置き、エージェントは `data/sources/catalog.yml` と `data/events/events.jsonl` に根拠を追記します。
3. エージェントは `data/inventory/current.yml`、`data/meals/log.yml`、`data/shopping/current.yml` を更新します。
4. ユーザーは「今あるものは？」「今日何を作る？」「何を買う？」「期限が近いものは？」と聞きます。

記録の中心は `AGENTS.md` です。迷った場合は、履歴を残し、確信度を付け、現在在庫を更新してから回答します。

レシートは食材台帳の更新に加えて、家計簿アプリ expenses.haya.homes へ **3層カテゴリ**（大分類・中分類・小分類）で登録します。仕分けの判断基準は `docs/expense-categorization.md`、カテゴリ定義は `data/expense-categories.json`（アプリ側と同期）です。

## Public files

`public/` は `https://files.haya.homes/agents/kawasaki/` として Caddy から配信されます。ユーザーに見せる生成物だけを置き、実ファイルは Git にコミットしません。
