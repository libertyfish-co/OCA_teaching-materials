## 13.1 Ruby on Rails：gemを使わない検索2

### 13.1.1 検索処理の作成

画面で入力した内容を受け取って検索を実行します。

`app/controllers/customers_controller.rb`

```
  # 変更前
  def index
    @customers = Customer.all
  end
  
  # 変更後
  def index
    return @customers = Customer.all if params[:name].blank? && params[:age].blank?

    # 条件が2つとも指定されている場合
    if params[:name].present? && params[:age].present?
      @customers = Customer.where(["name like ? and age = ?", 
                                  "%#{params[:name]}%",
                                  "#{params[:age]}"]);
    else
      # nameの条件だけ指定されている場合
      if params[:name].present?
        @customers = Customer.where(["name like ?", "%#{params[:name]}%"]);
      # ageの条件だけ指定されている場合
      else
        @customers = Customer.where(["age = ?", "#{params[:age]}"]);
      end
    end
  end
```

検索画面の検索条件には、`Customer`に対する`name`と、`Customer`に対する`age`の検索条件を配置しました。

検索画面で入力した検索条件は、画面の`submit`ボタンをクリックすると、`CustomersController`に検索条件が渡されます。

`CustomersController`は、渡された検索条件`params`に含まれる条件`:name`と`:age`を取り出して検索条件とします。

`present?`メソッドを使用して、検索条件が入力されているか判断します。

`Customer`の`name`に対しては、like 構文で`%`を使用して部分一致検索をするように記述しているので、nameの検索条件に'岸辺'と入力すると`like '%岸辺%'`と検索のための構文を作成します。

2つの項目を使って、検索する組み合わせは

|name|age|検索対象|
|---|---|---|
|条件なし|条件なし|検索条件なし(全て対象)|
|条件あり|条件あり|nameに部分一致、かつageに一致|
|条件あり|条件なし|nameに部分一致|
|条件なし|条件あり|ageに一致|

になります。

`rails s`でサーバを起動して`http://localhost:3000/customers`にアクセスしてみましょう。
検索フォームと検索結果が表示されていると思います。

条件を入力して検索してみましょう。

また、条件が多い場合は、以下のように記述することもできます。

```
  # 変更後
  def index
    @customers = Customer.all

    if params[:name].present?
      @customers = @customers.where("name like ?", "%#{params[:name]}%")
    end

    if params[:age].present?
      @customers = @customers.where(age: params[:age])
    end
  end
```

検索項目の組み合わせを考慮せず、検索項目が入力された項目だけをSQLのwhere句として組み立てます。

複数の条件がある場合でも、@customersにwhereを継ぎ足していくことで、検索の条件を追加できます。

#### ポイント

上記の例では、直接SQL文を組み立てていませんが、直接組み立てる場合は検索条件に入力される条件によっては全く予期しないSQL文になる場合があります。

例えば、nameの欄に`' or 1=1'`と入力して、`search`ボタンをクリックすると、Railsのログには
```
Started GET "/customers?utf8=%E2%9C%93&name=%27+or+1%3D1%27&age=&commit=Search" for 127.0.0.1 at 2019-11-16 12:03:12 +0900
Processing by CustomersController#index as HTML
  (略)
  Customer Load (0.2ms)  SELECT "customers".* FROM "customers" WHERE (name like '%'' or 1=1''%')
  (略)
Completed 200 OK in 54ms (Views: 51.5ms | ActiveRecord: 0.2ms)
```
のように実行結果が表示されます。nameに対する検索条件が`(`と`)`でくくられ、入力した条件がそのまま文字列として検索条件になるように`' or 1=1'`が`'' or 1=1''`と置換され、SQLで文字列や時刻の値をくくるための記号`'`が`''`に置換され、無害な文字列`'`と解釈されるようになります。

考え方の一例ですが、SQL文を組み立てる場合に、`name like '%' + 条件 + '%'`のように記述して、検索すると
```
SELECT customers.* FROM customers WHERE name like '%' or 1=1;
```
と、nameに対する条件以外に、or 1=1など、常に条件が成立するSQL文が含まれ、本来は表示できてはいけないデータが表示されることも考えられますので、Railsでは、
```
@customers.where("name like ?", "%#{params[:name]}%")
```
のように記述するようにしましょう。

このように、入力された条件で期待している結果が取得できるテストをすることはもちろんですが、入力された条件によって期待している結果以外の動作をしないこともテストをして問題がないことを確認するようにしましょう。
