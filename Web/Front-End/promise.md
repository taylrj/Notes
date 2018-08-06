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

- Promise中會吞掉(隱藏)throw例外的錯誤輸出，改用轉變狀態為rejected(已拒絕)來作錯誤處理

- 因為Promise物件在進行初始化時，執行的每一個回調，都是在try...catch語句的區塊中執行，才會在就算有錯誤發生時，並不會造成連鎖的中斷，而是改變Promise的狀態為rejected(已拒絕)，然後再向下一個連鎖傳遞。

- 使用throw語句與用reject方法似乎是同樣的結果，都會導致promise物件的狀態變為rejected(已拒絕)，那麼這兩種方式有差異嗎？
  - throw語句用於一般的程式碼中，它代表的意義是程式執行到某個時候發生錯誤，也就是throw語句會立即完成resolve(解決)，在then方法中按照規則，不論是onFulfilled函式或onRejected函式，只要丟出例外，就會導致新的promise物件的狀態直接變為Rejected(已拒絕)。
  - reject則是一個一旦呼叫了就會讓Promise物件狀態變為Rejected(已拒絕)的方法，用起來像是一般的可呼叫的方法。

- 流程的主要規則在Promises/A+標準 2.2.7章節，內容如下:
  - then必須回傳一個promise。 
  ```js
  promise2 = promise1.then(onFulfilled, onRejected);
  ```
  - 2.2.7.1 當不論是onFulfilled或onRejected其中有一個是有回傳值x，執行Promise解析程序
  [[Resolve]](promise2, x)
  - 2.2.7.2 當不論是onFulfilled或onRejected其一丟出例外e，promise2必須用e作為理由而拒絕(rejected)
  - 2.2.7.3 當onFulfilled不是一個函式，而且promise1是fulfilled(已實現)時，promise2必須使用與promise1同樣的值被fulfilled(實現)
  - 2.2.7.4 當onRejected不是一個函式，而且promise1是rejected(已拒絕)時，promise2必須使用與promise1同樣的理由被rejected(拒絕)

- 承上，rejected狀態的Promise物件，會一直往下找到有onRejected函式的then方法或是catch方法，才會再進入2.2.7.1規則，也就是正常的Promise解析程序
- 怎麼決定回傳的新Promise物件的狀態 ?
  - 主要是由經過onFulfilled或onRejected其中一個被執行後的回傳值`x`決定，以下為可能的狀況：
    - `x`不是函式或物件，直接用x實現(fulfill)promise物件( 注意: undefined也算，所以沒回傳值算這個規則)
    - `x`是promise，看x最後是實現(fulfill)或拒絕狀態，就直接對應到promise物件，回傳值或理由也會一併跟著
    - `x`是函式或物件，指定then = x.then然後執行then，而如果不是`thenable`物件，則直接用x作實現(fulfill)promise物件。

### then方法
- then方法中onFulfilled函式的回傳值
  - 值: then方法最後會回傳Promise物件，所以這個回傳值會跟著這個新Promise物件到下一個連鎖方法去，這個回傳值可以用下個then方法的onFulfilled函式取到第1個傳入參數值。
  - promise物件
  - thenable物件
- then是promise物件中的方法，以onFulfilled與onRejected作為兩個傳入參數，有幾個規則需要遵守:
  - 當onFulfilled或onRejected不是函式時，忽略跳過
  - 當promise是fulfilled時，執行onFulfilled函式，並帶有promise的value作為onFulfilled函式的傳入參數值
  - 當promise是rejected時，執行onRejected函式，並帶有promise的reason作為onRejected函式的傳入參數值
- then方法最後還要回傳一個新的 promise 物件

### 靜態方法 Promise.resolve與Promise.reject
- Promise.resolve 相當於直接回傳一個以then方法實現的新Promise物件，帶有傳入的參數值
  - Promise.resolve(value);
  - Promise.resolve(promise);
  - Promise.resolve(thenable);

- 直接用Promise.resolve代表這個新的Promise物件的狀態是直接設定為fulfilled(已實現)狀態，這方法只是方便產生根(第一個)Promise物件使用的，也就是除了用建構式的new Promise()外，提供另一種方式來產生新的Promise物件
  - onRejected function 不會被 call
- Promise.reject剛好與Promise.resolve相反，等於是要產生rejected(已拒絕)狀態的新Promise物件
  - onFullfilled function 不會被 call

- Promise.reject與Promise.resolve與使用Promise建構式的方式，在使用上仍然有大的不同，executor傳入參數可以讓程式設師自訂Promise產生的過程，而且在產生的過程中是throw safety(安全)的

Note: 結論是Promise.reject或Promise.resolve只用於單純的傳入物件、值或外部的thenable物件，轉換為Promise物件的場合。如果你要把一整段程式碼語句或函式轉為Promise物件，不要用這兩個靜態方法，要使用Promise建構式來產生物件才是正解。



