## 5.5 Ruby on Rails：Gemとは

Railsは様々なGemが集まったものだというのは、もうご存知だと思います。Gemを使いこなすためにはREDMEはもちろん、Gemのソースコードを読む必要が出てきます。また、Gemのバージョン管理、依存関係を解消してくれるbundlerの存在も欠かせません。簡単にbundlerとGemの構成を確認してみましょう。

### 5.5.1 bundlerについて
bundlerは、gemのバージョン管理と依存関係を解決した環境を提供してくれるとても便利なツールです。bundler自体もgemとして提供されています。
```
$ gem install bundle
```
あとは、`Gemfile`さえ用意すれば`bundle install`コマンドでGemfile.lockを作成してくれます。bundlerに関して、以下の2点は押さえておきましょう。

#### bundle execコマンド
新しいプログラミング言語やソフトの環境構築をするとき、PATHが間違っているために動かなかったという経験があるかと思います。Gemに関してそのPATH問題を解決するのが`bundle exec`です。ターミナル等でrspecを実行するとき、下記のとおりコマンドの前に書いて実行すると、Gemfileに沿ったバージョンのrspecを実行してくれます。
```
$ bundle exec rspec
```
railsコマンドを実行するときもPATH問題が起きやすいので、気をつけてください。bundlerでインストールしたGemに対するものは、`bundle exec`で実行するか、シェルの$PATH設定を追加して確実に意図したGemが使えるようにしましょう。

#### Gemfileの書き方
`Gemfile`にはGemの名前はもちろん、いろいろな情報を設定することができます。以下の項目を参考に、関わっているプロジェクトのGemfileをチェックしてみましょう。

- バージョン(>=,<,~>)
- 参照先(source,git,path等)
- グループ設定(development,test,production)

また、Rubyのバージョンを明記しておくこともできます。記入しておくとログにワーニングが出力されて間違いも防げますので、最近は設定しておくプロジェクトが多いです。

そのほかのbundlerの機能については、マニュアルを参照ください。シンプルで読みやすい英語ですので、

下記のReference(Primary Commands,Utilities)は一通り目を通すことをおすすめします。
````
http://bundler.io/docs.html
````

### 5.5.2 Gemの構成とコードリーディング

#### Gemの構成
最近は、`bundler gem`コマンドでGemを作成するのがデファクトスタンダードになってきましたので、Gemの構成はだいたい以下のとおりです。
```
gem_name/
├.git/
├bin/
├lib/
├test/
├.gitignore
├Gemfile
├LICENSE.txt
├Rakefile
└gem_name.gemspec
```
Gem自体もRailsアプリと同様にgitで管理し、テストも実装されています。特徴としては、Gemのライセンスを示すファイル(上記の場合は`LICENSE.txt`)とそのGemについての情報がまとめられている`*.gemspec`があります。Gemをrubygems.orgへ公開したとき、この`*.gemspec`の内容が詳細ページの情報として使用されます。Gemのメインのコードはlibフォルダに保存されています。

#### コードリーディング
みなさんがRubyで何かつまづいたとき、どうやって解決しますか?ほとんどの場合は検索したり、RubyやRailsのマニュアルを読んだりすると思います。これからはもう一歩進んで、コードを読んで解決してみてください。

コードリーディングのメリットは以下のとおりです。

- RubyやRailsの知識が広がり、理解が深まる
- READMEやマニュアルなどに記載されていない機能がわかる
- 正確な情報を得られる(ただし、まれにREADMEなどが間違っていることもある)
- コード内のコメントのほうがREADMEより分かりやすい場合がある
- 英語で解説されたブログを読まなくても最新情報が得られる

プログラミングを長く続けていると、どうしても慣れた書き方に偏っていきます。知識を広げるにはコードリーディングが一番です。

ここで、ActiveSupportの機能を1つ紹介します。紹介するコードは以下のところにあります。初めはGithubサイト上で見るのが一番見やすいかもしれません。RubyのバージョンやActiveSupportのバージョンは、みなさんが使っているものを参照してください。
- github: https://github.com/rails/rails/blob/master/activesupport/lib/active_support/core_ext/numeric/conversions.rb
- rbenv: ~/.rbenv/versions/2.4.2/lib/ruby/gems/2.4.0/gems/activesupport-5.1.4/lib/active_support/core_ext/numeric/conversions.rb
- rvm: ~/.rvm/gems/ruby-2.4.2/gems/activesupport-5.1.4/lib/active_support/core_ext/numeric/conversions.rb

ファイルを開いてすぐにわかると思いますが、`.to_s`メソッドに引数が付いています。機能の拡張です。当然、純粋なRubyにはこのような機能はありません。

```
$ irb
irb(main):001:0> 12345678.to_s(:phone)
TypeError: no implicit conversion of Symbol into Integer
        from (irb):3:in `to_s'
        from (irb):3
        from /Users/hogehoge/.rbenv/versions/2.4.2/bin/irb:11:in `<main>'
```

```
$ rails console
irb(main):001:0> 12345678.to_s(:phone)
=>"1-234-5678"
```

コードを読み進めていくと、`def to_s(format = nil, options = nil)`の中で、`:phone`を指示している部分があります。

```
when :phone
  return ActiveSupport::NumberHelper.number_to_phone(self, options || {})
```

どうも、ヘルパーとして`.number_to_phone`というものがあるようですね。興味のある方はヘルパーのコードものぞいてみてください。これらを`.to_s`にまとめた理由まではわかりませんが、RailsではこういったRubyメソッドの上書きはよくあります。

さらにファイルの一番下には、Ruby2.4からBignumがFixnumと一緒になった影響で救済措置が書いてあります。仕方なく取ってつけたような感じがするのは私だけでしょうか。

```
# Ruby 2.4+ unifies Fixnum & Bignum into Integer. 
if 0.class == Integer
  Integer.prepend ActiveSupport::NumericWithFormat
else
  Fixnum.prepend ActiveSupport::NumericWithFormat
  Bignum.prepend ActiveSupport::NumericWithFormat
end
```
このように、シンプルなファイルを1つ見るだけでも新しい発見や疑問が出てくると思います。最後に、呼び出すメソッドがどこにあるかを示すメソッドを紹介しておきますので参考にしてください。

#### VirtualBoxの場合
```
$rails console
Running via Spring preloader in process 7740
Loading development environment (Rails 6.1.4.6)
irb(main):001:0> 12345678.method(:to_s).source_location
=> ["/home/user/.rbenv/version/2.7.4/lib/ruby/gems/2.7.0/gems/activesupport-6.1.4.6/lib/active_support/core_ext/numeric/conversions.rb", 109]
```

### 5.5.3 よく使われるGemの紹介
以下に業務システムで使いやすいGemをあげます。基本的にはGithubにREADMEとコードがありますので、気になるものはどんどん試していってください(順不同)

#### __デバッグ系__
- gem 'pry' # irb代替&拡張:ハイライトなどでコードを見やすくする。プラグインでさらに拡張可能
- gem 'awesome_print' # pp拡張:irbなどでコードを見やすくする
- gem 'hirb' # irb拡張:DBテーブルの内容をasciiテーブルで表示するなど
- gem 'letter_opener' # 擬似メール送信:メールサーバーの代わりにブラウザへメール内容を表示
- gem 'simplecov' # テストカバレッジ分析

#### __デザイン系__
- gem 'html2slim' # htmlからslimへの変換
- gem 'html2haml' # htmlからhamlへの変換
- gem 'kaminari' # ページネーション

#### __認証・権限系__
- gem 'devise' # 認証設定
- gem 'koala' # Facebook向けライブラリ
- gem 'cancancan' # 権限設定
- gem 'pundit' # ポリシー設定

#### __インフラ系__
- gem 'net-ssh' # ssh接続
- gem 'whenever' # cron作成
- gem 'dotenv' # 環境変数管理
- gem 'sidekiq' # jobキュー:ActiveJobのバックエンド

#### __ドキュメント系__
- gem 'prawn' # pdf出力(Ruby記述)
- gem 'pdfkit' # pdf出力(html記述)
- gem 'thinreports' # pdf出力(GUI)
- gem 'docx_templater' # docxテンプレート出力
- gem 'axlsx' # xlsx出力
- gem 'rubyzip' # zip出力

##### __その他機能追加__
- gem 'carrierwave' # ファイルアップロード
- gem 'simple_form' # htmlフォーム作成支援

