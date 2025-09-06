# まとめ

本章までで、Laravelを使って、CRUD機能(Create（作成）、Read（読み取り）、Update（更新）、Delete（削除）)を実装する方法を学びました。

また、できあがったカートについてのルーティングは以下のようになります。

```php
Route::get('cart', [CartController::class, 'index'])->name('cart.index');
Route::get('cart/create', [CartController::class, 'create'])->name('cart.create');
Route::post('cart', [CartController::class, 'store'])->name('cart.store');
Route::delete('cart/{cart}',[CartController::class, 'destroy'])->name('cart.destroy');
Route::patch('cart/{cart}',[CartController::class, 'update'])->name('cart.update');
```

ルーティングを見るだけで、読み取り(`index`)、追加(`create`、`store`)、削除(`delete`)、更新(`update`)の機能があることがわかります。