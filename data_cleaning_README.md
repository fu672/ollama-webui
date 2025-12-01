# 会話データ加工・クリーニングスクリプト（ver2）

## 概要
本スクリプトは、CSV形式の会話ログデータを読み込み、 
個人情報（氏名・メールアドレス・電話番号・URL・署名文等）の削除、 
および Parent Id 単位での会話整理（user → assistant 形式への再構成）を行い、 
RAG・LLM 学習に適した JSON 形式で出力するためのツールです。

データクレンジングと会話単位での整理を自動化し、 
内部文書・問い合わせ対応ログ・FAQナレッジデータベースの構築などに利用できます。

## 主な処理内容
1. CSVファイルを読み込む
2. 必要な列のみを抽出（存在しない列は空列として補完）
3. Conversation Type が REPLY / REQFORWARD の行を削除
4. Parent Id 単位で氏名を抽出し、該当グループ内 Description を一括 [人名] へ置換
5. メールアドレス、電話番号、URL、署名文、返信禁止文言などの削除
6. 「部署 + 氏名 + です／と申します」形式の氏名も自動匿名化
7. 連続する同じ役割（user / assistant）のテキストを結合
8. user → assistant のメッセージペアのみを抽出（片方のみの会話は除外）
9. [人名][人名] のような重複マーカーを正規化
10. 最終的な JSON を UTF-8 (BOMなし) で出力

## コード上で変更して使用する主な箇所
| 項目 | 設定箇所 | 説明 |
|---|---|---|
| 使用する列 | USE_COLS | 入力CSVから抽出する列を指定 |
| 除外する Conversation Type | DROP_CTYPES | 処理対象外とするタイプ |
| ロールマッピング | ROLE_MAP | Conversation Type → user/assistant への変換 |
| 氏名抽出列 | A_col3 / A_col4 | データに応じて変更可 |
| 分類情報 | A_col5 / A_col6 | JSON出力の category / subcategory に使用 |
| テキストクレンジング規則 | clean_text() 及び各 *_RE 正規表現 | 社内テンプレに応じて調整可能 |

## 実行方法
python data_cleaning.py <入力CSVファイルパス> <出力JSONファイルパス>

例：
python data_cleaning.py ./input/input.csv ./output/output.json

## 入出力仕様
### 入力（CSV 必須列）
Parent Id / Conversation Type / Subject / Description / A_col5 / A_col6 / A_col7

### 出力（JSON形式）
[
  {
    "parent_id": "xxxxx",
    "category": "カテゴリ名",
    "subcategory": "サブカテゴリ名",
    "messages": [
      {"role": "user", "content": "ユーザー発話"},
      {"role": "assistant", "content": "応答文"}
    ]
  }
]

## 依存ライブラリ
Python 3.7+ 
pandas 
pip install pandas

## 注意事項
- user → assistant のペアが存在しない会話は出力されません
- 氏名匿名化は Parent Id 内スコープのみで実施
- 社内特有の署名文などがある場合は clean_text() 内に追加してください

## 更新履歴
2025-11-07 : 初版公開 

## 実行ファイルの場所
~/jupyter-analysis/work/scripts/data_cleaning.py
