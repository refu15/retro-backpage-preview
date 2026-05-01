# WordPress「HTMLべた張り＋メディアアップ」運用の落とし穴

**前提**: WPサーバ・テーマファイルには触らない。Gutenbergの「Custom HTML」ブロック等にHTMLを貼り、画像はメディアライブラリにアップ。

---

## 🔴 最重要（やらないとほぼ確実に表示崩壊）

### 1. **wpautop() が `<br>` と `<p>` を勝手に挿入** ← 最大の罠

WordPress は `the_content` フィルターで以下を自動実行:
- 空行を `<p>...</p>` で囲む
- 改行を `<br>` に変換
- `<div>` の中の改行も触る

**影響**:
- インライン `<style>` の中身がぐちゃぐちゃに
- インライン `<script>` のコメント `//` 以降が改行されてSyntaxError
- `<table>` `<ul>` レイアウト崩壊

**対策**:
```
A. プラグイン「Disable wpautop」をインストール
B. functions.php で remove_filter('the_content', 'wpautop') ← サーバ触ること
C. Custom HTML ブロック使用（内部はwpautopが効きにくい）
   → ただし長い<style>/<script>を分割するとフィルターが走る箇所がある
D. ショートコード化して raw 出力させる
```
→ **推奨: ブロックエディタで「カスタムHTML」ブロック1個にHTML全体を貼る**

### 2. **画像URL の53枚一括置換**

WP メディアにアップすると保存先は:
```
https://retro-backpage.com/wp-content/uploads/2026/04/{filename}.jpg
```

**罠**:
- WPは同名ファイルがあると `S__94887956-1.jpg` などサフィックス付与
- `Smush` `EWWW` `Imagify` 等の最適化プラグインで `.jpg` → `.webp` に置換
- `_scaled.jpg` バリアントが自動生成（2560px超で適用）→ src が切替わる
- ファイル名に `_` が `-` に変換されることも

**対策**:
```
□ アップロード前に画像最適化プラグインを一旦無効化
□ アップロード後、メディアライブラリで実URLを確認
□ 53枚分のURLマッピング表を作成
□ HTML内の images/extra/* を一括置換
□ ファイル名に日本語・特殊文字を含めない
```

### 3. **`<head>` 系メタタグが効かない**

カスタムHTMLブロックの内容は `<body>` 内に挿入されます。
つまり以下は **そのまま貼っても無効**:
- `<meta charset>` `<title>` `<meta name="description">`
- `<meta property="og:*">` `<meta name="twitter:*">`
- `<link rel="canonical">` `<link rel="preconnect">`
- `<script type="application/ld+json">` (Schema.org)
- `<meta http-equiv="Content-Security-Policy">`

**対策**:
```
□ Yoast SEO / RankMath で:
  - メタディスクリプション設定
  - OGP画像設定
  - canonical設定
  - JSON-LD自動生成（Restaurant スキーマなど）
□ プラグイン「Insert Headers and Footers」で<head>に追加コード挿入
□ → 結果、HTML側の<head>系は <body>内にある状態になり、無視される
```

### 4. **テーマヘッダー/フッターとの重複**

WPテーマは自動的に:
- `<header>` ロゴ・ナビゲーション
- `<footer>` コピーライト・SNSアイコン
- 固定 CTA / クッキー同意バナー / チャット

を出力します。

**HTML内の `<header class="site-header">` `<footer class="site-footer">` `<div class="float-cta">` は二重表示** になります。

**対策**:
```
A. HTML側から <header>...</header> と <footer>...</footer> と
   <div class="float-cta">...</div> を完全削除（推奨）
B. ページテンプレで no-header テンプレ作成（サーバ触る → スコープ外）
```

### 5. **CSS/JS の specificity 競合（クラス名汚染）**

汎用クラス名（`.hero` `.section` `.party` `.intro` `.plan` `.gallery` `.cal-cell`）は **テーマや他プラグインも使っている可能性大**。

**症状**:
- 余白がおかしい（テーマの `.entry-content p { margin: 1.5em 0 }` が効く）
- 色が違う
- レイアウトが想定外
- フォントが変わる

**対策**:
```
□ 全CSSを #live-party-page で囲んでスコープ化
  例: .hero { ... } → #live-party-page .hero { ... }
□ HTML全体を <div id="live-party-page">...</div> で囲む
□ または !important を多用（非推奨だが即効性あり）
```

---

## 🟡 中程度（少し気にする）

### 6. **WP の自動文字変換（wptexturize）**

`'` → `'`、`"` → `"`、`...` → `…`、`--` → `—` 等に勝手に変換。

**影響**: JavaScript内の文字列が壊れる
```js
var name = 'retro';  // ' が ' に変換されるとSyntaxError
```

**対策**:
```
□ プラグイン「Disable wptexturize」
□ JavaScript部分は <pre> や CDATA で囲む（一部効果）
□ 全体的に "..." よりも \" エスケープを使う
```

### 7. **外部リンク強制 noopener**

WP 5.6+ は `target="_blank"` のリンクに自動で `rel="noopener noreferrer"` を付与。
→ 動作的には問題なし（むしろセキュリティ向上）

### 8. **画像の自動 srcset / レスポンシブ**

WPは `<img>` に `srcset` と `sizes` を自動追加することがあります。
→ 我々の `.party-visual img { object-fit: cover }` は通常通り効くはずですが、
   `width` `height` 属性をWPが追加すると aspect ratio がズレる。

**対策**:
```
□ HTML内の<img>に明示的に width="" height="" 属性を入れる
□ または add_filter('wp_calculate_image_srcset', '__return_false') ← サーバ触る
```

### 9. **長すぎるHTMLでブロックエディタが固まる**

3,866行 ≈ 175KB の HTML を1つのCustom HTMLブロックに貼る場合:
- ブロックエディタの保存が遅い・タイムアウト
- 編集中に固まる
- スマホからの編集が困難

**対策**:
```
A. クラシックエディタプラグインで「テキストモード」で貼る（推奨）
B. Custom HTML ブロックを <head> / <body 上半分> / <body 下半分> で3分割
C. 編集はテキストエディタで行い、コピペで貼り直す運用
```

### 10. **キャッシュプラグインの干渉**

WP Rocket / W3 Total Cache / Autoptimize 等は:
- インラインCSSを外部ファイル化
- インラインJSを minify
- 一部の `<script>` を遅延ロード

→ JS の実行順序が変わって、`document.getElementById('genBtn')` がnullになる可能性。

**対策**:
```
□ 該当ページだけキャッシュ除外設定
□ または「Defer JavaScript」を無効化
```

---

## 🟢 軽微（多分大丈夫）

### 11. **CSP重複**

我々の `<meta http-equiv="Content-Security-Policy">` は body 内に挿入されるため **無効**。
→ 結果、WPサーバ/プラグインが設定するCSP（or なし）が支配的に。
→ **iframeやfontsが意図せずブロックされる可能性は低い**

### 12. **HTML属性「class」「id」のSEOプラグイン不適合**

Yoast の「キーフレーズ密度分析」が我々のHTMLを正しく解析できず、警告が出るかも。
→ ページ評価が下がるが、検索順位に直接影響なし。

### 13. **改行コード**

WP DBに保存される際、改行コードが LF / CRLF / CR の混在になることあり。
→ 表示には影響なし。

---

## 📋 推奨セットアップ手順（HTMLべた張り運用）

### Phase 1: 事前準備（10分）
```
□ プラグイン:
  - Classic Editor（テキストモード復活）
  - Disable wpautop（自動<p>挿入抑制）
  - Disable wptexturize（自動文字変換抑制）
  - Insert Headers and Footers（<head>追加コード挿入用）
  - Yoast SEO or RankMath（メタ設定用）
□ 既存テーマの header.php / footer.php の挙動確認
□ 既存 SEO プラグインの canonical 設定確認
```

### Phase 2: 画像アップロード（30分）
```
□ 最適化プラグイン（Smush等）を一時的に無効化
□ メディアライブラリで「2026/04/retro-lp/」フォルダ作成（FileBird等）
□ 53枚を一括アップロード
□ 各画像のURLをスプレッドシートにメモ
□ 最適化プラグインを再有効化（次回以降のため）
```

### Phase 3: HTML加工（30分）
```
□ HTMLから <header>...</header> 削除
□ HTMLから <footer>...</footer> 削除
□ HTMLから <div class="float-cta">...</div> 削除
□ <head>内の <meta>/<link>/<script type="ld+json"> を抜き出して保管
□ images/extra/* → https://retro-backpage.com/wp-content/uploads/2026/04/* に一括置換
□ 全CSSを #lp-party { ... } でスコープ化
□ HTML全体を <div id="lp-party">...</div> で囲む
```

### Phase 4: WP設定（20分）
```
□ Yoast/RankMath で:
  - タイトル: PARTY & LIVE | retro Back Page
  - ディスクリプション: HTML内のmeta description をコピー
  - OGP画像設定
  - canonical: https://retro-backpage.com/live_party/
□ Insert Headers and Footers で:
  - JSON-LD Schema をHEADに貼り付け
  - preconnect / preload を貼り付け
  - GTM コード貼り付け
□ ページ作成 → スラッグ /live_party/
□ Custom HTML ブロックに加工済HTMLを貼る
□ 公開
```

### Phase 5: 動作確認（30分）
```
□ PCブラウザで表示崩れなし
□ スマホで表示崩れなし
□ 各画像表示
□ 電話タップ→発信
□ メールボタン→メーラー起動 + プリセット
□ ギャラリーフィルター動作
□ 見積もりシミュレーター動作
□ Googleカレンダー表示
□ PageSpeed Insights ≥ 70（mobile）
□ Search Console 構造化データエラーなし
```

---

## ⚖️ 結論：めんどい？

| 項目 | めんどさ |
|---|---|
| **画像アップ＋URL置換** | ★★★ 中（53枚 × 確認） |
| **wpautop対策** | ★★ 低（プラグイン1個入れるだけ） |
| **テーマhead/footer重複** | ★★ 低（HTMLから3ブロック削除するだけ） |
| **CSS specificity競合** | ★★★★ 高（実機確認しながらスコープ調整） |
| **`<head>` 系設定** | ★★★ 中（SEOプラグイン再設定） |
| **編集の継続性** | ★★★★ 高（次回更新時も全文ペースト運用） |

**総合**: 初回 **2〜3時間** 、運用は **1更新あたり10分**（HTMLをローカルで編集→コピペ）

→ **やればできる、ただしCSS specificity競合と画像URL置換の作業量は覚悟**

---

## 💡 こうした方が楽な代替案

| 方式 | 楽さ | 制約 |
|---|---|---|
| **A. HTMLべた張り（現方針）** | ★★ | 上記の罠あり |
| **B. iframe で github.io を埋め込み** | ★★★★★ | UX違和感、SEO効果薄、認証問題 |
| **C. サブドメイン `lp.retro-backpage.com` に静的HTMLデプロイ** | ★★★★ | サーバ設定要、ドメイン分離 |
| **D. カスタムページテンプレ（PHP）** | ★★★ | サーバアクセス必要 |

→ サーバ触れないなら **A** が現実解。**B（iframe）は安易だが本当におすすめしない**（SEO・UX的に）

---

**作成者**: Claude Code  
**最終更新**: 2026-04-30
