## 3.6 Ruby: 自動テスト（minitest）

多くのプログラミング言語では、自動テストを記述するためのフレームワーク（テスティングフレームワーク）が用意されています。

ここでは、Ruby本体に組み込まれているminitestというテスティングフレームワークの概要と使い方を解説します。

### 概要

minitestはRuby本体にgem（ライブラリ）として組み込まれています。また、Railsの標準のテスティングフレームワークでもあります。Railsアプリケーションのひな形に最初からインストールされていて、Railsそのものの自動テストにもminitestが使われています。

minitestでは、テストケースのまとまりをRubyのクラスとして表し、個々のテストケースはインスタンスメソッドとして記述します。
構造が比較的シンプルで、Rubyの基本的な知識があれば使うことができます。

このようにテストケースをクラスとメソッドとして表現して、`assert_○○`というメソッドで値をチェックするテスティングフレームワークはxUnitと総称されます。他のプログラミング言語のxUnit系のフレームワークには、SmalltalkのSUnit、JavaのJUnitなどがあります。

### テストの書き方

`String`クラスのテストをminitestで書くと、以下のようになります。

```rb
# minitestの読み込み
require 'minitest/autorun'

# Minitest::Testクラスを継承する
class TestString < Minitest::Test

  # "test_"で始まるメソッドがテストメソッドになる
  def test_length

    # assert_equalで2つの値が等しいことをテストする
    assert_equal(3, 'abc'.length)
  end

  # テストメソッドはいくつでも作ることができる
  def test_match

    # assert_instance_ofで最初の引数が2番目の引数のインスタンスであることをテストする
    assert_instance_of(MatchData, 'abc'.match(/./))
  end
end
```

上記のソースコードを`test_string.rb`などの名前をつけて保存して、`ruby test_string.rb`で実行してみましょう。

以下のように、最後に`2 runs, 2 assertions, 0 failures, 0 errors, 0 skips`と表示されればテストは成功です。

```
Run options: --seed 61764

# Running:

..

Finished in 0.001184s, 1689.1889 runs/s, 1689.1889 assertions/s.
2 runs, 2 assertions, 0 failures, 0 errors, 0 skips
```

テストコードを順番に見ていきましょう。

最初に`require 'minitest/autorun'`でminitestを読み込んでいます。これによって、テストの定義に必要な`Minitest::Test`などのクラスが読み込まれます。

次に、`Minitest::Test`クラスを継承してテスト用クラスを作成します。このクラスに`test_length`と`test_match`というメソッドを定義しています。minitestでは、このように`test_`で始まるメソッド１つ１つがそのままテストケースになります。

`test_length`の中では`assert_equal`というメソッドを使って、`length`メソッドの実際の返り値と、期待する値が一致するか確認しています。最初の引数が期待する値で、2番目の引数が実際の値です。

`test_match`の中では`assert_instance_of`というメソッドを使っています。これは、2番目の引数が最初の引数のインスタンスである（`2番目の引数.instance_of?(最初の引数)`）ことを確認するメソッドです。

このように実行結果が期待している値と一致しているか確認することを、minitestでは「アサーション」（assertion）と呼びます。

### テストの実行結果を読む

テストの実行結果を読んでみましょう。

まず、わざと失敗するテストを作ってみます。先ほどのテストコードのアサーションを以下のように書き換えて、テストを再実行してみてください。

```rb
  def test_length
    assert_equal(100, 'abc'.length)
  end

  def test_match
    assert_instance_of(Integer, 'abc'.match(/./))
  end
```

実行結果と期待する値が食い違っているため、エラーになります。

```
Run options: --seed 45537

# Running:

FF

Finished in 0.001130s, 1769.9115 runs/s, 1769.9115 assertions/s.

  1) Failure:
TestString#test_length [test_string.rb:7]:
Expected: 100
  Actual: 3

  2) Failure:
TestString#test_match [test_string.rb:11]:
Expected #<MatchData "a"> to be an instance of Integer, not MatchData.

2 runs, 2 assertions, 2 failures, 0 errors, 0 skips
```

後ろからさかのぼって読んでいきましょう。

出力の最後に`2 runs, 2 assertions, 2 failures, 0 errors, 0 skips`とあります。これは「2件のテストケースを実行して、合計2つのアサーションを実行した。2つのテストケースが失敗して、エラー（例外）が発生したテストケースとスキップされたテストケースは0件だった」という意味です。

その前の数行には、それぞれのテストケースでアサーションがどのように失敗したかが表示されています。

その前の`Finished in ...`の行ではテストの実行速度をレポートしています。アプリケーションの規模によってはテストケースが数百件以上になり、テストが完了するまでに数十秒から数分、あるいはそれ以上を要することもあります。

`# Running:`の後に出てくる`FF`はテストケースごとの状態を表します。１文字が１テストケースの状態を表していて、成功したら`.`、失敗したら`F`となります。アプリケーションが育っていくと`.`がずらりと並ぶことになります。

最初に表示されている`Run options: --seed 45537`はテストを再実行するためのコマンドラインオプションです。`--seed`は自動的に追加されるオプションで、テストケースの実行順を決めるために使われます。これについては次の節で説明します。

### テストケースの実行順

minitestは実行するたびにテストケースの順番をランダムに変更します。

たとえば次のように、あるテストケースが他のテストケースの実行結果を前提にしているテストは、実行するごとに成功したり失敗したりします。

```rb
require 'minitest/autorun'

$array = []

class TestArray < Minitest::Test
  def test_plus
    $array += [1, 2]
    assert_equal([1, 2], $array)
  end

  def test_minus
    # test_plusで$array = [1, 2]になることを期待している
    $array -= [1]
    assert_equal([2], $array)
  end
end
```

なぜこのような機能がわざわざ実装されているのでしょうか？　

これは「テストケースは単体で実行可能であるべき」という原則を守ることを助けるための機能です。

テストケースが他のテストケースに依存していると、あるテストケースを変更したら他のテストケースが意図せず壊れてしまうようになります。そのような状態に陥ってしまうと、テストをメンテナンスするのが難しくなります。それを防ぐため、あえてテストケースの実行順が固定されないようにしているのです。

テストの実行順のシード値はコマンドラインの`--seed`オプションで指定することができます。同じ内容のテストファイルについて同じシード値を使うと、テストケースを同じ順番で実行することができます。

minitestに限らず、多くのテスティングフレームワークは同様の機能を持っています。

### さまざまなアサーションメソッド

minitestにはさまざまなアサーションメソッドがあります。チェックしたい値の性質に合ったアサーションメソッドを使うことで、テストの意図が明確になり、失敗したときのエラーメッセージもわかりやすくなります。

#### 基本的なアサーションメソッド

メソッド名 | 成功条件
---|---
`assert(a)` | `a`が真
`assert_empty(a)` | `a`が`empty?`に真を返す
`assert_equal(a, b)` | `a == b`が真
`assert_includes(a, b)` | `a.include?(b)`が真
`assert_match(a, b)` | `a =~ b`が真
`assert_nil(a)` | `a.nil?`が真
`assert_predicate(a, op)` | `a.send(op)`が真
`assert_operator(a, op, b)` | `a.send(op, b)`が真
`assert_raises(*e, &block)` | `block`を実行すると`e`のいずれかの例外を発生させる

```rb
# array.empty?が真なら成功
assert_predicate(array, :empty?)

# a <= bが真なら成功
assert_operator(a, :<=, b)

# ブロックがStopIteration例外を発生させれば成功
assert_raises(StopIteration) { raise StopIteration }
```

#### 誤差を許容するアサーションメソッド

主に計算誤差が発生する浮動小数点数を比較するために使われます。

メソッド名 | 成功条件
---|---
`assert_in_delta(a, b, delta)` | `a - b`の差の絶対値が`delta`以下（※1）
`assert_in_epsilon(a, b, epsilon)` | `a`と`b`の相対的な差が`epsilon`以下（※1）

#### クラスやメソッドに関するアサーションメソッド

メソッド名 | 成功条件
---|---
`assert_instance_of(a, b)` | `a.instance_of?(b)`が真
`assert_kind_of(a, b)` | `a.kind_of?(b)`が真
`assert_respond_to(a, b)` | `a.respond_to?(b)`が真

#### その他のアサーションメソッド

メソッド名 | 成功条件
---|---
`assert_output(stdout, stderr, &block)` | `block`が標準出力に`stdout`, 標準エラー出力に`stderr`にマッチする文字列を出力する
`assert_path_exists(path)` | `File.exist?(path)`が真
`assert_same(a, b)` | `a.equal?(b)`が真
`assert_send(receiver, method, *args)` | `receiver.send(method, *args)`が真
`assert_silent(&block)` | `block`が標準出力と標準エラー出力に何も表示しない
`assert_throws(tag, &block)` | `block`が`tag`を`throw`する

いくつかのアサーションメソッドには、成功条件を反転させた`refute_○○`というアサーションメソッドがあります。たとえば、`refute_equal(a, b)`は`assert_equal(a, b)`とは逆に、`a != b`のときだけアサーションが成功します。

また、アサーションメソッドの最後の引数に文字列を渡すと、デフォルトのエラーメッセージの代わりにその文字列が使われます。

### テストケースに影響を及ぼす機能

最後に、テストケースに影響を及ぼす機能をいくつか紹介します。

`setup`メソッドと`teardown`メソッドを定義すると、それぞれテストケースの実行前、実行後に呼び出されます。テストケースで使うデータを用意する処理をまとめるのに便利です。

テストケース内で`skip`メソッドを呼ぶと、そのテストケースをスキップすることができます。一時的にテストの実行を止めたいときに使うと便利です。

```rb
class TestSample < Minitest::Test
  # テストケースの実行前に呼ばれる
  def setup
    @user = User.create(name: '佐藤太郎')
  end

  # テストケースの実行後に呼ばれる
  def teardown
    @user.destroy
  end

  # skipメソッドを呼ぶとテストをスキップできる
  def test_foo
    skip 'このテストはxxの実装後に書く'

    # skipメソッドより後の処理は実行されない
    assert(false)
  end
end
```

minitestには他にも多くの機能があり、プラグインで拡張することもできます。関心のある人は<https://github.com/seattlerb/minitest>で公式のドキュメントを参照してみましょう。

### まとめ

ここではminitestによる自動テストの作り方について学びました。ポイントをおさらいしておきましょう。

- minitestではテストケースをメソッドとして表す
- minitestのテストケースでは`assert_○○`というメソッドを使って実行結果が期待通りか確認する

### 発展課題

- fizzbuzzメソッドの自動テストを書いてみましょう。
- Rubyの標準クラスをひとつ選んで、いくつかのメソッドの自動テストを書いてみましょう。
- minitestのアサーションメソッドの実装を読んでみましょう。
- minitestの実行結果はカスタムレポーターを定義することで自由にカスタマイズできます。カスタムレポーターとその自動テストを書いてみましょう。

minitestのソースコードはGitHubで公開されています（<https://github.com/seattlerb/minitest>）。
