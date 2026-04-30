# セキュリティ運用ガイド

**対象**: retro Back Page LP（Google Calendar連携を含む）
**作成日**: 2026-04-30

---

## 🔴 即対応すべきセキュリティ課題

### 1. Google アカウント `retrobackpage@gmail.com` パスワード強度

**現状**: `backpage1084`
- **強度評価: 弱（10文字、店名+電話番号末尾の推測可能パターン）**
- ブルートフォース耐性: 数時間〜数日で突破可能
- 辞書攻撃に脆弱

**推奨アクション**:
```
□ パスワード変更:
  - 16文字以上、英大小+数字+記号混在
  - 例: 1Password / Bitwarden 等で生成 (例: K3$mP9!nQ2vL@8wR)
□ 2段階認証（2FA）有効化:
  - 設定 → セキュリティ → 「2段階認証プロセス」をON
  - SMS / Authenticatorアプリ（推奨）
□ バックアップコード保管: 復旧用コード10個を金庫等で物理保管
□ ログイン通知: 新デバイスからのログインをメール通知
```

### 2. **認証情報の管理場所**

**現状**: `backpage.txt`（Downloadsフォルダのプレーンテキスト）

**リスク**:
- マルウェアでファイル盗難の危険
- バックアップ・クラウド同期で他端末に拡散
- 共有時にメールで送信→傍受リスク

**推奨**:
```
□ 1Password / Bitwarden / Google パスワードマネージャー に移行
□ プレーンテキストファイルは削除（または暗号化ZIP化）
□ Git リポジトリには絶対にコミットしない（→ .gitignore で予防）
□ チーム共有は password manager の共有機能 or 暗号化チャット
```

---

## 🟡 Google Calendar 公開設定（必須）

### カレンダー公開範囲設定

```
Google Calendar 設定手順:
1. Calendar 左サイドバー → カレンダー名にカーソル → ⋮ → 「設定と共有」
2. 「アクセス権限」セクションで:
   ☑ 一般公開する
   選択肢:
   - 「予定の有無のみ（詳細は非表示）」← 推奨（プライバシー保護）
   - 「すべての予定の詳細」← 顧客名漏洩リスク
3. 「カレンダーの統合」セクションで:
   - カレンダーID: retrobackpage@gmail.com
   - 公開URL を確認
4. 保存
```

### 🚨 プライバシー保護のための運用ルール

**個人名・連絡先を予定タイトルに入れない**:

| ❌ NG（個人情報漏洩） | ✅ OK（マスキング） |
|---|---|
| 山田太郎様 貸切 60名 | 貸切予約 60名（A社様） |
| 田中様 結婚式二次会 | ウェディング二次会 |
| 営業部 鈴木 忘年会 | 法人パーティー |

**メモ欄の扱い**:
- 予定の「説明」欄に個人名・電話番号を記入する場合は、カレンダー公開範囲を **「予定の有無のみ」** に設定
- それ以外は「すべて表示」でも可（ただし慎重に）

---

## 🛡 LP本体のセキュリティ実装状況

| 項目 | 状態 | 備考 |
|---|---|---|
| HTTPS常時化 | ✅ | github.io 自動HTTPS / WP導入後はXserver SSL |
| HSTS | ✅ | github.io 自動 max-age=31556952 |
| CSP meta | ✅ | `default-src 'self'` 基本、whitelist最小化 |
| X-Content-Type-Options nosniff | ✅ | meta 設定済 |
| Referrer-Policy strict-origin-when-cross-origin | ✅ | meta 設定済 |
| XSS対策（HTMLエスケープ） | ✅ | `esc()` 関数で & < > " ' を変換 |
| eval / Function | ✅ | 未使用 |
| 外部スクリプト | ✅ | GTM / GA4 のみ（プレースホルダ） |
| 入力フォーム | ✅ | radio / checkbox のみ（テキスト入力なし） |
| 依存ライブラリ | ✅ | Vanilla JS（3rd party無し） |

---

## 🔧 WordPress 本番導入時の追加対応

### .htaccess（Xserver）に以下を追加

```apache
# === セキュリティヘッダ ===
<IfModule mod_headers.c>
  Header set Strict-Transport-Security "max-age=63072000; includeSubDomains; preload"
  Header set X-Frame-Options "DENY"
  Header set X-Content-Type-Options "nosniff"
  Header set Referrer-Policy "strict-origin-when-cross-origin"
  Header set Permissions-Policy "geolocation=(), microphone=(), camera=()"
  
  # Calendar iframe 用に X-Frame-Options を SAMEORIGIN に緩和（必要時）
  # Header set X-Frame-Options "SAMEORIGIN"
</IfModule>

# === xmlrpc.php 攻撃対策 ===
<Files xmlrpc.php>
  Order deny,allow
  Deny from all
</Files>

# === wp-config.php / .htaccess へのアクセス禁止 ===
<FilesMatch "^(wp-config\.php|\.htaccess|\.user\.ini)$">
  Order allow,deny
  Deny from all
</FilesMatch>
```

### WordPress側

```
□ WordPress 最新版へアップデート
□ 不要プラグイン削除
□ 必須プラグイン:
  - Wordfence Security（WAF + マルウェアスキャン）
  - Limit Login Attempts Reloaded（ブルートフォース対策）
  - WP Activity Log（操作履歴）
□ 管理画面ログインURL変更（/wp-admin → /wp-login-xxx）
□ 管理者アカウントは admin 以外の名前
□ 管理者パスワード変更（現状 a^aD7gyDg0Zw1lN^Jx は十分強い ✅）
□ 自動更新ON（マイナーバージョン）
□ デバッグモードOFF（WP_DEBUG = false）
```

---

## 🔍 公開前 セキュリティテスト

### 必須

```
□ Mozilla Observatory: https://observatory.mozilla.org/
  → A 評価以上目標
□ Security Headers: https://securityheaders.com/
  → B+ 評価以上目標
□ SSL Labs: https://www.ssllabs.com/ssltest/
  → A 評価目標
□ CSP Evaluator: https://csp-evaluator.withgoogle.com/
  → 警告ゼロ目標
```

### 推奨（脆弱性スキャン）

```
□ OWASP ZAP（無料、ローカル実行）
  → サイト全体の自動スキャン、SQLi/XSS/CSRF検出
□ Nikto（無料、CLI）
  → サーバ設定の不備検出
□ WP scan（WordPress特化）
  → プラグイン・テーマの脆弱性検出
```

### 監視

```
□ Google Search Console「セキュリティの問題」タブを毎月確認
□ Wordfence のメール通知ON
□ Sentry / Loggly でエラーログ監視
```

---

## 📞 インシデント対応

### 不正ログイン疑い時

```
1. パスワード即変更
2. 2FA 強制再設定
3. Google アカウント「アクティビティ」で不審な操作確認
4. WP管理画面「ユーザー」で不審アカウント削除
5. データベースバックアップ取得 → クリーン状態に復元
```

### サイト改ざん時

```
1. メンテナンスモードに切替（503返却）
2. Wordfence でフルスキャン → マルウェア除去
3. WP コア・プラグイン・テーマ全再インストール
4. wp-content/uploads 内の不審ファイル削除
5. データベースから不審な投稿・ユーザー削除
6. パスワード全変更
7. 公開再開
```

---

## 📋 月次セキュリティチェック

```
□ WordPress / プラグイン / テーマ 更新確認・適用
□ Google アカウント アクティビティ確認
□ Wordfence スキャン結果確認
□ サイトのバックアップ取得（手動 or 自動）
□ Search Console「セキュリティ問題」確認
□ Lighthouse Best Practices ≥ 95
```

---

**作成者**: Claude Code  
**最終更新**: 2026-04-30  
**関連**: docs/wordpress-migration-checklist.md / docs/pre-launch-qa-checklist.md
