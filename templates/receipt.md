# Receipt Intake Template

## Source

- source_id:
- received_at:
- store:
- purchased_at:
- raw_location:
- confidence:

## Extracted Items

| name_on_receipt | normalized_name | quantity | unit | price_jpy | storage | confidence | notes |
| --- | --- | ---: | --- | ---: | --- | ---: | --- |
|  |  |  |  |  |  |  |  |

## Events To Add

- type: purchase
- summary:
- inventory_updates:
- shopping_updates:
- uncertainty:

## expenses.haya.homes Payload

`home-food` への記録後、同じ購入を `POST http://127.0.0.1:8092/api/receipts` に送る。
`source` は `kawasaki`。token は `/opt/homebrew/var/lib/expenses-haya-homes/app/.env` から読み、値は記録しない。

```json
{
  "purchasedAt": "YYYY-MM-DD",
  "storeName": "",
  "category": "食費|日用品|交通|医療|住居|光熱|通信|娯楽|教育|その他",
  "totalAmount": 0,
  "taxAmount": null,
  "paymentMethod": "未指定",
  "payer": "家計",
  "source": "kawasaki",
  "memo": "home-food event_id=...; source_ref=...; uncertainty=...",
  "rawText": "OCR/extracted summary/raw location",
  "items": [
    {
      "name": "",
      "amount": 0,
      "quantity": 1,
      "category": "食費|日用品|その他",
      "memo": "在庫反映/対象外/不確実点"
    }
  ]
}
```
