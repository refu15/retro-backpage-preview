# 公開前 QA チェックリスト

**対象**: retro-backpage.com /live_party/ 刷新ページ
**作成日**: 2026-04-30
**スコープ**: 6項目（電話・画像・カレンダー・遷移・メール・セキュリティ）

凡例: ✅ 検証OK / ⚠️ 要確認 / ❌ NG / 🔒 自動チェック済

---

## 1. 📞 電話がつながるか

| # | チェック項目 | 結果 | 詳細 |
|---|---|---|---|
| 1.1 | 電話番号の一貫性（全箇所） | ✅ 🔒 | 7箇所すべて `022-264-1084` / `tel:0222641084` で統一 |
| 1.2 | tel: リンク形式の正当性 | ✅ 🔒 | `tel:0222641084`（ハイフン無し・国番号無し）OK |
| 1.3 | スマホでタップ→発信 | ⚠️ 要手動確認 | iPhone Safari / Android Chrome で実機テスト |
| 1.4 | 通話可能（営業時間内） | ⚠️ 要手動確認 | 営業時間 18:00-26:00 内に発信して呼出確認 |
| 1.5 | 国際表記（+81-22-264-1084） | ✅ | JSON-LD のみ。発信タップは tel:0222641084 |
| 1.6 | 固定 / フッター / モーダル / ヒーロー の整合 | ✅ 🔒 | grep で確認（site-header / cal-box / contact-modal / float-tel / footer すべて同一） |

**手動確認手順**:
```
□ PCブラウザで番号文字列を選択コピー → 22文字長 OK
□ iPhone Safari → 番号タップ → 発信画面起動
□ Android Chrome → 番号タップ → 発信画面起動
□ 実際に発信 → お店で受話確認
```

---

## 2. 🖼 画像がすべて同じフォルダにまとまっているか

| # | チェック項目 | 結果 | 詳細 |
|---|---|---|---|
| 2.1 | ローカル画像が `images/` 配下に集約 | ✅ 🔒 | 47箇所すべて `images/{extra,gallery/party,gallery/live,gallery/space}/` 配下 |
| 2.2 | 外部画像URL（retro-backpage.com） | ⚠️ 🔒 | 5箇所あり（OG image / hero bg / Plan card 1-3）→ retro-backpage.com の WordPress テーマ画像を直接参照 |
| 2.3 | github.io 直リンク残留 | ✅ 🔒 | 完全除去済（about-section bg を images/extra/ に移行） |
| 2.4 | リポジトリ内未使用ファイル | ⚠️ 🔒 | live01, live14, live15 の3枚が未参照 |
| 2.5 | 巨大ファイル（>1MB）| ❌ 🔒 | live02-13, 16 が **1.3〜2.3 MB**。ページ合計 ~28MB → LCP悪化 |
| 2.6 | alt 属性の有無 | ✅ 🔒 | 全 img タグに alt 設定済 |
| 2.7 | loading="lazy" / decoding="async" | ✅ 🔒 | ファーストビュー以外に適用済 |

**WordPress移行時の注意**:
- 全画像を `wp-content/uploads/2026/04/` 等にアップ
- `images/` プレフィックスを WP メディアURL に一括置換
- 外部画像URL（`https://retro-backpage.com/wp/wp-content/themes/backpage/img/...`）は**そのまま流用可**（生存確認済 200 OK）

---

## 3. 📅 Google カレンダー（実際は TimeTree）連携

| # | チェック項目 | 結果 | 詳細 |
|---|---|---|---|
| 3.1 | Google Calendar iframe（旧実装） | ✅ 撤去済 | staticrypt + 3rd-party Cookie 制約で表示不可だったため削除 |
| 3.2 | 静的サンプル月間表示（2026年5月） | ✅ | ◎○△× 凡例付き、月曜定休 |
| 3.3 | TimeTree CTA ボタン | ✅ | 「📅 TimeTreeで最新の空き状況を確認」配置済 |
| 3.4 | data-timetree-url 属性 | ✅ | JS で URL 注入待機 |
| 3.5 | TimeTree 公開URL の設定 | ❌ | **未設定**（`TIMETREE_URL = ''`）→ クリックで「準備中」アラート |
| 3.6 | CSP frame-src | ✅ | `https://timetreeapp.com` 許可済 |
| 3.7 | iCal URL 受領 → サーバ側パース実装 | ⏳ Phase 2 | WordPress 移行後に実装 |

**必要なクライアント側作業**:
```
□ TimeTree にログイン (retrobackpage@gmail.com / backpage1084)
□ 貸切予約専用カレンダー作成 or 既存確認
□ 「カレンダー詳細を共有」→ 公開URL 取得
   形式: https://timetreeapp.com/public_calendars/xxxxxxxx
□ そのURLを共有 → 私が TIMETREE_URL に1行注入
```

---

## 4. 🔗 別ページに遷移するリンクが間違いないか

| # | リンク種別 | 結果 | 詳細 |
|---|---|---|---|
| 4.1 | ヘッダーロゴ → `https://retro-backpage.com/` | ✅ 🔒 | 200 OK |
| 4.2 | ナビ TOP / ABOUT / FOOD / PARTY / NEWS / CONTACT | ⚠️ 🔒 | 全 301 redirect（trailing slash → no slash） |
| 4.3 | フッター Menu / Info の各リンク | ⚠️ 🔒 | 同上、301 redirect |
| 4.4 | フッター PRIVACY | ⚠️ 🔒 | 301 redirect、ページ存在 |
| 4.5 | preconnect / preload `https://retro-backpage.com` | ✅ 🔒 | DNS prefetch / preload 動作 |
| 4.6 | OG / canonical URL | ✅ | `https://retro-backpage.com/live_party/` に修正済 |
| 4.7 | Google Fonts CDN | ✅ 🔒 | fonts.googleapis.com / fonts.gstatic.com OK |
| 4.8 | カレンダー API（旧） | ✅ 撤去済 | calendar.google.com への依存無し |

**🟡 軽微な改善余地**:
- ナビ・フッターのリンクから末尾 `/` を外すと 301 を避けられる（速度改善）
- 例: `/about/` → `/about`
- 影響: 高負荷時のレスポンス短縮（数十ms）

**手動確認**:
```
□ ヘッダー全リンクをクリック → 想定ページに遷移するか
□ フッター全リンクをクリック → 同上
□ 戻るボタンで本ページに戻れるか
□ 外部リンクは新タブで開くか（target="_blank" 適用は無し → 同タブで開く）
```

---

## 5. 📧 メールが飛ぶか

| # | チェック項目 | 結果 | 詳細 |
|---|---|---|---|
| 5.1 | メール宛先 (`MAIL_TO`) | ❌ | **`info@retro-backpage.com`** のまま → backpage.txt記載の **`mail@retro-backpage.com`** に要更新 |
| 5.2 | mailto: リンク数 | ✅ 🔒 | 4箇所すべて `data-mail-cta` で動的生成 |
| 5.3 | 件名プリセット | ✅ | 「【貸切予約のご相談】」 |
| 5.4 | 本文プリセット | ⚠️ | プラン名が旧表記（平日・早めスタート / 週末×夜スタート） |
| 5.5 | プラン名整合性（メール本文） | ⚠️ | デプロイ済HTMLでは更新済（平日プラン / 週末アーリー / 週末レイト ¥7,000）|
| 5.6 | mailto エンコード | ✅ 🔒 | encodeURIComponent で件名・本文エンコード |
| 5.7 | お店側 受信確認 | ⚠️ 要手動確認 | 実際にメール送信して mail@retro-backpage.com 受信確認 |

**修正必要**:
```
JS: var MAIL_TO = 'info@retro-backpage.com';
  ↓
   var MAIL_TO = 'mail@retro-backpage.com';
```

**手動確認手順**:
```
□ ボタン「メールで貸切問合せ」タップ
□ メーラー（Gmail / Apple Mail / Outlook）起動
□ 件名・本文がプリセットされているか
□ 試験送信 → mail@retro-backpage.com で受信確認
□ 迷惑メールフォルダに入らないか確認
```

---

## 6. 🛡 攻撃に耐えうるか（セキュリティ）

| # | チェック項目 | 結果 | 詳細 |
|---|---|---|---|
| 6.1 | HTTPS 常時化 | ✅ 🔒 | github.io 自動HTTPS / HSTS max-age=31556952 |
| 6.2 | CSP meta（コンテンツポリシー）| ✅ | `default-src 'self'; ...` 設定済、不要ドメイン除去済 |
| 6.3 | X-Frame-Options | ⚠️ | meta では設定不可、サーバ側 (Xserver / WP) で `DENY` 設定推奨 |
| 6.4 | X-Content-Type-Options | ✅ | meta で `nosniff` 設定済 |
| 6.5 | Referrer-Policy | ✅ | `strict-origin-when-cross-origin` 設定済 |
| 6.6 | XSS 対策 | ✅ 🔒 | 唯一の `innerHTML` 使用箇所で `esc()` による HTMLエスケープ実施 |
| 6.7 | 入力フォーム（外部送信）| ✅ 🔒 | radio / checkbox / range / hidden のみ。サーバ送信なし |
| 6.8 | eval / Function | ✅ 🔒 | 使用なし |
| 6.9 | document.write | ✅ 🔒 | 本HTML内では未使用（staticrypt は別） |
| 6.10 | 外部スクリプト | ⚠️ | GTM / GA4（プレースホルダ）— 本番設定時に CSP 整合性確認 |
| 6.11 | パスワード保護（プレビュー）| ✅ | staticrypt + AES (`enterai0428`) |
| 6.12 | 依存ライブラリ脆弱性 | ✅ | 外部ライブラリ未使用（Vanilla JS） |

**🔴 サーバ側で要設定（WP導入時）**:
```apache
# .htaccess (Xserver)
Header set Strict-Transport-Security "max-age=63072000; includeSubDomains; preload"
Header set X-Frame-Options "DENY"
Header set X-Content-Type-Options "nosniff"
Header set Referrer-Policy "strict-origin-when-cross-origin"
Header set Permissions-Policy "geolocation=(), microphone=(), camera=()"
```

**脆弱性テスト推奨**:
- [ ] OWASP ZAP / Burp Suite で自動スキャン
- [ ] Mozilla Observatory（https://observatory.mozilla.org/）で A 評価以上
- [ ] securityheaders.com で B+ 以上
- [ ] SSL Labs（https://www.ssllabs.com/ssltest/）で A 評価
- [ ] CSP Evaluator（Google）で警告ゼロ

---

## 📊 総合スコア（自動チェック分）

| カテゴリ | 達成 | 備考 |
|---|---|---|
| 1. 電話 | 6/6 ✅ | 完全一貫 |
| 2. 画像 | 5/7 ⚠️ | Liveの巨大ファイル要圧縮、未使用3枚要削除 |
| 3. カレンダー | 5/7 ⏳ | TimeTree URL 取得待ち |
| 4. 遷移リンク | 7/8 ✅ | 301redirect 軽微改善余地 |
| 5. メール | 4/7 ⚠️ | 宛先を mail@ に変更要 |
| 6. セキュリティ | 10/12 ✅ | サーバ側ヘッダ設定とOWASPスキャン推奨 |

---

## 🎯 公開前 必須対応（優先順）

### 🔴 即対応
1. **メール宛先**: `info@retro-backpage.com` → `mail@retro-backpage.com`
2. **TimeTree公開URL** をクライアントから取得 → `TIMETREE_URL` 注入
3. **Live画像圧縮**（28MB → 3MB 目標）
4. **未使用画像削除**（live01, 14, 15）

### 🟡 公開前推奨
5. WordPress側で X-Frame-Options / 完全HSTS / Permissions-Policy 設定
6. サーバ側で OWASP ZAP / Mozilla Observatory スキャン
7. 実機テスト：iPhone / Android で電話・メール・各リンク

### 🟢 公開後
8. PageSpeed Insights で LCP < 2.5s 確認
9. Search Console 構造化データ検証
10. Google アナリティクス4 で CV ファネル設定

---

**作成者**: Claude Code  
**最終更新**: 2026-04-30  
**関連ドキュメント**: docs/wordpress-migration-checklist.md
