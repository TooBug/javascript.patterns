# 对象创建模式

在JavaScript中创建对象是件很容易的事情，直接通过对象字面量或者构造函数就可以。本章将在此基础上介绍一些常用的对象创建模式。

JavaScript语言本身很简单、直观，也没有其他语言的一些语言特性：命名空间、模块、包、私有属性以及静态成员。本章将介绍一些常用的模式，以此实现这些语言特性。

我们将对命名空间、依赖声明、模块模式以及沙箱模式进行初探——它们可以帮助我们更好地组织应用程序的代码，有效地减少全局污染的问题。除此之外，还会讨论私有和特权成员、静态和私有静态成员、对象常量、链式调用以及一种像类式语言一样定义构造函数的方法等话题。  
/Users/TooBug/github/javascript.patterns/chapter5.markdown
## 命名空间模式

使用命名空间可以减少全局变量的数量，与此同时，还能有效地避免命名冲突和前缀的滥用。

JavaScript没有原生的命名空间语法，但很容易可以实现这个特性。为了避免产生全局污染，你可以为应用或者类库创建一个（通常是唯一一个）全局对象，然后将所有的功能都添加到这个对象上，而不是到处声明大量的全局函数、全局对象以及其他的全局变量。

看如下例子：

	// 重构前：5个全局变量
	// 注意：反模式
	// 构造函数
	function Parent() {} 
	function Child() {}
	// 一个变量
	var some_var = 1;

	// 一些对象
	var module1 = {}; 
	module1.data = {a: 1, b: 2}; 
	var module2 = {};

可以通过创建一个全局对象（通常代表应用名）比如`MYAPP`来重构上述这类代码，然后将上述例子中的函数和变量都变为该全局对象的属性：

	// 重构后：一个全局变量
	// 全局对象
	var MYAPP = {};

	// 构造函数
	MYAPP.Parent = function () {}; 
	MYAPP.Child = function () {};

	// 一个变量
	MYAPP.some_var = 1;

	// 一个对象容器
	MYAPP.modules = {};

	// 嵌套的对象
	MYAPP.modules.module1 = {}; 
	MYAPP.modules.module1.data = {a: 1, b: 2}; 
	MYAPP.modules.module2 = {};

这里的`MYAPP`就是命名空间对象，对象名可以随便取，可以是应用名、类库名、域名或者是公司名都可以。开发者经常约定全局变量都采用大写（所有字母都大写），这样可以显得比较突出（不过要记住，大写的变量也常用于表示常量）。

这种模式是一种很好的提供命名空间的方式，避免了自身代码的命名冲突，同时还避免了同一个页面上自身代码和第三方代码（比如JavaScript类库或者widget）的冲突。这种模式在大多数情况下非常适用，但也有它的缺点：

- 代码量稍有增加；在每个函数和变量前加上这个命名空间对象的前缀，会增加代码量，增大文件大小
- 该全局实例可以被随时修改
- 命名的深度嵌套会减慢属性值的查询

本章后续要介绍的沙箱模式则可以避免这些缺点。

### 通用命名空间函数

随着程序复杂度的提高，代码会被分拆在不同的文件中以按照页面需要来加载，这样一来，就不能保证你的代码一定是第一个定义命名空间或者某个属性的，甚至会发生属性覆盖的问题。所以，在创建命名空间或者添加属性的时候，最好先检查下是否存在，如下所示：

	// 不安全的做法
	var MYAPP = {};
	// 更好的做法
	if (typeof MYAPP === "undefined") {
		var MYAPP = {}; 
	}
	// 简写
	var MYAPP = MYAPP || {};

如上所示，如果每次做类似操作都要这样检查一下就会有很多重复的代码。例如，要声明`MYAPP.modules.module2`，就要重复三次这样的检查。所以，我们需要一个可复用的`namespace()`函数来专门处理这些检查工作，然后用它来创建命名空间，如下所示：

	// 使用命名空间函数
	MYAPP.namespace('MYAPP.modules.module2');

	// 等价于：
	// var MYAPP = {
	// 	modules: {
	// 		module2: {}
	// 	}
	// };

下面是上述`namespace`函数的实现示例。这种实现是非破坏性的，意味着如果要创建的命名空间已经存在，则不会再重复创建：

	var MYAPP = MYAPP || {};
	MYAPP.namespace = function (ns_string) { 
		var parts = ns_string.split('.'),
			parent = MYAPP, 
			i;

		// 去除不必要的全局变量层
		// 译注：因为namespace已经属于MYAPP
		if (parts[0] === "MYAPP") {
			parts = parts.slice(1); 
		}

		for (i = 0; i < parts.length; i += 1) {
			// 如果属性不存在则创建它
			if (typeof parent[parts[i]] === "undefined") {
				parent[parts[i]] = {}; 
			}
			parent = parent[parts[i]];
		}
		return parent;
	};

上述实现支持如下几种用法：

	// 将返回值赋给本地变量
	var module2 = MYAPP.namespace('MYAPP.modules.module2'); 
	module2 === MYAPP.modules.module2; // true

	// 省略全局命名空间`MYAPP`
	MYAPP.namespace('modules.module51');

	// 长命名空间
	MYAPP.namespace('once.upon.a.time.there.was.this.long.nested.property');

图5-1 展示了上述代码创建的命名空间对象在Firebug下的可视化结果

![MYAPP命名空间在Firebug下的可视结果](./Figure/chapter5/5-1.jpg)

图5-1 MYAPP命名空间在Firebug下的可视化结果

## 依赖声明

JavaScript库往往是模块化而且有用到命名空间的，这使得你可以只使用你需要的模块。比如在YUI2中，全局变量`YAHOO`就是一个命名空间，各个模块都是全局变量的属性，比如`YAHOO.util.Dom`（DOM模块）、`YAHOO.util.Event`（事件模块）。

将你的代码依赖在函数或者模块的顶部进行声明是一个好主意。声明就是创建一个本地变量，指向你需要用到的模块：

	var myFunction = function () {
		// 依赖
		var event = YAHOO.util.Event,
			dom = YAHOO.util.Dom;

		// 在函数后面的代码中使用event和dom……
	};

这是一个相当简单的模式，但是有很多的好处：

- 明确的依赖声明是告知使用你代码的开发者，需要保证指定的脚本文件被包含在页面中。
- 将声明放在函数顶部使得依赖很容易被查找和解析。
- 本地变量（如`dom`）永远会比全局变量（如`YAHOO`）要快，甚至比全局变量的属性（如`YAHOO.util.Dom`）还要快，这样会有更好的性能。使用了依赖声明模式之后，全局变量的解析在函数中只会进行一次，在此之后将会使用更快的本地变量。
- 一些高级的代码压缩工具比如YUI Compressor和Google Closure compiler会重命名本地变量（比如`event`可能会被压缩成一个字母，如`A`），这会使代码更精简，但这个操作不会对全局变量进行，因为这样做不安全。

下面的代码片段是关于是否使用依赖声明模式对压缩影响的展示。尽管使用了依赖声明模式的`test2()`看起来复杂，因为需要更多的代码行数和一个额外的变量，但在压缩后它的代码量却会更小，意味着用户只需要下载更少的代码：

	function test1() {
		alert(MYAPP.modules.m1);
		alert(MYAPP.modules.m2);
		alert(MYAPP.modules.m51);
	}

	/*
	test1()压缩后的函数体：
	alert(MYAPP.modules.m1);alert(MYAPP.modules.m2);alert(MYAPP.modules.m51)
	*/

	function test2() {
		var modules = MYAPP.modules;
		alert(modules.m1);
		alert(modules.m2);
		alert(modules.m51);
	}

	/*
	test2()压缩后的函数体：
	var a=MYAPP.modules;alert(a.m1);alert(a.m2);alert(a.m51)
	*/


## 私有属性和方法

JavaScript不像Java或者其它语言，它没有专门的提供私有、保护、公有属性和方法的语法。所有的对象成员都是公有的：

	var myobj = {
		myprop: 1,
		getProp: function () {
			return this.myprop;
		}
	};
	console.log(myobj.myprop); // myprop是公有的
	console.log(myobj.getProp()); // getProp()也是公有的

当你使用构造函数创建对象的时候也是一样的，所有的成员都是公有的：

	function Gadget() {
		this.name = 'iPod';
		this.stretch = function () {
			return 'iPad';
		};
	}
	var toy = new Gadget();
	console.log(toy.name); // name是公有的
	console.log(toy.stretch()); // stretch()也是公有的

### 私有成员

尽管语言并没有用于私有成员的专门语法，但你可以通过闭包来实现。在构造函数中创建一个闭包，任何在这个闭包中的部分都不会暴露到构造函数之外。但是，这些私有变量却可以被公有方法访问，也就是在构造函数中定义的并且作为返回对象一部分的那些方法。我们来看一个例子，`name`是一个私有成员，在构造函数之外不能被访问：

	function Gadget() {
		// 私有成员
		var name = 'iPod';
		// 公有函数
		this.getName = function () {
			return name;
		};
	}
	var toy = new Gadget();

	// name是是私有的
	console.log(toy.name); // undefined
	// 公有方法可以访问到name
	console.log(toy.getName()); // "iPod"

如你所见，在JavaScript创建私有成员很容易。你需要做的只是将私有成员放在一个函数中，保证它是函数的本地变量，也就是说让它在函数之外不可以被访问。

### 特权方法

特权方法的概念不涉及到任何语法，它只是一个给可以访问到私有成员的公有方法的名字（就好像它们有更多权限一样）。

在前面的例子中，`getName()`就是一个特权方法，因为它有访问`name`属性的特殊权限。

### 私有成员失效

当你使用私有成员时，需要考虑一些极端情况：

- 在Firefox的一些早期版本中，允许通过给`eval()`传递第二个参数的方法来指定上下文对象，从而允许访问函数的私有作用域。比如在Mozilla Rhino（译注：一个JavaScript引擎）中，允许使用`__parent__`来访问私有作用域。这些极端情况现在并没有广泛存在于浏览器中。
- 当你直接通过特权方法返回一个私有变量，而这个私有变量恰好是一个对象或者数组时，外部的代码可以修改这个私有变量，因为它是按引用传递的。

我们来看一下第二种情况。下面的`Gadget`的实现看起来没有问题：

	function Gadget() {
		// 私有成员
		var specs = {
			screen_width: 320,
			screen_height: 480,
			color: "white"
		};

		// 公有函数
		this.getSpecs = function () {
			return specs;
		};
	}

这里的问题是`getSpecs()`返回了一个`specs`对象的引用。这使得`Gadget()`的使用者可以修改貌似隐藏起来的私有成员`specs`：

	var toy = new Gadget(),
		specs = toy.getSpecs();

	specs.color = "black";
	specs.price = "free";

	console.dir(toy.getSpecs());

在Firebug控制台中打印出来的结果如图5-2：

![图5-2 私有对象被修改了](./Figure/chapter5/5-2.jpg)

图5-2 私有对象被修改了

这个问题有点出乎意料，解决方法就是不要将你想保持私有的对象或者数组的引用传递出去。达到这个目标的一种方法是让`getSpecs()`返回一个新对象，这个新对象只包含对象的使用者需要的数据。这也是众所周知的“最低授权原则”（Principle of Least Authority，简称POLA），指永远不要给出比真实需要更多的东西。在这个例子中，如果`Gadget()`的使用者关注它是否适应一个特定的盒子，它只需要知道尺寸即可。所以你应该创建一个`getDimensions()`，用它返回一个只包含`width`和`height`的新对象，而不是把什么都给出去。也就是说，也许你根本不需要实现`getSpecs()`方法。

当你需要传递所有的数据时，有另外一种方法，就是使用通用的对象复制函数创建`specs`对象的一个副本。下一章提供了两个这样的函数——一个叫`extend()`，它会浅复制一个给定的对象（只复制顶层的成员），另一个叫`extendDeep()`，它会做深复制，遍历所有的属性和嵌套的属性。

### 对象字面量和私有成员

到目前为止，我们只看了使用构建函数创建私有成员的示例。如果使用对象字面量创建对象时会是什么情况呢？是否有可能含有私有成员？

如你前面所看到的那样，私有数据使用一个函数来包裹。所以在使用对象字面量时，你也可以使用一个即时函数创建的闭包。例如：

	var myobj; // 一个对象
	(function () {
		// 私有成员
		var name = "my, oh my";

		// 实现公有部分，注意没有var
		myobj = {
			// 特权方法
			getName: function () {
				return name;
			}
		};
	}());

	myobj.getName(); // "my, oh my"

还有一个原理一样但看起来不一样的实现示例：

	var myobj = (function () {
		// 私有成员
		var name = "my, oh my";

		// 实现公有部分
		return {
			getName: function () {
				return name;
			}
		};
	}());

	myobj.getName(); // "my, oh my"

这个例子也是所谓的“模块模式”的基础，我们稍后将讲到它。

### 原型和私有成员

使用构造函数创建私有成员的一个弊端是，每一次调用构造函数创建对象时这些私有成员都会被创建一次。

这对在构建函数中添加到`this`的成员来说是一个问题。为了避免重复劳动，节省内存，你可以将共用的属性和方法添加到构造函数的`prototype`（原型）属性中。这样的话这些公共的部分会在使用同一个构造函数创建的所有实例中共享。你也同样可以在这些实例中共享私有成员，甚至可以将两种模式联合起来达到这个目的，同时使用构造函数中的私有属性和对象字面量中的私有属性。因为`prototype`属性也只是一个对象，可以使用对象字面量创建。

这是一个示例：

	function Gadget() {
		// 私有成员
		var name = 'iPod';
		// 公有函数
		this.getName = function () {
			return name;
		};
	}

	Gadget.prototype = (function () {
		// 私有成员
		var browser = "Mobile Webkit";
		// 公有函数
		return {
			getBrowser: function () {
				return browser;
			}
		};
	}());

	var toy = new Gadget();
	console.log(toy.getName()); // 自有的特权方法 
	console.log(toy.getBrowser()); // 来自原型的特权方法

### 将私有函数暴露为公有方法

“暴露模式”是指将已经有的私有函数暴露为公有方法，它在你希望尽量保护对象内的一些方法不被外部修改干扰的时候很有用。你希望能提供一些功能给外部访问，因为它们会被用到，如果你把这些方法公开，就会使得它们不再健壮，因为你的API的使用者可能修改它们。在ECMAScript5中，你可以选择冻结一个对象，但在之前的版本中这种方法不可用。下面进入暴露模式（原来是由Christian Heilmann创造的模式，叫“暴露模块模式”）。

我们来看一个例子，它建立在对象字面量的私有成员模式之上：

	var myarray;

	(function () {

		var astr = "[object Array]",
			toString = Object.prototype.toString;

		function isArray(a) {
			return toString.call(a) === astr;
		}

		function indexOf(haystack, needle) {
			var i = 0,
				max = haystack.length;
			for (; i < max; i += 1) {
				if (haystack[i] === needle) {
					return i;
				}
			}
			return −1;
		}

		myarray = {
			isArray: isArray,
			indexOf: indexOf,
			inArray: indexOf
		};

	}());

这里有两个私有变量（私有函数）——`isArray()`和`indexOf()`。在包裹函数的最后，用那些允许被从外部访问的函数填充`myarray`对象。在这个例子中，同一个私有函数	`indexOf()`同时被暴露为ECMAScript5风格的`indexOf()`和PHP风格的`inArry()`。测试一下`myarray`对象：

	myarray.isArray([1,2]); // true
	myarray.isArray({0: 1}); // false
	myarray.indexOf(["a", "b", "z"], "z"); // 2
	myarray.inArray(["a", "b", "z"], "z"); // 2

现在假如有一些意外的情况发生在暴露的`indexOf()`方法上，私有的`indexOf()`方法仍然是安全的，因此`inArray()`仍然可以正常工作：

	myarray.indexOf = null;
	myarray.inArray(["a", "b", "z"], "z"); // 2

## 模块模式

模块模式使用得很广泛，因为它可以为代码提供特定的结构，帮助组织日益增长的代码。不像其它语言，JavaScript没有专门的“包”（package）的语法，但模块模式提供了用于创建独立解耦的代码片段的工具，这些代码可以被当成黑盒，当你正在写的软件需求发生变化时，这些代码可以被添加、替换、移除。

模块模式是我们目前讨论过的好几种模式的组合，即：

- 命名空间模式
- 即时函数模式
- 私有和特权成员模式
- 依赖声明模式

第一步是初始化一个命名空间。我们使用本章前面部分的`namespace()`函数，创建一个提供数组相关方法的套件模块：

	MYAPP.namespace('MYAPP.utilities.array');

下一步是定义模块。使用一个即时函数来提供私有作用域供私有成员使用。即时函数返回一个对象，也就是带有公有接口的真正的模块，可以供其它代码使用：

	MYAPP.utilities.array = (function () {
		return {
			// todo...
		};
	}());

下一步，给公有接口添加一些方法：

	MYAPP.utilities.array = (function () {
		return {
			inArray: function (needle, haystack) {
				// ...
			},
			isArray: function (a) {
				// ...
			}
		};
	}());

如果需要的话，你可以在即时函数提供的闭包中声明私有属性和私有方法。同样，依赖声明放置在函数顶部，在变量声明的下方可以选择性地放置辅助初始化模块的一次性代码。函数最终返回的是一个包含模块公共API的对象：

	MYAPP.namespace('MYAPP.utilities.array');
	MYAPP.utilities.array = (function () {
	
			// 依赖声明
		var uobj = MYAPP.utilities.object,
			ulang = MYAPP.utilities.lang,

			// 私有属性
			array_string = "[object Array]",
			ops = Object.prototype.toString;

			// 私有方法
			// ……

			// 结束变量声明

		// 选择性放置一次性初始化的代码
		// ……

		// 公有API
		return {
	
			inArray: function (needle, haystack) {
				for (var i = 0, max = haystack.length; i < max; i += 1) {
					if (haystack[i] === needle) {
						return true;
					}
				}
			},

			isArray: function (a) {
				return ops.call(a) === array_string;
			}
			// ……更多的方法和属性
		};
	}());

模块模式被广泛使用，是一种值得强烈推荐的模式，它可以帮助我们组织代码，尤其是代码量在不断增长的时候。

### 暴露模块模式

我们在本章中讨论私有成员模式时已经讨论过暴露模式。模块模式也可以用类似的方法来组织，将所有的方法保持私有，只在最后暴露需要使用的方法来初始化API。

上面的例子可以变成这样：

	MYAPP.utilities.array = (function () {

			// 私有属性
		var array_string = "[object Array]",
			ops = Object.prototype.toString,

			// 私有方法
			inArray = function (haystack, needle) {
				for (var i = 0, max = haystack.length; i < max; i += 1) {
					if (haystack[i] === needle) {
						return i;
					}
				}
				return −1;
			},
			isArray = function (a) {
				return ops.call(a) === array_string;
			};
			// 结束变量定义

		// 暴露公有API
		return {
			isArray: isArray,
			indexOf: inArray
		};
	}());

### 创建构造函数的模块

前面的例子创建了一个对象`MYAPP.utilities.array`，但有时候使用构造函数来创建对象会更方便。你也可以同样使用模块模式来做。唯一的区别是包裹模块的即时函数会在最后返回一个函数，而不是一个对象。

看下面的模块模式的例子，创建了一个构造函数`MYAPP.utilities.Array`：

	MYAPP.namespace('MYAPP.utilities.Array');

	MYAPP.utilities.Array = (function () {

			// 依赖声明
		var uobj = MYAPP.utilities.object,
			ulang = MYAPP.utilities.lang,

			// 私有属性和方法……
			Constr;

			// 结束变量定义

		// 选择性放置一次性初始化代码
		// ……

		// 公有API——构造函数
		Constr = function (o) {
			this.elements = this.toArray(o);
		};
		// 公有API——原型
		Constr.prototype = {
			constructor: MYAPP.utilities.Array,
			version: "2.0",
			toArray: function (obj) {
				for (var i = 0, a = [], len = obj.length; i < len; i += 1) {
					a[i] = obj[i];
				}
				return a;
			}
		};

		// 返回构造函数
		return Constr;

	}());

像这样使用这个新的构造函数：

	var arr = new MYAPP.utilities.Array(obj);

### 在模块中引入全局上下文

作为这种模式的一个常见的变种，你可以给包裹模块的即时函数传递参数。你可以传递任何值，但通常情况下会传递全局变量甚至是全局对象本身。引入全局上下文可以加快函数内部的全局变量的解析，因为引入之后会作为函数的本地变量：

	MYAPP.utilities.module = (function (app, global) {

		// 全局对象和全局命名空间都作为本地变量存在
		
	}(MYAPP, this));

## 沙箱模式

沙箱模式主要着眼于命名空间模式的短处，即：

- 依赖一个全局变量成为应用的全局命名空间。在命名空间模式中，没有办法在同一个页面中运行同一个应用或者类库的不同版本，因为它们都会需要同一个全局变量名，比如`MYAPP`。
- 代码中以点分隔的名字比较长，无论写代码还是解析都需要处理这个很长的名字，比如`MYAPP.utilities.array`。

顾名思义，沙箱模式为模块提供了一个环境，模块在这个环境中的任何行为都不会影响其它的模块和其它模块的沙箱。

这个模式在YUI3中用得很多，但是需要记住的是，下面的讨论只是一些示例实现，并不讨论YUI3中的沙箱是如何实现的。

### 全局构造函数

在命名空间模式中 ，有一个全局对象，而在沙箱模式中，唯一的全局变量是一个构造函数，我们把它命名为`Sandbox()`。我们使用这个构造函数来创建对象，同时也要传入一个回调函数，这个函数会成为代码运行的独立空间。

使用沙箱模式是像这样：

	new Sandbox(function (box) {
		// 你的代码……
	});

`box`对象和命名空间模式中的`MYAPP`类似，它包含了所有你的代码需要用到的功能。

我们要多做两件事情：

- 通过一些手段（第3章中的强制使用`new`的模式），你可以在创建对象的时候不要求一定有`new`。
- 让`Sandbox()`构造函数可以接受一个（或多个）额外的配置参数，用于指定这个对象需要用到的模块名字。我们希望代码是模块化的，因此绝大部分`Sandbox()`提供的功能都会被包含在模块中。

有了这两个额外的特性之后，我们来看一下实例化对象的代码是什么样子。

你可以在创建对象时省略`new`并像这样使用已有的`ajax`和`event`模块：

	Sandbox(['ajax', 'event'], function (box) {
		// console.log(box);
	});

下面的例子和前面的很像，但是模块名字是作为独立的参数传入的：

	Sandbox('ajax', 'dom', function (box) {
		// console.log(box);
	});

使用通配符“＊”来表示“使用所有可用的模块”是个不错的想法，为了方便，我们也假设没有任何模块传入时，沙箱使用“＊”。所以有两种使用所有可用模块的方法：

	Sandbox('*', function (box) {
		// console.log(box);
	});

	Sandbox(function (box) {
		// console.log(box);
	});

下面的例子展示了如何实例化多个沙箱对象，你甚至可以将它们嵌套起来而互不影响：

	Sandbox('dom', 'event', function (box) {

		// 使用dom和event模块
	
		Sandbox('ajax', function (box) {
			// 另一个沙箱中的box，这个box和外面的box不一样

			//...

			// 使用ajax模块的代码到此为止

		});

		// 这里的代码与ajax模块无关
	});

从这些例子中看到，使用沙箱模式可以通过将代码包裹在回调函数中的方式来保护全局命名空间。

如果需要的话，你也可以利用函数也是对象这一事实，将一些数据作为静态属性存放到`Sandbox()`构造函数。

最后，你可以根据需要的模块类型创建不同的实例，这些实例都是相互独立的。

现在我们来看一下如何实现`Sandbox()`构造函数和它的模块来支持上面讲到的所有功能。

### 添加模块

在动手实现构造函数之前，我们先来看一下如何添加模块。

`Sandbox()`构造函数也是一个对象，所以可以给它添加一个`modules`静态属性。这个属性也是一个包含名值（key-value）对的对象，其中key是模块的名字，value是模块的功能实现。

	Sandbox.modules = {};

	Sandbox.modules.dom = function (box) {
		box.getElement = function () {};
		box.getStyle = function () {};
		box.foo = "bar";
	};

	Sandbox.modules.event = function (box) {
		// 如果有需要的话可以访问Sandbox的原型
		// box.constructor.prototype.m = "mmm";
		box.attachEvent = function () {};
		box.dettachEvent = function () {};
	};

	Sandbox.modules.ajax = function (box) {
		box.makeRequest = function () {};
		box.getResponse = function () {};
	};

在这个例子中我们添加了`dom`、`event`和`ajax`模块，这些模块在每个类库或者复杂的web应用中都很常见。

每个模块功能函数接受一个实例`box`作为参数，并给这个实例添加属性和方法。

### 实现构造函数

最后，我们来实现`Sandbox()`构造函数（你可能会很自然地想将这类构造函数命名为对你的类库或者应用有意义的名字）：

	function Sandbox() {
		// 将参数转换为数组
		var args = Array.prototype.slice.call(arguments),
			// 最后一个参数是回调函数
			callback = args.pop(),
			// 参数可以作为数组或者单独的参数传递
			modules = (args[0] && typeof args[0] === "string") ? args : args[0], i;

		// 保证函数是作为构造函数被调用
		if (!(this instanceof Sandbox)) {
			return new Sandbox(modules, callback);
		}

		// 根据需要给this添加属性
		this.a = 1;
		this.b = 2;

		// 给this对象添加模块
		// 未指明模块或者*都表示“使用所有模块”
		if (!modules || modules === '*') {
			modules = [];
			for (i in Sandbox.modules) {
				if (Sandbox.modules.hasOwnProperty(i)) {
					modules.push(i);
				}
			}
		}

		// 初始化指定的模块
		for (i = 0; i < modules.length; i += 1) {
			Sandbox.modules[modules[i]](this);
		}

		// 调用回调函数
		callback(this);
	}

	// 需要添加在原型上的属性
	Sandbox.prototype = {
		name: "My Application",
		version: "1.0",
		getName: function () {
			return this.name;
		}
	};

这个实现中的一些关键点：

- 有一个检查`this`是否是`Sandbox()`实例的过程，如果不是（也就是调用`Sandbox()`时没有加`new`），我们将这个函数作为构造函数再调用一次。
- 你可以在构造函数中给`this`添加属性，也可以给构造函数的原型添加属性。
- 被依赖的模块可以以数组的形式传递，也可以作为单独的参数传递，甚至以`*`通配符（或者省略）来表示加载所有可用的模块。值得注意的是，我们在这个示例实现中并没有考虑从外部文件中加载模块，但明显这是一个值得考虑的事情。比如YUI3就支持这种情况，你可以只加载最基本的模块（作为“种子”），其余需要的任何模块都通过将模块名和文件名对应的方式从外部文件中加载。
- 当我们知道依赖的模块之后就初始化它们，也就是调用实现每个模块的函数。
- 构造函数的最后一个参数是回调函数。这个回调函数会在最后使用新创建的实例来调用。事实上这个回调函数就是用户的沙箱，它被传入一个`box`对象，这个对象包含了所有依赖的功能。

## 静态成员

静态属性和方法是指那些在所有的实例中都一样的成员。在基于类的语言中，静态成员是用专门的语法来创建，使用时就像是类自己的成员一样。比如`MathUtils`类的`max()`方法会被像这样调用：`MathUtils.max(3, 5)`。这是一个公有静态成员的示例，即可以在不实例化类的情况下使用。同样也可以有私有的静态方法，即对类的使用者不可见，而在类的所有实例间是共享的。我们来看一下如何在JavaScript中实现公有和私有静态成员。

### 公有静态成员

在JavaScript中没有专门用于静态成员的语法。但通过给构造函数添加属性的方法，可以拥有和基于类的语言一样的使用语法。之所以可以这样做是因为构造函数和其它的函数一样，也是对象，可以拥有属性。前一章讨论过的记忆模式也使用了同样的方法，即给函数添加属性。

下面的例子定义了一个构造函数`Gadget()`，它有一个静态方法`isShiny()`和一个实例方法`setPrice()`。`isShiny()`是一个静态方法，因为它不需要指定一个具体的对象就能工作（你不需要先拿到一个特定的小工具（gadget）才知道所有小工具是不是有光泽的（shiny））。但setPrice()却需要一个对象，因为小工具可能有不同的定价：

	// 构造函数
	var Gadget = function () {};

	// 静态方法
	Gadget.isShiny = function () {
		return "you bet";
	};

	// 添加到原型的普通方法
	Gadget.prototype.setPrice = function (price) {
		this.price = price;
	};

现在我们来调用这些方法。静态方法`isShiny()`可以直接在构造函数上调用，但其它的方法需要一个实例：

	// 调用静态方法
	Gadget.isShiny(); // "you bet"

	// 创建实例并调用方法
	var iphone = new Gadget();
	iphone.setPrice(500);

使用静态方法的调用方式去调用实例方法并不能正常工作，同样，用调用实例方法的方式来调用静态方法也不能正常工作：

	typeof Gadget.setPrice; // "undefined"
	typeof iphone.isShiny; // "undefined"

有时候让静态方法也能用在实例上会很方便。我们可以通过在原型上加一个新方法来很容易地做到这点，这个新方法作为原来的静态方法的一个包装：

	Gadget.prototype.isShiny = Gadget.isShiny;
	iphone.isShiny(); // "you bet"

在这种情况下，你需要很小心地处理静态方法内的`this`。当你运行`Gadget.isShiny()`时，在`isShiny()`内部的`this`指向`Gadget`构造函数。而如果你运行`iphone.isShiny()`，那么`this`会指向`iphone`。

下面的例子展示了同一个方法被静态调用和非静态调用时明显不同的行为，这取决于调用的方式。这里的`instanceof`用于获取方法是如何被调用的：

	// 构造函数
	var Gadget = function (price) {
		this.price = price;
	};

	// 静态方法
	Gadget.isShiny = function () {

		// 这句始终正常工作
		var msg = "you bet";

		if (this instanceof Gadget) {
			// 这句只有在非静态方式调用时正常工作
			msg += ", it costs $" + this.price + '!';
		}

		return msg;
	};

	// 原型上添加的方法
	Gadget.prototype.isShiny = function () {
		return Gadget.isShiny.call(this);
	};

测试一下静态方法调用：

	Gadget.isShiny(); // "you bet"

测试一下实例中的非静态调用：

	var a = new Gadget('499.99');
	a.isShiny(); // "you bet, it costs $499.99!"

### 私有静态成员

到目前为止，我们都只讨论了公有的静态方法，现在我们来看一下如何实现私有静态成员。所谓私有静态成员是指：

- 被所有由同一构造函数创建的对象共享
- 不允许在构造函数外部访问

我们来看一个例子，`counter`是`Gadget()`构造函数的一个私有静态属性。在本章中我们已经讨论过私有属性，这里的做法也是一样，需要一个函数提供的闭包来包裹私有成员。然后让这个包裹函数立即执行并返回一个新的函数。将这个返回的函数赋值给`Gadget()`作为构造函数。

	var Gadget = (function () {

		// 静态变量/属性
		var counter = 0;

		// 返回构造函数的新实现
		return function () {
			console.log(counter += 1);
		};

	}()); // 立即执行

这个`Gadget()`构造函数只简单地增加私有变量`counter`的值然后打印出来。用多个实例测试的话你会看到`counter`在实例之间是共享的：

	var g1 = new Gadget();// logs 1
	var g2 = new Gadget();// logs 2
	var g3 = new Gadget();// logs 3

因为我们在创建每个实例的时候`counter`的值都会加1，所以它实际上成了唯一标识使用`Gadget`构造函数创建的对象的ID。这个唯一标识可能会很有用，那为什么不把它通过一个特权方法暴露出去呢？（译注：严格来讲，这里不能叫ID，只是一个记录有多少个实例的数字而已，因为如果有多个实例被创建的话，没有办法取到除了最后一个之外的实例的标识。）下面的例子是基于前面的例子，增加了用于访问私有静态属性的`getLastId()`方法：

	// 构造函数
	var Gadget = (function () {

		// 静态变量/属性
		var counter = 0,
			NewGadget;

		// 这将是Gadget的新实现
		NewGadget = function () {
			counter += 1;
		};

		// 特权方法
		NewGadget.prototype.getLastId = function () {
			return counter;
		};

		// 重写构造函数
		return NewGadget;

	}()); // 立即执行

测试这个新的实现：

	var iphone = new Gadget(); 
	iphone.getLastId(); // 1
	var ipod = new Gadget(); 
	ipod.getLastId(); // 2
	var ipad = new Gadget(); 
	ipad.getLastId(); // 3

静态属性（包括私有和公有）有时候会非常方便，它们可以包含和具体实例无关的方法和数据，而不用在每次实例中再创建一次。当我们在第七章中讨论单例模式时，你可以看到使用静态属性实现类式单例构造函数的例子。

## 对象常量

在一些比较现代的环境中可能会提供`const`来创建常量，但在其它的环境中，JavaScript是没有常量的。

一种常用的解决办法是通过命名规范，让不应该变化的变量使用全大写。这个规范实际上也用在JavaScript原生对象中：

	Math.PI; // 3.141592653589793
	Math.SQRT2; // 1.4142135623730951
	Number.MAX_VALUE; // 1.7976931348623157e+308

你自己的常量也可以用这种规范，然后将它们作为静态属性加到构造函数中：

	// 构造函数
	var Widget = function () {
		// 实现……
	};

	// 常量
	Widget.MAX_HEIGHT = 320;
	Widget.MAX_WIDTH = 480;

同样的规范也适用于使用字面量创建的对象，常量会是使用大写名字的属性。

如果你真的希望有一个不能被改变的值，那么可以创建一个私有属性，然后提供一个取值的方法（getter），但不给赋值的方法（setter）。这种方法在很多可以用命名规范解决的情况下可能有些矫枉过正，但不失为一种选择。

下面是一个通用的`constant`对象的实现，它提供了这些方法：

- set(name, value)
	
	定义一个新的常量

- isDefined(name)

	检查一个常量是否存在

- get(name)

	取常量的值

在这个实现中，只允许基本类型的值成为常量。同时还要使用`hasOwnProperty()`小心地处理那些恰好是原生属性的常量名，比如`toString`或者`hasOwnProperty`，然后给所有的常量名加上一个随机生成的前缀：

	var constant = (function () {
		var constants = {},
			ownProp = Object.prototype.hasOwnProperty,
			allowed = {
				string: 1,
				number: 1,
				boolean: 1
			},
			prefix = (Math.random() + "_").slice(2);
		return {
			set: function (name, value) {
				if (this.isDefined(name)) {
					return false;
				}
				if (!ownProp.call(allowed, typeof value)) {
					return false;
				}
				constants[prefix + name] = value;
				return true;
			},
			isDefined: function (name) {
				return ownProp.call(constants, prefix + name);
			},
			get: function (name) {
				if (this.isDefined(name)) {
					return constants[prefix + name];
				}
				return null;
			}
		};
	}());

测试这个实现：

	// 检查是否定义
	constant.isDefined("maxwidth"); // false

	// 定义
	constant.set("maxwidth", 480); // true

	// 再次检查
	constant.isDefined("maxwidth"); // true

	// 尝试重定义
	constant.set("maxwidth", 320); // false

	// 看看这个值是否被改变
	constant.get("maxwidth"); // 480

## 链式调用模式

使用链式调用模式可以让你在一对个象上连续调用多个方法，不需要将前一个方法的返回值赋给变量，也不需要将多个方法调用分散在多行：

	myobj.method1("hello").method2().method3("world").method4();

当你创建了一个没有有意义的返回值的方法时，你可以让它返回`this`，也就是这些方法所属的对象。这使得对象的使用者可以将下一个方法的调用和前一次调用链起来：

	var obj = {
		value: 1,
		increment: function () {
			this.value += 1;
			return this;
		},
		add: function (v) {
			this.value += v;
			return this;
		},
		shout: function () {
			alert(this.value);
		}
	};

	// 链式方法调用
	obj.increment().add(3).shout(); // 5

	// 单独调用每个方法
	obj.increment();
	obj.add(3);
	obj.shout(); // 5

### 链式调用模式的利弊

使用链式调用模式的一个好处就是可以节省代码量，使得代码更加简洁和易读，读起来就像在读句子一样。

另外一个好处就是帮助你思考如何拆分你的函数，创建更小、更有针对性的函数，而不是一个什么都做的函数。长时间来看，这会提升代码的可维护性。

一个弊端是调试这样写的代码会更困难。你可能知道一个错误出现在某一行，但这一行要做很多的事情。当链式调用的方法中的某一个出现问题而又没报错时，你无法知晓到底是哪一个出问题了。《代码整洁之道》的作者Robert Martion甚至叫这种模式为“train wreck”模式。（译注：直译为“火车事故”，指负面影响比较大。）

不管怎样，认识这种模式总是好的，当你写的方法没有明显的有意义的返回值时，你就可以返回`this`。这个模式应用得很广泛，比如jQuery库。如果你去看DOM的API的话，你会发现它也会以这样的形式倾向于链式调用：

	document.getElementsByTagName('head')[0].appendChild(newnode);

## method()方法

JavaScript对于习惯于用类来思考的人来说可能会比较费解，这也是很多开发者希望将JavaScript代码变得更像基于类的语言的原因。其中的一种尝试就是由Douglas Crockford提出来的`method()`方法。其实，他也承认将JavaScript变得像基于类的语言是不推荐的方法，但不管怎样，这都是一种有意思的模式，你可能会在一些应用中见到。

使用构造函数就像Java中使用类一样。它也允许你在构造函数体的`this`中添加实例属性。但是在`this`中添加方法却是不高效的，因为最终这些方法会在每个实例中被重新创建一次，这样会花费更多的内存。这也是为什么可重用的方法应该被放到构造函数的`prototype`属性（原型）中的原因。但对很多开发者来说，`prototype`可能跟个外星人一样陌生，所以你可以通过一个方法将它隐藏起来。

> 给语言添加一个使用起来更方便的方法一般叫作“语法糖”。在这个例子中，你可以将`method()`方法称为一个语法糖方法。

使用这个语法糖方法`method()`来定义一个“类”是像这样：

	var Person = function (name) {
		this.name = name;
	}.
		method('getName', function () {
			return this.name;
		}).
		method('setName', function (name) {
			this.name = name;
			return this;
		});

注意构造函数和调用`method()`是如何链起来的，接下来又链式调用了下一个`method()`方法。这就是我们前面讨论的链式调用模式，可以帮助我们用一个语句完成对整个“类”的定义。

`method()`方法接受两个参数：

- 新方法的名字
- 新方法的实现

然后这个新方法被添加到`Person`“类”。新方法的实现也只是一个函数，在这个函数里面`this`指向由`Person()`创建的对象，正如我们期望的那样。

下面是使用`Person()`创建和使用新对象的代码：

	var a = new Person('Adam');
	a.getName(); // 'Adam'
	a.setName('Eve').getName(); // 'Eve'

同样地注意链式调用，因为`setName()`返回了`this`就可以链式调用了。

最后是`method()`方法的实现：

	if (typeof Function.prototype.method !== "function") {
		Function.prototype.method = function (name, implementation) {
			this.prototype[name] = implementation;
			return this;
		};
	}

在`method()`的实现中，我们首先检查这个方法是否已经被实现过，如果没有则继续，将传入的参数`implementation`加到构造函数的原型中。在这里`this`指向构造函数，而我们要增加的功能正好在这个构造函数的原型上。

## 小结

在本章中你看到了好几种除了字面量和构造函数之外的创建对象的方法。

你看到了使用命名空间模式来保持全局空间干净和帮助组织代码，看到了简单而又有用的依赖声明模式。然后我们详细讨论了有关私有成员的模式，包括私有成员、特权方法以及一些涉及私有成员的极端情况，还有使用对象字面量创建私有成员以及将私有方法暴露为公有方法。所有这些模式都是搭建起现在流行而强大的模块模式的积木。

然后你看到了使用沙箱模式作为长命名空间的另一种选择，它可以为你的代码和模块提供独立的环境。

在最后，我们深入讨论了对象常量、静态成员（公有和私有）、链式调用模式，以及神奇的`method()`方法。