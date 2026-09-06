# connpass API v2リファレンス

## 共通仕様

- ベースURL：`https://connpass.com`
- 認証：すべてのエンドポイントで `X-API-Key` ヘッダーが必須
- アクセス制限：APIキーごとに1秒間に1リクエストまで
- ページネーション：
  - `start`：取得開始位置。既定値は1
  - `count`：取得件数。既定値は10、最大100
- 複数値：同じパラメーターを繰り返すか、カンマ区切りで指定する
- 一覧レスポンス：`results_returned`、`results_available`、`results_start` と対象リソースの配列を含む

## URLから検索条件への変換

- イベントURL `https://example.connpass.com/event/364/`：`event_id=364`
- グループURL `https://example.connpass.com/`：`subdomain=example`
- グループURL `https://connpass.com/series/1/`：`group_id=1`
- ユーザーURL `https://connpass.com/user/example/`：`nickname=example`

`group_id` はイベント一覧の検索条件として使用する。グループ一覧APIは `subdomain` だけを検索条件として受け付けるため、グループIDだけからグループの全項目を取得できるものとして扱わない。

## エンドポイント

### イベント一覧

`GET /api/v2/events/`

イベントを検索する。

主な検索パラメーター：

- `event_id`：イベントID
- `keyword`：タイトル、キャッチ、概要、住所をAND条件で部分一致検索
- `keyword_or`：タイトル、キャッチ、概要、住所をOR条件で部分一致検索
- `ym`：開催年月。`yyyymm`形式
- `ymd`：開催年月日。`yyyymmdd`形式
- `publish_ym`：公開年月。`yyyymm`形式
- `publish_ymd`：公開年月日。`yyyymmdd`形式
- `nickname`：指定ユーザーが参加しているイベント
- `owner_nickname`：指定ユーザーが管理しているイベント
- `group_id`：グループID
- `subdomain`：グループのサブドメイン
- `prefecture`：都道府県を表す英字値。オンラインは `online`
- `order`：`1`は更新日時順、`2`は開催日時順、`3`は新着順
- `start`、`count`：ページネーション

イベントURLが `https://example.connpass.com/event/364/` の場合、イベントIDは `364`。

主なレスポンス項目：

- 基本情報：`id`、`title`、`catch`、`description`、`url`、`image_url`
- 開催情報：`started_at`、`ended_at`、`published_at`、`address`、`place`、`lat`、`lon`
- 運営情報：`group`、`owner_id`、`owner_nickname`、`owner_display_name`
- 参加情報：`limit`、`accepted`、`waiting`、`event_type`、`open_status`
- その他：`hash_tag`、`updated_at`

日時はISO 8601形式で返る。レスポンスに含まれるタイムゾーンを保持し、変換が必要な場合だけユーザーが指定したタイムゾーンへ変換する。

### イベント資料

`GET /api/v2/events/{id}/presentations/`

指定イベントに投稿された資料を取得する。`id` はイベントID。`start` と `count` を指定できる。

主なレスポンス項目：

- `user`：資料の投稿者
- `url`：資料URL
- `name`：資料タイトル
- `presenter`：発表者
- `presentation_type`：`slide`、`movie`、`blog`
- `created_at`：投稿日

### グループ一覧

`GET /api/v2/groups/`

`subdomain` でグループを検索する。最大100個のサブドメインを指定できる。`start` と `count` を指定できる。

主なレスポンス項目：

- `id`、`subdomain`、`title`、`sub_title`、`url`
- `description`、`owner_text`
- `website_url`、`website_name`
- `twitter_username`、`facebook_url`
- `member_users_count`、`image_url`

### ユーザー一覧

`GET /api/v2/users/`

`nickname` でユーザーを検索する。最大100人を指定できる。`start` と `count` を指定できる。

主なレスポンス項目：

- `id`、`nickname`、`display_name`、`description`、`url`
- `created_at`、`image_url`
- `attended_event_count`、`organize_event_count`
- `presenter_event_count`、`bookmark_event_count`

### ユーザーに関連する情報

- 所属グループ：`GET /api/v2/users/{nickname}/groups/`
- 参加イベント：`GET /api/v2/users/{nickname}/attended_events/`
- 発表イベント：`GET /api/v2/users/{nickname}/presenter_events/`

いずれも `nickname` をパスに指定し、`start` と `count` でページネーションする。

## 制約

- APIで提供されていないconnpassページへのクロール、スクレイピング、その他の自動アクセスは禁止されている。
- `image_url` は一定時間で失効する。外部サイトから直接参照しない。
- APIキーを紛失した場合や漏えいの可能性がある場合は、connpassへ再発行を依頼する。

## 公式資料

- [connpass API v2](https://connpass.com/about/api/v2/)
- [connpass APIの利用案内](https://help.connpass.com/api/)
