#第一章 AngularJS簡介

我們創造驚人的基於Web的應用程式的能力是不可思議的, 但是建立這些應用程式時所涉及的複雜性也是讓人難以置信的. 我們的Angular團隊希望減輕我們在參與開發AJAX應用程式時的痛苦. 在Google, 我們曾經在構建像Gmail, Maps, Calendar以及其他幾個大型Web應用程式時經歷了最痛苦的教訓. 我想我們也許能夠利用這些經驗讓更多的人受益.

我們希望在編寫Web應用程式時感覺更像是第一次我們編寫了幾行代碼並站在後面驚訝於它發生了什麼. 我們希望編碼的過程感覺更像是創造而不是試圖滿足Web瀏覽器其奇怪的內部運作工作.

與此同時, 我們還希望環境能夠幫助我們作出設計選擇, 使應用程式很容易建立並從一開始就很容易理解, 並且隨著它們的不斷成長, 正確的選擇會讓我們的應用程式易於測試, 擴充和維護.

我們視圖在Angular中做到這些. 已經取得的結果讓我們非常興奮. 這很大程度上歸功於Angular開源社區中的成員的出色工作和相互幫助, 同時也讓我們學習到更多的東西. 我們希望你也加入到我們的社區中來, 並幫助我們努力讓Angular變得更好.

你可以在我們的[Github主頁](https://github.com/shyamseshadri/angularjs-book)的倉庫中查看那些較大的或者較複雜的例子和代碼片段, 你也可以拉取分支以及研究這些代碼.

##概念

在你將使用的Angular應用中有幾個核心的概念. 事實上, 任何這些概念並不是我們發明的. 相反, 我們從其他開發環境借鑒了大量成功的做法, 並使用包含HTML, 瀏覽器和許多其他Web標準的方式實現了它.

###客戶端樣板

多頁面的Web應用程式都是經由組裝和連接伺服器上資料來建立HTML, 然後將完成的頁面發送到瀏覽器中. 大多數的單頁應用程式 - 也稱為AJAX應用程式 - 這一點做的很好, 在某種程度上. Angular以不同的方式組裝樣板和資料並推送到瀏覽器中運行.  然後伺服器將變成給樣板提供靜態資源和給這些樣板提供所需的正確資料的角色.

讓我們來看一個例子, 在Angular中如何在瀏覽器中組裝樣板和資料. 我們照慣例使用一個Hello, World的例子, 但是並不是編寫一個"Hello, World"的單一字符串, 而是構造這個問候"Hello"作為稍後我們可能會改變的資料.

針對它, 我們建立了一個`hello.html`樣板:
```html
    <html ng-app>
    <head>
        <script src="angular.js"></script>
        <script src="controller.js"></script>
    </head>
    
    <body>
        <div ng-controller="HelloController">
            <p>{{greeting.text}}, World</p>
        </div>
    </body>
    </html>    
```
然後我們在`controller.js`中編寫邏輯:
```js
    function HelloController($scope){
        $scope.greeting = {text: 'Hello'};
    }
```
將`hello.html`載入任意瀏覽器中, 我們將看到如圖1-1所示:

![Hello](figure/hello.png)

圖1-1 Hello World

於現在使用廣泛的大多數方法相比, 這裡有一些有趣的事情需要注意:

+ HTML並沒有類或者ID以確定在哪裡添加事件監聽器.
+ 當`HelloController`設定`greeting.text`為`Hello`時, 我們並沒有註冊任何事件監聽器或者編寫任何回呼函數.
+ `HelloController`只是一個很普通的JavaScript類, 它並沒有繼承任何Angular提供的東西.
+ `HelloController`接收所需的`$scope`對像, 而無需建立它.
+ 我們並沒有呼叫HelloController自身的構造器, 或者在呼叫它時來計算它.

接下來我們會看到更多的差異, 但是我們應該清楚, Angular應用程式的結構與過去類似的應用程式完全不同.

為什麼我們作出了這些設計選擇以及Angular是如何工作的? 讓我們來看看Angular從其他地方借鑒的一個好的概念.

###模型/視圖/控制器(MVC)

MVC應用程式結構是20世紀70年代被作為Smalltalk的一部分引進的. 從Smalltalk開始, MVC在幾乎每一個涉及用戶界面的桌面開發環境中都變得受歡迎. 無論你是使用C++, Java還是Object-C, 都有可以利用MVC的氣息. 然而, 直到最近, MVC才開始應用於Web開發中. *[MVC was all but foreign to web development. 大意: 幾乎與Web開發都不相關]*

MVC背後的核心思想是你可以在你的代碼管理中清晰的分離資料(模型), 應用程式邏輯(控制器)以及給用戶呈現資料(視圖).

視圖從模型中取得資料並顯示給用戶. 當用戶經由點擊或者輸入與應用程式交互時, 控制器經由改變模型中的資料來回應. 最後, 該模型會通知視圖它已經發生改變. 以便它可以更新它顯示的訊息. 

在Angular應用程式中, 視圖就是文件對像模型(DOM), 控制器是JavaScript類, 模型資料存儲在對象的屬性中.

我們認為MVC的靈活主要是以下幾個原因. 首先, 它給你在哪裡擺放什麼提供了一個智能模型, 因此你不需要每次都創造它. 其他人參與到你的項目中合作時, 將能夠即時理解你已經編寫好的部分, 因為他們會知道你使用了MVC結構來組織你的代碼. 也許最重要的是, 它給你的應用程式更易於擴充, 維護和測試提供了極大的好處.

**譯注, 參考閱讀:** 

1. MVC是軟件工程中的一種軟件架構模式 - [MVC](http://zh.wikipedia.org/wiki/MVC)
2. Smalltalk是一門面向對象的程序設計語言 - [Smalltalk](http://zh.wikipedia.org/wiki/Smalltalk)

###資料繫結

之前常見的AJAX單頁應用程式, 像Rails, PHP或者JSP平台都是在經由在發送給用戶顯示之前將資料合併到HTML字符串中來幫助我們建立用戶界面(UI).

像jQuery這樣的函式庫也是將模型擴充到客戶端並讓我們使用類似的風格, 但是它只有單獨更新部分DOM的能力, 而不能更新整個頁面. 這裡, 我們將資料合併到字符串形式的HTML樣板中, 然後經由在一個佔位元素中設定`innerHTML`將返回的結果插入到我們所希望的DOM元素中.

這一切都工作得很好, 但是當你希望插入新的資料到用戶界面(UI)中時, 或者基於用戶的輸入改變資料, 你需要做一些相當不平凡的工作以確保你的資料狀態的準確性, 無論是在用戶界面中(UI)還是JavaScript屬性中.

但是如果我們可以不用編寫代碼就能處理好這一些的工作會怎樣? 如果我們可以只需聲明用戶界面的哪些部分對應JavaScript的哪些屬性並讓它們自動同步又會怎樣? 這種編程風格被稱為資料繫結. 由於它是MVC的一項偉大的工程, 便於你編寫視圖和模型時減少代碼, 我們把它引入到了Angular中. 大部分將資料從一處遷移到另一處的工作都會自動完成.

來看看這一行為, 讓我們使用第一個例子並讓它動起來. 原來是, 一旦HelloController設定了其模型`greeting.text`, 它便不再改變. 讓我們經由添加一個根據用戶輸入改變`greeting.text`值的文字輸入框來改變這個例子, 讓它能夠活動.

這裡是新的樣板:
```html
    <html ng-app>
    <head>
        <script src="angular.js"></script>
        <script src="controllers.js"></script>
    </head>
    
    <body>
        <div ng-controller="HelloController">
            <input ng-model="greeting.text" />
            <p>{{greeting.text}}, World</p>
        </div>
    </body>
    </html>
```
`HelloController`控制器可以保持不變.

將它載入到瀏覽器中, 我們將看到如圖1-2所示螢幕截圖:

![Hello with data binding](figure/hello2.png)

圖1-2 應用程式的默認狀態

如果我們使用'Hi'文字替換輸入框中的'Hello', 我們將看到如圖1-3所示截圖:

![Hi](figure/hello3.png)

圖1-3 改變文字框值之後的應用程式

我們並沒有在輸入字段上註冊改變值的監聽器, 我們有一個將會自動更新的UI.
同樣的情況也適用於來自伺服器端的改變. 在我們的控制器中, 我們可以對伺服器發出一個請求, 獲得回應, 然後設定`$scope.greeting.text`等於它返回的值. Angular會自動更文字輸入框和雙大括號中的text為該值.

###依賴注入

之前我們提到過, 在`HelloController`中有很多東西都可以重複, 這裡我們並沒有編寫. 例如, `$scope`對像將自動綁定資料並傳遞給我們; 我們不需要經由呼叫任何函數來建立它. 我們只是經由將它放置在HelloController的構造器中來訪問它.

正如我們將在後面的章節中會看到, `$scope`並不是我們唯一可以訪問的事物. 如果我們希望將資料繫結到用戶瀏覽器的URL地址中, 我們可以經由將用於管理它的對象`$location`放在我們的構造器中來訪問它, 就像這樣:
```js
    function HelloController($scope, $location){
        $scope.greeting = {text: 'Hello'};
        //use $location for something good here...
    }
```
我們經由Angular的依賴注入系統達到這個神奇的效果. 依賴注入讓我們得以延續這種開發的風格, 而不需要建立依賴, 我們的類只需要知道它需要什麼.

在此之前, 它是一個被稱為得墨忒耳定律的設計模式, 通常也被稱作最少知識原則. 由於我們的HelloController的工作只是設定greeting模型的初試狀態, 這種模式會告訴你無需擔心其他的事情, 例如`$scope`是如何建立的, 或者在哪裡可以找到它.

這個特性並不只是經由Angular框架建立的對象才有. 你也可以更好的按照這個風格編寫其他的代碼.

###指令

Angular最好的部分之一就是你可以如同編寫HTML一樣編寫你的樣板. 之所以可以這樣做, 是由於這個框架的核心部分我們已經包含了一個的DOM解析引擎, 它允許你擴充HTML的語法.

我們已經在我們的樣板中看到了一些不屬於HTML規範的屬性. 例如包含在大括號中的資料繫結, 用於指定哪個控制器對應哪部分視圖的`ng-controller`, 以及給輸入框綁定一個模型的`ng-model`. 這些我們都成為HTML擴充指令.

Angular帶有許多指定, 以幫助你定義應用程式的視圖. 很快我們就會看到更多的指令. 這些指令可以定義來我們通常用作視圖的樣板中. 它們可以用於聲明設定你的應用程式如何工作或者建立可復用的組件.

你並不僅限於使用Angular自帶的指令. 你也可以編寫你自己的HTML樣板擴充, 做你想做的事情. 

##範例:購物車

讓我們來看一個較大的例子, 它展示了更多的Angular的能力. 想像一下, 我們要建立一個購物應用程式. 在應用程式的某個地方, 我們需要展示用戶的購物車並允許他編輯. 接下來我們直接跳到那部分.
```html
    <html ng-app="myApp">
    <head>
    <title>Your Shopping Cart</title>
    </head>
    <body ng-controller="CartController">
        <h1>Your Order</h1>
        <div ng-repeat="item in items">
            <span{{item.title}}</span>
            <input ng-model="item.quantity" />
            <span>{{item.price | currency}}</span>
            <span>{{item.price * item.quantity | currency}}</span>
            <button ng-click="remove($index)">Remove</button>
        </div>
        <script src="angular.js"></script>
        <script>
        function CartController($scope){
            $scope.items = [
                {title: 'Paint pots', quantity: 8, price: 3.95},
                {title: 'Polka dots', quantity: 17, price: 12.95},
                {title: 'Pebbles', quantity: 5, price: 6.95}
            ];
            
            $scope.remove = function(index){
                $scope.items.splice(index, 1);
            }
        }
        </script>
    </body>
    </html>
```
返回的用戶界面截屏如圖1-4所示:

![shopping-cart](figure/shopping-cart.png)

圖1-4 購物車用戶界面

下面是關於這個範例的簡短參考. 本書其餘的部分提供了更深入的講解.

讓我們從頂部開始:
```html
    <html ng-app>
```
`ng-app`屬性告訴Angular應該管理頁面的哪一部分. 由於我們把它放在`<html>`元素中, 這將告訴Angular我們希望它管理整個頁面. 這往往也是你所希望的, 但是如果你是結合現有的Angular應用程式來使用其他方式管理頁面, 你可能希望把它放在應用程式中的某個`<div>`中.
```html
    <body ng-controller="CartController">
```
在Angular中, 你用於管理頁面區域的JavaScript類被稱為控制器. 經由在body標籤中包含一個控制器, 我們聲明這個`CartController`將管理`<body>`和`</body>`之間的所有事物.
```html
    <div ng-repeat="item in items">
```
`ng-repeat`的意思就是對於被稱為`$items`的數組的每一個元素都複製一次`<div>`中的DOM. 在每一個複製的div副本中, 我們都會給當前元素設定一個名為`item`的屬性, 因此我們可以在樣板中使用它. 正如你所看到的, 這將導致這三個`<div>`的每一個都包含產品的標題, 數量, 單價, 總價以及一個用於移除整個項的按鈕.
```html
    <span>{{item.title}}</span>
```
正如我們在"Hello, World"例子中所示,  資料繫結經由`{{ }}`讓我們將變數的值插入到頁面部分並保持同步. 完整的表達式`{{item.title}}`會迭代檢索當前item, 然後將由item的標題屬性組成內容插入到DOM中.
```html
    <input ng-model="item.quantity">
```
文字輸入字段和`item.quantity`的值之間的`ng-model`定義並建立資料繫結.

`<span>`中的`{{ }}`用於設定一個單向的聯繫, 意思就是"在這裡插入一個值". 這是我們希望的效果, 但是當用戶改變單價時應用程式也需要知道, 這樣它就能夠改變總價.

我們經由使用`ng-model`來同步模型中的變化. `ng-model`聲明將`item.quantity`的值插入到文字域中, 每當用戶輸入一個新的值時它也能夠自動更新`item.quantity`的值.
```html
    <span>{{item.price | currency}}</span>
    <span>{{item.price * item.quantity || currency}}</span>
```
我們希望單價和總價被格式化為美元形式. Angular自帶了一個稱為過濾器的特性讓我們可以轉換文字, 這裡綁定了一個稱為`currency`的過濾器用於給我們處理這裡的美元格式. 在下一章我們將會看到更多的過濾器.
```html
    <button ng-click="remove($index)">Remove</button>
```
這允許用戶經由點擊產品旁邊的Remove按鈕來從他們的購物車中移除項目. 我們設定它點擊這個按鈕時呼叫`remove()`函數. 我們還給它傳遞了一個`$index`, 它包含了`ng-repeat`索引值, 因此我們可以知道哪一項將會被移除.
```js
    function CartController($scope)
```
這個`CartContoller`用於管理購物車的邏輯. 這會告訴Angular, 控制器需要在這裡給他設定一個稱為`$scope`的東西. 這個$scope用於讓我們在用戶界面中將資料繫結到元素中.
```js
    $scope.items = [
        {title: 'Paint pots', quantity: 8, price: 3.95},
        {title: 'Polka dots', quantity: 17, price: 12.95},
        {title: 'Pebbles', quantity: 5, price: 6.95}
    ];
```
經由定義`$scope.items`, 我們建立了一個虛擬的資料哈希表來描述用戶購物車中的物品集. 我們希望它可以用於UI中的資料繫結, 因此將他添加到$scope中.

當然, 一個真實版本的購物車不可能只在內存中工作, 它需要訪問伺服器中正確存儲的資料. 我們將在後面的章節中討論這些.
```js
    $scope.remove = function(index){
        $scope.items.splice(index, 1);
    }
```
我們還希望`remove()`函數能夠用於用戶界面中的資料繫結, 因此我們同樣將它添加到$scope中. 對於這個工作在內存中的版本的購物車, `remove()`函數可以即時從數組中刪除項目. 由於`<div>`列表是經由`ng-repeat`建立的資料繫結, 所以當項目消失時列表將自動收縮. 記住, 每當用戶在UI上點擊一個Remove按鈕時, 這個`remove()`函數就會被呼叫.

##小結

我們已經看到了Angular最基本的用法以及一些非常簡單的例子. 本書後面的部分將專注於介紹這個框架提供的功能.
