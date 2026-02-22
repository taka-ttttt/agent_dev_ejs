# EJSコーディング規約

## ディレクトリ構造

```
src/ejs/
├── top/           # トップページ用セクション
│   ├── _case.ejs
│   ├── _hero.ejs
│   └── _news.ejs
├── service/       # サービスページ用セクション
│   ├── index.ejs  # サービスページファイル
│   └── _service-introduction.ejs
├── about/         # アバウトページ用セクション
│   ├── index.ejs  # アバウトページファイル
│   ├── _vision.ejs
│   └── _history.ejs
├── common/        # 全ページ共通（ヘッダー、フッター等）
│   ├── _head.ejs
│   ├── _header.ejs
│   └── _footer.ejs
├── component/     # 再利用コンポーネント
│   ├── _picture.ejs
│   └── _button.ejs
├── pageData/      # JSONデータ
│   └── pageData.json
└── index.ejs      # トップページファイル
```

## ファイル命名

### パーシャルファイル

先頭に `_` を付ける（例: `_header.ejs`, `_footer.ejs`, `_hero.ejs`）

### ページファイル

- **トップページ**: `src/ejs/index.ejs`
- **下層ページ**: 各ページフォルダ内に `index.ejs` を作成
  - 例: `src/ejs/service/index.ejs`, `src/ejs/about/index.ejs`

### セクションファイルの配置ルール

**セクションのパーシャルファイルは必ず該当ページのフォルダ内に配置する。**

- **命名規則**: `[ページ名]/_[セクション名].ejs`
- **例**:
  - トップページのcaseセクション → `top/_case.ejs`
  - サービスページのintroductionセクション → `service/_service-introduction.ejs`
  - アバウトページのvisionセクション → `about/_vision.ejs`

## 変数宣言ルール

### ページファイル

**ページファイルでは `ROOT_PATH` と `PAGE_DATA` のみ宣言する。**

```ejs
<% ROOT_PATH='./'; const PAGE_DATA="top"; %>
```

下層ページの場合は `ROOT_PATH` を適切に設定する。

```ejs
<% ROOT_PATH='../'; const PAGE_DATA="service"; %>
```

### ページファイルでのinclude記述

**ページファイルでは相対パスで直接指定し、必要な変数のみを渡す。**

```ejs
<%- include('./common/_head', { page: json[PAGE_DATA] }) %>
<%- include('./common/_header') %>
<%- include('./top/_case') %>
<%- include('./common/_footer') %>
```

- 相対パスで記述（例: `'./common/_head'`）
- 変数渡しは必要最小限のみ（例: `_head` には `page` のみ、他は省略）

### パーシャルファイルでのinclude記述

**パーシャルファイル内で他のパーシャルをincludeする場合も相対パスを使用する。**

```ejs
<%- include('../component/_picture', {
  file: "top/case01",
  type: "jpg",
  alt: "事例イメージ",
  sp: true,
  loading: "lazy",
  width: "320",
  height: "213",
  spWidth: "320",
  spHeight: "213"
}) %>
```

- パーシャルから component を呼ぶ場合: `'../component/_picture'`
- 同一フォルダ内を呼ぶ場合: `'./_button'`

## 画像の扱い

### 基本方針

**すべての画像は原則として pictureコンポーネント (`_picture.ejs`) を使用する。**

- レスポンシブ対応（PC/SP）
- WebP形式の自動適用
- フォールバック形式の自動生成
- ブレークポイント: 767px（コンポーネント内で設定済み）

### 画像ファイルの配置先

```
src/images/
├── common/       # 共通素材（ロゴ、アイコン等）
├── top/          # トップページ用
└── [ページ名]/   # 各ページ用
```

命名規則: `[セクション名]-[要素名][連番].[拡張子]`
例: `case01.jpg`, `case01_sp.jpg`（SP用）

### pictureコンポーネントの使い方

#### 必須ルール

1. **必ずdivで囲む**
2. **width/height属性を指定する**（推奨、CLS対策）

**注:** `ROOT_PATH` はページファイルで定義されているため、pictureコンポーネント呼び出し時に指定する必要はない（コンポーネント内で自動参照される）。

#### 基本的な記述例（width/height属性あり・推奨）

```ejs
<div class="p-section__img">
  <% // 画像ファイルパス: assets/images/top/mv01.jpg および mv01_sp.jpg %>
  <%- include('../component/_picture', {
    file: "top/mv01",           // 拡張子なし、assets/images/ からの相対パス
    type: "jpg",                // 元の画像形式（jpg, png等）
    alt: "説明文",
    sp: true,                   // SP用画像あり
    loading: "lazy",            // ファーストビュー以外は "lazy"
    width: "1440",              // PC用画像の幅
    height: "810",              // PC用画像の高さ
    spWidth: "375",             // SP用画像の幅
    spHeight: "720"             // SP用画像の高さ
  }) %>
</div>
```

#### width/height属性なしの場合

```ejs
<div class="p-section__img">
  <%- include('../component/_picture', {
    file: "top/mv01",
    type: "jpg",
    alt: "説明文",
    sp: true,
    loading: "lazy"
  }) %>
</div>
```

#### パラメータ一覧

| パラメータ | 必須 | 説明 |
|-----------|------|------|
| `file` | ○ | 画像パス（拡張子なし、`assets/images/` からの相対パス） |
| `type` | ○ | 元の画像形式（jpg, png等） |
| `alt` | ○ | alt属性 |
| `sp` | ○ | SP用画像の有無（true/false）<br>`true`の場合、`[file]_sp.[type]` ファイルが必要 |
| `loading` | - | `"lazy"` または省略<br>ファーストビューの画像は省略 |
| `width` | - | PC用画像の幅（指定推奨、CLS対策） |
| `height` | - | PC用画像の高さ（指定推奨、CLS対策） |
| `spWidth` | - | SP用画像の幅（`sp: true` かつ `width` 指定時のみ有効） |
| `spHeight` | - | SP用画像の高さ（`sp: true` かつ `height` 指定時のみ有効） |

### Figma由来のコーディング時の画像扱い

**実画像が未配置の場合は仮のファイルパスを指定する。**

```ejs
<div class="p-section__img">
  <% // 画像は仮パス。実画像配置後に差し替える %>
  <%- include('../component/_picture', {
    file: "top/case01",         // 仮パス
    type: "jpg",
    alt: "事例イメージ",
    sp: true,
    loading: "lazy",
    width: "1440",
    height: "810",
    spWidth: "375",
    spHeight: "720"
  }) %>
</div>
```

- 仮パスを指定した箇所には**必ずコメントで明記する**
- ビルド時にEJSではエラーにならないため、後から実画像を配置する運用が可能

### 直接imgタグを使用するケース

以下の場合のみ、pictureコンポーネントを使わず直接 `<img>` タグを使用する:

- SVG画像（ロゴ、アイコン等）
- WebP対応が不要な画像

```ejs
<img src="<%= ROOT_PATH %>assets/images/common/logo.svg" alt="ロゴ">
```

**必ず `<%= ROOT_PATH %>` を使用する。**

## 外部リンク

`target="_blank"` には必ず `rel="noopener noreferrer"` を付与。

```html
<a href="https://example.com" target="_blank" rel="noopener noreferrer">
  外部リンク
</a>
```
