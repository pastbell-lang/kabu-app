# 株式管理アプリ 構築手順書

## 全体像

- **アプリ本体**: iPhoneのSafariで開き「ホーム画面に追加」して使うWebアプリ(kabu-app/index.html)
- **データ置き場**: GitHubの**非公開**リポジトリ(kabu-dataフォルダの中身)。売買記録・株価・履歴・設定がここに保存されます
- **自動更新**: GitHub Actionsが月〜土の7:00 / 11:35 / 16:00(日本時間)に株価・為替・指数を取得。アプリの「↻ 最新に更新」ボタンでいつでも即時実行できます
- **保護**: アプリはパスワードで開きます。データへのアクセス鍵(トークン)はパスワードで暗号化されiPhone内にのみ保存されます
- **費用**: すべて無料枠内(GitHub Free)

Excelからの移行は完了済みです。トレード431件(保有10銘柄)、損益履歴159日分、限界リスク表、キャピタルGain累計(¥73,879,760)の連続性も引き継いでいます。

---

## 手順1: GitHubアカウント作成(お持ちなら不要・5分)

1. https://github.com で Sign up。メールアドレスとパスワードを登録
2. ユーザー名(例: taro-yamada)は後で使うのでメモしてください

## 手順2: データ用リポジトリ(非公開)を作る(10分)

1. GitHubにログイン → 右上「+」→「New repository」
2. Repository name: `kabu-data` / **Private を選択**(重要)/「Add a README file」にチェック →「Create repository」
3. お渡ししたZIPの `kabu-data` フォルダの中身をアップロードします:
   - リポジトリ画面「Add file」→「Upload files」→ `data` フォルダと `update_prices.py` をドラッグ&ドロップ →「Commit changes」
   - ワークフローファイルだけは手動作成が必要です:「Add file」→「Create new file」→ファイル名欄に `.github/workflows/update.yml` と入力し、お渡しした update.yml の中身を貼り付け →「Commit changes」
4. 「Actions」タブを開き、有効化を求められたら「I understand… enable them」をクリック

## 手順3: アプリ用リポジトリ(公開)とページを作る(10分)

アプリの入れ物です。**このリポジトリにはプログラムしか入っておらず、資産データは一切含まれません**(データは手順2の非公開側にあります)。

1. 「New repository」→ Repository name: `kabu-app` / Public /「Add a README file」にチェック → Create
2. 「Add file」→「Upload files」→ `index.html` をアップロード → Commit
3. リポジトリの「Settings」→ 左メニュー「Pages」→ Branch を `main`、フォルダ `/ (root)` にして「Save」
4. 数分後、同じPages画面に `https://ユーザー名.github.io/kabu-app/` というURLが表示されます。これがアプリのURLです

## 手順4: アクセストークン(PAT)を作る(10分)

アプリが非公開データを読み書きするための鍵です。

1. GitHub右上のアイコン →「Settings」→ 左最下部「Developer settings」→「Personal access tokens」→「Fine-grained tokens」→「Generate new token」
2. 設定:
   - Token name: `kabu-app`
   - Expiration: 最長(またはCustomで1年後。期限が切れたら再発行して設定し直します)
   - Repository access: **Only select repositories** → `kabu-data` だけを選択
   - Permissions → Repository permissions:
     - **Contents: Read and write**
     - **Actions: Read and write**
3. 「Generate token」→ 表示された `github_pat_...` を**必ずコピー**(この画面を閉じると二度と表示されません)

## 手順5: iPhoneで初回設定(5分)

1. Safariで手順3のURLを開く
2. 初回設定画面に入力:
   - GitHubユーザー名 / データリポジトリ名(kabu-data)/ トークン(手順4でコピーしたもの)
   - パスワード(お好きなもの。2回入力)→「開く」
3. ホーム画面が表示されたら成功。共有ボタン(□↑)→「ホーム画面に追加」でアイコンを設置
4. 「↻ 最新に更新」を一度タップし、1分ほどで株価が最新に切り替わることを確認してください

---

## 日々の使い方

- **売買の入力**: 下部の「入力」タブから登録。売却は ホーム → 銘柄をタップ → 売却欄に入力
- **株価**: 月〜土の7:00 / 11:35 / 16:00に自動更新。見たい時は「↻ 最新に更新」
- **RR分析**: 通算・直近1カ月(エントリー日基準)・清算済・保有の4区分と月毎RR。Excelシートと同じ計算式です
- **余力(現金)**: 入出金があったら「設定」タブで金額を更新(投資可能資産の計算に使われます)
- **バックアップ**: 「設定」→ CSVエクスポート。またGitHub自体が全変更履歴を自動保存しています

## よくある質問・トラブル

| 症状 | 対処 |
|---|---|
| 「接続できません」 | ユーザー名・リポジトリ名・トークンの入力ミスが大半。トークンの権限(Contents/Actions両方RW)と対象リポジトリ(kabu-data)も確認 |
| 更新ボタンが完了しない | kabu-dataの「Actions」タブで実行ログを確認。初回はActionsの有効化(手順2-4)を忘れがちです |
| 定時更新が来ない | GitHubのスケジュールは数分〜数十分遅れることがあります(仕様)。長期間コミットがないリポジトリは自動更新が停止されることがあり、その場合Actionsタブで再有効化してください |
| トークンの期限切れ | 手順4で再発行 → アプリの「設定」→「接続設定を消去」→ 初回設定をやり直し |
| パスワードを忘れた | 「接続設定をやり直す」→ トークンを再入力して新パスワードを設定(GitHub上のデータは無事です) |
| 別のiPhoneやPCでも使いたい | 同じURLを開き、同じトークン(または新規発行)で初回設定すればOK |

## セキュリティについて

- 資産データはすべて非公開リポジトリにあり、トークンを持つ人しか読めません
- アプリのURL(公開ページ)を誰かに知られても、中身はただのプログラムで、パスワードとトークンがなければデータには一切アクセスできません
- トークンは kabu-data リポジトリのみに効く限定権限です。万一漏れた場合はGitHubのトークン画面から失効(Revoke)できます
