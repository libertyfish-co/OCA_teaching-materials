## 7.3　Ruby on Rails：Railsテスト基礎 1

### 7.3.1 テスティングツールについて

この章では、Railsアプリケーションでのテストの実装方法について学習します。Railsの標準テスティングツールはMinitest::Testです。`rails new`コマンドで作成したアプリケーションの雛形を作成したときには、自動的にtestフォルダが作られています。

Minitest::Testの他にRSpecというテスティングツールもあります。どちらが優れているということはありません。RSpecはDSL（ドメイン固有言語）を提供しているので、いつも馴染みのあるRubyの文法とは異なる文法を学習する必要があります。RSpecの例を挙げると次のようなソースコードになります。

```
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

`it`、`expect`などいつも見ないメソッドに驚かれたのではないでしょうか。筆者も当時はかなり驚いてRSpecでの実装は億劫でした。ですが、DSLを利用することで書き方についてのルールを決定することができ、チームでの作業効率が格段とあがります。みなさんも多くのエンジニアと共に今後、開発していくことを考えて今回はRSpecを利用してテストを実装していきます。

### 7.3.2 RSpecの環境構築

テスト基礎用のアプリケーションを作成しましょう。

```bash
cd
rails new rspec_mockups
```

#### 必要なgemのインストール

RSpecを使ったテストの実装環境を整えましょう。RSpecは強力なテスティングツールですが、テストデータの作成には長けていません。もちろん、腕に自信のある人は自作しても良いですが、OSSの文化では「車輪の再発明」は避ける習慣があります。今回は`rspec-rails`、`factory_bot_rails`、`gimei`の3つを利用して実装します。

- `rspec-rails`  
RSpecをRailsで利用するためのgem
- `factory_bot_rails`  
テストデータ作成の補助をするgem
- `gimei`  
ランダムに日本人の名前を生成するgem

Gemfileにこれらのgemを追加してみましょう。テスティングツールは本番環境では利用しないのでGemfileのグループは`development`と`test`とします。

```
  group :development, :test do
    # Use RSpec
    gem 'rspec-rails', '~> 4.0.1'
    # Use FactoryBot
    gem 'factory_bot_rails'
    # Use gimei for generating a Japanese fake name
    gem 'gimei'
  end
```

```bash
bundle install
```

#### RSpec実行環境の初期化

RSpecを実行するにあたって、様々な設定が必要になります。複雑になるので、順を追ってゆっくり説明するので安心してくださいね。

##### RSpec設定ファイルの生成

設定ファイルを作成するために次のコマンドを実行してください。いくつかのフォルダ、ファイルが生成されます。

```
  $ rails generate rspec:install
  Running via Spring preloader in process 54405
        create  .rspec
        create  spec
        create  spec/spec_helper.rb
        create  spec/rails_helper.rb
```

生成されたファイルに次の内容を追加してください。

- `.rspec`  

```
  --require spec_helper
  --format documentation
```

- `spec/rails_helper.rb`  

```
  RSpec.configure do |config|
      # Simplify syntax
      config.include FactoryBot::Syntax::Methods
  end
```

##### Rails設定ファイルの修正

Minitest::TestがRailsでは標準のテスティングツールなのでRSpecを利用するようにしてあげる必要があります。

- `config/application.rb`

```
  module RspecMockups
    class Application < Rails::Application
      # Don't generate system test files.
      config.generators.system_tests = nil
    end
  end
```

- `config/initializers/generators.rb`（新規作成）

```
  Rails.application.config.generators do |g|
    g.test_framework :rspec
    g.view_specs false
    g.routing_specs false
    g.helper_specs false
    g.fixture_replacement :factory_bot, dir: 'spec/factories'
  end
```

##### testフォルダの削除

前章で説明があったとおり、testフォルダはMinitest::Testに関連するファイルがありますが今回はRSpecを利用してテストを実装しますので、削除します。

```bash
rm -r test
```
