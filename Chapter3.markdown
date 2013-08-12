#第三章 AngularJS開發

現在, 我們已經探究了組成AngularJS的一些輪子. 我們已經知道用戶進入我們的應用程式後如何取得資料, 如何顯示文字, 以及如何做一些時髦的驗證, 過濾和改變DOM. 但是我們要如何把它們組織在一起呢?

在本章, 我們將討論以下內容:

+ 如何適應快速開發部署AngularJS應用程式
+ 啟動伺服器查看應用程式行為
+ 使用Karma編寫和運行單元測試和場景測試
+ 針對產品部署編譯和壓縮你的AngularJS應用程式
+ 使用Batarang偵錯你的AngularJS應用程式
+ 簡化開發流程(從建立文件到運行應用程式和測試)
+ 使用依賴管理庫RequireJS整合AnguarJS項目

本章旨在提供一個20000英尺的視圖以告訴你如何可行的部署你的AngularJS應用程式. 我們不會進入實際應用程式本身. 在第4章, 深入一個使用和展示了各種各樣AngularJS特性的範例一用程序.

##專案架構

推薦使用Yeoman構建你的項目, 將會為你的AngularJS應用程式建立所有必要的輔助程序文件.

Yeoman是一個包含多個框架和客戶端庫的強大的工具. 它經由自動化一些日常任務的引導的方式提供了一個快速的開發環境和增強你的應用程式. 本章我們會使用一個完整的小節介紹如何安裝和使用Yeoman, 但是在那之前, 我們先來簡單的介紹以下使用Yeoman命令替代那些手動執行的操作.

我們還詳細介紹了涉及到讓你決定不使用Yeoman的情況, 因為在Windows平台的計算機上Yeoman確實存在一些問題, 並且設定它還稍微有一點挑戰性.

對於那些不使用Yeoman的情況, 我們將會看看一個範例應用程式結構(可以在Github範例倉庫的`chapter3/sample-app`目錄中找到 - [鏈接](https://github.com/shyamseshadri/angularjs-book/tree/master/chapter3/sample-app)), 下面是推薦的結構, 也是使用Yeoman產生的結構. 應用程式的文件可以分為以下類別:

**JS原始碼**

看看`app/scripts`目錄. 這裡是你所有的JS原始碼所在目錄. 一個主文件來給你的應用程式設定Angular模組和路由.

此外, 這裡還有一個單獨的文件夾--`app/scripts/controller`--這裡面是各個控制器. 控制器提供行為和發佈資料到作用域中, 然後顯示在視圖中. 通常, 它們與視圖都是一一對應的.

指令, 過濾器和服務也可以在`app/scripts`下面找到, 不管是否優雅和複雜, 作為一個完整的文件(direcyives.js, filters.js, services.js)或者單個的都行.

**Angular HTML樣板文件**

現在, 使用Yeoman建立的每一個AngularJS區域樣板都可以在`app/views`目錄中找到. 這是映射到大多數`app/scripts/controller`目錄中.

還有另外一個重要的Angular樣板文件, 就是主要的`app/index.html`. 這用戶取得AngularJS原始碼, 也是你為應用程式建立的任意原始碼.

如果你最終會建立一個新的JS文件, 要確保把它添加到`index.html`中, 同時還要更新的主模組和路由(Yeoman也會為你做這些).

**JS庫依賴**

Yeoman在`app/scripts/vendor`中為你提供了所有的JS原始碼依賴. 想在應用程式中使用[Underscore](http://underscorejs.org/)和[SocketIO](http://socket.io/)? 沒問題--將依賴添加到vendor目錄中(還有你的`index.html`), 並開始在你的用用程序中引用它.

**靜態資源**

最終你建立了一個HTML應用程式, 它還會考慮到你的應用程式還有作為需要的CSS和圖像依賴. `app/styles`和`app/img`目錄就是出於這個目的而產生的. 你只需要添加你需要的東西到目錄中並在你的應用程式中引用它們(當然, 要使用正確的絕對路徑).

> 默認情況下Yeoman不會建立`app/img`路徑.

**單元測試**

測試是非常重要的, 並且當它涉及到AngularJS時是毫不費力的. 在測試方面`test/spec`目錄應該映射到你的`app/scripts`目錄. 每個文件都應該有一個包含它的單元測試的spec文件映射(鏡像). 種子文件會給每個控制器文件建立一個存根文件, 在`test/spec/controllers`目錄下, 與原來的控制器具有相同的名稱. 它們都是Jasmine風格的規範, 描述了每個控制器預期行為的規範.

**整合測試**

AngularJS自帶了端對端的測試支持以正確的方式內置到庫裡面. 你所有的Jasmine規範形式的E2E(端對端)測試, 都保存在`tests/e2e`目錄下.

> 默認情況下Yeoman不會建立`test/目錄`.

> 雖然E2E測試可能看起來像Jasmine, 但實際上不是的. 它們的函數是非同步執行的, 來未來, 可以經由Angular場景運行器(Angular Scenario Runner)運行. 因此不要指望能夠做正常情況下Jasmine測試所做的事情(像使用console.log重複輸出一個值的情況).

還產生了一個簡單的HTML文件, 可以在瀏覽器中打開它來手動的運行測試. 然而Yeoman不會產生存根文件, 但是它們遵循相似風格的單元測試.

**配置文件**

這裡需要兩個配置文件. 第一個是`karma.conf.js`, 這是Yeoman為你產生的用於運行單元測試的. 第二個, 是Yeoman不會產生的`karma.e2e.conf.js`. 這用於運行場景測試. 在本場尾部的繼承RequireJS一節中有一個簡單的文件. 這用於配置依賴關係的詳情, 同時這個文件用在你使用karma運行單元測試的時候.

你可能會問: 如何運行我的應用程式? 什麼是單元測試? 甚至我該如何編寫你們所討論的這種各樣的零件?

別擔心, 年輕的蚱蜢, 所有的這些在適當的時間都會解釋. 在這一章裡面, 我們將處理設定項目和開發環境的問題, 因此一旦我們摻入一些驚人的代碼, 那些問題都可以快速的帶過. 你所編寫的代碼以及如何將它們與你最終的驚人的應用程式聯繫在一起的問題, 我們將在接下來的幾章中討論.

##工具

AngularJS只是你開發實際網頁的工具箱的一部分. 在這一節, 我們將一起開看看一些你用以確保高效和快速開發的不同的工具, 從IDEs到測試運行器到偵錯工具.

###IDEs

讓我們從你如何編寫原始碼開始. 有大量的JavaScript編輯器可以選擇, 有免費的也有付費的. 長時間以來的事實證明Emacs和Vi是開發JS的最好選擇. 現在, 各種IDEs都自帶了語法高亮, 自動完成以及其他功能, 它給你一個選擇的餘地, 這可能是值得的. 那麼, 應該使用那一個呢?

如果你不介意付幾十塊錢(儘管它有一個30天的免費試用期), [WebStorm](www.jetbrains.com/webstorm/□)是個不錯的選擇, 當今時代, WebStorm由JetBrains提供了最廣泛的Web開發平台. 它所具有的特性, 在之前只有強類型語言才有, 包括代碼自動完成(如圖3-1所示, 指定瀏覽器版本), 代碼導航, 語法和多無高亮, 同時支持多個函式庫和框架的啟動即可使用. 此外, 它還漂亮的整合了在IED中正確的偵錯JavaScript的功能, 而且這些都是基於Chrome執行的.

![ide](figure/3-1.png)

最大的你應該考慮使用WebStorm開發AngularJS原因是它是唯一繼承AngularJS插件的IDEs. 這個插件支持在你的HTML樣板中正確的自動完成AngularJS的HTML標籤. 這些都是常用的代碼片段, 否則你每次都要輸入拼湊的代碼片段. 因此不應該像下面這樣輸入:

	directive('$directiveName$', function factory($injectables$){
		var directiveDefinitionObject = {
			$directiveAttr$;
			compile: function complie(tElement, tAttrs, transclude){
				$END$;
				return function(scope, elements, attrs){
					//...
				}
			}
		};
		return directiveDefinitionObject;
	});

在WebStorm中, 你可以只輸入以下內容:

	ngdc

然後按`Tab`鍵取得同樣的代碼. 這只是大多數代碼自動完成插件提供的功能之一.

##運行你的應用程式

現在讓我們討論如何運行所有你所做的事情 - 查看應用程式活動起來, 在瀏覽器中. 真實的感受以下應用程式是如何工作, 我們需要一個伺服器來服務於我們的HTML和JavaScript代碼. 我將探討兩種方式, 一種非常簡單的方式是使用Yeoman運行應用程式, 另外一種不是很容易的不用Yeoman的方法, 但是同樣很好.

###使用Yeoman

Yeoman讓你很簡單的使用一個Web伺服器服務你所有的靜態資源和相關的JavaScript文件. 只需要運行以下命令:
	
	yeoman server

它將啟動一個伺服器同時在你的瀏覽器中打開AngularJS應用程式的主頁. 每當你改變你的原始碼時, 它甚至會刷新(自動刷新)瀏覽器. 很酷不是嗎?

###不使用Yeoman

如果不使用Yeoman, 你可能需要配置一個伺服器來服務你所有主目錄中的文件. 如果你不知道一個簡單的方法做到這一點, 或者不想浪費時間建立你自己的Web伺服器, 你可以在Node.js中使用ExpressJS快速的編寫一個簡單的Web伺服器(只要簡單的使用`npm install -g express`來取得它). 它可能看起來像下面這樣:

	//available at chapter3/sample-app/web-server.js

	var express = require('express'),
	    app = express(),
	    port = parseInt(process.env.PORT, 10) || 8080;
		app.configure(function(){
			app.use(express.methodOverride());
			app.use(express.bodyParser());
			app.use(express.static(__dirname + '/'));
			app.use(app.router);
		});

	app.listen(port);
	console.log("Now serving the app at http://localhost:" + port + "app");

一旦你有了這個文件, 你就可以使用Node運行這個文件, 經由使用下面的命令:

	node web-server.js

同時它將在8080端口啟動伺服器(或者你自己選擇端口).

可選的, 在應用程式文件夾中使用Python你應該運行:

	python -m SimpleHTTPServer

無論你是否決定繼續, 一旦你配置好伺服器並運行起來, 都將被導航導下面的URL:

	http://localhost:[port-number]/app/index.html

然後你就可以在瀏覽器中查看你剛剛建立的應用程式. 注意, 你需要手動的刷新瀏覽器來查看改變, 不同於使用Yeoman.

##測試AngularJS

之前已經說過(甚至在本章的前面), 我們再重新說一次: 測試是必不可少的, AngularJS使編寫合理的單元測試和整合測試變得很簡單. 雖然AngularJS使用多個測試運行器運行的很好, 但我們堅信[Karma](http://karma-runner.github.io/0.8/index.html)勝過大多數你所需要的提供強大, 堅實和及其快速的運行器.

###Karma

Karma存在的主要的原因是它讓你的測試驅動開發(TDD)流程變得簡單, 快速和有趣. 它使用NodeJS和SocketIO(你不需要知道它們是什麼, 只需要假設它們是很棒很酷的函式庫), 並允許在多個瀏覽器中及其快速的運行你的代碼和測試. 在[https:// github.com/vojtajina/karma/](https:// github.com/vojtajina/karma/)中可以找到更多訊息.

> **TDD簡介**
>
> 測試驅動開發或者TDD, 是一個經由確保在開發生命週期內首先編寫測試的敏捷方法, 這是在代碼實現之前進行的, 這就是測試驅動的開發(不只是作為一種驗證工具).
>
> TDD的原則很簡單:
> 
+ 代碼只需要在一個所需要的代碼測試失敗時編寫.
+  編寫最少的代碼以確保測試經由.
+  在每一步刪除重複代碼.
+  一旦所有的測試經由, 接下來就是給下一個所需要的功能添加失敗測試.
>
> 以下是確保這些原則的簡單規則:
>
+ 有組織的開發你的代碼, 每一行代碼都要有目的的編寫.
+ 你的代碼仍然是高度模組化, 緊密結合和可復用的(你需要能夠測試它)
+ 提供一系列的全面的測試列表, 以防後期的破壞和Bugs.
+ 測試也應該充當規範和文件, 以適應後期需要和變化.
>
> 在AngularJS中我們發現這是真的, 同時在整個AngularJS代碼庫中我們都是使用TDD來開發. 對於像JavaScript這樣的無需編譯的動態語言, 我們堅信良好的單元測試可以減輕未來的頭痛.

那麼, 我們如何取得迷人的Karma呢? 好吧, 首先確保在你的機器上安裝了NodeJS. 它自帶了NPM(Node包管理器), 這使得它易於管理和安裝成千上萬的NodeJS可用的函式庫.

一旦你安裝了NodeJS和NPM, 安裝Karma只需要簡單的運行下面的命令:

	sudo npm install -g karma

到這裡. 你只要簡單的三部來開始使用Karma(我剛才說了, 請不要瞭解它現實上是怎麼使用的).

**取得配置文件**:

如果你是用Yeoman建立應用程式骨架, 那麼你就已經有一個現成的Karma配置文件等你來使用. 如果不是, 那麼繼續, 並且在你的應用程式目錄的根文件夾中執行下面的命令:

	karma init

在你的終端控制器中執行(定位到目錄文件夾,然後執行命令), 他會產生一個虛擬的配置文件(`karma.conf.js`), 你可以根據你的喜好來編輯它, 它默認帶有一些很好的標準. 你可以使用它.

**啟動Karma伺服器**

只需要運行下面的命令:

	karma start [optionalPathToConfigFile]

這將會在9876端口啟動Karma伺服器(這是默認情況, 你可以經由編輯在上一步提到的`karma.conf.js`文件來改變). 雖然Karma應該打開一個瀏覽器並自動捕獲它, 它將在控制台中打印所有其他瀏覽器中捕獲所需要的指令. 如果你懶得這樣做, 只需要在其他瀏覽器或者設備中瀏覽`http://localhost:9876`, 並且你最好在多個瀏覽器中運行測試.

> 雖然Karma可以自動捕獲常用的瀏覽器, 在啟動時.(FireFox, Chrome, IE, Opera, 甚至是PhantomJS), 但它不僅限於只是這些瀏覽器. 任何可以瀏覽一個URL的設備都可能可以作為Karma運行器. 因此如果你打開你的iPhone或者Android設備上瀏覽器並瀏覽`http://machinename:9876`(只要是可訪問的), 你都可能在移動設備上運行同樣的測試.

**運行測試**

執行下面的命令:

	karma run

就是這樣. 運行這個命令之後, 你應該獲得正好打印在控制台中的結果. 很簡單, 不是嗎?

##單元測試

AngularJS是的編寫單元測試變得更簡單, 默認情況下支持編寫[Jasmine](http://pivotal.github.io/jasmine/)風格的測試(就是Karma). Jasmine就是我們所說的行為驅動開發框架, 它允許你編寫規範來說明你的代碼行為應該如何表現. 一個Jasmine測試範例看起來可能是這樣子的.

	describe("MyController:", function(){
		it("to work correctly", function(){
			var a = 12;
			var b = a;

			expect(a).toBe(b);
			expect(a).not.toBe(null);
		});
	});

正如你可以看到, 它本身就是非常容易閱讀的格式, 大部分的代碼都可以用簡單的英文來閱讀理解. 它還提供了一個非常多樣化和強大的匹配集合(就像`expect`從句), 當然它還有[xUnit](http://xunit.codeplex.com/)常用的`setUp`和`tearDowns`(函數在每個獨立的測試用例之前或者之後執行).

AngularJS提供了一些優雅的原型, 和測試函數一樣, 它允許你有在單元測試中建立服務, 控制器和過濾器的權利, 以及模擬`HTTPRequests`輸出等等. 我們將在第五章討論這個.

Karma可以使它很容易的整合到你的開發流程中, 以及在你編寫的代碼中取得即時的反饋.

**整合到IDEs中**

Karma並沒有所有最新版和最好的(greatest)IDEs使用的插件(已經實現的還沒有), 但實際上你並不需要. 所有你所需要做的就是在你的IDEs中添加一個執行"karma start"和"karma run"的快捷命令. 這通常可以經由添加一個簡單的腳本來執行, 或者實際的shell命令, 依賴於你所選擇的編輯器. 當然, 每次完成運行你都應該看到結果.

**在每一個變化上運行測試**

這是許多TDD開發者的理想國: 能夠運行在它們所有的測試中, 每次它們按下保存, 在急毫秒之內迅速的得到返回結果. 使用AngularJS和Karma可以很容易做到這一點. 事實證明, Karma配置文件(記住就是前面的`karma.conf.js`)有一個看似無害的名為**`autoWatch`**的標誌. 設定它為true來告訴Karma每次運行你的測試文件(這就是你的原始碼和測試代碼)都監控它的變化. 然後在你的IDE中執行"karma start", 猜猜會怎樣? Karma運行結果將可供你的IDE使用. 你甚至不需要切換控制台或者終端來瞭解發生了什麼.

##端到端/整合測試

隨著應用程式的發展(或者有這個趨勢, 事實上很快, 之前你甚至已經意識到這一點), 測試它們是否如預期那樣工作而不需要手動的裁剪任何功能. 畢竟, 沒一添加新的特性, 你不僅要驗證新特性的工作, 還要驗證老特性是否仍然更夠正常工作, 並且沒有bug和功能也沒有退化. 如果你開始添加多個瀏覽器, 你可以很容看出, 其實這些可以變成一個組合.

AngularJS視圖經由提供一個Scenario Runner來模擬用戶與應用程式交互來緩解這種現象.

Scenario Runner允許你按照類Jasmine的語法來描述應用程式. 正如之前的單元測試, 我們將會有一些的`describes`(針對這個特性), 同時它還是獨立(描述每個單獨功能的特性). 和往常一樣, 你可以有一些共同的行為, 對於執行每個規範之前和之後.(我們稱之為測試).

來看一個應用程式範例, 他返回過濾器結果列表, 看起來可能像下面這樣:

	describe("Search Results", function(){
		beforeEach(function(){
			browser().navigateTo("http://localhost:8000/app/index.html");
		});
		it("Should filter results", function(){
			input("searchBox").enter("jacksparrow");
			element(":button").click();
			expect(repeater("ul li").count()).toEqual(10);
			input("filterText").enter("Bees");
			expect(repeater("ul li").count()).toEqual(1);
		});
	});

有兩種方式運行這些測試. 不過, 無論使用那種方式運行它們, 你都必須有一個Web伺服器來啟動你的應用程式服務(請參見上一節來查看如何做到這一點). 一旦做到這一點, 可以使用下列方法之一:

1. **自動化**: Karma現在支持運行Angular情景測試. 建立一個Karma配置文件然後進行以下改變:

  a. 添加一個ANGULAR_SCENARIO & ANGULAR_SCENARIO_ADAPTER到配置的文件部分.

  b. 添加一個代理伺服器將請求定位到正確的測試文件所在目錄, 例如:

  	proxies = {'/': 'http://localhost:8000/test/e2e'};
  
  c. 添加一個Karma root(根目錄/基礎路徑)以確保Karma的原始碼不會干擾你的測試, 像這樣:

  	urlRoot = '/_karma_/';

  然後只需要記得經由瀏覽`http://localhost:9876/_karma_`來捕捉Karma伺服器, 你應該使用Karma自由的運行你的測試.

2. **手動**: 手動的方法允許你從你的Web伺服器上打開一個簡單的頁面運行(和查看)所有的測試.

  a. 建立一個簡單的`runner.html`文件, 這來源於Angular庫的`angular-scenario.js`文件.

  b. 所有的JS原始碼都遵循你所編寫的你的場景組件部分的規範.

  c. 啟動你的Web伺服器, 瀏覽`runner.html`文件.

為什麼你應該使用Angular場景運行器, 或者說是外部的第三方庫活端對端的測試運行器? 使用場景運行器有令人驚訝的好處, 包括:

**AngularJS意識**

Angular場景情運行器, 顧名思義, 它是由Angular建立的. 因此, 他就是Angular aware, 它直到並理解各種各樣的Angular元素, 類似綁定. 需要輸入一些文字? 檢查綁定的值? 驗證中繼器(repeater)狀態? 所有的這些都可以經由場景運行器輕鬆的完成.

**無需隨機等待**

Angular意識也意味著Angular直到所有的XHR何時向伺服器放出, 從而可以避免頁面加載所等待的間隔時間. 場景運行器直到何時加載一個頁面, 從而比Selenium測試更具確定性, 例如, 超時等待頁面加載時任務可能失敗.

**偵錯功能** 

探究JavaScript, 如果你查看你的代碼不是很好; 當你希望暫停和恢復測試時, 所有的這些都運行場景測試嗎? 然而所有的這一切經由Angular場景運行器都是可行的, 等等.

##編譯

在JavaScript世界裡, 編譯通常意味著壓縮代碼, 雖然一些實際的編譯可能使用的時Google的Closure庫. 但是為麼你會希望將所有漂亮的, 寫的很好, 很容易理解代碼變得不可讀呢?

原因之一是我們的目標是是應用程式更快的回應用戶. 這是為什麼客戶端應用程式幾年前不想現在騰飛得這麼快的主要原因. 能夠越早取得應用程式並運行, 回應得也越早.

這種快速回應是壓縮JS代碼的動機. 代碼越小, 越能有效的減小負載, 同時能夠更快的將相關文件發送給用戶. 這在移動應用程式中顯得尤為重要, 因為其規模為成為瓶頸.

這裡有集中方法可以壓縮你給應用程式所編寫的AngularJS代碼, 每種方法都具有不同的效果.

**基本的和簡單的優化**

這包括壓縮所有在你的代碼中使用的變數, 但是不會壓縮屬性. 這就是所謂的傳遞給Closure Compiler的簡單優化.

者不會給你帶來多大的文件大小的節省, 但是你仍然可以獲得一個可觀的最小開銷.

這項工作的原因是編譯器(Closure或者UglifyJS)並不會重命名你從樣板中引用的屬性.  因此, 你的樣板會繼續工作, 僅僅重命名區域變數和參數.

對於Google Closure, 只需簡單的呼叫下面的命令:

	java -jar closure_compiler.js --compilation_level | SIMPLE_OPTIMIZATIONS --js path/to/files.js

**進階優化**

進階優化有一點棘手, 它會試圖重名幾乎任何東西和每個函數. 得到這個級別的優化工作, 你將需要經由顯示的告訴它哪些函數, 變數和屬性需要重命名(經由使用一個externsfile). 者通常是經由樣板訪問函數和屬性.

編譯將會使用這個`externs`文件, 然後重命名所有的東西. 如果處理好, 這可能會導致的你的JavaScript文件大幅度的減小, 但是它的確需要相當大的工作像, 包括每次改變代碼都要更急externs文件.

要記住一件事: 當你想要壓縮代碼時, 你要使用依賴注入的形式(在控制器上指定`$inject`屬性).

下面的代碼不會工作

	function MyController($scope, $resource){
		//Stuff here
	}

你需要像下面這樣做:

	function MyController($scope, $resource){
		//Some Stuff here
	}
	MyController.$inject = ['$scope', '$resource'];

或者是使用模組, 像這樣:

	myAppModule('MyController', ['$scope', '$resource', function($scope, $resource){
		//Some stuff here
	}]);

一旦所有的變數都混餚或者壓縮只有, 這是使用Angular找出那些你最初使用的服務和變數的方式.

> 每次都是數組的方式注入是比較好的處理發方式, 以避免開始編譯代碼時的錯誤. 撓頭並視圖找出為什麼提供的$e變數丟失了(一些任務的混餚版本壓縮了它)是不值得的.

##其他優秀工具

在本節, 我們將會看一些其他有助於簡化你的開發流程和提高效率的工具. 這包括使用Batarang偵錯真實的代碼和使用Yeoman開發.

###偵錯

當你使用JavaScript工作時, 在瀏覽器中偵錯你的代碼會成為一個習慣. 你越早接受這個事實, 對你越有好處. 值得慶幸的是, 當過去沒有Firebug時, 這件事已經走過了漫長的路. 現在, 不管選擇什麼瀏覽器, 一般來說你都可以介入代碼來分析錯誤和判斷應用程式的狀態. 只需要去瞭解Chrome和Internet Explorer的開發者工具, 能同時在FireFox和Chrome中工作的Firebug.

這裡有一些幫助你進一步偵錯應用程式的技巧提示:

+ 永遠記住, 當你希望偵錯引用程序時, 記得切換到非壓縮版本的代碼和依賴中進行. 你不僅會獲得更好(可讀)變數名, 也會獲得代碼行號和實際有用的訊息以及偵錯功能.
+ 盡量保持你的原始碼為獨立的JS文件, 而不是內聯在HTML中.
+ 斷點偵錯是很有用的! 它們允許你檢查你的應用程式狀態, 模型, 以及給定的時間點上的所有訊息.
+ "暫停所有異常"是內置在當今大多數開發者工具中的一個非常有用的選項. 當發現一個異常是偵錯器會終止繼續運行並高亮導致異常的代碼行.

###Batarang

當然, 我們有Batarang. Batarang是一個添加AngularJS知識的Chrome擴充, 它是嵌套在Google Chrome中內置開發者工具. 一旦安裝(你可以從[http://bit.ly/batarangjs](http://bit.ly/batarangjs)中取得), 它就會在Chorme的開發者工具面板中添加一個AngularJS選項.

你有沒有想過你的AngularJS應用程式的當前狀態是什麼? 當前(視圖)包含的每個模型, 作用域和變數是什麼? 你的應用程式性能如何?  如果你還沒有想過, 相信我, 你會的. 當你需要這麼做時, Batarang會為你服務.

這裡有Batarang的四個主要的有用的附加功能:

####模型選項

Batarang允許你從根源向下深入探究`scope`. 然後你可以看到這些`scopes`是如何嵌套以及模型是如何與之關聯的.(如圖3-2所示). 你甚至可以實時的改變它們並在應用程式中查看變化的反映. 很酷, 不是嗎?

![model tab](figure/3-2.png)

Figure 3-2 Model tree in Batarang

####性能選項

性能選項必須單獨啟用, 它會注入一些特殊的JavaScript代碼到你的應用程式中. 一旦你啟用它, 你就可以看到不同的作用域和模型, 並且可以在每個作用域執行所有的性能監控表達式(如圖3-3所示). 隨著你使用應用程式, 性能也會得到更新, 因此它可以很好的實時工作.

![perforemance tab](figure/3-3.png)

Figure 3-3. Performance tab in Batarang

####服務依賴

對於一個簡單的應用程式, 不會超過1-2個控制器和服務依賴. 但是事實上, 全面的應用程式, 如果沒有工具支持, 服務依賴管理會成為噩夢. 幸好這裡有Batarang可以給你提供服務, 填補這個洞, 因為它給你提供了一個乾淨, 簡單的方法來查看可視化的服務依賴圖表(如圖3-4所示).

![service dependencies](figure/3-4.png)

Figure 3-4. Charting dependencies in Batarang

####元素屬性和控制台訪問

當你經由HTML樣板來探究一個AngularJS應用程式時, 元素選項的屬性窗格中現在有一個額外的AngularJS屬性部分. 這允許你檢查模型所連接的給定元素的`scope`. 它也會公開這個元素的`scope`到控制台中, 因此你可以在控制台中經由`$scope`變數來訪問它. 如圖3-5所示:

![properties](figure/3-5.png)

Figure 3-5. AngularJS properties within Batarang

##Yeoman: 優化你的工作流程

相當多的工具如雨後春筍般湧現, 以幫助你在開發應用程式時優化工作流程. 我們在前面章節所談及的Yeoman就是這樣一種工具, 它擁有令人印象深刻的功能集, 包括:

+ 輕快的腳手架
+ 內置預覽伺服器
+ 整合包管理
+ 一流的構建過程
+ 使用PhantomJS進行單元測試

它還很好的整合和擴充了AngularJS, 這也是我們為什麼強烈推薦任何AngularJS項目使用它的主要原因之一. 讓我們經由上面的集中方式使用Yeoman時你的生活更輕鬆.

###安裝Yeoman

安裝Yeoman是一個相當複雜的過程, 但也可以經由一些腳本來幫助你安裝.

在Mac/Linux機器上, 運行下面的命令:

	curl -L get .yeoman.io | bash

然後只需按照打印的只是來取得Yeoman.

對於Windows機器, 或者運行它是遇到任何問題, 到[https://github.com/yeoman/yeoman/ wiki/Manual-Install](https://github.com/yeoman/yeoman/ wiki/Manual-Install)並按照說明來安裝會讓你暢通無阻.

###啟動一個新的AngularJS項目

正如前面所提到的, 甚至一個簡單的項目都有許多技術需要處理, 從樣板到基礎控制, 再到庫依賴, 一切事情都需要結構化. 你可以手動的做這些工作, 或者也可以使用Yeoman處理它.

只需為你的項目中簡單的建立一個目錄(Yeoman將把目錄名稱當作項目名稱), 然後運行下面的命令:

	yeoman init angular

這將建立一個本章項目優化部分所詳細描述的一個完整的結構, 包括渲染路由的框架, 單元測試等等.

###運行伺服器

如果你不適用Yeoman, 那麼你不得不建立一個HTTP伺服器來服務你的前端代碼. 但是如果使用Yeoman, 那麼你將獲得一個內置的預先配置好的伺服器並且還能獲得一些額外的好處. 你可以使用下面的命令啟動伺服器:

	yeoman server

這不單單只啟動一個Web伺服器來服務於你的代碼, 它還會自動打開你的Web瀏覽器並在你改變你的應用程式時刷新你的瀏覽器.

###添加新的路由, 視圖和控制器

添加一個新的Angular路由涉及多個步驟, 其中包括:

+ 在`index.html`中啟用新的控制器JS文件
+ 添加正確的路由到AngularJS模組中
+ 建立HTML樣板
+ 添加單元測試

所有的這些在Yeoman中使用下面的命令可以在一步完成:

	yeoman init angular:route routeName

因此, 如果你運行`yeoman iniy angular route home`結束之後它將執行以下操作:

+ 在`app/scripts/controllers`目錄中建立一個`home.js`控制器骨架
+ 在`test/specs/controllers`目錄中建立一個`home.js`測試規範
+ 將`home.html`樣板添加到`app/views`目錄中
+ 鏈接主引用模組中的home路由(在app/scripts/app.js`文件中)

所有的這些都只需要一條單獨的命令!

###測試的故事

我們已經看過使用Karma如何輕鬆的啟動和運行測試. 最終, 運行所有的單元測試只需要兩條命令.

Yeoman使它變得更容易(如果你相信它). 每當你使用Yeoman產生一個文件, 它都會給你建立一個填充它的測試存根. 一旦你安裝了Karma, 使用Yeoman運行測試只需執行下面的命令即可:

	yeoman test

###構建項目

構建一個完備的應用程式可能是痛苦的, 或者至少涉及到需要步驟. Yeoman經由允許你像下面這樣做減輕了不少痛苦:

+ 連接(合併)所有JS腳本到一個文件中
+ 版本化文件
+ 優化圖片
+ 產生應用程式快取清單

所有的這些好處都來自於一條命令:

	yeoman build

Yeoman不支持壓縮文件, 但是根據來發者提供的訊息, 它很快會到來.

##使用RequireJS整合AngularJS

如果你提單做好更多的事情, 正好會讓你的開發環境更簡單. 後期修改你的開發環境, 會需要修改更多的文件. 依賴管理和建立包部署是任何規模的項目所憂慮的.

使用JavaScript設定你的開發環境是相當困難的, 因為它涉及Ant構建維護, 連接你的文件來構建腳本, 壓縮它們等等. 值得慶幸的是, 在不久之前已經出現了像RequireJS這樣的工具, 它允許你定義和管理你的JS依賴關係, 以及將他們掛到一個簡單的構建過程中. 隨著這些非同步加載管理的工具誕生, 能夠確保所有的依賴文件在執行之前加載好, 重點工作可以放在實際的功能開發, 在此之前從未如此簡單過.

值得慶幸的是, AngularJS能夠很好的發揮[RequireJS](http://requirejs.org/), 因此你可以做到兩全其美. 這裡有一個目標範例, 我們找到了在一個系統中能夠工作的很好而且易於遵循的方式來提供一個樣本設定.

讓我們一起來看看這個項目的組織(類似前面描述的骨架, 稍微有一點變化):

1. **app**: 這個目錄是所有顯示給用戶的應用程式代碼宿主目錄. 包括HTML, JS, CSS, 圖片和依賴的函式庫.

    a. /**style**: 包含所有的CSS/Less文件

    b. /**images**: 包含項目的所有圖片文件

    c. /**script**: 主AngularJS代碼庫. 這個目錄也包括我們的引導程序代碼, 主要的RequireJS整合

        i. /**controllers**: 這裡是AngularJS控制器
        
        ii. /**directives**: 這裡是AngularJS指令

        iii. /**filters**: 這裡是AngularJS過濾器

        iv. /**services**: 這裡是AngularJS服務

    d. /**vendor**: 我們所依賴的函式庫(Bootstrap, RequireJS, jQuery)

    e. /**views**: 視圖的HTML樣板部分和項目所使用的組件

2. **config**: 包含單元測試和場景測試的Karma配置

3. **test**: 包含應用程式的單元測試和場景測試(整合的)

    a. /**spec**: 包含應用程式的JS目錄中的單元測試和鏡像結構

    b. /**e2e**: 包含端到端的場景規範

我們所需要做的第一件事情是在`main.js`文件(在app目錄)中引入RequireJS, 然後使用它加載所有的其他依賴項. 這裡有一個例子, 我們的JS項目除了自己的代碼還會依賴於jQuery和Twitter的Bootstrap.

	//the app/scripts/main.js file, which defines our RequireJS config
	require.config({
		paths: {
			angular: 'vendor/angular.min',
			jquery: 'vendor/jquery',
			domReady: 'vendor/require/domReady',
			twitter: 'vendor/bootstrap',
			angularResource: 'vendor/angular-resource.min'
		},
		shim: {
			'twitter/js/bootstrap': {
				deps: ['jquery/jquery']
			},
			angular: {
				deps: ['jquery/jquery', 'twitter/js/bootstrap'],
				exports: 'angular'
			},
			angularResource: {
				deps: ['angular']
			}
		}
	});

	require([
		'app',
		//Note this is not Twitter Bootstrap
		//but our AngularJS bootstrap
		'bootstrap',
		'controllers/mainControllers',
		'services/searchServices',
		'directives/ngbkFocus'
		//Any individual controller, service, directive or filter file
		//that you add will need to be pulled in here.
		//This will have to be maintained by hand.
		],
		function(angular, app){
			'use strict';

			app.config(['$routeProvider',
				function($routeProvider){
					//define your Routes here
				}
			]);
		}
	);

然後我們定義一個`app.js`文件. 這個文件定義我們的AngularJS應用程式, 同時告訴它, 它依賴於我們所定義的所有控制器, 服務, 過濾器和指令. 我們所看到的RequireJS依賴列表中所提到的只是一點點.

你可以認為RequireJS依賴列表就是一個JavaScript的import語句塊. 也就是說, 代碼塊內的函數直到所有的依賴列表都滿足或者加載完成它都不會執行.

另外請注意, 我們不會單獨 告訴RequireJS, 載入的執行,服務或者過濾器是什麼, 因為這些並不屬於項目的結構. 每個控制器, 服務, 過濾器和指令都是一個模組, 因此只定義這些為我們的依賴就可以了.

	// The app/scripts/app.js file, which defines our AngularJS app
	define(['angular', 'angularResource', 'controllers/controllers','services/services', 'filters/filters','directives/directives'],function (angular) {
		return angular.module(『MyApp』, ['ngResource', 'controllers', 'services','filters', 'directives']);
	});

我們還有一個`bootstrap.js`文件, 它到等到DOM準備就緒(這裡使用的RequireJS的插件`domReady`), 然後告訴AngularJS繼續執行, 這是很好的.

	// The app/scripts/bootstrap.js file which tells AngularJS
	// to go ahead and bootstrap when the DOM is loaded
	define(['angular', 'domReady'], function(angular, domReady) {
		domReady(function() {
			angular.bootstrap(document, [『MyApp』]);
		});
	});

這裡將引導從應用程式中分割出來, 還有一個有事, 即我們可以使用一個偽造的文件潛在的取代我們的`mainApp`或者出於測試的目的使用一個`mockApp`. 例如 如果你所依賴的伺服器不可開, 你只需要建立一個`fakeApp`使用偽造的資料來替換所有的`$http`請求, 以保持你的開發秩序. 這樣的話, 你就可以只悄悄的使用一個`fakeBootstrap`和一個`fakeApp`到你的應用程式中.

現在, 你的`main.html`主樣板(app目錄中)可能看起來像下面這樣:

	<!DOCTYPE html>
	<html> <!-- Do not add ng-app here as we bootstrap AngularJS manually-->
	<head>
		<title>My AngularJS App</title>
		<meta charset="utf-8" />
		<link rel="stylesheet" type="text/css" href="styles/bootstrap.min.css">
		<link rel="stylesheet" type="text/css"
		href="styles/bootstrap-responsive.min.css">
		<link rel="stylesheet" type="text/css" href="styles/app.css">
	</head>
	<body class="home-page" ng-control ler="RootController">
		<div ng-view ></div>
		<script data-main="scripts/main" src="lib/require/require.min.js"></script>
	</body>
	</html>

現在, 我們來看看`js/controllers/controllers.js`文件, 這看起來幾乎與`js/directives/directives.js`, `js/filters/filters.js`和`js/services/services.js`一模一樣:

	define(['angular'], function(angular){
		'use strict';
		return angular.module('controllers', []);
	});

因為我們使用了RequireJS依賴的結構, 所有的這些都會保證只在Angular依賴滿足並加載完成的情況下才會運行.

每個文件都定義為一個Angular模組, 然後經由將單個的控制器, 指令, 過濾器和服務添加到定義中來使用.

讓我們來看看一個指定定義(比如第二章的`focus`指令):

	//File: ngbkFocus.js

	define(['directives/directives'], function(directives) {
		directives.directive(ngbkFocus, ['$rootScope'], function($rootScope){
			return {
				restict: 'A',
				scope: true,
				link: function(scope, element, attrs){
					element[0].focus();
				}
			}
		});
	});

指令自什麼很瑣碎的, 讓我們仔細看看發生了什麼. 圍繞著文件的RequireJS shim告訴我們`ngbkFocus.js`依賴於在模組中聲明的`directices/directives.js`文件. 然後它使用注入指令模組將自身指令聲明添加進來. 你可以選擇多個指令或者一個單一的對應的文件. 這完全由你決定.

一個重點的注意事項: 如果你有一個控制器進入到服務中(也就是說你的`RootController`依賴於你的`UserSevice`, 並且取得`UserService`注入), 那麼你必須確保將你定義的文件加入RequireJS依賴中, 就像這樣:

	define(['controllers/controllers', 'services/userService'], function(controllers){
		controllers.controller('RootController', ['$scope', 'UserService', function($scope, UserService){
			//Do what's needed
		}]);
	});

這基本上是你整個原始碼目錄的結構設定.

但是你會問, 這如何處理我的測試? 我很高興你會問這個問題, 因為你會得到答案.

有個很好的消息, Karma支持RequireJS. 只需安裝最新和最好版本的Karma.(使用`npm install -g karma`).

一旦你安裝好Karma, Karma針對單元測試的配置也會回應的改變. 以下是我們如果設定我們的單元測試來運行我們之前定義的項目結構:

    // This file is config/karma.conf.js.
    // Base path, that will be used to resolve files
    // (in this case is the root of the project)
    basePath = '../';

    // list files/patterns to load in the browser
    files = [
        JASMINE,
        JASM I NE_ADAPTER
        REQUIRE,
        REQU I RE_ADAPTER ,
        // !! Put all libs in RequireJS 'paths' config here (included: false).
        // All these files are files that are needed for the tests to run,
        // but Karma is being told explicitly to avoid loading them, as they
        // will be loaded by RequireJS when the main module is loaded.
        {pattern: 'app/scripts/vendor/**/*.js', included: false},
        // all the sources, tests // !! all src and test modules (included: false)
        {pattern: 'app/scripts/**/*.js', included: false},
        {pattern: 'app/scripts/*.js', included: false},
        {pattern: 'test/spec/*.js', included: false},
        {pattern: 'test/spec/**/*.js', included: false},
        // !! test main require module last
        'test/spec/main.js'
     ];
    // list of files to exclude
    exclude = [];

    // test results reporter to use
    // possible values: dots || progress
    reporter = 'progress';

    // web server port
    port = 8989;
    
    // cli runner port
    runnerPort = 9898;

    // enable/disable colors in the output (reporters and logs)
    colors = true;

    // level of logging
    logLevel = LOG_INFO;

    // enable/disable watching file and executing tests whenever any file changes
    autoWatch = true;

    // Start these browsers, currently available: 
    // - Chrome
    // - ChromeCanary
    // - Firefox
    // - Opera
    // - Safari
    // - PhantomJS
    // - IE if you have a windows box 
    browsers = ['Chrome'];
    
    // Cont inuous Integrat ion mode
    // if true, it captures browsers, runs tests, and exits 
    singleRun = false;

 我們使用一個稍微不同的格式來定義的我們的依賴(包括: false是非常重要的). 我們還添加了REQUIRE_JS和適配依賴. 最終進行這一系列工作的是`main.js`, 他會觸發我們的測試.

	// This file is test/spec/main.js

	require.config({
		// !! Karma serves files from '/base'
		// (in this case, it is the root of the project /your-project/app/js) 
		baseUrl: ' /base/app/scr ipts' ,
		paths: {
        angular: 'vendor/angular/angular.min',
			jquery: 'vendor/jquery',
			domReady: 'vendor/require/domReady',
			twitter: 'vendor/bootstrap',
			angularMocks: 'vendor/angular-mocks',
			angularResource: 'vendor/angular-resource.min',
			unitTest: '../../../base/test/spec'
		},

		// example of using shim, to load non-AMD libraries
		// (such as Backbone, jQuery)
		shim: {
			angular: {
				exports: 'angular'
			},
			angularResource: { deps:['angular']},
			angularMocks: { deps:['angularResource']}
		}
	});

	// Start karma once the dom is ready.
	require([
        'domReady' ,
        // Each individual test file will have to be added to this list to ensure
        // that it gets run. Again, this will have to be maintained manually.
        'unitTest/controllers/mainControllersSpec',
        'unitTest/directives/ngbkFocusSpec',
        'unitTest/services/userServiceSpec'
        ], function(domReady) {
            domReady(function() {
                window.__karma__.start();
        });
    });

由此設定, 我們可以運行下面的命令

	karma start config/karma.conf.js

然後我們就可以運行測試了.

當然, 當它涉及到編寫單元測試就需要稍微的改變一下. 它們需要RequireJS支持的模組, 因此讓我們來看一個測試範例:

	// This is test/spec/directives/ngbkFocus.js
	define(['angularMocks', 'directives/directives', 'directives/ngbkFocus'], function() {
		describe('ngbkFocus Directive', function() {
			beforeEach(module('directives'));

			// These will be initialized before each spec (each it(), that is),
			// and reused
			var elem;
			beforeEach(inject(function($rootScope, $compile) {
				elem = $compi le('<input type=」text」 ngbk-focus>')($rootScope);
			}));

			it('should have focus immediately', function() { 
				expect(elem.hasClass('focus')).toBeTruthy();
			});
		});
	});

我們的每個測試將做到以下幾點:

1. 拉取`angularMocks`取得我們的angular, angularResource, 當然還有angularMocks.

2. 拉取進階模組(directives中的指令, controllers中的控制器等等), 然後它實際上測試的是單獨的文件(loadingIndicator).

3. 如果你的測試愈來愈其他的服務或者控制器, 除了在AngularJS中告知意外, 要確保也定義在RequireJS的依賴中.

這種方法可以用於任何測試, 而且你應該這麼做.

值得慶幸的是, RequireJS的處理方式並不會影響我們所有的端到端的測試, 因此可以使用我們目前所看到的方式簡單的做到這一點. 一個範例配置如下, 假設你的服務其在http://localhost:8000上運行你的應用程式:

	// base path, that will be used to resolve files
	// (in this case is the root of the project 
	basePath = '../';

	// list of files / patterns to load in the browser
	files = [
		ANGULAR_SCENARIO,
		ANGULAR_SCENARIO_ADAPTER,
		'test/e2e/*.js'
	];

	// list of files to exclude
	exclude = [];

	// test results reporter to use
	// possible values: dots || progress
	reporter = 'progress';

	// web server port
	port = 8989;

	// cli runner port
	runnerPort = 9898;
	
	// enable/disable colors in the output (reporters and logs)
	colors = true;

	// level of logging
	logLevel = LOG_INFO;

	// enable/disable watching file and executing tests whenever any file changes
	autoWatch = true;

	urlRoot = '/_karma_/';

	proxies = {
		'/': 'http://localhost:8000/'
	};

	// Start these browsers, currently available:
	browsers = ['Chrome'];

	// Cont inuous Integrat ion mode
	// if true, it capture browsers, run tests and exit
	singleRun = false;
