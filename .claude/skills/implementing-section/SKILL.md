---
name: implementing-section
description: FigmaのURLからページセクションのEJSパーシャルとSCSSファイルをFLOCSS/BEM規約で作成する。トップページのヒーローやサービス紹介等、ページ固有のセクションを実装する場合に使用。
argument-hint: [Figma URL] [ページ名] [セクション名]（例: /implementing-section https://figma.com/... top hero）
---

# セクション実装スキル（オーケストレータ）

FigmaのURLからページセクションのEJS/SCSSを作成します。

**あなたはオーケストレータです。**
- Figma MCPツールはこのコンテキスト（オーケストレータ）でのみ使用する
- サブエージェントにはFigmaから取得した情報を「デザインブリーフ」として渡す
- サブエージェントはFigma MCPを使用しない

**引数:** `$ARGUMENTS` — `[Figma URL] [ページ名] [セクション名]`

- 第1引数（必須）: FigmaのURL
- 第2引数（任意）: ページ名（例: top, service, about）
- 第3引数（任意）: セクション名（例: hero, service, about）

---

## Phase 1: 事前確認

### 1-1. ページ名・セクション名の確定

**引数にページ名・セクション名がある場合:**
- 引数の値をそのまま使用し、質問せずに進める

**引数にページ名がない場合:**
- ユーザーに質問する: 「このセクションはどのページに配置しますか？（例: top, service, about など）」

**引数にセクション名がない場合:**
- Figmaのレイヤー名・デザイン内容から推測して決定する（確認不要）

ページ名・セクション名の確定後、以下を決定する:

| 項目 | 値 |
|------|----|
| pageName | `[pageName]` |
| sectionName | `[sectionName]` |
| EJSパス | `src/ejs/[pageName]/_[sectionName].ejs` |
| SCSSパス | `src/sass/object/project/_p-[pageName]-[sectionName].scss` |
| ルートクラス | `.p-[pageName]-[sectionName]` |
| ページindexパス | `src/ejs/[pageName]/index.ejs`（トップは `src/ejs/index.ejs`） |

### 1-2. URLパラメータの抽出

- `fileKey`: `/design/` 直後の文字列
- `nodeId`: `node-id=` の値（`-` を `:` に変換）

### 1-3. 変数定義ファイルの読み込み

以下を Read ツールで読み込む（変数マッピングに使用）:

- `src/sass/global/_setting.scss`

---

## Phase 2: Figma MCPツールの実行

[figma-mcp-workflow.md](../shared-agents/figma-mcp-workflow.md) の手順に従ってFigma情報を取得する。

---

## Phase 3: デザインブリーフの作成

取得した情報を以下の形式でデザインブリーフとして整理する。
これ以降のすべてのサブエージェントに渡す共通情報になる。

```
## デザインブリーフ

### ページ情報
- pageName: [pageName]
- sectionName: [sectionName]
- ejsPath: src/ejs/[pageName]/_[sectionName].ejs
- scssPath: src/sass/object/project/_p-[pageName]-[sectionName].scss
- rootClass: .p-[pageName]-[sectionName]
- pageIndexPath: [ページのindex.ejsパス]

### デザイン情報
[get_design_context の結果を貼り付ける]

### 変数マッピング
_setting.scss と照合した結果:
- [Figma変数名] → [対応する$変数名 または 直接カラーコード]

### 画像アセット
[ダウンロードした画像のパス一覧]
- [画像名]: src/images/[保存先パス]
```

---

## Phase 4: 画像アセットの処理

`get_design_context` は `dirForAssetWrites` に指定した `src/images/common/` へ画像をハッシュ名で**自動保存**する。
このファイルを命名規則に従ったパスへ手動でリネーム・移動する。

**手順:**

1. **Glob** で `src/images/common/` 直下を確認し、追加されたハッシュ名のファイルを特定する
2. `get_screenshot` のデザイン確認でアセットの種類を判断し、保存先を決定する（下表）
3. **Read** でハッシュ名ファイルの内容を読み込み、**Write** で正式パスへ保存する
4. **Delete** でハッシュ名のファイルを削除する

| 条件 | 正式パス |
|------|--------|
| 50px以下のアイコン・複数ページ共通 | `src/images/common/icons/` （`src/images/common/` 内でリネームのみ） |
| ページ固有の画像・装飾 | `src/images/[pageName]/` （`common/` から移動） |
| ロゴ | `src/images/common/logo/` （`src/images/common/` 内でリネームのみ） |

命名規則: `[sectionName]-[要素名][連番].[拡張子]`（SP版は `_sp` サフィックス）

**図形の誤認識チェック:** スクリーンショットで確認し、不自然な1文字テキストや図形・アイコンと判断されるものはSVGとして扱う。判断が難しい場合はユーザーに確認する。

デザインブリーフの「画像アセット」セクションを正式パスで更新する。

---

## Phase 5: サブエージェントの実行

以下の順番でサブエージェントをTaskツールで起動する。
各エージェントのプロンプトファイルを Read ツールで読み込んでからTaskツールを呼び出すこと。

**プロンプト構成:** `[エージェントファイルの内容]\n\n---\n\n[デザインブリーフ]\n\n---\n\n[追加で渡すファイルの内容]`

### 5-1. Markup Agent

1. `.claude/skills/implementing-section/agents/markup.md` を読み込む
2. 以下を渡して `subagent_type: "general-purpose"` でTaskツールを起動する:
   - エージェントファイルの内容
   - デザインブリーフ
3. 完了後、作成されたEJSファイルを Read ツールで読み込んでおく

### 5-2. Style Agent

1. `.claude/skills/shared-agents/style.md` を読み込む
2. 以下を渡して `subagent_type: "general-purpose"` でTaskツールを起動する:
   - エージェントファイルの内容
   - デザインブリーフ
   - Markup Agentが作成したEJSファイルの内容（クラス名参照用）
3. 完了後、作成されたSCSSファイルを Read ツールで読み込んでおく

### 5-3. Integration Agent

1. `.claude/skills/implementing-section/agents/integration.md` を読み込む
2. 以下を渡して `subagent_type: "general-purpose"` でTaskツールを起動する:
   - エージェントファイルの内容
   - デザインブリーフ
   - 最新のEJSファイルの内容
   - 最新のSCSSファイルの内容

---

## Phase 6: 完了報告

各エージェントの結果をまとめて報告する。

| ファイル | パス | 状態 |
|---------|------|------|
| EJSパーシャル | `src/ejs/[pageName]/_[sectionName].ejs` | 作成 |
| SCSSファイル | `src/sass/object/project/_p-[pageName]-[sectionName].scss` | 作成 |
| 画像 | `src/images/...` | ダウンロード済み |

必要に応じて `/code-review` `/visual-check` を実行してください。
