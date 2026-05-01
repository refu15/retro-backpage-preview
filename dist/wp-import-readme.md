# WordPress導入手順書

**生成日**: 2026-04-30
**対象ページ**: retro-backpage.com /live_party/

## 📦 含まれるファイル

| ファイル | 用途 | 貼付先 |
|---|---|---|
| `wp-paste-body.html` | 本体（CSS/JS含む） | Gutenberg「カスタムHTML」ブロック |
| `wp-paste-head.html` | meta/JSON-LD/GTM | 「Insert Headers and Footers」プラグインの Header |
| `image-url-mapping.csv` | 画像URL対応表 | 参照用（ズレた時の確認） |

---

## 🔧 事前準備

### 必須プラグイン（4つ）
```
□ Classic Editor      — テキストモード復活（任意・編集楽）
□ Disable wpautop     — 自動<p>挿入抑制
□ Insert Headers and Footers — head にコード追加
□ Yoast SEO or RankMath — メタ設定
```

### 一時無効化
```
□ Smush / EWWW / Imagify など画像最適化プラグイン
   → アップロード時にWebP変換やリネームを防ぐため
```

---

## ステップ 1: 画像 53枚 アップロード（30分）

```
1. WP管理画面 → メディア → 新規追加
2. images/ フォルダ内の 53枚 を一括アップ
3. 各画像のURLをコピーして CSV の「実URL」列に貼る
4. 想定URL（B列）と実URL（F列）が一致するか確認
   → 一致しない場合は wp-paste-body.html 内のURLを修正
```

**画像リスト（53枚）**:
  1. `276226_0.jpg` → `https://retro-backpage.com/wp-content/uploads/2026/04/276226_0.jpg`
  2. `S__94887956.jpg` → `https://retro-backpage.com/wp-content/uploads/2026/04/S__94887956.jpg`
  3. `live-add.jpg` → `https://retro-backpage.com/wp-content/uploads/2026/04/live-add.jpg`
  4. `live-extra-01.jpg` → `https://retro-backpage.com/wp-content/uploads/2026/04/live-extra-01.jpg`
  5. `live-extra-02.jpg` → `https://retro-backpage.com/wp-content/uploads/2026/04/live-extra-02.jpg`
  6. `live-extra-03.jpg` → `https://retro-backpage.com/wp-content/uploads/2026/04/live-extra-03.jpg`
  7. `live-extra-04.jpg` → `https://retro-backpage.com/wp-content/uploads/2026/04/live-extra-04.jpg`
  8. `party-toast.jpg` → `https://retro-backpage.com/wp-content/uploads/2026/04/party-toast.jpg`
  9. `piano-01.jpg` → `https://retro-backpage.com/wp-content/uploads/2026/04/piano-01.jpg`
  10. `piano-02.jpg` → `https://retro-backpage.com/wp-content/uploads/2026/04/piano-02.jpg`
  11. `live02.jpg` → `https://retro-backpage.com/wp-content/uploads/2026/04/live02.jpg`
  12. `live03.jpg` → `https://retro-backpage.com/wp-content/uploads/2026/04/live03.jpg`
  13. `live04.jpg` → `https://retro-backpage.com/wp-content/uploads/2026/04/live04.jpg`
  14. `live05.jpg` → `https://retro-backpage.com/wp-content/uploads/2026/04/live05.jpg`
  15. `live06.jpg` → `https://retro-backpage.com/wp-content/uploads/2026/04/live06.jpg`
  16. `live07.jpg` → `https://retro-backpage.com/wp-content/uploads/2026/04/live07.jpg`
  17. `live08.jpg` → `https://retro-backpage.com/wp-content/uploads/2026/04/live08.jpg`
  18. `live09.jpg` → `https://retro-backpage.com/wp-content/uploads/2026/04/live09.jpg`
  19. `live10.jpg` → `https://retro-backpage.com/wp-content/uploads/2026/04/live10.jpg`
  20. `live11.jpg` → `https://retro-backpage.com/wp-content/uploads/2026/04/live11.jpg`
  21. `live12.jpg` → `https://retro-backpage.com/wp-content/uploads/2026/04/live12.jpg`
  22. `live13.jpg` → `https://retro-backpage.com/wp-content/uploads/2026/04/live13.jpg`
  23. `live16.jpg` → `https://retro-backpage.com/wp-content/uploads/2026/04/live16.jpg`
  24. `party01.jpg` → `https://retro-backpage.com/wp-content/uploads/2026/04/party01.jpg`
  25. `party02.jpg` → `https://retro-backpage.com/wp-content/uploads/2026/04/party02.jpg`
  26. `party03.jpg` → `https://retro-backpage.com/wp-content/uploads/2026/04/party03.jpg`
  27. `party04.jpg` → `https://retro-backpage.com/wp-content/uploads/2026/04/party04.jpg`
  28. `party05.jpg` → `https://retro-backpage.com/wp-content/uploads/2026/04/party05.jpg`
  29. `party06.jpg` → `https://retro-backpage.com/wp-content/uploads/2026/04/party06.jpg`
  30. `party08.jpg` → `https://retro-backpage.com/wp-content/uploads/2026/04/party08.jpg`
  31. `party09.jpg` → `https://retro-backpage.com/wp-content/uploads/2026/04/party09.jpg`
  32. `party10.jpg` → `https://retro-backpage.com/wp-content/uploads/2026/04/party10.jpg`
  33. `party11.jpg` → `https://retro-backpage.com/wp-content/uploads/2026/04/party11.jpg`
  34. `party12.jpg` → `https://retro-backpage.com/wp-content/uploads/2026/04/party12.jpg`
  35. `party13.jpg` → `https://retro-backpage.com/wp-content/uploads/2026/04/party13.jpg`
  36. `party14.jpg` → `https://retro-backpage.com/wp-content/uploads/2026/04/party14.jpg`
  37. `party15.jpg` → `https://retro-backpage.com/wp-content/uploads/2026/04/party15.jpg`
  38. `space01.jpg` → `https://retro-backpage.com/wp-content/uploads/2026/04/space01.jpg`
  39. `space02.jpg` → `https://retro-backpage.com/wp-content/uploads/2026/04/space02.jpg`
  40. `space03.jpg` → `https://retro-backpage.com/wp-content/uploads/2026/04/space03.jpg`
  41. `space04.jpg` → `https://retro-backpage.com/wp-content/uploads/2026/04/space04.jpg`
  42. `space05.jpg` → `https://retro-backpage.com/wp-content/uploads/2026/04/space05.jpg`
  43. `space06.jpg` → `https://retro-backpage.com/wp-content/uploads/2026/04/space06.jpg`
  44. `space07.jpg` → `https://retro-backpage.com/wp-content/uploads/2026/04/space07.jpg`
  45. `space08.jpg` → `https://retro-backpage.com/wp-content/uploads/2026/04/space08.jpg`
  46. `space09.jpg` → `https://retro-backpage.com/wp-content/uploads/2026/04/space09.jpg`
  47. `space10.jpg` → `https://retro-backpage.com/wp-content/uploads/2026/04/space10.jpg`
  48. `space11.jpg` → `https://retro-backpage.com/wp-content/uploads/2026/04/space11.jpg`
  49. `space12.jpg` → `https://retro-backpage.com/wp-content/uploads/2026/04/space12.jpg`
  50. `space13.jpg` → `https://retro-backpage.com/wp-content/uploads/2026/04/space13.jpg`
  51. `space14.jpg` → `https://retro-backpage.com/wp-content/uploads/2026/04/space14.jpg`
  52. `space15.jpg` → `https://retro-backpage.com/wp-content/uploads/2026/04/space15.jpg`
  53. `space16.jpg` → `https://retro-backpage.com/wp-content/uploads/2026/04/space16.jpg`


---

## ステップ 2: <head> 設定（10分）

`wp-paste-head.html` をテキストエディタで開く。
内容を「Insert Headers and Footers」の **Scripts in Header** にすべて貼る。

**注意**: Yoast/RankMath を使う場合、以下は **削除** してから貼る（重複防止）:
- `<title>`
- `<meta name="description">`
- `<link rel="canonical">`
- `<meta property="og:*">`
- `<meta name="twitter:*">`

JSON-LD / GTM / preconnect / preload は残してOK。

---

## ステップ 3: ページ作成（5分）

```
1. WP管理画面 → 固定ページ → 新規追加
2. タイトル: 「PARTY & LIVE」（適宜変更）
3. スラッグ: live_party
4. 「カスタムHTML」ブロック追加 → wp-paste-body.html を全文貼り付け
5. プレビュー → 表示確認
6. 公開 → 既存 /live_party/ ページを上書き or 別URLで作成
```

---

## ステップ 4: 動作確認（15分）

| 確認項目 | 方法 |
|---|---|
| 画像表示 | 全セクションで画像が表示されるか |
| 電話タップ | スマホで `022-264-1084` をタップ → 発信画面 |
| メールボタン | mail@retro-backpage.com に件名・本文プリセットで起動 |
| ギャラリー切替 | All / Party / Live / Space タブ動作 |
| 見積シミュレーター | 3プラン選択・人数入力・PDF出力 |
| Googleカレンダー | iframe表示（公開設定済の場合）|
| レスポンシブ | スマホ／タブレット／PC で崩れない |

---

## 🚨 トラブルシューティング

### 表示崩れる
→ テーマCSSが #lp-party を上書きしている可能性
→ ブラウザDevTools で要素検査、`!important` 追加で個別対応

### 画像404
→ `image-url-mapping.csv` の実URLと wp-paste-body.html 内のURLが不一致
→ 一括置換: VSCode等で wp-paste-body.html を開き、誤URLを正URLに置換

### JSが動かない
→ wpautop / wptexturize で改変された可能性
→ 該当プラグインを有効化したか確認
→ ブラウザDevTools Console でエラー確認

### Googleカレンダー表示されない
→ retrobackpage@gmail.com にログインし、カレンダーを「一般公開」に設定
→ 設定 → アクセス権限 → 「予定の有無のみ」推奨

---

## 📞 サポート

設定で詰まったら、ブラウザDevToolsの **Console / Network タブのスクショ** + **問題の画面スクショ** を送ってください。

---

**生成スクリプト**: `wp_prepare.py`
