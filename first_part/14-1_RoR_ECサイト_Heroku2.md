## 14.1 Ruby on Rails：ECサイトの開発 Herokuへのデプロイ2

### 14.1.1 Herokuへの公開

#### (a) Herokuコマンドの紹介

Herokuへのデプロイはとてもシンプルです。コマンドを順番に実行するだけでデプロイができてしまいます。以下の通りにターミナルへ入力していってください。
（参考：<https://devcenter.heroku.com/articles/getting-started-with-rails5>）

  ①　Heroku CLIのインストール

Herokuアプリをターミナルから操作するためHeroku CLIをインストールします。

    username:~/workspace (master) $ sudo snap install --classic heroku

以下のコマンドでインストールされているか確認してみましょう。

    username:~/workspace (master) $ heroku --version
    heroku/7.59.3 linux-x64 node-v12.21.0

　②　Herokuへのログイン

ターミナルからHerokuへログインしておきます。これであとのHerokuコマンドが利用できるようになります。

    username:~/workspace (master) $ heroku login -i
    Enter your Heroku credentials.
    Email: your@email.com
    Password (typing will be hidden):
    Logged in as your@email.com

(補足)  
Herokuコマンドを実行した際に、以下のようなエラーが表示される場合があります。  

    (node:3083) SyntaxError Plugin: heroku: /home/username/.local/share/heroku/config.json: Unexpected end of JSON input

`/home/(username)/.local/share/heroku/config.json` に何も記述がないことによるエラーですので、  
ファイルを開き、波括弧を追加することで解消されます。
``` json
{

}
```

　③　アプリケーション公開の初期設定

`app-name` の部分にアプリケーションの名前を入力してください。これは、Herokuのダッシュボードで識別するためのもので、Herokuが用意するurlにも反映されます。railsのプロジェクト名と同じである必要はありません。自由に名前をつけることができます。もし他のプロジェクトと名前が重複している、または大文字や禁止記号を使用した場合はメッセージが出ますので、その場合は違う名前で実行しなおしてください。

    usernamey:~/workspace (master) $ heroku create app-name
    Creating ⬢ app-name... done
    https://app-name.herokuapp.com/ | https://git.heroku.com/app-name.git

この中の `https://app-name.herokuapp.com` がこのアプリケーションに割り当てられたURLです。
これが終わると、gitのリモートリポジトリとしてのHerokuが追加されます。

    usernamey:~/workspace (master) $ git remote -v
    heroku  https://git.heroku.com/app-name.git (fetch)
    heroku  https://git.heroku.com/app-name.git (push)

　④　Herokuへのデプロイ

デプロイは、gitコマンドでHerokuのリモートリポジトリへpushするだけです。push先のブランチ名は必ず `master` にしてください。

    usernamey:~/workspace (master) $ git push heroku master
    Counting objects: 478, done.
    Delta compression using up to 8 threads.
    Compressing objects: 100% (440/440), done.
    Writing objects: 100% (478/478), 105.43 KiB | 0 bytes/s, done.
    Total 478 (delta 207), reused 0 (delta 0)
    remote: Compressing source files... done.
    remote: Building source:
    remote:
    remote: -----> Ruby app detected
    remote: -----> Compiling Ruby/Rails
    remote: -----> Using Ruby version: ruby-2.2.6
    remote: -----> Installing dependencies using bundler 1.13.6
    remote:        Running: bundle install
                    --without development:test
                    --path vendor/bundle
                    --binstubs vendor/bundle/bin -j4 --deployment
    remote:        Fetching gem metadata from https://rubygems.org/.........
    remote:        Fetching version metadata from https://rubygems.org/..
    remote:        Fetching dependency metadata from https://rubygems.org/.
    remote:        Installing i18n 0.7.0
    remote:        Installing rake 12.0.0
    (中略)
    remote: -----> Discovering process types
    remote:        Procfile declares types     -> (none)
    remote:        Default types for buildpack -> console, rake, web, worker
    remote:
    remote: -----> Compressing...
    remote:        Done: 30M
    remote: -----> Launching...
    remote:        Released v5
    remote:        https://app-name.herokuapp.com/ deployed to Heroku
    remote:
    remote: Verifying deploy... done.
    To https://git.heroku.com/app-name.git
     * [new branch]      master -> master

　⑤　アプリケーションのデータベース設定

データベースの反映は自動的には行われません。 `db:migrate` を実行しましょう。2回目以降、アプリケーションの修正などでデータベースの反映がなければ、これを実行する必要はありません。

    usernamey:~/workspace (master) $ heroku run rails db:migrate
    Running rake db:migrate on ⬢ app-name... up, run.4419 (Free)
    == 20161210052143 CreateBooks: migrating =================================
    -- create_table(:books)
       -> 0.0085s
    == 20161210052143 CreateBooks: migrated (0.0086s) ========================

    == 20161210052334 CreateTags: migrating ==================================
    -- create_table(:tags)
       -> 0.0078s
    == 20161210052334 CreateTags: migrated (0.0079s) =========================

    == 20161210052429 CreateTaggings: migrating ==============================
    -- create_table(:taggings)
       -> 0.0227s
    == 20161210052429 CreateTaggings: migrated (0.0228s) =====================

    == 20161219102255 DeviseCreateUsers: migrating ===========================
    -- create_table(:users)
       -> 0.0114s
    -- add_index(:users, :email, {:unique=>true})
       -> 0.0078s
    -- add_index(:users, :reset_password_token, {:unique=>true})
       -> 0.0085s
    == 20161219102255 DeviseCreateUsers: migrated (0.0280s) ==================

    == 20161221085504 AddColumnsToUser: migrating ============================
    -- add_column(:users, :name, :string)
       -> 0.0016s
    -- add_column(:users, :role, :string)
       -> 0.0014s
    == 20161221085504 AddColumnsToUser: migrated (0.0033s) ===================

これでHeorkuでのデプロイが完成しました。これでHerokuに割り当てられたURLからアプリケーションにアクセスすることができるかどうか、確認してください。

期待した画面が表示されていれば確認できたらデプロイ完了です。おめでとうございます！

#### (b) Herokuのその他機能の紹介

ここでは、運用、デバッグしていく際によく使うHerokuコマンドをご紹介します。コマンドのオプションなどが変更される可能性もありますので、最新情報はHerokuの開発者向けホームページを参照ください。

<https://devcenter.heroku.com>

　①　環境変数の一覧

デフォルトでの一覧は以下のとおりです。 `heroku config:set` などで、ここに任意の環境変数を追加することもできます。詳しくは `heroku help config` を見てください。

    username:~/workspace (master) $ heroku config
    === app-name Config Vars
    DATABASE_URL:             postgres://(何かの羅列).amazonaws.com:(何かの羅列)
    LANG:                     en_US.UTF-8
    RACK_ENV:                 production
    RAILS_ENV:                production
    RAILS_LOG_TO_STDOUT:      enabled
    RAILS_SERVE_STATIC_FILES: enabled
    SECRET_KEY_BASE:          (何かの羅列)

　②　ログ閲覧

デフォルトでは直近から1500行のログが保存されています。それ以上保存したい場合は、Add-onsで拡張機能を利用してください。
もしくは、ブラウザの管理画面でもログを見ることができます。`--tail` オプションをつけるとアプリケーションへのアクセスログがインクリメンタルに見ることができます。

<https://dashboard.heroku.com/apps/app-name/logs>

    username:~/workspace (master) $ heroku logs --tail
    2016-12-24T08:14:44.612193+00:00 app[api]:
      Initial release by user test@example.com
    2016-12-24T08:14:44.841327+00:00 app[api]:
      Enable Logplex by user test@example.com
    2016-12-24T08:14:44.612193+00:00 app[api]:
      Release v1 created by user test@example.com
    2016-12-24T08:14:44.841327+00:00 app[api]:
      Release v2 created by user test@example.com
    2016-12-24T08:16:17.196667+00:00 app[api]:
      Set LANG, RACK_ENV, RAILS_ENV, RAILS_LOG_TO_STDOUT,
      RAILS_SERVE_STATIC_FILES,
      SECRET_KEY_BASE config vars by user test@example.com
    2016-12-24T08:16:17.196667+00:00 app[api]:
      Release v3 created by user test@example.com
    2016-12-24T08:16:18.396611+00:00 app[api]:
      Release v4 created by user test@example.com
    2016-12-24T08:16:18.396611+00:00 app[api]:
      Attach DATABASE (@ref:postgresql-rugged-32907) by user test@example.com
    2016-12-24T08:16:19.247083+00:00 heroku[slug-compiler]:
      Slug compilation started
    2016-12-24T08:16:19.247093+00:00 heroku[slug-compiler]:
      Slug compilation finished
    2016-12-24T08:16:18.927856+00:00 app[api]:
      Deploy 69acfa3 by user test@example.com
    2016-12-24T08:16:18.927856+00:00 app[api]:
      Release v5 created by user test@example.com
    2016-12-24T08:16:22.171682+00:00 heroku[web.1]:
      Starting process with command `bin/rails server -p 27764 -e production`
    ....

　③　アプリケーションのリスタート

Dyno（Linuxコンテナ）をリスタートします。

    username:~/workspace (master) $ heroku restart
    Restarting dynos on ⬢ app-name... done

　④　railsコンソールの起動

Dyno（Linuxコンテナ）でアプリケーションのコンソールを起動します。

    usernmae:~/workspace (master) $ heroku run rails console
    Running rails console on ⬢ app-name... up, run.1384 (Free)
    Loading production environment (Rails 5.0.0.1)
    irb(main):001:0>

　⑤　bashの起動

Dyno（Linuxコンテナ）でターミナルを起動します

    username:~/workspace (master) $ heroku run bash
    Running bash on ⬢ app-name... up, run.6008 (Free)
    ~ $ ls
    app  config  db  Gemfile.lock  log  Rakefile  README.rdoc  test  
    vendor  bin  config.ru  Gemfile  lib  public  README.md  spec tmp

　⑥　データバックアップ

PostgreSQLを使用した場合のデータのバックアップ方法です。
まず、データセットを作成します。

    username:~/workspace (master) $ heroku pg:backups capture
    Use Ctrl-C at any time to stop monitoring progress; the backup
    will continue running. Use heroku pg:backups info to check progress.
    Stop a running backup with heroku pg:backups cancel.

    DATABASE ---backup---> b001

    Backup completed

そうすると、下記のようにIDが付けられたバックアップファイルが作成されます。バックアップファイルは最大で2つまで保存され、古いものから順に削除されていきます。

    username:~/workspace (master) $ heroku pg:backups
    === Backups
    ID    Backup Time                Status                               
    ----  -------------------------  -----------------------------------  
    b001  2016-12-27 07:41:11 +0000  Completed 2016-12-27 07:41:15 +0000  

    Size    Database
    ------  --------
    14.8kB  DATABASE

    === Restores
    No restores found. Use `heroku pg:backups restore` to restore a backup

    === Copies
    No copies found. Use `heroku pg:copy` to copy a database to another

ローカルでHerokuコマンドを使用している場合、ここで `heroku pg:backups:download` とすれば勝手にダンプファイルのダウンロードができます。しかし、Cloud9では制限されているため、バックアップ保存先のurlを取得し、そこに直接アクセスすることになります。

    username:~/workspace (master) $ heroku pg:backups public-url b001
    The following URL will expire at 2016-12-27 09:35:29 +0000:
      "https://xxxx.s3.amazonaws.com/xxxxx"

httpsで始まるAmazon S3の長いURLが表示されます。そこにアクセスするとダンプファイルをダウンロードすることができます。

　　⑦　データリストア

データのリストアでも制限があり、バックアップIDがついたものを直接リストアすることしかできません。ですので、Herokuで運用する場合は、すべてHeroku上で解決する方法が必要です。リストアのコマンドは以下の通りです。

    username:~/workspace (master) $ heroku pg:backups restore b003 DATABASE_URL

     !    WARNING: Destructive Action
     !    This command will affect the app: app-name
     !    To proceed, type "app-name" or re-run this command with
     !    --confirm app-name

    > app-name
    Use Ctrl-C at any time to stop monitoring progress; the backup
    will continue restoring. Use heroku pg:backups to check progress.
    Stop a running restore with heroku pg:backups cancel.

    r005 ---restore---> DATABASE
    Restore completed

`restore` のあとに、バックアップIDと、環境変数のDATABASE_URLまたは該当のデータベースのURLを指定します。ここでのDATABASE_URLは、先の `heroku config` で確認できるURLです。

Cloud9やHerokuなどのいわゆるクラウドサービスは進化・変化も早く、ここにあげた内容がすぐに陳腐化する可能性もあります。常に最新の情報を元に開発を進めるようにしてください。
