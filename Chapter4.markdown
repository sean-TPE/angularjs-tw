#分析一個AngularJS應用程式

在第2章中, 我們已經討論了一些AngularJS常用的功能, 然後在第3章討論了該如何結構化開發應用程式. 現在, 我們不再繼續深單個的技術點, 第4章將著眼於一個小的, 實際的應用程式進行講解. 我們將從一個實際有效的應用程式中感受一下我們之前已經討論過的(範例)所有的部分.

我們將每次介紹一部分, 然後討論其中有趣和關聯的部分, 而不是討論完整應用程式的前端和核心, 最後在本章的後面我們會慢慢簡歷這個完整的應用程式.

##應用程式

Guthub是一個簡單的食譜管理應用, 我們設計它用於存儲我們超級沒味的食譜, 同時展示AngularJS應用程式的各個不同的部分. 這個應用程式包含以下內容:

+ 一個兩欄的部署
+ 在左側有一個導航欄
+ 允許你建立新的食譜
+ 允許你瀏覽現有的食譜列表

主視圖在左側, 其變化依賴於URL, 或者食譜列表, 或者單個食譜的詳情, 或者可添加新食譜的編輯功能, 或者編輯現有的食譜. 我們可以在圖4-1中看到這個應用的一個截圖:

![Guthub](figure/4-1.png)

Figure 4-1. Guthub: A simple recipe management application

這個完整的應用程式可以在我們的Github中的`chapter4/guthub`中得到.

##模型, 控制器和樣板之間的關係

在我們深入應用程式之前, 讓我們來花一兩段文字來討論以下如何將標題中的者三部分在應用程式中組織在一起工作, 同時來思考一下其中的每一部分.

`model`(模型)是真理. 只需要重複這句話幾次. 整個應用程式顯示什麼, 如何顯示在視圖中, 保存什麼, 等等一切都會受模型的影響. 因此你要額外花一些時間來思考你的模型, 對象的屬性將是什麼, 以及你打算如何從伺服器取得並保存它. 視圖將經由資料繫結的方式自動更新, 所以我們的焦點應該集中在模型上.

`controller`保存業務邏輯: 如何取得模型, 執行什麼樣的操作, 視圖需要從模型中取得什麼樣的訊息, 以及如何將模型轉換為你所想要的. 驗證職責, 使用呼叫伺服器, 引導你的視圖使用正確的資料, 大多數情況下所有的這些事情都屬於控制器來處理.

最後, `template`代表你的模型將如何顯示, 以及用戶將如何與你的應用程式交互. 它主要約束以下幾點:

+ 顯示模型
+ 定義用戶可以與你的應用程式交互的方式(點擊, 文字輸入等等)
+ 應用程式的樣式, 並確定何時以及如何顯示一些元素(顯示或隱藏, hover等等)
+ 過濾和格式化資料(包括輸入和輸出)

要意識到在Angular中的模型-視圖-控制器涉及模式中樣板並不是必要的部分. 相關, 視圖是樣板取得執行被編譯後的版本. 它是一個樣板和模型的組合.

任何類型的業務邏輯和行為都不應該進入樣板中; 這些訊息應該被限制在控制器中. 保持樣板的簡單可以適當的分離關注點, 並且可以確保你只使用單元測試的情況下就能夠測試大多數的代碼. 而樣板必須使用場景測試的方式來測試.

但是, 你可能會問, 在哪裡操作DOM呢? DOM操作並不會真正進入到控制器和樣板中. 它會存在於Angular的指令中(有時候也可以經由服務來處理, 這樣可以避免重複的DOM操作代碼). 我們會在我們的GutHub的範例文件中涵蓋一個這樣的例子.

廢話少說, 讓我們來深入探討一下它們.

##模型

對於應用程式我們要保持模型非常簡單. 這一有一個菜譜. 在整個完整的應用程式中, 它們是一個唯一的模型. 它是構建一切的基礎.

每個菜譜都有下面的屬性:

+ 一個用於保存到伺服器的ID
+ 一個名稱
+ 一個簡短的描述
+ 一個烹飪說明
+ 是否是一個特色的菜譜
+ 一個成份數組, 每個成分的數量, 單位和名稱

就是這樣. 非常簡單. 應用程式的中一切都基於這個簡單的模型. 下面是一個讓你食用的範例菜譜(如圖4-1一樣):

	{
		'id': '1',
		'title': 'Cookies',
		'description': 'Delicious. crisp on the outside, chewy' +
			' on the outside, oozing with chocolatey goodness' +
			' cookies. The best kind',
		'ingredients': [
			{
				'amount': '1',
				'amountUnits': 'packet',
				'ingredientName': 'Chips Ahoy'
			}
		],
		'instructions': '1. Go buy a packet of Chips Ahoy\n'+
			'2. Heat it up in an oven\n' +
			'3. Enjoy warm cookies\n' +
			'4. Learn how to bake cookies from somewhere else'
	}

下面我們將會看到如何基於這個簡單的模型構建更複雜的UI特性.

##控制器, 指令和服務

現在我們終於可以得到這個讓我們牙齒都咬到肉裡面去的美食應用程式了. 首先, 我們來看看代碼中的指令和服務, 以及討論以下它們都是做什麼的, 然後我們我們會看看這個應用程式需要的多個控制器.

###服務

	//this file is app/scripts/services/services.js

	var services = angular.module('guthub.services', ['ngResource']);

	services.factory('Recipe', ['$resource', function(){
		return $resource('/recipes/:id', {id: '@id'});
	}]);

	services.factory('MultiRecipeLoader', ['Recipe', '$q', function(Recipe, q){
		return function(){
			var delay = $.defer();
			Recipe.query(function(recipes){
				delay.resolve(recipes);
			}, function(){
				delay.reject('Unable to fetch recipes');
			});
			return delay.promise;
		};
	}]);

	services.factory('RecipeLoader', ['Recipe', '$route', '$q', function(Recipe, $route, $q){
		return function(){
			var delay = $q.defer();
			Recipe.get({id: $route.current.params.recipeId}, function(recipe){
				delay.resolve(recipe);
			}, function(){
				delay.reject('Unable to fetch recipe' + $route.current.params.recipeId);
			});
			return delay.promise;
		};
	}]);

首先讓我們來看看我們的服務. 在33頁的"使用模組組織依賴"小節中已經涉及到了服務相關的知識. 這裡, 我們將會更深一點挖掘服務相關的訊息.

在這個文件中, 我們實例化了三個AngularJS服務.

有一個菜譜服務, 它返回我們所呼叫的Angular Resource. 這些是RESTful資源, 它指向一個RESTful伺服器. Angular Resource封裝了低層的`$http`服務, 因此你可以在你的代碼中只處理對像.

注意單獨的那行代碼 - `return $resource` - (當然, 依賴於`guthub.services`模型), 現在我們可以將`recipe`作為參數傳遞給任意的控制器中, 它將會注入到控制器中. 此外, 每個菜譜對象都內置的有以下幾個方法:

+ Recipe.get()
+ Recipe.save()
+ Recipe.query()
+ Recipe.remove()
+ Recipe.delete()

> 如果你使用了`Recipe.delete`方法, 並且希望你的應用程式工作在IE中, 你應該像這樣呼叫它: `Recipe[delete]()`. 這是因為在IE中`delete`是一個關鍵字.

對於上面的方法, 所有的查詢眾多都在一個單獨的菜譜中進行; 默認情況下`query()`返回一個菜譜數組.

`return $resource`這行代碼用於聲明資源 - 也給了我們一些好東西:

1. 注意: URL中的id是指定的RESTFul資源. 它基本上是說, 當你進行任何查詢時(`Recipe.get()`), 如果你給它傳遞一個id字段, 那麼這個字段的值將被添加早URL的尾部.

也就是說, 呼叫`Recipe.get{id: 15})將會請求/recipe/15.

2. 那第二個對象是什麼呢? {id: @id}嗎? 是的, 正如他們所說的, 一行代碼可能需要一千行解釋, 那麼讓我們舉一個簡單的例子.

比方說我們有一個recipe對像, 其中存儲了必要的訊息, 並且包含一個id.

然後, 我們只需要像下面這樣做就可以保存它:

	//Assuming existingRecipeObj has all the necessary fields,
	//including id(say 13)
	var recipe = new Recipe(existingRecipeObj);
	recipe.$save();

這將會觸發一個POST請求到`/recipe/13`.

`@id`用於告訴它, 這裡的id字段取自它的對象中同時用於作為id參數. 這是一個附加的便利操作, 可以節省幾行代碼.

在`apps/scripts/services/services.js`中有兩個其他的服務. 它們兩個都是加載器(Loaders); 一個用於加載單獨的食譜(RecipeLoader), 另一個用於加載所有的食譜(MultiRecipeLoader). 這在我們連接到我們的路由時使用. 在核心上, 它們兩個表現得非常相似. 這兩個服務如下:

1. 建立一個`$q`延遲(deferred)對像(它們是AngularJS的promises, 用於鏈接非同步函數).
2. 建立一個伺服器呼叫.
3. 在伺服器返回值時resolve延遲對像.
4. 經由使用AngularJS的路由機制返回promise.

> **AngularJS中的Promises**
>
> 一個promise就是一個在未來某個時刻處理返回對像或者填充它的接口(基本上都是非同步行為). 從核心上講, 一個promise就是一個帶有`then()`函數(方法)的對象.
>
>讓我們使用一個例子來展示它的優勢, 假設我們需要取得一個用戶的當前配置:

	var currentProfile = null;
	var username = 'something';

	fetchServerConfig(function(){
		fetchUserProfiles(serverConfig.USER_PROFILES, username, 
			function(profiles){
				currentProfile = profiles.currentProfile;	
		});	
	});

> 對於這種做法這裡有一些問題:
>
> 1. 對於最後產生的代碼, 縮進是一個噩夢, 特別是如果你要鏈接多個呼叫時.
> 
> 2. 在回呼和函數之間錯誤報告的功能有丟失的傾向, 除非你在每一步中處理它們.
>
> 3. 對於你想使用`currentProfile`做什麼, 你必須在內層回呼中封裝其邏輯, 無論是直接的方式還是使用一個單獨分離的函數.
>
> Promises解決了這些問題. 在我們進入它是如何解決這些問題之前, 先讓我們來看看一個使用promise對同一問題的實現.

	var currentProfile = fetchServerConfig().then(function(serverConfig){
		return fetchUserProfiles(serverConfig.USER_PROFILES, username);
	}).then(function{
		return profiles.currentProfile;
	}, function(error){
		// Handle errors in either fetchServerConfig or
		// fetchUserProfile here
	});

> 注意其優勢:
>
> 1. 你可以鏈接函數呼叫, 因此你不會產生縮進帶來的噩夢.
>
> 2. 你可以放心前一個函數呼叫會在下一個函數呼叫之前完成.
>
> 3. 每一個`then()`呼叫都要兩個參數(這兩個參數都是函數). 第一個參數是成功的操作的回呼函數, 第二個參數是錯誤處理的函數.
> 4. 在鏈接中發生錯誤的情況下, 錯誤訊息會經由錯誤處理器傳播到應用程式的其他部分. 因此, 任何回呼函數的錯誤都可以在尾部被處理.
>
> 你會問, 那什麼是`resolve`和`reject`呢? 是的, `deferred`在AngularJS中是一種建立promises的方式. 呼叫`resolve`來滿足promise(呼叫成功時的處理函數), 同時呼叫`reject`來處理promise在呼叫錯誤處理器時的事情.

當我們鏈接到路由時, 我們會再次回到這裡.

###指令

我們現在可以轉移到即將用在我們應用程式的指令上來. 在這個應用程式中將有兩個指令:

`butterbar`

這個指令將在路由發生改變並且頁面仍然還在加載訊息時處理顯示和隱藏任務. 它將連接路由變化機制, 基於頁面的狀態來自動控制顯示或者隱藏是否使用哪個標籤.

`focus`

這個`focus`指令用於確保指定的文字域(或者元素)擁有焦點.

讓我們來看一下代碼:

	// This file is app/scripts/directives/directives.js

	var directive = angular.module('guthub.directives', []);

	directives.directive('butterbar', ['$rootScope', function($rootScope){
		return {
			link: function(scope, element attrs){
				element.addClass('hide');

				$rootScope.$on('$routeChangeStart', function(){
					element.removeClass('hide');
				});

				$routeScope.$on('$routeChangeSuccess', function(){
					element.addClass('hide');
				});
			}
		};
	}]);

	directives.dirctive('focus',function(){
		return {
			link: function(scope, element, attrs){
				element[0].focus();
			}
		};
	});

上面所述的指令返回一個對像帶有一個單一的屬性, link. 我們將在第六章深入討論你可以如何建立你自己的指令, 但是現在, 你應該知道下面的所有事情:

1. 指令經由兩個步驟處理. 在第一步中(編譯階段), 所有的指令都被附加到一個被查找到的DOM元素上, 然後處理它. 任何DOM操作否發生在編譯階段(步驟中). 在這個階段結束時, 產生一個連接函數.

2. 在第二步中, 連接階段(我們之前使用的階段), 產生前面的DOM樣板並連接到作用域. 同樣的, 任何觀察者或者監聽器都要根據需要添加, 在作用域和元素之前返回一個活動(雙向)綁定. 因此, 任何關聯到作用域的事情都發生在連接階段.

那麼在我們指令中發生了什麼呢? 讓我們去看一看, 好嗎?

`butterbar`指令可以像下面這樣使用:

	<div butterbar>My loading text...</div>

它基於前面隱藏的元素, 然後添加兩個監聽器到根作用域中. 當每次一個路由開始改變時, 它就顯示該元素(經由改變它的class[className]), 每次路由成功改變並完成時, 它再一次隱藏`butterbar`.

另一個有趣的事情是注意我們是如何注入`$rootScopr`到指令中的. 所有的指令都直接掛接到AngularJS的依賴注入系統, 因此你可以注入你的服務和其他任何需要的東西到其中.

最後需要注意的是處理元素的API. 使用jQuery的人會很高興, 因為他直到使用的是一個類似jQuery的語法(addClass, removeClass). AngularJS實現了一個呼叫jQuery的自己, 因此, 對於任何AngularJS項目來說, jQuery都是一個可選的依賴項. 如果你最終在你的項目中使用完整的jQuery庫, 你應該直到它使用的是它自己內置的jQlite實現.

第二個指令(focus)簡單得多. 它只是在當前元素上呼叫`focus()`方法. 你可以用過在任何input元素上添加`focus`屬性來呼叫它, 就像這樣:

	<input type="text" focus></input>

當頁面加載時, 元素將立即獲得焦點.

###控制器

隨著指令和服務的覆蓋, 我們終於可以進入控制器部分了, 我們有五個控制器. 所有的這些控制器都在一個單獨的文件中(`app/scripts/controllers/controllers.js`), 但是我們會一個個來瞭解它們. 讓我們來看第一個控制器, 這是一個列表控制器, 負責顯示系統中所有的食譜列表.

	app.controller('ListCtrl', ['scope', 'recipes', function($scope, recipes){
		$scope.recipes = recipes;
	}]);

注意列表控制器中最重要的一個事情: 在這個控制器中, 它並沒有連接到伺服器和取得是食譜. 相反, 它只是使用已經取得的食譜列表. 你可能不知道它是如何工作的. 你可能會使用路由一節來回答, 因為它有一個我們之前看到`MultiRecipeLoader`. 你只需要在腦海裡記住它.

在我們提到的列表控制器下, 其他的控制器都與之非常相似, 但我們仍然會逐步指出它們有趣的地方:

	app.controller('ViewCtrl', ['$scope', '$location', 'recipe', function($scope, $location, recipe){
			$scope.recipe = recipe;

			$scope.edit = function(){
				$location.path('/edit/' + recipe.id);
			};
	}]);

這個視圖控制器中有趣的部分是其編輯函數公開在作用域中. 而不是顯示和隱藏字段或者其他類似的東西, 這個控制器依賴於AngularJS來處理繁重的任務(你應該這麼做)! 這個編輯函數簡單的改變URL並跳轉到編輯食譜的部分, 你可以看見, AngularJS並沒有處理剩下的工作. AngularJS識別已經改變的URL並加載回應的視圖(這是與我們編輯模式中相同的食譜部分). 來看一看!

接下來, 讓我們來看看編輯控制器:

	app.controller('EditCtrl', ['$scope', '$location', 'recipe', function($scope, $location, recipe){
		$scope.recipe = recipe;

		$scope.save = function(){
			$scope.recipe.$save(function(recipe){
				$location.path('/view/' + recipe.id);
			});
		};

		$scope.remove = function(){
			delete $scope.recipe;
			$location.path('/');
		};
	}]);

那麼在這個暴露在作用域中的編輯控制器中新的`save`和`remove`方法有什麼.

那麼你希望作用域內的`save`函數做什麼. 它保存當前食譜, 並且一旦保存好, 它就在螢幕中將用戶重定向到相同的食譜. 回呼函數是非常有用的, 一旦你完成任務的情況下執行或者處理一些行為.

有兩種方式可以在這裡保存食譜. 一種是如代碼所示, 經由執行$scope.recipe.$save()方法. 這只是可能, 因為`recipe`是一個經由開頭部分的RecipeLoader返回的資源對像.

另外, 你可以像這樣來保存食譜:

	Recipe.save(recipe);

`remove`函數也是很簡單的, 在這裡它會從作用域中移除食譜, 同時將用戶重定向到主選單頁. 請注意, 它並沒有真正的從我們的伺服器上刪除它, 儘管它很再做出額外的呼叫.

接下來, 我們來看看New控制器:

	app.controller('NewCtrl', ['$scope', '$location', 'Recipe', function($scope, $location, Recipe){
		$scope.recipe = new Recipe({
			ingredents: [{}]
		});

		$scope.save = function(){
			$scope.recipe.$save(function(recipe){
				$location.path('/view/' + recipe.id);
			});
		};
	}]);

New控制器幾乎與Edit控制器完全一樣. 實際上, 你可以結合兩個控制器作為一個單一的控制器來做一個練習. 唯一的主要不同是New控制器會在第一步建立一個新的食譜(這也是一個資源, 因此它也有一個`save`函數).其他的一切都保持不變.

最後, 我們還有一個Ingredients控制器. 這是一個特殊的控制器, 在我們深入瞭解它為什麼或者如何特殊之前, 先來看一看它:

	app.controller('Ingredients', ['$scope', function($scope){
		$scope.addIngredients = function(){
			var ingredients = $scope.recipe.ingredients;
			ingredients[ingredients.length] = {};
		};

		$scope.removeIngredient = function(index) {
			$scope.recipe.ingredients.splice(index, 1);
		};
	}]);

到目前為止, 我們看到的所有其他控制器斗魚UI視圖上的相關部分聯繫著. 但是這個Ingredients控制器是特殊的. 它是一個子控制器, 用於在編輯頁面封裝特定的恭喜而不需要在外層(父級)來處理. 有趣的是要注意, 由於它是一個字控制器, 繼承自作用域中的父控制器(在這裡就是Edit/New控制器). 因此, 它可以訪問來自父控制器的`$scope.recipe`.

這個控制器本身並沒有什麼有趣或者獨特的地方. 它只是添加一個新的成份到現有的食譜成份數組中, 或者從食譜的成份列表中刪除一個特定的成份.

那麼現在, 我們就來完成最後的控制器. 唯一的JavaScript代碼塊展示了如何設定路由:

	// This file is app/scripts/controllers/controllers.js

	var app = angular.module('guthub', ['guthub.directives', 'guthub.services']);

	app.config(['$routeProvider', function($routeProvider){
		$routeProvider.
			when('/', {
				controller: 'ListCtrl',
				resolve: {
					recipes: function(MultiRecipeLoader) {
						return MultiRecipeLoader();
					}
				},
				templateUrl: '/views/list.html'
			}).when('/edit/:recipeId', {
				controller: 'EditCtrl',
				resolve: {
					recipe: function(RecipeLoader){
						return RecipeLoader();
					}
				},
				templateUrl: '/views/recipeForm.html'
			}).when('/view/:recipeId', {
				controller: 'ViewCtrl',
				resolve: {
					recipe: function(RecipeLoader){
						return RecipeLoader();
					}
				},
				templateUrl: '/views/viewRecipe.html'
			}).when('/new', {
					controller: 'NewCtrl',
					templateUrl: '/views/recipeForm.html'
			}).otherwise({redirectTo: '/'});
	}]);

正如我們所承諾的, 我們終於到瞭解析函數使用的地方. 前面的代碼設定Guthub AngularJS模組, 路由以及參與應用程式的樣板.

它掛接到我們已經建立的指令和服務上, 然後指定不同的路由指向應用程式的不同地方.

對於每個路由, 我們指定了URL, 它備份控制器, 加載樣板, 以及最後(可選的)提供了一個`resolve`對像.

這個`resolve`對像會告訴AngularJS, 每個resolve鍵需要在確定路由正確時才能顯示給用戶. 對我們來說, 我們想要加載所有的食譜或者個別的配方, 並確保在顯示頁面之前伺服器要回應我們. 因此, 我們要告訴路由提供者我們的食譜, 然後再告訴他如何來取得它.

這個環節中我們在第一節中定義了兩個服務, 分別時`MultiRecipeLoader`和`RecipeLoader`. 如果`resolve`函數返回一個AngularJS promise, 然後AngularJS會智能在獲得它之前等待promise解決問題. 這意味著它將會等待到伺服器回應.

然後將返回結果傳遞到構造函數中作為參數(與來自對像字段的參數的名稱一起作為參數).

最後, `otherwise`函數表示當沒有路由匹配時重定向到默認URL.

> 你可能會注意到Edit和New控制器兩個路由通向同一個樣板URL, `views/recipeForm.html`. 這裡發生了什麼呢? 我們復用了編輯樣板. 依賴於相關的控制器, 將不同的元素顯示在編輯食譜樣板中.

完成這些工作之後, 現在我們可以聚焦到樣板部分, 來看看控制器如何掛接到它們之上, 以及如何管理現實給最終用戶的內容.

##樣板

讓我們首先來看看最外層的主樣板, 這裡就是`index.html`. 這是我們單頁應用程式的基礎, 同時所有其他的視圖也會裝在到這個樣板的上下文中:

	<!DOCTYPE html>
	<html lang="en" ng-app="guthub">
	<head>
		<title>Guthub - Create and Share</title>
		<script src="scripts/vendor/angular.min.js"></script>
		<script src="scripts/vendor/angular-resource.min.js"></script>
		<script src="scripts/directives/directives.js"></script>
		<script src="scripts/services/services.js"></script>
		<script src="scripts/controlers/controllers.js"></script>
		<link rel="stylesheet" href="styles/bootstrap.css">
		<link rel="stylesheet" href="styles/guthub.css">
	</head>
	<body>
		<header>
			<h1>Guthub</h1>
		</header>
		<div butterbar>Loading...</div>

		<div class="container-fluid">
			<div class="row-fluid">
				<div class="span2">
					<!-- Sidebar -->
					<div class="focus"><a href="/#/new">New Recipe</a></div>
					<div><a href="/#/">Recipe List</a></div>
				</div>
				<div class="span10">
					<div ng-view></div>
				</div>
			</div>
		</div>
	</body>
	</html>

注意前面的樣板中有5個有趣的元素, 其中大部分你在第2章中都已經見過了. 讓我們逐個來看看它們:

`ng-app`

我們設定了`ng-app`模組為Guthub. 這與我們在`angular.module`函數中提供的模組名稱相同. 這便是AngularJS如何知道兩個掛接在一起的原因.

`script`標籤

這表示應用程式在哪裡加載AngularJS. 這必須在所有使用AngularJS的JS文件被加載之前完成. 理想的情況下, 它應該在body的底部完成(\</body\>之前).

`Butterbar`

我們第一次使用自定義指令. 在我們定義我們的`butterbar`指令之前, 我們希望將它用於一個元素, 以便在路由改變時顯示它, 在成功的時候隱藏它(loading...處理). 需要突出顯示這個元素的文字(在這裡我們使用了一個非常煩人的"Loading...").

鏈接的`href`值

`href`用於鏈接到我們單頁應用程式的各個頁面. 追它們如何使用#字符來確保頁面不會重載的, 並且相對於當前頁面. AngularJS會監控URL(只要頁面沒有重載), 然後在需要的時候起到神奇的作用(或者通常, 將這個非常煩人的路由管理定義為我們路由的一部分).

`ng-view`

這是最後一個神奇的傑作. 在我們的控制器一節, 我們定義了路由. 作為定義的一部分, 每個路由表示一個URL, 控制器關聯路由和一個樣板. 當AngularJS發現一個路由改變時, 它就會加載關聯的樣板, 並將控制器添加給它, 同時替換`ng-view`為該樣板的內容.

有一件引人注目的事情是這裡缺少`ng-controller`標籤. 大部分應用程式某種程度上都需要一個與外部樣板關聯的MainController. 其最常見的位置是在body標籤上. 在這種情況下, 我們並沒有使用它, 因為完整的外部樣板沒有AngularJS內容需要引用到一個作用域.

現在我們來看看與每個控制器關聯的單獨的樣板, 就從"食譜列表"樣板開始:

	<!-- File is chapter4/guthub/app/view/list.html -->
	<h3>Recipe List</h3>
	<ul class="recipes">
		<li ng-repeat="recipe in recipes">
			<div><a ng-href="/#/view/{{recipe.id}}">{{recipe.title}}</a></div>
		</li>
	</ul>

是的, 它是一個非常無聊(普通)的樣板. 這裡只有兩個有趣的點. 第一個是非常標準的`ng-repeat`標籤用法. 他會獲得作用域內的所有食譜並重複檢出它們.

第二個是`ng-href`標籤的用法而不是`href`屬性. 這是一個在AngularJS加載期間純粹無效的空白鏈接. `ng-href`會確保任何時候都不會給用戶呈現一個畸形的鏈接. 總是會使用它在任何時候使你的URLs都是動態的而不是靜態的.

當然, 你可能感到奇怪: 控制器在哪裡? 這裡沒有`ng-controller`定義, 也確實沒有Main Controller定義. 這是路由映射發揮的作用. 如果你還記得(或者往前翻幾頁), `/`路由會重定向到列表樣板並且帶有與之關聯的ListController. 因此, 當引用任何變數或者類似的東西時, 它都在List Controller作用域內部.

現在我們來看一些有更多實質內容的東西: 視圖形式.

	<!-- File is chapter4/guthub/app/views/viewRecipe.html -->
	<h2>{{recipe.title}}</h2>

	<div>{{recipe.decription}}</div>

	<h3>Ingredients</h3>
	<span ng-show="recipe.ingredients.length == 0">No Ingredients</span>
	<ul class="unstyled" ng-hide="recipe.ingredients.length == 0">
		<li ng-repeat="ingredient in recipe.ingredients">
			<span>{{ingredient.amount}}</span>
			<span>{{ingredient.amountUnits</span>
			<span>{{ingredient.ingredientName}}</span>
		</li>
	</ul>

	<h3>Instructions</h3>
	<div>{{recipe.instructions}}</div>

	<form ng-submit="edit()" class="form-horizontal">
		<div class="form-actions">
			<button class="btn btn-primary">Edit</button>
		</div>
	</form>

這是另一個不錯的, 很小的包含樣板. 我們將提醒你注意三件事, 雖然不會按照它們所出現的順序.

第一個就是非常標準的`ng-repeat`. 食譜(recipes)再次出現在View Controller作用域中, 這是用過在頁面現實給用戶之前經由`resolve`函數加載的. 這確保用戶查看它時也面不是一個破碎的, 未加載的狀態.

接下來一個有趣的用法是使用`ng-show`和`ng-class`(這裡應該是`ng-hide`)來設定樣板的樣式. `ng-show`標籤被添加到\<i\>標籤上, 這是用來顯示一個星號標記的icon. 現在, 這個星號標記只在食譜是一個特色食譜的時候才顯示(例如經由`recipe.featured`布爾值來標記). 理想的情況下, 為了確保適當的間距, 你需要使用一個空白的空格圖標, 並給這個空格圖標綁定`ng-hide`指令, 然後同歸同樣的AngularJS表達式`ng-show`來顯示. 這是一個常見的用法, 顯示一個東西並在給定的條件下來隱藏.

`ng-class`用於添加一個類(CSS類)給\<h2\>標籤(在這種情況下就是"特色")當食譜是一個特色食譜時. 它添加了一些特殊的高亮來使標題更加引人注目.

最後一個需要注意的時表單上的`ng-submit`指令. 這個指令規定在表單被提交的情況下呼叫`scope`中的`edit()`函數. 當任何沒有關聯明確函數的按鈕被點擊時機會提交表單(這種情況下便是Edit按鈕). 同樣, AngularJS足夠智能的在作用域中(從模組,路由,控制器中)在正確的時間裡引用和呼叫正確的方法.

> **上面這段解釋與原書代碼有一些差別, 讀者自行理解. 原書作者暫未給出解答.**

現在我們可以來看看我們最後的樣板(可能目前為止最複雜的一個), 食譜表單樣板:

	<!-- file is chapter4/guthub/app/views/recipeForm.html -->
	<h2>Edit Recipe</h2>
	<form name="recipeForm" ng-submit="save()" class="form-horizontal">
		<div class="control-group">
			<label class="control-label" for="title">Title:</label>
			<div class="controls">
				<input ng-model="recipe.title" class="input-xlarge" id="title" focus required>
			</div>
		</div>

		<div class="control-group">
			<label class="control-label" for="description">Description:</label>
			<div class="controls">
				<textarea ng-model="recipe.description" class="input-xlarge" id="description"></textarea>
			</div>
		</div>

		<div class="control-group">
			<label class="control-label" for="ingredients">Ingredients:</label>
			<div class="controls">
				<ul id="ingredients" class="unstyled" ng-controller="IngredientsCtrl">
				<li ng-repeat="ingredient in recipe.ingredients">
					<input ng-model="ingredient.amount" class="input-mini">
					<input ng-model="ingredient.amountUnits" class="input-small">
					<input ng-model="ingredient.ingredientName">
					<button type="button" class="btn btn-mini" ng-click="removeIngredient($index)"><i class="icon-minus-sign"></i> Delete </button>
				</li>
				<button type="button" class="btn btn-mini" ng-click="addIngredient()"><i class="icon-plus-sign"></i> Add </button>
			</ul>
			</div>
		</div>

		<div class="control-group">
			<label class="control-label" for="instructions">Instructions:</label>
			<div class="controls">
				<textarea ng-model="recipe.instructions" class="input-xxlarge" id="instructions"></textarea>
			</div>
		</div>

		<div class="form-actions">
			<button class="btn btn-primary" ng-disabled="recipeForm.$invalid">Save</button>
			<button type="button" ng-click="remove()" ng-show="!recipe.id" class="btn">Delete</button>
		</div>
	</form>

不要驚慌. 它看起來像很多代碼, 並且它時一個很長的代碼, 但是如果你認真研究以下它, 你會發現它並不是非常複雜. 事實上, 其中很多都是很簡單的, 比如重複的顯示可編輯輸入字段用於編輯食譜的樣板:

+ `focus`指令被添加到第一個輸入字段上(`title`輸入字段). 這確保當用戶導航到這個頁面時, 標題字段會自動聚焦, 並且用戶可以立即開始輸入標題訊息.

+ `ng-submit`指令與前面的例子非常相似, 因此我們不會深入討論它, 它只是保存是食譜的狀態和編輯過程的結束信號. 它會掛接到Edit Controller中的`save()`函數.

+ `ng-model`指令用於將不同的文字輸入框和文字域綁定到模型中.

在這個頁面更有趣的一方面, 並且我們建議你花一點之間來瞭解它的便是配料列表部分的`ng-controller`標籤. 讓我們花一分鐘的事件來瞭解以下這裡發生了什麼.

我們看到了一個顯示配料成份的列表, 並且容器標籤關聯了一個`ng-controller`. 這意味著這個`\<ul\>`標籤是Ingredients Controller的作用域. 但是這個樣板實際的控制器是什麼呢, 是Edit Controller? 事實證明, Ingredients Controller是作為Edit Controller的子控制器建立的, 從而繼承了Edit Controller的作用域. 這就是為什麼它可以從Edit Controller訪問食譜對像(recipe object)的原因.

此外, 它還增加了一個`addIngredient()`方法, 這是經由處理高亮的`ng-click`使用的, 那麼就只能在`\<ul\>`標籤作用域內訪問. 那麼為什麼你需要這麼做呢? 因為這是分離你擔憂的最好的方式. 為什麼Edit Controller需要一個`addIngredients()`方法, 問99%的樣板都不會關心它. 因為如此精確你的子控制器和嵌套控制器是很不錯的, 它可以包含任務並循序你分離你的業務邏輯到更多的可維護模組中.

+ 另外一個控制器便是我們在這裡想要深入討論的表單驗證控制. 它很容易在AngularJS中設定一個特定的表單字段為"必填項". 只需要簡單的添加required標籤到輸入框上(與前面的代碼中的情況一樣). 但是現在你要怎麼對它.

為此, 我們先跳到保存按鈕部分. 注意它上面有一個`ng-disabled`指令, 這換言之就是`recipeForm.$invalid`. 這個`recipeForm`是我們已經聲明的表單名稱. AngularJS增加了一些特殊的變數(`$valid`和`$invalid`只是其中兩個)允許你控制表單的元素. AngularJS會查找到所有的必填元素並更新所對應的特殊變數. 因此如果我們的Recipe Title為空, `recipeForm.$invalid`就會被這只為true(`$valid`就為false), 同時我們的保存(Save)按鈕就會立刻被禁用.

我們還可以給一個文字輸入框設定最大和最小長度(輸入長度), 以及一個用於驗證一個輸入字段的正則表達式模式. 另外, 這裡還有只在滿足特定條件時用於顯示特定錯誤消息的進階用法. 讓我們使用一個很小的分支例子來看看:

	<form name="myForm">
		User name: <input type="text" name="userName" ng-model="user.name" ng-minlength="3">
		<span class="error" ng-show="myForm.userName.$error.minlength">Too Short!</span>
	</form>

在前面的這個例子中, 我們添加了一要求: 用戶名至少是三個字符(經由使用`ng-minlength`指令). 現在, 表單範圍內會關心每個命名輸入框的填充形式--在這個例子中我們只有一個`userName`--其中每個輸入框都會有一個`$error`對像(這裡具體的還包括什麼樣的錯誤或者沒有錯誤: `required`, `minlength`, `maclength`或者模式)和一個`$valid`標籤來表示輸入框本身是否有效.

我們可以利用這個來選擇性的將錯誤訊息顯示給用戶, 這根據不用的輸入錯誤類型來顯示, 正如我們上面的實例所示.

跳回到我們原來的樣板中--Recipe表單樣板--在這裡的ingredients repeater裡面還有另外一個很好的`ng-show`高亮的用法. 這個Add Ingredient按鈕只會在最後的一個配料的旁邊顯示. 著可以經由在一個repeater元素範圍內呼叫一個`ng-show`並使用特殊的`$last`變數來完成.

最後我們還有最後的一個`ng-click`, 這是附加的第二個按鈕, 用於刪除該食譜. 注意這個按鈕只會在食譜尚未保存的時候顯示. 雖然通常它會編寫一個更有意義的`ng-hide="recipe.id", 有時候它會使用更有語義的`ng-show="!recipe.id". 也就是說, 如果食譜沒有一個id的時候顯示, 而不是在食譜有一個id的時候隱藏.

##測試

隨著控制器部分, 我們已經推遲向你顯示測試部分了, 但你知道它會即將到來, 不是嗎? 在這一節, 我們將會涵蓋你已經編寫部分的代碼測試, 以及涉及你要如何編寫它們.

###單元測試

第一個, 也是非常重要的一種測試是單元測試. 對於控制器(指令和服務)的測試你已經開發和編寫的正確的結構, 並且你可能會想到它們會做什麼.

在我們深入到各個單元測試之前, 讓我們圍繞所有我們的控制器單元測試來看看測試裝置:

	describle('Controllers', function() {
		var $scope, ctrl;
		//you need to include your module in a test
		beforeEach(module('guthub'));
		beforeEach(function() {
			this.addMatchers({
				toEqualData: function(expected) {
					return angular.equals(this.actual, expected);
				}
			});
		});

		describle('ListCtrl', function() {....});
		// Other controller describles here as well
	});

這個測試裝置(我們仍然使用Jasmine的行為方式來編寫這些測試)做了幾件事情:

1. 建立一個全域(至少對於這個測試規範是這個目的)可訪問的作用域和控制器, 所以我們不用擔心每個控制器會建立一個新的變數.

2. 初始化我們應用程式所用的模組(在這裡是Guthub).

3. 添加一個我們稱之為`equalData`的特殊的匹配器. 這基本上允許我們在資源對像(就像食譜)經由`$resource`服務和呼叫RESTful來執行斷言(測試判斷).

> 記得在任何我們處理在`ngRsource`上返回對象的斷言時添加一個稱為`equalData`特殊匹配器. 這是因為`ngRsource`返回對像還有額外的方法在它們失敗時默認希望呼叫equal方法.

這個裝置到此為止, 讓我們來看看List Controller的單元測試:

	describle('ListCtrl', function(){
		var mockBackend, recipe;
		// _$httpBackend_ is the same as $httpBackend. Only written this way to diiferentiate between injected variables and local variables
		breforeEach(inject(function($rootScope, $controller, _$httpBackend_, Recipe) {
			recipe = Recipe;
			mockBackend = _$httpBackend_;
			$scope = $rootScope.$new();
			ctrl = $controller('ListCtrl', {
				$scope: $scope,
				recipes: [1, 2, 3]
			});
		}));

		it('should have list of recipes', function() {
			expect($scope.recipes).toEqual([1, 2, 3]);
		});
	});

記住這個List Controller只是我們最簡單的控制器之一. 這個控制器的構造器只是接受一個食譜列表並將它保存到作用域中. 你可以編寫一個測試給它, 但它似乎有一點不合理(我們還是這麼做了, 因為這個測試很不錯).

相反, 更有趣的是MulyiRecipeLoader服務方面. 它負責從伺服器上取得食譜列表並將它作為一個參數傳遞(當經由`$route`服務正確的連接時).

	describe('MultiRecipeLoader', function() {
		var mockBackend, recipe, loader;
		// _$httpBackend_ is the same as $httpBackend. Only written this way to differentiate between injected variables and local variables. 

		beforeEach(inject(function(_$httpBackend_, Recipe, MultiRecipeLoader) {
			recipe = Recipe;
			mockBackend = _$httpBackend_;
			loader = MultiRecipeLoader;
		}));

		it('should load list of recipes', function() { 
			mockBackend.expectGET('/recipes').respond([{id: 1}, {id: 2}]);

			var recipes;

			var promise = loader(); promise.then(function(rec) {
				recipes = rec;
			});

			expect(recipes).toBeUndefined( ) ;

			mockBackend. f lush() ;

			expect(recipes).toEqualData([{id: 1}, {id: 2}]); });
	});
	// Other controller describes here as well

在我們的測試中, 我們經由掛接到一個模擬的`HttpBackend`來測試MultiRecipeLoader. 這來自於測試運行時所包含的`angular-mocks.js`文件. 只需將它注入到你的`beforeEach`方法中就足以讓你設定預期目的. 接下來, 我們進行了一個更有意義的測試, 我們期望設定一個伺服器的GET請求來取得recipes, 浙江返回一個簡單的數組對像. 然後使用我們新的自定義的匹配器來確保正確的返回資料. 注意在模擬backend中的`flush()`呼叫, 這將告訴模擬Backend從伺服器返迴回應. 你可以使用這個機制來測試控制流程和查看你的應用程式在伺服器返回一個回應之前和之後是如何處理的.

我們將跳過View Controller, 因為它除了在作用域中添加一個`edit()`方法之外基於與List Controller一模一樣. 這是非常簡單的測試, 你可以在你的測試中注入`$location`並檢查它的值.

現在讓我們跳到Edit Controller, 其中有兩個有趣的點我們進行單元測試. 一個是類似我們之前看到過的`resolve`函數, 並且可以以同樣的方式測試. 相反, 我們現在想看看我們可以如和測試`save()`和`remove()`方法. 讓我們來看看對於它們的測試(假設我們的測試工具來自於前面的例子):

	describle('EditController', function() {
		var mockBackend, location;
		beforeEach(inject($rootScope, $controller, _$httpBackend_, $location, Recipe){
			mockBackend = _$httpBackend_;
			location = $location;
			$scope = $rootScope.$new();

			ctrl = $controller('EditCtrl', {
				$scope: $scope,
				$location: $location,
				recipe: new Recipe({id: 1, title: 'Recipe'});
			});
		}));

		it('should save the recipe', function(){
			mockBackend.expectPOST('/recipes/1', {id: 1, title: 'Recipe'}).respond({id: 2});

			// Set it to something else to ensure it is changed during the test
			location.path('test');

			$scope.save();
			expect(location.path()).toEqual('/test');

			mockBackend.flush();

			expect(location.path()).toEqual('/view/2');
		});

		it('should remove the recipe', function(){
			expect($scope.recipe).toBeTruthy();
			location.path('test');

			$scope.remove();

			expect($scope.recipe).toBeUndefined();
			expect(location.path()).toEqual('/');
		});
	});

在第一個測試用, 我們測試了`save()`函數. 特別是, 我們確保在我們的對象保存時首先建立一個到伺服器的POST請求, 然後, 一旦伺服器回應, 地址就改變到新的持久對象的視圖食譜頁面.

第二個測試更簡單. 我們進行了簡單的檢測以確保在作用域中呼叫`remove()`方法的時候移除當前食譜, 然後重定向到用戶主頁. 這可以很容易經由注入`$location`服務到我們的測試中並使用它.

其餘的針對控制器的單元測試遵循非常相似的模式, 因此在這裡我們跳過它們. 在他們的底層中, 這些單元測試依賴於一些事情:

+ 確保控制器(或者更可能是作用域)在結束初始化時達到正確的狀態

+ 確認經行正確的伺服器呼叫, 以及經由作用域在伺服器呼叫期間和完成後去的正確的狀態(經由在單元測試中使用我們的模擬後端服務)

+ 利用AngularJS的依賴注入框架著手處理元素以及控制器對像用於確保控制器會設定正確的狀態.

###腳本測試

一旦我們對單元測試很滿意, 我們可能禁不住的往後靠一下, 抽根雪茄, 收工. 但是AngularJS開發者不會這麼做, 直到他們完成了他們的腳本測試(場景測試). 雖然單元測試確保我們的每一塊JS代碼都按照預期工作, 我們也要確保樣板加載, 並正確的掛接到控制器上, 以及在樣板重點擊做正確的事情.

這正是AngularJS帶給你的腳本測試(場景測試), 它允許你做以下事情:

+ 加載你的應用程式

+ 瀏覽一個特定的頁面

+ 隨意的點擊周圍和輸入文字

+ 確保發生正確的事情

所以, 腳本測試如何在我們的"食譜列表"頁面工作? 首先, 在我們開始實際的測試之前, 我們需要做一些基礎工作.

對於該腳本測試工作, 我們需要一個工作的Web伺服器以準備從Guthub應用上接受請求, 同時將允許我們從它上面存儲和取得一個食譜列表. 隨意的更改代碼以使用內存中的食譜列表(移除`$resource`食譜並只是將它轉換為一個JSON對像), 或者復用和修改我們前面章節向你展示的Web伺服器, 或者使用Yeoman!

一旦我們有了一個伺服器並運行起來, 同時服務於我們的應用程式, 然後我們就可以編寫和運行下面的測試:

	describle('Guthub App', function(){
		it('should show a list of recipes', function(){
			browser().navigateTo('/index.html');
			//Our Default Guthub recipes list has two recipes
			expect(repeater('.recipes li').count()).toEqual(2);
		});
	});
