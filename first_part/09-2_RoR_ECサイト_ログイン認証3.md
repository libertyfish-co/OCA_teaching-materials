## 9.2 Ruby on Rails：ECサイトの開発 ログイン認証とユーザー管理3

### 9.2.1 例題(つづき)

#### ログイン中のユーザの取得

マイページにはログインしているユーザの名前などの情報を表示することがよくあります。
ここではマイページにログインしているユーザのメールアドレスを表示してみましょう。

Deviseではヘルパーメソッドが利用できます。
ログインしているユーザを取得するには`current_user`というヘルパーメソッドを利用します。

`app/views/mypage/index.html.erb`

```html
<h1>マイページ</h1>
<p>ここはマイページです</p>
<p>あなたのメールアドレスは<%= current_user.email %>です</p><!-- 追加 -->
<p><%= link_to 'ログアウト', destroy_user_session_path, data: { turbo_method: :delete } %></p>
```

ログインしてからマイページにアクセスして確認してみましょう。
メールアドレスが表示されています。


#### デザインの変更

今まで見てきたログイン画面や登録画面はDeviseのデフォルト画面です。
Deviseがインストールされているディレクトリにあるファイルなので修正することはできません。
`rails g devise:views`でviewファイルを作成することによって画面の編集ができます。  

これらのデザインを変更する方法を見ていきましょう。  

```sh
$ rails g devise:views
$ ls app/views/devise/
confirmations/  mailer/  passwords/  registrations/  sessions/  shared/  unlocks/
```

各機能で利用しているviewファイルが作成されました。


まずはログイン画面のviewを開いて見ましょう。  
<http://localhost:3000/users/sign_in>

次に先ほどのページを編集します。
ログイン画面のviewは`app/views/devise/sessions/new.html.erb`です。

`app/views/devise/sessions/new.html.erb`
```html
<h2>ログイン</h2> <!-- 編集 -->

<%= form_for(resource, as: resource_name, url: session_path(resource_name)) do |f| %>
  <div class="field">
    <%= f.label :email %><br />
    <%= f.email_field :email, autofocus: true, autocomplete: "email" %>
  </div>
・
・
・
```

再度ログイン画面にアクセスしましょう。  

![画像](images/09-1-1-2.png)  

![画像](images/09-2-1-3.png)  

今まで英語で表示されていた`Log in`が日本語で`ログイン`と表示されました。

このようにDeviseが提供している画面を修正するには`devise:views`のジェネレータを使ってviewファイルを作成し、そのファイルを修正します。
