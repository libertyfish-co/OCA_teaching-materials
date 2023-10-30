## 7.4 Ruby on Rails：Railsテスト基礎 3

### 7.4.1 例題：テストの実装

#### Model テスト

Model でバリデータを定義してそれが正しく定義されているか確認するテストを実装してみましょう。利用者は名前と電話番号を持ち、それらは必ず入力しなければならないとします。

- app/models/user.rb
```rb
class User < ApplicationRecord
  validates :name, presence: true
  validates :phone_number, presence: true
end
```

また、テストデータはFactoryBotを利用するので次のように実装します。

- spec/factories/users.rb
```rb
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

```bash
rails console
```

結果
```
Running via Spring preloader in process 55944
Loading development environment (Rails 6.1.7.4)
irb(main)001:0>
```

テストデータを作成するコマンドを実行してみましょう。
```
FactoryBot.build :user
```

結果
```
=> #<User id: nil, name: "池上 羽海", phone_number: "XXX-YYYY-ZZZZ", created_at: nil, updated_at: nil>
irb(main)002:0>
```

もう一度テストデータを作成するコマンドを実行してみましょう。
```
FactoryBot.build :user
```

結果
```
=> #<User id: nil, name: "秋田 健夫", phone_number: "XXX-YYYY-ZZZZ", created_at: nil, updated_at: nil>
irb(main)003:0>
```

nameの値はGimeiによってランダムで作成されています。


肝心のテストですが、利用者についてのテストは次のようになります。実際に入力してテストの結果が変更されることを確認してみましょう。

- spec/models/user_spec.rb
```rb
require 'rails_helper'

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

テストを実行します。
```bash
bin/rake
```

結果
```
User
  is valid with name and phone number
  is invalid without name
  is invalid without phone number

(中略)

Finished in 0.30021 seconds (files took 1.36 seconds to load)
16 examples, 0 failures, 12 pending
```

テストが正しく記述できていれば、0 failuresになります。

ここで、テストコードを一部修正して、エラーを発生させてみましょう。

先程の2つ目のテストコードはnameの値がnilのuserを作成して、userの作成が失敗することを確認していた部分を、nameに値が設定されるように修正してみましょう。

- spec/models/user_spec.rb
```rb
require 'rails_helper'

RSpec.describe User, type: :model do

(中略)

  it "is invalid without name" do
    user = build(:user)
    user.valid?
    expect(user.errors[:name]).to include("can't be blank")
  end

(中略)

end
```

テストを実行します。
```bash
bin/rake
```

結果
```
User
  is valid with name and phone number
  is invalid without name (FAILED - 1)
  is invalid without phone number

(中略)

Failures:

  1) User is invalid without name
     Failure/Error: expect(user.errors[:name]).to include("can't be blank")
       expected [] to include "can't be blank"
     # ./spec/models/user_spec.rb:12:in `block (2 levels) in <top (required)>'

Finished in 0.32781 seconds (files took 1.39 seconds to load)
16 examples, 1 failure, 12 pending

Failed examples:

rspec ./spec/models/user_spec.rb:9 # User is invalid without name
```

nameの値がnilでないのでuserの作成が成功し、テストの期待値(失敗)と異なるのでテスト結果がエラーになっています。

エラーにならないように、もとのコードに戻しておきましょう。

メソッドをひとつずつ解説するときりがありませんが、RSpecの構造は次のようになります。

- `describe(context)`
引数にどのようなテストを行うか示すために、メソッド名やルーティングを記述してテストのグルーピングを行います。
```rb
  # Model の場合
  describe "#post" do
    # post メソッドのテスト
  end

  # Controller の場合
  describe "GET #edit" do
    # edit メソッドのテスト
  end
```
`context`は`describe`の内部で使用され、テストケースをさらに細分化するために使います。
```rb
  describe "#post" do
    context "params is invalid" do
      # params に不正な値がある場合
    end
  end
```
- `it`
期待する結果を記述します。テスト結果が期待値であるかの検証しません。
```rb
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
```rb
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

```rb
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

```rb
require 'rails_helper'

RSpec.describe "Users", type: :system do
  describe "GET /users" do
    it "renders new user link" do
      visit users_path
      assert_text "New user"
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

      click_on "New user"
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

      assert_text "Editing user"

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
      visit user_path(user)

      click_on 'Destroy this user'
      expect(page).to have_content("User was successfully destroyed.")
    end
  end
end
```

テストを実行します。
```bash
bin/rake
```

結果
```
(中略)

Users
  GET /users
    renders new user link
  GET /users/:id
    renders a details of a user
  GET /users/new
    renders a new user form
  GET /users/:id/edit
    shows a details of a user
  PATCH /users/:id/edit
    redirects to '/users/:id'
  DELETE /users/:id
    redirect_to '/users'

(中略)

Finished in 0.6424 seconds (files took 1.35 seconds to load)
22 examples, 0 failures, 12 pending
```

ここで、先ほどと同じようにテストコードを一部修正して、エラーを発生させてみましょう。

先程の3つ目のテストコードでnameの値を設定している行を削除するか、コメント行に修正してみましょう。

```rb

(中略)

  describe "GET /users/new" do
    it "renders a new user form" do
      visit users_path

      click_on "New user"
      # fill_in "Name", with: "User_Name"
      fill_in "Phone number", with: "XXX-YYYY-ZZZZ"

      click_on "Create User"
      assert_text "User was successfully created."
    end
  end

(中略)

end
```

テストを実行します。
```bash
bin/rake
```

結果
```
(中略)

Users
  GET /users
    renders new user link
  GET /users/:id
    renders a details of a user
  GET /users/new
    renders a new user form (FAILED - 1)
  GET /users/:id/edit
    shows a details of a user
  PATCH /users/:id/edit
    redirects to '/users/:id'
  DELETE /users/:id
    redirect_to '/users'

(中略)

Failures:

  1) Users GET /users/new renders a new user form
     Failure/Error: assert_text "User was successfully created."

     Capybara::ExpectationNotMet:
       expected to find text "User was successfully created." in "New user\n1 error prohibited this user from being saved:\nName can't be blank\nName\nPhone number\nBack to users"
     # ./spec/system/users_spec.rb:31:in `block (3 levels) in <main>'

Finished in 0.55425 seconds (files took 1.4 seconds to load)
22 examples, 1 failure, 12 pending

Failed examples:

rspec ./spec/system/users_spec.rb:23 # Users GET /users/new renders a new user form
```

nameの値が入力されていないのでuserの作成が失敗し、テストの期待値(成功)と異なるのでテスト結果がエラーになっています。

エラーにならないように、もとのコードに戻しておきましょう。
