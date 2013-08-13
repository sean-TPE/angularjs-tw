#與伺服器通信

到現在為止，我們已經看到下面這些內容的主要部分，這些內容是你的Angular應用如何規劃設計、不同的AngularJS部件如何裝配在一起並正常工作以及AngularJS中的樣板代碼運行機制的一小部分內容。把這些結合在一起，你就可以搭建一些苗條性感的Web應用，但他們的運作主要還是限制在客戶端.在前面第二章,我們看了一點用`$http`服務做伺服器端通信的內容,但是在這一章,我們將會深入探討一下如何在現實世界的應用中使用它(`$http`)

在這一章，我們將討論一下AngularJS如何幫你同伺服器端通信，這其中包括在最底抽像等級的層面或者用它提供的優雅的封裝器。而且我們將會深入探討AngularJS如何用內建快取機制來幫你加速你的應用.如果你想用`SocketIO`開發一個實時的Angular應用,那麼第八章有一個例子，演示了如何把·SocketIO·封裝成一個指令然後如何使用這個指令，在這一章，我們就不涉及這方面內容了.

##經由$http進行通行

從Ajax應用(使用XMLHttpRequests)發動一個請求到伺服器的傳統方式包括：得到一個XMLHttpRequest目標的引用、發起請求、讀取回應、檢驗錯誤代碼然後最後處理伺服器回應。它就是下面這樣：
    
    var xmlhttp = new XMLHttpRequest();
    xmlhttp.onreadystatechange = function() {
        if (xmlhttp.readystate == 4 && xmlhttp.status == 200) {
            var response = xmlhttp.responseText;
        } else if (xmlhttp.status == 400) { // or really anything in the 4 series
            // Handle error gracefully
        }
    };
    // Setup connection
    xmlhttp.open(「GET」, 「http://myserver/api」, true);
    // Make the request
    xmlhttp.send();
    
對於這樣一個簡單、常用且經常重複的任務，上面這個代碼量比較大.如果你想重複性地做這件事,你最終可能會做一個封裝或者使用現成的函式庫.

AngularJS XHR(XMLHttpRequest) API遵循Promise接口.因為XHRs是非同步方法呼叫，伺服器回應將會在未來一個不定的時間返回(當然希望是越快越好).Promise接口保證了這樣的回應將會如何處理,它允許Promise接口的消費者以一種可預計的方式使用這些回應.

假設我們想從我們的伺服器取回用戶的訊息.如果接口在/api/user地址可用,並且接受id作為url參數，那麼我們的XHR請求就可以像下面這樣使用Angular的核心$http服務:
    
    $http.get('api/user', {params: {id: '5'}
    }).success(function(data, status, headers, config) {
    // Do something successful.
    }).error(function(data, status, headers, config) {
    // Handle the error
    });
    
如果你來自jQuery世界,你可能會注意到：AngularJS和jQuery處理非同步需求的方式很相似.

我們上面例子中使用的$htttp.get方法僅僅是AngularJS核心服務$http提供的眾多簡便方法之一.類似的，如果你想使用AngularJS向相同URL帶一些POST請求資料發起一個POST請求，你可以像下面這樣做：

    var postData = {text: 'long blob of text'};
    // The next line gets appended to the URL as params
    // so it would become a post request to /api/user?id=5 
    var config = {params: {id: '5'}};
    $http.post('api/user', postData, config
    ).success(function(data, status, headers, config) {
    // Do something successful
    }).error(function(data, status, headers, config) {
    // Handle the error
    });

AngularJS為大多數常用請求類型都提供了類似的簡便方法，他們包括：

+ GET
+ HEAD
+ POST
+ DELETE
+ PUT
+ JSONP

###進一步配置你的請求

有時，工具箱提供的標準請求配置還不夠,它可能是因為你想做下面這些事情:

+ 你可能想為請求添加權限驗證的頭訊息
+ 改變請求資料的快取方式
+ 在請求被發送或者回應返回時，對資料以一些方式做一定的轉換處理

在上面這樣的情況之下,你可以進一步配置請求，經由可選的傳遞進請求的配置對像.在之前的例子中,我們使用配置對像來標明可選的URL參數，即便我們哪兒演示的GET和POST方法是簡便方法。內部的原生方法可能看上面像相面這樣：

    $http(config)

下面演示的是一個呼叫這個方法的偽代碼樣板:

    $http({
        method: string,
        url: string,
        params: object,
        data: string or object,
        headers: object,
        transformRequest: function transform(data, headersGetter) or an array of functions,
        transformResponse: function transform(data, headersGetter) or an array of functions,
        cache: boolean or Cache object,
        timeout: number,
        withCredentials: boolean
    });

GET、POST和其它的簡便方法已經設定了請求的method類型,所以不需要再設定這個，config配置目標是傳給·$http.get·、·$http.post·方法的最後一個參數,所以當你使用任何簡便方法的時候，你任何能用這個config配置對像.

你也可以經由傳入含有下面這些鍵的屬性集config對像來改變已有的request對像

+ method : 一個表示http請求類型的字符串，比如GET,或者POST
+ url : 一個URL字符串代表要請求資源的絕對或相對URL
+ params : 一個對像(準確的說是鍵值映射)包含字符串到字符串內容，它代表了將會轉換為URL參數的鍵值對，比如下面這樣：
    [{key1: 'value1', key2: 'value2'}]
它將會被轉換為:
    ?key1=value&key2=value2
這串字符將會加在URL後面，如果在value的位置你用一個對像取代字符串或數字，那這個目標將會轉換為JSON字符串.
+ data ：一個字符串或一個目標，它將會被作為請求消息資料被發送.
+ timeout : 這是請求被認定為過期之前所要等待的毫秒數.

還有部分另外的選項可以被配置,在下面的章節中，我們將會深度探索這些選項.

###設定HTTP頭訊息(Headers)

AngularJS有一個默認的頭訊息,這個頭訊息將會對所有的發送請求使用,它包含以下訊息:
    1.Accept: application/json, text/plain, /
    2.X-Requested-With:XMLHttpRequest

如果你想設定任何特定的頭訊息,這兒有兩種方法來做這件事：

第一種方法,如果你相對所有的發送請求都使用這些特定頭訊息,那你需要把特定有訊息設定為Angular默認頭訊息的一部分.可以在`$httpProvider.defaults.headers`配置對像裡面設定這個,這個步驟通常會在你的app設定config部分來做.所以如果你想移除"Requested-With"頭訊息且對所有的GET請求啟用"DO NOT TRACK"設定,你可以簡單地經由以下代碼來做:

    angular.module('MyApp',[]).
        config(function($httpProvider) {
            // Remove the default AngularJS X-Request-With header
            delete $httpProvider.default.headers.common['X-Requested-With'];
            // Set DO NOT TRACK for all Get requests
            $httpProvider.default.headers.get['DNT'] = '1';
    });
    
如果你只想對某個特定的請求設定頭訊息,而不是設定默認頭訊息.那麼你可以經由給$http服務傳遞包含指定頭訊息的config對像來做.相同的客製化頭訊息可以作為第二個參數傳遞給GET請求,第一個參數是URL字符串：
    
    $http.get('api/user', {
    // Set the Authorization header. In an actual app, you would get the auth
    // token from a service
    headers: {'Authorization': 'Basic Qzsda231231'},
    params: {id: 5}
    }).success(function() { // Handle success });
    
如何在應用中處理權限驗證頭訊息的成熟範例將會在第八章的Cheetsheets範例部分給出.

###快取回應資料

AngularJS為HTTP GET請求提供了一個開箱即用的簡單快取系統.缺省情況下,它對所有的請求都是禁用的,但是如果你想對你的請求啟用快取系統，你可以使用以下代碼:

    $http.get('http://server/myapi', {
        cache: true
    }).success(function() { // Handle success });

這段代碼啟用了快取系統，然後AngularJS將會快取來自Server的回應資料.但對相同的URL的請求第二次發出時,AngularJS將會從快取裡面取出前一次的回應資料作為回應返回.這個快取系統也很智能,即使你同時對相同URL發出多個請求,只有一個請求會發向Server,這個請求的回應資料將會反饋給所有(同時發起的)請求。

然而這種做法從可用性的角度看可能是有所衝突的,當一個用戶首先看到舊的結果,然後新的結果突然冒出來，比如一個用戶可能即將單擊一個資料項,而實際上這個資料項後台已經發生了變化.

注意所有回應(即使是從快取裡取出的)本質上仍舊是非同步回應.換句話說，期望你的利用快取回應時的非同步代碼運行仍舊和他向後台伺服器發出請求時的代碼運行機制是一樣的.

###對請求(Request)和回應(Response)的資料所做的轉換

AngularJS對所有`$http`服務發起的請求和回應做一些基本的轉換,它們包括:

+ 請求(Request)轉換:
    如果請求的Cofig配置目標的data屬性包含一個目標，將會把這個目標序列化為JSON格式.
+ 回應(Response)轉換:
    如果探測到一個XSRF頭,把它剝離掉.如果回應資料被探測為JSON格式,用JSON解析器把它反序列化為JSON對像.

如果你需要部分系統默認提供的轉換,或者想使用你自己的轉換,你可以把你的轉換函數作為Config配置目標的一部分傳遞進去(後面有細述).這些轉換函數得到HTTP請求和HTTP回應的資料主體以及它們的頭訊息.然後把序列化的修改後版本返回出來.在Config對像裡面配置這些函數需要使用·transformRequest·鍵和·transformResponse·鍵,這些都可以經由使用`$httpProvider·服務在模組的config函數里面配置它.

我們什麼時候使用這些哪?讓我假設我們有一個伺服器，它更習慣於jQuery運行的方式.它可能希望我們的POST資料以`key1=val1&key2=val2`字符串的形式傳遞，而不是以`{key1:val1,key2:val2}`這樣的JSON格式.這個時候，我們可能相對每個請求做這樣的轉換,或者單個地增加transformRequest轉換函數,為了達成這個範例這樣的目標,我們將要設定一個通用transformRequet轉換函數,以便對所有的發出請求,這個函數都可以把JSON格式轉換為鍵值對字符串,下面代碼演示了如何做這個工作:

    var module = angular.module('myApp');
    module.config(function ($httpProvider) {
        $httpProvider.defaults.transformRequest = function(data) {
            // We are using jQuery』s param method to convert our
            // JSON data into the string form
            return $.param(data);
       };
    });

##單元測試

目前為止，我們已經瞭解如何使用`$http`服務以及如何以可能的方式做你需要的配置.但是如何寫一些單元測試來保證這些夠真實有效的運行哪?

正如我們曾經三番五次的提到的那樣,AngularJS一直以測試為先的原則而設計.所以Angualr有一個模擬伺服器後端，在單元測試中,它可以幫你就可以測試你發出的請求是否正確,甚至可以精確控制模擬回應如何得到處理,什麼時候得到處理.

讓我們探索一下下面這樣的單元測試場景:一個控制向伺服器發起請求,從伺服器得到資料,把它賦給作用域內的模型,然後以具體的樣板格式顯示出來.

我們的`NameListCtrl`控制器是一個非常簡單的控制器.它的存在只有一個目的：訪問`names`API接口，然後把得到資料存儲在作用域scope模型內.

    function NamesListCtrl($scope, $http) {
        $http.get('http://server/names', {params: {filter: 『none』}}).
            success(function(data) {
                $scope.names = data;
        });
    }

怎樣對這個控制器做單元測試？在我們的單元測試中,我們必須保證下面這些條件:

+ `NamesListCtrl`能夠找到所有的依賴項(並且正確的得到注入的這些依賴)》
+ 當控制器加載時盡可能快地立刻發情請求從伺服器得到names資料.
+ 控制器能夠正確地把回應資料存儲到作用域scope的`names`變數屬性中.

在我們的單元測試中構造一個控制器時，我們給它注入一個scope作用域和一個偽造的HTTP服務,在構建測試控制器的方式和生產中構建控制器的方式其實是一樣的.這是推薦方法，儘管它看上去上有點複雜。讓我看一下具體代碼：
    
    describe('NamesListCtrl', function(){
        var scope, ctrl, mockBackend;

        // AngularJS is responsible for injecting these in tests
        beforeEach(inject(function(_$httpBackend_, $rootScope, $controller) {
            // This is a fake backend, so that you can control the requests
            // and responses from the server
            mockBackend = _$httpBackend_;

            // We set an expectation before creating our controller,
            // because this call will get triggered when the controller is created
            mockBackend.expectGET('http://server/names?filter=none').
                respond(['Brad', 'Shyam']);
            scope = $rootScope.$new();

            // Create a controller the same way AngularJS would in production
            ctrl = $controller(PhoneListCtrl, {$scope: scope});
        }));

        it('should fetch names from server on load', function() {
            // Initially, the request has not returned a response
            expect(scope.names).toBeUndefined();
            
            // Tell the fake backend to return responses to all current requests
            // that are in flight.
            mockBackend.flush();
            // Now names should be set on the scope
            expect(scope.names).toEqual(['Brad', 'Shyam』]);
        });
    });

##使用RESTful資源

·$http·服務提供一個比較底層的實現來幫你發起XHR請求,但是同時也給提供了很強的可控性和彈性.在大多數情況下,我們處理的是對像集或者是封裝有一定屬性和方法的目標模型,比如帶有個人資料的自然人對像或者信用卡對像.

在上面這樣的情況下，如果我們自己構建一個JS對像來表示這種較複雜對像模型，那做法就有點不夠nice.如果我們僅僅想編輯某個目標的屬性、保存或者更新一個目標，那我們如何讓這些狀態在伺服器端持久化.

`$resource`正好給你提供這種能力.AngularJS resources可以幫助我們以描述的方式來定義對像模型，可以定義一下這些特徵：

+ resource的伺服器端URL
+ 這種請求常用參數的類型
+ (你可以免費自動得到get、save、query、remove和delete方法),除了那些方法，你可以定義其它的方法，這些方法封裝了目標模型的特定功能和業務邏輯(比如信用卡模型的charge()付費方法)
+ 回應的期望類型(數組或者一個獨立對像)
+ 頭訊息

------------------------------------------------------
什麼時候你可以用Angular Resources組件？

只有你的伺服器端設施是以RESTful方式提供服務的時候，你才應該用Angular resources組件.比如信用卡那個案例,我們將用它作為本章這一部分的例子，他將包括以下內容:

1. 給地址`/user/123/card`發送一個GET請求，將會得到用戶123的信用卡列表.
2. 給地址`/user/123/card/15`發送一個GET請求,將會得到用戶123本人的ID為15的信用卡訊息
3. 給地址`/user/123/card`發送一個在POST請求資料部分包含信用卡訊息的POST請求,將會為用戶123新建立一個信用卡
4. 給地址`/user/123/card/15`發送一個在POST請求資料部分包含信用卡訊息的POST請求,將會更新用戶123的ID為5的信用卡的訊息.
5. 給地址`/user/123/card/15`一個方法為DELETE類型的請求,將會刪除掉用戶123的ID為5的信用卡的資料.

-------------------------------------------------------

除了按照你的要求給你提供一個查詢伺服器端訊息的目標,`$resource`還可以讓你使用返回的資料目標就像他們是持久化資料模型一樣，可以修改他們，還可以把你的修改持久化存儲下來.

`ngResource`是一個單獨的、可選的模組.要想使用它，你看需要做以下事情：

+ 在你的HTML文件裡面引用angular-resource.js的實際地址
+ 在你的模組依賴裡面聲明對它的依賴(例如,angular.module('myModule',['ngResource'])).
+ 在需要的地方，注入$resource這個依賴項.

在我們看怎樣用ngResource方法建立一個resource資源之前，我們先看一下怎樣用基本的$http服務做類似的事情.比如我們的信用卡resource，我們想能夠讀取、查詢、保存信用卡訊息，另外還要能為信用卡還款.

這兒是上述需求一個可能的實現：

    myAppModule.factory('CreditCard', ['$http', function($http) {
        var baseUrl = '/user/123/card';
        return {
            get: function(cardId) {
                return $http.get(baseUrl + '/' + cardId);
            },
            save: function(card) {
                var url = card.id ? baseUrl + '/' + card.id : baseUrl;
                return $http.post(url, card);
            },
            query: function() {
                return $http.get(baseUrl);
            },
            charge: function(card) {
                return $http.post(baseUrl + '/' + card.id, card, {params: {charge: true}});
            }
        };
    }]);

取代以上方式，你也可以輕鬆建立一個在你的應用中始終如一的Angular資源服務，就像下面代碼這樣：
    
    myAppModule.factory('CreditCard', ['$resource', function($resource) {
        return $resource('/user/:userId/card/:cardId',
            {userId: 123, cardId: '@id'},
            {charge: {method:'POST', params:{charge:true}, isArray:false});
    }]);

做到現在，你就可以任何時候從Angular注入器裡面請求一個CreditCard依賴，你就會得到一個Angular資源,默認情況下，這個資源會提供一些初始的可用方法.表格5-1列出了這些初始方法以及他們的運行行為，這樣你就可以知道在伺服器怎樣配置來配合這些方法了.

表格5-1 一個信用卡reource
<table>
<thead>
 <tr>
   <th>Resource Function</th>
   <th>Method</th>
   <th>URL</th>
   <th>Expected Return</th>
 </tr>
</thead>
<tbody>
  <tr>
    <td>CreditCard.get({id: 11})</td>
    <td>GET</td>
    <td>/user/123/card/11</td>
    <td>Single JSON</td>
  </tr>
  <tr>
    <td>CreditCard.save({}, ccard)</td>
    <td>POST</td>
    <td>/user/123/card with post data 「ccard」</td>
    <td>Single JSON</td>
  </tr>
  <tr>
    <td>CreditCard.save({id: 11}, ccard)</td>
    <td>POST</td>
    <td>/user/123/card/11 with post data 「ccard」</td>
    <td>Single JSON</td>
  </tr>
  <tr>
    <td>CreditCard.query()</td>
    <td>GET</td>
    <td>/user/123/card</td>
    <td>JSON Array</td>
  </tr>
  <tr>
    <td>CreditCard.remove({id: 11})</td>
    <td>DELETE</td>
    <td>/user/123/card/11</td>
    <td>Single JSON</td>
  </tr>
  <tr>
    <td>CreditCard.delete({id: 11})</td>
    <td>DELETE</td>
    <td>/user/123/card/11</td>
    <td>Single JSON</td>
  </tr>
</tbody>
</table>

讓我們看一個信用卡resource使用的代碼樣例，這樣可以讓你理解起來覺得更淺顯易懂.

    // Let us assume that the CreditCard service is injected here
    // We can retrieve a collection from the server which makes the request
    // GET: /user/123/card
    var cards = CreditCard.query();
    // We can get a single card, and work with it from the callback as well
    CreditCard.get({cardId: 456}, function(card) {
        // each item is an instance of CreditCard
        expect(card instanceof CreditCard).toEqual(true);
        card.name = "J. Smith";
        // non-GET methods are mapped onto the instances
        card.$save();
        // our custom method is mapped as well.
        card.$charge({amount:9.99});
        // Makes a POST: /user/123/card/456?amount=9.99&charge=true
        // with data {id:456, number:'1234', name:'J. Smith'}
    });

前面這個樣例代碼裡面發生了很多事情，所以我們將會一個一個地認真講解其中的重要部分:

###resource資源的聲明

聲明你自己的`$resource`非常簡單，只要呼叫注入的$resource函數,並給他傳入正確的參數即可。(你現在應該已經知道如何注入依賴,對吧?)

$resource函數只有一個必須參數,就是提供後台資源資料的URL地址,另外還有兩個可選參數:默認request參數訊息和其它的想在資源上要配置的動作.

請注意：第一個URL地址參數中的的變數資料是參數化可配置的(:冒號是參數變數的語法符號,比如`:userId`以為這個參數將會被實際的userId參數變數取代(譯者注:寫過參數化SQL語句的人應該很熟悉),而`:cardId`將會被cardId參數變數的值所取代),如果沒有給函數傳遞這些參數變數,那那個位置將會被空字符取代.

函數的第二個參數則負責提供所有請求的默認參數變數訊息.在這個案例中，我們給userId參數傳遞一個常量:123,cardId參數則更有意思,我們給cardId參數傳遞了一個"@id"字符串.這意味著如果我們使用一個從伺服器返回的目標而且我們可以呼叫這個目標的任何方法(比如$save),那麼這個目標的id屬性將會被取出來賦給cardId字段.

函數的第三個參數是一些我們想要暴露的其它方法，這些方法是對客製化資源做操作的方法.在下面的章節，我們將會深度討論這個話題

###客製化方法

$resource函數的第三個參數是可選的，主要用來傳遞要在resource資源上暴露的其它自定義方法。

在這個案例中，我們自定義了一個方法charge.這個自定義方法可以經由傳遞一個對像而被配置上.這個目標裡有個鍵值，表明了此方法的暴露名稱.這個配置需要頂一個request請求的方法類型(GET,POST等等)，以及該請求中需要的參數也要被傳遞(比如charge=true),並且聲明返回目標是數組還是單個普通目標。這一切到搞定之後，你就可以在有這個業務實際需要求的時候，自由地呼叫`CreditCard.charge()`方法.

###不要使用回呼函數機制!(除非你真的需要它們)

第三個需要注意的事情是資源呼叫的返回類型.讓我們再次關注一下`CreditCard.query()`這個函數.你將會看到不是在回呼函數中給cards賦值,而是直接把它賦給card變數.在非同步伺服器請求的情況下唉，這樣的代碼是如何運作的哪?

你擔心代碼是否正常工作是對的，但是代碼實際上是可以正常工作的.這裡實際發生的是AngularJS賦值了一個引用(是普通目標的還是數組的取決於你期望的返回類型)，這個引用將會在未來伺服器請求回應返回時被填充.在這期間，這個引用是個空應用.

因為在AngularJS應用中的大多數通用過程都是從伺服器端取資料，把它賦給一個變數，然後在模版上顯示它,而上面這樣的簡化機制非常優雅.在你的控制器代碼中,你所有需要去做的就是發出伺服器端請求,把返回值賦給正確的作用域(scope)變數.然後剩下的合適渲染這些資料就由樣板系統去操心了.

如果你想對返回值做一些業務邏輯處理，拿著匯總方法就不能奏效了.在這種情況下，你就得依賴回呼函數機制了，比如在Credit.get()呼叫中使用的那種機制.

###簡化的伺服器端操作

無論你使用資源簡化函數機制還是回呼函式，關於返回目標都有幾點問題需要我們注意.

返回的目標不是一個普通JS目標，實際上，他是「resource」類型的目標.這就意味著對像裡除了包含伺服器回傳的資料以外,還有一些附加的行為函數(在這個案例中如$save()和$charge函數).這樣我們就可以很方便的執行服務器端操作,比如取資料、修改資料並把修改在伺服器端序列化儲存下來(其實也就是一般CURD應用裡面的一般操作).

###對ngResource做單元測試

ngResource依賴項是一個封裝,它以Angular核心服務`$http`為基礎.因此，你可能已經知道如何對它做單元測試了.它和我們看到的對`$http`做單元測試的範例比起來基本沒什麼真正的變化.你只需要知道最終的伺服器端請求應該由resource發起,告訴模擬`$http`服務關於要求的訊息.其他的基本都一樣.下面我們來看看如何本節測試前面的代碼:

    describe('Credit Card Resource', function(){
        var scope, ctrl, mockBackend;
        beforeEach(inject(function(_$httpBackend_, $rootScope, $controller) {
            mockBackend = _$httpBackend_;
            scope = $rootScope.$new();
            // Assume that CreditCard resource is used by the controller
            ctrl = $controller(CreditCardCtrl, {$scope: scope});
        }));
        
        it('should fetched list of credit cards', function() {
            // Set expectation for CreditCard.query() call
            mockBackend.expectGET('/user/123/card').
                respond([{id: '234', number: '11112222'}]);
            
            ctrl.fetchAllCards();

            // Initially, the request has not returned a response
            expect(scope.cards).toBeUndefined();

            // Tell the fake backend to return responses to all current requests
            // that are in flight.
            mockBackend.flush();

            // Now cards should be set on the scope
            expect(scope.cards).toEqualData([{id: '234', number: '11112222'}]);
        });
    });

這個測試看上去和簡單的`$http`單元測試非常相似,除了一些細微區別.注意在我們的expect語句裡面,取代了簡單的"equals"方法,哦我們用的是特殊的"toEqualData"方法.這種expect語句會智慧地省略ngResource新增到目標上的附加方法.

##`$q`和預期(Promise)


