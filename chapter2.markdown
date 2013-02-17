<a name="a1"></a>
# 第二章 高质量JavaScript基本要点

本章将对一些实质内容展开讨论，这些内容包括最佳实践、模式和编写高质量JavaScript代码的习惯，比如避免全局变量、使用单var声明、循环中的length预缓存、遵守编码约定等等。本章还包括一些非必要的编程习惯，但更多的关注点将放在总体的代码创建过程上，包括撰写API文档、组织相互评审以及使用JSLint。这些习惯和最佳实践可以帮助你写出更好的、更易读的和可维护的代码，当几个月后或数年后再重读你的代码时，你就会深有体会了。

<a name="a2"></a>
## 编写可维护的代码

修复软件bug成本很高，而且随着时间的推移，它们造成的损失也越来越大，特别是在已经打包发布了的软件发现了bug的时候。当然最好是发现bug立刻解决掉，但前提是你对你的代码依然很熟悉，否则当你转身投入到另外一个项目的开发中后，根本不记得当初代码的模样了。过了一段时间后你再去阅读当初的代码你需要：

- 时间来重新学习并理解问题
- 时间去理解问题相关的代码

对大型项目或者公司来说还有一个不得不考虑的问题，就是解决这个bug的人和制造这个bug的人往往不是同一个人。因此减少理解代码所需的时间成本就显得非常重要，不管是隔了很长时间重读自己的代码还是阅读团队内其他人的代码。这对于公司的利益底线和工程师的幸福指数同样重要，因为每个人都宁愿去开发新的项目而不愿花很多时间和精力去维护旧代码。

另外一个软件开发中的普遍现象是，在读代码上花的时间要远远超过写代码的时间。常常当你专注于某个问题的时候，你会坐下来用一下午的时间产出大量的代码。当时的场景下代码是可以正常运行的，但当应用趋于成熟，会有很多因素促使你重读代码、改进代码或对代码做微调。比如：

- 发现了bug
- 需要给应用添加新需求
- 需要将应用迁移到新的平台中运行（比如当市场中出现了新的浏览器时）
- 代码重构
- 由于架构更改或者更换另一种语言导致代码重写

这些不确定因素带来的后果是，少数人花几小时写的代码需要很多人花几个星期去阅读它。因此，创建可维护的代码对于一个成功的应用来说至关重要。

可维护的代码意味着代码是：

- 可读的
- 一致的
- 可预测的
- 看起来像是同一个人写的
- 有文档的

本章接下来的部分会对这几点深入讲解。

<a name="a3"></a>
## 减少全局对象

JavaScript 使用函数来管理作用域，在一个函数内定义的变量称作“局部变量”，局部变量在函数外部是不可见的。另一方面，“全局变量”是不在任何函数体内部声明的变量，或者是直接使用而未声明的变量。

每一个JavaScript运行环境都有一个“全局对象”，不在任何函数体内使用this就可以获得对这个全局对象的引用。你所创建的每一个全局变量都是这个全局对象的属性。为了方便起见，浏览器都会额外提供一个全局对象的属性window，（常常）用以指向全局对象本身。下面的示例代码中展示了如何在浏览器中创建或访问全局变量：

	myglobal = "hello"; // antipattern
	console.log(myglobal); // "hello"
	console.log(window.myglobal); // "hello"
	console.log(window["myglobal"]); // "hello"
	console.log(this.myglobal); // "hello"

<a name="a4"></a>
### 全局对象带来的困扰

全局变量的问题是，它们在JavaScript代码执行期间或者整个web页面中始终是可见的。它们存在于同一个命名空间中，因此命名冲突的情况时有发生，毕竟在应用程序的不同模块中，经常会出于某种目的定义相同的全局变量。

同样，常常网页中所嵌入的代码并不是这个网页的开发者所写，比如：

- 网页中使用了第三方的JavaScript库
- 网页中使用了广告代码
- 网页中使用了用以分析流量和点击率的第三方统计代码
- 网页中使用了很多组件，挂件和按钮等等

假设某一段第三方提供的脚本定义了一个全局变量result。随后你在自己写的某个函数中也定义了一个全局变量result。这时，第二个变量就会覆盖第一个，这时就会导致第三方脚本停止工作。

因此，为了让你的脚本和这个页面中的其他脚本和谐相处，要尽可能少的使用全局变量，这一点非常重要。本书随后的章节中会讲到一些减少全局变量的技巧和策略，比如使用命名空间或者立即执行的匿名函数等，但减少全局变量最有效的方法是坚持使用var来声明变量。

由于JavaScript的特点，我们经常有意无意的创建全局变量，毕竟在JavaScript中创建全局变量实在太简单了。首先，你可以不声明而直接使用变量，再者，JavaScirpt中具有“隐式全局对象”的概念，也就是说任何不通过var声明（译注：在JavaScript1.7及以后的版本中，可以通过let来声明块级作用域的变量）的变量都会成为全局对象的一个属性（可以把它们当作全局变量）。看一下下面这段代码：

	function sum(x, y) {
		// antipattern: implied global
		result = x + y;
		return result;
	}

这段代码中，我们直接使用了result而没有事先声明它。这段代码是能够正常工作的，但在调用这个方法之后，会产生一个全局变量result，这会带来其他问题。

解决办法是，总是使用var来声明变量，下面代码就是改进了的sum()函数：

	function sum(x, y) {
		var result = x + y;
		return result;
	}

这里我们要注意一种反模式，就是在var声明中通过链式赋值的方法创建全局变量。在下面这个代码片段中，a是局部变量，但b是全局变量，而作者的意图显然不是如此：

	// antipattern, do not use
	function foo() {
		var a = b = 0;
		// ...
	}

为什么会这样？因为这里的计算顺序是从右至左的。首先计算表达式b=0，这里的b是未声明的，这个表达式的值是0，然后通过var创建了局部变量a，并赋值为0。换言之，可以等价的将代码写成这样：

	var a = (b = 0);

如果变量b已经被声明，这种链式赋值的写法是ok的，不会意外的创建全局变量，比如：

	function foo() {
		var a, b;
		// ...
		a = b = 0; // both local
	}

> 避免使用全局变量的另一个原因是出于可移植性考虑的，如果你希望将你的代码运行于不同的平台环境（宿主），使用全局变量则非常危险。很有可能你无意间创建的某个全局变量在当前的平台环境中是不存在的，你认为可以安全的使用，而在其他的环境中却是存在的。

<a name="a5"></a>
### 忘记var时的副作用

隐式的全局变量和显式定义的全局变量之间有着细微的差别，差别在于通过delete来删除它们的时候表现不一致。

- 通过var创建的全局变量（在任何函数体之外创建的变量）不能被删除。
- 没有用var创建的隐式全局变量（不考虑函数内的情况）可以被删除。

也就是说，隐式全局变量并不算是真正的变量，但他们是全局对象的属性成员。属性是可以通过delete运算符删除的，而变量不可以被删除：

	// define three globals
	var global_var = 1;
	global_novar = 2; // antipattern
	(function () {
		global_fromfunc = 3; // antipattern
	}());

	// attempt to delete
	delete global_var; // false
	delete global_novar; // true
	delete global_fromfunc; // true

	// test the deletion
	typeof global_var; // "number"
	typeof global_novar; // "undefined"
	typeof global_fromfunc; // "undefined"

在ES5严格模式中，给未声明的变量赋值会报错（比如这段代码中提到的两个反模式）。

<a name="a6"></a>
### 访问全局对象

在浏览器中，我们可以随时随地通过window属性来访问全局对象（除非你定义了一个名叫window的局部变量）。但换一个运行环境这个方便的window可能就换成了别的名字（甚至根本就被禁止访问全局对象了）。如果不想通过这种写死window的方式来得到全局变量，有一个办法，你可以在任意层次嵌套的函数作用域内执行：

	var global = (function () {
		return this;
	}());

这种方式总是可以得到全局对象，因为在被当作函数执行的函数体内（而不是被当作构造函数执行的函数体内），this总是指向全局对象。但这种情况在ECMAScript5的严格模式中行不通，因此在严格模式中你不得不寻求其他的替代方案。比如，如果你在开发一个库，你会将你的代码包装在一个立即执行的匿名函数中（在第四章会讲到），然后从全局作用域中给这个匿名函数传入一个指向this的参数。

<a name="a7"></a>
### 单 var 模式

在函数的顶部使用一个单独的var语句是非常推荐的一种模式，它有如下一些好处：

- 在同一个位置可以查找到函数所需的所有变量
- 避免当在变量声明之前使用这个变量时产生的逻辑错误（参照下一小节“声明提前：分散的 var 带来的问题”）
- 提醒你不要忘记声明变量，顺便减少潜在的全局变量
- 代码量更少（输入更少且更易做代码优化）

单var模式看起来像这样：

	function func() {
		var a = 1,
			b = 2,
			sum = a + b,
			myobject = {},
			i,
			j;
		// function body...
	}

你可以使用一个var语句来声明多个变量，变量之间用逗号分隔。也可以在这个语句中加入变量的初始化，这是一个非常好的实践。这种方式可以避免逻辑错误（所有未初始化的变量都被声明了，且值为undefined）并增加了代码的可读性。过段时间后再看这段代码，你会体会到声明不同类型变量的惯用名称，比如，你一眼就可看出某个变量是对象还是整数。

你可以在声明变量时多做一些额外的工作，比如在这个例子中就写了sum=a+b这种代码。另一个例子就是当代码中用到对DOM元素时，你可以把对DOM的引用赋值给一些变量，这一步就可以放在一个单独的声明语句中，比如下面这段代码：

	function updateElement() {
		var el = document.getElementById("result"),
			style = el.style;
		// do something with el and style...
	}

<a name="a8"></a>
### 声明提前：分散的 var 带来的问题 

JavaScript 中是允许在函数的任意地方写任意多个var语句的，其实相当于在函数体顶部声明变量，这种现象被称为“变量提前”，当你在声明之前使用这个变量时，可能会造成逻辑错误。对于JavaScript来说，一旦在某个作用域（同一个函数内）里声明了一个变量，这个变量在整个作用域内都是存在的，包括在var声明语句之前。看一下这个例子：

	// antipattern
	myname = "global"; // global variable
	function func() {
		alert(myname); // "undefined"
		var myname = "local";
		alert(myname); // "local"
	}
	func();

这个例子中，你可能期望第一个alert()弹出“global”，第二个alert()弹出“local”。这种结果看起来是合乎常理的，因为在第一个alert执行时，myname还没有声明，这时就应该“寻找”全局变量中的myname。但实际情况并不是这样，第一个alert弹出“undefined”，因为myname已经在函数内有声明了（尽管声明语句在后面）。所有的变量声明都提前到了函数的顶部。因此，为了避免类似带有“歧义”的程序逻辑，最好在使用之前一起声明它们。

上一个代码片段等价于下面这个代码片段：

	myname = "global"; // global variable
	function func() {
		var myname; // same as -> var myname = undefined;
		alert(myname); // "undefined"
		myname = "local";
		alert(myname); // "local"
	}
	func();

>这里有必要对“变量提前”作进一步补充，实际上从JavaScript引擎的工作机制上看，这个过程稍微有点复杂。代码处理经过了两个阶段，第一阶段是创建变量、函数和参数，这一步是预编译的过程，它会扫描整段代码的上下文。第二阶段是代码的运行，这一阶段将创建函数表达式和一些非法的标识符（未声明的变量）。从实用性角度来讲，我们更愿意将这两个阶段归成一个概念“变量提前”，尽管这个概念并没有在ECMAScript标准中定义，但我们常常用它来解释预编译的行为过程。

<a name="a9"></a>
## for 循环

在for循环中，可以对数组或类似数组的对象（比如arguments和HTMLCollection对象）作遍历，最普通的for循环模式形如：

	// sub-optimal loop
	for (var i = 0; i < myarray.length; i++) {
		// do something with myarray[i]
	}

这种模式的问题是，每次遍历都会访问数组的length属性。这降低了代码运行效率，特别是当myarray并不是一个数组而是一个HTMLCollection对象的时候。

HTMLCollection是由DOM方法返回的对象，比如：

- document.getElementsByName()
- document.getElementsByClassName()
- document.getElementsByTagName()

还有很多其他的HTMLCollection，这些对象是在DOM标准之前就已经在用了，这些HTMLCollection主要包括：

**document.images**

页面中所有的IMG元素

**document.links**

页面中所有的A元素

**document.forms**

页面中所有的表单

**document.forms[0].elements**

页面中第一个表单的所有字段

这些对象的问题在于，它们均是指向文档（HTML页面）中的活动对象。也就是说每次通过它们访问集合的length时，总是会去查询DOM，而DOM操作则是很耗资源的。

更好的办法是为for循环缓存住要遍历的数组的长度，比如下面这段代码：

	for (var i = 0, max = myarray.length; i < max; i++) {
		// do something with myarray[i]
	}

通过这种方法只需要访问DOM节点一次以获得length，在整个循环过程中就都可以使用它。

不管在什么浏览器中，在遍历HTMLCollection时缓存length都可以让程序执行的更快，可以提速两倍（Safari3）到一百九十倍（IE7）不等。更多细节可以参照Nicholas Zakas的《高性能JavaScript》，这本书也是由O'Reilly出版。

需要注意的是，当你在循环过程中需要修改这个元素集合（比如增加DOM元素）时，你更希望更新length而不是更新常量。

遵照单var模式，你可以将var提到循环的外部，比如：

	function looper() {
		var i = 0,
			max,
			myarray = [];
		// ...
		for (i = 0, max = myarray.length; i < max; i++) {
			// do something with myarray[i]
		}
	}

这种模式带来的好处就是提高了代码的一致性，因为你越来越依赖这种单var模式。缺点就是在重构代码的时候不能直接复制粘贴一个循环体，比如，你正在将某个循环从一个函数拷贝至另外一个函数中，必须确保i和max也拷贝至新函数里，并且需要从旧函数中将这些没用的变量删除掉。

最后一个需要对循环做出调整的地方是将i++替换成为下面两者之一：

	i = i + 1
	i += 1

JSLint提示你这样做，是因为++和--实际上降低了代码的可读性，如果你觉得无所谓，可以将JSLint的plusplus选项设为false（默认为true），本书所介绍的最后一个模式用到了: i += 1。

关于这种for模式还有两种变化的形式，做了少量改进，原因有二：

- 减少一个变量（没有max）
- 减量循环至0，这种方式速度更快，因为和零比较要比和非零数字或数组长度比较要高效的多

第一种变化形式是：

	var i, myarray = [];
	for (i = myarray.length; i--;) {
		// do something with myarray[i]
	}

第二种变化形式用到了while循环：

	var myarray = [],
		i = myarray.length;
	while (i--) {
		// do something with myarray[i]
	}

这些小改进只体现在性能上，此外，JSLint不推荐使用i--。

<a name="a10"></a>
## for-in 循环

for-in 循环用于对非数组对象作遍历。通过for-in进行循环也被称作“枚举”。

从技术角度讲，for-in循环同样可以用于数组（JavaScript中数组即是对象），但不推荐这样做。当使用自定义函数扩充了数组对象时，这时更容易产生逻辑错误。另外，for-in循环中属性的遍历顺序是不固定的，所以最好数组使用普通的for循环，对象使用for-in循环。

可以使用对象的hasOwnProperty()方法将从原型链中继承来的属性过滤掉，这一点非常重要。看一下这段代码：

	// the object
	var man = {
		hands: 2,
		legs: 2,
		heads: 1
	};
	// somewhere else in the code
	// a method was added to all objects
	if (typeof Object.prototype.clone === "undefined") {
		Object.prototype.clone = function () {};
	}

在这段例子中，我们定义了一个名叫man的对象直接量。在代码中的某个地方（可以是man定义之前也可以是之后），给Object的原型中增加了一个方法clone()。原型链是实时的，这意味着所有的对象都可以访问到这个新方法。要想在枚举man的时候避免枚举出clone()方法，则需要调用hasOwnProperty()来对原型属性进行过滤。如果不做过滤，clone()也会被遍历到，而这不是我们所希望的：

	// 1.
	// for-in loop
	for (var i in man) {
		if (man.hasOwnProperty(i)) { // filter
			console.log(i, ":", man[i]);
		}
	}
	/*
	result in the console
	hands : 2
	legs : 2
	heads : 1
	*/

	// 2.
	// antipattern:
	// for-in loop without checking hasOwnProperty()
	for (var i in man) {
		console.log(i, ":", man[i]);
	}
	/*
	result in the console
	hands : 2
	legs : 2
	heads : 1
	clone: function()
	*/

另外一种的写法是通过Object.prototype直接调用hasOwnProperty()方法，像这样：

	for (var i in man) {
		if (Object.prototype.hasOwnProperty.call(man, i)) { // filter
			console.log(i, ":", man[i]);
		}
	}

这种做法的好处是，当man对象中重新定义了hasOwnProperty方法时，可以避免调用时的命名冲突（译注：明确指定调用的是Object.prototype上的方法而不是实例对象中的方法），这种做法同样可以避免冗长的属性查找过程（译注：这种查找过程多是在原型链上进行查找），一直查找到Object中的方法，你可以定义一个变量来“缓存”住它（译注：这里所指的是缓存住Object.prototype.hasOwnProperty）：

	var i,
		hasOwn = Object.prototype.hasOwnProperty;
	for (i in man) {
		if (hasOwn.call(man, i)) { // filter
			console.log(i, ":", man[i]);
		}
	}

>严格说来，省略hasOwnProperty()并不是一个错误。根据具体的任务以及你对代码的自信程度，你可以省略掉它以提高一些程序执行效率。但当你对当前要遍历的对象不确定的时候，添加hasOwnProperty()则更加保险些。

这里提到一种格式上的变化写法（这种写法无法通过JSLint检查），这种写法在for循环所在的行加入了if判断条件，他的好处是能让循环语句读起来更完整和通顺（“如果元素包含属性X，则拿X做点什么”）：

	// Warning: doesn't pass JSLint
	var i,
		hasOwn = Object.prototype.hasOwnProperty;
	for (i in man) if (hasOwn.call(man, i)) { // filter
		console.log(i, ":", man[i]);
	}

<a name="a11"></a>
## （不）扩充内置原型

我们可以扩充构造函数的prototype属性，这是一种非常强大的特性，用来为构造函数增加功能，但有时这个功能强大到超过我们的掌控。

给内置构造函数比如Object()、Array()、和Function()扩充原型看起来非常诱人，但这种做法严重降低了代码的可维护性，因为它让你的代码变得难以预测。对于那些基于你的代码做开发的开发者来说，他们更希望使用原生的JavaScript方法来保持工作的连续性，而不是使用你所添加的方法（译注：因为原生的方法更可靠，而你写的方法可能会有bug）。

另外，如果将属性添加至原型中，很可能导致在那些不使用hasOwnProperty()做检测的循环中将原型上的属性遍历出来，这会造成混乱。

因此，不扩充内置对象的原型是最好的，你也可以自己定义一个规则，仅当下列条件满足时做例外考虑：

1. 未来的ECMAScript版本的JavaScirpt会将你实现的方法添加为内置方法。比如，你可以实现ECMAScript5定义的一些方法，一直等到浏览器升级至支持ES5。这样，你只是提前定义了这些有用的方法。
2. 如果你发现你自定义的方法已经不存在，要么已经在代码其他地方实现了，要么是浏览器的JavaScript引擎已经内置实现了。
3. 你所做的扩充附带充分的文档说明，且和团队其他成员做了沟通。

如果你遇到这三种情况之一，你可以给内置原型添加自定义方法，写法如下：

	if (typeof Object.protoype.myMethod !== "function") {
		Object.protoype.myMethod = function () {
			// implementation...
		};
	}

<a name="a12"></a>
## switch 模式

你可以通过下面这种模式的写法来增强switch语句的可读性和健壮性：

	var inspect_me = 0,
		result = '';
	switch (inspect_me) {
	case 0:
		result = "zero";
		break;
	case 1:
		result = "one";
		break;
	default:
		result = "unknown";
	}

这个简单的例子所遵循的风格约定如下：

- 每个case和switch对齐（这里不考虑花括号相关的缩进规则）
- 每个case中的代码整齐缩进
- 每个case都以break作为结束
- 避免连续执行多个case语句块（当省略break时会发生），如果你坚持认为连续执行多case语句块是最好的方法，请务必补充文档说明，对于其他人来说，这种情况看起来是错误的。
- 以default结束整个switch，以确保即便是在找不到匹配项时也会有正常的结果，

<a name="a13"></a>
## 避免隐式类型转换

在JavaScript的比较操作中会有一些隐式的数据类型转换。比如诸如false == 0或""==0之类的比较都返回true。

为了避免隐式类型转换造对程序造成干扰，推荐使用===和!===运算符，它们较除了比较值还会比较类型。

	var zero = 0;
	if (zero === false) {
		// not executing because zero is 0, not false
	}
	// antipattern
	if (zero == false) {
		// this block is executed...
	}

另外一种观点认为当==够用的时候就不必多余的使用===。比如，当你知道typeof的返回值是一个字符串，就不必使用全等运算符。但JSLint却要求使用全等运算符，这当然会提高代码风格的一致性，并减少了阅读代码时的思考（“这里使用==是故意的还是无意的？”）。

<a name="a14"></a>
### 避免使用eval()

当你想使用eval()的时候，不要忘了那句话“eval()是魔鬼”。这个函数的参数是一个字符串，它可以执行任意字符串。如果事先知道要执行的代码是有问题的（在运行之前），则没有理由使用eval()。如果需要在运行时动态生成执行代码，往往都会有更佳的方式达到同样的目的，而非一定要使用eval()。例如，访问动态属性时可以使用方括号：

	// antipattern
	var property = "name";
	alert(eval("obj." + property));
	// preferred
	var property = "name";
	alert(obj[property]);

eval()同样有安全隐患，因为你需要运行一些容易被干扰的代码（比如运行一段来自于网络的代码）。在处理Ajax请求所返回的JSON数据时会常遇到这种情况，使用eval()是一种反模式。这种情况下最好使用浏览器的内置方法来解析JSON数据，以确保代码的安全性和数据的合法性。如果浏览器不支持JSON.parse()，你可以使用JSON.org所提供的库。

记住，多数情况下，给setInterval()、setTimeout()和Function()构造函数传入字符串的情形和eval()类似，这种用法也是应当避免的，这一点非常重要，因为这些情形中JavaScript最终还是会执行传入的字符串参数：

	// antipatterns
	setTimeout("myFunc()", 1000);
	setTimeout("myFunc(1, 2, 3)", 1000);
	// preferred
	setTimeout(myFunc, 1000);
	setTimeout(function () {
		myFunc(1, 2, 3);
	}, 1000);

new Function()的用法和eval()非常类似，应当特别注意。这种构造函数的方式很强大，但往往被误用。如果你不得不使用eval()，你可以尝试用new Function()来代替。这有一个潜在的好处，在new Function()中运行的代码会在一个局部函数作用域内执行，因此源码中所有用var定义的变量不会自动变成全局变量。还有一种方法可以避免eval()中定义的变量转换为全局变量，即是将eval()包装在一个立即执行的匿名函数内（详细内容请参照第四章）。

看一下这个例子，这里只有un成为了全局变量，污染了全局命名空间：

	console.log(typeof un);// "undefined"
	console.log(typeof deux); // "undefined"
	console.log(typeof trois); // "undefined"

	var jsstring = "var un = 1; console.log(un);";
	eval(jsstring); // logs "1"

	jsstring = "var deux = 2; console.log(deux);";
	new Function(jsstring)(); // logs "2"

	jsstring = "var trois = 3; console.log(trois);";
	(function () {
		eval(jsstring);
	}()); // logs "3"

	console.log(typeof un); // "number"
	console.log(typeof deux); // "undefined"
	console.log(typeof trois); // "undefined"

eval()和Function构造函数还有一个区别，就是eval()可以修改作用域链，而Function更像是一个沙箱。不管在什么地方执行Function，它只能看到全局作用域。因此它不会太严重的污染局部变量。在下面的示例代码中，eval()可以访问且修改其作用域之外的变量，而Function不能（注意，使用Function和new Function是完全一样的）。

	(function () {
		var local = 1;
		eval("local = 3; console.log(local)"); // logs 3
		console.log(local); // logs 3
	}());

	(function () {
		var local = 1;
		Function("console.log(typeof local);")(); // logs undefined
	}());

<a name="a15"></a>
## 使用parseInt()进行数字转换

可以使用parseInt()将字符串转换为数字。函数的第二个参数是转换基数（译注：“基数”指的是数字进制的方式），这个参数通常被省略。但当字符串以0为前缀时转换就会出错，例如，在表单中输入日期的一个字段。ECMAScript3中以0为前缀的字符串会被当作八进制数处理（基数为8）。但在ES5中不是这样。为了避免转换类型不一致而导致的意外结果，应当总是指定第二个参数：

	var month = "06",
		year = "09";
	month = parseInt(month, 10);
	year = parseInt(year, 10);

在这个例子中，如果省略掉parseInt的第二个参数，比如parseInt(year)，返回值是0，因为“09”被认为是八进制数（等价于parseInt(year,8)），而且09是非法的八进制数。

字符串转换为数字还有两种方法：

	+"08" // result is 8
	Number("08") // 8

这两种方法要比parseInt()更快一些，因为顾名思义parseInt()是一种“解析”而不是简单的“转换”。但当你期望将“08 hello”这类字符串转换为数字，则必须使用parseInt()，其他方法都会返回NaN。

<a name="a16"></a>
## 编码风格

确立并遵守编码规范非常重要，这会让你的代码风格一致、可预测、可读性更强。团队新成员通过学习编码规范可以很快进入开发状态、并写出团队其他成员易于理解的代码。

在开源社区和邮件组中关于编码风格的争论一直不断（比如关于代码缩进，用tab还是空格？）。因此，如果你打算在团队内推行某种编码规范时，要做好应对各种反对意见的心理准备，而且要吸取各种意见，这对确立并一贯遵守某种编码规范是非常重要，而不是斤斤计较的纠结于编码规范的细节。

<a name="a17"></a>
### 缩进

代码没有缩进几乎就不能读了，而不一致的缩进更加糟糕，因为它看上去像是遵循了规范，真正读起来却磕磕绊绊。因此规范的使用缩进非常重要。

有些开发者喜欢使用tab缩进，因为每个人都可以根据自己的喜好来调整tab缩进的空格数，有些人则喜欢使用空格缩进，通常是四个空格，这都无所谓，只要团队每个人都遵守同一个规范即可，本书中所有的示例代码都采用四个空格的缩进写法，这也是JSLint所推荐的。

那么到底什么应该缩进呢？规则很简单，花括号里的内容应当缩进，包括函数体、循环（do、while、for和for-in）体、if条件、switch语句和对象直接量里的属性。下面的代码展示了如何正确的使用缩进：

	function outer(a, b) {
		var c = 1,
			d = 2,
			inner;
		if (a > b) {
			inner = function () {
				return {
					r: c - d
				};
			};
		} else {
			inner = function () {
				return {
					r: c + d
				};
			};
		}
		return inner;
	}

<a name="a18"></a>
### 花括号

应当总是使用花括号，即使是在可省略花括号的时候也应当如此。从技术角度讲，如果if或for中只有一个语句，花括号是可以省略的，但最好还是不要省略。这让你的代码更加工整一致而且易于更新。

假设有这样一段代码，for循环中只有一条语句，你可以省略掉这里的花括号，而且不会有语法错误：

	// bad practice
	for (var i = 0; i < 10; i += 1)
		alert(i);

但如果过了一段时间，你给这个循环添加了另一行代码？

	// bad practice
	for (var i = 0; i < 10; i += 1)
		alert(i);
		alert(i + " is " + (i % 2 ? "odd" : "even"));

第二个alert实际处于循环体之外，但这里的缩进会迷惑你。长远考虑最好还是写上花括号，即便是在只有一个语句的语句块中也应如此：

	// better
	for (var i = 0; i < 10; i += 1) {
		alert(i);
	}

同理，if条件句也应当如此：

	// bad
	if (true)
		alert(1);
	else
		alert(2);
	
	// better
	if (true) {
		alert(1);
	} else {
		alert(2);
	}

<a name="a19"></a>
###  左花括号的位置

开发人员对于左大括号的位置有着不同的偏好，在同一行呢还是在下一行？

	if (true) {
		alert("It's TRUE!");
	}

或者：

	if (true)
	{
		alert("It's TRUE!");
	}


在这个例子中，看起来只是个人偏好问题。但有时候花括号位置的不同则会影响程序的执行。因为JavaScript会“自动插入分号”。JavaScript对行结束时的分号并无要求，它会自动将分号补全。因此，当函数return语句返回了一个对象直接量，而对象的左花括号和return不在同一行时，程序的执行就和预想的不同了：

	// warning: unexpected return value
	function func() {
		return
		{
			name: "Batman"
		};
	}

可以看出程序作者的意图是返回一个包含了name属性的对象，但实际情况不是这样。因为return后会填补一个分号，函数的返回值就是undefined。这段代码等价于：

	// warning: unexpected return value
	function func() {
		return undefined;
		// unreachable code follows...
		{
			name: "Batman"
		};
	}

结论，总是使用花括号，而且总是将左花括号与上一条语句放在同一行：

	function func() {
		return {
			name: "Batman"
		};
	}

>关于分号应当注意：和花括号一样，应当总是使用分号，尽管在JavaScript解析代码时会补全行末省略的分号。严格遵守这条规则，可以让代码更加严谨，同时可以避免前面例子中所出现的歧义。

<a name="a20"></a>
###　空格

空格的使用同样有助于改善代码的可读性和一致性。在写英文句子的时候，在逗号和句号后面会使用间隔。在JavaScript中，你可以按照同样的逻辑在表达式（相当于逗号）和语句结束（相对于完成了某个“想法”）后面添加间隔。

适合使用空格的地方包括：

- for循环中的分号之后，比如 `for (var i = 0; i < 10; i += 1) {...}`
- for循环中初始化多个变量，比如 `for (var i = 0, max = 10; i < max; i += 1) {...}`
- 分隔数组项的逗号之后，`var a = [1, 2, 3];`
- 对象属性后的逗号以及名值对之间的冒号之后，`var o = {a: 1, b: 2};`
- 函数参数中，`myFunc(a, b, c)`
- 函数声明的花括号之前，`function myFunc() {}`
- 匿名函数表达式function之后，`var myFunc = function () {};`

另外，我们推荐在运算符和操作数之间添加空格。也就是说在+, -, *, =, <, >, <=, >=, ===, !==, &&, ||, +=符号前后都添加空格。

	// generous and consistent spacing
	// makes the code easier to read
	// allowing it to "breathe"
	var d = 0,
		a = b + 1;
	if (a && b && c) {
		d = a % c;
		a += d;
	}

	// antipattern
	// missing or inconsistent spaces
	// make the code confusing
	var d= 0,
		a =b+1;
	if (a&& b&&c) {
		d=a %c;
		a+= d;
	}

最后，还应当注意，最好在花括号旁边添加空格：

- 在函数、if-else语句、循环、对象直接量的左花括号之前补充空格（{）
- 在右花括号和else和while之间补充空格

>垂直空白的使用经常被我们忽略，你可以使用空行来将代码单元分隔开，就像文学作品中使用段落作分隔一样。

<a name="a21"></a>
## 命名规范

另外一种可以提升你代码的可预测性和可维护性的方法是采用命名规范。也就是说变量和函数的命名都遵照同种习惯。

下面是一些建议的命名规范，你可以原样采用，也可以根据自己的喜好作调整。同样，遵循规范要比规范本身更加重要。

<a name="a22"></a>
### 构造器命名中的大小写

JavaScript中没有类，但有构造函数，可以通过new来调用构造函数：

	var adam = new Person();

由于构造函数毕竟还是函数，不管我们将它用作构造器还是函数，当然希望只通过函数名就可分辨出它是构造器还是普通函数。

首字母大写可以提示你这是一个构造函数，而首字母小写的函数一般只认为它是普通的函数，不应该通过new来调用它：

	function MyConstructor() {...}
	function myFunction() {...}

下一章将介绍一些强制将函数用作构造器的编程模式，但遵守我们所提到的命名规范会更好的帮助程序员阅读源码。

<a name="a23"></a>
### 单词分隔

当你的变量名或函数名中含有多个单词时，单词之间的分隔也应当遵循统一的约定。最常见的做法是“驼峰式”命名，单词都是小写，每个单词的首字母是大写。

对于构造函数，可以使用“大驼峰式”命名，比如MyConstructor()，对于函数和方法，可以采用“小驼峰式”命名，比如myFunction()，calculateArea()和getFirstName()。

那么对于那些不是函数的变量应当如何命名呢？变量名通常采用小驼峰式命名，还有一个不错的做法是，变量所有字母都是小写，单词之间用下划线分隔，比如，first_name，favorite_bands和old_company_name，这种方法可以帮助你区分函数和其他标识符——原始数据类型或对象。

ECMAScript的属性和方法均使用Camel标记法，尽管多字的属性名称是罕见的（正则表达式对象的lastIndex和ignoreCase属性）。

在ECMAScript中的属性和方法均使用驼峰式命名，尽管包含多单词的属性名称（正则表达式对象中的lastIndex和ignoreCase）并不常见。

<a name="a24"></a>
### 其他命名风格

有时开发人员使用命名规范来弥补或代替语言特性的不足。

比如，JavaScript中无法定义常量（尽管有一些内置常量比如Number.MAX_VALUE），所以开发者都采用了这种命名习惯，对于那些程序运行周期内不会更改的变量使用全大写字母来命名。比如：

	// precious constants, please don't touch
	var PI = 3.14,
		MAX_WIDTH = 800;

除了使用大写字母的命名方式之外，还有另一种命名规约：全局变量都大写。这种命名方式和“减少全局变量”的约定相辅相成，并让全局变量很容易辨认。

除了常量和全局变量的命名惯例，这里讨论另外一种命名惯例，即私有变量的命名。尽管在JavaScript是可以实现真正的私有变量的，但开发人员更喜欢在私有成员或方法名之前加上下划线前缀，比如下面的例子：

	var person = {
		getName: function () {
			return this._getFirst() + ' ' + this._getLast();
		},
		_getFirst: function () {
			// ...
		},
		_getLast: function () {
			// ...
		}
	};

在这个例子中，getName()的身份是一个公有方法，属于稳定的API，而_getFirst()和_getLast()则是私有方法。尽管这两个方法本质上和公有方法无异，但在方法名前加下划线前缀就是为了警告用户不要直接使用这两个私有方法，因为不能保证它们在下一个版本中还能正常工作。JSLint会对私有方法作检查，除非设置了JSLint的nomen选项为false。

下面介绍一些_private风格写法的变种：

- 在名字尾部添加下划下以表明私有，比如`name_`和`getElements_()`
- 使用一个下划线前缀表明受保护的属性_protected，用两个下划线前缀表明私有属性__private
- 在Firefox中实现了一些非标准的内置属性，这些属性在开头和结束都有两个下划线，比如`__proto__`和`__parent__`

<a name="a25"></a>
## 书写注释

写代码就要写注释，即便你认为你的代码不会被别人读到。当你对一个问题非常熟悉时，你会很快找到问题代码，但当过了几个星期后再来读这段代码，则需要绞尽脑汁的回想代码的逻辑。

你不必对显而易见的代码作过多的注释：每个变量和每一行都作注释。但你需要对所有的函数、他们的参数和返回值补充注释，对于那些有趣的或怪异的算法和技术也应当配备注释。对于阅读你的代码的其他人来说，注释就是一种提示，只要阅读注释、函数名以及参数，就算不读代码也能大概理解程序的逻辑。比如，这里有五到六行代码完成了某个功能，如果提供了一行描述这段代码功能的注释，读程序的人就不必再去关注代码的细节实现了。代码注释的写法并没有硬性规定，有些代码片段（比如正则表达式）的确需要比代码本身还多的注释。

>由于过时的注释会带来很多误导，这比不写注释还糟糕。因此保持注释时刻更新的习惯非常重要，尽管对很多人来说这很难做到。

在下一小节我们会讲到，注释可以自动生成文档。

<a name="a26"></a>
## 书写API文档

很多人都觉得写文档是一件枯燥且吃力不讨好的事情，但实际情况不是这样。我们可以通过代码注释自动生成文档，这样就不用再去专门写文档了。很多人觉得这是一个不错的点子，因为根据某些关键字和格式化的文档自动生成可阅读的参考手册本身就是“某种编程”。

传统的APIdoc诞生自Java世界，这个工具名叫“javadoc”，和Java SDK（软件开发工具包）一起提供。但这个创意迅速被其他语言借鉴。JavaScript领域有两个非常优秀的开源工具，它们是JSDoc Toolkit（http://code.google.com/p/jsdoc-toolkit/ ）和YUIDoc（http://yuilibrary.com/projects/yuidoc ）。

生成API文档的过程包括：

- 以特定的格式来组织书写源代码
- 运行工具来对代码和注释进行解析
- 发布工具运行的结果，通常是HTML页面

你需要学习这种特殊的语法，包括十几种标签，写法类似于：

	/**
	 * @tag value
	 */

比如这里有一个函数reverse()，可以对字符串进行反序操作。它的参数和返回值都是字符串。给它补充注释如下：

	/**
	* Reverse a string
	*
	* @param {String} input String to reverse
	* @return {String} The reversed string
	*/
	var reverse = function (input) {
		// ...
		return output;
	};

可以看到，@param是用来说明输入参数的标签，@return是用来说明返回值的标签，文档生成工具最终会为将这种带注释的源代码解析成格式化好的HTML文档。

<a name="a27"></a>
### 一个例子：YUIDoc

YUIDoc最初的目的是为YUI库（Yahoo! User Interface）生成文档，但也可以应用于任何项目，为了更充分的使用YUIDoc你需要学习它的注释规范，比如模块和类的写法（当然在JavaScript中是没有类的概念的）。

让我们看一个用YUIDoc生成文档的完整例子。

图2-1展示了最终生成的文档的模样，你可以根据项目需要随意定制HTML模板，让生成的文档更加友好和个性化。

这里同样提供了在线的demo，请参照 http://jspatterns.com/book/2/。

这个例子中所有的应用作为一个模块（myapp）放在一个文件里（app.js），后续的章节会更详细的介绍模块，现在只需知道用可以用一个YUIDoc的标签来表示模块即可。

图2-1 YUIDoc生成的文档

![pic](http://img02.taobaocdn.com/tps/i2/T1fSCgXdBsXXXXXXXX-781-647.png)

app.js的开始部分：

	/**
	 * My JavaScript application
	 *
	 * @module myapp
	 */

然后定义了一个空对象作为模块的命名空间：

	var MYAPP = {};

紧接着定义了一个包含两个方法的对象math_stuff，这两个方法分别是sum()和multi()：

	/**
	* A math utility
	* @namespace MYAPP
	* @class math_stuff
	*/
	MYAPP.math_stuff = {
		/**
		* Sums two numbers
		*
		* @method sum
		* @param {Number} a First number
		* @param {Number} b The second number
		* @return {Number} The sum of the two inputs
		*/
		sum: function (a, b) {
			return a + b;
		},

		/**
		* Multiplies two numbers
		*
		* @method multi
		* @param {Number} a First number
		* @param {Number} b The second number
		* @return {Number} The two inputs multiplied
		*/
		multi: function (a, b) {
			return a * b;
		}
	};

这样就结束了第一个“类”的定义，注意粗体表示的标签。

@namespace

指向你的对象的全局引用

@class

代表一个对象或构造函数的不恰当的称谓（JavaScript中没有类）

@method

定义对象的方法，并指定方法的名称

@param

列出函数需要的参数，参数的类型放在一对花括号内，跟随其后的是参数名和描述

@return

和@param类似，用以描述方法的返回值，可以不带名字

我们用构造函数来实现第二个“类”，给这个类的原型添加一个方法，能够体会到YUIDoc采用了不同的方式来创建对象：

	/**
	* Constructs Person objects
	* @class Person
	* @constructor
	* @namespace MYAPP
	* @param {String} first First name
	* @param {String} last Last name
	*/
	MYAPP.Person = function (first, last) {
		/**
		* Name of the person
		* @property first_name
		* @type String
		*/
		this.first_name = first;
		/**
		* Last (family) name of the person
		* @property last_name
		* @type String
		*/
		this.last_name = last;
	};
	/**
	* Returns the name of the person object
	*
	* @method getName
	* @return {String} The name of the person
	*/
	MYAPP.Person.prototype.getName = function () {
		return this.first_name + ' ' + this.last_name;
	};

在图2-1中可以看到生成的文档中Person构造函数的生成结果，粗体的部分是：

- @constructor 暗示了这个“类”其实是一个构造函数
- @prototype 和 @type 用来描述对象的属性

YUIDoc工具是语言无关的，只解析注释块，而不是JavaScript代码。它的缺点是必须要在注释中指定属性、参数和方法的名字，比如，@property first_name。好处是一旦你熟练掌握YUIDoc，就可以用它对任何语言源码进行注释的文档化。

<a name="a28"></a>
## 编写易读的代码

这种将APIDoc格式的代码注释解析成API参考文档的做法看起来很偷懒，但还有另外一个目的，通过代码重审来提高代码质量。

很多作者或编辑会告诉你“编辑非常重要”，甚至是写一本好书或好文章最最重要的步骤。将想法落实在纸上形成草稿只是第一步，草稿给读者提的信息往往重点不明晰、结构不合理、或不符合循序渐进的阅读习惯。

对于编程也是同样的道理，当你坐下来解决一个问题的时候，这时的解决方案只是一种“草案”，尽管能正常工作，但是不是最优的方法呢？是不是可读性好、易于理解、可维护佳或容易更新？当一段时间后再来review你的代码，一定会发现很多需要改进的地方，需要重新组织代码或删掉多余的内容等等。这实际上就是在“整理”你的代码了，可以很大程度提高你的代码质量。但事情往往不是这样，我们常常承受着高强度的工作压力，根本没有时间来整理代码，因此通过代码注释写文档其实是不错的机会。

往往在写注释文档的时候，你会发现很多问题。你也会重新思考源代码中不合理之处，比如，某个方法中的第三个参数比第二个参数更常用，第二个参数多数情况下取值为true，因此就需要对这个方法接口进行适当的改造和包装。

写出易读的代码（或API），是指别人能轻易读懂程序的思路。所以你需要采用更好的思路来解决手头的问题。

尽管我们认为“草稿”不甚完美，但至少也算“抱佛脚”的权宜之计，一眼看上去是有点“草”，不过也无所谓，特别是当你处理的是一个关键项目时（会有人命悬与此）。其实你应当扔掉你所给出的第一个解决方案，虽然它是可以正常工作的，但毕竟是一个草率的方案，不是最佳方案。你给出的第二个方案会更加靠谱，因为这时你对问题的理解更加透彻。第二个方案不是简单的复制粘贴之前的代码，也不能投机取巧寻找某种捷径。

<a name="a29"></a>
## 相互评审

另外一种可以提高代码质量的方法是组织相互评审。同行的评审很正式也很规范，即便是求助于特定的工具，也不失是一种开发生产线上值得提倡的步骤。但你可能觉得没有时间去作代码互审，没关系，你可以让坐在你旁边的同事读一下你的代码，或者和她（译注：注意是“她”而不是“他”）一起过一遍你的代码。

同样，当你在写APIDoc或任何其他文档的时候，同行的评审能帮助你的产出物更加清晰，因为你写的文档是让别人读的，你必须确保别人能理解你所作的东西。

同行的评审是一种非常不错的习惯，不仅仅是因为它能让代码变得更好，更重要的，在评审的过程中，评审人和代码作者通过分享和讨论，两人都能取长补短、相互促进。

如果你的团队只有你一个开发人员，找不出第二个人能给你作代码评审，这也没关系。你可以通过将你的代码片段开源，或把有意思的代码片段贴在博客中，会有人对你的代码感兴趣的。

另外一个非常好的习惯是使用版本管理工具（CVS，SVN或Git），一旦有人修改并提交了代码，都会发邮件通知组内成员。虽然大部分邮件都进入了垃圾箱，但总是会碰巧有人在工作间隙看到你所提交的代码，并对代码做出一些评价。

<a name="a30"></a>
## 生产环境中的代码压缩（Minify）

这里所说的代码压缩（Minify）是指去除JavaScript代码中的空格、注释以及其他不必要的部分，用以减少JavaScript文件的体积，降低网络带宽损耗。我们通常使用类似YUICompressor（Yahoo!）或Closure Compiler（Google）的压缩工具来为网页加载提速。对于生产环境（译注：“生产环境”指的是项目上线后的正式环境）中的脚本是需要作压缩的，压缩后的文件体积能减少至原来的一半以下。

下面这段代码是压缩后的样子（这段代码是YUI2库中的Event模块）：

	YAHOO.util.CustomEvent=function(D,C,B,A){this.type=D;this.scope=C||window;this.silent
	=B;this.signature=A||YAHOO.util.CustomEvent.LIST;this.subscribers=[];if(!this.silent)
	{}var E="_YUICEOnSubscribe";if(D!==E){this.subscribeEvent=new
	YAHOO.util.CustomEvent(E,this,true);}...

除了去除空格、空行和注释之外，压缩工具还能缩短命名的长度（前提是保证代码的安全），比如这段代码中的参数A、B、C、D。压缩工具只会重命名局部变量，因为更改全局变量会破坏代码的逻辑。这也是要尽量使用局部变量的原因。如果你使用的全局变量是对DOM节点的引用，而且程序中多次用到，最好将它赋值给一个局部变量，这样能提高查找速度，代码也会运行的更快，此外还能提高压缩比、加快下载速度（译注：在服务器开启Gzip的情况下，对下载速度的影响几乎可以忽略不计）。

补充说明一下，Goolge Closure Compiler还会对全局变量进行压缩（在“高级”模式中），这是很危险的，且对编程规范的要求非常苛刻。它的好处是压缩比非常高。

对生产环境的脚本做压缩是相当重要的步骤，它能提升页面性能，你应当使用工具来完成压缩。千万不要试图手写“压缩好的”代码，你应当坚持使用语义化的变量命名，并保留足够的空格、缩进和注释。你写的代码是需要被人阅读的，所以应当将注意力放在代码可读性和可维护性上，代码压缩的工作交给工具去完成。

<a name="a31"></a>
## 运行 JSLint

在上一章我们已经介绍了JSLint，这里我们介绍更多的使用场景。对你的代码进行JSLint检查是非常好的编程习惯，你应该相信这一点。

JSLint的检查点都有哪些呢？它会对本章讨论过的一些模式（单var模式、parseInt()的第二个参数、总是使用花括号）做检查。JSLint还包括其他方面的检查：

- 不可达代码
- 在使用变量之前需要声明
- 不安全的UTF字符
- 使用void、with、和eval
- 无法正确解析的正则表达式

JSLint是基于JavaScript实现的（它是可以通过JSLint检查的），它提供了在线工具，也可以下载使用，可以运行于很多种平台的JavaScript解析器。你可以将源码下载后在本地运行，支持的环境包括WSH（Windows Scripting Host，Windows）、JSC（JavaScriptCore，MacOSX）或Rhino（Mozilla开发的JavaScript引擎）。

可以将JSLint下载后和你的代码编辑器配置在一起，着是一个不错的注意，这样每次你保存代码的时候都会自动执行代码检查（比如配置快捷键）。

<a name="a32"></a>
## 小结

本章我们讲解了编写可维护性代码的含义，本章的讨论非常重要，它不仅关系着软件项目的成功与否，还关系到参与项目的工程师的“精神健康”和“幸福指数”。随后我们讨论了一些最佳实践和模式，它们包括：

- 减少全局对象，最好每个应用只有一个全局对象
- 函数都使用单var模式来定义，这样可以将所有的变量放在同一个地方声明，同时可以避免“声明提前”给程序逻辑带来的影响。
- for循环、for-in循环、switch语句、“禁止使用eval()”、不要扩充内置原型
- 遵守统一的编码规范（在任何必要的时候保持空格、缩进、花括号和分号）和命名约定（构造函数、普通函数和变量）。

本章还讨论了其他一些和代码本身无关的实践，这些实践和编码过程紧密相关，包括书写注释、生成API文档，组织代码评审、不要试图去手动了“压缩”（minify）代码而牺牲代码可读性、坚持使用JSLint来对代码做检查。

