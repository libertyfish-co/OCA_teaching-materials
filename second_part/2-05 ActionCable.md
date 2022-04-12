## 2.5 ActionCable

### 2.5.1 ActionCableについて
ActionCableとはRailsでWebSocketを扱うためのライブラリです。

通常のHTTP通信では

- リクエストを送るたびにコネクションが切れる
- リクエストはクライアントから送るだけで、双方向通信ができない

といった特徴があり、サーバ側で更新があった場合、クライアントをリロードするなりして再度リクエスト送らないと更新内容を受け取ることができません。

それらの問題を解消するためにWebSocketというプロトコルを利用します。

WebSocketを利用すると、一度確立したコネクションを利用して、サーバ・クライアント双方から通信を行うことができるようになります。

### 2.5.2 チャット機能の実装
それではさっそく、ActionCableを利用した簡単なチャットの仕組みを実装してみましょう。

チャットページを表示するためのコントローラを追加します。

```
rails g controller chat index
```

`views/chat/index.html.erb`

```
<h1>Chat#index</h1>
<div id="messages">
</div>

<form id="message_form">
  <input type="text" name="message" id="message_field">
  <input type="submit" id="post_message">
</form>
```

ここまでできたら、実際にアクセスしてみましょう。

http://hogehoge/chat/index

入力欄とボタンが表示されています。現時点では入力してボタンをクリックしても
同じページが表示されるだけです。

ここからActionCableを利用してチャットの動きを実装していきます。

ActionCableのコードを生成しましょう。

```
rails g channel room post_message
```

roomと言う名前のチャンネルを準備します。

出来上がったファイルを見てみましょう。

`app/channels/room_channel.rb`

```
class RoomChannel < ApplicationCable::Channel
  def subscribed
    # stream_from "some_channel"
  end

  def unsubscribed
    # Any cleanup needed when channel is unsubscribed
  end

  def post_message
  end
end
```

`RoomChannel` はサーバ側の処理を行うクラスです。
`subscribed`はクライアントと接続した際に呼び出されるメソッドです。
`unsubscribed`は接続を解除した際に呼び出されるメソッドです。

`app/javascript/channels/room_channel.js`

```
import consumer from "./consumer"

consumer.subscriptions.create("RoomChannel", {
  connected() {
    // Called when the subscription is ready for use on the server
  },

  disconnected() {
    // Called when the subscription has been terminated by the server
  },

  received(data) {
    // Called when there's incoming data on the websocket for this channel
  },

  post_message: function() {
    return this.perform('post_message');
  }
});
```
`consumer.subscriptions.create`で出来るオブジェクトは、クライアント側の処理を行うオブジェクトです。
`connected` は接続が繋がった時に呼び出される関数です。
`disconnected`は接続を解除した時に呼び出される関数です。
`received`はデータを受信した時に呼び出される関数です。

また、それぞれにpost_messageと言う関数やメソッドがあります。
これらを利用して、実際のチャットのメッセージを送信します。

クライアント側の`post_message`を呼び出すと、`return this.perform('post_message');`が実行されそこに記述した`'post_message'`と同じメソッドがサーバ側で実行されます。

以下を修正します。
`app/javascript/channels/room_channel.js`

```
var appRoom = consumer.subscriptions.create("RoomChannel", { #修正
```

下記の記述を追加します。
これはsubmitボタンがクリックされた時の動きをJavaScriptで記述したものです。
入力欄の内容を引数にして`appRoom.post_message`を呼び出しています。

`app/javascript/channels/room_channel.js`

```
document.addEventListener('DOMContentLoaded', function(){
  const form = document.getElementById('message_form')
  form.addEventListener('submit', function(e){
    e.preventDefault()
    var input = document.getElementById('message_field').value
    appRoom.post_message(input)
    document.getElementById('message_field').value = ''
  })
});
```

試しにsubmitボタンを押下し、ブラウザのコンソールを確認してみましょう。

すると、サーバ側のログに以下のログが追加されます。

```
RoomChannel#post_message
```

クライアント側のチャンネルのpost_messageを実行すると
サーバ側のチャンネルのpost_messageが呼び出されていることが確認できました。

次にサーバ側からクライアントへ通信するための実装を行います。

`RoomChannel`クラスの`subscribed`メソッドと`post_message`メソッドに以下の記述を追加します。

```
def subscribed
  stream_from 'chat' # 追加
end

def post_message(data) # 修正
  ActionCable.server.broadcast 'chat', message: data['message'] # 追加
end
```

`appRoom`のオブジェクトを修正します。

```
receive(data) { # 修正
  console.log(data) } # 追加
  
post_message: function(message) { # 修正
  return this.perform('post_message', {message: 'chat_message'}); #修正
```

ここまで記述できたら、もう一度submitボタンを押下し、ブラウザのコンソールを確認してみましょう。

するとブラウザのコンソールに以下の内容が表示されました。

```
{message: 'chat message'}
```

これはコンソールで呼び出したpost_messageの引数がサーバに渡り、サーバから各クライアントに通知が行われ表示されています。

試しに2つのブラウザを開いて、同じことをしてみましょう。
両方のブラウザのコンソールに通知が行われているはずです。


それでは実際にブラウザのフォームで入力したメッセージをサーバに送ってチャットを完成させましょう。

`appRoom`のオブジェクトを修正します。

```
post_message: function(message) {
    return this.perform('post_message', {message: message});} #修正
```

次にサーバから受けた内容をブラウザに表示するためのコードを追加します。
クライアント側のチャンネルの`received`関数に実装します。
受け取ったメッセージからDOMを生成してappendしています。

```
received(data) {
  var element = document.createElement('div')
  element.className = 'message'
  element.innerHTML = `<p>${data.message}</p>`
  document.getElementById('messages').appendChild(element)
}
```

これでブラウザの入力欄に文字を入力してsubmitボタンをクリックしてみましょう。
入力した内容が表示されたはずです。

また、2つのブラウザを開いて同じことをしてみましょう。
両方のブラウザに表示されます。
