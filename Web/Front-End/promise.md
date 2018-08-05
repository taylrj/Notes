# Promise
[Reference](https://eyesofkids.gitbooks.io/javascript-start-es6-promise/content/contents/promise_a_plus.html)
- Promise 結構是異步執行的控制流程架構

## 異步執行流程語法結構之實作內容（ES6）
  - 一個建構函式
  - 一個 `then` 方法
  - 一個 `catch` 方法
  - 加上四個必須以 Promise 關鍵字呼叫之靜態方法
    - Promise.resolve
    - Promise.reject
    - Promise.all
    - Promise.race
  - 重點功能：
    - 把多個異步執行的函式，執行流程轉變為序列執行（一個完成再接著做下一個），或並行執行（全部處理完）
    - 異步程式設計與 Promise
      - Promise 物件設計：要不用一個回傳值來變成已實現(`fullfilled`)狀態，要不就是用一個理由(錯誤)來變成已拒絕(`rejected`)狀態

## Promises/A+標準定義
  - 名詞定義：
    - promise (承諾)是一個帶有遵照這個規格的then方法的物件
      - promise 物件代表的是一個懸而未決的值
    - thenable 是一個有定義then方法的物件
    - value 合法的JavaScript值(包含undefined、thenable與promise)
    - exception (例外)使用throw語句丟出來的值
    - reason (理由)是表明為什麼promise被拒絕(rejected)的值
    - settled(固定的) 一個 promise最後的狀態，也就是fulfilled(已實現)或rejected(已拒絕)
  - 狀態定義：
    - promise物件必定是以下三種狀態中的其中一種: pending(等待中)、fulfilled(已實現)或rejected(已拒絕)
    - 一開始在建立時都是處於 pending 之狀態
    - 當處在pending(等待中)時，一個promise:
      - 可能會轉變到不是fulfilled(已實現)就是rejected(已拒絕)狀態
    - 當處在fulfilled(已實現)時，一個promise:
      - 必定不會再轉變到其他任何狀態
      - 必定有不能再更動的值
    - 當處在rejected(已拒絕)時，一個promise:
      - 必定不會再轉變到其他任何狀態
      - 必定有不能再更動的值reason(理由)
## Promise 物件建立
  ```js
  const promise = new Promise(function(resolve, reject) {
    // 成功時
    resolve(value)
    // 失敗時
    reject(reason)
  });

  promise.then(function(value) {
    // on fulfillment(已實現時)
  }, function(reason) {
    // on rejection(已拒絕時)
  })
  ```

### promise 的建構函式
  ```js
  new Promise( function(resolve, reject) { ... } )
  ```
  這個函式中又有兩個傳入參數值，resolve(解決)與reject(拒絕)都是要求一定是`函式類型`。成功的話，也就是有合法值的情況下執行resolve(value)，promise物件的狀態會跑到fulfilled(已實現)固定住。失敗或是發生錯誤時用執行reject(reason)，reason(理由)通常是用Error物件，然後promise物件的狀態會跑到rejected(已拒絕)狀態固定住。

  因為它與一般的物件實體化的過程不太一樣，所以常會先包到一個函式中，使用時再呼叫這個函式來產生promise物件:
  ```js
  function asyncFunction(value) {
    return new Promise(function(resolve, reject){
        if(value)
            resolve(value) // 已實現，成功
        else
            reject(reason) // 有錯誤，已拒絕，失敗
    });
}
```
### then與catch
  - then 語法
    - `then` 方法用兩個函式當作傳入參數
  ```js
  p.then(onFulfilled, onRejected);

  p.then(function(value) {
    // fulfillment
    }, function(reason) {
    // rejection
  });
  ```
  onFulfilled是當promise物件的狀態轉變為fulfilled(已實現)呼叫的函式，有一個傳入參數值可用，就是value(值)。而onRejected是當promise物件的狀態轉變為rejected(已拒絕)呼叫的函式，也有一個傳入參數值可以用，就是reason(理由)。

  - `then` 方法必須回傳一個「新的」promise 物件。這樣設計的主要目的，是要能作連鎖(chained)的語法結構，也就是稱之為合成(composition)的一種運算方式，在JavaScript中如果回傳值相同的函式，可以使用連鎖的語法。
  ```js
  promise2 = promise1.then(onFulfilled, onRejected);
  ```
  
  - 連鎖語法如下：
  ```js
  const promise = new Promise(function(resolve, reject) {
    resolve(1)
  })
  promise.then(function(value) {
    console.log(value) // 1
    return value + 1
  }).then(function(value) {
    console.log(value) // 2
    return value + 2
  }).then(function(value) {
    console.log(value) // 4
  })
  ```
  - 回傳值往下傳遞
    經過連鎖的結構，如果要把值往下傳遞，可以用回傳值的方式，上面的例子可以看到用return語句來回傳值，這個值可以繼續的往下面的then方法傳送。`

  then方法中的兩個函式傳入參數，第1個onFulfilled函式是在promise物件`有值`的情況下才會執行，也就是進入到`fulfilled(已實現)`狀態。第2個`onRejected`函式則是在promise物件發生`錯誤、失敗`，才會執行。
  
  `這兩個函式都可以寫出來，但為了方便進行多個不同程式碼的連鎖，通常在只使用then方法時，都只寫第1個函式傳入參數。而錯誤處理通常交給另一個catch方法來作，catch只需要一個函式傳入參數`

  - catch 語法
  ```js
  p.catch(onRejected);

  p.catch(function(reason) {
    // rejection
  });
  ```
  catch方法相當於
  ```js
  then(undefined, onRejected)
  ```
  也就是then方法的第一個函式傳入參數沒有給定值的情況

  Note: 需要注意的是，catch方法中的回調(相當於then中的onRejected函式)是只能得到理由(錯誤)而得不到值的

### 執行流程與錯誤處理






