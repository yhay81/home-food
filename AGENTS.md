# Home Food Agent Guide

このリポジトリの目的は、家の冷蔵庫・冷凍庫・常温在庫・日々の食事・買い物を、AIエージェントが継続的に記録し、ユーザーが「今あるもの」「今日食べるとよいもの」「買うべきもの」「期限が近いもの」「健康・食費の観点で気をつけること」を自然に聞ける状態に保つことです。

すべてのエージェントは、この `AGENTS.md` を最優先の運用手順として扱ってください。

## 基本方針

1. 記録を更新してから答える  
   レシート、冷蔵庫画像、食事報告、買い物報告、廃棄報告、ユーザーの訂正が入力されたら、回答前に台帳を更新します。

2. 履歴を消さない  
   推測が外れることを前提に、`data/events/events.jsonl` に追記型で事実・推測・訂正を残し、`data/inventory/current.yml` は現在状態のスナップショットとして更新します。

3. 根拠と確信度を残す  
   画像から見えたもの、レシートから読めたもの、ユーザーが明言したもの、料理から推定した消費量を区別します。各記録には `source_refs` と `confidence` を付けます。

4. 不明点は不明のまま管理する  
   量・期限・保存場所・開封状態が曖昧なら、勝手に確定しません。`unknown`、`estimated`、`needs_confirmation` を使って管理します。

5. ユーザーの負担を増やしすぎない  
   質問は、記録更新や安全な提案に必要なときだけ行います。軽微な不明点は仮置きし、次の画像・レシート・会話で自然に補正します。

6. 最新性が必要な情報は確認する  
   価格、商品仕様、外部レシピ、栄養・食品安全の一般情報など、変わり得る情報を根拠にする場合は最新情報を確認し、参照元を `data/sources/catalog.yml` または回答に残します。

## ディレクトリ

- `data/profile.yml`  
  家族構成、食の好み、健康目標、予算、避けたい食材、アレルギーなど。

- `data/inbox/`  
  受け取ったレシート画像・冷蔵庫画像・料理写真などの**原本を保存する場所**。`data/inbox/YYYY-MM-DD/<元のファイル名>` に置く。**家計簿の証憑になるレシート画像は必ずここへ原本をコピーする**（外食のテキスト報告など、そもそも原本画像が無い入力は対象外）。保存したら `data/sources/catalog.yml` の `raw_location` から参照する。`raw_location` は **この workspace の相対パス**（例 `data/inbox/2026-06-28/xxxx.jpg`）を正準にし、`~/.openclaw/inbound/line/...` のような workspace 外の絶対パスは書かない（このエージェントは `workspaceOnly` でその場所に到達できず、バックアップもされないため）。

- `data/events/events.jsonl`  
  すべての入力・観測・推定・訂正の追記型ログ。現在状態を再構築するための一次台帳。

- `data/inventory/current.yml`  
  現在の冷蔵庫・冷凍庫・常温在庫のスナップショット。ユーザーから「今何がある？」と聞かれたらここを中心に答えます。

- `data/meals/log.yml`  
  食事、弁当、外食、間食、食材消費の記録。

- `data/shopping/current.yml`  
  買い物候補、優先度、買わなくてよいもの、予算メモ。

- `data/sources/catalog.yml`  
  レシート、画像、レシピURL、外部情報、ユーザー発言などの情報源カタログ。

- `docs/recording-policy.md`  
  どう読み取り、どう更新し、どう矛盾を扱うかの詳細。

- `docs/recommendation-policy.md`  
  献立・買い物・期限・健康・食費の提案ルール。

- `templates/`  
  レシート、冷蔵庫画像、食事ログ、在庫アイテムの記録テンプレート。


## 受信した原本の保持（重要）

レシート・冷蔵庫・料理など、**画像/PDF として受け取った原本は、台帳を更新する前に必ず自分の workspace 内へコピーする**。これを省くと、原本は `~/.openclaw/inbound/`（このエージェントから到達不能・バックアップ対象外）にしか残らず、ディスク故障で家計の一次証憑が失われる。

- コピー先: `data/inbox/YYYY-MM-DD/<元のファイル名>`（受信日でディレクトリを分ける）。
- 委譲時、タチコマから渡された添付は `.openclaw/attachments/<id>/` 経由で読める。読めたらまず上記へコピーする。
- `data/sources/catalog.yml` の `raw_location` には、コピー後の **workspace 相対パス** を入れる。`~/.openclaw/inbound/...` の絶対パスを `raw_location` に書かない。
- 原本が手に入らない（テキスト報告のみ等）場合は `raw_location: null` のままでよい。その旨を `notes` に残す。
- `data/inbox/` は Git 追跡対象（`.gitignore` で除外していない）。push すれば private remote にバックアップされる。レシートの個人情報の扱いに注意し、極端に機微なものはコミット前に確認する。

## ブラウザ表示

Caddy はこのリポジトリ全体を `https://files.haya.homes/agents/kawasaki/` として配信します。ユーザーに開いてほしいファイルは、既存の `data/`、`docs/`、`templates/` など目的に合う場所に置き、対応する URL を回答に含めます。別途 `public/` は作りません。

## 作業開始時に読むもの

ユーザーから食材・料理・買い物・冷蔵庫状態について聞かれたら、最低限次を確認します。

1. `data/profile.yml`
2. `data/inventory/current.yml`
3. `data/meals/log.yml`
4. `data/shopping/current.yml`
5. 必要に応じて `data/events/events.jsonl` の末尾

入力が画像・レシート・レシピURL・食事報告を含む場合は、先に記録を更新してから回答します。

## イベント記録ルール

`data/events/events.jsonl` は1行1JSONです。削除や並べ替えはしません。訂正は `correction` イベントとして追記します。

推奨フィールド:

```json
{
  "event_id": "2026-06-27T21:00:00+09:00-receipt-001",
  "occurred_at": "2026-06-27T20:30:00+09:00",
  "recorded_at": "2026-06-27T21:00:00+09:00",
  "type": "purchase",
  "summary": "スーパーのレシートから牛乳、卵、鶏むね肉を追加",
  "items": [],
  "source_refs": ["src-20260627-receipt-001"],
  "confidence": 0.86,
  "notes": ""
}
```

主な `type`:

- `purchase`: レシート、買い物報告、宅配到着
- `inventory_observation`: 冷蔵庫・冷凍庫・棚の画像や目視報告
- `meal`: 食事、調理、弁当、外食、間食
- `consumption`: 食材を使い切った、量が減った
- `discard`: 廃棄、腐敗、期限切れ処分
- `correction`: ユーザー訂正、過去推定の修正
- `recipe_reference`: きょうの料理など外部レシピの参照
- `shopping_plan`: 買い物リストの追加・更新
- `system_init`: 初期化や構成変更

## 在庫スナップショット

`data/inventory/current.yml` は、ユーザーがそのまま読める形を優先します。厳密なグラム数が不明なものは、個数、袋、半分、少量などの自然な単位で構いません。

在庫アイテムの推奨フィールド:

```yaml
- id: "egg-2026-06-27"
  name: "卵"
  normalized_name: "卵"
  category: "卵・乳製品"
  storage: "fridge"
  amount:
    value: 10
    unit: "個"
    confidence: 0.9
  state: "unopened"
  purchased_at: "2026-06-27"
  opened_at: null
  use_by: null
  best_before: null
  expiry_basis: "label|receipt|default_estimate|user_report|unknown"
  consume_priority: "normal"
  last_seen_at: "2026-06-27"
  source_refs: ["src-20260627-receipt-001"]
  confidence: 0.9
  notes: ""
```

`consume_priority` は `urgent`、`soon`、`normal`、`stock`、`unknown` のいずれかを使います。

## 入力別の処理

### レシート

1. 店名、購入日時、合計金額、商品名、数量、単価を読み取ります。
2. 食材名を家庭で使う名前に正規化します。
3. 冷蔵・冷凍・常温の保存場所を推定します。
4. 期限が書かれていない場合は `expiry_basis: default_estimate` とし、過度に断定しません。
5. `purchase` イベントを追記し、`data/inventory/current.yml` と `data/shopping/current.yml` を更新します。
6. 同じ内容を `expenses.haya.homes` にも登録します。下の「expenses.haya.homes への家計簿登録」を必ず確認します。

#### expenses.haya.homes への家計簿登録

レシートや買い物報告を `purchase` として記録したら、`home-food` の台帳更新後に、同じ購入を `expenses.haya.homes` の SQLite 家計簿へ `POST /api/receipts` で登録します。

- API は `http://127.0.0.1:8092/api/receipts` を使います。LAN/DNS 経由ではなく loopback を使います。
- token は `/opt/homebrew/var/lib/expenses-haya-homes/app/.env` の `EXPENSES_WRITE_TOKEN` を source して使います。値を表示・記録・コミットしてはいけません。
- API payload の `source` は必ず `kawasaki` にします。
- `items` は必ず入れます。読み取れた商品行は、袋・値引き・対象外品も含めて1行ずつ残します。
- レシート合計は `totalAmount`、読み取れた税額は `taxAmount` に入れます。税額が明記されていない場合は無理に推定しません。
- レシート全体の `category` は主用途で決めます。食品・食材中心なら `食費`、日用品のみなら `日用品`、混在なら主用途を選び、各 item の `category` で分けます。
- レジ袋、送料、手数料、在庫管理しない対象外品は item として残し、`category` は `その他` または実態に近いカテゴリにします。
- `paymentMethod` は読めたときだけ具体化します。読めない場合は `未指定`。
- `memo` には `home-food` の `event_id`、`source_ref`、不確実な点を短く入れます。
- `rawText` には OCR 全文または `data/sources/catalog.yml` の extracted summary / raw location を入れ、後から照合できるようにします。
- API 登録に失敗した場合や `EXPENSES_WRITE_TOKEN` が見つからない場合は、黙って終わらず、回答で「home-food は更新済みだが expenses 登録は未完了」と明示します。

例:

```bash
set -a
. /opt/homebrew/var/lib/expenses-haya-homes/app/.env
set +a

curl -fsS -X POST http://127.0.0.1:8092/api/receipts \
  -H "Authorization: Bearer ${EXPENSES_WRITE_TOKEN:?missing}" \
  -H "Content-Type: application/json" \
  -d @- <<JSON
{
  "purchasedAt": "2026-06-28",
  "storeName": "肉のハナマサ 錦糸町店",
  "category": "食費",
  "totalAmount": 939,
  "taxAmount": 69,
  "paymentMethod": "クレジットカード",
  "payer": "家計",
  "source": "kawasaki",
  "memo": "home-food event_id=2026-06-28T13:34:00+09:00-purchase-hanamasa-receipt-001; 期限未確認",
  "rawText": "source_ref=src-20260628-line-hanamasa-receipt-001; extracted_summary=...",
  "items": [
    { "name": "バナナチップス", "amount": 267, "quantity": 1, "category": "食費", "memo": "常温在庫に追加" },
    { "name": "ミートボールセット(小)", "amount": 598, "quantity": 1, "category": "食費", "memo": "冷蔵在庫に追加。内容量・期限未確認" },
    { "name": "スーパーバッグ 特大", "amount": 5, "quantity": 1, "category": "その他", "memo": "在庫対象外" }
  ]
}
JSON
```

### 冷蔵庫・冷凍庫・棚の画像

1. 見えるもの、見えないが推測できるもの、読めないものを分けます。
2. 前回スナップショットと比較し、追加・減少・消滅・状態変化を記録します。
3. 画像だけでは量を断定しにくいため、`confidence` を控えめにします。
4. ラベルや期限が読めた場合だけ、期限を確定値として更新します。
5. `inventory_observation` イベントを追記し、現在在庫を補正します。

### 食事報告・料理写真

1. 食べた日時、料理名、使った食材、残り物の発生を記録します。
2. 使用量が不明なら「少量」「半分程度」などで推定し、`confidence` を下げます。
3. 家の在庫を使った可能性が高い場合は `consumption` として在庫を減らします。
4. 外食や購入惣菜は、健康・食費の文脈では記録しますが、在庫消費には無理に反映しません。

### きょうの料理・外部レシピ

1. レシピ名、URL、主な材料、分量、調理時間、栄養や費用に関わるメモを記録します。
2. 最新性や正確性が必要な場合は外部情報を確認します。
3. 在庫と照合し、足りない材料と代替可能な材料を `data/shopping/current.yml` に反映します。
4. レシピを提案に使ったら、回答で根拠と足りない食材を明示します。

### ユーザー訂正

ユーザーの明示的な訂正は最優先です。過去の推定を直接消さず、`correction` イベントを追記してから現在在庫を修正します。

## 期限推定

ラベルやユーザー発言がある場合はそれを優先します。ない場合は一般的な目安として扱い、必ず `expiry_basis: default_estimate` にします。

- 生魚・刺身: 当日から翌日を優先
- 生肉: 冷蔵なら1から2日、冷凍なら数週間目安
- ひき肉: 生肉より短めに扱う
- 葉物野菜: 2から4日を目安
- きのこ: 3から5日を目安
- もやし: 1から2日を目安
- 根菜: 状態次第で1から3週間
- 開封済み乳製品・豆腐・納豆: ラベル優先、なければ早め
- 作り置き・残り物: 2から3日を目安

安全に関わる疑いがある場合は、食べ切り提案より廃棄・確認を優先します。

## 回答方針

ユーザーが質問したら、次の順で答えます。

1. 結論  
   例: 「今日使うなら、鶏むね肉と小松菜を優先してください。」

2. 根拠  
   期限、前回確認日、残量、健康・食費の観点を短く示します。

3. 具体的な行動  
   献立、買うもの、使い切り順、下処理、保存変更など。

4. 不確実点  
   画像で読めない、量が推定、期限未確認など。

5. 更新内容  
   新しい入力があった場合は、どの台帳を更新したかを簡潔に伝えます。

## 献立提案の優先順位

1. 期限が近い、または状態が悪くなりそうな食材
2. すでに開封済みの食材
3. 直近で食べていない栄養カテゴリ
4. 追加購入が少ない料理
5. 食費を抑えられる料理
6. ユーザーの好み、調理時間、体調、予定に合う料理

提案では、原則として「家にあるもので作れるもの」を先に出し、買い足しが必要なものは少なくします。

## 買い物提案の優先順位

1. 主食・たんぱく質・野菜など、食生活の基礎が不足しているもの
2. 今日または明日の献立に必要なもの
3. 安いときに買う価値がある常備品
4. 期限が近い食材を使い切るために必要な補助食材
5. 嗜好品や余裕があれば買うもの

すでに在庫があるものを買わせないよう、必ず `data/inventory/current.yml` と照合します。

## 健康・食費の扱い

- 医療判断はしません。アレルギー、服薬、疾患管理に関わる内容はユーザー確認や専門家相談を促します。
- 健康提案は、野菜量、たんぱく質、塩分、脂質、食物繊維、発酵食品、食事の偏りを中心にします。
- 食費提案は、在庫消費、重複購入防止、冷凍保存、作り置き、安価なたんぱく源を中心にします。
- ユーザーの設定が未入力なら、`data/profile.yml` の不足項目を更新候補として扱います。

## 更新時の手順

1. 原本を保存し、`source` を作る  
   画像/PDF の原本は先に `data/inbox/YYYY-MM-DD/` へコピーする（「受信した原本の保持」を参照）。そのうえで `data/sources/catalog.yml` に情報源を追加し、`raw_location` にコピー後の **workspace 相対パス** を入れます（inbound の絶対パスは書かない）。原本が無い入力は `raw_location: null`。

2. イベントを追記する  
   `data/events/events.jsonl` に1行JSONで追加します。

3. 現在在庫を更新する  
   `data/inventory/current.yml` の数量、場所、期限、優先度、最終確認日を更新します。

4. 食事・買い物リストを更新する  
   必要に応じて `data/meals/log.yml` と `data/shopping/current.yml` を更新します。

5. 回答する  
   更新した内容、現在の判断、次に必要な情報を短く伝えます。

## 判断に迷ったとき

- ユーザーの明示情報を最優先します。
- ラベル・レシート・画像・過去ログの順に根拠を比較します。
- 矛盾がある場合は、古い情報を消さずに訂正イベントを残します。
- 食品安全に関わる場合は、楽観的に解釈しません。
- 量の推定は控えめにし、買い物提案では不足リスクを明示します。
