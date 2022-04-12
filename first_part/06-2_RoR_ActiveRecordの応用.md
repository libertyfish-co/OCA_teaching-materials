## 6.2 Ruby on Rails: ActiveRecordの応用

ここではActiveRecordの応用的な利用方法「N+1問題について」「サブクエリの作り方」について学習します。

### 6.2.1 「N+1問題」とは

一覧画面などに表示するデータを取得するとき、SQLのクエリが「データ量(N) + 1」発行され、データ量が多くなるにつれてパフォーマンスを低下させてしまう問題です。例えばデータの数だけSELECT文を発行すると1回が1msでも、データの数が増えるとその数だけSELECT文を発行することになり、パフォーマンスが悪くなっていきます。

では、N+1問題について具体的なコードを見ながら考えてみましょう。

Modelは `User` と `Favorite` のものがあったとします。

`User: ユーザーテーブル`

| field名 | 名称 | 型 |
|---|---|---|
| id | ID | integer |
| name | 名前 | string |

`Favorite: お気に入りテーブル`

| field名 | 名称 | 型 |
|---|---|---|
| id | ID | integer |
| user_id | ユーザーID | integer |
| title | タイトル | string |

ユーザー(User)はたくさんのお気に入り(Favorite)を登録することができるという「1 対 多」の関連付をします。

`app/models/user.rb`

```ruby
class User < ApplicationRecord
  has_many :favorites
end
```

`app/models/favorite.rb`

```ruby
class Favorite < ApplicationRecord
  belongs_to :user
end
```

Controllerでは `Favorite` の一覧を取得するよう以下の内容になっているとします。

```ruby
class Favorite < ApplicationController
  def index
    @favorites = Favorite.order(:id)
  end
end
```

`View` では取得したお気に入りに関連する `user` の名前を表示するとします。

```ruby
<h1>お気に入り一覧</h1>
  <% @favorites.each do |fav| %>
    <div><%= fav.user.name %></div>
  <% end %>
```

「お気に入り一覧を表示する」という機能的には問題なく満たしていますが、レコードを取得する際のデータベースへのアクセスに問題があります。

ログを確認してみましょう。

**ログ**

`users`テーブルへの `SELECT` が `favorite` の数だけ行われています。
これではレコードが増えれば増えるほどSQLのクエリが発行されてしまいパフォーマンスが非常に悪くなってしまいます。
例えば1回のクエリにかかる時間が0.1ms前後だったとして、取得するレコードが何千何万とあった場合それだけ時間がかかってしまうということになります。これが「N+1問題」の実態です。

```bash
Favorite Load (0.5ms)  SELECT "favorites".* FROM "favorites" ORDER BY "favorites"."id" ASC
  User Load (0.2ms)  SELECT  "users".* FROM "users" WHERE "users"."id" = ? LIMIT ?  [["id", 1], ["LIMIT", 1]]
  CACHE User Load (0.0ms)  SELECT  "users".* FROM "users" WHERE "users"."id" = ? LIMIT ?  [["id", 1], ["LIMIT", 1]]
  CACHE User Load (0.1ms)  SELECT  "users".* FROM "users" WHERE "users"."id" = ? LIMIT ?  [["id", 1], ["LIMIT", 1]]
  CACHE User Load (0.0ms)  SELECT  "users".* FROM "users" WHERE "users"."id" = ? LIMIT ?  [["id", 1], ["LIMIT", 1]]
  CACHE User Load (0.0ms)  SELECT  "users".* FROM "users" WHERE "users"."id" = ? LIMIT ?  [["id", 1], ["LIMIT", 1]]
  CACHE User Load (0.0ms)  SELECT  "users".* FROM "users" WHERE "users"."id" = ? LIMIT ?  [["id", 1], ["LIMIT", 1]]
  CACHE User Load (0.0ms)  SELECT  "users".* FROM "users" WHERE "users"."id" = ? LIMIT ?  [["id", 1], ["LIMIT", 1]]
  CACHE User Load (0.0ms)  SELECT  "users".* FROM "users" WHERE "users"."id" = ? LIMIT ?  [["id", 1], ["LIMIT", 1]]
  CACHE User Load (0.0ms)  SELECT  "users".* FROM "users" WHERE "users"."id" = ? LIMIT ?  [["id", 1], ["LIMIT", 1]]
  CACHE User Load (0.1ms)  SELECT  "users".* FROM "users" WHERE "users"."id" = ? LIMIT ?  [["id", 1], ["LIMIT", 1]]
  CACHE User Load (0.0ms)  SELECT  "users".* FROM "users" WHERE "users"."id" = ? LIMIT ?  [["id", 1], ["LIMIT", 1]]
  CACHE User Load (0.0ms)  SELECT  "users".* FROM "users" WHERE "users"."id" = ? LIMIT ?  [["id", 1], ["LIMIT", 1]]
  CACHE User Load (0.0ms)  SELECT  "users".* FROM "users" WHERE "users"."id" = ? LIMIT ?  [["id", 1], ["LIMIT", 1]]
  CACHE User Load (0.0ms)  SELECT  "users".* FROM "users" WHERE "users"."id" = ? LIMIT ?  [["id", 1], ["LIMIT", 1]]
  CACHE User Load (0.0ms)  SELECT  "users".* FROM "users" WHERE "users"."id" = ? LIMIT ?  [["id", 1], ["LIMIT", 1]]
  CACHE User Load (0.0ms)  SELECT  "users".* FROM "users" WHERE "users"."id" = ? LIMIT ?  [["id", 1], ["LIMIT", 1]]
  CACHE User Load (0.0ms)  SELECT  "users".* FROM "users" WHERE "users"."id" = ? LIMIT ?  [["id", 1], ["LIMIT", 1]]
  CACHE User Load (0.0ms)  SELECT  "users".* FROM "users" WHERE "users"."id" = ? LIMIT ?  [["id", 1], ["LIMIT", 1]]
  CACHE User Load (0.0ms)  SELECT  "users".* FROM "users" WHERE "users"."id" = ? LIMIT ?  [["id", 1], ["LIMIT", 1]]
  CACHE User Load (0.0ms)  SELECT  "users".* FROM "users" WHERE "users"."id" = ? LIMIT ?  [["id", 1], ["LIMIT", 1]]
  User Load (0.1ms)  SELECT  "users".* FROM "users" WHERE "users"."id" = ? LIMIT ?  [["id", 2], ["LIMIT", 1]]
  CACHE User Load (0.0ms)  SELECT  "users".* FROM "users" WHERE "users"."id" = ? LIMIT ?  [["id", 2], ["LIMIT", 1]]
  CACHE User Load (0.0ms)  SELECT  "users".* FROM "users" WHERE "users"."id" = ? LIMIT ?  [["id", 2], ["LIMIT", 1]]
  CACHE User Load (0.0ms)  SELECT  "users".* FROM "users" WHERE "users"."id" = ? LIMIT ?  [["id", 2], ["LIMIT", 1]]
  CACHE User Load (0.0ms)  SELECT  "users".* FROM "users" WHERE "users"."id" = ? LIMIT ?  [["id", 2], ["LIMIT", 1]]
  CACHE User Load (0.0ms)  SELECT  "users".* FROM "users" WHERE "users"."id" = ? LIMIT ?  [["id", 2], ["LIMIT", 1]]
  CACHE User Load (0.0ms)  SELECT  "users".* FROM "users" WHERE "users"."id" = ? LIMIT ?  [["id", 2], ["LIMIT", 1]]
  CACHE User Load (0.0ms)  SELECT  "users".* FROM "users" WHERE "users"."id" = ? LIMIT ?  [["id", 2], ["LIMIT", 1]]
  CACHE User Load (0.0ms)  SELECT  "users".* FROM "users" WHERE "users"."id" = ? LIMIT ?  [["id", 2], ["LIMIT", 1]]
  CACHE User Load (0.0ms)  SELECT  "users".* FROM "users" WHERE "users"."id" = ? LIMIT ?  [["id", 2], ["LIMIT", 1]]
  CACHE User Load (0.0ms)  SELECT  "users".* FROM "users" WHERE "users"."id" = ? LIMIT ?  [["id", 2], ["LIMIT", 1]]
  CACHE User Load (0.0ms)  SELECT  "users".* FROM "users" WHERE "users"."id" = ? LIMIT ?  [["id", 2], ["LIMIT", 1]]
  CACHE User Load (0.0ms)  SELECT  "users".* FROM "users" WHERE "users"."id" = ? LIMIT ?  [["id", 2], ["LIMIT", 1]]
  CACHE User Load (0.0ms)  SELECT  "users".* FROM "users" WHERE "users"."id" = ? LIMIT ?  [["id", 2], ["LIMIT", 1]]
  CACHE User Load (0.0ms)  SELECT  "users".* FROM "users" WHERE "users"."id" = ? LIMIT ?  [["id", 2], ["LIMIT", 1]]
  CACHE User Load (0.0ms)  SELECT  "users".* FROM "users" WHERE "users"."id" = ? LIMIT ?  [["id", 2], ["LIMIT", 1]]
  CACHE User Load (0.0ms)  SELECT  "users".* FROM "users" WHERE "users"."id" = ? LIMIT ?  [["id", 2], ["LIMIT", 1]]
  CACHE User Load (0.0ms)  SELECT  "users".* FROM "users" WHERE "users"."id" = ? LIMIT ?  [["id", 2], ["LIMIT", 1]]
  CACHE User Load (0.0ms)  SELECT  "users".* FROM "users" WHERE "users"."id" = ? LIMIT ?  [["id", 2], ["LIMIT", 1]]
  CACHE User Load (0.0ms)  SELECT  "users".* FROM "users" WHERE "users"."id" = ? LIMIT ?  [["id", 2], ["LIMIT", 1]]
  ...
  ```

### 4.6.2 「N+1問題」解消方法

解決方法としては以下の4つのメソッドを使用するのが一般的です。<br>
メソッドは組み合わせて使うこともできます。

#### (a) includesを使用する

引数には、Association先を指定します。

```ruby
def index
  @favorites = Favorite.order(:id).includes(:user)
end
```

するとクエリの実行は2回で済みます。
1回目でお気に入りを全件取得し、2回目で `user_id` を指定して紐づく `user` を取得しています。

```bash
Favorite Load (1.0ms)  SELECT "favorites".* FROM "favorites" ORDER BY "favorites"."id" ASC
User Load (0.1ms)  SELECT "users".* FROM "users" WHERE "users"."id" IN (1, 2, 3, 4, 5, 6, 7, 8, 9)
```

#### (b) preloadを使用する

```ruby
def index
  @favorites = Favorite.preload(:user).order(:id)
end
```

```bash
Favorite Load (11.2ms)  SELECT "favorites".* FROM "favorites" ORDER BY "favorites"."id" ASC
User Load (0.3ms)  SELECT "users".* FROM "users" WHERE "users"."id" IN (1, 2, 3, 4, 5, 6, 7, 8, 9)
```

#### (c) eager_loadを使用する

```ruby
def index
  @favorites = Favorite.eager_load(:user).order(:id)
end
```

引数で指定したAssociationを `LEFT OUTER JOIN`(左外部結合)します。
1回のクエリで済みます。

```sql
SELECT
    "favorites"."id" AS t0_r0, 
    "favorites"."user_id" AS t0_r1, 
    "favorites"."title" AS t0_r2, 
    "favorites"."created_at" AS t0_r3, 
    "favorites"."updated_at" AS t0_r4, 
    "users"."id" AS t1_r0, 
    "users"."name" AS t1_r1, 
    "users"."created_at" AS t1_r2, 
    "users"."updated_at" AS t1_r3
  FROM 
    "favorites" 
    LEFT OUTER JOIN "users" 
      ON "users"."id" = "favorites"."user_id" 
 ORDER BY 
    "favorites"."id" ASC
```

#### (d) joinsを使用する

joinsは、指定したAssociationを `INNER JOIN` します。
これだけでは「N+1問題」の解消はできませんが、結合先のテーブルに対しての絞り込みが可能です。

```ruby
def index
  @favorites = Favorite.joins(:user).where(users: {id: 1})
end
```

```bash
Favorite Load (0.2ms)  SELECT "favorites".* FROM "favorites" INNER JOIN "users" ON "users"."id" = "favorites"."user_id" WHERE "users"."id" = ?  [["id", 1]]
```

### 4.6.3 サブクエリの作り方

Userテーブルから抽出する場合

**時間がかかるクエリ(SQL構文)**

userテーブルのデータを複数取得したい場合、id毎にクエリを発行すると、データベースへのアクセス回数が増えパフォーマンスが悪くなってしまいます。

```sql
SELECT * FROM users WHERE id = 1;
SELECT * FROM users WHERE id = 2;
SELECT * FROM users WHERE id = 3;
.
.
.
```

**INを使用したクエリ(SQL構文)**

userテーブルから複数のデータを取得したい場合IN句を使用すれば発行するクエリは1回で済みます。

```sql
SELECT * FROM users WHERE id IN (1, 2, 3);
```

**Railsで書いた場合**

railsでuserテーブルから1度に複数のデータを取得する場合を見てみましょう。

配列で指定する

```ruby
User.where(id: [1, 2, 3])
```

`ids`メソッドを使用し同じことができます

```ruby
User.where(id: User.ids)
```

どのようなSQLクエリが実行されているのか`to_sql`メソッドを使用して確認してみましょう。
  
```ruby
User.where(id: [1, 2, 3]).to_sql

# => 'SELECT "users".* FROM "users" WHERE "users"."id" IN (1, 2, 3)'
```

**複数条件で絞り込みを行う**

`where`メソッドをチェーンしサブクエリを発行します。

```bash
Favorite.where(title: "title1").where(user: User.where(name: "name2"))
```

`to_sql`メソッドで確認してみましょう。<br>
`favorites`テーブルに対してサブクエリを含む2つの条件(AND条件)で絞り込みを行うクエリが発行されているのが確認できると思います。
  
```sql
SELECT "favorites".* 
  FROM "favorites" 
 WHERE "favorites"."title" = 'title1' 
   AND "favorites"."user_id" IN (
       SELECT "users"."id" FROM "users" WHERE "users"."name" = 'name2'
   )
```

`or`メソッドを使用し、OR条件で絞り込みを行う。<br>
Rails5系から`or`メソッドが実装されました。

```bash
User.where(id: 1).or(User.where(name: "user2"))
```

`to_sql`メソッドで確認してみましょう。<br>
`users`テーブルに対して2つの条件(OR条件)で絞り込みを行うクエリが発行されているのが確認できると思います。

```sql
SELECT "users".* 
  FROM "users" 
 WHERE (
   "users"."id" = 1 OR 
   "users"."name" = 'user2'
 )
```

`not`メソッドを使い否定する。<br>
`not`メソッドを使用し`where`メソッドなどで指定した条件を否定することができます。
  
```bash
User.where.not(id: 1)
```

`to_sql`メソッドで確認すると「idが1ではない(`"id" != 1"`)」というクエリが発行されているのがわかります。

```sql
SELECT "users".* 
  FROM "users" 
 WHERE "users"."id" != 1
```
