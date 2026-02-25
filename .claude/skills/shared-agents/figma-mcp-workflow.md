# Figma MCPツール実行手順

**このPhaseはオーケストレータのみが実行する。**

以下の順で呼び出す（すべて必須）:

1. `get_screenshot` — デザイン確認
2. `get_design_context` — コードとアセットURL取得
3. `get_variable_defs` — 色・スペーシング変数取得

すべてのツール呼び出しで指定:

```
clientLanguages: "html,scss,javascript"
clientFrameworks: "ejs,gulp"
```

`get_design_context` には追加で必須指定:

```
dirForAssetWrites: "[ワークスペース絶対パス]/src/images/common"
```

MCPが利用できない場合は**処理を中止してユーザーに報告する**。推測で実装しない。

出力サイズ超過でメタデータのみ返る場合:
1. `get_metadata` で子ノードIDを取得
2. 子ノードごとに `get_design_context` を個別に呼び出す
