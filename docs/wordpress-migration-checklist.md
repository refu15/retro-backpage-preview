# WordPress 移行前 チェックシート

**対象**: `proposal_01_elegant_white.html` → retro-backpage.com（WordPress）の `/live_party/` ページ
**作成日**: 2026-04-30
**ステータス凡例**: 🔴 必須対応 / 🟡 要確認 / 🟢 そのままOK

---

## A. 構造的な問題（最重要）

| # | 項目 | ステータス | 詳細 | 対応 |
|---|---|---|---|---|
| A1 | **HTML全体が独立ページ** | 🔴 | `<!DOCTYPE>` `<html>` `<head>` `<body>` `<header>` `<footer>` `<nav>` を全て持つスタンドアロンHTML（3,866行） | WordPress のページに `<head>` `<header>` `<footer>` がダブる。**カスタムページテンプレート（PHP）** または **Custom HTML ブロック + 重複削除** で対応 |
| A2 | **`<header class="site-header">`** | 🔴 | L2321-2339 にロゴ・ナビ・電話番号 | WordPress の theme header と重複 → 削除 or 統合判断 |
| A3 | **`<footer class="site-footer">`** | 🔴 | L3244 以降にフッター | WordPress の theme footer と重複 → 削除 or 統合判断 |
| A4 | **`<div class="float-cta">`** | 🟡 | L3274 以降の右下固定CTAボタン | WordPress 既存サイトに同等機能あれば削除 |
| A5 | **`<style>` 大量インライン**（〜2300行）| 🟡 | CSS 全部 `<head>` 内 | テーマCSSと特異性競合の可能性 → スコープ化（`#party-page .xxx` 等で囲う）推奨 |
| A6 | **`<script>` 大量インライン**（〜500行）| 🟡 | gallery filter / lightbox / simulator など | jQuery 等との衝突確認、`<script type="module">` 化検討 |

---

## B. 画像パス（30+ 箇所）

| # | 項目 | ステータス | 詳細 | 対応 |
|---|---|---|---|---|
| B1 | **`images/extra/*.jpg`** | 🔴 | S__94887956 / 276226_0 / live-add / live-extra-01〜04 / piano-01,02 / party-toast = **9枚** | `wp-content/uploads/2026/04/` 等にアップロード → src を絶対URLに置換 |
| B2 | **`images/gallery/party/*.jpg`** | 🔴 | party01〜13 = **13枚** | 同上（既存サイトに同名がある場合は再利用可） |
| B3 | **`images/gallery/live/*.jpg`** | 🔴 | live02〜04（live08は別途） = **4枚** | 同上 |
| B4 | **`images/gallery/space/*.jpg`** | 🔴 | space02〜14 = **6枚** | 同上 |
| B5 | **既存 `https://retro-backpage.com/wp/wp-content/themes/backpage/img/live_party/photo01.jpg` 等** | 🟢 | OG画像・ヒーロー背景・JSON-LD 画像参照 | 既に本番URL指定。テーマパス変更がなければOK |
| B6 | **`https://onorikuto7978-sys.github.io/retro-backpage-site/images/retro/retro-about-heading.png`** | 🔴 | L2111 の `.about-section` 背景画像 | retro-backpage.com のメディアにアップロード→差替必須 |
| B7 | **`https://retro-backpage.com/wp/wp-content/themes/backpage/img/index/mv_img01.jpg`** | 🟢 | `<link rel="preload">` (L251) + `.hero` 背景 (L457) | 本番URL ✓ |

---

## C. 外部URL / canonical / JSON-LD

| # | 項目 | ステータス | 詳細 | 対応 |
|---|---|---|---|---|
| C1 | **canonical** | 🔴 | L48: `https://y-hashimoto-vaizo.github.io/retro-backpage-demo/...` | `https://retro-backpage.com/live_party/` へ変更 |
| C2 | **og:url** | 🔴 | L56: GitHub Pages URL | 同上 |
| C3 | **og:image / twitter:image** | 🟢 | retro-backpage.com の photo01.jpg | OK（テーマ依存） |
| C4 | **JSON-LD `@id` / `url`**（4箇所）| 🔴 | L76, L80, L147, L158, L186, L197, L225, L236 が GitHub Pages | 全て `https://retro-backpage.com/live_party/` 系に置換 |
| C5 | **JSON-LD Event エントリ** | 🟡 | デモの placeholder Event 3つ | 実公演に差替 or Event スキーマ自体削除 |
| C6 | **header ナビゲーションのリンク** | 🟢 | L2328-2333 すべて `https://retro-backpage.com/...` | OK（ただし A2 で `<header>` を使うかどうかの判断と連動） |
| C7 | **ロゴリンク** | 🟢 | L2322: `https://retro-backpage.com/` | OK |

---

## D. CSP / セキュリティヘッダ

| # | 項目 | ステータス | 詳細 | 対応 |
|---|---|---|---|---|
| D1 | **CSP meta タグ** | 🔴 | L16: `img-src ... https://onorikuto7978-sys.github.io` を含む | 自ドメイン (retro-backpage.com) を `'self'` でカバー、不要ドメイン削除。**WordPress側でCSPを別管理する場合は meta 削除推奨** |
| D2 | **HSTS** | 🟡 | コメントでサーバ側設定を案内 | Xserver の HSTS 設定確認 |
| D3 | **GTM_ID = ''** | 🟡 | L29: 空 → スキップ動作 | 本番GTM IDを設定 or 削除（既存テーマでGAが入っているなら不要） |

---

## E. 動的要素 / フォーム / カレンダー

| # | 項目 | ステータス | 詳細 | 対応 |
|---|---|---|---|---|
| E1 | **静的カレンダー（2026年5月）** | 🔴 | 月毎に手動更新が必要。本日（4/30）→ 5月で良いか、6月も載せるか要判断 | 運用ルール決め。可能ならJSで動的生成（同じデザインを保ったまま） |
| E2 | **見積シミュレーター** | 🟢 | クライアント完結。電話・メール導線あり | OK |
| E3 | **`<a href="#" data-mail-cta>`** | 🟢 | JSが `mailto:` 動的生成 | OK |
| E4 | **`<a href="#" data-contact-cta>`** | 🟢 | スクロール to `#reservation` | OK |
| E5 | **印刷ボタン**（PDF出力）| 🟢 | `window.print()` で動作 | OK |
| E6 | **電話 `tel:0222641084`** | 🟢 | OK | 番号確認のみ |
| E7 | **メール宛先 `info@retro-backpage.com`（推測）** | 🟡 | JS内のMAIL_TO値を確認 | **実宛先の確定**必要 |

---

## F. WordPress 統合方式の判断

| 方式 | メリット | デメリット | 推奨度 |
|---|---|---|---|
| **A. カスタムページテンプレート（page-live_party.php）** | CSS/JS そのまま使える、SEO/構造完全制御 | テーマファイル編集必要、PHP知識必須 | ★★★★★ |
| **B. 固定ページ + Custom HTML ブロック** | 編集ラク、テンプレ不要 | テーマheader/footerと衝突、スタイル競合多発 | ★★ |
| **C. ショートコード化** | 再利用可、ブロックエディタと併用 | 開発工数大 | ★★★ |
| **D. 別ドメイン / サブディレクトリで静的HTML** | そのまま使える | メイン導線統合不可、SEO分散 | ★ |

→ **推奨: A（カスタムページテンプレート）**。テーマの `header.php` `footer.php` を呼ばず、独自で全描画。

---

## G. 移行手順（推奨フロー）

1. **画像アップロード（B1〜B4, B6）**
   - WP管理画面 → メディア で全画像（約32枚）をアップロード
   - URLメモ（後で一括置換）
2. **画像URL一括置換**
   - sed / 検索置換で `images/extra/` → `https://retro-backpage.com/wp-content/uploads/2026/04/`
3. **canonical / OG / JSON-LD URL書換（C1〜C4）**
4. **背景画像のURL書換（B6）**
5. **CSP meta調整（D1）**
6. **`<header>` `<footer>` `<style>` `<script>` を残すか・分離するか判断**
7. **`page-live_party.php` 作成 → テーマにアップロード**
8. **WordPress管理画面でページ作成 → テンプレート割当**
9. **ステージングで動作確認**
   - 全リンク・全フォーム・スマホ表示・PDF出力
10. **本番公開**

---

## H. 公開直前の最終チェック（QAリスト）

- [ ] 画像が全て表示される（404なし）
- [ ] 電話番号タップで発信
- [ ] メールボタンで件名・本文プリセットされたメーラー起動
- [ ] 各CTA（プラン申込・問合せ）がスクロール／モーダル表示
- [ ] 見積シミュレーターが計算する（3プラン × 人数 × 設備）
- [ ] PDF印刷出力が見栄え良い（@media print）
- [ ] スマホ（320 / 375 / 414 / 768）で崩れない
- [ ] PC（1280 / 1440 / 1920）で崩れない
- [ ] About見出し改行が綺麗
- [ ] ギャラリーのフィルター（All/Party/Live/Space）が動作
- [ ] ライトボックス（画像クリックで拡大）が動作
- [ ] 既存サイトのナビゲーションと整合
- [ ] OGP プレビュー（Twitter/Facebook デバッガで確認）
- [ ] PageSpeed Insights LCP < 2.5s
- [ ] Search Console 構造化データエラー無し
- [ ] HTTPS / HSTS有効

---

## I. 残課題（クライアント連絡待ち）

- [ ] 予約用 Google Calendar の共有URL or カレンダーID（カレンダーを動的化する場合）
- [ ] メール送信先アドレス確定
- [ ] GTM コンテナID（既存サイトに有るなら統合）
- [ ] `<header>` `<footer>` の最終仕様（既存テーマ流用 or 新ページ用に独自）
- [ ] JSON-LD Event の実データ（or 削除判断）
- [ ] ファビコン / `<link rel="icon">` の追加

---

## J. リスクと対応

| リスク | 影響度 | 対応策 |
|---|---|---|
| 既存テーマCSSが当ページに干渉 | 高 | `body.page-live_party { ... }` で全体スコープ化、または `header.php`/`footer.php` を読まないテンプレート |
| WordPressプラグイン（Yoast SEO等）がmeta上書き | 中 | プラグイン設定でこのページの自動meta生成を停止 |
| Page Builderプラグイン（Elementor等）が干渉 | 中 | このページだけ無効化 |
| 画像のCDN/最適化プラグインが既存image src壊す | 中 | 該当プラグインを除外設定 |
| カレンダーの月次更新忘れ | 低 | 運用フロー化（毎月25日リマインダー）or 動的化 |
| メール宛先誤設定でリード喪失 | 高 | 公開前にテスト送信 |

---

**作成者**: Claude Code  
**最終更新**: 2026-04-30
