# Learning Kotlin

[toc]

# 基础

## 变量声明

* val: 变量只能赋值一次

* var: 变量可以赋值多次

## 类型推断

：后加上类型，也可以不写类型，kotlin会根据所赋值的类型来推断。

但是编写接口和公共API时建议保留类型。

```kotlin
var count: Int = 10
var count2 = 10

val str: String = "test string"
val str2 = "test string"
```

## Null安全

变量类型后加上？后缀，才可以将变量指定为可以为null

这样声明是错的：

```kotlin
val languageName: String = null
```

这样声明才对：

```kotlin
val languageName: String? = null
```

这个规则可以使代码更安全且简洁，可降低NullPointerException的几率，可减少代码中进行null检查的次数。

### 与java的映射

java的类中不带注释的成员，编译器将不知道映射到kotlin中类型有没有?，这种不确定性通过**平台类型!**表示。

```java
public class Account {
  public String name; // ->kotlin中String!
  public @Nullable String type; // ->kotlin中String?
  public @NonNull String id; // ->kotlin中String
}
```

String!可以表示String或者String?，编译器可让你赋予任一类型的值。

所以建议写java类的时候，加上@Nullable或者@NonNull的注释。

### 处理Null

```kotlin
class Account(var name: String?, var type: String?)
val account = Account("name", "type")
```

#### 1. `!!`运算符

`val accountName = account.name!!.trim()`  如果name为null则会抛出NullPointerException

#### 2. `?.`安全调用符

`val accountName = account.name?.trim() ` 如果name为null则`name?.trim()`的结果为null。

这意味着`?.`可以避免NullPointerException，将null传递给下一个语句。

#### 3. `?:`Elvis运算符

`val accountName = account.name?.trime() ?: "Default Name"` 

如果name为null则`name?.trim()`的结果为null，则accountName值为"Default Name"。

还可以用elvis运算符提前return:

```kotlin
fun validateAccount(account: Account?) {
    val accountName = account?.name?.trim() ?: "Default name"
    // account cannot be null beyond this point
    account ?: return
    ...
}
```

#### 4. `as?`

```kotlin 
val y = 66
val x: String? = y as? String
```

如果类型转换失败，则返回null，而不是抛出ClassCastException

**一句话总结： **

**当不是null的情况下，?和!没作用；**

**当是null的情况下，有?则把null传递给下一个语句，没有?则抛出异常。**

#### 5. 集合中非null元素

使用`filterNotNull`API

```kotlin
val nullableList: List<Int?> = listOf(1, 2, null, 4)
val intList: List<Int> = nullableList.filterNotNull()
```



## 初始化

* 声明对象时如果不能赋值，则需要加上`lateinit`关键字，或者在init中赋值.

这样声明是错的：

```kotlin
var param: String
```

1. 要么赋值：

`val param: String = "name"`

或者暂时没有值的话，只能先声明为?并赋值null: `val param: String? = null`，这种情况建议用lateinit关键字

2. 要么加上lateinit关键字：

`lateinit var param: String`

**使用lateinit时，应尽快初始化属性。lateinit适用于预计从不为null但在其定义位置无法初始化的变量。**

* lateinit 必须是var不能是val, 必须是非null的，非原始类型

```kotlin
lateinit var param: String // correct
lateinit val param: String // wrong! 不能是val
lateinit var count: Int // wrong! 不能是原始类型
lateinit var param: String? // wrong! 不能是可为null
```

* 检查是否init了可以用.isInitialized

```kotlin
class TestLateInit() {
  lateinit var param: String
  fun test() {
    if (::param.isInitialized) {
      // do something
    }
  }
}
```

3. 要么在constructor/init中赋值：

```kotlin
class Car {
  var param: String
  init {
    param = "name"
  }
}
```

* 在属性初始化之前访问它的话，会抛出`UninitializedPropertyAccessException`

## 条件语句

* kotlin不包含传统的三元运算符，而是提供了条件表达式

### if-else

```kotlin
val answerString: String = if (count == 42) {
    "I have the answer."
} else if (count > 35) {
    "The answer is close."
} else {
    "The answer eludes me."
}
```

### when

这个示例中的代码功能上跟上一个if-else中的代码**等效**:

```kotlin
val answerString = when {
	count == 42 -> "I have the answer."
	count > 35 -> "The answer is close."
	else -> "The answer eludes me."
}
```

when的其他写法：

```kotlin
answerStr = when (count) {
  42 -> "correct"
  35 -> "smaller"
  else -> ""
}

when (count) {
  0 -> return
  -1 -> println()
}
```

## 函数

* 格式

fun 函数名(输入参数): 返回类型 {

}

```kotlin
fun generateAnswerString(count: Int): String {
    val answerString = if (count == 42) {
        "I have the answer."
    } else {
        "The answer eludes me"
    }
    return answerString
}
```

可简化为：

```kotlin
fun generateAnswerString(count: Int): String {
    return if (count > 42) {
        "I have the answer."
    } else {
        "The answer eludes me."
    }
}
```

可将return关键字替换为赋值运算，继续简化为：

```kotlin
fun generateAnswerString(count: Int): String = if (count > 42) {
        "I have the answer"
    } else {
        "The answer eludes me"
    }
```

### 匿名函数

* 格式

val 匿名函数的引用: (输入) -> 输出 = {

}

```kotlin
val stringLengthFunc: (String) -> Int = { input ->
    input.length
}

val stringLength: Int = stringLengthFunc("Android")
```

## 高阶函数

一个函数可以将另一个函数当作参数。将其他函数用作参数的函数称为“高阶函数”。此模式对组件之间的通信（其方式与在 Java 中使用回调接口相同）很有用。

### todo

## 类

```kotlin
class Car {
    val wheels = listOf<Wheel>()
}
```

wheels是public属性

调用构造函数不需要new关键字

```kotlin
val car = Car() // construct a Car
val wheels = car.wheels // retrieve the wheels value from the Car
```

属性的set可以自定义是否public (get方法是否public跟属性相同)

```kotlin
class Car {
        var count: Int = 15
        private set
    }
```

## SAM转换

SAM转换可以使代码更简洁。

例如，setOnClickListener()接收OnClickListener作为参数，且OnClickListener有单一抽象方法onClick()，所以在kotlin中可以使用匿名函数来表示。

```kotlin
loginButton.setOnClickListener {
  // it 表示onClick(View v)中传入的参数v
  it.isEnabled = false
}
```

## 伴生对象

类似于java中的static，与某个类型相关但不与某个特定对象关联的变量或函数。

```kotlin
class LoginFragment : Fragment() {
    ...
    companion object {
        private const val TAG = "LoginFragment"
    }
}
```

### todo

Const 标量??

后备属性？？

## 属性委托

初始化属性时，您可能会重复 Android 的一些比较常见的模式，例如在 `Fragment` 中访问 `ViewModel`。为避免过多的重复代码，您可以使用 Kotlin 的属性委托语法。

```kotlin
private val viewModel: LoginViewModel by viewModels()
```

属性委托使用反射，这样会增加一些性能开销。这种代价换来的是简洁的语法，可让您节省开发时间。

### todo

## 其他

kotlin与java可以直接互相调用。

每个语句都后跟一个换行符，不使用分号`；`。

### 格式

当函数签名不适合放在一行上时，应让每个参数声明独占一行。以这种格式定义的参数应使用单缩进 (+4)。右圆括号 (`)`) 和返回类型独占一行，没有额外的缩进。

```kotlin
fun <T> Iterable<T>.joinToString(
    separator: CharSequence = ", ",
    prefix: CharSequence = "",
    postfix: CharSequence = ""
): String {
    // …
}
```

### 表达式函数

当函数只包含一个表达式时，可以使用表达式函数

```kotlin
fun getUpperCaseName(): String? = this.name?.toUpperCase()
```

# Kotlin Koans

## 默认参数

函数的默认参数可以简化调用方的输入。

例如joinToString已有如下默认参数：

```kotlin
fun <T> Array<out T>.joinToString(
    separator: CharSequence = ", ", 
    prefix: CharSequence = "", 
    postfix: CharSequence = "", 
    limit: Int = -1, 
    truncated: CharSequence = "...", 
    transform: ((T) -> CharSequence)? = null
): String
```

当我想拼接一个字符串格式是`[a,b,c]`时，只需要输入prefix和postfix即可，其他参数不用输入

```kotlin
fun joinOptions(options: Collection<String>) =
        options.joinToString(prefix = "[", postfix = "]")
```

## String

### 拼接

1. `+`拼接

```kotlin
val s1 = "abc" + 1 + "def" // s1 = "abc1def"
```

2. `$`模版拼接

```kotlin
val s = "abc"
println("$s.length is ${s.length}") // 输出abc.length is 3
```

如果字符串中要包含`$`, 需要用`"""${'$'}"""`转义

```kotlin
val s = "abc"
val s2 = "$s" // s2 = "abc"
val s3 = """
        ${'$'}s
    """.trimIndent() // s3 = "$s"
```

### trim

#### trimIntent

找到开头空格最少的一行（这行空格数为n个），每一行都去掉开头的n个空格

```kotlin
val text = """
                                for (c in "foo")
                                    print(c)
        """.trimIndent()
println(text)
val text2 = """
                for (c in "foo")
                        print(c)    
        """.trimIndent()
println(text2)
```

输出

```kotlin
for (c in "foo")
    print(c)
for (c in "foo")
        print(c) 
```

#### trimMargin

接收一个字符串参数，每一行开头都去除这个字符串。如果开头不匹配该字符串，则这行不做处理。

```kotlin
val s = """
        ###test
        ##hello
    """.trimMargin("##")
```

输出

```kotlin
#test
hello
```

#### pattern

```kotlin
val month = "(JAN|FEB|MAR|APR|MAY|JUN|JUL|AUG|SEP|OCT|NOV|DEC)"
fun getPattern(): String = """\d{2} $month \d{4}"""
println("13 JUN 1992".matches(getPattern().toRegex()))
```

## Class

创建类的对象不需要new关键字，直接调用构造函数即可。

```kotlin
class Person(name: String) {}
val person = Person("LiLei")
```

### Constructors

类有一个主构造函数和多个次构造函数。

#### Primary constructor

主构造函数写在类名之后：

`class Person constructor(name: String) {}`

可以省略constructor关键字：

`class Person(name: String) {}`

没有参数的话可以省略括号：

`class Person() {}`等于`class Person {}`

主构造函数的代码写在init代码块中，多个init代码块按照顺序执行：

```kotlin
class Person {
  init {
    println("call init1")
  }
  init {
    println("call init2")
  }
}
```

主函数的参数可以在init代码块中直接使用：

```kotlin
class Person(name: String) {
  init {
    println("call init $name")
  }
}
```

主函数的参数可以被声明为val和var，默认为val:

`class Person(name: String)`等于`class Person(val name: String)`

如果name是可变的：
```kotlin
class Person(var name: String) {
  init {
    name = "change"
  }
}
```

#### Secondary constructors

次构造函数写在类主体中，每个次构造函数都要直接或间接地委托主构造函数，运行次构造函数时会**先**运行其委托的构造函数。

```kotlin
class Person() {
  constructor(name: String): this() {}
  constructor(name: String, index: Int): this(name) {}
}
```

如果类名后省略了主构造函数的声明，则次构造函数也要省略`: this()`

```kotlin
class Person {
  constructor(name: String) {}
  constructor(name: String, index: Int): this(name) {}
}
```

调用举例：

```kotlin
class TryKotlinClass (name: String) {
    constructor(name: String, name2: String): this(name) {
        println("call constructor2 $name $name2")
    }
    constructor(name: String, name2: String, index: Int): this(name, name2) {
        println("call constructor3 $name $name2 $index")
    }
    init {
        println("call init1 $name")
    }
    init {
        println("call init2 $name")
    }
}

fun main(args: Array<String>) {
    TryKotlinClass("haha", "oo", 0)
}
```

输出：

```kotlin
call init1 haha
call init2 haha
call constructor2 haha oo
call constructor3 haha oo 0
```

#### Inheritance

所有的类都继承父类`Any`，`Any`包含三个方法：`equals()`, `hashCode()`, `toString()`

类默认都是final不可被继承的，如果要被继承，需要加上关键字`open`: `open class Base {}`

如果子类有主构造函数，则必须写上父类的主构造函数：

```kotlin
class ChildClass(str: String) : FatherClass(str) {}
class ChildClass(str: String) : FatherClass() {}
```

如果父类没有主构造函数，则要写次构造函数：

```kotlin
class MyView : View {
    constructor(ctx: Context) : super(ctx)
    constructor(ctx: Context, attrs: AttributeSet) : super(ctx, attrs)
}
```

### Overriding

有`open`关键字的方法才能被override，子类override了父类的方法，又不想被孙子类再override，可以加上`final`关键字。

```kotlin
open class FatherClass {
  open fun action() {}
  open fun doSomething() {}
  fun close() {}
}

open class ChildClass : FatherClass() {
  override fun action() {} // correct
  final override fun doSomething() {} // correct
  override fun close() {} // wrong!
  fun close() {} // wrong!
}

open class GrandChildClass : ChildClass() {
  override fun action() {} // correct
  override fun doSomething() {} // wrong!
}
```

属性也可以override, **要注意的是可以把`val` override成`var`, 但不能反过来**

```kotlin
open class FatherClass {
  open val name: String = ""
  open var count: Int = 0
}

class ChildClass : FatherClass() {
  override var name = "child" // correct
  override val count = 1 // wrong
}
```

内部类对外部类方法的调用：`super@OutClassName.function()`

```kotlin
class OutClass {
  fun test() {}
  inner class InnerClass {
    fun innerTest() {
      super@OutClass.test()
    }
  }
}
```

如果类继承的父类和接口都包含同样签名的方法，在调用super时需要用`<>`指定调用的是哪里的方法：

```kotlin
open class Rectangle {
    open fun draw() {}
}

interface Polygon {
    fun draw() {}
}

class Square() : Rectangle(), Polygon {
    override fun draw() {
        super<Rectangle>.draw() // call to Rectangle.draw()
        super<Polygon>.draw() // call to Polygon.draw()
    }
}
```

### data class

```kotlin
data class User(var name: String, var age)
```

编译器会自动生成equals()/hashCode(), toString(), componentN(), copy()

## Properties

定义：

`var <propertyName>[: <PropertyType>] [= <property_initializer>]
    [<getter>]
    [<setter>]`

get和set前可加private property

举例：

```kotlin
var value = -1
	get() = value + 1
  set(value) {
    field = value + 1
  }
```

对value的引用会调用get, 赋值会调用set

```kotlin
var index: Int = 0
  get() = field + 1
  set(value) {
    field = value + 1
  }
override fun test() {
  println("index is $index")
  index = 3
  println("index is $index")
}
```

输出：

```kotlin
index is 1
index is 5
```

### todo

用`::`对属性引用











# Android开发中常见使用

## Fragment

```kotlin
class TestKotlinFragment : Fragment() {
  override fun onCreateView(
        inflater: LayoutInflater, 
    		container: ViewGroup?,
        savedInstanceState: Bundle?
    ): View? {
        return inflater.inflate(R.layout.fragment_test_kotlin, container, false)
    }
  override fun onViewCreated(view: View, savedInstanceState: Bundle?) {
  	super.onViewCreated(view, savedInstanceState)
	}
}
```





































































