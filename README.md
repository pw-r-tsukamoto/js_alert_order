# js_alert_order

- jsのalert実行順序について研究したもの。


## 問題

- alertよりも上に書いたコードが、alertより後に実行されてしまう！
  - confilm, promptなどのダイアログ系の関数も同様とする。

## 分析

### 前提

- まず、Javascriptはプログラムを上から順番に**ではなく**ある特定の順序で実行する。
  - 以下の通り。
  1. windowオブジェクトを生成
  2. 「例外的な操作やI/Oブロッキングのある操作」をすべて一つのイベントキューに置く（ここではまだ実行しない。なんなのかよくわからん）
  3. **DOM操作を除いた**Js(script内に書かれたJs)を実行
  4. ページのレンダリングを行う。ここでページが表示される。
  5. イベントキューに置かれた同期的処理(script内に書かれたJsのうち、DOM操作など)を上から順番に実行する
  6. DOMContentLoadedイベントを発生させる
  7. 非同期処理を、それぞれに設定されたタイミングで実行
 
### alertはなぜ先に実行されてしまうのか？

- alertは上でいうところの「DOM操作を除いたJs」にあたる。
- DOM操作は必ず後回しにされてしまうため、結果として「DOM操作を先に書いたのに、alertが先に実行されてしまう」という現象が起きる

#### Tips

  - 3.では、DOMからの情報取得はできる。DOMへの変更が不可
  - DOM操作を組み込んだループの場合、ループがまるごと後回しにされる
 
  
  
      console.log('tier ZZZ');
      alert('tier2');
      console.log('tier FFF');
      $('.ret').append('<p>tier1</p>');
      alert(document.readyState); // -> 初回 'loading'
      document.addEventListener('readystatechange', function () {   
        alert(document.readyState); // -> 2回目 'interactive'、 3回目 'complate'
      });
  
  
  tier ZZZ　-> tier2 -> tier FFF -> loading -> きんいろモザイク -> tier1 -> interactive -> complate
  
 
### 解決策

- alertを人力で実装してしまう。(bootstrapにあるとかないとか)
- alertを非同期処理の内側に書く
  - setTimeoutやonclick, thenのメソッドチェーンなど
      
      
## 参考資料

- https://jpdebug.com/p/184087
- https://kde.hateblo.jp/entry/2017/05/20/212928

- https://qiita.com/l1lhu1hu1/items/57dcc7cb867eee951f36