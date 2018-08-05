# Progressive Web App

- Progressive Web App 的目標是想要能夠 讓 Web application 盡可能的在各種環境（網路環境、手機作業系統等）下都能順暢且不減功能性的運作。
- Progressive web app 的重點其實在於如何將元件與資料做 cache ，以及 app-like 的顯示模式
- PWA 建構一個 model，將不經常變動的內容(App Shell)和變動的內容(Content)區分開來，可以把 App Shell 下載後、將內容透過 service work cache 至本地端資料庫，之後的 loading 就能夠使用 cache 的內容、立即獲取頁面

- App Shell Architecture
  - 將 應用程式的基礎設施（infrastructure）、UI 與資料做分離，並利用 service worker 將 infra 與 UI 做 Cache
  - 因此，打開 app 以後需要載入的只有 data
  - 一打開app就要看到的東西設計進 App shell architecture 的 component
  - A service worker is a script that your browser runs in the background, separate from a web page, opening the door to features that don't need a web page or user interaction.
    - 當處理完第一次資料 fetch 後，要能夠 cache 起來，才能應付 slow connection 或是 offline 的狀況。也就是service-worker 要負責的事情

- sw-precache: A node module to generate service worker code that will precache specific resources so they work offline.
  - sw-precache 是用來解決什麼問題 ？
    解決許多 edge cases 如下:
    - 剛剛提過的，當你的 content 有變動時，cache key 要更新（範例內我們需要手動更新）。
    - 只要你的檔案有一點點更動，小至 typo，大至 code refactor，都會造成 cache invalidated，需要重新下載，效率不好。
    - 必須確保 service-worker 中的 install handler 所發出的 https request 不會被 Brower cache 影響，否則會無法 update。
    - 最重要的一點，範例採用 cache-first 的策略，任何 request 只要有 cache 在就會先拿 cache，若 service worker 相關的註冊與設定也被 cache 時，更新會變得很困難。

- The service worker life cycle:
  - register (註冊)
    - 在app中要先偵測是否有 service worker 存在，若沒有，則透過`navigator.serviceWorker` 註冊
    ```js
    if('serviceWorker' in navigator) {  
      navigator.serviceWorker  
        .register('/service-worker.js')  
        .then(function() { console.log('Service Worker Registered'); });  
    }
    ``` 
  - install
    - 在註冊完以後，初次開啟時，會觸發一個install event，在`event handler`之中去指定要cache的資源
    ```js
    self.addEventListener('install', function(e) {
      console.log('[ServiceWorker] Install');
      e.waitUntil(
        caches.open(cacheName).then(function(cache) {
          console.log('[ServiceWorker] Caching App Shell');
          return cache.addAll(filesToCache); //filesToCache can be URL lists
        })
      );
    });
    ```
  - activate
    - A service worker won't receive events like fetch and push until it successfully finishes installing and becomes "active" 
    - clients.claim: Can take control of uncontrolled clients by calling `clients.claim()` within service worker once it's activated.
      - clients: We call pages, workers, and shared workers clients
      - When a service worker is initially registered, pages(or say, clients) won't use it until they next load. The `claim()` method causes those pages to be controlled immediately. Be aware that this results in your service worker controlling pages that loaded regularly over the network, or possible via a different service worker.
  - fetch

