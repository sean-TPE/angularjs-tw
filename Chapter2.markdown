#第二章 Angular應用程式剖析

不像典型的函式庫, 你需要挑選你喜歡的功能, 在Angular中所有的東西都被設計成一個用於協作的套件. 在本章中我們將涵蓋Angular中所有的基本構建塊, 這樣你就可以理解如何將它們組合在一起. 這些塊都將在後面的章節中有更詳細的討論.

##啟用Angular

任何應用程式都必須做兩件事來啟用Angular:

1. 加載`angular.js`庫
2. 使用`ng-app`指令來告訴Angular它應該管理哪部分DOM

###加載腳本

加載庫很簡單, 與加載其他任何JavaScript庫遵循同樣的規則. 你可以從Google的內容分發網絡(CDN)中載入腳本, 就像這樣:
```html
    <script src="http://ajax.google.com/ajax/libs/angularjs/1.0.4/angular.min.js"></script>
```
推薦使用Google的CDN. Google的伺服器很快, 並且這個腳本是跨應用程式快取的. 這意味著, 如果你的用戶有多個應用程式使用Angular, 那麼他將只需要下載腳本一次. 此外, 如果用戶訪問過其他使用Google CDN連接Angular的站點, 那麼他在訪問你的站點時就不需要再次下載該腳本.

如果你更喜歡本地主機(或者其他的方式), 你也可以這樣做. 只需要在`src`中指定正確的地址.

###使用ng-app聲明Angular的界限

> 原文是Boundaries, 意思是聲明應用程式的作用域, 即Angular應用程式的作用範圍.

`ng-app`指令用於讓你告訴Angular你期望它管理頁面的哪部分. 如果你在建立一個完全的Angular應用程式, 那麼你應該在`<html>`標籤中包含`ng-app`部分, 就像這樣:
```html
    <html ng-app>
    …
    </html>
```
這會告知Angular要管理頁面中的所有DOM元素. 

如果你有一個現有的應用程式, 要求使用其他的技術來管理DOM, 例如Java或者Rails, 你可以經由將它放置在頁面的一些元素例如`<div>`中來告訴Angular只需要管理頁面的一部分即可.
```html
    <html>
    …
        <div ng-app>
        …
        </div>
    …
    </html>
```
###模型/視圖/控制器

在第一章中, 我們提到Angular支持模型/視圖/控制器的應用程式設計風格. 雖然在設計你的Angular應用程式時有很大的靈活性, 但是總是別有一番風味的:

+ 模型包含代表你的應用程式當前狀態的資料
+ 視圖顯示資料
+ 控制器管理你的模型和視圖之間的關係

你需要使用對像屬性的方式建立模型, 或者只包含原始類型的資料. 這裡並沒有特定的模型變數. 如果你希望給用戶顯示一些文字, 你可以使用一個字符串, 就像這樣:
```js
    var someText = 'You have started your journey';
```
你可以經由編寫一個樣板作為HTML頁面, 並從模型中合併資料的方式來建立視圖. 正如我們已經看過的, 你可以在DOM中插入一個佔位符, 然後再像這樣設定它的文字:
```html
    <p>{{someText}}</p>
```
我們呼叫這個雙大括號語法來插入值, 它將插入新的內容到一個現有的模版中.

控制器就是你編寫用於告訴Angular哪些對像和原始值構成你的模型的類, 經由將這些對像或者原始值分配給`$scope`對像傳遞到控制器中.
```js
    function TextController($scope){
        $scope.someText = someText;
    }
```
把他們放在一起, 我們得到如下代碼:
```html
    <html ng-app>
    <body ng-controller="TextController">
        <p>{{someText}}</p>
        
        <script src="https://ajax.googleapis.com/ajax/libs/angularjs/1.0.1/angular.min.js"></script>
        
        <script>
            function TextController($scope){
                $scope.someText = 'You have started your journey';
            }
        </script>
    </body>
    </html>
```
將它載入到瀏覽器中, 你就會看到

> 'You have started you journey'

雖然這個原始風格的模型工作在簡單的情況下, 然而大多數的應用程式你都希望建立一個模型對像來包裹你的資料. 我們將建立一個訊息模型對像, 並用它來存儲我們的`someText`. 因此不是這樣的:
```js
    var someText = 'You have started your journey';
```
你應該這樣編寫:
```js
    var messages = {};
    messages.someText = 'You have started your journey';
    function TextController($scope){
        $scope.messages = messages;
    }
```
然後在你的樣板中這樣使用:
```html
    <p>{{messages.someText}}</p>
```
正如我們後面會看到, 當我們討論`$scope`對像時, 像這樣建立一個模型對像將有利於防止從`$scope`對象的原型中繼承的意外行為.

我們正在討論的這些方法從長遠看來能夠幫助你, 在上面的例子中, 我們在全域作用域中建立了`TextController`. 雖然這是一個很好的例子, 但是正確定義一個控制器的做法應該是將它作為模組的一部分, 它給你的應用程式部分提供了一個命名空間. 更新之後的代碼看起來應該是下面這樣.
```html
    <html = ng-app="myApp">
    <body ng-controller="TextController">
        <p>{{someText.message}}</p>
        
        <script src="https://ajax.googleapis.com/ajax/libs/angularjs/1.0.1/angular.min.js"></script>
        
        <script>
            var myAppModule = angular.module('myApp',[]);
            
            myAppModule.controller('TextController', function($scope){
                var someText = {};
                someText.message = 'You have started your journey';
                $scope.someText = someText;
            });
        </script>
    </body>
    </html>
```
在這個版本中, 我們聲明模組中`ng-app`元素的名稱為`myApp`. 然後我們呼叫Angular對像建立了一個名為myApp的模組, 然後呼叫模組的`controller`方法並將我們的控制器函數傳遞給它.

一會兒我們就會知道為什麼, 以及如何取得所有的模組. 但是現在, 只需要記住將所有的訊息都保存在全域的命名空間中是一件好事, 並且這也是我們使用模組的機制.

##樣板和資料繫結

在Angular應用程式中樣板只是HTML文件, 就像我們從從服務端載入或者定義在`<script>`標籤中的任何其他靜態資源一樣. 在你的樣板中定義用戶界面, 可以使用標準的HTML加Angular指令來定義你所需要的UI組件.

一旦進入瀏覽器中, Angular就會進入到你的整個應用程式中經由合併樣板和資料的方式來解析這些樣板. 在第一章中我們已經在購物車應用中看過了顯示一個項目列表的例子.
```html
    <div ng-repeat="item in items">
        <span>{{item.title}}</span>
        ...
    </div>
```
這裡, 它只是外層`<div>`的一個副本, 裡面所有的一切, 都一一對應`items`數組中的每個元素.

那麼這些資料從哪裡來? 在我們的購物車例子中, 在我們的代碼中我們只將它定義為一個數組. 對於你開始建立一個UI並希望測試它是如何工作的, 這是非常合適的. 然而大多數的應用程式, 將使用一些伺服器上的持久性資料. 在瀏覽器中你的應用程式連接你的伺服器, 用戶在頁面上請求他們所需要的一切, 然後Angular將它[請求的資料]與你的樣板合併.

基本的運作流程看起來像這樣:

1. 用戶請求你的應用程式的第一個頁面
2. 用戶瀏覽器發出一個HTTP請求連接到你的伺服器, 然後加載包含樣板的*index.html*頁面
3. Angular載入到頁面中, 等到頁面完全加載, 然後查詢定義在樣板範圍內的`ng-app`
4. Angular遍歷樣板並查詢指令和綁定. 這將導致註冊事件監聽器和DOM操作, 以及從伺服器上取得初始資料. 這項工作的最終結果是展示應用程式並將樣板作為DOM轉換為視圖.
5. 連接到你的伺服器加載你需要展示給用戶所需的附加資料.

第1步至第3步是每個Angular應用程式的標準. 第4步和第5步對你來說是可選的. 這些步驟可以同步或者非同步發生. 出於性能的考慮, 你應用程式所需的資料在第一個視圖中[首屏]顯示給用戶, 可以減少並避免重複的請求HTML樣板.

經由使用Angular組織你的應用程式, 你可以在你的應用程式中分離樣板和資料. 這樣做的結果是這些樣板是可以快取的. 在第一次載入之後, 實質上瀏覽器中就只需要請求新的資料了. 正如JavaScript, 圖片, CSS以及其他資源, 快取這些樣板可以給你的應用程式提供更好的性能.

###顯示文字

你可以使用`ng-bind`指令在你UI的任何地方顯示和更新文字. 它有兩種等價的形式. 一種是我們見過的大括號形式:
```html
    <p>{{greeting}}</p>
```
然後就是一個被稱為`ng-bind`的基於屬性的指令:
```html
    <p ng-bind="greeting"><p>
```
這兩者的輸出是等價的. 如果模型中的變數`greeting`設定為"Hi, there", Angular將產生這樣的HTML:
```html
    <p>Hi, there</p>
```
瀏覽器將顯示"Hi, There".

那麼為什麼你會使用上面的另外一種形式? 我們建立的雙括號插入值的語法讀起來更加自然並且只需要更少的輸入. 雖然兩種形式產生相同的輸出, 但使用大括號語法, 加載你應用程式的第一個頁面`index.html`時, 在Angular替換花括號中的資料之前, 用戶可能會看到一個未渲染的樣板. 隨後的視圖將不會經歷這一點.

原因是瀏覽器加載HTML頁面, 渲染它, 直到那時Angular才可能準備解析它們.

好消息是你仍然可以在大多數樣板中使用`{{ }}`. 然而, 在你的`index.html`頁面中綁定資料, 應該使用`ng-bind`. 這樣, 直到資料加載完你的用戶將什麼也看不到.

###表單輸入

在Angular中處理表單元素是很簡單的. 正如我們見過的幾個例子, 你可以使用`ng-model`屬性綁定到你的模型屬性元素上. 這適用於所有標準的表單元素, 例如文字輸入框, 單選按你, 復選框等等.  我們可以像這樣綁定一個復選框到一個屬性:
```html
    <form controller="SomeController">
        <input type="checkbox" ng-model="youCheckedIt">
    </form>
```
這意味著:

1. 當用戶選擇復選框, `SomeController`的`$scope`中一個名為`youCheckedIt`的屬性將變成true. 取消選擇時使`youCheckedIt`變成false.
2. 如果你在`SomeController`中設定`$scope.youCheckedIt`為true, 這個復選框在UI中會被自動選擇. 設定它為false則取消選擇.

現在我想說的是我們真正想要的是, 當用戶做了一些什麼事情時作出回應. 對於文字輸入框元素, 你使用`ng-change`屬性來指定一個控制器方法, 那麼無論什麼時候用戶改變輸入框的值時, 這個控制器方法都應該被呼叫. 讓我們做一個簡單的計算器來幫助用戶自己理解他們需要多少錢才能得到某些東西:
```html
    <form ng-controller="StartUpController">
        Starting: <input ng-change="computeNeeded()" ng-model="funding.startingEstimate">
        Recommendation: {{funding.needed}}
    </form>
```
對於我們這個簡單的例子, 讓我們只設定輸出用戶預算十倍的值. 我們還將設定一個默認為0的值來開始:
```js
    function StartUpController($scope){
    
        $scope.funding = { startingEstimate: 0 };
        
        $scope.computeNeeded = function(){
            $scope.needed = $scope.startingEstimate * 10;
        };
        
    }
```
然而, 前面的代碼中有一個潛在的策略問題. 問題是當用於在文字輸入框中輸入時我們只是重新計算了所需的金額. 如果這個輸入框只在用戶在這個特定的輸入框中輸入時更新, 這工作得很好. 但是如果其他的輸入框也在模型中綁定了這個屬性會怎樣呢? 如果它從伺服器取得資料來更新又會怎樣?

無論這個字段如何更新, 我們要使用一個名為`$watch()`的`$scope`函數[$scope對象的方法]. 我們將在本章的後面詳細討論`watch`方法. 基本的用法是, 可以呼叫`$watch()`並給他傳遞一個監控表達式和一個用於回應表達式變化的回呼函數.

在這種情況下, 我們希望監控`funding.startEstimate`以及每當它改變時呼叫`computeNeeded()`. 然後我們使用這個方法重寫了`StartUpController`.
```js
    function StartUpController($scope){
    
        $scope.funding = { startingEstimate: 0 };
        
        $scope.computeNeeded = function(){
            $scope.needed = $scope.startingEstimate * 10;
        };
        
        $scope.$watch('funding.startingEstimate', computeNeeded);
        
    }
```
注意引號中的監控表達式. 是的, 它是一個字符串. 這個字符串是評估某些東西價格的Angular表達式. 表達式可以進行簡單的運算和訪問`$scope`對象的屬性. 在本章的後面我們會涵蓋更多關於表達式的訊息.

你也可以監控一個函數返回值, 但是它並不會監控`funding.startingEstimate`, 因為它賦值為0, 並且0[初始值]不再會改變.

然後, 由於每當我們的`funding.statingEstimates`改變時`funding.needed`都會自動更新, 我們可以像這樣編寫一個更簡單的樣板.
```html
    <form cn-controller="StartUpController">
        Starting: <input ng-model="funding.startEstimate">
        Recommendation: {{funding.needed}}
    </form>
```
在某些情況下, 你並不希望每一個改變都發生回應, 相反, 你希望等到用戶來告訴你它準備好了. 例如可能完成購買或者發送一個聊天記錄.

如果你的表單中有一組輸入框, 那麼你可以在這個表單上使用`ng-submit`指令給它指定一個提交表單時的回呼函數. 我們可以讓用戶經由點擊一個按鈕請求幫助他們啟動應用的方式來擴充上面的例子:
```html
    <form ng-submit="requestFunding()" ng-controller="StartUpController">
        Starting: <input ng-change="computeNeeded()" ng-model="startingEstimate">
        Recommendation: {{needed}}
        <button>Fun my startup</button>
    </form>
```
```js
    function StartUpController($scope){
        $scope.conputedNeeded = function(){
            $scope.needed = $scope.startingEstimate * 10;  
        };
        
        $scope.requestFunding = function(){
            window.alert("Sorry, please get more customers first.");
        };
    }
```
當嘗試提交這個表單時, `ng-submit`指令也會自動阻止瀏覽器處理其默認的`POST`行為.

> 原文此處有錯誤, 表單提交的默認行為是`GET`.

在需要處理其他事件的情況下, 就像當你想要提供交互而不是提交表單一樣, Angular提供了類似於瀏覽器原生事件屬性的事件處理指令. 對於`onclick`, 你應該使用`ng-click`. 對於`ondblclick`你應該使用`ng-dblclick`等等.

我們可以嘗試最後一次擴充我們的計算器啟動應用, 使用一個重置按鈕用於將輸入框的值重置為0.
```html
    <form ng-submit="requestFunding()" ng-controller="StartUpController">
        Starting: <input ng-change="computeNeeded()" ng-model="StartingEstimate">
        Recommendation: {{need}}
        <button>Fund my startup!</button>
        <button ng-click="reset()">Reset</button>
    </form>
    
    function StartUpController($scope){
    
        $scope.computeNeeded = function(){
            $scope.needed = $scope.startEstimate * 10;
        };
        
        $scope.requestFunding = function(){
            window.alert("Sorry, please get more customers first");
        };
        
        $scope.reset = function(){
            $scope.startEstimate = 0;
        }
    
    }
```
###不唐突JavaScript的一些話

在你JavaScript開發生涯的某些時刻, 有人可能會告訴你, 你應該編寫"不唐突的JavaScript", 在你的HTML中使用`click`, `mousedown`以及其他類似的內聯事件處理程序是不好的. 那麼他是正確.

不唐突的JavaScript思想已經有很多解釋, 但是其編碼風格的原理大致如下:

1. 不是每個人的瀏覽器都支持JavaScript. 讓每個人都能夠看到你所有的內容和使用你的應用程式, 而不需要在瀏覽器中執行代碼.

2. 有些人使用的瀏覽器工作方式不同. 視障人員使用的螢幕閱讀器和一些手機用戶並不能使用網站的JavaScript.

3. JavaScript在不同的平台工作機制不一樣. IE瀏覽器通常是罪魁禍首. 你需要根據瀏覽器的不同而使用不同的事件處理代碼.

4. 這些事件處理程序引用全域命名空間中的函數. 當你嘗試整合其他庫中的同名函數時, 它會讓你頭疼.

5. 這些事件處理程序合併了結構和行為. 這使你的代碼更加難以維護, 擴充和理解.

總體來看, 當你按照這種風格編寫JavaScript代碼, 一切都很好. 然而有一件事並不是好的, 那就是代碼的複雜度和可讀性. 並不是給元素聲明事件處理程序不起作用, 你通常給這些元素分配了ID, 獲得這些元素的引用, 並給它設定了事件處理的回呼函數. 你可以發明一個結構只用於清晰的創造它們之間的關聯, 但大多數應用程式結束於設定在各處的事件處理函數.

在Angular中, 我們決定重新審視這個問題.

在這些概念誕生以來世界就已經改變了. 第1點, 這類有趣的群體已經不再有了. 如果你運行的瀏覽器不支持JavaScript, 那麼你應該去使用20世紀90年代建立的網站. 至於第2點, 現代的螢幕閱讀器已經跟上來了. 隨著RAIA語義標籤的正確使用,  你可以創造易訪問的富UI應用. 現在手機上運行JavaScript與也能台式機能相提並論了.

因此現在的問題是: 重新恢復內聯技術來解決我們第3點和第4點的可讀性和簡潔性的問題嗎? 

正如前面所提到的, 對於大多數的內聯事件處理程序, Angular都有一個等價形式的`ng-eventhandler="expression"`來替代`click`, `mousedown`, `change`等事件處理程序. 當用戶點擊一個元素時, 如果你希望得到一個回應, 你只需要簡單的使用`ng-click`這樣的指令:
```html
    <div ng-click="doSomething()">…</div>
```
你的大腦裡可能會說"不, 這樣並不好"? 好消息是你可以放鬆下來. 這些指令不同於它們事件處理程序的前身(標準事件處理程序的原始形式):

+ 在每個瀏覽器中的行為一致. Angular會給你處理好差異.

+ 不會在全域命名空間操作. 你所指定的表達式僅僅能夠訪問元素控制器作用域內的函數和資料.

最後一點聽起來可能有點神秘, 因此讓我們來看一個例子. 在一個典型的應用程式中, 你會建立一個導航欄和一個隨著你從導航欄選擇不同選單而變化的內容區. 我們可以這樣編寫它的框架:
```html
    <div class="navbar" ng-controller="NavController">
    …
        <li class="menu-item" ng-click="doSomething()">Something</li>
    …
    </div>
    
    <div class="contentArea" ng-controller="ContentAreaController">
    …
        <div ng-click="doSomething()">…</div>
    …
    <div>
```
這裡當用戶點擊navbar中的`<li>`和conent區中的`<div>`時都會呼叫一個稱為`doSomething()`的函數. 作為開發人員, 你設定該函數呼叫你的控制器中的代碼引用. 它們可能是相同或者不同的函數:
```js
    function NavController($scope){
        $scope.doSomething = doA;
    }
    
    function ContentAreaController($scope){
        $scope.doSomething = doB;
    }    
```
這裡, `doA()`和`doB()`函數可能時相同或者不同的, 取決於你給它們的定義.

現在我們還剩下第5點, 合併結構和行為. 這是一個有爭議的話題, 因為你不能指出任何負面的結果, 但它與我們大腦裡所想的合併表現職責和應用程式邏輯的行為非常類似. 當人們談及關於標記結構和行為分離的時候, 這當然會有負面的影響.

如果我們的系統面臨這種耦合問題時, 這裡有一個簡單的測試可以幫助我們找出來: 我們可以給我們的應用程式邏輯建立一個單元測試, 而不需要DOM的存在.

在Angular中, 是的, 我們可以在控制器中只編寫包含業務邏輯的代碼而不必引用DOM. 在我們之前編寫的JavaScript中, 這個問題在事件處理程序中是不存在的. 注意, 在這裡以及在這本書的其他地方, 目前我們所編寫的控制器中, 都沒有引用DOM和任何DOM事件處理程序. 你可以很輕鬆建立出這些不帶DOM的控制器. 所有的元素定位和事件處理程序都發生在Angular中.

對於這個問題在編寫單元測試時. 如果你需要DOM, 你在測試中建立它, 只會增加測試程序的複雜度. 當你的頁面發生變化時, 你需要在你的測試中改變DOM, 這樣只會帶來更多的維護工作. 最後, 訪問DOM是很慢的, 測試緩慢意味著反饋不會及時以及最終解析都是緩慢的. Angular的控制器測試並沒有這些問題. 

因此你可以很輕鬆的聲明事件處理程序的簡單性和可讀性, 毫無罪惡感的違反最佳實踐.

###列表, 表格和其他重複的元素

最有用可能就是Angular指令, `ng-repeat`對於集合中的每一項都建立一次一組元素的一份副本. 你應該在你想建立列表問題的任何地方使用它.

比如說我們給老師編寫一個學生花名冊的應用程式. 我們可能從伺服器獲得學生的資料, 但是在這個例子中, 我們只在JavaScript將它定義為一個模型:
```js
    var students = [{name: 'Mary Contrary', id:'1'},
                    {name: 'Jack Sprat', id: '2'},
                    {name: 'Jill Hill', id: '3'}];
                    
    function StudentListController($scope){
        $scope.students = students;
    }
```
我們可以像下面這樣來顯示學生列表:
```html
    <ul ng-controller="">
        <li ng-repeat="student in students">
            <a href="/student/view/{{student.id}}">{{student.name}}</a>
        </li>
    </ul>
```
`ng-repeat`將會製作標籤內所有HTML的副本, 包括標籤內的東西. 這樣, 我們將看到:

+ Mary Contrary
+ Jack Sprat
+ Jill Hill

分別鏈接到*/student/view/1, /student/view/2, /student/view/3*.

正如我們之前所見, 改變學生數組將會自動改變渲染列表. 如果我們做一些例如插入一個新的學生到列表的事情:
```js
    var students = [{name: 'Mary Contrary', id:'1'},
                    {name: 'Jack Sprat', id: '2'},
                    {name: 'Jill Hill', id: '3'}];
                    
    function StudentListController($scope){
        $scope.students = students;
        
        $scope.insertTom = function(){
            $scope.students.splice(1, 0, {name: 'Tom Thumb', id: '4'});
        };
    }
```
然後在樣板中添加一個按鈕來呼叫:
```html
    <ul ng-controller="">
        <li ng-repeat="student in students">
            <a href="/student/view/{{student.id}}">{{student.name}}</a>
        </li>
    </ul>
    <button ng-click="insertTom()">Insert</button>
```
現在我們可以看到:

+ Mary Contrary
+ Tom Thumb
+ Jack Sprat
+ Jill Hill

`ng-repeat`指令還經由`$index`給你提供了當前元素的索引, 如果是集合中第一個元素, 中間的某個元素, 或者是最後一個元素使用`$first`, `$middle`和`$last`會給你提供一個布爾值.

你可以想像使用`$index`來標記表格中的行. 給定一個這樣的樣板:
```html
    <table ng-controller="AlbumController">
        <tr ng-repeat="track in album">
            <td>{{$index + 1}}</td>
            <td>{{track.name}}</td>
            <td>{{track.duration}}</td>
        </tr>
    </table>
```
這是控制器:
```js
    var album = [{name: 'Southwest Serenade', duration: '2:34'},
                 {name: 'Northern Light Waltz', duration: '3:21'},
                 {name: 'Eastern Tango', duration: '17:45'}];
                 
    function AlbumController($scope){
        $scope.album = album;
    };
```
我們得到如下結果:

1. Southwest Serenade     2:34
2. Northern Light Waltz   3:21
3. Eastern Tango         17:45

###隱藏與顯示

對於選單, 上下文敏感的工具[*原文:context-sensitive tools*]以及其他許多情況, 顯示和隱藏元素是一個關鍵的特性. 正如在Angular中, 我們基於模型的變化觸發UI的改變, 以及經由指令將改變反映到UI中. 

這裡, `ng-show`和`ng-hide`用於處理這些工作. 它們基於傳遞給它們的表達式提供顯示和隱藏的功能. 即, 當你傳遞的表達式為true時`ng-show`將顯示元素, 當為false時則隱藏元素. 當表達式為true時`ng-hide`隱藏元素, 為false時顯示元素. 這取決於你使用哪個更能表達的你意圖.

這些指令經由適當的設定元素的樣式為`display: block`來顯示元素, 設定樣式為`display: none`來隱藏元素. 讓我們看以個正在構建的Death Ray控制板的虛擬的例子:
```html
    <div ng-controller="DeathrayMenuController">
        <p><button ng-click="toggleMenu()">Toggle Menu</button></p>
        <ul ng-show="menuState.show">
            <li ng-click="stun()">Stun</li>
            <li ng-click="disintegrate()">Disintegrate</li>
            <li ng-click="erase()">Erase from history</li>
        </ul>
    </div>
```
```js
    function DeathrayMenuController($scope){
        $scope.menuState.show = false;
        
        $scope.toggleMenu = function(){
            $scope.menuState.show = !$scope.menuState.show;
        };
        
        // death ray functions left as exercise to reader
    };
```
###CSS類和樣式

顯而易見, 現在你可以在你的應用程式中經由使用{{ }}插值符號綁定資料的方式動態的設定類和樣式. 甚至你可以在你的應用程式中組成匹配的類名. 例如, 你想根據條件禁用一些選單, 你可以像下面這樣從視覺上顯示給用戶.

有如下CSS:
```css
    .menu-disabled-true {
        color: gray;
    }
```
你可以使用下面的樣板在你的DeathRay指示`stun`函數來禁用某些元素:
```html
    <div ng-controller="DeatrayMenuController">
        <ul>
            <li class="menu-disabled-{{isDisabled}}" ng-click="stun()">Stun</li>
            ...
        </ul>
    </div>
```
你可以經由控制器適當的設定`isDisabled`屬性的值:
```js
    function DeathrayMenuController($scope){
        $scope.isDisabled = false;
        
        $scope.stun = function(){
            //stun the target, then disable menu to allow regeneration
            $scope.isDisabled = 'true';
        };
    }
```
`stun`選單項的class將設定為`menu-disabled-`加`$scope.isDisabled`的值. 因為它初始化為false, 默認情況下結果為`menu-disabled-false`. 而此時這裡沒有與CSS規則匹配的元素, 則沒有效果. 當`$scope.isDisabled`設定為true時, CSS規則將變成`menu-disabled-true`, 此時則呼叫規則使文字為灰色.

這種技術也同樣適用於嵌入內聯樣式, 例如`style="{{some expression}}"`.

雖然想法很好, 但是這裡有一個缺點就是它使用了一個水平分割線來組合你的類名. 雖然在這個例子中很容易理解, 但是它可能很快就會變得難以管理, 你必須不斷的閱讀你的樣板和JavaScript來正確的建立你的CSS規則.

因此, Angular提供了`ng-class`和`ng-style`指令. 它們都接受一個表達式. 這個表達式的計算結果可以是下列之一:

+ 一個使用空格分割類名的字符串
+ 一個類名數組
+ 類名到布爾值的映射

讓我們想像一下, 你希望在應用程式頭部的一個標準位置顯示錯誤和警告給用戶. 使用`ng-class`指令, 你可以這樣做:
```css
    .error {
        background-color: red;
    }
    .warning {
        background-color: yellow;
    }
```
```html
    <div ng-controller="HeaderController">
        ...
        <div ng-class="{error: isError, warning: isWarning}">{{messageText}}</div>
        ...
        <button ng-click="showError()">Simulate Error</button>
        <button ng-click="showWarning()">Simulate Warning</button>
    </div>
```
```js
    function HeaderController($scope){
        $scope.isError = false;
        $scope.isWarning = false;
        
        $scope.showError = function(){
            $scope.messageText = 'This is an error';
            $scope.isError = true;
            $scope.isWarning = false;
        };
        
        $scope.showWarning = function(){
            $scope.messageText = 'Just a warning. Please carry on';
            $scope.isWarning = true;
            $scope.isError = false;
        };
    }
```
你甚至可以做出更漂亮的事情, 例如高亮表格中選中的行. 比方說, 我們要構建一個餐廳目錄並且希望高亮用戶點擊的那行.

在CSS中, 我們設定一個高亮行的樣式:
```css
    .selected {
        background-color: lightgreen;
    }
```
在模版中, 我們設定`ng-class`為`{selected: $index==selectedRow}`. 當模型中的`selectedRow`屬性匹配ng-repeat的`$index`時設定class為selected. 我們還設定一個`ng-click`來通知控制器用戶點擊了哪一行:
```html
    <table ng-controller="RestaurantTableController">
        <tr ng-repeat="restaurant in directory" ng-click="selectRestaurant($index)" ng-class="{selected: $index==selectedRow">
            <td>{{restaurant.name}}</td>
            <td>{{restaurant.cuisine}}</td>
        </tr>
    </table>
```
在我們的JavaScript中, 我們只設定虛擬的餐廳和建立`selectRow`函數:
```js
    function RestuarantTableController($scope){
        $scope.directory = [{name: 'The Handsome Heifer', cuisine: 'BBQ'},
                            {name: 'Green\'s Green Greens', cuisine: 'Salads'},
                            {name: 'House of Fine Fish', cuisine: 'Seafood'}];
        $scope.selectRestaurant = function(row){
            $scope.selectedRow = row;
        };
    }
```
###`src`和`href`屬性注意事項

當資料繫結給一個`<img>`或者`<a>`標籤時, 像上面一樣在`src`或者`href`屬性中使用{{ }}處理路徑將無法正常工作. 因為在瀏覽器中圖片與其他內容是並行加載的, 所以Angular無法攔截資料繫結的請求.

對於`<img>`而言最明顯的語法便是:
```html
    <img src="/images/cats/{{favoriteCat}}">
```
相反, 你應該使用`ng-src`屬性並像下面這樣編寫你的樣板:
```html
    <img ng-src="/images/cats/{{favoriteCat}}">
```
同樣的道理, 對於`<a>`標籤你應該使用`ng-href`:
```html
    <a ng-href="/shop/category={{numberOfBalloons}}">some text</a>
```
###表達式

表達式背後的思想是讓你巧妙的在你的樣板, 應用程式邏輯以及資料之間建立鉤子而與此同時防止應用程式邏輯偷偷摸摸的進入模版中.

直到現在, 我們一直主要是引用原生的資料作為表達式傳遞給Angular指令. 但是其實這些表達式可以做更多的事情. 你可以處理簡單的數學運算(+, -, /, *, %), 進行比較(==, !=, >, <, >=, <=), 執行布爾邏輯運算(&&, !!, !)以及按位運算(\^, &, |). 你可以呼叫暴露在控制器的`$scope`對像上的函數, 你還可以引用資料和對像表示法([], {}, …).

下面都是有效表達式的例子:
```html
    <div ng-controller="SomeController">
        <div>{{recompute() / 10}}<div>
        <ul ng-repeat="thing in things">
            <li ng-class="{highlight: $index % 4 >= threshold($index)}">
                {{otherFunction($index)}}
            </li>
        </ul>
    </div>
```
這裡的第一個表達式`recompute() / 10`是有效的, 是在樣板中設定邏輯很好的好例子, 但是應該避免這種方式. 保持視圖和控制器之間的職責分離可以確保它們容易理解和測試.

雖然你可以使用表達式做很多事情, 它們由Angular自定義的解釋器部分計算. 他們並不使用JavaScript的`eval()`執行, eval()有相當多的限制.

相反, 它們使用Angular自帶的自定義解釋器執行. 在裡面, 你不會看到循環結構(for, while等等), 流程控制語句(if-else, throw)或者改變資料的運算符(++, --). 當你需要使用這些類型的運算時, 你應該在你的控制器中使用指令進行處理.

儘管表達式在很多方面比JavaScript更加嚴格, 但它們對`undefined`和`null`並不是很嚴格(更寬鬆). 樣板只是簡單的渲染一些東西, 並不會拋出一個`NullPointerException`的錯誤. 這樣就允許你安全的使用模型而沒有限制, 並且只要它們得到資料填充就讓它們出現在用戶界面中.

###分離用戶界面(UI)和控制器職責

在你的應用程式中控制器有三個職責:

+ 在你的應用程式的模型中設定初試狀態.[初始化應用程式]
+ 經由`$scope`暴露模型和函數到視圖中.
+ 監控模型的改變並觸發行為.

對於第一點第二點在本章的已經看過更多例子. 稍候我們會討論最後一點. 然而, 控制器其概念上的目的, 是提供代碼或者執行用戶與視圖交互願望的邏輯. 

為了保持控制器的小巧和易於管理, 我們建議你針對視圖的每一個區域建立一個控制器. 也就是說, 如果你有一個選單則建立一個`MenuController`. 如果你有一個麵包屑導航, 則編寫一個`BreadcrumbController`, 等等.

你可能開始懂了, 但是需要明確的將控制器綁定到一個指定的DOM塊中用於管理它們. 有兩種主要的方式關聯控制器與DOM節點, 一種方式是在樣板中指定一個`ng-controller`屬性, 另一種方式是經由`route`(路由)關聯一個動態加載的DOM樣板片段, 也稱作視圖.

我們將在本章的後面再討論關於視圖和路由的訊息.

如果你的UI中有一個複雜的片段, 你可以經由建立嵌套的控制器, 經由繼承樹來共享模型和函數來保持你的代碼間接性和可維護性. 嵌套控制器很簡單, 你可以簡單的在另一個DOM中分配一個控制器到一個DOM元素中做到這一點, 就像這樣:
```html
    <div ng-controller="ParentController">
        <div ng-controller="ChildController">…</div>
    </div>
```
雖然我們將這個表達為控制器嵌套, 實際的嵌套發生在作用域中($scope對像中). 傳遞給嵌套控制器的`$scope`繼承自父控制器的`$scope`原型, 這意味著傳遞給`ChildController`的`$scope`將有權訪問傳遞給`ParentController`的`$scope`的所有屬性.

###使用作用域發佈模型資料

將`$scope`對像傳遞給我們的控制器便是我們將模型資料暴露給視圖的機制. 可能你的應用程式中還有其他的資料, 但Angular中只能夠經由scope訪問它可以訪問的模型部分的屬性. 你可以認為scope就是作為一個上下文環境用於在你的模型中觀察變化的.

我們已經看過了很多明確設定作用域的例子, 就像`$scope.count = 5`. 也有一些間接的方法在樣板內設定其自身的模型. 你可以像下面這樣做:

1. 經由表達式. 由於表達式運行在控制器的作用域關聯的元素的上下文中, 在表達式中設定屬性與在控制器的作用域中設定一個屬性一樣. 

也就是像這樣:  
```html
    <button ng-click="count=3">Set count to three</button>
```
這樣做也有相同的效果:
```html
    <div ng-controller="CountController">
        <button ng-click="setCount()">Set count to three</button>
    </div>
```
CountController定義如下:
```js
    function CountController($scope){
        $scope.setCount = function(){
            $scope.count = 3;
        }
    }
```
2. 在表單的輸入框中使用`ng-model`. 在表達式中, 模型被指定為`ng-model`的參數也適用於控制器作用域範圍. 此外, 這將在表單字段和你指定的模型之間建立一個雙向資料繫結.

###使用$watch監控模型變化

所有scope函數中最常用的可能就是$watch了, 當你的模型部分發生變化時它會通知你. 你可以監控單個對象屬性, 也可以監控計算結果(函數), 幾乎所有的事物都可當作一個屬性或者一個JavaScript運算能夠被訪問. 該函數的簽名如下:
```js
    $watch(watchFn, watchAction, deepWatch);
```
每個參數的詳細訊息如下:

**watchFn**

> 這個參數是一個Angular字符串表達式或者是一個返回你所希望監控的模型當前值的函數. 這個表達式會被多次執行, 因此你需要確保它不會有副作用. 也就是說, 它可以被呼叫多次而不改變狀態. 同樣的原因, 監控表達式也應該是運算複雜度低的(執行簡單的運算). 如果你傳遞一個字符串的表達式, 它將會對其呼叫的(執行的表達式)作用域中的有效對像求值.

**watchAction**

> 這是`watchFn`繁盛變化時會被呼叫的函數或者表達式. 在函數形式中, 它接受`watchFn`的新值, 舊值以及作用域的引用. 其簽名就是`function(newValue, oldValue, scope)`.

**deepWatch**

> 如果設定為true, 這個可選的布爾參數用於告訴Angular檢查所監控的對象中每一個屬性的變化. 如果你希望監控數組的個別元素或者對象的屬性而不是一個普通的值, 那麼你應該使用它. 由於Angular需要遍歷數組或者對像, 如果集合(數組元素/對像成員)很大, 那麼計算的代價會非常高.

當你不再想收到變化通知時, `$watch`函數將返回一個註銷監聽器的函數.

如果我們像監控一個屬性, 然後在稍後註銷它, 我們將使用下面的方式:
```js
    ...
    var dereg = $scope.$watch('someModel.someProperty', callbackOnChange);
    ...
    dereg();
```
讓我們回顧一下第一章中完整的購物車範例. 比方說, 當用戶在他的購物車中添加了超出100美元的商品時, 我們希望申請10美元的優惠. 我們使用下面的樣板:
```html
    <div ng-controller="CartController">
        <div ng-repeat="item in items">
            <span>{{item.title}}</span>
            <input ng-model="item.quantity">
            <span>{{item.price | currency}}</span>
            <span>{{item.price * item.quantity | currency}}</span>
        </div>
        <div>Total: {{totalCart() | currency}}</div>
        <div>Discount: {{bill.discount | currency}}</div>
        <div>Subtotal: {{subtotal() | currency}}</div>
    </div>
```
緊接著是`CartController`, 它看起來像下面這樣:
```js
    function CartController($scope){
        $scope.bill = {};

        $scope.items = [
            {title: 'Paint pots', quantity: 8, price: 3.95},
            {title: 'Polka dots', quantity: 17, price: 12.95},
            {title: 'Pebbles', quantity: 5, price: 6.95}
        ];

        $scope.totalCart = function(){
            var total = 0;
            for (var i = 0, len = $scope.items.length; i < len; i++){
                total = total + $scope.items[i].price* $scope.items[i].quantity;
            }

            return total;
        };

        $scope.subtotal = function(){
            return $scope.totalCart() - $scope.discount;
        };

        function calculateDiscount(newValue, oldValue, scope){
            $scope.bill.discount = newValue > 100 ? 10 : 0;
        }

        $scope.$watch($scope.totalCart, calculateDiscount);
    }
```
注意`CartController`的底部, 我們給用於計算所購買商品總價的`totalCart()`的值設定了一個監控. 每當這個值變化時, 監控都會呼叫`calculateDiscount()`, 並且會給discount(優惠項)設定一個適當的值. 如果總價為$100, 我們將設定優惠為$10. 否則, 優惠就為$0.

你可以看到這個展示給用戶的例子如圖2-1所示:

![use-$watch](figure/watch1.png)

圖2-1 Shopping cart with discount

###watch()中的性能注意事項

前面例子會正確的執行, 但是這裡有一個潛在的性能問題. 雖然並不明顯, 如果你在`totalCart()`中設定一個偵錯斷點, 你會發現在渲染頁面時它被呼叫了6次. 雖然在這個應用程式中你從來沒有注意到它, 但是在更多複雜的應用程式中, 運行它6次可能是一個問題.

為什麼是6次? 其中3次我們可以很輕易的跟蹤到, 因為它分別在下面三個過程中運行一次:

+ 在`{{totalCart() | currency}}`樣板中
+ `subtotal()`函數中
+ `$watch()`函數中

然後是Angular再運行它們一次, 因而帶給我們6次運行. Angular這樣做是為了驗證在你的模型中變化是否完全傳播出去以及驗證你的模型是否穩定. Angular經由檢查一份所監控屬性的副本與它們當前值比較來確認它們是否改變. 事實上, Angular也可以運行它多達十次來確保是否完全傳播開. 如果發生這種情況, 你可能需要依賴循環來修復它.

雖然你現在會擔心這個問題, 但是當你閱讀完本書時它可能就不再是問題了. 然而Angular不得不在JavaScript中實現資料繫結, 我們一直與TC39的人共同努力實現一個底層的原生的`Object.observe()`. 一旦有了它, Angular將自動使用`Object.observe()`隨時隨地呈現給你一個原生效率的資料繫結.

> 譯注: [TC39](http://www.ecma-international.org/memento/TC39.htm)

在下一章中你會看到, Angular有一個很好的Chrome偵錯擴充程序(Chrome插件)Batarang, 它將自動給你突出(高亮)昂貴的資料繫結(從性能的角度而言, 表示資料繫結的方式並不是較好的方式).

> 譯注:
> 
> + [Batarang](https://chrome.google.com/webstore/detail/ighdmehidhipcmcojjgiloacoafjmpfk) - 這是一個Angular偵錯與性能監控工具.
> + [Batarang-Github](https://github.com/angular/angularjs-batarang)

現在我們知道了這個問題, 這裡有一些方法可以解決它. 一種方式是在items數組變化時建立`$watch`並且只重新計`$scope`的total, discount和subtotal屬性值.

做到這一點, 我們只需要使用這些屬性更新樣板:
```html
    <div>Total: {{bill.total | currency}}</div>
    <div>Discount: {{bill.discount | currency}}</div>
    <div>Subtotal: {{bill.subtotal | currency}}</div>
```
然後, 在JavaScript中, 我們要監控items數組, 以及呼叫一個函數來計算數組任意改變的總值:
```js
    function CartController($scope){
        $scope.bill = {};
        
        $scope.items = [
            {title: 'Paint pots', quantity: 8, price: 3.95},
            {title: 'Polka dots', quantity: 17, price: 12.95},
            {title: 'Pebbles', quantity: 5, price: 6.95}
        ];
        
        var calculateTotals = function(){
            var total = 0;
            for(var i = 0, len = $scope.items.length; i < len; i++){
                total = total + $scope.items[i].price * $scope.items[i].quantity;
            }
            
            $scope.bill.totalCart = total;
            $scope.bill.discount = total > 100 ? 10 : 0;
            $scope.bill.subtotal = total - $scope.bill.discount;
        };
        
        $scope.$watch('items', calculateTotals, true);
    }
```
注意這裡`$watch`指定了一個`items`字符串. 這可能是因為`$watch`函數可以接受一個函數(正如我們之前那樣)或者一個字符串. 如果傳遞一個字符串給`$watch`函數, 在`$scope`呼叫的作用域中它將被當作一個表達式.

這種測策略在你的應用程式中可能工作得很好. 然而, 由我監控的是items數組, Angular將會製作一個副本以供我們進行比較. 對於一個較大的items清單, 如果我們在Angular每一次計算頁面結果時只重新計算bill屬性值, 它可能表現得更好. 我們可以經由建立一個`$watch`來做到這一點, 它帶有只用於重新計算屬性的`watchFn`函數. 就像這樣:
```js
    $scope.$watch(function(){
        var total = 0;
        for(var i = 0, i < $scope.items.length; i++){
            total = total + $scope.items[i].price * $scope.items[i].quantity;
        };
        
        $scope.bill.totalCart = total;
        $scope.bill.discount = total > 100 ? 10 : 0;
        $scope.bill.subtotal = total - $scope.bill.discount;
    });
```
####多個監控

如果你想監控多個屬性或者對像, 並且每當它們發生任何變化時都執行一個函數. 你有兩個基本的選擇:

+ 監控屬性索引值.
+ 把它們放入數組或者對像總並且將傳遞的`deepWatch`設定為true.

> 譯注: 原文中兩個選項排列順序顛倒. 譯文中糾正了順序並給出對應的訊息.

在第一種情況下, 如果作用域中有一個對像擁有兩個屬性`a`和`b`, 並且希望在發生變化時執行`callMe()`函數, 你應該同時監控它們, 就像這樣:
```js
    $scope.$watch('things.a + things.b', callMe(…));
```
當然, 屬性`a`和`b`可能在不同的對象中, 只要你喜歡你也可以製作這個列表. 如果列表很長, 你可能更喜歡編寫一個返回索引值的函數而不是依靠一個邏輯表達式.

在第二種情況下, 你可能希望監控`things`對像中的所有屬性. 在這種情況下, 你可以這樣做:
```js
    $scope.$watch('things' calMe(…), true);
```
這裡, 經由將第三個參數設定為`true`來要求Angular遍歷`things`對象的屬性並在它們發生任何改變時呼叫`callMe()`. 這同樣適用於數組, 只是這裡是針對一個對像.

##使用模組組織依賴

在任何不平凡的應用程式中, 在你的代碼領域中弄清楚如何組織功能職責通常都是一項艱巨的任務. 我們已經看到了控制器是如何到視圖樣板中給我們提供一個存放暴露正確資料和函數的區域. 但是我們在哪裡安置支持應用程式的的其他代碼呢? 最明顯的方式就是將它們放置在控制器中的函數中.

對於小型應用程式和目前我們所見過的例子這中方式工作得很好, 但是在實際的應用程式中將很快變得難以管理. 控制器將成為堆積一切以及我們需要做任何事情的垃圾場. 它們可能很難理解, 也可能很難改變(難以維護).

引入模組. 在你的應用程式功能區, 它們提供了一種組織依賴的方式, 以及一種自解決依賴的機制(也稱為依賴注入[*第一章中已經介紹了什麼是依賴注入*]). 一般情況下, 我們稱之為依賴關係服務, 它們給我們的應用程式提供特殊服務.

比如, 如果在我們的購物網站中控制器需要從伺服器取得一個出售項目列表, 我們需要一些對像--讓我們稱之為`Items`--注意這裡是從伺服器取得的項目. 反過來, `Items`對像, 需要一些方式經由XHR或者WebSockets與伺服器上的資料庫通信.

不適用模組處理看起來像這樣:
```js
    function ItemsViewController($scope){
        // 向伺服器發起請求
        ...
        
        // 進入Items對像解析回應
        ...
        
        // 在$scope中設定Items數組以便視圖可以顯示它
    }
```
然而這確實能夠工作, 但是它存在一些潛在的問題.

+ 如果一些其他的控制器還需要從伺服器取得`Items`, 那我們現在要複製這個代碼. 這造成了維護的負擔, 如果我們現在要構造模式或者其他的變化, 我們必須在好幾個地方更新這個代碼.
+ 考慮到其他因素, 如伺服器驗證, 解析複雜度等等, 這也是很難推斷控制器對像職責界限的原因, 代碼也很難閱讀.
+ 對這段代碼進行單元測試, 我們需要一台實際運行的伺服器或者使用XMLHttpRequest打補丁返回模擬資料. 運行伺服器進行測試將導致測試很慢, 配置它很痛苦, 它通常展示了測試中的碎片. 而打補丁的方式解決了速度和碎片問題, 但是這意味著你必須記住在測試中清理任何不定對像, 這樣就帶來了額外的複雜度和脆弱性, 因為它迫使你指定準確的線上版本的資料格式(每當格式變化時都需要更新測試).

對於模組和從它們哪裡取得的依賴注入, 我們就可以編寫更簡潔的控制器, 像這樣:

    function ShoppingController($scope, Items){
        $scope.items = Items.query();
    }
        
現在你可能會問自己, '當然, 這看起來很酷, 但是這個Items從哪裡來?'. 前面的代碼假設我們已經定義了作為服務的`Items`.

服務是一個單獨的對象(單例對像), 它執行必要的任務來支持應用程式的功能. Angular自帶了很多服務, 例如`$location`, 用於與瀏覽器中的地址交互, `$route`, 用於基於位置(URL)的變化切換視圖, 以及`$http`用於與伺服器通信.

你可以也應該建立你自己的服務去處理應用程式所有的特殊任務. 在需要它們時服務可以共享給任何控制器. 因此, 當你需要跨控制器通信和共享狀態時使用它們是一個很好的機制. Angular綁定的服務都以`$`開頭, 所以你也能夠命名它們為任何你喜歡的東西, 這是一個很好的主意, 以避免使用`$`開頭帶來的命名衝突問題.

你可以使用模組對象的API來定義服務. 這裡有三個函數用於建立通用服務, 它們都有不同層次的複雜性和能力:
```html
<table>
    <thead>
        <tr>
            <th>Function</th>
            <th>定義(Defines)</td>
        </tr>
    </thead>
    <tbody>
        <tr>
            <td>provider(name, Object/constructor())</td>
            <td>一個可配置的服務, 帶有複雜的建立邏輯. 如果你傳遞一個對像, 它應該有一個名為`$get`的函數, 用於返回服務的實例. 否則, Angular會假設你傳遞了一個構造函數, 當呼叫它時建立實例.</td>
        </tr>
        <tr>
            <td>factory(name, $get Function())</td>
            <td>一個不可配置的服務也帶有複雜的建立邏輯. 你指定一個函數, 當呼叫時, 返回服務實例. 你可以認為這和<code>provider(name, { $get: $getFunction()})</code>一樣</td>
        </tr>
        <tr>
            <td>service(name, constructor())</td>
            <td>一個不可配置的服務, 其建立邏輯簡單. 就像<code>provider</code>的構造函數選項, Angular呼叫它來建立服務實例.</td>
        </tr>                
    </tbody>
</table>
```
我們稍後再來看`provider()`的配置選項, 現在我們先來使用`factory()`討論前面的Items例子. 我們可以像這樣編寫服務:
```js
    // Create a module to support our shopping views.
    var shoppingModule = angular.module('ShoppingModule', []);
    
    // Set up service factory to create our Items interface to the server-side database
    shoppingModule.factory('Items', function(){
        var items = {};
        items.query = function(){
            // In real apps, we'd pull this data from the server…
            return [
                {title: 'Paint pots', description: 'Pots full of paint', price: 3.95},
                {title: 'Polka dots', description: 'Dots with polka', price: 2.95},
                {title: 'Pebbles', description: 'Just little rocks', price: 6.95}
            ];
        };
        
        return items;
    });
```
當Angular建立`ShoppingController`時, 它會將`$scope`和我們剛才定義的新的Items服務傳遞進來. 這是經由參數名稱匹配完成的. 也就是說, Angular會看到我們的`ShoppingController`類的函數簽名, 並通知它(控制器)發現一個Items對像. 由於我們定義Items為一個服務, 它會知道從哪裡取得它.

以字符串的形式查詢這些依賴結果意味著作為參數注入的函數就像控制器的構造函數一樣是順序無關的. 並不是必須這樣:
```js
    function ShoppingController($scope, Items){...}    
```
我們也可以這樣編寫:
```js
    function ShoppingController(Items, $scope){...}
```
依然和我們所希望的功能一樣.

為了在樣板中使用它, 我們需要告訴`ng-app`指令我們的模組名稱, 就像下面這樣:
```html
    <html ng-app="ShoppingModule">
```
為了完成這個例子, 我們可以這樣實現樣板的其餘部分:
```html
    <body ng-controller="ShoppingController">
        <h1>Shop!</h1>
        <table>
            <tr ng-repeat="item in items">
                <td>{{item.title}}</td>
                <td>{{item.description}}</td>
                <td>{{item.price | currency}}</td>
            </tr>
        </table>
    </body>
```
應用的返回結果看起來如圖2-2所示:

![use-module](figure/useModule.png)

圖2-2 Shop items

###我們需要多少模組?

作為服務本身可以有依賴關係, Module API允許你在的依賴中定義依賴關係.

在大多數應用程式中, 建立一個單一的模組將所有的代碼放入其中並將所有的依賴也放在裡面足以很好的工作. 如果你使用來自第三方庫的服務或者指令, 它們自帶有其自身的模組. 由於你的應用程式依賴它們, 你可以引用它們作為你的應用程式的依賴.

舉個例子, 如果你要包含(虛構的)模組SnazzyUIWidgets和SuperDataSync, 應用程式的模組聲明看起來像這樣:
```js
    var appMod = angular.module('app', ['SnazzyUIWidgets', 'SuperDataSync']);
```
##使用過濾器格式化資料

過濾器允許你在樣板中使用插值方式聲明如何轉換資料並顯示給用戶. 使用過濾器的語法如下:

    {{expression | filterName : parameter1 : … parameterN }}
    
其中表達式是任意的Angular表達式, `filterName`是你想使用的過濾器名稱, 過濾器的參數使用冒號分割. 參數自身也可以是任意有效的Angular表達式.

Angular自帶了幾個過濾器, 像我們已經看到的currency:

    {{12.9 | currency}}
    
這段代碼顯示如下:

> $12.9

你不僅限於使用綁定的過濾器(Angular內置的), 你可以簡單的編寫你自己的過濾器. 例如, 如果我們想建立一個過濾器來讓標題的首字母大寫, 我們可以像下面這樣做:
```js
    var homeModule = angular.module('HomeModule', []);
    homeModule.filter('titleCase', function(){
        var titleCaseFilter = function(input){
            var words = input.split(' ');
            for(var i = 0; i < words.length; i++){
                words[i] = words[i].charAt(0).toUpperCase() + words[i].slice(1);
            }
            
            return words.join(' ');
        };
        return titleCaseFilter;
    });
```
有一個像這樣的樣板:
```html
    <body ng-app="HomeModule" ng-controller="HomeController">
        <h1>{{pageHeading | titleCase}}</h1>
    </body>
```
然後經由控制器插入`pageHeading`作為一個模型變數:
```js
    function HomeController($scope){
        $scope.pageHeading = 'behold the majesty of you page title';
    }
```
我們會看到如圖2-3所示的東西:

![titleCase](figure/titleCase.png)

圖2-3 Title case filter

##使用路由和$location更新視圖

儘管Ajax從技術上講是單頁應用程式(理論上它們僅僅在第一次請求時加載HTML頁面, 然後只需在DOM中更新區塊), 我們通常會有多個子頁面視圖用於適當的顯示給用戶或者隱藏.

我們可以使用Angular的`$route`服務來給我們管理這個場景. 讓你指定路由, 對於瀏覽器指向給定的URL, Angular將加載並顯示一個樣板, 並且實例化一個控制器給樣板提供上下文環境.

經由呼叫`$routeProvider`服務的功能作為配置塊來在你的應用程式中建立視圖. 就像這樣的偽代碼:
```js
    var someModule = angular.module('someModule', [… Module dependencies …]);
    someModule.config(function($routeProvider){
        $routeProvider.
            when('url', {controller: aController, templateUrl: '/path/to/template'}).
            when(…other mappings for your app …).
            … 
            otherwise(…what to do if nothing else matches…);
    });
```
上面的代碼表示當瀏覽器的URL變化為指定的URL時, Angular將從`/path/to/template`中加載樣板, 並使用`aController`關聯這個樣板的根元素(就像我們輸入`ng-controller=aController`).

在最後一行呼叫`otherwise()`用於告訴路由如果沒有其他的匹配則跳到哪裡.

讓我們來使用一下. 我們正在構建一個email應用程式將輕鬆的戰勝Gmail, Hotmail以及其他的. 我們暫且稱它為A-mail. 現在, 讓我們從簡單的開始. 我們的首屏中顯示一個包括日期, 標題以及發送者的郵件訊息列表. 當你點擊一個訊息, 它應該向將郵件的正文訊息顯示給你.

> 由於瀏覽器的安全限制, 如果你想自己測試這些代碼, 你需要在一個Web服務奇商進行而不是使用`file://`. 如果你安裝了Python, 你可以在你的工作目錄經由執行`python -m SimpleHTTPServer 8888`來使用這些代碼.

對於主樣板, 我們會做一點不同的東西. 而不是將所有的東西都放在首屏來加載, 我們只會建立一個用於放置視圖的部署樣板. 我們會持續在視圖中放置視圖, 比如選單. 在這種情況下, 我們只需要顯示一個標題包含應用的名稱. 然後使用`ng-view`指令來告訴Angular我們希望視圖出現在哪裡.

###*index.html*
```html
    <html ng-app="Amail">
        <head>
            <script src="js/angular.js"></script>
            <script src="js/controllers.js"></script>
        </head>
        <body>
            <h1>A-Mail</h1>
            <div ng-view></div>
        </body>
    </html>
```
由於我們的視圖樣板將被插入到剛剛建立的容器中, 我們可以把它們編寫為區域的HTML文件. 對於郵件列表, 我們將使用`ng-repeat`來遍歷訊息列表並將它們渲染到一個表格中.

###*list.html*
```html
    <table>
        <tr>
            <td><strong>Sender</strong></td>
            <td><strong>Subject</strong></td>
            <td><strong>Date</string></td>
        </tr>
        <tr ng-repeat="message in messages">
            <td>{{message.sender}}</td>
            <td><a href='#/view/{{message.id}}'>{{message.subject}}</a></td>
            <td>{{message.date}}</td>
        </tr>
    </table>
```
注意這裡我們打算讓用戶經由點擊主題將他導航到詳細訊息中. 我們將URL資料繫結到`message.id`上, 因此點擊一個`id=1`的消息將使用戶跳轉到`/#/view/1`. 我們將經由url進行導航, 也稱為深度鏈接, 在詳細訊息視圖的控制器中, 讓特定的消息對應一個詳情視圖.

為了建立消息的詳情視圖, 我們將建立一個顯示單個message對像屬性的樣板.

###*detail.html*
```html
    <div><strong>Subject:</strong> {{message.subject}}</div>
    <div><strong>Sender:</strong> {{message.sender}}</div>
    <div><strong>Date:</strong> {{message.date}}</div>
    <div>
        <strong>To:</strong>
        <span ng-repeat="recipient in message.recipients">{{recipient}}</span>
    </div>
    <div>{{message.message}}</div>
    <a href="#/">Back to message list</a>
```
現在, 將這些樣板與一些控制器關聯起來, 我們將配置`$routeProvider`與URLs來呼叫控制器和樣板.

###*controllers.js*
```js
    //Create a module for our core AMail services
    var aMailServices = angular.module('AMail', []);
    
    //Set up our mappings between URLs, tempaltes. and  controllers
    function emailRouteConfig($routeProvider){
        $routeProvider.
        when('/', {
            controller: ListController,
            templateUrl: 'list.html'
        }).
        // Notice that for the detail view, we specify a parameterized URL component by placing a colon in front of the id
        when('/view/:id', {
            controller: DetailController,
            templateUrl: 'detail.html'
        }).
        otherwise({
            redirectTo: '/'
        });
    };
    
    //Set up our route so the AMailservice can find it
    aMailServices.config(emailRouteConfig);
    
    //Some take emails
    messages = [{
        id: 0, sender: 'jean@somecompany.com',
        subject: 'Hi there, old friend',
        date: 'Dec 7, 2013 12:32:00',
        recipients: ['greg@somecompany.com'],
        message: 'Hey, we should get together for lunch somet ime and catch up. There are many things we should collaborate on this year.'
    },{
        id: 1, sender: 'maria@somecompany.com',
        subject : 'Where did you leave my laptop?' ,
        date: 'Dec 7, 2013 8:15:12',
        recipients: ['greg@somecompany.com'],
        message: 'I thought you were going to put it in my desk drawer. But i t does not seem to be there. '
    },{
        id: 2, sender: 'bill@somecompany.com',
        subject: 'Lost python',
        date: 'Dec 6, 2013 20:35:02',
        recipients: ['greg@somecompany.com'],
        message: "Nobody panic, but my pet python is missing from her cage. She doesn't move too fast, so just call me if you see her."
    }];

    // Publish our messages for the list template

    function ListController($scope){
        $scope.messages = messages;
    }

    //Get the message id fron the route (parsed from the URL) and use it to find the right message object.
    function DetailController($scope, $routeParams){
        $scope.message = messages[$routeParams.id];
    }
```
我們已經建立了一個帶有多個視圖的應用程式的基本結構. 我們經由改變URL來切換視圖. 這意味著用戶也能夠使用前進和後退按鈕進行工作. 用戶可以在我們的應用程式中添加書籤和郵件鏈接, 即使只有一個真正的HTML頁面.

##對話伺服器

好了, 閒話少說. 實際的應用程式通常與真正的伺服器通訊. 移動應用和新興的Chrome桌面應用程式可能有些例外, 但是對於其他的一切, 你是否希望它持久保存雲端或者與用戶實時交互, 你可能希望你的應用程式與伺服器通信.

對於這一點Angular提供了一個名為`$http`的服務. 它有一個抽像的廣泛的列表使得它能夠很容易與伺服器通信. 它支持普通的HTTP, JSONP以及CORS. 還包括防止JSON漏洞和XSRF的安全協議. 它讓你很容易轉換請求和資料回應, 甚至還實現了簡單的快取. 

比方說, 我們希望從伺服器檢索購物站點的商品而不是我們的內存中模擬. 編寫伺服器的訊息超出了本書的範圍, 因此讓我們想像一下我們已經建立了一個服務, 當你構造一個`/product`查詢時, 它返回一個JSON形式的產品列表.

給定一個回應, 看起來像這樣:
```json
    [
        {
            "id": 0,
            "title": "Paint pots",
            "description": "Pots full of paint",
            "price": 3.95
        },
        {
            "id": 1,
            "title": "Polka dots",
            "description": "Dots with that polka groove",
            "price": 12.95
        },
        {
            "id": 2,
            "title": "Pebbles",
            "description": "Just little rocks, really",
            "price": 6.95
        }
        … etc …     
    ]
```
我們可以這樣編寫查詢:
```js
    function ShoppingController($scope, $http){
        $http.get('/products').success(function(data, status, headers, config){
            $scope.items = data;
        });
    }
```
然後像這樣在樣板中使用它:
```html
    <body ng-controller="ShoppingController">
        <h1>Shop!<h1>
        <table>
            <tr ng-repeat="item in items">
                <td>{{item.title}}</td>
                <td>{{item.description}}</td>
                <td>{{item.price | currency}}</td>
            </tr>
        </table>
    </body>
```
正如我們之前所學習到的, 從長遠來看我們將這項工作委託到伺服器通信服務上可以跨控制器共享是明智的. 我們將在第5章來看這個結構和全方位的討論`$http`函數.

##使用指令更新DOM

指令擴充HTML語法, 也是將行為與DOM轉換的自定義元素和屬性關聯起來的方式. 經由它們, 你可以建立復用的UI組件, 配置你的應用程式, 做任何你能想到在樣板中要做的事情.

你可以使用Angular自帶的內置指令編寫應用, 但是你可能會希望運行你自己所編寫的指令的情況. 當你希望處理瀏覽器事件和修改DOM時, 如果無法經由內置指令支持, 你會知道是時候打破指令規則了. 你所編寫的代碼在指令中, 不是在控制器中, 服務中, 也不是應用程式的其他地方.

與服務一樣, 經由module對象的API呼叫它的`directive()`函數來定義指令, 其中`directiveFunction`是一個工廠函數用於定義指令的功能(特性).
```js
	var appModule = angular.module('appModule', [...]);
	appModule.directive('directiveName', directiveFunction);
```
編寫指令工廠函數是很深奧的, 因此在這本書中我們專門頂一個完整的一章. 吊吊你的胃口, 不過, 我們先來看一個簡單的例子.

HTML5中有一個偉大的稱為`autofocus`的新屬性, 將鍵盤的焦點放到一個input元素. 你可以使用它讓用戶第一時間經由他們的鍵盤與元素交互而不需要點擊. 這是很好的, 因為它可以讓你聲明指定你希望瀏覽器做什麼而無需編寫任何JavaScript. 但是如果你希望將焦點放到一些非input元素上, 像鏈接或者任何`div`上會怎樣? 如果你希望它也能工作在不支持HTML5中會怎樣? 我們可以使用一個指令做到這一點.
```js
	var appModule = angular.module('app', []);
	
	appModule.directive('ngbkFocus', function(){
		return {
			link: function(scope, elements, attrs, controller){
				element[0].focus();
			}
		};
	});
```
這裡, 我們返回指令配置對像帶有指定的link函數. 這個link函數取得了一個封閉的作用域引用, 作用域中的DOM元素, 傳遞給指令的任意屬性數組, 以及DOM元素的控制器, 如果它存在. 這裡, 我們僅僅只需要取得元素並呼叫它的`focus()`方法.

然後我們可以像這樣在一個例子中使用它:

###*index.html*
```html
	<html lang="en" ng-app="app">
		...include angular and other scripts...
		<body ng-controller="SomeController">
			<button ng-click="clickUnfocused()">
				Not focused
			</button>
			<button ngbk-focus ng-click="clickFocused()">
				I'm very focused!
			</button>
			<div>{{message.text}}</div>
		</body>
	</html>
```
###*controller.js*
```js
	function SomeController($scope) {
		$scope.message = { text: 'nothing clicked yet' };

		$scope.clickUnfocused = function() {
			$scope.message.text = 'unfocused button clicked';
		};

		$scope.clickFocused = function {
			$scope.message.text = 'focus button clicked';
		}
	}

	var appModule = angular.module('app', ['directives']);
```
當載入頁面時, 用戶將看到標記為"I'm very focused!"按鈕帶有高亮焦點. 敲擊空格鍵或者回車鍵將導致點擊並呼叫`ng-click`, 將設定div的文字為"focus button clicked". 在瀏覽器中打開這個頁面, 我們將看到如圖2-4所示的東西:

![foucsed](figure/custom-directive.png)

圖2-4 Foucs directive

##驗證用戶輸入

Angular帶有幾個適用於單頁應用程式的不錯的功能來自動增強`<form>`元素. 其中之一個不錯的特性就是Angular讓你在表單內的input中聲明驗證狀態, 並允許在整組元素經由驗證的情況下才提交.

例如, 如果我們建立一個登錄表單, 我們必須輸入一個名稱和email, 但是有一個可選的年齡字段, 我們可以在他們提交到伺服器之前驗證多個用戶輸入. 如下加載這個例子到瀏覽器中將顯示如圖2-5所示:

![valid](figure/signup.png)

圖2-5. Form validation

我們還希望確保用戶在名稱字段輸入文字, 輸入正確形式的email地址, 以及他可以輸入一個年齡, 它才是有效的.

我們可以在樣板中做到這一點, 使用Angular的`<form>`擴充和各個input元素, 就像這樣:
```html
	<h1>Sign Up</h1>
	<form name='addUserForm'>
		<div>First name: <input ng-model='user.first' required></div>
		<div>Last name: <input ng-model='user.last' required></div>
		<div>Email: <input type='email' ng-model='user.email' required></div>
		<div>Age: <input type='number' ng-model='user.age' ng-maxlength='3' ng-minlength='1'></div>
		<div><button>Submit</button></div>
	</form>
```
注意, 在某些字段上我們使用了HTML5中的`required`屬性以及`email`和`number`類型的input元素來處理我們的驗證. 這對於Angular來說是很好的, 在老式的不支持HTML5的瀏覽中, Angular將使用形式相同職責的指令.

然後我們可以經由改變引用它的形式來添加一個控制器處理表單的提交.
```html
	<form name='addUserForm' ng-controller="AddUserController">
```
在控制器裡面, 我們可以經由一個稱為`$valid`的屬性來訪問這個表單的驗證狀態. 當所有的表單input經由驗證的時候, Angular將設定它($valid)為true. 我們可以使用`$valid`屬性做一些時髦的事情, 比如當表單還沒有完成時禁用提交按鈕.

我們可以防止表單提交進入無效狀態, 經由給提交按鈕添加一個`ng-disabled`.
```html
	<button ng-disabled='!addUserForm.$valid'>Submit</button>
```
最後, 我們可能希望控制器告訴用戶她已經添加成功了. 我們的最終樣板看起來像這樣:
```html
	<h1>Sign Up</h1>
	<form name='addUserForm' ng-controller="AddUserController">
		<div ng-show='message'>{{message}}</div>
		<div>First name: <input name='firstName' ng-model='user.first' required></div>
		<div>Last name: <input ng-model='user.last' required></div>
		<div>Email: <input type='email' ng-model='user.email' required></div>
		<div>Age: <input type='number' ng-model='user.age' ng-maxlength='3'
		ng-min='1'></div>
		<div><button ng-click='addUser()'
		ng-disabled='!addUserForm.$valid'>Submit</button>
	</form>
```
接下來是控制器:
```js
	function AddUserController($scope) {
		$scope.message = '';

		$scope.addUser = function () {
			// TODO for the reader: actually save user to database...
			$scope.message = 'Thanks, ' + $scope.user.first + ', we added you!';
		};
	}
```
##小結

在前兩章中, 我們看到了Angular中所有最常用的功能(特性). 對每個功能的討論, 許多額外的細節訊息都沒有覆蓋到. 在下一章, 我們將讓你經由研究一個典型的工作流程瞭解更多的訊息.
