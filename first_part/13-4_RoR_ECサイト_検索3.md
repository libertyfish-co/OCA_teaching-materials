## 13.4 Ruby on Rails：ECサイトの開発 検索3

### 13.4.1 例題(つづき)

#### ソート機能の追加
`ransack`が提供する`ViewHelper`の`sort_link`を利用することで簡単にソート機能を実装することが出来ます。

`app/views/customers/index.html.erb`

```
# 変更前
<table>
  <thead>
    <tr>
      <th>Employee</th>
      <th>Name</th>
      <th>Age</th>
      <th colspan="3"></th>
    </tr>
  </thead>
  
  
# 変更後
<table>
  <thead>
    <tr>
      <th><%= sort_link(@q, :employee_name, 'Employee') %></th>
      <th><%= sort_link(@q, :name, 'Name') %></th>
      <th><%= sort_link(@q, :age, 'Age') %></th>
      <th colspan="3"></th>
    </tr>
  </thead>
```
変更したら、検索結果のテーブルのヘッダ部分をクリックしてみましょう。
クリックした項目でソートされるようになっています。

### 13.4.2 問題

ECサイトの商品一覧に下記の項目で絞り込み検索ができるように機能を追加しましょう

- タイトル
- 著者
- 価格
