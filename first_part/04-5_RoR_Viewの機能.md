## 4.5 Ruby on Rails:Viewの機能

ここでは、Ruby on Rails の「Viewの機能」、「SASS」「SCSS」「ERB」について説明します。

### 4.5.1 SASS/SCSSとは

[SASS](https://SASS-lang.com/)とは、「Syntactically Awesome StyleSheet」の略で、CSSを効率的に記述できるように開発された言語です。
簡単に言えば「CSSをプログラムの要素を入れて効率的に記述できるようにした言語」です。
SASSでは、変数を利用したり、演算を行ったり、関数を作成・利用したり、「`@extend`」を使用しスタイルを継承したり、「`@import`」でファイルをインポートしたり、よりプログラムらしく書くことで、効率よくコーティングでき、より保守性の高いプログラムを作成することができます。この仕組みはSCSSでも同様です。

### 4.5.2 SASSとSCSSの違い

SCSSとSASSの違いは、記述方法にあります。
SASSはインデント構文です。
インデント構文は、先頭からのスペースの数に意味があります。スタイルブロックを区切る`{}`の代わりにインデントを使用し、プロパティを区切る`;`の代わりに改行を用いて記述します。
SASSはスタイルブロックをインデントで表したり、`;`を改行で表しますが、SCSSでは、`{}`を使用しスタイルブロックを表します。
またファイルの拡張子もSASSの場合「`.sass`」、SCSSの場合「`.scss`」となります。
SCSSは、SASSとCSSとの互換性が不十分だったために開発された言語で、SCSSの書き方だけではなく、CSSの書き方でそのまま記述することも可能なため、一般的にSCSSの方が使用されています。

#### SASSの場合

```sass
#main
  color: red
  font-size: 12px
```

CSSにコンパイル後

```css
#main {
  color: red;
  font-size: 12px;
}
```

今回は一般的に使われているSCSSの書き方について見ていきます。

### 4.5.3 SCSSの書き方

#### SCSSでコメントを記述する

コメント行先頭に`/*`を付ける。
インデントを加えてコメント行とする。

```scss
// 単一行のコメント

/*
  複数行の
  コメントです
*/

#main {
  color: blue;
  font-size: 12px;
}
```

#### 変数を定義する

先頭に「`$`」を付ける

```scss
$main_color: yellow;

#main {
  color: $main_color;
  font-size: 12px;
}
```

CSSにコンパイル後

```css
#main {
  color: yellow;
  font-size: 12px;
}

```

#### 演算を行う

下記のようにプロパティの値に対して演算を行うことができます。

```scss
$main_height: 500px;
$main_width: 500px;


#main {
  height: $main_height + 100;
  width: $main_width / 2;
  background-color: red;
}
```

CSSにコンパイル後

```scss
#main {
  height: 600px;
  width: 250px;
  background-color: red;
}
```

#### 関数を使う

```scss
@function Double($value) {
  @return $value * 2;
}

@function Halve($value) {
  @return $value / 2;
}

.box {
  height: Double(100px);
  width: Halve(500px);
  background-color: green;
}
```

CSSにコンパイル後

```scss
.box {
  height: 200px;
  width: 250px;
  background-color: green;
}
```

#### @extendでスタイルを継承する

「`@extend`」を使用し他のスタイル宣言ブロックを継承することができます。

```scss
.main_box {
  background-color: yellow;
  height: 400px;
  width: 500px;
}


.sub_box1 {
  @extend .main_box;
  background-color: blue;
}

.sub_box2 {
  @extend .main_box;
  background-color: red;
}
```

CSSにコンパイル後

```css
.sub_box1 {
  background-color: blue;
  height: 400px;
  width: 500px;
}

.sub_box2 {
  background-color: red;
  height: 400px;
  width: 500px;
}
```

#### @importでファイルを読み込む

1つのSCSSファイルで全てのスタイルを書くと記述量が膨大になるため、CSSファイルを分割し別ファイルでまとめて読み込むといった方法が用いられます。
ファイルを読み込む場合「`@import`」を使用します。
拡張子を省略することも可能です。

```scss
@import "top.scss";
@import "index";
```

#### 「@mixin」を使用する

「`@mixin`」で処理をひとまとめにし、「`@include`」でまとめた処理を呼び出します。

```scss
@mixin border($color: yellow) { // 引数にデフォルト値を指定している
  border: solid $color;
  height: 300px;
  width: 300px;
}

#header {
  @include border;
}

#footer {
  @include border(gray);
  height: 200px;
  width: 200px;
}
```

CSSにコンパイル後

```css
#header {
  border-top: 2px solid white;
}

#footer {
  border-top: 2px solid gray;
}
```

### 4.5.4 ERBとは

ERBは、「**Embedded Ruby**」の略で、Railsでの基本的なView開発に使用されます。
拡張子に「`.erb`」をつけることでrubyのコードを埋め込むことができます。
Railsで使用される「`.html.erb`」ファイルはRubyのスクリプトが埋め込まれたHTMLファイルと考えてよいでしょう。
Rubyのスクリプトを埋め込むことができるので、htmlファイル内でif文を使用した条件分岐や繰り返し構文などの処理も記述できます。


### 4.5.5 ERBの書き方
実際にRubyのスクリプトを埋め込む記述方法を説明します。

 「 `<%  %>` 」 と 「 `<%=  %>` 」を使用しRubyのコードを記述します。

```erb
<% Rubyのコード %>
<%= Rubyのコード %>
```

「 `<%  %>` 」と「 `<%=  %>` 」の違いは下記の通りです。

「 `<%  %>` 」はブロック内のコードを実行するだけで、値は返しません。「 `<%=  %>` 」は、ブロック内の式を評価結果を返します。

```erb
<% price = 10000 %>  <!-- 代入処理を行うのみ -->
<% result = price * 1.1 %>  <!-- 代入処理を行うのみ -->
%>
<div><%= result %></div>  <!-- 変数resultの値を返す -->
```

このようにHTMLファイルにRubyのスクリプトを埋め込むことで、Controllerから送られてきた配列やハッシュなどのオブジェクトに対して繰り返し構文を使用し値を参照し、HTMLで表示するなどといった処理を記述することが可能です。

### 4.5.6 RailsプロジェクトでのSCSSとERBの使い方

RailsでSASS(SCSS)を使用するには、 `sass-rails` というgemをインストールする必要がありますが、Rails3.1以降から `Rails new` でプロジェクトを作成すれば自動的にインストールされる仕様となったため、 `sass-rails` をインストールするために必要な作業は特にありません。
またSASS(SCSS)は、最終的にブラウザが解釈できるCSSにコンパイルする必要がありますが、Railsでは、アセットパイプラインで、「 `.scss` 」から「 `.css` 」へのコンパイルは自動的に行われるので、これについても特に意識する必要はありません。
