# docker-compose-wordpress

- docker-composeでWordPress開発環境を構築する手順書。
- 新規テーマ開発のための環境構築について記述しています。
- 復元とは切り分ける。
  - [復元時の補足はこちら](wp-restore.md)

## 実行環境

- Ubuntu-20.04
  - windows user
  - linux user（owner問題あり）
- Windows 10
- Mac OS Big Sur

## Reference source

- [Quickstart: Compose and WordPress](https://docs.docker.com/compose/wordpress/)
- [コマンドラインリファレンス](https://docs.docker.jp/compose/reference/toc.html)
- [Gulp公式：Quick Start](https://gulpjs.com/docs/en/getting-started/quick-start)
- [Docker Composeを使ってWordPressが動作するローカル環境を作る](https://codeaid.jp/blog/docker-wp/)
- [いまさら始めるGulpでWordPressテーマ開発](https://olein-design.com/blog/gulp-wp-starter)
- [絶対つまずかないGulp 4入門(2021年版)インストールとSassを使うまでの手順](https://ics.media/entry/3290/)
- [Gulp ＋ Browsersyncを使ったブラウザ自動リロードでコーディング効率化を目指す](https://designsupply-web.com/media/knowledgeside/3785/)
- [忘備録](https://sotoogre.hatenablog.jp/entry/2020/10/25/121040)

## Docker Composeを使う理由

- 環境構築の統一化が図れる。
- Docker Composeは複数のコンテナーを起動するツールであり、シンプルなコマンドで実行可能。
- WordPressを動作させるための、Webサーバー、PHP、データベースをまとめて起動できる。

## 環境構築手順

初期設定（手順0）済みならば、手順1～3を実行しテーマ開発をスタートする。

### 手順0. 準備

1. Docker Desktopをインストール。（Docker Composeは同梱されている）
2. __※only WSL※__ Docker Desktopの設定をする。
   1. docker-desktop > settings > Resources > WSL INTEGRATION > Ubuntu-20.04をONにする。
   2. VScode > ターミナル > Ubuntu-20.04(WSL) を選択する。

### 手順1. リポジトリをCloneする

- このリポジトリをCloneし、ディレクトリ名を変更。

### 手順2. Dockerコンテナーの起動

1. **コンテナー起動前の確認事項：[開発しやすくなる設定](#開発しやすくする設定)に目を通し、必要に応じて設定する。**
2. 作業フォルダー（wp-sample）に移動し、コンテナーを起動する。（コマンド例は下記）

```bash
cd wp-sample
docker-compose up -d
```

### 手順3. WordPressの設定

1. Dockerコンテナー起動後、ブラウザで「localhost:8000」にアクセスする。
2. WordPressのセットアップを実行する。

## ディレクトリ構成

ここまでの操作では以下となる。

```markdown
wp-sample
├── html/
├── phpmyadmin/   # volumesを追記した場合
├── wp-content/   #プラグイン操作するなら
├── my-theme/
└── docker-compose.yml
```

## Docker主要コマンド

- Docker Composeのバージョンチェック ```docker-compose --version```
- Docker Composeでコンテナー起動 ```docker-compose up -d```
- コンテナーの状態を確認する ```docker-compose ps```
- 起動した環境の停止・削除 ```docker-compose down```
- 停止・削除・データベース削除 ```docker-compose down --volumes```
- dockerで立ち上げたコンテナーにログインする ```docker exec -it [コンテナ名] /bin/bash```
- ログインしたいコンテナー名やIDを確認する ```docker ps```

## 開発しやすくする設定

テーマ、プラグイン開発を効率よく行うための設定。

### 1. wp-contentディレクトリをマウントする

1. テーマやプラグインを直接扱えるように、作業フォルダー内にサブフォルダーを作る。
2. volumesオプションで定義する。（データが保持される）
3. マウント例↓
   - サブフォルダーhtml：WordPressファイル群すべて
   - サブフォルダーwp-content：wp-contentフォルダーのみ
   - 上記どちらか一方でもよい

```yml
  wordpress:
    volumes:
      - ./html:/var/www/html
      - ./wp-content:/var/www/html/wp-content
```

### 2. 開発用空テーマをダウンロードし、そのディレクトリをマウントする

1. [underscores.me](https://underscores.me/) で、空テーマを取得する。
   1. sassを使う場合、Advanced Optionsをクリックし、sassify!にチェックを入れる。
2. *my-theme*という名前のテーマを作成したい場合、*my-theme*と入力してGENERATEボタンをクリック（ダウンロード）する。
3. ダウンロードしたものを作業フォルダー（wp-sample）に配置する。
4. ymlファイルのwordpressサービスに追記、*my-theme*を*volumes*で*wp-contente/themes*にマウントする。

```yml
 volumes:
      - ./my-theme:/var/www/html/wp-content/themes/my-theme
```

### 3. タスクランナーの設定手順

- 【注1】**前提：タスクランナーはテーマディレクトリ（ローカル=作業フォルダー）で動かす。**
- 【注2】__*package.json*がない場合、*npm install*する前に*npm init*で*package.json*を作成する。__

1. underscoresでSassを使うためにはnpm（Node.jsのパッケージ管理ツール）が必要。
2. Nodeインストール有無の確認。```node -v```
3. テーマディレクトリに移動。```cd my-theme```
4. npm初期化でpackage.jsonを作成する。```npm init -y```
5. Gulpをローカルにインストール。```npm install -D gulp```//-Dは--save-devの短縮形（ローカル指定）

### 4. タスクの作成（document整備中）

1. 各タスクで使うGulpのプラグインをローカルにインストールする。
    1. ```npm i -D gulp```//gulp本体
    2. ```npm i -D gulp-cssmin```//CSS圧縮
    3. ```npm i -D gulp-uglify```//javascript圧縮
    4. ```npm i -D gulp-imagemin```//画像圧縮
    5. ```npm i -D gulp-sass node-sass```//Sassコンパイル
    6. ```npm i -D gulp-rename```//ファイル名リネーム
    7. ```npm i -D gulp-watch```//Sassファイル更新監視
    8. ```npm i -D gulp-plumber```//コンパイルエラー検知
    9. ```npm i -D gulp-notify```//デスクトップ通知
2. 作業フォルダー直下にgulpfile.jsを作成する。
    1. **Gulp3系とGulp4系でgulpfile.jsの記述が異なっている。4系で進める。**

### 5. タスクの実行

```npx gulp```

### 6. テーマユニットテストデータ日本語版のインポート

投稿、固定ページ、コメント、メニュー等のダミーデータをインポートする。

1. Forkした[テーマユニットテストデータ日本語版](https://github.com/chum9625/theme-test-data-ja)からwordpress-theme-test-data-ja.xmlを取得。
2. 管理画面のツール→インポートの「WordPress」からインポートを実行する。

### 7. phpMyadminを使えるようにする

- データベース操作のGUIツール：phpMyAdminを使えるようにする。

```yml
phpmyadmin:
  depends_on:
    - db
  image: phpmyadmin/phpmyadmin
  environment:
    PMA_HOST: db
  restart: always
  ports:
    - "8080:80"
```

---

## エラーの記録

実行時に起きたエラーについて記録する。

### 【Linux特有】volumeマウント時のファイルowner問題

#### 背景

- 新規ファイル作成や編集はパーミッションエラーのため不可。
- WSL2(Ubuntu-20.04) userで起きた。
- windows userでは編集可能。

#### 回避策

- windows user環境で開発する。

#### 解決策

検証中

---

## Linux Tips

### ユーザーのユーザーIDやグループIDを調べる

- ユーザーID（uid）やグループID（gid）を調べるには、idコマンドを使用する。

```bash
whoami
hoge

id hoge
uid=500（hoge） gid=501（hoge） 所属グループ=501（hoge）
```

【解説】ユーザーIDが500、グループIDが501、所属グループが501。ユーザー名と同じ名前のグループ名である。
