---
name: aidd-setup
description: "AI駆動開発勉強会（AIDD）Agent Pluginの構成、スキル、ロゴアセット、参照ファイル、connpass API認証を確認し、不足と次の対応を報告する。`/aidd-setup`、AIDDプラグインの初期設定、導入確認、またはconnpass APIの接続確認を依頼されたときに使用する。"
---

# AIDD setup

AIDD Agent Pluginを利用できる状態か確認し、不足している設定を特定する。再実行しても既存の正常な設定を変更しない。

## 手順

### 1. プラグイン構成を確認する

プラグインルートで次を確認する。

- `plugin.json` が存在し、Agent Plugins 1.0.0のマニフェスト仕様に適合する
- `skills/aidd-mode/SKILL.md`
- `skills/aidd-setup/SKILL.md`
- `skills/connpass-api/SKILL.md`
- `skills/aidd-logo-management/SKILL.md`
- `skills/create-event-slides/SKILL.md`

マニフェストはJSON構文だけでなく、[Agent Plugins 1.0.0仕様](https://agent-plugins.org/specification)に従って次を検証する。

- `$schema` が `https://agent-plugins.org/schemas/1.0.0/plugin.schema.json` と一致する
- `name` が存在し、1〜64文字で、小文字英数字、ハイフン、ピリオドだけを使用する。先頭と末尾は英数字とし、`--` と `..` を含めない
- `$schema`、`name`、`version`、`description`、`author`、`homepage`、`repository`、`license`、`keywords`、`extensions` 以外のトップレベルフィールドがなく、各フィールドの型が仕様に適合する

`plugin.json` がない場合または仕様に適合しない場合は、Agent Pluginとしての配布を妨げる項目として報告する。必要なメタデータを推測して作成しない。

### 2. スキルの依存ファイルを確認する

- **aidd-mode** に記載された次のplaybookがすべて存在し、リンク先を開ける
  - `playbooks/connpass-query.md`
  - `playbooks/logo-selection.md`
  - `playbooks/event-slides.md`
  - `playbooks/prepare-event-materials.md`
- **connpass-api** が指定する参照ファイルを開ける
- **aidd-logo-management** に記載されたすべてのロゴパスが存在する
- **create-event-slides** が指定するルールと完了チェックリストを開ける

すべてのplaybookを確認できた場合だけAIDD modeを `準備完了` とする。不足しているファイルを別のファイルで代用しない。

### 3. connpass API認証を確認する

環境変数の値を表示せず、設定の有無だけを確認する。

```bash
if [[ -n "${CONNPASS_API_KEY:-}" ]]; then
  printf 'CONNPASS_API_KEY: configured\n'
else
  printf 'CONNPASS_API_KEY: missing\n'
fi
```

APIキーが未設定の場合は、[connpass APIの利用案内](https://help.connpass.com/api/)から取得し、ユーザー自身が実行環境の `CONNPASS_API_KEY` に設定するよう案内する。値を会話へ貼り付けるよう求めない。APIキーを会話、スキル、ルール、設定例、またはリポジトリ内のファイルへ書かない。

設定済みの場合は **connpass-api** の `SKILL.md` と、そこから参照されるAPI v2リファレンスを全文読む。そのリクエストルールに従い、`subdomain=aid` でグループ情報を1件取得する。次の値と一致すれば接続済みとする。

- グループID：`14543`
- サブドメイン：`aid`
- URL：`https://aid.connpass.com/`

認証エラーやアクセス制限は **connpass-api** の手順に従って処理する。

### 4. 結果を報告する

次の項目を `準備完了`、`要対応`、`未確認` のいずれかで報告する。

- プラグインマニフェスト
- AIDD mode
- connpass API
- ロゴアセット
- スライド作成ルール

- `準備完了`：対象項目の確認がすべて成功した
- `要対応`：ファイルや設定の欠落、マニフェスト不正、APIキー未設定、`401 Unauthorized`、`403 Forbidden`、`429 Too Many Requests` を除くその他の4xx応答、またはAIDDグループ情報の不一致がある
- `未確認`：前提項目の失敗、ネットワーク障害、5xx応答、再試行上限に達した `429 Too Many Requests`、不正なJSON、または必須項目を欠くAPIレスポンスにより確認できない

`要対応` と `未確認` には、確認できた原因と次に必要な操作を1つずつ添える。

## 制約

- APIキーの値を読み上げたり記録したりしない。
- 接続確認では読み取り専用のAPIだけを使用する。
- 未承認の外部サービス、MCPサーバー、テンプレートを追加しない。
- 確認を通すために不足ファイルや設定値を捏造しない。
