# Instagram 毎日自動投稿ボット

`images/` フォルダに入れた写真から、**毎日1枚をランダムに選び、同じ投稿文で Instagram に自動投稿**します。

投稿した写真は、自動で `posted/` フォルダに移動します。

GitHub Actions で動くので、**PC を起動しておく必要はありません。料金もかかりません。**

---

## 目次

1. [先に知っておくこと](#先に知っておくこと)
2. [準備するもの](#準備するもの)
3. [Git の基礎](#git-の基礎)
4. [手順1: Instagram をプロアカウントにする](#手順1-instagram-をプロアカウントにする)
5. [手順2: Meta アプリを作り、トークンとユーザーIDを取る](#手順2-meta-アプリを作りトークンとユーザーidを取る)
6. [手順3: このテンプレートから自分のリポジトリを作る](#手順3-このテンプレートから自分のリポジトリを作る)
7. [手順4: Secrets を3つ登録する](#手順4-secrets-を3つ登録する)
8. [手順5: 写真を追加して動作テストをする](#手順5-写真を追加して動作テストをする)
9. [毎日の使い方](#毎日の使い方)
10. [投稿の成功・失敗を通知する](#投稿の成功失敗を通知する)
11. [困ったとき](#困ったとき)
12. [仕組み](#仕組み)
13. [参考URL](#参考url)

---

## 先に知っておくこと

### プロアカウントが必要です

公式 API で投稿できるのは、プロ（クリエイター／ビジネス）アカウントだけです。

プロアカウントは**鍵をかけられません**（公開になります）。

鍵付きの個人アカウントには使えません。

### 写真は公開されます

Instagram に写真を渡すために、GitHub の**公開リポジトリ**に写真を置きます。

まだ投稿していない写真も、URL を知っている人は見られます。

見せたくない写真は入れないでください。

### Meta アプリとトークンは、各自で作ります

アクセストークンは、他の人と共有できません。

自分のアプリと自分のトークンを使ってください。

### 写真の条件

- 形式: **JPEG**
- 縦横比: **4:5 〜 1.91:1**
- サイズ: **8MB 以下**

`add.py` で写真を追加すれば、縦横比は自動で整います（手順5で説明します）。

---

## 準備するもの

次のものを用意してください。

- Instagram のアカウント（手順1でプロに切り替えます）
- GitHub のアカウント（無料）: <https://github.com/signup>
- Facebook のアカウント（Meta の開発者登録に使います）
- Windows / Mac の PC
- Git: <https://git-scm.com/downloads>
- Python 3: <https://www.python.org/downloads/>

Git の操作が不安な方は、画面で操作できる GitHub Desktop も使えます: <https://desktop.github.com/>

---

## Git の基礎

このボットは Git と GitHub を使います。

最低限の用語と操作を説明します。

### 用語

| 用語 | 意味 |
|---|---|
| Git | ファイルの変更履歴を記録する仕組み |
| GitHub | Git で管理したファイルを、ネット上に置くサービス |
| リポジトリ | 1つのプロジェクトのフォルダ（履歴を含む） |
| コミット | 変更を1つの記録として保存すること |
| push（プッシュ） | 自分の PC のコミットを、GitHub に送ること |
| pull（プル） | GitHub のコミットを、自分の PC に取り込むこと |
| clone（クローン） | GitHub のリポジトリを、自分の PC にコピーすること |

### 最初の1回だけ: 名前とメールを設定する

コミットには、作った人の名前とメールが記録されます。

次のコマンドを、PowerShell（Mac はターミナル）で実行してください。

```powershell
git config --global user.name "あなたの名前"
git config --global user.email "あなたのメール"
```

メールを公開したくない場合は、GitHub の noreply アドレスを使えます（GitHub の Settings → Emails で確認できます）。

### よく使う操作

```powershell
git clone https://github.com/アカウント名/リポジトリ名.git   # GitHub から PC にコピーする
git status                                                   # 変更されたファイルを確認する
git add .                                                    # 変更を、コミットする対象に入れる
git commit -m "変更の説明"                                   # 変更を記録する
git push                                                     # GitHub に送る
git pull                                                     # GitHub の最新を取り込む
```

### 複数の GitHub アカウントを使い分けるとき

普段のアカウントとは別のアカウントで push したい場合は、そのフォルダだけに設定を入れます。

```powershell
cd このボットのフォルダ
git remote set-url origin https://使いたいアカウント名@github.com/使いたいアカウント名/リポジトリ名.git
git config --local user.name "使いたいアカウント名"
git config --local user.email "使いたいアカウントのメール"
```

初回の push でログイン画面が出るので、使いたいアカウントでサインインします。

ブラウザが別のアカウントでログイン済みなら、シークレットウィンドウで開いてください。

### push で 403 エラーになったとき

保存されている GitHub の認証情報が、別のアカウントのものになっています。

1. Windows の「資格情報マネージャー」→「Windows 資格情報」を開きます。
2. `git:https://github.com` の項目を削除します。
3. もう一度 `git push` を実行し、出てきたログイン画面で正しいアカウントにサインインします。

### 詳しく学ぶには

- Pro Git（日本語・無料）: <https://git-scm.com/book/ja/v2>
- Git の初期設定（GitHub Docs）: <https://docs.github.com/ja/get-started/git-basics/set-up-git>

---

## 手順1: Instagram をプロアカウントにする

1. Instagram アプリを開きます。
2. 設定 → 「アカウントの種類とツール」を開きます。
3. 「プロアカウントに切り替える」を選びます。
4. 種類は**クリエイター**がおすすめです。

無料で、いつでも元に戻せます。

---

## 手順2: Meta アプリを作り、トークンとユーザーIDを取る

### 2-1. アプリを作る

1. <https://developers.facebook.com> にログインします。
2. 「マイアプリ」→「アプリを作成」を押します。
3. アプリ名と連絡先メールを入力します。
4. ユースケースは、「**Instagram でメッセージとコンテンツを管理**」を選びます。
5. 別の画面が出たら、「その他」→ アプリの種類「**ビジネス**」を選びます。
6. ビジネスポートフォリオの連携を聞かれたら、スキップして構いません。

### 2-2. トークンを生成する

1. 左メニューの **Instagram → 「Instagram ログインによる API 設定」** を開きます。
   - 「Facebook ログイン」版ではありません。
2. 「アクセストークンを生成」のセクションにある **「アカウントを追加」** を押します。
3. プロアカウントでログインし、許可します。
4. 「テスターとして招待が必要」と出たら、次の操作をします。
   - アプリの「アプリの役割 → ロール」で、自分を Instagram テスターに追加します。
   - Instagram アプリの 設定 → ウェブサイトのアクセス許可 → テスターの招待 で、承認します。
5. 接続したアカウントの横にある **「トークンを生成」** を押します。
6. もう一度ログインして許可します。
7. 表示された長い文字列をコピーします。これが **`IG_TOKEN`** です。

有効期間は60日ですが、ボットが毎日自動で延長します。

### 2-3. ユーザーIDを調べる

PowerShell で次を実行します（`トークン` は、コピーしたものに置き換えます）。

```powershell
curl.exe "https://graph.instagram.com/v23.0/me?fields=user_id,username&access_token=トークン"
```

返ってきた JSON の `user_id` の数字が、**`IG_USER_ID`** です。

`username` が自分のアカウント名になっていれば成功です。

> **トークンは、投稿するための鍵です。**
>
> チャット、スクリーンショット、リポジトリのファイルには貼らないでください。
>
> 貼る場所は、手順4の Secrets だけです。

---

## 手順3: このテンプレートから自分のリポジトリを作る

1. このページの緑色のボタン **「Use this template」** を押します。
2. 「Create a new repository」を選びます。
3. リポジトリ名を付けます（例: `insta-bot`）。
4. **Public** を選びます。
5. 「Create repository」を押します。

自分の PC に取り込みます（コマンドで写真を追加する場合に必要です）。

```powershell
git clone https://github.com/自分のアカウント名/insta-bot.git
cd insta-bot
```

詳しくは: <https://docs.github.com/ja/repositories/creating-and-managing-repositories/creating-a-repository-from-a-template>

---

## 手順4: Secrets を3つ登録する

Secrets は、GitHub が安全に保管する「秘密の設定値」です。

### 4-1. `GH_PAT` を作る

`GH_PAT` は、ボットがトークンを自動更新するための、GitHub 用の鍵です。

1. GitHub の右上のアイコン → **Settings** を開きます。
2. 左メニューの一番下の **Developer settings** を開きます。
3. **Personal access tokens → Fine-grained tokens** を開きます。
4. 「Generate new token」を押します。
5. Token name は、何でも構いません（例: `insta-bot-secrets`）。
6. Expiration（期限）は、長め、または無期限にします。
7. Repository access は、「Only select repositories」を選び、**手順3で作ったリポジトリだけ**を指定します。
8. Repository permissions の **Secrets** を、**Read and write** にします。
9. 「Generate token」を押します。
10. 表示された `github_pat_...` をコピーします。

この画面を閉じると、二度と見られません。

詳しくは: <https://docs.github.com/ja/authentication/keeping-your-account-and-data-secure/managing-your-personal-access-tokens>

### 4-2. リポジトリに登録する

1. 自分のリポジトリのページを開きます。
2. **Settings → Secrets and variables → Actions** を開きます。
3. 「New repository secret」を押します。
4. 次の3つを、1つずつ登録します。

| Name | 入れる値 |
|---|---|
| `IG_TOKEN` | 手順2でコピーしたアクセストークン |
| `IG_USER_ID` | 手順2で調べた `user_id` の数字 |
| `GH_PAT` | 4-1 でコピーした `github_pat_...` |

Name は、大文字小文字も含めて、表のとおりに入力してください。

詳しくは: <https://docs.github.com/ja/actions/security-for-github-actions/security-guides/using-secrets-in-github-actions>

---

## 手順5: 写真を追加して動作テストをする

### 5-1. 投稿文を決める

`caption.txt` を開き、毎回付けたい投稿文に書き換えます。

### 5-2. 写真を追加する

初回だけ、Pillow（画像を扱うライブラリ）を入れます。

```powershell
pip install pillow
```

写真を追加します。

```powershell
python add.py C:\Users\自分\Pictures\写真.jpg        # 1枚
python add.py C:\Users\自分\Pictures\写真フォルダ     # フォルダごと
```

`add.py` は、次のことを自動で行います。

1. 写真の向きを正しくそろえる。
2. 縦横比が合わない写真は、中央を切り抜く。
3. JPEG にして `images/` に入れる。
4. コミットして、GitHub に push する。

ブラウザの GitHub 画面から `images/` に直接アップロードすることもできます。

ただしその場合は、縦横比の自動調整が行われません。

### 5-3. 手動で実行してテストする

1. 自分のリポジトリの **Actions** タブを開きます。
2. 左の一覧から **post** を選びます。
3. 「**Run workflow**」を押します。
4. 緑のチェックが付けば成功です。
5. Instagram に投稿され、写真が1枚 `posted/` に移動していることを確認します。

詳しくは: <https://docs.github.com/ja/actions/managing-workflow-runs-and-deployments/managing-workflow-runs/manually-running-a-workflow>

---

## 毎日の使い方

何もしなくても、毎日 **日本時間の9:00ごろ**に自動で投稿されます。

GitHub の定期実行は、数分〜数十分遅れることがあります。

### やること

- 写真が減ってきたら、`add.py` で追加します。
- 投稿文を変えたいときは、`caption.txt` を書き換えて push します。

### 投稿時刻を変える

`.github/workflows/post.yml` の `cron` の行を編集します。

```yaml
- cron: "0 0 * * *"    # 分 時 日 月 曜日（UTC）
```

時刻は **UTC** で指定します。日本時間から9時間引いてください（例: 日本時間 21:00 → UTC 12:00 → `"0 12 * * *"`）。

書き方の確認には、このサイトが便利です: <https://crontab.guru/>

### 投稿済みの写真をもう一度使う

```powershell
git mv posted/写真の名前.jpg images/
git commit -m "restore image"
git push
```

---

## 投稿の成功・失敗を通知する

GitHub のスマホアプリを入れると、投稿（Actions の実行）が終わるたびに、プッシュ通知が届きます。

コードの変更は要りません。GitHub の通知設定だけで使えます。

投稿に失敗すると Actions が赤く失敗し、失敗の通知が届きます。

### 設定

1. スマホに **GitHub アプリ**を入れ、このリポジトリを作ったアカウントでログインします。
   - 別のアカウントでログインしていると、通知は届きません。
2. アプリの Settings → Notifications で、プッシュ通知を ON にします。
3. パソコンのブラウザで <https://github.com/settings/notifications> を開き、**Actions** の項目を設定します。
   - 通知先に **GitHub Mobile** を含めます。
   - 「Send notifications for failed workflows only」のチェックを、**外す**と成功も通知されます。**入れる**と失敗だけ通知されます。

### 届かないとき

- 定期実行の通知は、`post.yml` の `cron` の行を**最後に編集した人**に届きます。cron を変えたときは、そのアカウントでアプリにログインしてください。
- 確認するには、Actions タブから **Run workflow** で手動実行します。投稿も実際に行われるので、投稿してよいときに試してください。

詳しくは: <https://docs.github.com/ja/account-and-profile/managing-subscriptions-and-notifications-on-github/setting-up-notifications/configuring-notifications>

---

## 困ったとき

| 症状 | 確認すること |
|---|---|
| Actions が赤く失敗する | 失敗したステップのログを開き、エラー文を読みます。 |
| `API error 400` などで投稿されない | 写真が条件（JPEG、4:5〜1.91:1、8MB以下）を満たしているか確認します。`add.py` を通した写真か確認します。 |
| `API error 190` / `API error 200` | トークンが無効か、アプリが制限されています。ログの末尾に対処のヒントが出ます。Meta の開発者ダッシュボードでアプリの状態と通知を確認し、手順2-2 でトークンを作り直して `IG_TOKEN` を更新します。そのあと **Run workflow** で再実行します。 |
| `warning: token refresh failed` と出る | トークンの延長に失敗した警告です。投稿は続行されます。続くようなら、トークンを作り直します。 |
| `IG_TOKEN rotation failed` と出る | `IG_TOKEN` を自動更新できませんでした。`GH_PAT` の **Secrets** の権限が **Read and write** か、期限が切れていないか確認します。 |
| `images/ is empty` と出て投稿されない | 写真を使い切っています。`add.py` で追加します。 |
| push で 403 になる | 「Git の基礎」の「push で 403 エラーになったとき」を参照します。 |
| 「テスターとして招待が必要」と出る | 手順2-2 の4を参照します。 |

---

## 仕組み

1. `post.py` が、`images/` から写真を1枚ランダムに選びます。
2. 公式の Instagram API に、写真の公開URLと投稿文を渡して、投稿します。
3. 成功したら、写真を `posted/` に移動します。
4. GitHub Actions が、その移動をコミットして push します。
5. 実行のたびにトークンを延長し、新しいトークンで `IG_TOKEN` を自動更新します（`GH_PAT` を使います）。

補足です。

- 前回の延長から24時間未満だと、延長は拒否されます。その場合も、投稿は続行されます。
- トークンの延長や `IG_TOKEN` の自動更新が失敗しても、投稿は成功扱いのままです。ログに警告が出るだけなので、Actions のログで確認できます。
- API が `190`（トークン無効）や `200`（アクセス制限）を返したときは、ログに対処のヒントを出して、失敗として終了します。
- 毎日コミットが入るため、「無活動だと定期実行が止まる」GitHub の仕様を避けられます。
- 追加のライブラリは不要です。`add.py` の切り抜きだけが、手元で Pillow を使います。

---

## 参考URL

### Instagram / Meta

- Instagram API（Instagram ログイン版）の始め方: <https://developers.facebook.com/docs/instagram-platform/instagram-api-with-instagram-login/get-started>
- コンテンツの公開（投稿）API: <https://developers.facebook.com/docs/instagram-platform/instagram-api-with-instagram-login/content-publishing>

### Git / GitHub

- Git のダウンロード: <https://git-scm.com/downloads>
- Pro Git（日本語・無料）: <https://git-scm.com/book/ja/v2>
- GitHub Desktop（画面で操作できるアプリ）: <https://desktop.github.com/>
- Git の初期設定: <https://docs.github.com/ja/get-started/git-basics/set-up-git>
- テンプレートからリポジトリを作る: <https://docs.github.com/ja/repositories/creating-and-managing-repositories/creating-a-repository-from-a-template>
- Personal access token: <https://docs.github.com/ja/authentication/keeping-your-account-and-data-secure/managing-your-personal-access-tokens>
- Actions の Secrets: <https://docs.github.com/ja/actions/security-for-github-actions/security-guides/using-secrets-in-github-actions>
- ワークフローの手動実行: <https://docs.github.com/ja/actions/managing-workflow-runs-and-deployments/managing-workflow-runs/manually-running-a-workflow>

### その他

- Python のダウンロード: <https://www.python.org/downloads/>
- Pillow（画像ライブラリ）: <https://pillow.readthedocs.io/>
- cron の書き方の確認: <https://crontab.guru/>
