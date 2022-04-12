## 11.5 Ruby on Rails：ECサイトの開発 enum/状態遷移2

### 11.5.1 例題(つづき)

貸出中（`rented`）、書庫（`stack`）には、`0`と`1`がそれぞれ割り当てられるのでRSpecのテストで確認してみましょう。

- `spec/models/book_spec.rb`
```
  require 'rails_helper'

  RSpec.describe Book, type: :model do
    describe 'testing for book status' do
      context 'assign primitive value' do
        it "expects 0 to be mapped to 'rented'" do
          book = Book.create(status: 0)
          expect(book.status).to eq('rented')
        end
        it "expects 1 to be mapped to 'stack'" do
          book = Book.create(status: 1)
          expect(book.status).to eq('stack')
        end
      end

      context 'assign enum value' do
        it "expects 'rented' to be mapped to 0" do
          book = Book.create(status: 'rented')
          expect(Book.statuses[book.status]).to eq(0)
        end
        it "expects 'stack' to be mapped to 1" do
          book = Book.create(status: 'stack')
          expect(Book.statuses[book.status]).to eq(1)
        end
      end
    end
  end
```

また、検索クエリも状態名で実行できるようにメソッドが定義されます。RSpecを利用して動作を確認してみましょう。

- `spec/factories/books.rb`
```
  FactoryBot.define do
    factory :rented_book, class: Book do
      sequence(:title) do |n|
        "坊っちゃん 第#{n}巻"
      end
      status { 0 }
    end
    factory :stack_book, class: Book do
      sequence(:title) do |n|
        "吾輩は猫である 第#{n}巻"
      end
      status { 1 }
    end
  end
```

- `spec/models/book_spec.rb`
```
  require 'rails_helper'

  RSpec.describe Book, type: :model do
    describe 'testing for book status' do
      describe 'enum attributes are available for where clause' do
        before do
          FactoryBot.create_list(:rented_book, 7)
          FactoryBot.create_list(:stack_book, 11)
        end

        it "expects the number of 'rented' books is 7" do
          expect(Book.rented.count).to eq(7)
        end

        it "expects the number of 'stack' books is 11" do
          expect(Book.stack.count).to eq(11)
        end
      end
    end
  end
```

Railsアプリケーション開発を進めるにつれ、要件が変わっていくことはよくあります。本の貸出状態に予約済（`reserved`）を増やしてみましょう。`book.rb`は次のようになります。

```
  class Book < ApplicationRecord
    enum status: [:reserved, :rented, :stack]
  end
```

#### ② テストの実行

ここで既存の機能に影響がないか、RSpecテストを実行して確認してみましょう。

```
$ rspec

Book
  testing for book status
    assign primitive value
      expects 0 to be mapped to 'rented' (FAILED - 1)
      expects 1 to be mapped to 'stack' (FAILED - 2)
    assign enum value
      expects 'rented' to be mapped to 0 (FAILED - 3)
      expects 'stack' to be mapped to 1 (FAILED - 4)
    enum attributes are available for where clause
      expects the number of 'rented' books is 7 (FAILED - 5)
      expects the number of 'stack' book is 11 (FAILED - 6)

Failures:

  1) Book testing for book status assign primitive
  value expects 0 to be mapped to 'rented'
     Failure/Error: expect(book.status).to eq('rented')

       expected: "rented"
            got: "reserved"

       (compared using ==)
     # ./spec/models/book_spec.rb:8:in `block (4 levels) in <top (required)>'

  2) Book testing for book status assign primitive
  value expects 1 to be mapped to 'stack'
     Failure/Error: expect(book.status).to eq('stack')

       expected: "stack"
            got: "rented"

       (compared using ==)
     # ./spec/models/book_spec.rb:12:in `block (4 levels) in <top (required)>'

  3) Book testing for book status assign enum value
  expects 'rented' to be mapped to 0
     Failure/Error: expect(Book.statuses[book.status]).to eq(0)

       expected: 0
            got: 1

       (compared using ==)
     # ./spec/models/book_spec.rb:19:in `block (4 levels) in <top (required)>'

  4) Book testing for book status assign enum
  value expects 'stack' to be mapped to 1
     Failure/Error: expect(Book.statuses[book.status]).to eq(1)

       expected: 1
            got: 2

       (compared using ==)
     # ./spec/models/book_spec.rb:23:in `block (4 levels) in <top (required)>'

  5) Book testing for book status enum attributes are available for
  where clause expects the number of 'rented' books is 7
     Failure/Error: expect(Book.rented.count).to eq(7)

       expected: 7
            got: 11

       (compared using ==)
     # ./spec/models/book_spec.rb:34:in `block (4 levels) in <top (required)>'

  6) Book testing for book status enum attributes are available for
  where clause expects the number of 'stack' book is 11
     Failure/Error: expect(Book.stack.count).to eq(11)

       expected: 11
            got: 0

       (compared using ==)
     # ./spec/models/book_spec.rb:38:in `block (4 levels) in <top (required)>'

Finished in 0.106 seconds (files took 1.63 seconds to load)
6 examples, 6 failures

Failed examples:

rspec ./spec/models/book_spec.rb:6 # Book testing for book status assign
primitive value expects 0 to be mapped to 'rented'
rspec ./spec/models/book_spec.rb:10 # Book testing for book status assign
primitive value expects 1 to be mapped to 'stack'
rspec ./spec/models/book_spec.rb:17 # Book testing for book status assign
enum value expects 'rented' to be mapped to 0
rspec ./spec/models/book_spec.rb:21 # Book testing for book status assign
enum value expects 'stack' to be mapped to 1
rspec ./spec/models/book_spec.rb:33 # Book testing for book status enum
attributes are available for where
clause expects the number of 'rented' books is 7
rspec ./spec/models/book_spec.rb:37 # Book testing for book status enum
attributes are available for where
clause expects the number of 'stack' book is 11
```

いままで苦労して作ったテストがすべてダメになりました。これはなぜでしょうか？`ActiveRecord::Enum`にバグがあるわけではありません。`enum`の追加の方法に問題があります。状態名を配列にした場合、対応する値は添字になるためです。なので、先頭に予約済（`reserved`）を追加した場合に結果が変わってしまいました。

追加する状態によらず、今まで提供してきた機能を崩さないためにはハッシュで状態を定義することが多いです。`book.rb`を次のように変更してテストを再度実行してみましょう。

```
  class Book < ApplicationRecord
    enum status: {rented: 0, stack: 1, reserved: 2}
  end
```
