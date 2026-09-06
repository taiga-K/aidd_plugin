---
name: connpass-api
description: "connpass API v2を使用して、イベント、発表資料、グループ、ユーザー、参加・登壇履歴を検索・取得する。connpassのイベントURLやID、キーワード、開催日、グループ、ユーザー情報から公式データを取得するときに使用する。"
---

# connpass API

APIを呼び出す前に、[connpass API v2リファレンス](references/api-v2.md)を全文読む。

## 手順

1. 要望から取得対象と検索条件を特定する。
2. connpassのURLが渡された場合は、URLからイベントID、グループID、グループのサブドメイン、またはユーザーのニックネームを取得する。
3. 実行環境に `CONNPASS_API_KEY` が設定されていることを確認する。
4. 必要最小限の検索条件で、対応するAPI v2のエンドポイントへGETリクエストを送る。
5. 全件取得が必要な場合だけページネーションを行う。
6. 要望に必要な項目だけを整理し、取得件数とconnpass上のURLを添えて返す。

## AI駆動開発勉強会

- グループID：`14543`
- サブドメイン：`aid`
- URL：`https://aid.connpass.com/`

AI駆動開発勉強会のイベント検索には `group_id=14543` または `subdomain=aid` を使用する。グループ情報の取得には `subdomain=aid` を使用する。

## リクエストのルール

- APIキーは環境変数 `CONNPASS_API_KEY` から読み、`X-API-Key` ヘッダーに設定する。
- APIキーをコマンドの引数へ直接記載したり、出力や会話に表示したりしない。
- APIキーが設定されていない場合は、ユーザーに環境変数への設定を依頼し、リクエストを実行しない。
- APIリクエスト時はシェルのトレースやcurlの詳細ログを有効にせず、リクエストヘッダーを記録しない。
- API v2だけを使用する。
- リクエストは直列に実行し、各リクエストの間を1秒以上空ける。
- connpassのWebページをクロールまたはスクレイピングしない。
- `401 Unauthorized` の場合は再試行せず、APIキーの確認を依頼する。
- `429 Too Many Requests` の場合は、`Retry-After` ヘッダーがあればその時間だけ待つ。なければ2秒、4秒と待機時間を延ばし、再試行は2回までにする。解消しない場合は停止して報告する。
- 不明な値やAPIが返さない情報を推測で補わない。
- `image_url` は失効するため、外部サイトから直接参照する恒久URLとして扱わない。

## 基本形

```bash
: "${CONNPASS_API_KEY:?CONNPASS_API_KEY is not set}"

curl --config - \
  --fail-with-body --silent --show-error --get \
  'https://connpass.com/api/v2/events/' \
  --data-urlencode 'keyword=AI駆動開発' \
  --data-urlencode 'order=2' \
  --data-urlencode 'count=100' <<EOF
header = "X-API-Key: ${CONNPASS_API_KEY}"
EOF
```

APIキーは標準入力からcurlへ渡す。検索値は `--data-urlencode` で渡し、URLへ文字列を直接連結しない。

## ページネーション

レスポンスの `results_available`、`results_returned`、`results_start` を確認する。`results_returned` が0の場合、または `results_start + results_returned - 1` が `results_available` 以上の場合は終了する。続きが必要な場合は、次の `start` に `results_start + results_returned` を指定する。
