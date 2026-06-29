# Receipt Intake Template

## Source

- source_id:
- received_at:
- store:
- purchased_at:
- raw_location:
- confidence:

## 家計簿登録（expenses.haya.homes）

レシートは食材台帳の更新に加えて、家計簿アプリ（expenses.haya.homes）へ **3層カテゴリ**で登録する。
仕分けの判断基準は `docs/expense-categorization.md`、カテゴリ定義は `data/expense-categories.json`。
L1 必須・L2 推奨・L3 任意。レシート全体は主用途のカテゴリ、明細は行ごとの実態のカテゴリを付ける。

- receipt_category_l1:        # 必須（例: 食費）
- receipt_category_l2:        # 推奨（例: 食料品）
- receipt_category_l3:        # 任意（例: 生鮮食品）
- total_amount_jpy:
- tax_amount_jpy:
- payment_method:             # 現金 / クレジットカード / 電子マネー / QR決済 / 口座振替 など
- payer:
- source: kawasaki
- registered: false           # POST /api/receipts 済みなら true

## Extracted Items

| name_on_receipt | normalized_name | quantity | unit | price_jpy | storage | category_l1 | category_l2 | category_l3 | confidence | notes |
| --- | --- | ---: | --- | ---: | --- | --- | --- | --- | ---: | --- |
|  |  |  |  |  |  |  |  |  |  |  |

> 明細の `category_*` を省くと、家計簿側ではレシート全体のカテゴリを継承する。
> 食品以外の品（化粧品=衣服・美容、洗剤=日用品 など）が同じレシートに混ざるときは、その行だけ別カテゴリにする。

## Events To Add

- type: purchase
- summary:
- inventory_updates:
- shopping_updates:
- expense_registered:         # 家計簿へ登録した receipt id（POST /api/receipts のレスポンス）
- uncertainty:
