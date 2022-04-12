## 3.7 Ruby: 自動テスト（RSpec）

RSpecは、前回取りあげたminitestと並んで人気のあるテスティングフレームワークです。ここではRSpecの概要と使い方を紹介し、minitestとの違いについて学びます。

### 概要

RSpecの「R」はRubyの頭文字です。「Spec」はSpecification、つまり「プログラムの仕様」を意味しています。RSpecは単なる自動テストではなく、プログラムの振る舞いをソースコードで表現することを重視したテスティングフレームワークです。

RSpecのテストコードはminitestと同じくRubyそのものですが、テストコードは自然言語（日本語、英語などの日常的に使う言語）に近づけられています。そのため、テストケースの定義や実行結果の検証の構文はminitestとは異なります。

たとえば、「`Array`クラスのインスタンスを引数なしで作った場合、そのインスタンスは`empty?`メソッドで`true`を返す」というテストコードをそれぞれのフレームワークで書くと、以下のようになります。

```rb
# minitest
class TestArray < Minitest::Test
  def test_empty
    assert(Array.new.empty?)
  end
end
```

```rb
# RSpec
RSpec.describe Array do
  describe '.new' do
    context '引数がないとき' do
      it '空であること' do
        expect(Array.new).to be_empty
      end
    end
  end
end
```

### gemのインストール

minitestと違って、RSpecはRuby本体には組み込まれていません。`gem install rspec`でgemをインストールしてください。

gemをインストールすることで、RSpecのテストを実行するための`rspec`コマンドもインストールされます。

### Arrayクラスのテストを書く

Arrayクラスを題材にしてRSpecのテストを書いてみましょう。

#### プロジェクトを作る

まず、`rspec_array`という名前のディレクトリを作ってください。

次に、RSpecの設定ファイルを用意します。`rspec_array`ディレクトリで`rspec --init`を実行してください。

- `.rspec`
- `spec/spec_helper.rb`

という2つのファイルが作られます。これらの設定ファイルの役割はあとで説明します。

#### 空のテスト

RSpecでは、テストコードは`spec`ディレクトリに`***_spec.rb`という名前で配置することになっています。

以下のプログラムを`spec/array_spec.rb`に保存してください。

```rb
# spec/array_spec.rb

RSpec.describe Array do
end
```

`rspec_array`ディレクトリで`rspec`コマンドを実行してみましょう。以下のように、最後に`0 examples, 0 failures`と表示されれば成功です。

```
$ rspec
No examples found.


Finished in 0.0003 seconds (files took 0.14107 seconds to load)
0 examples, 0 failures
```

#### テストケースを追加する

まずは、`Array.new`が空の配列を返すことをテストしてみましょう。テストコードは次のようになります。

```rb
# spec/array_spec.rb

# RSpec.describeでテストの対象を指定する
RSpec.describe Array do

  # itでテストケースを定義する
  it do

    # expect(...).to ...で実行結果をテストする
    expect(Array.new).to eq []
  end
end
```

最初の`RSpec.describe`でテストの対象を指定します。そのブロックの中にある`it`がテストケースです。`it`の中の`expect(Array.new).to eq []`が「`Array.new == []`である」というテストコードになります。

minitestではクラスとメソッドを使ってテストケースを表現していましたが、RSpecではこのように入れ子のブロックを使ってテストケースを表現します。

また、minitestでは`assert_○○`というアサーションで処理結果を検証していました。それに対して、RSpecでは`expect(...).to ...`というエクスペクテーションで処理結果を検証します。

新しいテストコードを入力して`rspec`コマンドを実行してみましょう。以下のように`1 example, 0 failures`と表示されていれば成功です。

```
$ rspec
.

Finished in 0.0055 seconds (files took 0.15194 seconds to load)
1 example, 0 failures
```

最初の`.`はminitestと同じようにテストケースの結果を表しています。次の`Finished in`の行はテストの実行時間を示しています。最後の`1 example, 0 failures`は、「1つのテストケースを実行して、失敗は0件だった」という意味になります。

### テストケースを失敗させる

次はテストケースを失敗させてみましょう。テストコードを以下のように書き換えてください。

```rb
# spec/array_spec.rb

RSpec.describe Array do
  it do
    expect(Array.new).to eq [nil]
  end
end
```

これで「`Array.new == [nil]`である」というテストになります。

`rspec`コマンドを実行すると以下のように出力されます。

```
$ rspec
F

Failures:

  1) Array is expected to eq [nil]
     Failure/Error: expect(Array.new).to eq [nil]

       expected: [nil]
            got: []

       (compared using ==)

       Diff:
       @@ -1,2 +1,2 @@
       -[nil]
       +[]

     # ./spec/array_spec.rb:3:in `block (2 levels) in <top (required)>'

Finished in 0.01932 seconds (files took 0.12781 seconds to load)
1 example, 1 failure

Failed examples:

rspec ./spec/array_spec.rb:2 # Array is expected to eq [nil]
```

出力を上から順番に見ていきましょう。

最初の`F`はテストケースが1つ失敗したことを表しています。

次に失敗したテストケースの詳細が表示されています。

- どのようなエクスペクテーションで失敗したか
- どんな値を期待していて、実際の値はどうだったのか
- 失敗したエクスペクテーションはどのファイルの何行目か

という情報が読み取れます。

その次の`Finished in`は成功時の出力と同じです。`1 example, 1 failures`は、「1つのテストケースを実行して、失敗は1件だった」という意味になります。

最後に、失敗したテストケースが列挙されています。

### テストケースを整理する

RSpecの出力では、テストケースを`example`と呼んでいることに気づいたでしょうか？　RSpecのテストケースは単なるテストではなく、サンプルコード（実行例）のように書くことが勧められています。

RSpecはテストケースを実行例として整理するために、このようにテストケースに名前をつけて階層化する機能を備えています。

まず、テストケースに名前をつけてみましょう。次のように`it`の最初の引数に文字列を渡すと名前をつけることができます。

```rb
RSpec.describe Array do
  it '引数なしでArray.newを実行すると空の配列を返す' do
    expect(Array.new).to eq []
  end

  it '整数を引数にしてArray.newを実行するとその長さの配列を返す' do
    expect(Array.new(3).size).to eq 3
  end
end
```

さらに、次のように`describe`メソッドと`context`メソッドを入れ子にすることができます。

```rb
RSpec.describe Array do

  # describeメソッドで「何についてのテストなのか」を示す
  describe '#new' do

    # contextメソッドで「どんな場合のテストなのか」を示す
    context '引数がない場合' do
      it '空の配列を返す' do
        expect([]).to eq []
      end
    end

    context '整数を引数に渡した場合' do
      it '整数と同じ長さの配列を返す' do
        expect(Array.new(3).size).to eq 3
      end
    end
  end
end
```

`--format documentation`オプションをつけて`rspec`コマンドを実行すると、次のようにテストの実行結果がドキュメント形式で表示されます。

```
$ rspec --format documentation

Array
  #new
    引数がない場合
      空の配列を返す
    整数を引数に渡した場合
      整数と同じ長さの配列を返す

Finished in 0.00261 seconds (files took 0.11846 seconds to load)
2 examples, 0 failures
```

ここまでに`describe`, `context`, `it`, `expect`という４つのメソッドを紹介しました。それぞれの役割は以下の通りです。

メソッド名|含むことができるメソッド|役割
---|---|---
`describe` | `describe`, `context`, `it` | 「何についてのテストなのか」を示す
`context` | `describe`, `context`, `it` | 「どんな場合のテストなのか」を示す
`it` | `expect` | １つのテストケースを表す
`expect` | なし | 期待する実行結果を表す

### さまざまなマッチャー

minitestには、検証したい値の種類に応じていろいろな種類のアサーションメソッドがありました。RSpecでは同じものを「matchするかどうか確かめるもの」という意味でマッチャーと呼んでいます。

マッチャーは`expect(actual).to`への引数を返すメソッドです。サンプルコードでは`eq`マッチャーだけを使っていましたが、RSpecのマッチャーはminitestよりも数が多く、機能も複雑です。

ここでは代表的なものだけを紹介します。

#### `eq`

```rb
expect(actual).to eq expected
```

`actual == expected`であることを確かめます。もっとも基本的なマッチャーです。

#### `include`

```rb
expect(actual).to include expected
```

`actual.include?(expected)`が真であることを確かめます。

#### `be_truthy`, `be_falsey`

```rb
expect(actual).to be_truthy
expect(actual).to be_falsey
```

`be_truthy`は`actual`が真値であることを確かめます。`be_falsey`は偽値であることを確かめます。

#### 述語マッチャー

```rb
expect(actual).to be_positive
expect(actual).to be_empty
```

`actual`に`○○?`というメソッドが定義されているとき、`be_○○`という形式のマッチャーが使えます。

`actual.○○?`が真であることを確かめます。

#### `raise_error`

```rb
expect { ... }.to raise_error(exception_class)
```

`expect`に渡したブロックが`exception_class`の例外を起こすことを確かめます。`expect`の引数がブロックになっていることに注意してください。

#### マッチャーの否定

```rb
expect(actual).not_to be_empty
```

`to`のかわりに`not_to`を使うことで、マッチャーの意味を逆にすることができます。

この例では`actual.empty?`が偽であることを確かめています。

### テストケースに影響を及ぼす機能

`before`メソッドにブロックを渡すと、テストケースの実行前にブロックが実行されます。

```rb
RSpec.describe Array do
  context '配列が空' do
    before { @array = [] }
    it { expect(@array).to be_empty }
    it { expect(@array.size).to be_zero }
  end

  context '配列が空ではない' do
    before { @array = [1, 2, 3] }
    it { expect(@array).not_to be_empty }
    it { expect(@array.size).to be_positive }
  end
end
```

`after`メソッドにブロックを渡すと、テストケースの実行後にブロックが実行されます。

以下のように、`it`を`skip`に置き換えたり、`it`のブロックの中で`skip`メソッドを呼ぶことでテストケースをスキップすることができます。

```rb
RSpec.describe Integer do
  skip { expect(3.fizzbuzz).to eq 'Buzz' }

  it do
    skip 'このテストはfizzbuzzの実装後に書く'
    expect(5.fizzbuzz).to eq 'Fizz'
  end
end
```

### minitestとRSpecの違い

RSpecのような記法のテスティングフレームワークは、プログラムの振る舞い（Behavior）を重視するという意味で、BDD (Behavior Driven Development) スタイルと呼ばれています。

minitestはBDDスタイルもサポートしているので、以下のようなテストコードを書くこともできます。

```rb
require 'minitest/autorun'

describe Array do
  describe '配列が空' do
    before { @array = [] }
    it { _(@array).must_be_empty }
    it { _(@array.size).must_be :zero? }
  end

  describe '配列が空ではない' do
    before { @array = [1, 2, 3] }
    it { _(@array).wont_be_empty }
    it { _(@array.size).must_be :positive? }
  end
end
```

RSpecとminitestの大きな違いは内部の構造にあります。RSpecでは、テストケースもマッチャーもすべてがオブジェクトです。

そのため、テストケースにラベルをつけて特別な処理をしたり、複数のマッチャーを合成するなど、minitestにはない拡張性があります。その反面、minitestに比べると、ライブラリとしての構造が複雑で、何か意図しないことが起きたときに内部の処理を追うのに手間がかかります。

次の章からはRSpecを使ってテストコードを書いていきますが、みなさんが自分でプロジェクトを始めるときには、チーム全体の考え方などに合わせて適切なテスティングフレームワークを選択してください。

### まとめ

ここではRSpecによる自動テストの作り方について学びました。ポイントをおさらいしておきましょう。

- RSpecはプログラムの振る舞いを記述することを重視したテスティングフレームワークである
- RSpecでは`it`メソッドを使ってテストケースを定義する
- RSpecのテストケースでは`expect`メソッドで実行結果が期待通りか確認する
- RSpecでは`describe`, `context`を使ってテストケースを階層化して整理する

### 発展課題

- minitestで書いた自動テストをRSpecを使うように書き換えてみましょう。
- 入れ子になった`describe`や`context`のそれぞれで`before`メソッドを呼び出してみましょう。`before`メソッドのブロックはどのような順番で実行されるでしょうか？
- RSpecのドキュメントを読んで、このテキストで説明されていないマッチャーを使ってみましょう。（<https://relishapp.com/rspec/rspec-expectations/docs/built-in-matchers>）
- RSpecのマッチャーを自分で定義してみましょう。

RSpecのソースコードはGitHubで公開されています。minitestと違い、３つのgemに分割されています。

- <https://github.com/rspec/rspec-core>: テストケースの実行機能などを提供します
- <https://github.com/rspec/rspec-expectations>: エクスペクテーションを提供します
- <https://github.com/rspec/rspec-mocks>: テスト用のダミーオブジェクトを作るための機能を提供します
