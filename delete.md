---
sort: 2
---
# 削除機能をもったカート一覧画面の作成

まずは、カート内の商品を削除する機能を実装します。

## ルーティングの設定

`routes/web.php`に以下のルーティングを追加します。

```php
<?php

use Illuminate\Support\Facades\Route;
use App\Http\Controllers\ItemController;
use App\Http\Controllers\CartController;

// 途中省略

Route::get('cart/create', [CartController::class, 'create'])->name('cart.create');
Route::post('cart', [CartController::class, 'store'])->name('cart.store');
// --- 以下を追加 ---
Route::get('cart', [CartController::class, 'index'])->name('cart.index');
Route::delete('cart/{cart}',[CartController::class, 'destroy'])->name('cart.destroy');
```

**【解説】**

`Route::get('cart', [CartController::class, 'index'])->name('cart.index');`: <br>
カート内の商品を一覧表示するためのルーティングです。
`CartController`の`index`メソッドを呼び出します。

`Route::delete('cart/{cart}',[CartController::class, 'destroy'])->name('cart.destroy');`: <br>
カート内の商品を削除するためのルーティングです。
`CartController`の`destroy`メソッドを呼び出します。
`Route::delete`メソッドを使って、HTTPメソッドが`DELETE`のリクエストを受け付けるように設定しています。

今までのリクエストは`GET`と`POST`だけでしたが、今回は`DELETE`リクエストを使います。
`GET`や`POST`を使っても同様のルーティングを設定できますが、処理の性質によって使い分けます。

## ビューの作成(カート一覧画面の作成とリンク修正)

`resources/views/cart/index.blade.php`を作成し、以下のように記述します。

{% raw %}

```php
<!DOCTYPE html>
<html lang="ja">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0">
<title>サンプル</title>
</head>
<body>
    <h3>カート一覧</h3>
    @if (session('message'))
        <font color="red">{{ session('message') }}</font>
    @endif
    @if( count($carts) == 0 )
        <p>カート内に商品はありません</p>
    @else
        <table border="1">
        <tr>
            <th>商品番号</th>
            <th>数量</th>
            <th>削除</th>
        </tr>
        @foreach( $carts  as  $cart )
            <tr>
                <td> {{ $cart->ident }} </td>
                <td> {{ $cart->quantity }} </td>
                <td>
                    <form method="POST" action="{{ route('cart.destroy', ['cart' => $cart->ident]) }}">
                        @csrf
                        @method('DELETE')
                        <input type="submit" value="削除">
                    </form>
                </td>
            </tr>
        @endforeach
        </table>
    @endif
    <a href="{{ route('cart.create') }}">カートに追加</a>
</body>
</html>
```

**【解説】**

`<form method="POST" action="{{ route('cart.destroy', ['cart' => $cart->ident]) }}">`: <br>
削除ボタンを押すと、カート内の商品が削除されるように設定しています。
`route`関数の第2引数に`['cart' => $cart->ident]`を指定しています。
これにより、`$cart->ident`の値が`{cart}`に代入されます。

`@method('DELETE')`: <br>
`DELETE`リクエストを使うことを指定しています。
HTTPのリクエストメソッドにおける`DELETE`リクエストは、リソースの削除を行うためのメソッドです。

`GET`リクエストや`POST`リクエストを使う場合は特に指定しなかったのに対し、`DELETE`リクエストを使う場合は明示的に指定する必要があります。
なぜなら、HTMLの`form`の`method`属性には`DELETE`リクエストがサポートされていないためです。
そのため、`@method('DELETE')`を使って、`DELETE`リクエストを使うことを明示的に指定しています。
これにより、ルーティングを見るだけで、削除機能があることがわかります。

次に、カート追加画面からカート一覧画面に遷移するリンクを修正します。

**resources/views/cart/create.blade.php**

```php
<!DOCTYPE html>
<html lang="ja">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0">
<title>サンプル</title>
</head>
<body>
    <h3>カートに追加</h3>
    @if (session('message'))
        <font color="red">{{ session('message') }}</font>
    @endif
    @if ($errors->any())
        @foreach ($errors->all() as $error)
            <font color="red">{{ $error }}</font><br>
        @endforeach
    @endif
    <form action="{{ route('cart.store') }}" method="POST">
    @csrf
    番号:<input type="number" name="ident" min="1" max="15"><br>
    数量:<input type="number" name="quantity" min="1" max="10"><br><br>
    <input type="submit" value="カートに追加">
    </form>
    <!-- 以下を追加 -->
    <a href="{{ route('cart.index') }}">カート一覧へ</a>
    <!-- ここまで -->
</body>
</html>
```

{% endraw %}

## コントローラを修正(`index`メソッドと`destroy`メソッドの追加)

`app/Http/Controllers/CartController.php`を以下のように修正します。

```php
<?php
namespace App\Http\Controllers;

use Illuminate\Http\Request;
use App\Models\Cart;

class CartController extends Controller
{
    
    // 途中省略

    // --- 以下を追加 ---
    public function index()
    {
        $carts = Cart::all();
        return view('cart.index', ['carts' => $carts]);
    }

    public function destroy(Request $request, Cart $cart)
    {
        $cart->delete();
        $request->session()->flash('message', '削除しました');
        return redirect()->route('cart.index');
    }
    // --- ここまで追加 ---
}
```

**【解説】**

`public function destroy(Cart $cart)`: <br>
カート内の商品を削除するためのメソッドです。
Laravelでは、コントローラのメソッドにdestroyと命名する場合、削除処理を行うことが一般的です。

`$cart->delete();`: <br>
`$cart`の`delete`メソッドを使って、レコードを削除しています。
この記述のみで、指定したレコードが削除されます。
ここでいう指定したレコードとは、ルーティングで指定した`{cart}`の値が`$cart`に代入されたレコードです。
つまり、`$cart->delete();`は、`Cart::find($cart->ident)->delete();`と同じ意味になります。
以前もお伝えしましたが、Laravelのこの機能のことをルートモデルバインディングといいます。

`return redirect()->route('cart.index');`: <br>
削除処理が終わったら、カート内の商品一覧画面にリダイレクトします。

以上で、カート内の商品を削除する機能が実装できました。

## 動作確認(1回目:削除機能)

以下の手順で動作確認をしてみましょう。

1. `http://localhost:{ポート番号}/cart`でカート一覧画面にアクセスする<br>
    ![](./images/delete_kakunin1.png)
2. カート追加画面にアクセスする<br>
3. 商品番号と数量を入力し、カートに追加する<br>
    ![](./images/delete_kakunin2.png)
   ![](./images/delete_kakunin3.png)
4. カート一覧画面に遷移し、商品が追加されていることを確認する<br>
   ![](./images/delete_kakunin4.png)
5. 削除ボタンを押し、商品がカート内から削除されていることを確認する<br>
   ![](./images/delete_kakunin5.png)