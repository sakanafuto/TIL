# Material Theme を生成する

[**Material Theme Builder**](https://m3.material.io/theme-builder#/custom) というサイトから好きな色を選び、用途に合わせて Export する。

## Sample

### Flutter

README.md に記載がある通り、カラーテーマファイルを定義してそれを main.dart で読み込んであげるだけ。

![](/assets/material_colors_flutter.png)

### Confluence

こちらは手動で色をつけてあげるしかない…。

CSS でテーマをエクスポートし、`tokens.css` から primary や secondary-container などのカラーコードを確認する（/_ light _/ 辺り、下部のコード参照）。

Confluence > スペースツール > ルックアンドフィール > 配色 からカラーコードを指定する。

いい感じの組み合わせを示す。

| confluence                       | material               | sample  |
| -------------------------------- | ---------------------- | ------- |
| トップ バー                      | primary                | #6750A4 |
| トップ バーの文字                | on-primary             | #FFFFFF |
| ヘッダー ボタンの背景            | on-primary-container   | #21005D |
| ヘッダー ボタンの文字            | on-tertiary            | #FFFFFF |
| トップ バー メニュー選択時の背景 | on-primary             | #FFFFFF |
| トップ バー メニュー選択時の文字 | secondary              | #625B71 |
| トップ バー メニュー項目の文字   | secondary              | #625B71 |
| メニュー項目選択時の背景         | tertiary-container     | #ffd9e2 |
| メニュー項目選択時の文字         | on-tertiary-container  | #3e001d |
| 検索フィールド背景               | on-primary-container   | #1D192B |
| 検索フィールド テキスト          | secondary-container    | #E8DEF8 |
| ページ メニュー選択時の背景      | on-secondary-container | #1D192B |
| ページ メニュー項目の文字        | secondary              | #625B71 |
| 見出しの文字                     | secondary              | #625B71 |
| リンク                           | tertiary               | #7D5260 |
| 枠線と区切り線                   | on-secondary-container | #1D192B |

```css
:root {
  ...

  /* light */
  --md-sys-color-primary-light: #6750a4;
  --md-sys-color-on-primary-light: #ffffff;
  --md-sys-color-primary-container-light: #eaddff;
  --md-sys-color-on-primary-container-light: #21005d;
  --md-sys-color-secondary-light: #625b71;
  --md-sys-color-on-secondary-light: #ffffff;
  --md-sys-color-secondary-container-light: #e8def8;
  --md-sys-color-on-secondary-container-light: #1d192b;
  --md-sys-color-tertiary-light: #7d5260;
  --md-sys-color-on-tertiary-light: #ffffff;
  --md-sys-color-tertiary-container-light: #ffd8e4;
  --md-sys-color-on-tertiary-container-light: #31111d;
  --md-sys-color-error-light: #b3261e;
  --md-sys-color-on-error-light: #ffffff;
  --md-sys-color-error-container-light: #f9dedc;
  --md-sys-color-on-error-container-light: #410e0b;
  --md-sys-color-outline-light: #79747e;
  --md-sys-color-background-light: #fffbfe;
  --md-sys-color-on-background-light: #1c1b1f;
  --md-sys-color-surface-light: #fffbfe;
  --md-sys-color-on-surface-light: #1c1b1f;
  --md-sys-color-surface-variant-light: #e7e0ec;
  --md-sys-color-on-surface-variant-light: #49454f;
  --md-sys-color-inverse-surface-light: #313033;
  --md-sys-color-inverse-on-surface-light: #f4eff4;
  --md-sys-color-inverse-primary-light: #d0bcff;
  --md-sys-color-shadow-light: #000000;
  --md-sys-color-surface-tint-light: #6750a4;
  --md-sys-color-outline-variant-light: #cac4d0;
  --md-sys-color-scrim-light: #000000;

  ...

}
```
