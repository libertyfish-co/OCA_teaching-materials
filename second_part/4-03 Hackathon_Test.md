# 4.3 ハッカソン( テスト(Rspec,Capybara) )

## 4.3.1 RSpec,Capybaraの利用

RSpecとCapyparaを利用して、開発中のアプリケーションのUnitテストとUIテスト(エンドツーエンドテスト)を行いましょう。

## 4.3.2 ポイント

### RspecによるUnitテストが行えること

RspecでUnitテストを書きましょう。

### CapybaraによるUIテストが行えること

CapybaraでUIテストを書きましょう。

### Rspecについての理解を深め、正しく利用できること

RSpec特有のDSL（context / describe / it / expectの役割、let / subjectの使い方、to / not_to / beなどのマッチャ)などの理解を深めて、
DRYで簡潔なテストコードが書けるようになりましょう。

### Capybaraについての理解を深め、正しく利用できること

Capybara特有の基本的なページ移動やクリックなどのDSL(visit / click_on / fill_in / select)について理解を深めましょう。

## 4.3.3 アドバイス

### Unitテストから行う

UIテストも大切ですが、最小単位であり基礎となるUnitテストから行い、徐々にテストの範囲を広げましょう。
Unitテスト、統合テスト、UIテストの順が理想ではないでしょうか。

### FactoryBot(gem)を使用する

FactoryBotとは、テストデータの作成をサポートしてくれるgemです。
FactoryBotを使用すると、簡単にテストデータを作成できるため、より簡潔にUnitテストを書くことができます。
FactoryBotについても理解を深めて、導入してみましょう。

### UI操作以外のテスト方法も学習する

Capybaraは4.3.2ポイントで記載したUIの操作以外にも、要素を操作したり、検証することができます。
他の機能についても理解を深めて、テストを書いてみましょう。
