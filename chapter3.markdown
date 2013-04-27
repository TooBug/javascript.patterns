# 第三章 字面量和构造函数

JavaScript中的字面量模式更加简洁、有表现力，而且在定义对象时不容易出错。本章将会讨论字面量，包括对象、数组和正则表达式字面量，以及为什么字面量要比等价的内置构造函数（如`Object()`、`Array()`等）要更好。本章还会介绍JSON格式，JSON是使用数组和对象字面量的形式定义的一种数据交换格式。本章还会讨论自定义构造函数，包括如何强制使用`new`以确保构造函数正确执行。

为了方便使用字面量而不是构造函数，本章还会补充一些知识，比如内置包装构造函数`Number()`、`String()`和`Boolean()`，以及如何将它们和原始值（数字、字符串和布尔值）比较。最后，快速介绍一下`Error()`构造函数的用法。

## 对象字面量

我们可以将JavaScript中的对象简单地理解为名值对组成的散列表（hash table，也叫哈希表）。在其他编程语言中被称作“关联数组”。其中的值可以是原始值也可以是对象。不管是什么类型，它们都是“属性”（property），属性值同样可以是函数，这时属性就被称为“方法”（method）。

JavaScript中自定义的对象（用户定义的本地对象）任何时候都是可变的。内置本地对象的属性也是可变的。你可以先创建一个空对象，然后在需要时给它添加功能。“对象字面量写法（object literal notation）”是按需创建对象的一种理想方式。

看一下这个例子：

	// 定义空对象
	var dog = {};

	// 添加一个属性
	dog.name = "Benji";

	// 添加一个方法
	dog.getName = function () {
		return dog.name;
	};

在这个例子中，我们首先定义了一个空对象，然后添加了一个属性和一个方法，在程序的生命周期内的任何时刻都可以：

- 更改属性和方法的值，比如：

	dog.getName = function () {
		// 重新定义方法，返回一个硬编码的值
		return "Fido";
	};

- 删除属性/方法

	delete dog.name;

- 添加更多的属性和方法

	dog.say = function () {
		return "Woof!";
	};
	dog.fleas = true;

每次都创建空对象并不是必须的，对象字面量模式可以直接在创建对象时添加功能，就像下面这个例子：

	var dog = {
		name: "Benji",
		getName: function () {
			return this.name;
		}
	};

> 在本书中多次提到“空对象”（“blank object”和“empty object”），这只是一种简称，在JavaScript中根本不存在真正的空对象，理解这一点至关重要。即使最简单的`{}`对象也会包含从`Object.prototype`继承来的属性和方法。我们提到的“空（empty）对象”只是说这个对象没有自有属性(own properties)，不考虑它是否有继承来的属性。

### 对象字面量语法

如果你从来没有接触过对象字面量的写法，可能会感觉怪怪的。但越到后来你就越喜欢它。本质上讲，对象字面量语法包括：

- 将对象主体包含在一对花括号内（`{` 和 `}`）。
- 对象内的属性或方法之间使用逗号分隔。最后一个名值对后也可以有逗号，但在IE下会报错，所以尽量不要在最后一个属性或方法后加逗号。
- 属性名和值之间使用冒号分隔。
- 如果将对象赋值给一个变量，不要忘了在右括号`}`之后补上分号。

### 通过构造函数创建对象

JavaScript中没有类的概念，这给JavaScript带来了极大的灵活性，因为你不必提前知晓关于对象的任何信息，也不需要类的“蓝图”（译注：指类的结构）。但JavaScript同样具有构造函数，它的语法和Java或其他语言中基于类的对象创建非常类似。

你可以使用自定义的构造函数来创建对象实例，也可以使用内置构造函数来创建，比如`Object()`、`Date()`、`String()`等等。

下面这个例子展示了用两种等价的方法分别创建两个独立的实例对象：

	// 一种方法，使用字面量
	var car = {goes: "far"};

	// 另一种方法，使用内置构造函数
	// 注意：这是一种反模式
	var car = new Object();
	car.goes = "far";

从这个例子中可以看到，字面量写法的一个明显优势是，它的代码更少。“创建对象的最佳模式是使用字面量”还有一个原因，它可以强调对象就是一个简单的可变的散列表，而不必一定派生自某个类。

另外一个使用字面量而不是`Object()`构造函数创建实例对象的原因是，对象字面量不需要“作用域解析”（scope resolution）。因为存在你已经创建了一个同名的构造函数`Object()`的可能，当你调用`Object()`的时候，解析器需要顺着作用域链从当前作用域开始查找，直到找到全局`Object()`构造函数为止。

### `Object()`构造函数的参数

> 译注：这小节的标题是Object Constructor Catch，恕译者水平有限，实在不知如何翻译，故自作主张修改了本节标题。

创建实例对象时能用对象字面量就不要使用`new Object()`构造函数，但有时你可能是在别人写的代码基础上工作，这时就需要了解构造函数的一个“特性”（也是不使用它的另一个原因），就是`Object()`构造函数可以接收参数，通过这个参数可以把对象实例的创建过程委托给另一个内置构造函数，并返回另外一个对象实例，而这往往不是你想要的。

下面的示例代码中展示了给`new Object()`传入不同的参数（数字、字符串和布尔值），最终得到的对象是由不同的构造函数生成的：

	// 注意：这是反模式

	// 空对象
	var o = new Object();
	console.log(o.constructor === Object); // true

	// 数值对象
	var o = new Object(1);
	console.log(o.constructor === Number); // true
	console.log(o.toFixed(2)); // "1.00"

	// 字符串对象
	var o = new Object("I am a string");
	console.log(o.constructor === String); // true
	// 普通对象没有substring()方法，但字符串对象有
	console.log(typeof o.substring); // "function"

	// 布尔值对象
	var o = new Object(true);
	console.log(o.constructor === Boolean); // true

`Object()`构造函数的这种特性会导致一些意想不到的结果，特别是当参数不确定的时候。最后再次提醒不要使用`new Object()`，尽可能的使用对象字面量来创建实例对象。

## 自定义构造函数

除了对象字面量和内置构造函数之外，你也可以通过自定义的构造函数来创建对象实例，正如下面的代码所示：

	var adam = new Person("Adam");
	adam.say(); // "I am Adam"

这种写法非常像Java中用`Person`类创建了一个实例，两者的语法非常接近，但实际上JavaScript中没有类的概念，`Person()`是一个函数。

`Person()`构造函数是如何定义的呢？看下面的代码：

	var Person = function (name) {
		this.name = name;
		this.say = function () {
			return "I am " + this.name;
		};
	};

当你通过`new`来调用这个构造函数时，函数体内将发生这些事情：

- 创建一个空对象，将它的引用赋给`this`，并继承函数的原型。
- 通过`this`将属性和方法添加至这个对象。
- 最后返回this指向的新对象（如果没有手动返回其他的对象）。

用代码表示这个过程如下：

	var Person = function (name) {
		// 使用对象字面量创建新对象
		// var this = {};

		// 添加属性和方法
		this.name = name;
		this.say = function () {
			return "I am " + this.name;
		};

		//return this;
	};

上例中，为简便起见，`say()`方法被添加至`this`中，结果就是不论何时调用`new Person()`，在内存中都会创建一个新函数（`say()`），显然这是效率很低的，因为所有实例的`say()`方法是一模一样的。最好的办法是将方法添加至`Person()`的原型中。

	Person.prototype.say = function () {
		return "I am " + this.name;
	};

我们将会在下一章里详细讨论原型和继承，现在只要记住将需要重用的成员放在原型里即可。

关于构造函数的内部工作机制也会在后续章节中有更细致的讨论。这里我们只做概要的介绍。刚才提到，构造函数执行的时候，首先创建一个新对象，并将它的引用赋给`this`：

	// var this = {};

其实事实并不完全是这样，因为“空”对象并不是真的空，这个对象继承了`Person`的原型，看起来更像：

	// var this = Object.create(Person.prototype);

在后续章节会进一步讨论`Object.create()`。

### 构造函数的返回值

当使用`new`调用的时候，构造函数总是会返回一个对象，默认情况下返回`this`所指向的对象。如果构造函数内没有给`this`赋任何属性，则返回一个“空”对象（除了继承构造函数的原型之外，没有自有属性）。

尽管在构造函数中没有`return`语句的情况下，也会隐式返回`this`。但事实上我们是可以返回任意指定的对象的，在下面的例子中就返回了新创建的`that`对象。

	var Objectmaker = function () {

		// name属性会被忽略，因为返回的是另一个对象
		this.name = "This is it";

		// 创建并返回一个新对象
		var that = {};
		that.name = "And that's that";
		return that;
	};

	// 测试
	var o = new Objectmaker();
	console.log(o.name); // "And that's that"

可以看到，构造函数中其实是可以返回任意对象的，只要你返回的东西是对象即可。如果返回值不是对象（字符串、数字或布尔值），程序不会报错，但这个返回值被忽略，最终还是返回`this`所指的对象。

## 强制使用new的模式

我们知道，构造函数和普通的函数本质一样，只是通过`new`调用而已。那么如果调用构造函数时忘记`new`会发生什么呢？漏掉`new`不会产生语法错误也不会有运行时错误，但可能会造成逻辑错误，导致执行结果不符合预期。这是因为如果不写`new`的话，函数内的`this`会指向全局对象（在浏览器端`this`指向`window`）。

当构造函数内包含`this.member`之类的代码，并直接调用这个函数（省略`new`），实际上会创建一个全局对象的属性`member`，可以通过`window.member`或`member`访问到。这不是我们想要的结果，因为我们要努力确保全局命名空间干净。

	// 构造函数
	function Waffle() {
		this.tastes = "yummy";
	}

	// 新对象
	var good_morning = new Waffle();
	console.log(typeof good_morning); // "object"
	console.log(good_morning.tastes); // "yummy"

	// 反模式，漏掉new
	var good_morning = Waffle();
	console.log(typeof good_morning); // "undefined"
	console.log(window.tastes); // "yummy"

ECMAScript5中修正了这种出乎意料的行为逻辑。在严格模式中，`this`不再指向全局对象。如果在不支持ES5的JavaScript环境中，也有一些方法可以确保有没有`new`时构造函数的行为都保持一致。

### 命名规范

一种简单解决上述问题的方法就是命名规范，前面的章节已经讨论过，构造函数首字母大写（`MyConstructor()`），普通函数和方法首字母小写（`myFunction`）。

### 使用that

遵守命名规范有一定的作用，但规范毕竟不是强制，不能完全避免出现错误。这里给出了一种模式可以确保构造函数一定会按照构造函数的方式执行，那就是不要将所有成员添加到`this`上，而是将它们添加到`that`上，并返回`that`。

	function Waffle() {
		var that = {};
		that.tastes = "yummy";
		return that;
	}

如果要创建更简单一点的对象，甚至不需要局部变量`that`，直接返回一个对象字面量即可，就像这样：

	function Waffle() {
		return {
			tastes: "yummy"
		};
	}

不管用什么方式调用它（使用`new`或直接调用），它都会返回一个实例对象：

	var first = new Waffle(),
		second = Waffle();
	console.log(first.tastes); // "yummy"
	console.log(second.tastes); // "yummy"

这种模式的问题是会丢失原型，因此在`Waffle()`的原型上的成员不会被继承到这些对象中。

> 需要注意的是，这里用的`that`只是一种命名规范，`that`并不是语言特性的一部分，它可以被替换为任何你喜欢的名字，比如`self`或`me`。

### 调用自身的构造函数

为了解决上述模式的问题，能够让对象继承原型上的属性，我们使用下面的方法：在构造函数中首先检查`this`是否是构造函数的实例，如果不是，则通过`new`再次调用自己：

	function Waffle() {

		if (!(this instanceof Waffle)) {
			return new Waffle();
		}
		this.tastes = "yummy";

	}
	Waffle.prototype.wantAnother = true;

	// 测试
	var first = new Waffle(),
		second = Waffle();

	console.log(first.tastes); // "yummy"
	console.log(second.tastes); // "yummy"

	console.log(first.wantAnother); // true
	console.log(second.wantAnother); // true

还有一种比较通用的用来检查实例的方法是使用`arguments.callee`，而不是直接将构造函数名写死在代码中：

	if (!(this instanceof arguments.callee)) {
		return new arguments.callee();
	}

这种模式利用了一个事实，即在任何函数内部都会创建一个`arguments`对象，它包含函数调用时传入的参数。同时`arguments`包含一个`callee`属性，指向正在被调用的函数。需要注意，ES5严格模式中已经禁止了`arguments.callee`的使用，因此最好对它的使用加以限制，并尽可能删除现有代码中已经用到的地方。

## 数组字面量

和JavaScript中大多数“东西”一样，数组也是对象。可以通过内置构造函数`Array()`来创建数组，也可以通过字面量形式创建，就像对象字面量那样。而且更推荐使用字面量创建数组。

这里的示例代码给出了创建两个具有相同元素的数组的两种方法，使用`Array()`和使用字面量模式：

	// 有三个元素的数组
	// 注意：这是反模式
	var a = new Array("itsy", "bitsy", "spider");

	// 完全相同的数组
	var a = ["itsy", "bitsy", "spider"];

	console.log(typeof a); // "object"，因为数组也是对象
	console.log(a.constructor === Array); // true

### 数组字面量语法

数组字面量写法非常简单：整个数组使用方括号括起来，数组元素之间使用逗号分隔。数组元素可以是任意类型，包括数组和对象。

数组字面量语法简单直观而且优雅，毕竟数组只是从0开始索引的一些值的集合，完全没必要引入构造器和`new`运算符（还要写更多的代码）。

### Array()构造函数的“陷阱”

我们对`new Array()`敬而远之还有一个原因，就是为了避免构造函数带来的陷阱。

如果给`Array()`构造函数传入一个数字，这个数字并不会成为数组的第一个元素，而是设置了数组的长度。也就是说，`new Array(3)`创建了一个长度为3的数组，而不是某个元素是3。如果你访问数组的任意元素都会得到`undefined`，因为元素并不存在。下面的示例代码展示了字面量和构造函数的区别：

	// 含有1个元素的数组
	var a = [3];
	console.log(a.length); // 1
	console.log(a[0]); // 3

	// 含有3个元素的数组
	var a = new Array(3);
	console.log(a.length); // 3
	console.log(typeof a[0]); // "undefined"

构造函数的行为可能有一点出乎意料，但当给`new Array()`传入一个浮点数时情况就更糟糕了，这时会出错（译注：给new Array()传入浮点数会报“范围错误”RangError），因为数组长度不可能是浮点数。

	// 使用数组字面量
	var a = [3.14];
	console.log(a[0]); // 3.14

	var a = new Array(3.14); // RangeError: invalid array length
	console.log(typeof a); // "undefined"

为了避免在运行时动态创建数组时出现这种错误，强烈推荐使用数组字面量来代替`new Array()`。

> 有些人用`Array()`构造器来做一些有意思的事情，比如用来生成重复字符串。下面这行代码返回的字符串包含255个空格（请读者思考为什么不是256个空格）。`var white = new Array(256).join(' ');`

### 检查是否数组

如果`typeof`的操作数是数组的话，将返回“object”。

	console.log(typeof [1, 2]); // "object"

这个结果勉强说得过去，毕竟数组也是一种对象，但对我们来说这个结果却没什么用，实际上你往往是需要知道一个值是不是真正的数组。有时候你会见到一些检查数组的方法：检查`length`属性、检查数组方法比如`slice()`等等，但这些方法非常脆弱，非数组的对象也可以拥有这些同名的属性。还有些人使用`instanceof Array`来判断数组，但这种方法在某些版本的IE里的多个iframe的场景中会出问题（译注：原因就是在不同iframe中创建的数组不会相互共享其`prototype`属性）。

ECMAScript5定义了一个新的方法`Array.isArray()`，如果参数是数组的话就返回true。比如：

	Array.isArray([]); // true

	// 尝试用一个类似数组的对象去测试
	Array.isArray({
		length: 1,
		"0": 1,
		slice: function () {}
	}); // false

如果你的开发环境不支持ECMAScript5，可以通过`Object.prototype.toString()`方法来代替。如调用`toString`的`call()`方法并传入数组上下文，将返回字符串“[object Array]”。如果传入对象上下文，则返回字符串“[object Object]”。因此可以这样做：

	if (typeof Array.isArray === "undefined") {
		Array.isArray = function (arg) {
			return Object.prototype.toString.call(arg) === "[object Array]";
		};
	}

## JSON

我们刚刚讨论了对象和数组字面量，你应该很熟悉了，现在我们来看一看JSON。JSON（JavaScript Object Notation）是一种轻量级的数据交换格式，可以很容易地用在多种语言中，尤其是在JavaScript中。

JSON格式及其简单，它只是数组和对象字面量的混合写法，看一个JSON字符串的例子：

	{"name": "value", "some": [1, 2, 3]}

JSON和对象字面量在语法上的唯一区别是，合法的JSON属性名均需要用引号包含。而在对象字面量中，只有属性名是非法的标识符时才使用引号包含，比如，属性名中包含空格`{"first name": "Dave"}`。

在JSON字符串中，不能使用函数和正则表达式字面量。

### 使用JSON

在前面的章节中讲到，出于安全考虑，不推荐使用`eval()`来粗暴地解析JSON字符串。最好是使用`JSON.parse()`方法，ES5中已经包含了这个方法，并且现代浏览器的JavaScript引擎中也已经内置支持JSON了。对于老旧的JavaScript引擎来说，你可以使用JSON.org所提供的JS文件（<http://www.json.org/json2.js>）来获得JSON对象和方法。

	// 输入JSON字符串
	var jstr = '{"mykey": "my value"}';
	
	// 反模式
	var data = eval('(' + jstr + ')');

	// 更好的方式
	var data = JSON.parse(jstr);

	console.log(data.mykey); // "my value"

如果你已经在使用某个JavaScript库了，很可能这个库中已经提供了解析JSON的方法，就不必再额外引入JSON.org的库了，比如，如果你已经使用了YUI3，你可以这样：

	// 输入JSON字符串
	var jstr = '{"mykey": "my value"}';

	// 使用YUI来解析并将结果返回为一个对象
	YUI().use('json-parse', function (Y) {
		var data = Y.JSON.parse(jstr);
		console.log(data.mykey); // "my value"
	});

如果你使用的是jQuery，可以直接使用它提供的`parseJSON()`方法：

	// 输入JSON字符串
	var jstr = '{"mykey": "my value"}';

	var data = jQuery.parseJSON(jstr);
	console.log(data.mykey); // "my value"

和`JSON.parse()`方法相对应的是`JSON.stringify()`。它将对象或数组（或任何原始值）转换为JSON字符串。

	var dog = {
		name: "Fido",
		dob:new Date(),
		legs:[1,2,3,4]
	};

	var jsonstr = JSON.stringify(dog);

	// jsonstr的值为
	// {"name":"Fido","dob":"2010-04-11T22:36:22.436Z","legs":[1,2,3,4]}

## 正则表达式字面量

JavaScript中的正则表达式也是对象，可以通过两种方式创建它们：

- 使用`new RegExp()`构造函数
- 使用正则表达式字面量

下面的示例代码展示了创建用来匹配一个反斜杠（\）的正则表达式的两种方法：

	// 正则表达式字面量
	var re = /\\/gm;

	// 构造函数
	var re = new RegExp("\\\\", "gm");

显然正则表达式字面量写法的代码更短，而且不会让你觉得在用像类一样的构造函数的思想在写正则表达式，因此更推荐使用字面量写法。

另外，如果使用`RegExp()`构造函数写法，还需要考虑对引号和反斜杠进行转义，正如上段代码所示的那样，用了四个反斜杠来匹配一个反斜杠。这会增加正则表达式的长度，而且让它变得难于理解和维护。正则表达式入门不是件容易的事，所以不要放弃任何一个简化它们的机会，尽量使用字面量而不是通过构造函数来创建正则表达式。

### 正则表达式字面量语法

正则表达式字面量使用两个斜杠包裹，主体部分不包括两端的斜线。在第二个斜线之后可以指定模式匹配的修饰符，修饰符不需要用引号引起来，JavaScript中有三个修饰符：

- `g`，全局匹配
- `m`，多行匹配
- `i`，忽略大小写的匹配

修饰符可以自由组合，而且与顺序无关：

	var re = /pattern/gmi;

使用正则表达式字面量可以让代码更加简洁高效，比如当调用`String.prototype.replace()`方法时，可以传入正则表达式参数：

	var no_letters = "abc123XYZ".replace(/[a-z]/gi, "");
	console.log(no_letters); // 123

有一种不得不使用`new RegExp()`的场景，就是正则表达式是不确定，只有等到运行时才能确定下来的情况。

正则表达式字面量和构造函数还有另一个区别，就是字面量只在解析时创建一次正则表达式对象（译注：多次解析同一个正则表达式，会产生相同的实例对象）。如果在循环体内反复使用相同的字面量创建对象，则会返回第一次创建的对象以及它的属性（比如`lastIndex`）。下面这个例子展示了两次返回相同的正则表达式的情形。

	function getRE() {
		var re = /[a-z]/;
		re.foo = "bar";
		return re;
	}

	var reg = getRE(),
		re2 = getRE();

	console.log(reg === re2); // true
	reg.foo = "baz";
	console.log(re2.foo); // "baz"

> 在ECMAScript5中这种情况有所改变，相同正则表达式字面量的每次计算都会创建新的实例对象，目前很多现代浏览器也对此做了纠正。

最后需要提一点，不带`new`调用`RegExp()`（作为普通的函数）和带`new`调用`RegExp()`是完全一样的。

## 原始值的包装对象

JavaScript中有五种原始类型：数字、字符串、布尔值、`null`和`undefined`。除了`null`和`undefined`之外，其他三种都有对应的“包装对象”（primitive wrapper object）。可以通过内置构造函数`Number()`、`String()`和`Boolean()`来生成包装对象。

为了说明数字原始值和数字对象之间的区别，看一下下面这个例子：

	// 一个数字原始值
	var n = 100;
	console.log(typeof n); // "number"

	// 一个Number对象
	var nobj = new Number(100);
	console.log(typeof nobj); // "object"

包装对象带有一些有用的属性和方法。比如，数字对象就带有`toFixed()`和`toExponential()`之类的方法，字符串对象带有`substring()`、`chatAt()`和`toLowerCase()`等方法以及`length`属性。这些方法非常方便，和原始值相比，这是包装对象的优势，但其实原始值也可以调用这些方法，因为原始值会首先转换为一个临时对象，如果转换成功，则调用包装对象的方法。

	// 像使用对象一样使用一个字符串原始值
	var s = "hello";
	console.log(s.toUpperCase()); // "HELLO"

	// 值本身也可以像对象一样
	"monkey".slice(3, 6); // "key"

	// 数字也是一样
	(22 / 7).toPrecision(3); // "3.14"

因为原始值可以根据需要转换成对象，这样的话，也不必为了用包装对象的方法而将原始值手动“包装”成对象。比如，不必使用new String("hi")，直接使用"hi"即可。

	// 避免这些：
	var s = new String("my string");
	var n = new Number(101);
	var b = new Boolean(true);

	// 更好更简洁的办法：
	var s = "my string";
	var n = 101;
	var b = true;

不得不使用包装对象的一个场景是，有时我们需要对值进行扩充并保持值的状态。原始值毕竟不是对象，不能直接对其进行扩充。

	// 字符串原始值
	var greet = "Hello there";

	// 为使用split方法，原始值被转换为对象
	greet.split(' ')[0]; // "Hello"

	// 给原始值添加属性并不会报错
	greet.smile = true;

	// 但实际上却没有作用
	typeof greet.smile; // "undefined"

在这段示例代码中，`greet`只是临时被转换成了对象，以保证访问其属性、方法时不会出错。而如果是另一种情况，`greet`通过`new String()`被定义为一个对象，那么扩充`smile`属性的过程就会像我们预期的那样。对字符串、数字或布尔值进行扩充的情况很少见，因此建议只在确实有必要的情况下使用包装对象。

当省略`new`时，包装对象的构造函数将传给它的参数转换为原始值：

	typeof Number(1); // "number"
	typeof Number("1"); // "number"
	typeof Number(new Number()); // "number"
	typeof String(1); // "string"
	typeof Boolean(1); // "boolean"

## 错误处理对象

JavaScript中有很多内置的错误处理构造函数，比如`Error()`、`SyntaxError()`，`TypeError()`等等，它们通常和`throw`语句一起被使用。这些构造函数创建的错误对象包含这些属性：

- `name`

	name属性是指产生这个对象的构造函数的名字，通常是“Error”，有时会有特定的名字比如“RangeError”

- `message`

	创建这个对象时传入构造函数的字符串

错误对象还有一些其他的属性，比如产生错误的行号和文件名，但这些属性是浏览器自行实现的，不同浏览器的实现也不一致，因此出于兼容性考虑，并不推荐使用这些属性。

`throw`可以抛出任何对象，并不限于“错误对象”，因此你可以根据需要抛出自定义的对象。这些对象包含属性“name”和“message”或其他你希望传递给异常处理逻辑的信息，异常处理逻辑由`catch`语句指定。你可以灵活运用抛出的错误对象，将程序从错误状态恢复至正常状态。

	try {
		// 一些不好的事情发生了，抛出错误
		throw {
			name: "MyErrorType", // 自定义错误类型
			message: "oops",
			extra: "This was rather embarrassing",
			remedy: genericErrorHandler // 应该由谁处理
		};
	} catch (e) {
		// 通知用户
		alert(e.message); // "oops"

		// 优雅地处理错误
		e.remedy(); // 调用genericErrorHandler()
	}

使用`new`调用和省略`new`调用错误构造函数是一模一样的，他们都返回相同的错误对象。

## 小结

在本章里，我们讨论了多种字面量模式，它们是使用构造函数写法的替代方案，本章讲述了这些内容：

- 对象字面量写法——一种简洁优雅的定义对象的方法，通过花括号包裹，名值对之间用逗号分隔
- 构造函数——内置构造函数（内置构造函数通常都有对应的字面量语法）和自定义构造函数
- 一种强制函数以构造函数的模式运行行（不管用不用`new`调用构造函数，都始终返回`new`出来的实例）的技巧
- 数组字面量写法——通过方括号包裹，数组元素之间使用逗号分隔
- JSON——一种轻量级的数据交换格式
- 正则表达式字面量
- 避免使用其他的内置构造函数：`String()`、`Number()`、`Boolean()`以及不同种类的`Error()`构造函数

通常情况下，除了`Date()`之外，其他的内置构造函数并不常用，下面的表格对这些构造函数以及它们的字面量语法做了整理。

<table>
	<tr>
		<td>内置构造函数（不推荐）</td>
		<td>字面量语法和原始值（推荐）</td>
	</tr>
	<tr>
		<td>var o = new Object();</td>
		<td>var o = {};
		</td>
	</tr>
	<tr>
		<td>var a = new Array();
		</td>
		<td>var a = [];
		</td>
	</tr>
	<tr>
		<td>var re = new RegExp("[a-z]","g"); 
		</td>
		<td>var re = /[a-z]/g;
		</td>
	</tr>
	<tr>
		<td>var s = new String(); 
		</td>
		<td>var s = "";
		</td>
	</tr>
	<tr>
		<td>var n = new Number(); 
		</td>
		<td>var n = 0;
		</td>
	</tr>
	<tr>
		<td>var b = new Boolean();
		</td>
		<td>var b = false;
		</td>
	</tr>
	<tr>
		<td>throw new Error("uh-oh");
		</td>
		<td>throw { name: "Error",message: "uh-oh"};或者throw Error("uh-oh");
		</td>
	</tr>
</table>


