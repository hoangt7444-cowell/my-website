# DOMプロセッサ テキスト収集テスト

[index.html](index.html) からテストページを開けます。すべてのページに、テストサイトで使用する翻訳スクリプトを設定しています。

各ページは、クローラーの収集処理を手動で確認するための小さなfixtureです。

- `01-basic-structure.html`: ブロック要素、入れ子要素、見出し、リスト項目。
- `02-inline-markup.html`: 同じフレーズに残るインライン要素と `stt-id` マーカー。
- `03-attributes.html`: `alt`、`title`、`aria-label`、inputのvalue/placeholder、textareaのplaceholder、optionのlabel。
- `04-head-metadata.html`: ページタイトルと許可されたmeta content。robotsのmetaは無視します。
- `05-ignored-content.html`: 収集対象外タグと `stt-ignore`、`data-stt-ignore` マーカー。
- `06-links-and-empty-nodes.html`: 電話リンク、コメント、空のインライン要素、ブロック境界。
- `07-css-phrasing.html`: `cssPhrasing` 有効時のブロック表示インライン要素とインライン表示要素。
- `08-unicode-and-filters.html`: 空白の正規化、Unicodeの正規化、重複フレーズ、オプションで除外する記号だけのテキスト。
- `09-mixed-all-cases.html`: 上記のケースを1ページにまとめた総合テスト。

`ignoreSymbols` と `removeTranslationTexts` は、対応するクローラーオプションが必要です。ブラウザ用のscriptタグに設定を追加せず、各ページに確認用の入力テキストを用意しています。
