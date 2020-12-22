# 所以 API 是什麼東西?
+ 開發者 (Developer) 和一個系統溝通的一種渠道
+ 例如外賣平台 (E.g. Foodpanda) 可能會用到地圖平台 (E.g. Google maps) 取得所需資訊來輔助開發，此時他們就會利用地圖平台的 API 來取得一家店的導航指示。
+ 又或者新聞節目下方的滾動條常常會出現一些股票資料，這些資料就是開發者利用 API 接觸股票資訊公司的數據庫。
+ 這是最直接有效的方法來獲得資料，不然想像用 [Selenium](https://www.selenium.dev/) 這樣的瀏覽器自動化工具慢慢開，應該會直接吐死
+ 與此同時，這些服務也可以透過寫成 API 的方式供別人使用，以達到擴大受眾或者是增加營收來源的目的，下面的例子就是如此。
<hr>

# 例子: Google Maps
## 背景
因為疫情關係，假設我想製作一張感染地圖。我希望可以獲得某一座確診者曾經前往的建築物的經緯度（例如太平山頂）用作地區分析之用，那麼我就可以用 Google Map 的 [Geocoding API](https://developers.google.com/maps/documentation/geocoding/overview?hl=zh-tw) 來輔助我完成這項工作。
## 前置工作
1. 申請 Google 帳號, 以及開通 Google Cloud Platform
   + 由於申請有 90 天免費，上限 300 美元的使用額度，以學習 API 的基礎程度不太機會扣到錢
   + 以 Google Geocoding API 為例，前十萬條查詢，每一千條查詢付五美金
2. 在 Google Cloud Platform 中的 Marketplace 啟用 geocoding API
3. 在「API 和服務」欄位找到 Maps API Key，這個是告訴 Google 是誰在索取資訊，旁邊亦會有 API Key

## 透過 API 取得資料 
在瀏覽器網址欄輸入 https://maps.googleapis.com/maps/api/geocode/json?address=1600+Amphitheatre+Parkway,+Mountain+View,+CA&key=[你的 API Key]，就可以獲取地址的分析，以及經緯度資訊。

下面拆解分析 URL 的意思
   + ```https://maps.googleapis.com/``` - 通訊協定 DNS 網域
   + ```/maps/api/geocode/json``` - API interface 的路徑
   + ```?address=1600+Amphitheatre+Parkway,+Mountain+View,+CA&key=[你的 API Key]``` - 第一類參數 (地址找經緯度)
     + 參數1: Address, 用 ```'+'``` 或者 ```'%20'``` 來取代空格
     + 參數2: key
     + 其他參數可以參考 [Google Maps Platform](https://developers.google.com/maps/documentation/geocoding/overview#ReverseGeocoding)
   + ```?latlng=40.714224,-73.961452&key=[你的 API Key]``` - 第二類參數 (經緯度找地址)
     + 參數1: latlng, 經緯度
     + 參數2: key
     + 其他參數可以參考 [Google Maps Platform](https://developers.google.com/maps/documentation/geocoding/overview#geocoding-lookup)
## 使用 JavaScript Fetch 來完成 API 的資料取得
在網頁開發的環境下，JavaScript 的使用可以說是必需的，而 [Fetch](https://developer.mozilla.org/zh-TW/docs/Web/API/Fetch_API/Using_Fetch) 就是可以其中一個較為便利地完成 API 資料取得的方法。而 Fetch 的語法如下:
```javascript
fetch(URL)
```
fetch 會傳回包含 Response 的 Promise
+ Promise - Constructor function, 只有兩種結果
  + 若成功完成，則用 ```.then()``` 來完成接下來的工作
  + 若沒有完成，則用 ```.catch()``` 來完成接下來的工作
+ Response - Object, 擁有以下會用的 Properties 和 Methods
  + ```.ok``` - 若結果為 ```true```，配合 promise 為成功，則 fetch 成功
  + ```.json()``` - 以 json 格式返還結果的 Promise
  + ```.txt()``` - 以 txt 格式返還結果的 Promise

那麼我們來查查香港太平山頂 (Victoria Peak)，而最基本的代碼就是
```javascript
    fetch('https://maps.googleapis.com/maps/api/geocode/json?address=Victoria+Peak&key=[Your Key]')
    //fetch() 傳出 promise 之後，送出 json()
    .then(response => response.json())  
    //json() 傳出 promise 之後，可以獲得所需資料，即經緯度
    .then(json => console.log(json.results[0].geometry.location.lat + ", " + json.results[0].geometry.location.lng)) 
```
最後產出的結果是
```
22.2758835, 114.145532
```

# HTTP 請求方法
剛才講了 Fetch 可以用作資料取得，但其實 Fetch 是一種發送 HTTP 請求的方法。那麼 HTTP 請求又是什麼呢？

HTTP 作為傳送超媒體文件的協定，是一個瀏覽器和伺服器溝通的一種規則。在請求伺服器取得資料的方法，或者**請求方法**，是一個必須理解的概念。

下文參考 [Progess Bar](https://progressbar.tw/posts/53)
1. GET - 取得所需的資料，或者狀態 (如上例)，這並不會更改原本的資料狀態
2. [POST](https://developer.mozilla.org/zh-CN/docs/Web/HTTP/Methods/POST) - 新增資料 (重複會改變資料狀態)
3. [PUT](https://developer.mozilla.org/zh-CN/docs/Web/HTTP/Methods/PUT) - 新增資料或者修改資料 (重複不會改變資料狀態)
4. PATCH - 修改資料
5. DELETE - 刪除資料

# REST API
回到剛才 Google Map Geocoding API 的 URL
```https://maps.googleapis.com/maps/api/geocode/json?address=Victoria+Peak&key=AIzaSyDQ22zQPHdaM6Z3fpjmYYiL9toYfM8lebM```時，有提到前往 API 的路徑，也就是說傳統的 API 設計是需要為每一種服務寫出一個新的 interface 路徑。

REST API 就是利用了上面提到的 HTTP 請求方法來輕易解決 API 路徑問題，以達到簡潔化。
