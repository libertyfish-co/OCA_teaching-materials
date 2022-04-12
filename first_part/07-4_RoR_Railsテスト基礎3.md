## 7.4 Ruby on Rails：Railsテスト基礎 2

### 7.4.1 例題：テストの実装

#### Model テスト

Model でバリデータを定義してそれが正しく定義されているか確認するテストを実装してみましょう。利用者は名前と電話番号を持ち、それらは必ず入力しなければならないとします。

- app/models/user.rb
```
  class User < ApplicationRecord
    validates :name, presence: true
    validates :phone_number, presence: true
  end
```

また、テストデータはFactoryBotを利用するので次のように実装します。

- spec/factories/users.rb
```
  FactoryBot.define do
    factory :user do
      name { Gimei.new.kanji }
      phone_number { "XXX-YYYY-ZZZZ" }
    end
  end
```

FactoryBotを使って実装したデータは`build(:user)`や`create(:user)`メソッドを使ってテストデータを作成することが出来ます。`rails console`コマンドでプロンプトを立ち上げて実行してみましょう。テストデータが作成されることが確認出来ましたか？

テーブルを作成します。

```bash
rails db:migrate
```

コンソールを起動します。

```
  $ rails console
  Running via Spring preloader in process 55944
  Loading development environment (Rails 5.1.3)
  2.4.1 :001 > FactoryBot.build :user
   => #<User id: nil, name: "池上 羽海", phone_number: "XXX-YYYY-ZZZZ",
   created_at: nil, updated_at: nil>
  2.4.1 :002 >
```

肝心のテストですが、利用者についてのテストは次のようになります。実際に入力してテストの結果が変更されることを確認してみましょう。

- spec/models/user_spec.rb
```
  RSpec.describe User, type: :model do
    it "is valid with name and phone number" do
      user = build(:user)
      expect(user).to be_valid
    end

    it "is invalid without name" do
      user = build(:user, name: nil)
      user.valid?
      expect(user.errors[:name]).to include("can't be blank")
    end

    it "is invalid without phone number" do
      user = build(:user, phone_number: nil)
      user.valid?
      expect(user.errors[:phone_number]).to include("can't be blank")
    end
  end
```

メソッドをひとつずつ解説するときりがありませんが、RSpecの構造は次のようになります。

- `describe(context)`  
引数にどのようなテストを行うか示すために、メソッド名やルーティングを記述してテストのグルーピングを行います。
```
  # Model の場合
  describe "#post" do
    # post メソッドのテスト
  end

  # Controller の場合
  describe "GET #edit" do
    # edit メソッドのテスト
  end
```
`context`は`describe`のエイリアスですが、こちらは条件を記述します。
```
  describe "#post" do
    context "params is invalid" do
      # params に不正な値がある場合
    end
  end
```
- `it`  
期待する結果を記述します。テスト結果が期待値であるかの検証しません。
```
  describe "POST #create" do
    context "with valid params" do
      it "creates a new User" do
        #　create メソッドが実行されると、User モデルにレコードが保存されることを期待する
      end
    end
  end
```
- `expect`  
テスト結果が期待値であるかの検証をします。構文は`expect(期待値).to マッチャ`になります。マッチャはテスト対象によってことなります。次の例では利用者が新規作成されていますので、マッチャは次の通りです。
  - テーブルの総数は1つ増える  
  `change(User, :count).by(1)`
  - 新規作成に成功したら詳細ページに遷移する  
  `redirect_to(User.last)`
```
  describe "POST #create" do
      context "with valid params" do
        it "creates a new User" do
          expect {
            post :create, params: {user: valid_attributes},
            session: valid_session
          }.to change(User, :count).by(1)
        end

        it "redirects to the created user" do
          post :create, params: {user: valid_attributes},
          session: valid_session
          expect(response).to redirect_to(User.last)
        end
      end
  end
```

始めはわかりにくいかもしれませんが、形式に囚われずに「何をテストしたいか」を主軸にテストを作成することをおすすめします。

#### System テスト

 エンドツーエンド (`E2E`) のテストを行います。

Javascriptを利用した画面等のテストを作成します。

 `spec/system` ディレクトリを作成します。

 ```bash
 mkdir spec/system
 ```
 
 テストコードを記述する `users_spec.rb` を作成します。

 ```bash
 touch spec/system/users_spec.rb
 ```

SystemSpecは毎回Chromeを利用しますが、今回のE2Eテストは、JavaScriptがなくても実行可能なテストなので、Chromeを利用しない設定にします。 `spec/rails_helper.rb` に下記の設定を記述しましょう。
これで `js: true` のタグが付いているテストケースだけChromeを利用するようになりました。

- `spec/rails_helper.rb`

```ruby
RSpec.configure do |config|
  #(省略)

  config.before(:each) do |example|
    if example.metadata[:type] == :system
      if example.metadata[:js]
        driven_by :selenium_chrome_headless, screen_size: [1400, 1400]
      else
        driven_by :rack_test
      end
    end
  end
end
```

- spec/system/users_spec.rb

```ruby
require 'rails_helper'

RSpec.describe "Users", type: :system do
  describe "GET /users" do
    it "renders new user link" do
      visit users_path
      assert_text "New User"
    end
  end

  describe "GET /users/:id" do
    let(:user) { create(:user) }

    it "renders a details of a user" do
      visit user_path(user)

      assert_text "#{user.name}"
      assert_text "#{user.phone_number}"
    end
  end

  describe "GET /users/new" do
    it "renders a new user form" do
      visit users_path

      click_on "New User"
      fill_in "Name", with: "User_Name"
      fill_in "Phone number", with: "XXX-YYYY-ZZZZ"

      click_on "Create User"
      assert_text "User was successfully created."
    end
  end

  describe "GET /users/:id/edit" do
    let(:user) { create(:user) }

    it "shows a details of a user" do
      visit edit_user_path(user)

      assert_text "Editing User"

      click_on "Update User"
      assert_text "User was successfully updated."
    end
  end

  describe "PATCH /users/:id/edit" do
    let(:user) { create(:user) }
    let(:new_name) { Gimei.new.kanji }
    let(:new_phone_number) { "AAA-BBBB-CCCC" }

    it "redirects to '/users/:id'" do
      visit edit_user_path(user)

      fill_in "Name", with: "#{new_name}"
      fill_in "Phone number", with: "#{new_phone_number}"

      click_on "Update User"
      expect(page).to have_content("User was successfully updated.")
    end
  end

  describe "DELETE /users/:id" do
    let!(:user) { create(:user) }

    it "redirect_to '/users'" do
      visit users_path

      click_on 'Destroy'
      expect(page).to have_content("User was successfully destroyed.")
    end
  end
end
```