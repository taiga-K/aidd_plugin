# AI駆動開発勉強会 Agent Plugin

AI駆動開発勉強会の運営業務を効率化し、継続的に改善するための Agent Plugin です。

> [!NOTE]
> 本プロジェクトは初期開発中です。現在、プラグイン本体と利用可能な機能はまだ公開していません。

## 概要

イベントの企画から開催後のフォローまで、勉強会運営で繰り返し発生する作業を AI エージェントから再利用できる形に整理します。

特定の AI ベンダーに依存しないよう、プラグインのパッケージ形式には [Agent Plugins 1.0.0](https://agent-plugins.org/specification) を採用します。

## リポジトリ構成

```text
.
├── README.md
└── aidd_plugin/                      # プラグインルート
    ├── plugin.json                   # 必須: マニフェスト
    ├── skills/                       # 任意: Agent Skills
    │   └── <skill-name>/
    │       ├── SKILL.md
    │       ├── scripts/              # 任意
    │       ├── references/           # 任意
    │       └── assets/               # 任意
    ├── mcp.json                      # 任意: MCP サーバー定義
    └── <reverse-domain-namespace>/   # 任意: クライアント固有拡張
```

`aidd_plugin/` を Agent Plugin のルートとし、Agent Plugins 1.0.0 が定める固定パスに各コンポーネントを配置します。

## はじめ方

開発に参加する場合は、リポジトリをクローンしてください。

```bash
git clone https://github.com/taiga-K/AIDD_Plugin.git
cd AIDD_Plugin
```

