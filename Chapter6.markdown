#指令

對於指令, 你可以擴充HTML來以添加聲明性語法來做任何你喜歡做的事情. 經由這樣做, 你可以替換一些特定於你的應用程式的通用的\<div\>s和\<span\>s元素和屬性的實際意義. 它們都帶有Angular提供的基礎功能, 但是你可以建立特定於應用程式的你自己想做的事情.

首先我們要複習以下指令API以及它在Angular啟動和運行生命週期裡是如何運作的. 從那裡, 我們將使用這些只是來建立一個指令類型. 在本將完成時我們將學習到如何編寫指令的單元測試和使它們運行得更快.

但是首先, 我們來看看一些使用指令的語法說明.

##指令和HTML驗證

在本書中, 我們已經使用了Angular內置指令的`ng-directive-name`語法. 例如`ng-repeat`, `ng-view`和`ng-controller`. 這裡, `ng`部分是Angular的命名空間, 並且dash之後的部分便是指令的名稱.

雖然我們喜歡這個方便輸入的語法, 但是在大部分的HTML驗證機制中它不是有效的. 為了支持這些, Angular指令允許你以幾種方式呼叫任意的指令. 以下在表6-1中列出的語法, 都是等價的並能夠讓你偏愛的[首選的]驗證器正常工作

Table 6-1 HTML Validation Schemes

<table>
	<thead>
		<tr>
			<th>Validator</th>
			<th>Format</th>
			<th>Example</th>
		</tr>
	</thead>
	<tbody>
		<tr>
			<td>none</td>
			<td>namespace-name</td>
			<td>ng-repeat=<i>item in items</i></td>
		</tr>
		<tr>
			<td>XML</td>
			<td>namespace:name</td>
			<td>ng:repeat=<i>item in items</i></td>
		</tr>
		<tr>
			<td>HTML5</td>
			<td>data-namespace-name</td>
			<td>data-ng-repeat=<i>item in items</i></td>
		</tr>
		<tr>
			<td>xHTML</td>
			<td>x-namespace-name</td>
			<td>x-ng-repeat=<i>item in items</i></td>
		</tr>
	</tbody>
</table>

由於你可以使用任意的這些形式, [AngularJS文件](http://docs.angularjs.org/)中列出了一個駝峰式的指令, 而不是任何這些選項. 例如, 在`ngRepeat`標題下你可以找到`ng-repeat`. 稍後你會看到, 在你定義你自己的指令時你將會使用這種命名格式.

如果你不適用HTML驗證器(大多數人都不使用), 你可以很好的使用在目前你所見過的例子中的命名空間-指令[namespace-directive]語法

##API預覽

下面是一個建立任意指令偽代碼樣板

	var myModule = angular.module(...);

	myModule.directive('namespaceDirectiveName', function factory(injectables) {
		var directiveDefinitionObject = {
			restrict: string,
			priority: number,
			template: string,
			templateUrl: string,
			replace: bool,
			transclude: bool,
			scope: bool or object,
			controller: function controllerConstructor($scope, $element, $attrs, $transclude){...},
			require: string,
			link: function postLink(scope, iElement, iAttrs) {...},
			compile: function compile(tElement, tAttrs, transclude){
				return: {
					pre: function preLink(scope, iElement, iAttrs, controller){...},
					post: function postLink(scope, iElement, iAttrs, controller){...}
				}
			}
		};
		return directiveDefinitionObject;
	});

有些選項是互相排斥的, 它們大多數都是可選的, 並且它們都有有價值的詳細說明:

當你使用每個選項時, 表6-2提供了一個概述.

Table 6-2 指令定義選項

<table>
	<thead>
		<tr>
			<th>Property</th>
			<th>Purpose</th>
		</tr>
	</thead>
	<tbody>
		<tr>
			<td>restrict</td>
			<td>聲明指令可以作為一個元素, 屬性, 類, 註釋或者任意的組合如何用於樣板中</td>
		</tr>
		<tr>
			<td>priority</td>
			<td>設定樣板中相對於其他元素上指令的執行順序</td>
		</tr>
		<tr>
			<td>template</td>
			<td>指令一個作為字符串的內聯樣板. 如果你指定一個樣板URL就不要使用這個樣板屬性.</td>
		</tr>
		<tr>
			<td>templateUrl</td>
			<td>指定經由URL加載的樣板. 如果你指定了字符串的內聯樣板就不需要使用這個.</td>
		</tr>
		<tr>
			<td>replace</td>
			<td>如果為true, 則替換當前元素. 如果為false或者未指定, 則將這個指令追加到當前元素上.</td>
		</tr>
		<tr>
			<td>transclude</td>
			<td>讓你將一個指令的原始自節點移動到心樣板位置內.</td>
		</tr>
		<tr>
			<td>scope</td>
			<td>為這個指令建立一個新的作用域而不是繼承父作用域.</td>
		</tr>
		<tr>
			<td>controller</td>
			<td>為跨指令通信建立一個發佈的API.</td>
		</tr>
		<tr>
			<td>require</td>
			<td>需要其他指令服務於這個指令來正確的發揮作用.</td>
		</tr>
		<tr>
			<td>link</td>
			<td>以編程的方式修改產生的DOM元素實例, 添加事件監聽器, 設定資料繫結.</td>
		</tr>
		<tr>
			<td>compile</td>
			<td>以編程的方式修改一個指令的DOM樣板的副本特性, 如同使用`ng-repeat`時. 你的編譯函數也可以返回鏈接函數來修改產生元素的實例.</td>
		</tr>
	</tbody>
</table>

下面讓我們深入細節來看看.

###為你的指令命名

你可以用模組的指令函數為你的指令建立一個名稱, 如下所示:

	myModule.directive('directiveName', function factory(injectables){...});

雖然你可以使用任何你喜歡的名字命名你的指令, 該符號會選擇一個前綴命名空間標識你的指令, 同時避免與可能包含在你的項目中的外部指令衝突.

你當然不希望它們使用一個`ng-`前綴, 因為這可能與Angular自帶的指令相衝突. 如果你從事於SuperDuper MegaCorp, 你可以選擇一個super-, superduper-, 或者甚至是superduper-megacorp-, 雖然你可能選擇第一個選項, 只是為了方便輸入.

正如前面所描述的, Angular使用一個標準化的指令命名機制, 並且試圖有效的在樣板中使用駝峰式的指令命名方式來確保在5個不同的友好的驗證器中正常工作. 例如, 如果你已經選擇了`super-`作為你的前綴, 並且你在編寫一個日期選擇(datepicker)組件, 你可能將它命名為`superDatePicker`. 在樣板中, 你可以像這樣來使用它: `super-date-picker`, `super:date-picker`, `data-super-date-picker`或者其他多樣的形式.

###指令定義對像

正如前面提到的, 在指令定義中大多數的選項都是可選的. 實際上, 這裡並沒有硬性的要求必須選擇哪些選項, 並且你可以構造出許多有利於指令的子集參數. 讓我們來逐步討論這些選項是做什麼的.

####restrict

`restrict`屬性允許你指定你的指令聲明風格--也就是說, 它是否能夠用於作為元素名稱, 屬性, 類[className], 或者註釋. 你可以根據表6-3來指定一個或多個聲明風格, 只需要使用一個字符串來表示其中的每一中風格:

Table 6-3 指令聲明用法選項

<table>
	<thead>
		<tr>
			<th>Character</th>
			<th>Declaration style</th>
			<th>Example</th>
		</tr>
	</thead>
	<tbody>
		<tr>
			<td>E</td>
			<td>element</td>
			<td>&lt;my-menu title=<i>Products</i>&gt;&lt;/my-menu&gt;</td>
		</tr>
		<tr>
			<td>A</td>
			<td>attribute</td>
			<td>&lt;div my-menu=<i>Products</i>&gt;&lt;/div&gt;</td>
		</tr>
		<tr>
			<td>C</td>
			<td>class</td>
			<td>&lt;div class=my-menu:<i>Products</i>&gt;&lt;/div&gt;</td>
		</tr>
		<tr>
			<td>M</td>
			<td>comment</td>
			<td>&lt;!--directive:my-menu Products--&gt;</td>
		</tr>
	</tbody>
</table>

如果你希望你的指令用作一個元素或者一個屬性, 那麼你應該傳遞`EA`作為`restrict`字符串.

如果你忽略了`restrict`屬性, 則默認為`A`, 並且你的指令只能用作一個屬性(屬性指令).

如果你計劃支持IE8, 那麼基於attribute-和class-的指令就是你最好的選擇, 因為它需要額外的努力來使新元素正常工作. 可以查看Angular文件來詳細瞭解這一點.

####Priorities

在你有多個指令綁定在一個單獨的DOM元素並要確定它們的應用順序的情況下, 你可以使用`priority`屬性來指定應用的順序. 數值高的首先運行. 如果你沒有指定, 則默認的priority為0.

很難發生需要設定優先級的情況. 一個需要設定優先級例子是`ng-repeat`指令. 當重複元素時, 我們希望Angular在應用指令之前床在一個樣板元素的副本. 如果不這麼做, 其他的指令將會應用到標準的樣板元素上而不是我們所希望在應用程式中重複我們的元素.

雖然它(proority)不在文件中, 但是你可以搜尋Angular資源中少數幾個使用`priority`的其他指令. 對於`ng-repeat`, 我們使用優先級值為1000, 這樣就有足夠的優先級處理優先處理它.

####Templates

當建立組件, 掛件, 控制器一起其他東西時, Angular允許你提供一個樣板替換或者包裹元素的內容. 例如, 如果你在視圖中建立一組tab選項卡, 可能會呈現出如圖6-1所示視圖.

![tab](figure/tab.png)

圖6-1 tab選項卡視圖

並不是一堆\<div\>, \<ul\>\<li\>和\<a\>元素, 你可以建立一個\<tab-set\>和\<tab\>指令, 用來聲明每個單獨的tab選項卡的結構. 然後你的HTML可以做的更好來表達你的樣板意圖. 最終結果可能看起來像這樣:

	<tab-set>
		<tab title="Home">
			<p>Welcome home!</p>
		</tab>
		<tab title="Preferences">
			<!-- preferences UI goes here -->
		</tab>
	</tab-set>

你還可以給title綁定一個字符串資料, 經由在\<tab\>或者\<tab-set\>上綁定控制器處理tab選項內容. 它不僅限於用在tabs上--你還可以用於選單, 手風琴, 彈窗, dialog對話框或者其他任何你希望以這種方式實現的地方.

你可以經由`template`或者`templateUrl`屬性來指定替換的DOM元素. 使用`template`經由字符串來設定樣板內容, 或者使用`templateUrl`來從伺服器的一個文件上來加載樣板. 正如你在接下來的例子中會看到, 你可以預先快取這些樣板來減少GET請求, 這有利於提高應用的性能.

讓我們來編寫一個dumb指令: 一個\<hello\>元素, 只是用於使用\<div\>Hi there\</div\>來替換自身. 在這裡, 我們將設定`restrict`來允許元素和設定`template`顯示我們所希望的東西. 由於默認的行為只將內容追加到元素中, 因此我們將設定`replace`屬性為true來替換原來的樣板:

	var appModule = angular.module('app', []);
	appModule.directive('hello', function(){
		return {
			restrict: 'E',
			template: '<div>Hi there</div>',
			replace: true
		};
	});

在頁面中我們可以像這樣使用它:

	<html lang="en" ng-app="app">
	...
	<body>
		<hello></hello>
	</body>
	...

將它載入到瀏覽器中, 我們會看到"Hi there".

如果你查看頁面的原始碼, 在頁面上你仍然會看到\<hello\>\</hello\>, 但是如果你查看產生的原始碼(在Chrome中, 你可以在"Hi there"上右擊然後選擇審查元素), 你會看到:

	<body>
		<div>Hi there</div>
	</body>

\<hello\>\</hello\>被樣板中的\<div\>替換了.

如果你從指令定義中移除`replace: true`, 那麼你會看到\<hello\>\<div\>Hi there\</div\>\</hello\>.

通常你會希望使用`templateUrl`而不是`template`, 因為輸入HTML字符串並不是那麼有趣. `template`屬性通常有利於非常小的樣板. 使用templateUrl`同樣非常有用, 可以設定適當的頭來使樣板可快取. 我們可以像下面這樣重寫我們的`hello`指令:

	var appModule = angular.module('app', []);
	appModule.directive('hello', function(){
		return {
			restrict: 'E',
			templateUrl: 'helloTemplate.html',
			replace: true
		};
	});

在`helloTemplate.html`中, 你只需要輸入:

	<div>Hi there</div>

如果你使用Chrome瀏覽器, 它的"同源策略"會組織Chrome從`file://`中加載這些樣板, 並且你會得到一個類似"Origin null is not allowed by Access-Control-Allow-Origin."的錯誤. 那麼在這裡, 你有兩個選擇:

+ 經由伺服器來加載應用
+ 在Chrome中設定一個標誌. 你可以經由在命令行中使用`chrome --allow-file-access-from-files`命令來運行Chrome做到這一點.

這將會經由`templateUrl`加載這些文件, 然而, 這會讓你的用戶要等待到指令加載. 如果你希望在首頁加載樣板, 你可以在一個`script`標籤中將它作為這個頁面的一部分包含進來, 就像這樣:

	<script type="text/ng-template" id="helloTemplateInline.html">
		<div>Hi there</div>
	</script>

這裡的id屬性很重要, 因為這是Angular用來存儲樣板的URL鍵. 稍候你將會使用這個id在指令的`templateUrl`中指定要插入的樣板.

這個版本能夠很好的載入而不需要伺服器, 因為沒有必要的`XMLHttpRequest`來取得內容.

最後, 你可以越過`$http`或者以其他機制來加載你自己的樣板, 然後將它們直接設定在Angular中稱為`$templateCache`的對象上. 我們希望在指令運行之前快取中的這個樣板可用, 因此我們將經由module上的run函數來呼叫它.

	var appModule = angular.module('app', []);

	appModule.run(function($templateCache){
		$templateCache.put('helloTemplateCached.html', '<div>Hi there</div>');
	});

	appModule.directive('hello', function(){
		return {
			restrict: 'E',
			templateUrl: 'helloTemplateCached.html',
			replace: true;
		};
	});

你可能希望在產品中這麼做, 僅僅作為一個減少所需的GET請求數量的技術. 你可以運行一個腳本將所有的樣板合併到一個單獨的文件中, 並在一個新的模組中加載它, 然後你就可以從你的主應用程式模組中引用它.

####Transclusion

除了替換或者追加內容, 你還可以經由`transclude`屬性將原來的內容移到新樣板中. 當設定為true時, 指令將刪除原來的內容, 但是在你的樣板中經由一個名為`ng-transclude`的指令重新插入來使它可用. 

我們可以使用transclusion來改變我們的範例:

	appModule.directive('hello', function() {
		return {
			template: '<div>Hi there <span ng-transclude></span></div>',
			transclude: true
		};
	});

像這樣來應用它:

	<div hello>Bob</div>

你會看到: "Hi there Bob."

###編譯和鏈接功能

雖然插入樣板是有用的, 任何指令真正有趣的工作發生在它的`compile`和它的`link`函數中.

`compile`和`link`函數被指定為Angular用來建立應用程式實際視圖的後兩個階段. 讓我們從更高層次來看看Angular的初始化過程, 按一定的順序:

**Script loads**

Angular加載和查找`ng-app`指令來判定應用程式界限.

**Compile phase(階段)**

在這個階段, Angular會遍歷DOM節點以確定所有註冊在樣板中的指令. 對於每一個指令, 然後基於指令的規則(`template`,`replace`,`transclude`等等)轉換DOM, 並且如果它存在就呼叫`compile`函數. 它的返回結果是一個編譯過的`template`函數, 這將從所有的指令中呼叫`link`函數來收集.

**Link phase(階段)**

建立動態的視圖, 然後Angular會對每個指令運行一個`link`函數. `link`函數通常在DOM或者模型上建立監聽器. 這些監聽器用於視圖和模型在所有的時間裡都保持同步.

因此我們必須在編譯階段處理樣板的轉換, 同時在鏈接階段處理在視圖中修改資料. 按照這個思路, 指令中的`compile`和`link`函數之間主要的區別是`compile`函數處理樣板自身的轉換, 而`link`函數處理在模型和視圖之間創造一個動態的連接. 作用域掛接到編譯過的`link`函數正是在這個第二階段, 並且經由資料繫結將指令變成活動的.

出於性能的考慮, 者兩個階段才分開的. `compile`函數僅在編譯階段執行一次, 而`link`函數會被執行多次, 對每個指令實例. 例如, 讓我們來說說你上面使用的`ng-repeat`指令. 你並不想小勇`compile`, 這回導致在每次`ng-repeat`重複時都產生一個DOM遍歷的操作. 相反, 你會希望一次編譯, 然後鏈接.

雖然你毫無疑問的應該學習編譯和鏈接之間的不同, 以及每個功能, 你需要編寫的大部分的指令都不需要轉換樣板; 你還會編寫大部分的鏈接函數.

讓我們再看看每個語法來比較一下, 我們有:

	compile: function compile(tElement, tAttrs, transclude) {
		return {
			pre: function preLink(scope, iElement, iAttrs, controller) {...},
			post: function postLink(scope, iElement, iAttrs, controller) {...}
		}
	}

以及鏈接:

	link: function postLink(scope, iElement, iAttrs) {...}

注意這裡有一點不同的是`link`函數獲得了一個作用域的訪問, 而`compile`沒有. 這是因為在編譯階段期間, 作用域並不存在. 然而你有能力從`compile`函數返回`link`函數. 這些`link`函數能夠訪問到作用域.

還要注意的是`compile`和`link`都會獲得一個到它們對應的DOM袁術和這些元素屬性[attributes]列表的引用. 這裡的一點區別是`compile`函數是從樣板中獲得樣板元素和屬性, 並且會取得到`t`前綴. 而`link`函數使用樣板建立的視圖實例中獲得它們的, 它們會取得到`i`前綴.

這種區別只存在於當指令位於其他指令中製造樣板副本的時候. `ng-repeat`就是一個很好的例子.

	<div ng-repeat="thing in things">
		<my-widget config="thing"></my-widget>
	</div>

這裡, `compile`函數將只被呼叫一次, 而`link`函數在每次複製`my-widget`時都會被呼叫一次--等價於元素在things中的數量. 因此, 如果`my-widget`需要到所有`my-widget`副本(實例)中修改一些公共的東西, 為了提升效率, 正確的做法是在`compile`函數中處理. 

你可能還會注意到`compile`函數好哦的了一個`transclude`屬性函數. 這裡, 你還有機會以編寫一個函數以編程的方式transcludes內容, 對於簡單的的基於樣板不足以transclusion的情況.

最後, `compile`可以返回一個`preLink`和`postLink`函數, 而`link`僅僅指向一個`posyLink`函數. `preLink`, 正如它的名字所暗示的, 它運行在編譯階段之後, 但是會在指令鏈接到子元素之前. 同樣的, `postLink`會運行在所有的子元素指令被鏈接之後. 這意味著如果你需要改變DOM結構, 你將在`posyLink`中處理. 在`preLink`中處理將會混淆流程並導致一個錯誤.

###作用域

你會經常希望從指令中訪問作用域來監控模型的值並在它們改變時更新UI, 同時在外部時間造成模型改變時通知Angular. 者時最常見的, 當你從jQuery, Closure或者其他庫中包裹一些非Angular組件或者實現簡單的DOM事件時. 然後將Angular表達式作為屬性傳遞到你的指令中來執行. 

這也是你期望使用一個作用域的原因之一, 你可以獲得三種類型的作用域選項:

1. 從指令的DOM元素中獲得**現有的作用域**.
2. 建立一個**新作用域**, 它繼承自你閉合的控制器作用域. 這裡, 你見過能夠訪問樹上層作用域中的所有值. 這個作用域將會請求這種作用域與你DOM元素中其他任意指令共享它並被用於與它們通信.
3. 從它的父層**隔離出來的作用域**不帶有模型屬性. 當你在建立可重用的組件而需要從父作用域中隔離指令操作時, 你將會希望使用這個選項.

你可以使用下面的語法來建立這些作用域類型的配置:

<table>
	<thead>
		<tr>
			<th>Scope Type</th>
			<th>Syntax</th>
		</tr>
	</thead>
	<tbody>
		<tr>
			<td>existing scope</td>
			<td>scope: false(如果不指定將使用這個默認值)
		</tr>
		<tr>
			<td>new scope</td>
			<td>scope: true</td>
		</tr>
		<tr>
			<td>isolate scope</td>
			<td>scope: { /* attribute names and binding style */ }</td>
		</tr>
	<tbody>
</table>

當你建立一個隔離的作用域時, 默認情況下你不需要訪問父作用域中模型中的任何東西. 然而, 你也可以指定你想要的特定屬性傳遞到你的指令中. 你可以認為是吧這些屬性名作為參數傳遞給函數的.

注意, 雖然隔離的作用域不就成模型屬性, 但它們仍然是其副作用域的成員. 就像所有其他作用域一樣, 它們都有一個`$parent`屬性引用到它們的父級.

你可以經由傳遞一個指令屬性名的映射的方式從父作用域傳遞特定的屬性到隔離的作用域中. 這裡有三種合適的方式從父作用域中傳遞資料. 我們稱這些傳遞資料不同的方式為"綁定策略". 你也可以可選的指定一個區域別名給屬性名稱.

以下是沒有別名的語法:

	scope: {
		attributeName1: 'BINDING_STRATEGY',
		attributeName2: 'BINDING_STRATEGY',...
	}

以下是使用別名的方式:

	scope: {
		attributeAlias: 'BINDING_STRATEGY' + 'templateAttributeName',...
	}

綁定策略被定義為表6-4中的符號:

表6-4 綁定策略

<table>
	<thead>
		<tr>
			<th>Symbol</th>
			<td>Meaning</th>
		</tr>
	</thead>
	<tbody>
		<tr>
			<td>@</td>
			<td>將屬性作為字符串傳遞. 你也可以經由在屬性值中使用插值符號{{}}來從閉合的作用域中綁定資料值.</td>
		</tr>
		<tr>
			<td>=</td>
			<td>使用你的指令的副作用域中的一個屬性綁定資料到屬性中.</td>
		</tr>
		<tr>
			<td>&</td>
			<td>從父作用域中傳遞到一個函數中, 以後呼叫.</td>
		</tr>
	</tbody>
</table>

這些都是相當抽像的概念, 因此讓我們來看一個具體的例子上的變化來進行說明. 比方說我們希望建立一個`expander`指令在標題欄被點擊時顯示額外的內容.

收縮時它看起來如圖6-2所示.

![6-2](figure/6-2.png)

圖6-2 Expander in closed state

展開時它看起來如圖6-3所示.

![6-3](figure/6-3.png)

圖6-3 Expander in open state

我們會編寫如下代碼:

	<div ng-controller="SomeController">
		<expander class="expander" expander-title="title">
			{{text}}
		</expander>
	</div>

標題(Cliked me to expand)和文字(Hi there folks...)的值來自於閉合的作用域中. 我們可以像下面這樣來設定一個控制器:

	function SomeController($scope) {
		$scope.title = 'Clicked me to expand';
		$scope.text = 'Hi there folks, I am the content that was hidden but is now shown.';
	}

然後我們可以來編寫指令:

	angular.module('expanderModule', [])
		.directive('expander', function(){
			return {
				restrict: 'EA',
				replace: true,
				transclude: true,
				scope: { title: '=expanderTitle'},
				template: '<div>' +
						'<div class="title" ng-click="toggle()">{{title}}</div>' +
						'<div class="body" ng-show="showMe" ng-tansclude></div>' +
						'</div>',
				link: function(scope, element, attris){
					scope.showMe = false;
					scope.toggle = function toggle(){
						scope.showMe = !scope.showMe;
					}
				}
			}
		});

然後編寫下面的樣式:

	.expander {
		border: 1px solid black;
		width: 250px;
	}
	.expander > .title {
		background-colo: black;
		color: white;
		padding: .1em .3em;
		cursor: pointer;
	}
	.expander > .body {
		padding: .1em .3em;
	}

接下來讓我們來看看指令中的每個選項是做什麼的, 在表6-5中.

表6-5 Functions of elements

<table>
	<thead>
		<tr>
			<th>FunctionName</th>
			<th>Description</th>
		</tr>
	</thead>
	<tbody>
		<tr>
			<td>restrict: EA</td>
			<td>一個元素或者屬性都可以呼叫這個指令. 也就是說, \<expander ...\>...\</expander\>與\<div expander...\>...\</div\>是等價</td>
		</tr>
		<tr>
			<td>replace:true</td>
			<td>使用我們提供的樣板替換原始元素</td>
		</tr>
		<tr>
			<td>transclude:true</td>
			<td>將原始元素的內容移動到我們所提供的樣板的另外一個位置.</td>
		</tr>
		<tr>
			<td>scope: {title: =expanderTitle}</td>
			<td>建立一個稱為`title`的區域作用域, 將父作用域的屬性資料繫結到聲明的`expanderTitle`屬性中. 這裡, 我們重命名title為更方便的expanderTitle. 我們可以編寫`scope: { expanderTitle: '='}`, 那麼在樣板中我們就要使用`expanderTitle`了. 但是在其他指令也有一個`title`屬性的情況下, 在API中消除title的歧義和只是重命名它用於在區域使用是有意義的. 請注意, 這裡自定義指令也使用了相同的駝峰式命名方式作為指令名.</td>
		</tr>
		<tr>
			<td>template: \<'div'\>+</td>
			<td>聲明這個指令要插入的樣板. 注意我們使用了`ng-click`和`ng-show`來顯示和隱藏自身並使用`ng-transclude`聲明了原始內容會去哪裡. 還要注意的是transcluded的內容能夠訪問父作用域, 而不是指令閉合中的作用域.</td>
		</tr>
		<tr>
			<td>link...</td>
			<td>設定`showMe`模型來檢測expander的展開/關閉狀態, 同時定義在用於點擊`title`這個div的時候呼叫定義的`toggle()`函數.</td>
		</tr>
	</tbody>
</table>

如果我們像使用更多有意義的東西來在樣板中定義`expander title`而不是在模型中, 我們還可以使用傳遞經由在作用域聲明中使用`@`符號傳遞一個字符串風格的屬性, 就像下面這樣:

	scope: { title: '@expanderTitle'},

在樣板中我們就可以實現相同的效果:

	<expander class="expander" expander-title="Click mr to expand">
		{{text}}
	</expander>

注意, 對於@策略我們仍然可以經由使用插入法將title資料繫結到我們的控制器作用域中:

	<expander class="expander" expander-title="{{title}}">
		{{text}}
	</expander>

###操作DOM元素