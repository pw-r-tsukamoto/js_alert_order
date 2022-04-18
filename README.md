# js_alert_order

- jsのalert実行順序について研究したもの。


## 問題

- alertよりも上に書いたコードが、alertより後に実行されてしまう！
  - confilm, promptなどのダイアログ系の関数も同様とする。
  
```javascript

$('.ret').append('<p>tier1</p>');
alert('tier2');
$('.ret').append('<p>tier3</p>');

// これだと、tier2 -> tier1 -> tier3 になってしまう！
```

## 解決策

1. alertを使わず、似た機能を人力で実装する。(bootstrapにあるとかないとか)
2. alertを非同期処理の内側に書く
  - setTimeoutやonclick, thenのメソッドチェーンなど
  - 例：jQueryを利用した書き方
  ```javascript

    $('.ret').append('<p>tier1</p>');
    var dfdd = $.Deferred();
    dfdd.resolve()
    .then(function(){
      alert('tier2');
    })
    .then(function(){
      $('.ret').append('<p>tier3</p>');
    });

  // これなら、tier1 -> tier2 -> tier3 になる。

    $('.ret').append('<p>tier1</p>');
    var dfdd_2 = $.Deferred();
    dfdd_2.resolve()
    .then(function(){
      alert('tier2');
      $('.ret').append('<p>tier3</p>');
    });

  // これでもOK。tier1 -> tier2 -> tier3 になる。

  ```

## 分析パート

### 前提

- まず、Javascriptはプログラムを上から順番に**ではなく**ある特定の順序で実行する。以下の通り。

1. windowオブジェクトを生成
2. 「例外的な操作やI/Oブロッキングのある操作」をすべて一つのイベントキューに置く（ここではまだ実行しない。なんなのかよくわからん）
3. **DOM操作を除いた**Js(script内に書かれたJs)を上から順に実行、途中にDOM操作があれば飛ばす
4. ページのレンダリングを行う。ここでページが表示される。
5. 残りのJsコード(DOM操作など)を上から順番に実行する
6. DOMContentLoadedイベントを発生させる
7. 非同期処理を、それぞれに設定されたタイミングで実行
 
### alertはなぜ先に実行されてしまうのか？

- alertは上でいうところの「DOM操作を除いたJs」にあたる。
- DOM操作は必ず後回しにされてしまうため、結果として「DOM操作を先に書いたのに、alertが先に実行されてしまう」という現象が起きる

#### Tips

  - 3.では、DOMからの情報取得はできる。DOMへの変更が不可
  - DOM操作を組み込んだループの場合、ループがまるごと後回しにされる
  - **alertをはじめとするダイアログを開くと、閉じられるまでの間、それ以外のJsの処理は止まる**（時間カウント系もストップする）
      
## 参考資料

- https://jpdebug.com/p/184087
- https://kde.hateblo.jp/entry/2017/05/20/212928

- https://qiita.com/l1lhu1hu1/items/57dcc7cb867eee951f36
