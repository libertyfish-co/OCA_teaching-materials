## 13.6 Ruby on Rails：ECサイトの開発 Herokuへのデプロイ1

###	13.6.1 Herokuについて

アプリケーションを公開するには、サーバーにプログラムファイルを配置し、それにセキュリティなどを考慮して動く状態にする必要があります。これらの作業をデプロイといいます。
今までのデプロイは、サーバー用のPCを用意し、OSや使用する言語をインストールし、セキュリティーなどの設定を追加し、そのうえでプログラムファイルを設置して動作確認をするという、とても手間がかかるものでした。
しかし、近年のクラウドサービスの進歩は目覚ましいものがあり、ブラウザから管理画面にログインして表示されるボタンをいくつかクリックしたり、コマンドを数行実行したりするだけでデプロイができてしまうというものが主流になりつつあります。
その中でHerokuは、PaaS（Platform as a Service）と呼ばれるクラウドサービスで、ローカルのターミナルから直接Gitコマンドでデプロイを行ったり、Add-onsと呼ばれるいろいろな拡張機能をクリックするだけでそれらの機能が追加できるという特徴があります。Herokuが用意したGitリポジトリに`git push`することで、Dynoと呼ばれるLinuxコンテナが作られアプリケーションをインストールします。デフォルトのままであれば、そのとき同時にDBコンテナも用意されます。Dynoはアプリケーションの規模によって拡張され、同時に課金の対象となります。

#### Dynoとは

個々のアプリケーションが動作するための独立した環境です。アプリケーションを動作させるためにはOSが必要です。HerokuはPaaSですから、OSはあらかじめ用意されているものから選択します。

  （参考：<https://www.heroku.com/dynos>）

#### スタックとは

dynoなどで動作するOSのイメージです。2019年12月現在では、以下のイメージがあります。

|Stack Version|Base Technology|
|:--|:--|
|Heroku-18 (default)|Ubuntu 18.04|
|Heroku-16|Ubuntu 16.04|
|Container|Docker|

  （参考：<https://devcenter.heroku.com/articles/stack>）


気をつけなければいけないのは、このDynoはデプロイごとに作り直されてしまうということです。例えば、Railsのプロジェクトディレクトリ内に画像データなどを蓄積するものがありますが、デプロイすると削除されてしまいますので、その場合はAmazon S3など外部サービスと連携させる必要があります。一方、DBコンテナは一番最初にアプリケーションをHerokuに置いたときに作られますが、自動的にRailsのmigraionファイルを適応しません。手動でコマンドを実行することになります。ただ、DBコンテナはアプリケーションを削除するまで消されることはありません。

### 13.6.2 Herokuへのリリース準備

Herokuのアカウントを持っていない人は、以下のサイトのSign upからアカウントを作成してください。アカウントの作成は無料です。今回のアプリケーションのレベルでは課金されることはありませんのでご安心ください。また、ログインIDとパスワードはデプロイのときに必要ですのですぐわかるようにしておいてください。

<https://www.heroku.com/>

では、Herokuへのリリースの準備をしましょう。  
（参考：<https://devcenter.heroku.com/articles/getting-started-with-rails5>）

Herokuでアプリケーションにアクセスした際に初期表示するページを設定します。  

`config/routes.rb` を開いて、表示したい任意のページを設定してください。
``` ruby
Rails.application.routes.draw do
  root 'books#index' # 追加例
end
```

データベースの変更をします。Herokuでは、デフォルトのデータベースはPostgreSQLです。

`config/datagase.yml` を開いて、production環境のデータベース指定を変更してください。

    production:
      adapter: postgresql
      database: heroku_production # わかりやすい名前で自由につけてかまいません

それから、PostgreSQLを使用するためのgemを追加しなければいけません。ただし、developmentモードとtestモードではこのままSQLiteを利用しますので、gem sqlite3はdevelopment/test専用とし、gem pgはproduction専用になるように変更してください。  
また、Gemfileの先頭でRubyのversionも指定しましょう。今回は、2.7.4で記載していますが、ここのversionは開発しているversionを指定してください。versionを指定しない場合は、herokuのデフォルトのバージョンになります。何か問題が生じた場合の切り分けや、開発環境の確認のためにも明示的に記載したほうが良いでしょう。

`Gemfile`
``` ruby
    ruby '2.7.4'
         ・
         ・
    group :development, :test do
      # Use sqlite3 as the database for Active Record # 移動
      gem 'sqlite3', '~> 1.4' # 移動
    end

    group :production do # 追加
      gem 'pg' # 追加
    end # 追加
```

このあと、 `bundle install` を忘れずに実行してコミットまでしておいてください。
(補足)
`bundle install` 時にエラーが表示された場合、`sudo apt install libpq-dev` を実行して、再試行してください。
