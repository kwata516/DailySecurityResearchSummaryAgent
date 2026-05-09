# Daily Security Research Summary Agent

[English](#english) | [日本語](#japanese)

---

<a id="english"></a>

## English

### Overview

A Microsoft Security Copilot AI Agent that automatically collects daily updates from major security research sites and generates a structured daily security research summary report in Japanese.

The list of monitored sites is centrally managed in a dedicated configuration plugin (`DailySecurityResearchConfig`). Adding or removing sites requires only editing the JSON in the config file and reinstalling the plugin — no changes to the agent itself are needed.

### Architecture

```
Security Copilot Agent (DailySecurityResearchSummaryAgent)
├── Config Plugin (DailySecurityResearchConfig)
│   └── GetMonitoredSites          - Returns the list of monitored sites as JSON
├── Fetcher Plugin (DailySecurityResearchFetcher)
│   └── FetchWebPage               - Fetches any RSS/Atom feed or web page via Jina.ai Reader
└── Agent Orchestration
    ├── Step 0: Load site config from GetMonitoredSites
    ├── Step 1: Determine today's UTC date
    ├── Step 2: Parallel fetch all enabled RSS sites via FetchWebPage
    ├── Step 3: Extract today's entries from feed text
    ├── Step 4: Evaluate severity of each item
    └── Step 5: Generate structured Japanese summary report
```

### Monitored Sites (default: 12 sources)

| # | Site | Method |
|---|------|--------|
| 1 | Microsoft Security Response Center Blog | FetchWebPage (Jina.ai) |
| 2 | Google Project Zero | FetchWebPage (Jina.ai) |
| 3 | Krebs on Security | FetchWebPage (Jina.ai) |
| 4 | SANS Internet Storm Center | FetchWebPage (Jina.ai) |
| 5 | SANS Newsbites | FetchWebPage (Jina.ai) ⚠️ |
| 6 | Unit 42 (Palo Alto Networks) | FetchWebPage (Jina.ai) |
| 7 | Secureworks CTU Research | FetchWebPage (Jina.ai) |
| 8 | CrowdStrike Blog | FetchWebPage (Jina.ai) |
| 9 | The Hacker News | FetchWebPage (Jina.ai) |
| 10 | Dark Reading | FetchWebPage (Jina.ai) |
| 11 | ScanNetSecurity | FetchWebPage (Jina.ai) ⚠️ |
| 12 | Security Memo (kjm) | FetchWebPage (Jina.ai) ⚠️ |

### Features

- **Daily automated execution** — runs every 24 hours via scheduled trigger
- **Parallel data collection** — fetches all sources simultaneously for fast execution
- **Site config externalized** — add/remove sites by editing a single JSON config file
- **Severity classification** — 🔴 Critical / 🟠 High / 🟡 Medium / 🔵 Info
- **Server-side compatible fetching** — uses Jina.ai Reader (no API key required, works from Azure servers)
- **Japanese report output** — structured report with highlights, site sections, and recommended actions

### Files

| File | Description |
|------|-------------|
| `DailySecurityResearchConfig_ja.yaml` | **★ Config plugin** — defines the monitored site list as JSON (edit this to add/remove sites) |
| `DailySecurityResearchFetcher_ja.yaml` | Data fetcher API plugin (single SkillGroup using combined OpenAPI spec) |
| `openapi_combined.yaml` | OpenAPI spec for **FetchWebPage** (Jina.ai Reader) |
| ~~`openapi_rss2json.yaml`~~ | Superseded by `openapi_combined.yaml` |
| ~~`openapi_cisa_kev.yaml`~~ | Superseded by `openapi_combined.yaml` |
| ~~`openapi_nvd_cves.yaml`~~ | Superseded by `openapi_combined.yaml` |
| `DailySecurityResearchSummaryAgent_ja.yaml` | AI Agent plugin (scheduled orchestration) |
| [DailySecurityResearchSummaryAgent_ja_card.html](DailySecurityResearchSummaryAgent_ja_card.html) | Plugin Card — visual reference of all plugins, skills, and workflow |

### Prerequisites

- Microsoft Security Copilot license
- A publicly hosted URL for `openapi_combined.yaml` (GitHub raw URL recommended)

> **No Azure resources required.** The agent uses only [Jina.ai Reader](https://r.jina.ai) (free, no API key needed) — no Logic Apps or Azure Functions needed.

### Quick Start

#### 1. Host the OpenAPI specs

Push this repository to a **public GitHub repository**, then update `DailySecurityResearchFetcher_ja.yaml` with the actual raw URL:

```yaml
# Replace <YOUR_ORG> and <YOUR_REPO> in DailySecurityResearchFetcher_ja.yaml
OpenApiSpecUrl: https://raw.githubusercontent.com/<YOUR_ORG>/<YOUR_REPO>/main/DailySecurityResearchSummaryAgent/openapi_combined.yaml
```

Alternatively, host the spec on Azure Blob Storage (static website) or any HTTPS-accessible public URL.

#### 2. Verify RSS URLs for ⚠️ unconfirmed sites

Before installing, confirm the RSS URLs for these 3 sites in your browser:

| Site | Assumed RSS URL | How to verify |
|------|----------------|---------------|
| SANS Newsbites | `https://www.sans.org/newsletters/newsbites/rss` | Visit https://www.sans.org/newsletters/newsbites |
| ScanNetSecurity | `https://scan.netsecurity.ne.jp/article/rss/` | Visit https://scan.netsecurity.ne.jp/ |
| Security Memo (kjm) | `https://www.st.ryukoku.ac.jp/~kjm/security/memo/index.rdf` | Visit https://www.st.ryukoku.ac.jp/~kjm/security/memo/ |

Update the `url` field in `DailySecurityResearchConfig_ja.yaml` if the URL differs.

#### 3. Install plugins in order

Upload to Security Copilot in this order:

1. `DailySecurityResearchConfig_ja.yaml`
2. `DailySecurityResearchFetcher_ja.yaml`
3. `DailySecurityResearchSummaryAgent_ja.yaml`

#### 4. Run the agent

**Manual execution:**
```
今日のセキュリティリサーチサマリーレポートを生成してください
```

**Scheduled execution:** The agent runs automatically every 24 hours via the `DailyReport` trigger.

### Report Output Format

```
🛡️ 日次セキュリティリサーチ・サマリー [YYYY年MM月DD日]
├── ⚡ 重要・緊急ハイライト（🔴 緊急 / 🟠 高 のアイテム、最大 10 件）
├── 📰 サイト別最新情報（カテゴリ別: government / vendor / news / blog / ja）
├── 🎯 本日の推奨アクション（優先度順、最大 5 件）
└── 📊 調査統計（fetch statistics）
```

### Customizing the Site List

Edit `DailySecurityResearchConfig_ja.yaml` — the Template section contains the full JSON site list:

```json
// Add a new RSS site
{
  "name": "New Site Name",
  "url": "https://example.com/rss",
  "category": "vendor",   // government | vendor | news | blog | ja
  "enabled": true
}
```

To **disable** a site without removing it, set `"enabled": false`.

After editing, reinstall only `DailySecurityResearchConfig_ja.yaml` — no changes to the fetcher or agent are needed.

### Agent Parameters

| Parameter | Required | Default | Description |
|-----------|----------|---------|-------------|
| `ReportDate` | No | UTC today | Target date in YYYY-MM-DD format (e.g. `2026-05-09`) |
| `OutputLanguage` | No | 日本語 | Output language (e.g. `Japanese`, `English`) |

### License

MIT License — see [LICENSE](../LICENSE) for details.

---

<a id="japanese"></a>

## 日本語

### 概要

主要セキュリティリサーチャサイトの本日更新コンテンツを毎日自動収集し、日本語の日次セキュリティリサーチ・サマリーレポートを生成する Microsoft Security Copilot AIエージェントです。

監視対象サイトは専用の設定プラグイン（`DailySecurityResearchConfig`）で一元管理します。サイトの追加・削除は設定ファイルの JSON を編集して再インストールするだけで反映されます。エージェント本体の変更は不要です。

### アーキテクチャ

```
Security Copilot Agent (DailySecurityResearchSummaryAgent)
├── 設定プラグイン (DailySecurityResearchConfig)
│   └── GetMonitoredSites          - 監視対象サイトリストを JSON で返す
├── データ取得プラグイン (DailySecurityResearchFetcher)
│   └── FetchWebPage               - Jina.ai Reader 経由で任意の RSS/Atom フィードまたは Web ページをテキストで取得
└── Agent オーケストレーション
    ├── Step 0: GetMonitoredSites でサイト設定を読み込み
    ├── Step 1: UTC 本日日付を決定
    ├── Step 2: 有効な全 RSS サイトを FetchWebPage で並列フェッチ
    ├── Step 3: フィードテキストから当日分エントリを抽出
    ├── Step 4: 各アイテムの重要度を評価
    └── Step 5: 構造化された日本語サマリーレポートを生成
```

### 監視対象サイト（デフォルト: 12 ソース）

| # | サイト | 取得方式 |
|---|-------|--------|
| 1 | Microsoft Security Response Center Blog | FetchWebPage (Jina.ai) |
| 2 | Google Project Zero | FetchWebPage (Jina.ai) |
| 3 | Krebs on Security | FetchWebPage (Jina.ai) |
| 4 | SANS Internet Storm Center | FetchWebPage (Jina.ai) |
| 5 | SANS Newsbites | FetchWebPage (Jina.ai) ⚠️ |
| 6 | Unit 42 (Palo Alto Networks) | FetchWebPage (Jina.ai) |
| 7 | Secureworks CTU Research | FetchWebPage (Jina.ai) |
| 8 | CrowdStrike Blog | FetchWebPage (Jina.ai) |
| 9 | The Hacker News | FetchWebPage (Jina.ai) |
| 10 | Dark Reading | FetchWebPage (Jina.ai) |
| 11 | ScanNetSecurity | FetchWebPage (Jina.ai) ⚠️ |
| 12 | Security Memo (kjm) | FetchWebPage (Jina.ai) ⚠️ |

### 機能

- **毎日自動実行** — スケジュールトリガーで 24 時間ごとに起動
- **並列データ収集** — 全ソースを同時並列フェッチして高速処理
- **サイト設定の外部化** — 設定 JSON を編集するだけでサイト追加・削除が可能
- **重要度分類** — 🔴 緊急 / 🟠 高 / 🟡 中 / 🔵 情報
- **サーバーサイド対応フェッチ** — Jina.ai Reader 使用（API Key 不要、Azure サーバーからアクセス可能）
- **日本語レポート出力** — 重要ハイライト・サイト別情報・推奨アクションを含む構造化レポート

### ファイル構成

| ファイル | 説明 |
|---------|------|
| `DailySecurityResearchConfig_ja.yaml` | **★ 設定プラグイン** — 監視対象サイトリストを JSON で管理（サイト追加・削除はここを編集） |
| `DailySecurityResearchFetcher_ja.yaml` | データ取得 API プラグイン（単一 SkillGroup、統合 OpenAPI spec 使用） |
| `openapi_combined.yaml` | **FetchWebPage** (Jina.ai Reader) の OpenAPI spec |
| ~~`openapi_rss2json.yaml`~~ | `openapi_combined.yaml` に統合済み（参照不要） |
| ~~`openapi_cisa_kev.yaml`~~ | `openapi_combined.yaml` に統合済み（参照不要） |
| ~~`openapi_nvd_cves.yaml`~~ | `openapi_combined.yaml` に統合済み（参照不要） |
| `DailySecurityResearchSummaryAgent_ja.yaml` | AI エージェントプラグイン（スケジュール・オーケストレーション） |
| [DailySecurityResearchSummaryAgent_ja_card.html](DailySecurityResearchSummaryAgent_ja_card.html) | Plugin Card — 全プラグイン・スキル・ワークフローのビジュアル参照 |

### 前提条件

- Microsoft Security Copilot ライセンス
- `openapi_combined.yaml` を公開ホスティングできる URL（GitHub raw URL 推奨）

> **Azure リソース不要。** [Jina.ai Reader](https://r.jina.ai) のみ使用（無料・API Key 不要）。Logic App や Azure Functions は不要です。

### クイックスタート

#### 1. OpenAPI spec をホスティング

このリポジトリを **公開 GitHub リポジトリ** に push し、`DailySecurityResearchFetcher_ja.yaml` の URL を実際の値に更新します:

```yaml
# DailySecurityResearchFetcher_ja.yaml 内の <YOUR_ORG> と <YOUR_REPO> を置換
OpenApiSpecUrl: https://raw.githubusercontent.com/<YOUR_ORG>/<YOUR_REPO>/main/DailySecurityResearchSummaryAgent/openapi_combined.yaml
```

代替ホスティング: Azure Blob Storage（静的ウェブサイト）/ GitHub Pages / その他 HTTPS アクセス可能な公開 URL

#### 2. ⚠️ 要確認の RSS URL を検証

インストール前に以下の 3 サイトの RSS URL をブラウザで確認してください:

| サイト | 想定 RSS URL | 確認方法 |
|-------|-------------|--------|
| SANS Newsbites | `https://www.sans.org/newsletters/newsbites/rss` | https://www.sans.org/newsletters/newsbites にアクセス |
| ScanNetSecurity | `https://scan.netsecurity.ne.jp/article/rss/` | https://scan.netsecurity.ne.jp/ にアクセス |
| Security Memo (kjm) | `https://www.st.ryukoku.ac.jp/~kjm/security/memo/index.rdf` | https://www.st.ryukoku.ac.jp/~kjm/security/memo/ にアクセス |

URL が異なる場合は `DailySecurityResearchConfig_ja.yaml` の `url` フィールドを更新してください。

#### 3. プラグインをこの順番でインストール

Security Copilot に以下の順でアップロードします:

1. `DailySecurityResearchConfig_ja.yaml`
2. `DailySecurityResearchFetcher_ja.yaml`
3. `DailySecurityResearchSummaryAgent_ja.yaml`

#### 4. エージェントを実行

**手動実行:**
```
今日のセキュリティリサーチサマリーレポートを生成してください
```

**スケジュール実行:** `DailyReport` トリガーにより 24 時間ごとに自動実行されます。

### レポート出力形式

```
🛡️ 日次セキュリティリサーチ・サマリー [YYYY年MM月DD日]
├── ⚡ 重要・緊急ハイライト（🔴 緊急 / 🟠 高 のアイテム、最大 10 件）
├── 📰 サイト別最新情報（カテゴリ別: government / vendor / news / blog / ja）
├── 🎯 本日の推奨アクション（優先度順、最大 5 件）
└── 📊 調査統計（フェッチ統計）
```

### サイトリストのカスタマイズ

`DailySecurityResearchConfig_ja.yaml` の Template セクション内の JSON を編集します:

```json
// RSS サイトを追加する例
{
  "name": "新しいサイト名",
  "url": "https://example.com/rss",
  "category": "vendor",   // government | vendor | news | blog | ja
  "enabled": true
}
```

サイトを一時的に無効化するには `"enabled": false` に変更します（URLを残したまま無効化できます）。

編集後は `DailySecurityResearchConfig_ja.yaml` のみを再インストールすれば反映されます。フェッチャーやエージェントの変更は不要です。

### エージェントパラメータ

| パラメータ | 必須 | デフォルト | 説明 |
|-----------|------|---------|------|
| `ReportDate` | いいえ | UTC 本日 | レポート対象日（YYYY-MM-DD 形式、例: `2026-05-09`） |
| `OutputLanguage` | いいえ | 日本語 | 出力言語（例: `日本語`, `English`） |

### カスタマイズ

| 設定 | ファイル | 説明 |
|------|---------|------|
| 実行頻度 | Agent YAML: `DefaultPollPeriodSeconds` | `86400`=1日, `43200`=12時間, `0`=手動のみ |
| 出力言語 | Agent 呼び出し時: `OutputLanguage` パラメータ | 日本語 / English など |
| 監視サイト | `DailySecurityResearchConfig_ja.yaml` の JSON | サイトの追加・削除・無効化 |

### ライセンス

MIT License — 詳細は [LICENSE](../LICENSE) を参照してください。
