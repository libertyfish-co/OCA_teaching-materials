## 5.2 Ruby on Rails：Rails基礎 3

次は、呼び出されたControllerのアクションメソッドがどういう働きをしているか見てみましょう。  

__【users_controller.rb】__  
今回使用されているControllerは、`/app/controllers/`の配下にある`users_controller.rb`です。  
この中身を確認してみましょう。  

```rb
class UsersController < ApplicationController
  before_action :set_user, only: %i[ show edit update destroy ]

  # GET /users or /users.json
  def index
    @users = User.all
  end

  # GET /users/1 or /users/1.json
  def show
  end

  # GET /users/new
  def new
    @user = User.new
  end

  # GET /users/1/edit
  def edit
  end

  # POST /users or /users.json
  def create
    @user = User.new(user_params)

    respond_to do |format|
      if @user.save
        format.html { redirect_to user_url(@user), notice: "User was successfully created." }
        format.json { render :show, status: :created, location: @user }
      else
        format.html { render :new, status: :unprocessable_entity }
        format.json { render json: @user.errors, status: :unprocessable_entity }
      end
    end
  end

  # PATCH/PUT /users/1 or /users/1.json
  def update
    respond_to do |format|
      if @user.update(user_params)
        format.html { redirect_to user_url(@user), notice: "User was successfully updated." }
        format.json { render :show, status: :ok, location: @user }
      else
        format.html { render :edit, status: :unprocessable_entity }
        format.json { render json: @user.errors, status: :unprocessable_entity }
      end
    end
  end

  # DELETE /users/1 or /users/1.json
  def destroy
    @user.destroy!

    respond_to do |format|
      format.html { redirect_to users_url, notice: "User was successfully destroyed." }
      format.json { head :no_content }
    end
  end

  private
    # Use callbacks to share common setup or constraints between actions.
    def set_user
      @user = User.find(params[:id])
    end

    # Only allow a list of trusted parameters through.
    def user_params
      params.require(:user).permit(:name, :email)
    end
end
```

scaffoldで生成すると、親切にもメソッドに対応するURLをコメントで記載してくれています。  
最初にアクセスした時は`/users`だったので、実行されたのは`index`メソッドになります。  

メソッドの中身には`@users = User.all`しか書かれていませんが、このメソッドで行っていることは他にもあります。  
実はアクションメソッドには、「メソッドの最後に特定の場所にあるViewを表示する」という特徴があります。  
そのViewとは、「Controller名と同じ名前のフォルダ内にある、メソッド名と同じ名前のView」のことです。  
例えばindexメソッドなら、usersフォルダ内にあるindex.html.erbというファイルが表示されることになります。  

そして、`@users = User.all`では何をやっているかというと、  
usersテーブルのレコードを全て取ってきてインスタンス変数に代入しています。  
`User`がModelに該当し、これに対してallメソッドを使用するとテーブルの全件取得が出来ます。  
それをなぜインスタンス変数に代入しているかというと、Controller内で定義されたインスタンス変数は、  
同じController内のアクションメソッドから表示されるViewでも参照が出来るようになるからです。

以上をまとめると、呼び出されたindexメソッドでは、  
「`/views/users/index.html.erb`をusersテーブルのレコードを全て取ってきた結果を渡した状態で表示する」  
という処理を行っていることになります。  
<br>

他にも、Controllerにはいくつか特徴があります。  

【Controllerの定義】

確認してみると、Controllerはクラスとして定義されていることが分かります。  
更に、`ApplicationController`というクラスを継承していることも分かります。  
この`ApplicationController`は、あらかじめController用に用意されているクラスで、  
Controller全体で共通の処理等を書いておく場所になっています。
この辺りは、前に述べたDRYの理念に則った構造と言えるでしょう。  

ちなみに、`ApplicationController`は`ActionController::Base`というクラスを継承していますが、  
実はこの`ActionController::Base`を継承していることが、Controllerとして認識される条件になっています。  
なので、Controllerクラスを定義する際には、直接`ActionController::Base`を継承するか、  
`ApplicationController`のような、継承しているクラスを更に継承する必要があります。  

【StrongParameter】

StrongParameterとは、画面上の操作で送られてきたデータを安全に受け取る仕組みです。
Railsに標準的に組み込まれていて、アプリの作成者が意図していない値を受け取らないように、  
アプリのプログラム内で受け取ることのできる値に制限を設けます。  
上記のController内であれば、以下の部分が該当します。  
```rb
def user_params
  params.require(:user).permit(:name, :email)
end
```
この`permit`メソッドで指定された値のみが扱えるようになります。  
scaffoldで生成した場合はこのように自動で追加してくれますが、  
自分で何か追加した時にはpermitメソッドを使用して指定しなくてはいけません。  
<br>

__【/views/users/index.html.erb】__  
今度は実際に表示されているViewファイルの中身を見てみましょう。  
```html
<p style="color: green"><%= notice %></p>

<h1>Users</h1>

<div id="users">
  <% @users.each do |user| %>
    <%= render user %>
    <p>
      <%= link_to "Show this user", user %>
    </p>
  <% end %>
</div>

<%= link_to "New user", new_user_path %>

```
HTMLの知識がある人なら、このファイルを見た時違和感があったと思います。  
上記のファイルには、通常のHTMLの構造でいうbodyの部分しか書かれていません。  
それ以外の部分はどこに書かれているかというと、`/views/layouts/application.html.erb`というファイルに書かれています。  
```html
<!DOCTYPE html>
<html>
  <head>
    <title>Railsbasic</title>
    <meta name="viewport" content="width=device-width,initial-scale=1">
    <%= csrf_meta_tags %>
    <%= csp_meta_tag %>

    <%= stylesheet_link_tag "application", "data-turbo-track": "reload" %>
    <%= javascript_importmap_tags %>
  </head>

  <body>
    <%= yield %>
  </body>
</html>
```
表示される時にはこの二つが組み合わさって表示されます。  

またこれらのViewファイルには、`<%= %>`や`<% %>`を使うことでHTMLの中にRubyのコードが書けるようになっています。  
これを利用して、Rubyの繰り返し文を使ってHTMLの要素を生成する事も出来ます。  
<br>

__【user.rb】__  
最後に、データの取得や登録時にはModelが使用されているのでそちらを確認してみましょう。  
```rb
class User < ApplicationRecord
end
```
現在はデータに対しての操作が何もないので、Modelの中身には何も書かれていません。  
実際にアプリを作成していくと、ここにメソッドを定義してデータの操作が出来るようにしていきます。  

Modelにはいくつか特徴的な機能があります。  
せっかくなので、今回のアプリもよりWEBアプリらしい形になるように機能を追加してみましょう。  

現在の状態では、登録されるデータに対して制限が何もありません。  
例えば、名前が空白のまま登録されたりすると困るので、空白が入力された時は登録出来ないようにしたいですね。  

制限をかける時には`バリデーション`という機能が使用されます。  
実際にバリデーションを使用して、上記の処理が出来るようにしてみましょう。  
```rb
class User < ApplicationRecord
  validates :name, presence: true
end
```
これで、画面上から登録する時に、名前が空白だと登録が出来ずに警告が出るようになっているはずです。  

また、もしメールアドレスをログインのID代わりにするなら、二つ同じものがあっては困ります。  
そういう時には以下のようにしてみましょう。  
```rb
class User < ApplicationRecord
  validates :name, presence: true
  validates :email, uniqueness: true
end
```
これで、一度登録したメールアドレスを使って登録しようとした時には警告が出るようになります。  

このように、Modelへコードを追加していくことで、色々なデータの操作が出来るようになります。  
<br>

ここまでで紹介したものが、Railsの基本的な作り方です。  
今回のものではまだ簡単な登録だけだったり、デザインも何もついていないので、  
次の章からはその辺りも考慮しながら、よりWEBアプリらしいものを作成してみましょう。
