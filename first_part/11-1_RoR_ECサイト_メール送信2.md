## 11.1 Ruby on Rails：ECサイトの開発 メール送信2

### 11.1.1 例題

簡単なToDoリストアプリケーションを作成し、ToDoの登録時に、メールを送信できるように実装します。

#### ① スケジュール登録アプリケーションの作成

まず最初に、例題用のRailsアプリケーションを作成します。  
その後、generatorのscaffoldコマンドを使って、ToDoを保存するCRUDを作成します。

```
$ rails new MyToDoTask
```

```
$ rails generate scaffold ToDoTask title:string description:text alert_mail_address:string
Running via Spring preloader in process 2183
      invoke  active_record
      create    db/migrate/20170829090051_create_to_do_tasks.rb
      create    app/models/to_do_task.rb
      invoke    test_unit
      create      test/models/to_do_task_test.rb
      create      test/fixtures/to_do_tasks.yml
      invoke  resource_route
       route    resources :to_do_tasks
      invoke  scaffold_controller
      create    app/controllers/to_do_tasks_controller.rb
      invoke    erb
      create      app/views/to_do_tasks
      create      app/views/to_do_tasks/index.html.erb
      create      app/views/to_do_tasks/edit.html.erb
      create      app/views/to_do_tasks/show.html.erb
      create      app/views/to_do_tasks/new.html.erb
      create      app/views/to_do_tasks/_form.html.erb
      invoke    test_unit
      create      test/controllers/to_do_tasks_controller_test.rb
      invoke    helper
      create      app/helpers/to_do_tasks_helper.rb
      invoke      test_unit
      invoke    jbuilder
      create      app/views/to_do_tasks/index.json.jbuilder
      create      app/views/to_do_tasks/show.json.jbuilder
      create      app/views/to_do_tasks/_to_do_task.json.jbuilder
      invoke  test_unit
      create    test/system/to_do_tasks_test.rb
      invoke  assets
      invoke    coffee
      create      app/assets/javascripts/to_do_tasks.coffee
      invoke    scss
      create      app/assets/stylesheets/to_do_tasks.scss
      invoke  scss
      create    app/assets/stylesheets/scaffolds.scss
```

```
$ rails db:migrate
== 20170829090051 CreateToDoTasks: migrating ==================================
-- create_table(:to_do_tasks)
   -> 0.0008s
== 20170829090051 CreateToDoTasks: migrated (0.0009s) =========================

```

#### ② Action Mailerクラスとメイラービューの作成

次に、MailerクラスとViewを作成していきます。今まで通り、`rails generate`コマンドを使用して作成できます。

```
$ rails generate mailer TodoTaskMailer registration_mail
Running via Spring preloader in process 2439
      create  app/mailers/todo_mailer.rb
      invoke  erb
      create    app/views/todo_mailer
      create    app/views/todo_mailer/registration_mail.text.erb
      create    app/views/todo_mailer/registration_mail.html.erb
      invoke  test_unit
      create    test/mailers/todo_mailer_test.rb
      create    test/mailers/previews/todo_mailer_preview.rb
```

コマンドの最後に指定した`registration_mail`がToDo登録時にメールを送信するメソッド名になります。  
そして、`registration_mail`メソッドから呼び出されるビューが`app/views/todo_mailer/registration_mail.text.erb`と`app/views/order_mailer/registration_mail.html.erb`になります。
今までのControllerとViewの関係と同じように、メソッド名と同じViewファイルが作成されています。  
ファイルが2種類作成されているのは、HTML形式用とテキスト形式用の2つのフォーマットです。

#### ③ Mailerクラスの編集

Mailerクラスに送信処理を記述していきます。
`registration_mail`メソッド内に最初から記述されている不要な処理は、削除してください。

`app/mailers/todo_task_mailer.rb`

```
def registration_mail(todotask)
  @todotask = todotask
  mail to: todotask.alert_mail_address, subject: 'ToDo登録のお知らせ'
end
```

`mail`メソッドの`to`が送信先アドレス、`subject`が件名になります。

#### ④ Viewの編集

Viewは実際に送信するメール本文になります。
ここでは、ToDoのタイトルと内容を表示するようにしています。

`app/views/todo_task_mailer/registration_mail.html.erb`

```
<!DOCTYPE html>
<html>
  <head>
    <meta content='text/html; charset=UTF-8' http-equiv='Content-Type' />
  </head>
  <body>
    <h1><%= @todotask.title %></h1>
      <p>
        <%= @todotask.description %>
      </p>
  </body>
</html>
```

`app/views/todo_task_mailer/registration_mail.text.erb`

```
<%= @todotask.title %>
===============================================
<%= @todotask.description %>
```

ActionMailerは2種類(HTML/テキスト)のフォーマットが存在すると、`multipart/alternative`形式のメールを作成します。
`multipart/alternative`についての説明は省略しますが、HTML形式で表示できない環境の場合はテキスト形式で表示されるようになります。


#### ⑤ Mailerの呼び出し

Mailerクラスとメール本文(View)が作成できました。
実際にメールを送るために、`registration_mail`メソッドを呼び出す処理を追加しましょう。  

ToDo登録後に送信するメールなので、コントローラのCreateアクションで、DBの登録完了後にメール送信するようにします。

`app/controllers/to_do_tasks_controller.rb`

```
# POST /to_do_tasks
# POST /to_do_tasks.json
def create
  @to_do_task = ToDoTask.new(to_do_task_params)

  respond_to do |format|
    if @to_do_task.save
      #以下の処理を追加
      TodoTaskMailer.registration_mail(@to_do_task).deliver

      format.html { redirect_to @to_do_task,
      notice: 'To do task was successfully created.' }
      format.json { render :show, status: :created, location: @to_do_task }
    else
      format.html { render :new }
      format.json { render json: @to_do_task.errors,
      status: :unprocessable_entity }
    end
  end
end
```

#### ⑥ メールのプレビュー

Action Mailer Previewsの機能で、メールの本文を確認する事が可能です。
以下のようにソースコードを変更し、下記のアドレスにアクセスし確認してみましょう。  
http://localhost:3000/rails/mailers/todo_task_mailer/registration_mail

`test/mailers/previews/todo_task_mailer_preview.rb`

```
# Preview all emails at http://localhost:3000/rails/mailers/todo_task_mailer
class TodoTaskMailerPreview < ActionMailer::Preview

  # Preview this email at http://localhost:3000/rails/mailers/todo_task_mailer/registration_mail
  def registration_mail
    #以下のように変更
    TodoTaskMailer.registration_mail(ToDoTask.new(title:"テスト",description:"プレビュー テスト",alert_mail_address:'自身のメールアドレス'))
  end
end
```

#### ⑦ ActionMailerの設定

最後にActionMailerの設定を行います。  
各環境ファイルごとに設定が必要になります。
下記に記載しているのは、`development`環境になります。本番環境で使用するには`config/environments/production.rb`の編集が必要です。
今回はメール送信に、Gmailを使用しています。もし、設定を編集してもメールが届かない場合は、Gmail側の設定を確認してください。Gmailはセキュリティ対策のため、安全性が低いとみなしたアプリからのアクセスを拒否するようにデフォルトで設定されています。  
また、`config.action_mailer.raise_delivery_errors`を`true`にすることでメールが送信できない場合のエラーを出力することができます。

`config/environments/development.rb`

```
config.action_mailer.raise_delivery_errors = true
config.action_mailer.delivery_method = :smtp
config.action_mailer.smtp_settings = {
  address: 'smtp.gmail.com',
  port: 587,
  domain: 'gmail.com',
  user_name: ENV['MAIL_USER_NAME'],
  password: ENV['MAIL_PASSWORD'],
  authentication: 'plain',
  enable_starttls_auto: true
}
```

`user_name`と`password`の部分が、`ENV['MAIL_USER_NAME']`や`ENV['MAIL_PASSWORD']`になっています。
これはユーザ名やパスワードをそのまま設定しているとセキュリティ的に危険なため、環境変数に設定して使用します。

開発環境(Cloud9)
```
$ export MAIL_USER_NAME="monkaec@gmail.com"
$ export MAIL_PASSWORD="monkaec123"
```

本番環境(Heroku)
```
$ heroku config:add MAIL_USER_NAME="monkaec@gmail.com"
Setting MAIL_USER_NAME and restarting ⬢ sample-monka-ec2... done, v3
MAIL_USER_NAME: monkaec@gmail.com
$ heroku config:add MAIL_PASSWORD="monkaec123"
Setting MAIL_PASSWORD and restarting ⬢ sample-monka-ec2... done, v4
MAIL_PASSWORD: monkaec123
```

これで、メール送信できるようになったはずです。ToDoの登録を通知するメールアドレスを自分のアドレスなどに変更して、メールが届くか確認してみましょう。

#### ⑧ 動作確認 
それでは自身のGmailアドレスにメールが届くか確認してみましょう。

まずは、以下の記事を参考にGmailの設定で2段階認証をオンにします。

（ 参考：https://support.google.com/accounts/answer/185839?hl=ja&co=GENIE.Platform%3DDesktop ）

2段階認証の設定が完了すると、以下の記事を参考にアプリログインパスワードの取得します。

（ 参考：https://support.google.com/mail/answer/185833?hl=ja ）

16桁のアプリパスワードを取得することができたら、`config/environments/development.rb`を以下に変更しましょう。

```
config.action_mailer.raise_delivery_errors = true
config.action_mailer.delivery_method = :smtp
config.action_mailer.smtp_settings = {
  address: 'smtp.gmail.com',
  port: 587,
  domain: 'gmail.com',
  user_name: '自身のGmailアドレス', #変更
  password: '16桁のアプリログインパスワード', #変更
  authentication: 'plain',
  enable_starttls_auto: true
}
```

以上の設定が完了するとメールを送信しましょう。

`$ rails c`でコンソールを開き以下を入力しましょう。

```
irb(main):001:0> test = ToDoTask.new(title:"テスト", description:"プレビュー テスト", alert_mail_address:'自身のGmailアドレス')
irb(main):002:0> TodoTaskMailer.registration_mail(test).deliver
```

`OpenSSL::SSL::SSLError`のエラーが発生し、メールの送信ができないことがあります。本来はセキュリティの都合上、SSL証明書の検証を行ったほうが良いのですが、今回はそれを省いた設定を行います。本番環境では避けるべき設定ですので注意してください。  
以下のようにコードを修正してください。

`config/environments/development.rb`
``` ruby
require "active_support/core_ext/integer/time"

Rails.application.configure do
         ・
      ~~ 中略 ~~
         ・
  config.action_mailer.raise_delivery_errors = true
  config.action_mailer.delivery_method = :smtp
  config.action_mailer.smtp_settings = {
    address: 'smtp.gmail.com',
    port: 587,
    domain: 'gmail.com',
    user_name: '自身のGmailアドレス',
    password: '16桁のアプリログインパスワード',
    authentication: 'plain',
    openssl_verify_mode: 'none', #追加
    enable_starttls_auto: true    　
  }
end
```

コードを修正したら、再度メールを送信してみましょう。

ご自身のGmailにメールが届いていれば、設定は完了です。