---
title: Dart入门？一篇文章足矣
date: 2019-08-20 17:54:54
tags: 
- Flutter
- Dart
categories:
- Flutter
- Dart
toc: true
---
近期公司准备启动新项目，经过技术团队一番调研，决定采用 Flutter 来开发第一版App，故作此文，常来回顾温习。由于项目采用敏捷开发模式，故本文主要总结和记录 Dart 常用语法，更多高级和生僻用法将在后面开发过程中不定期更新。

## First of all

在我们正式接触 Dart 语法之前，需要铭记以下内容，这将会对后续 Dart 语法的学习、理解和应用有很大帮助：

- 万物皆**对象**， 每个对象都是一个类的实例。在 Dart 中，甚至连数字、方法和 `null` 都是对象，并且所有的对象都继承于 [Object](https://api.dartlang.org/stable/dart-core/Object-class.html) 类。
- 尽管 Dart 语言是一种强类型语言，但你在类型声明时仍然可以不指定类型，因为 Dart 可以自动进行类型推断。如在代码 `var number = 5;` 中，`number` 变量的类型就被推断为 `int`，当你并不想显式地声明类型时，你可以使用特有的类型 [dynamic](https://www.dartlang.org/guides/language/effective-dart/design#do-annotate-with-object-instead-of-dynamic-to-indicate-any-object-is-allowed) 来标识。
- Dart 语言同样支持泛型，如 `List<int>`、`List<dynamic>`（同 Java 中的 `List<Object>`）。
- Dart 语言支持顶级方法（即不与类绑定的方法，如上的 `main` 方法），以及绑定类和实例的方法（分别对应静态方法和实例方法），而且还支持方法嵌套（同 Python 和 JS）。
- 同样，Dart 还支持顶级变量，以及在类中定义的变量（如静态变量和实例变量）。<!--more-->
- 和 Java 不同的是，Dart 没有 `public`、 `protected` 以及  `private` 关键字。它通过给变量或者方法名加上 `_` 前缀来将其标识为私有域。

## 变量

你可以通过 `var` 或者特定的类型来修饰变量，如果变量的类型不是固定的，也可以通过 `dynamic` 或者 `Object` 来修饰。如：

```dart
// 类型推断
var name = 'Bob';
// 指定类型
String name = 'Bob';
// 类型不局限于string
dynamic name = 'Bob';
```

#### 默认值

Dart 中声明时如果未初始化，会被自动初始化为`null`：

```dart
int lineCount;
assert(lineCount == null);
```

> 注：`assert` 只在开发环境有效，在实际的生产环境无效。

#### final 与 const

`final` 和 `const` 标识该变量只能被赋值一次，区别在于 `final` 是运行时赋值，`const` 是编译时赋值。

```dart
final String nickname = 'Jack';
// final修饰时, 可省略类型声明
final name = 'Jack';
```

`const`不仅可以用来声明变量，还能用来创建常量值，如：

```dart
// 此时不能在对列表a进行更新操作
var a = const [1, 2];
// 注意以下 const 修饰的区别
var foo = const [];
const baz = []; // Equivalent to `const []`
...
foo = [1, 2, 3]; // Was const []
baz = [42]; // Error: Constant variables can't be assigned a value.
```

## 数据类型

Dart 语言支持的数据类型与 Java 类似，但也有所不同，下面我们来按照“求同存异”的法则来分别认识它们。

#### 数字类型

Dart 中的数字类型只有两种：**Int** 和 **double**。与 Java 不同的是，它并没有提供 **float** 类型。例如，我们可以通过以下方式来定义数字：

```dart
// 定义整型
var x = 123;
var hex = 0xDEADBEEF;
// 定义双精度类型
var y = 1.199;
var exponents = 1.42e5;

```

再来看下常用的数据转换该如何执行：

```dart
// String -> int
var a = int.parse('1');
assert(a == 1);

// String -> double
var b = double.parse('1.1');
assert(b == 1.1);

// int -> String
String c = 1.toString();
assert(c == '1');

// double -> String
String d = 3.14159.toStringAsFixed(2);
assert(d == '3.14');
```

`int` 类型可以执行传统的按位移位( `<<`，`>>` )，AND( `&` )和OR( `|` )运算符，如：

```dart
assert((3 << 1) == 6); // 0011 << 1 == 0110
assert((3 >> 1) == 1); // 0011 >> 1 == 0001
assert((3 | 4) == 7);  // 0011 | 0100 == 0111
```

#### 字符串类型

字符串的声明可以通过单引号或者双引号来引入：

```dart
var s1 = 'Single quotes work well for string literals.';
var s2 = "Double quotes work just as well.";
var s3 = 'It\'s easy to escape the string delimiter.';
var s4 = "It's even easier to use the other delimiter.";
```

字符串拼接可以通过 `+` 或者 `换行` 来实现，如：

```dart
var a = '123' + '456'
var b = '123'
    '456';
assert(a == b)
print(a);
// Out: 123456
```

另外，可以通过三重引号来修饰多行字符串（维持特定格式）：

```dart
var s1 = '''
You can create
multi-line strings like this one.
''';

var s2 = """This is also a
multi-line string.""";
```

#### 布尔类型

Dart 中的 `bool` 类型可以通过 `if-else` 或者是 `assert` 来检查，如：

```dart
// Check for an empty string.
var fullName = '';
assert(fullName.isEmpty);

// Check for zero.
var hitPoints = 0;
assert(hitPoints <= 0);

// Check for null.
var unicorn;
assert(unicorn == null);

// Check for NaN.
var iMeantToDoThis = 0 / 0;
assert(iMeantToDoThis.isNaN);
```

#### 数组类型

在 Dart 语言中，数组即列表，列表即数组。

```dart
void main() {
  var list = [1, 2, 3];
  list.add(4);
  list.remove(3);
  list[1] = 5;
  var last = list[list.length - 1];
  print('list is $list');
  print('last is $last');
}
```

另外，你可以通过 `…` 或者 `...?`（避免空异常） 来批量插入多个元素：

```dart
var list1 = [1, 2, 3];
var list2 = [0, ...list1];

var list3;
var list24 = [0, ...?list3];
```

除此以外，还可以通过 `if` 或者 `for` 条件运算符来插入元素：

```dart
var nav = [
  'Home',
  'Furniture',
  'Plants',
  if (promoActive) 'Outlet'
];

var listOfInts = [1, 2, 3];
var listOfStrings = [
  '#0',
  for (var i in listOfInts) '#$i'
];
assert(listOfStrings[1] == '#1');
```

#### Set 和 Map

Dart中的Set是无序的唯一项的集合。要创建一个空集合，有以下两种方式：

```dart
var names = <String>{};
Set<String> names = {}; // This works, too.
// var names = {}; // Creates a map, not a set.
```

通过以下方式创建常量 Set：

```dart
final constantSet = const {
  'fluorine',
  'chlorine',
  'bromine',
  'iodine',
  'astatine',
};
// constantSet.add('helium'); // Uncommenting this causes an error.
```

至于 Map 存储的是键值对数据，`key` 和 `value` 可以是任何类型。可以通过以下方式创建一个 Map：

```dart
var gifts = Map();
gifts['first'] = 'partridge';
gifts['second'] = 'turtledoves';
gifts['fifth'] = 'golden rings';

// 或者通过以下方式
var gifts = {
  // Key:    Value
  'first': 'partridge',
  'second': 'turtledoves',
  'fifth': 'golden rings'
};
```

## 函数

Dart 是一门完全面向对象的语言，即使它的函数也是一个对象，并且有自己的类型—— `Function`。这就意味着**函数可以被赋值为一个变量，或者作为一个参数在其他函数之间传递**。

对于只包含一个表达式的方法体，你也可以使用 `=>` 的简写形式，如：

```dart
void say() {
    print('123');
}

// 该写法与以上有同样效果
void say2() => print('123');
```

此外，函数同样支持类型推断，如：

```dart
bool isNoble(int atomicNumber) => _nobleGases[atomicNumber] != null;

// 与下面的效果一样
isNoble(atomicNumber) => _nobleGases[atomicNumber] != null;
```

#### 可选参数

与 kotlin 语言类似，Dart 的 函数同样支持可选参数这一功能，主要分为**命名可选参数**和**位置可选参数**，它们不能在一个函数中同时使用。但**它们都支持默认参数设置**。

- 命名可选参数

  命名可选参数通过花括号 `{}` 来指定可选的命名参数，如：

  ```dart
  void main() {
    hello(name: 'Tom');
  }
  
  void hello({@required String name='Jack', int age=18}) {
    print('Hello, my name is $name, i\'m $age years old.');
  }
  ```

  我们可以为命名可选参数设置默认值，在使用的时候如果不指定该可选参数的值，那么它就会自动使用默认值。另外，我们可以通过 `@required` 来标注那些不想被忽略的参数。

- 位置可选参数

  顾名思义，可选位置参数就允许我们在使用某个函数时可以忽略某个参数，如：

  ```dart
  void main() {
    sayHello('Tom', 19);
  }
  
  void sayHello(String name, int age, [String hobby = null]) {
  	var result = 'Hello, my name is $name, i\'m $age years old';
  	if (hobby != null) {
  		result += ', my bobby is $hobby.';
  	}
  	print(result);
  }
  ```

#### 匿名函数

不管什么语言，好像都能看到匿名函数的身影，我们可以通过以下方式来简单声明一个匿名函数：

```dart
var loge = (msg) => Logger.print(Logger.Error, msg);

void main() {
  loge("there has some errors");
}
```

Dart 中 list 就提供了匿名函数 —— `forEach`，如：

```dart
var list = ['apples', 'bananas', 'oranges'];
list.forEach((item) {
  print('${list.indexOf(item)}: $item');
});
```

## 常用操作符

Dart 提供了很多功能强大的操作符，这里列举几个常用的：

- **类型判断**

  如：**`is`** 相当于 Java 中的 `instanceof`

- **整除**

  如：**`a ~/ b`** 等价于 `(a /b) as int`

- **非空调用**

  如：**`a?.b`** 相当于 `a == null ? null : a.b`

- **三目运算**

  如：**`a??b`** 相当于 `a == null ? b : a`

- **三目赋值运算**

  如：**`a ??= b`** 相当于 `a = a == null ? b : a`

- **类型转换**

  如：**`a as int`** 相当于 Java 中的 `(int) a`

- **级联操作符**

  级联操作符用**`..`** 来表示，主要用来对一个对象进行连续操作，有点类似 kotlin 中的 apply 和 let，如：

  ```dart
  var button = Button(this);
  button.text = 'Confirm';
  button.color = Color.parse("#f3f355");
  button.onClick.listen((v) => toast('Confirmed'));
  
  // 等价于下面写法
  Button(this) 
    ..text = 'Confirm' // Use its members.
    ..color = Color.parse("#f3f355")
    ..onClick.listen((v) => toast('Confirmed'));
  ```

## 异常处理

#### 抛异常

在 Dart 中可以抛出非空对象（不仅仅是 `Exception` 或 `Error`）作为异常。抛出异常的方式很简单，只需要这样：

```dart
throw FormatException('Expected at least 1 section');

throw 'Out of llamas!';

void distanceTo(Point other) => throw UnimplementedError();
```

#### 捕获异常

我们可以通过 `try-on` 来捕获某个特定异常，如：

```dart
try {
  breedMoreLlamas();
} on OutOfLlamasException {
  buyMoreLlamas();
}
```

当然，你可以抛出多个类型的 `Exception`，但是由第一个 `catch` 到的分句来处理
如果 `catch` 分句没有指定类型，那么它可以处理任何类型的异常：

```dart
try {
      throw 'This a Exception!';
  } on Exception catch(e) {
    print('Unknown exception: $e');
  } catch(e) {
    print('Unknown type: $e');
  }
```

无论是否发生异常，如果想要执行某段代码，可以通过 `finally` 实现：

```dart
try {
      throw 'This a Exception!';
  } catch(e) {	// 如果去掉 catch 语句，那么异常将会在finally代码块之后传递。
    print('Catch Exception: $e');
  } finally {
    print('Close');
  }
```

## 类和对象

Dart 中，所有的对象都是类的实例，并且所有的类都是 `Object` 的子类。

#### 类的定义和构造函数

类的定义用 `class` 关键字，如果未显式定义构造函数，会默认一个空的构造函数，这一点与 Java 不谋而合。另外，Dart 还提供 **命名构造函数**这一功能，格式为：`Class.costructorName(var params)`。这种用法在某些场景下很实用，比如：

```dart
class Person{
  String name;
  int age;
  bool sex;
  String hobby;

  Person(this.name, this.age, this.sex, this.hobby);

  /// 命名构造函数
  Person.fromJson(Map json){
    print("Person constructor...");
    this.name  = json['name'];
    this.age   = json['age'];
    this.sex   = json['sex'];
    this.hobby = json['hobby'];
  }
}
```

通过命名构造函数，我就可以直接通过多种方式来生成当前类的实例，这里通过 Json 格式数据生成 `Person` 对象。另外，如果构造函数只是简单的参数传递，也可以直接在构造函数后加 `:` 来对参数进行赋值，多个参数可通过 `,` 相连：

```dart
class Point {
    num x;
    num y;
    num z;
    
    Point(this.x, this.y, z) { //第一个值传递给this.x，第二个值传递给this.y
            this.z = z;
    }
    
    Point.fromeList(var list): //命名构造函数，格式为Class.name(var param)
            x = list[0], y = list[1], z = list[2]{//使用冒号初始化变量
    }

    //当然，上面句你也可以简写为：
    //Point.fromeList(var list): this(list[0], list[1], list[2]);

     String toString() => 'x:$x  y:$y  z:$z';
}

```

如果你要创建一个不可变的对象，你可以定义编译时常量对象
需要在构造函数前加 `const`：

```dart
class ImmutablePoint {
    final num x;
    final num y;
    const ImmutablePoint(this.x, this.y); // 常量构造函数
    static final ImmutablePoint origin = const ImmutablePoint(0, 0); // 创建一个常量对象不能用new，要用const
}
```

#### 属性访问器

即我们常说的 `Setter/Getter`，主要是用来读写一个属性的方法。每个字段都对应一个隐式的 `Getter` 和 `Setter `，但是调用的时候是 `obj.name`，而不是 `obj.name()`。当然，你可以通过 `get` 和 `set` 关键字扩展功能
来实现自己的 `Setter/Getter` ， 如果字段为 `final` 或者 `const` 的话，那么它只能有一个 `getter` 方法。如：

```dart
class Rectangle {
  num left, top, width, height;

  Rectangle(this.left, this.top, this.width, this.height);

  // Define two calculated properties: right and bottom.
  num get right => left + width;
  set right(num value) => left = value - width;
  num get bottom => top + height;
  set bottom(num value) => top = value - height;
}

void main() {
  var rect = Rectangle(3, 4, 20, 15);
  assert(rect.left == 3);
  rect.right = 12;
  assert(rect.left == -8);
}
```

#### 工厂构造函数

并不是任何时候我们都需要创建一个新的对象，如：可以返回一个已经缓存好的对象或者该类的子类对象。我们以返回该类的一个缓存对象为例：

```dart
class Logger {
  final String name;
  bool mute = false;

  // _cache is library-private, thanks to
  // the _ in front of its name.
  static final Map<String, Logger> _cache =
      <String, Logger>{};

  factory Logger(String name) {
    return _cache.putIfAbsent(
        name, () => Logger._internal(name));
  }

  Logger._internal(this.name);

  void log(String msg) {
    if (!mute) print(msg);
  }
}
```

> 注意：factory 构造函数不能通过 `this` 关键字来获取类的成员。

#### 抽象类

Dart 中并没有 `interface` 关键字，只有 `abstract` 来修饰"抽象类"，但是，这里的抽象类既可以被**继承（extends）**，也可以被**实现（implements）**。如：

```dart
abstract class Person {  // 可以不用 abstract 修饰，如果加上 abstract 的话 Person 不能被实例化 
    String greet(who);   
}

class Student implements Person { 
    String name; 
    Student(this.name); 
     
    String greet(who) => 'Student: I am $name!'; 
} 

class Teacher implements Person { 
    String name; 
    Teacher(this.name); 
   
    String greet(who) => 'Teacher: I am $name!'; 
} 

void main() { 
    Person student = new Student('Wang'); 
    Person teacher = new Teacher('Lee'); 
    
    print( student.greet('Chen')); 
    print(teacher.greet('Chen')); 
}
```

同样地，Dart 中的类是**单继承，多实现**。这里使用的 `implements` 后，子类 `Student` 无法去访问父类 `Persion` 中的变量，但是抽象方法必须显式实现。相反，如果使用 `extends` 继承，子类就可以直接使用父类的非私有变量，倘若父类不是抽象类，那么子类同样不需要显式实现里面方法。如：

```dart
// A person. The implicit interface contains greet().
class Person {
  // In the interface, but visible only in this library.
  final _name;

  // Not in the interface, since this is a constructor.
  Person(this._name);

  // In the interface.
  String greet(String who) => 'Hello, $who. I am $_name.';
}

// An implementation of the Person interface.
class Impostor implements Person {
  get _name => '';

  String greet(String who) => 'Hi $who. Do you know who I am?';
}

String greetBob(Person person) => person.greet('Bob');

void main() {
  print(greetBob(Person('Kathy')));
  print(greetBob(Impostor()));
}
```

## 运算符重载

以下的运算符支持重载功能：

| `<`  | `+`  | `|`  | `[]` |
| ---- | ---- | ---- | ---- |
| `>`  | `/`  | `^`  | `[]=` |
| `<=` | `~/` | `&`  | `~`   |
| `>=` | `*`  | `<<` | `==` |
| `–`  | `%`  | `>>` |       |

这一点与 kotlin 语言中的运算符重载功能类似。我们以下面这个表示**平面向量**的类为例，为它增加两个向量相加和相减功能：

```dart
class Vector {
  final int x, y;

  Vector(this.x, this.y);

  Vector operator +(Vector v) => Vector(x + v.x, y + v.y);
  Vector operator -(Vector v) => Vector(x - v.x, y - v.y);

  // Operator == and hashCode not shown. For details, see note below.
  // ···
}

void main() {
  final v = Vector(2, 3);
  final w = Vector(2, 2);

  assert(v + w == Vector(4, 5));
  assert(v - w == Vector(0, 1));
}
```

## 枚举

枚举的使用比较简单，和 Java 中的用法类似，声明的元素 `index` 值从 `0` 开始计算。

```dart
enum Color { red, green, blue }
......
assert(Color.red.index == 0);
assert(Color.green.index == 1);
assert(Color.blue.index == 2);
```

## Mixin 混合模式

Mixins 是一种在多个类的层级中复用类中代码的一种方式。在类名后添加 `with` 关键字，并在 `with` 后紧跟**类的标识符列表**，通过逗号分隔，如：

```dart
class Maestro extends Person
    with Musical, Aggressive, Demented {
  Maestro(String maestroName) {
    name = maestroName;
    canConduct = true;
  }
}
```

如果想要声明一个混合模式，可以通过 `mixin` 来代替 `class` 定义一个不含构造方法的类：

```dart
mixin Musical {
  bool canPlayPiano = false;
  bool canCompose = false;
  bool canConduct = false;

  void entertainMe() {
    if (canPlayPiano) {
      print('Playing piano');
    } else if (canConduct) {
      print('Waving hands');
    } else {
      print('Humming to self');
    }
  }
}
```

当然，我们也可以指定某个类来使用定义的混合模式，通过 `on` 关键字：

```dart
mixin MusicalPerformer on Musician {
  // ···
}
```

## 静态变量和函数

定义静态变量和函数的方法与 Java 类似：

```dart
class Point {
  num x, y;
  Point(this.x, this.y);
  static const originalPoint = Point(0, 0);

  static num distanceBetween(Point a, Point b) {
    var dx = a.x - b.x;
    var dy = a.y - b.y;
    return sqrt(dx * dx + dy * dy);
  }
}

// usage
void main() {
  var a = Point(2, 2);
  var b = Point(4, 4);
  var distance = Point.distanceBetween(a, b);
  assert(2.8 < distance && distance < 2.9);
  print(distance);
}
```

## 异步操作

Dart 是单线程执行的，如果进行 I/O 操作或进行耗时的操作时，程序可能会发生阻塞。Dart 库中的很多函数都返回 `Future` 或者 `Stream` 对象，这些方法都是异步执行的，它们可以在建立一个耗时操作之后返回，而无需等待耗时操作执行完成。

我们可以通过两种方式来实现异步操作：

- 通过 `async` 和 `await` 实现
- 借助于 Future API 来实现

#### async 和 await

通过 `async` 和 `await` 关键字，我们可以轻松实现异步操作。要使用 `await`，必须要将函数用 `async` 修饰，同时函数会返回一个 `Future` 对象。简单用法如下：

```dart
Future checkVersion() async {
  try {
    version = await lookUpVersion();
  } catch (e) {
    // React to inability to look up the version
  }
}
```

#### 生成器

- **同步生成器 — sync***

  值得注意的是，同步生成器会返回 `Iterable` 对象：

  ```dart
  Iterable<int> naturalsTo(int n) sync* {
    int k = 0;
    while (k < n) yield k++;
  }
  ```

- **异步生成器 — async***

  异步生成器返回 `Stream` 对象：

  ```dart
  Stream<int> asynchronousNaturalsTo(int n) async* {
    int k = 0;
    while (k < n) yield k++;
  }
  ```

- **yield***

  通过 `yield*` 在递归中提升性能：

  ```dart
  Iterable<int> naturalsDownFrom(int n) sync* {
    if (n > 0) {
      yield n;
      yield* naturalsDownFrom(n - 1);
    }
  }
  ```

## 将类对象作为函数调用

Dart 允许我们**将一个类的对象作为函数来调用**，只需要实现 `call()` 函数即可：

```dart
class WannabeFunction {
  call(String a, String b, String c) => '$a $b $c!';
}

main() {
  var wf = new WannabeFunction();
  var out = wf("Hi","there,","gang");
  print('$out');
}
```

## 未完待续

> 此文只是 Dart 征程的一个起点，亦是 Flutter 中的一块基石，后续还有更多进阶用法和细节 API 等着我们去挖掘，keep learning！