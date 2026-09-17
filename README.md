# PFCカロリーノート (Firebase + GitHub版)

Claude上のプロトタイプを、一般公開できる形に移植したものです。
Googleアカウントでログインした人ごとに、PFC(たんぱく質・脂質・炭水化物)とカロリーの記録を個別に保存します。他の人の記録は見えません。

- ログイン: Firebase Authentication (Googleログイン)
- データ保存: Firestore (利用者ごとに `users/{その人のuid}/...` 以下に保存、本人以外はアクセス不可)
- 公開先: Firebase Hosting
- デプロイ方法: GitHubの `main` ブランチにpushすると、GitHub Actionsが自動的にFirebase Hostingへ反映

以下の手順を上から順番に行ってください。すべて無料枠の範囲で始められます。

## 1. Firebaseプロジェクトを作る

1. https://console.firebase.google.com/ を開き、Googleアカウントでログイン
2. 「プロジェクトを追加」→ 好きなプロジェクト名を入力(例: `pfc-calorie-note`)→ 案内に従って作成

## 2. Googleログインを有効にする

1. 左メニュー「構築」→「Authentication」→「始める」
2. 「Sign-in method」タブ →「Google」を選び、有効にして保存

## 3. Firestore Databaseを有効にする

1. 左メニュー「構築」→「Firestore Database」→「データベースの作成」
2. 本番環境モードを選択、ロケーションは `asia-northeast1`(東京)などお好きな場所を選択して作成

作成できたら、「ルール」タブを開き、このリポジトリの `firestore.rules` の中身を丸ごと貼り付けて「公開」してください。これにより、本人以外は自分のデータを読み書きできないよう制限されます。

## 4. Hostingを有効にする

左メニュー「構築」→「Hosting」→「始める」を選び、案内に従って有効化してください(実際のアップロードはGitHub Actionsが行うので、ここでは有効化するだけでOKです)。

## 5. ウェブアプリを登録し、設定値を取得する

1. プロジェクトの概要ページ →「</>」(ウェブ)のアイコンをクリック
2. アプリのニックネームを適当に入力して「アプリを登録」
3. 表示される `firebaseConfig` の値(apiKey, authDomain, projectId, storageBucket, messagingSenderId, appId)をコピー
4. `public/index.html` を開き、`firebaseConfig` の部分(`YOUR_API_KEY` などプレースホルダーの箇所)を、コピーした実際の値に書き換える

## 6. プロジェクトIDを設定ファイルに反映する

以下の2箇所にある `YOUR_FIREBASE_PROJECT_ID` を、実際のFirebaseプロジェクトIDに書き換えてください(手順1で決めたプロジェクト名がそのままIDになっていることが多いです。Firebaseコンソールの「プロジェクトの設定」で確認できます)。

- `.firebaserc`
- `.github/workflows/firebase-hosting-merge.yml`

## 7. GitHubリポジトリを作る

1. https://github.com/new で新規リポジトリを作成(Public/Privateどちらでも動作は変わりません)
2. このフォルダの中身一式を、そのリポジトリにpushしてください

```bash
git init
git add .
git commit -m "first commit"
git branch -M main
git remote add origin https://github.com/<あなたのアカウント>/<リポジトリ名>.git
git push -u origin main
```

## 8. サービスアカウントキーをGitHubに登録する

GitHub Actionsが、あなたに代わってFirebaseへデプロイできるようにするための鍵です。

1. Firebaseコンソール→ 画面左上の歯車アイコン →「プロジェクトの設定」→「サービス アカウント」タブ
2. 「新しい秘密鍵の生成」→ ダウンロードされるJSONファイルを開き、中身を全部コピー
3. GitHubのリポジトリ画面 →「Settings」→「Secrets and variables」→「Actions」→「New repository secret」
4. Name: `FIREBASE_SERVICE_ACCOUNT`、Secret: 先ほどコピーしたJSONの中身を貼り付けて保存

## 9. 完了・公開URLの確認

手順7で `main` ブランチにpushした時点で、GitHub Actionsが自動的にビルド・デプロイを行います(リポジトリの「Actions」タブで進み具合を確認できます)。

数分後、以下のURLでアクセスできるようになります。

```
https://<あなたのFirebaseプロジェクトID>.web.app
```

以降は、`main` ブランチにpushするたびに自動的に反映されます。

## 補足

- 料金: Firebase Hosting・Authentication・FirestoreはいずれもSparkプラン(無料)の範囲内で、個人利用〜小規模な公開であれば十分足ります。
- ログイン方法を増やしたい場合(メール/パスワードなど): Firebase Authenticationの「Sign-in method」から他の方法も有効化し、`public/index.html` 内のログイン処理を追加してください。
- 見た目やPFCの計算ロジックは、これまでClaude上で作っていたものと同じです。変更したい場合は `public/index.html` を直接編集してpushしてください。
