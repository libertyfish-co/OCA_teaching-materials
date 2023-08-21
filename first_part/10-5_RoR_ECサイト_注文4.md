## 10.5 Ruby on Rails：ECサイトの開発 注文4

### 10.5.1 テストの追加

ここでは、
- 商品詳細画面に購入ボタンが表示されている
- 購入ボタンをクリックすると、個数と届け先の入力フォームや次へボタン等が表示される
- 次へボタンを押すと確認画面に遷移される
- 確認画面にて購入確定ボタンをクリックすると購入完了画面が表示される

という動作をする注文画面について、

- 購入ボタンをクリックすると、購入情報入力画面が表示できる
- 購入確定ボタンをクリックすると、購入完了画面が表示できる

ことを確認するE2Eのテストを作成していきます。

#### E2Eテストの準備

テストコードを記述する `products_spec.rb` を作成します。

```bash
touch spec/system/products_spec.rb
```

それから、今回のテストで使用するテストデータをFactoryBotに用意しておきます。

`spec/factories/books.rb`

```ruby
FactoryBot.define do
  factory :book do
    title { "テスト用本" }
    author { "テスト太郎" }
    published_on { "1981-09-01" }
    showing { true }
    price { 1020 }
  end
end
```

#### 商品詳細画面にて購入ボタンをクリックすると、購入情報入力画面が表示できる

それでは、実際に画面の作成と、テストを記述していきましょう。

まずは、商品詳細画面とそのテストから記述します。

`app/views/products/show.html.erb`

```ruby
<h2 class="sub-header">商品詳細画面</h2>

<p id="notice"><%= notice %></p>

<p>
  <strong>Title:</strong>
  <%= @book.title %>
</p>

<p>
  <strong>Author:</strong>
  <%= @book.author %>
</p>

<p>
  <strong>Published on:</strong>
  <%= @book.published_on %>
</p>

<p>
  <strong>Showing:</strong>
  <%= @book.showing %>
</p>

<p>
  <strong>Price:</strong>
  <%= @book.price %>
</p>

<%= link_to '購入', new_order_path(product_id: @book.id), class: 'btn btn-default' %>
<%= link_to 'Back', products_path %>

```

`app/views/orders/new.html.erb`

```ruby
<%= form_with(model: @order, local: true, url: confirm_orders_path) do |f| %>
  <% if @order.errors.any? %>
    <div id="error_explanation">
    <h2><%= pluralize(@order.errors.count, "error") %> prohibited this order from being saved:</h2>

    <ul>
      <% order.errors.full_messages.each do |message| %>
        <li><%= message %></li>
      <% end %>
    </ul>
    </div>
  <% end %>

  <h2 class="sub-header">商品</h2>

  <p>
    <strong>タイトル：</strong>
    <%= @product.title %>
  </p>

  <div class="field">
    <%= f.label :'個数' %>
    <%= f.text_field :count %>
  </div>


  <div class="field">
    <%= f.label :'届け先' %>
    <%= f.text_field :address %>
  </div>

  <%= f.hidden_field :product_id, value: @product.id %>

  <div class="actions">
    <%= f.submit '次へ', class: "btn btn-default" %>
  </div>
<% end %>
```

`spec/system/products_spec.rb`

```ruby
require 'rails_helper'

RSpec.describe "Products", type: :system do
  describe "GET /products/:id" do
    it "renders product_path(id)" do
      book = FactoryBot.create(:book)
      visit product_path(book.id)
      click_link "購入"
      assert_text "#{book.title}"
      have_button '次へ'
    end
  end
end
```

#### 確認画面にて購入確定ボタンをクリックすると、購入完了画面が表示できる

続いて、購入確定ボタンをクリックすると、購入完了画面が表示されるテストを記述します。

`app/controllers/orders_controller.rb`

```ruby
class OrdersController < ApplicationController
  def create
    @order = Order.new(order_params)

    if params[:back].present?
      render :new
    else
      if @order.save
        redirect_to products_path, notice: '登録が完了しました。'
      else
        render :new
      end
    end
  end
  
  private
    def order_params
      params.require(:order).permit(:product_id, :count, :address)
    end
end
```

`spec/system/products_spec.rb`

```ruby
require 'rails_helper'

RSpec.describe "Products", type: :system do
  describe "GET /products/:id" do
  (省略)
  end

  describe "POST /orders" do
    it "renders products_path" do
      book = FactoryBot.create(:book)
      visit product_path(book.id)
      click_link "購入"
      assert_text "#{book.title}"
      fill_in "order_count", with: 1
      fill_in "order_address", with: "大阪市"
      click_button '次へ'
      expect(current_path).to eq confirm_orders_path
      assert_text "#{book.title}"
      assert_text "1"
      assert_text "大阪市"
      click_button '注文確定'
      expect(current_path).to eq products_path
      assert_text "登録が完了しました。"
    end
  end
end
```

### 10.5.2 まとめ

- 基本の７つ以外のルーティングが必要な場合は`member`や`collection`を利用しましょう。
- `form_with`にurlを指定することで、submit時に任意のURLに遷移できます。
- `render`でレイアウトを指定することで、レイアウトを変更できます。

