## 9.6 Ruby on Rails：ECサイトの開発 画像アップロード2

Active Storageを利用して表示する画像のリサイズをするためにImageMagickを利用します。

ImageMagickは、画像のリサイズや画像フォーマットの変換、画像の加工、GIFアニメーションの作成などもできる画像処理のコマンドです。

### 9.6.1 画像のリサイズ

#### ImageMagickをインストール

ここでは、ImageMagickをインストールする手順を説明します。

様々な環境でのImageMagickのインストール手順は、`https://imagemagick.org/script/download.php`を参考にしてください。

```
$ sudo apt install imagemagick
```

#### gemの追加

`Gemfile`に`gem 'image_processing'を追加しましょう。コメントになっている場合は、コメントを外しましょう。

`ImageProcessing`は画像サイズを調整する機能を持つ`gem`です。

`Gemfile`の変更ができたら、`bundle install`を実行しましょう。

```bash
$ bundle install
```

#### モデルにリサイズするメソッドを定義

 1. 大きめの画像がアップロードされた場合でも、一定の大きさで表示されるようにリサイズします。

    Active Storageでは、アップロードされた画像はそのまま保存し、表示する時にリサイズをします。
    画像の大きさを縦横300x300ピクセルに収まる画像として表示するようにモデルにメソッドを追加します。

    `app/models/user.rb`

    ```ruby
    class User < ApplicationRecord
      has_one_attached :photo

      def thumbnail
        photo.variant(resize: '300x300')
      end
    end
    ```

#### リサイズされた画像の確認 

 1. リサイズした画像を表示できるようにViewファイルを修正します。

    追加したリサイズのメソッドを利用して画像を表示するようビューを修正します。
    ここでは、元の画像と比較できるように、`@user.thumbnail`を利用して表示できるように処理を追加しましょう。

    `app/views/users/show.html.erb`

    ```ruby
    (省略)
    <p>
      <strong>Photo:</strong>
      <% if @user.photo.attached? %>
        <%= image_tag @user.photo %>
        <%= image_tag @user.thumbnail %>
      <% end %>
    </p>
    (省略)
    ```

 2. ブラウザで`show`をクリックして画像を確認します。

    アップロードした画像が指定のサイズ(300x300)で表示されればOKです。

### 9.6.2 アップロード機能の自動テスト

 1. アップロードできるファイルの種類と大きさを制限して、その制限が機能しているかテストを作成しましょう。

    まずは、アップロードされる画像に対してのバリデーション条件を記述しましょう。

    `app/models/user.rb`

    ```ruby
    class User < ApplicationRecord
      has_one_attached :photo

      # (1)photoに対するバリデーションを適用
      validate :validate_photo

      # (2)photoに対するファイルサイズの制限と、ファイルの種類の制限
      def validate_photo
        return unless photo.attached?
        if photo.blob.byte_size > 2.megabytes
          errors.add(:photo, 'File too large.')
        elsif !allowed_image_type?
          errors.add(:photo, 'File type not allowed.')
        end
      end

      def thumbnail
        photo.variant(resize: '300x300')
      end

      private

        # (3)ここではJPEG形式の画像のみを受け入れ可能とします
        def allowed_image_type?
          %w[image/jpg image/jpeg].include?(photo.blob.content_type)
        end
    end
    ```

 2. バリデーションの確認
 
    実際に、2MB以上の画像をアップロードしたり、PNG形式などのJPEG形式とは異なる画像をアップロードしてバリデーションが機能していることを確認しましょう。

 3. テストの実装

    `Gemfile`にRSpecに必要な`gem`を追加します。

    ```ruby
    group :development, :test do
      # Use RSpec
      gem 'rspec-rails', '~> 4.0.1'
      # Use FactoryBot
      gem 'factory_bot_rails'
    end
    ```

    `bundle install`も忘れずに実行します。

    ```bash
    $ bundle install
    ```

    `RSpec`のインストールも忘れずに実行します。
    ```bash
    $ rails generate rspec:install
    ```

    `config/application.rb`に`RSpec`を利用する設定を記述します。
    ```ruby
      module Storage
        class Application < Rails::Application
          # Don't generate system test files.
          config.generators.system_tests = nil
        end
      end
    ```

    `config/initializers/generators.rb`を新規作成して、以下の内容を記述します。
    ```ruby
      Rails.application.config.generators do |g|
        g.test_framework :rspec
        g.view_specs false
        g.routing_specs false
        g.helper_specs false
        g.fixture_replacement :factory_bot, dir: 'spec/factories'
      end
    ```

    Userモデルに対するspecファイルを作成します。

    ```
    $ rails generate rspec:model user
    ```

    `spec/models/user_spec.rb`にテストを記述します。

    ```ruby
    require 'rails_helper'

    RSpec.describe User, type: :model do
      it "is valid content_type with jpg, jpeg" do
        formats = %w(jpg jpeg)
        formats.each do |format|
          user = User.new( name: "test" )
          user.photo.attach(io: File.open("spec/fixtures/photo.#{format}"),
                            filename: "photo.#{format}", content_type: 'image/#{format}')
          expect(user).to be_valid
        end
      end
      
      it "is invalid content_type with png" do
        formats = %w(png)
        formats.each do |format|
          user = User.new( name: "test" )
          user.photo.attach(io: File.open("spec/fixtures/photo.#{format}"),
                            filename: "photo.#{format}", content_type: 'image/#{format}')
          expect(user).not_to be_valid
        end
      end

      it "is invalid to large file" do
        user = User.new( name: "test" )
        user.photo.attach(io: File.open("spec/fixtures/large-photo.jpg"),
                          filename: "large-photo.jpg", content_type: 'image/jpg')
        expect(user).not_to be_valid
      end
    end
    ```

    テストに必要なファイルを準備します。

    ```
    spec/fixtures/photo.jpg (ファイルサイズが2MB以下のJPEG形式画像)
    spec/fixtures/photo.jpeg (ファイルサイズが2MB以下のJPEG形式画像,photo.jpgのコピーでもOK)
    spec/fixtures/photo.png (許可されていないPNG形式画像)
    spec/fixtures/large-photo.jpg (ファイルサイズが2MBより大きなJPEG形式画像)
    ```

    テストを実行します。

    ```
    $ bundle exec rspec spec/models/user_spec.rb
    2019-11-20 13:45:56 WARN Selenium [DEPRECATION] Selenium::WebDriver::Chrome#driver_path= is deprecated.
    Use Selenium::WebDriver::Chrome::Service#driver_path= instead.
    ...

    Finished in 0.55947 seconds (files took 5.28 seconds to load)
    3 examples, 0 failures
    ```
    とエラーがゼロ件なら、テストは成功です。

    今度は、ファイルサイズの制限をlarge-photo.jpgの画像のファイルサイズより大きく(例:10MB)、また、受け入れ可能な画像形式に`image/png`を追加してみましょう。

    `app/models/user.rb`

    ```ruby
    class User < ApplicationRecord
      (省略)
      def validate_photo
        return unless photo.attached?
        if photo.blob.byte_size > 10.megabytes
          errors.add(:photo, 'File too large.')
        elsif !allowed_image_type?
          errors.add(:photo, 'File type not allowed.')
        end
      end

      private

        def allowed_image_type?
          %w[image/jpg image/jpeg image/png].include?(photo.blob.content_type)
        end
    end
    ```

    テストを実行します。

    ```
    $ bundle exec rspec spec/models/user_spec.rb
    2019-11-20 13:49:43 WARN Selenium [DEPRECATION] Selenium::WebDriver::Chrome#driver_path= is deprecated.
    Use Selenium::WebDriver::Chrome::Service#driver_path= instead.
    .FF

    Failures:

      1) User is invalid with png
        Failure/Error: expect(user).not_to be_valid
          expected #<User id: nil, name: "test", created_at: nil, updated_at: nil> not to be valid
        # ./spec/models/user_spec.rb:18:in `block (3 levels) in <top (required)>'
        # ./spec/models/user_spec.rb:15:in `each'
        # ./spec/models/user_spec.rb:15:in `block (2 levels) in <top (required)>'

      2) User is invalid to large file
        Failure/Error: expect(user).not_to be_valid
          expected #<User id: nil, name: "test", created_at: nil, updated_at: nil> not to be valid
        # ./spec/models/user_spec.rb:25:in `block (2 levels) in <top (required)>'

    Finished in 0.71279 seconds (files took 6.11 seconds to load)
    3 examples, 2 failures

    Failed examples:

    rspec ./spec/models/user_spec.rb:13 # User is invalid with png
    rspec ./spec/models/user_spec.rb:22 # User is invalid to large file
    ```
    と、2件のエラーが検出されると期待通りの動作です。ですが、コードのバランスがとれていませんので、修正した箇所は、元に戻してもかまいませんし、`app/models/user.rb`の変更をそのままにするのであれば、テストが正常に終了するように`spec/models/user_spec.rb`を修正しておきましょう。
