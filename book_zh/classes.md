> One has no right to love or hate anything if one has not acquired a thorough
> knowledge of its nature. Great love springs from great knowledge of the
> beloved object, and if you know it but little you will be able to love it only
> a little or not at all.
>
> 如果一个人没有完全了解任何事物的本质，他就没有权利去爱或恨它。伟大的爱来自于对所爱之物的深刻了解，如果你对它知之甚少，你就只能爱一点点，或者根本不爱它。
>
> <cite>Leonardo da Vinci</cite>
> <cite>列奥纳多 · 达 · 芬奇</cite>

We're eleven chapters in, and the interpreter sitting on your machine is nearly
a complete scripting language. It could use a couple of built-in data structures
like lists and maps, and it certainly needs a core library for file I/O, user
input, etc. But the language itself is sufficient. We've got a little procedural
language in the same vein as BASIC, Tcl, Scheme (minus macros), and early
versions of Python and Lua.
我们已经完成了11章，你机器上的解释器几乎是一个完整的脚本语言实现了。它可以使用一些内置的数据结构，如列表和map，当然还需要一个用于文件IO、用户输入等的核心库。但作为语言本身已经足够了。我们有一个与BASIC、Tcl、Scheme（不包括宏）以及早期版本的Python和Lua相同的小程序语言。

If this were the '80s, we'd stop here. But today, many popular languages support
"object-oriented programming". Adding that to Lox will give users a familiar set
of tools for writing larger programs. Even if you personally don't <span
name="hate">like</span> OOP, this chapter and [the next][inheritance] will help
you understand how others design and build object systems.
如果现在是80年代，我们就可以到此为止。但是现在，很多流行的语言都支持“面向对象编程”。在Lox中添加该功能，可以为用户提供一套熟悉的工具来编写大型程序。即使你个人不喜欢OOP，这一章和下一章将帮助你理解别人是如何设计和构建对象系统的。

[inheritance]: inheritance.html

<aside name="hate">

If you *really* hate classes, though, you can skip these two chapters. They are
fairly isolated from the rest of the book. Personally, I find it's good to learn
more about the things I dislike. Things look simple at a distance, but as I get
closer, details emerge and I gain a more nuanced perspective.
但是，如果你 *真的* 讨厌类，也可以跳过这两章。它们与本书的其它部分是相当孤立的。就我个人而言，我觉得多了解自己不喜欢的对象是好事。有些事情乍一看很简单，但当我近距离观看时，细节出现了，我也获得了一个更细致入微的视角。

</aside>

## OOP 与 Classes

There are three broad paths to object-oriented programming: classes,
[prototypes][], and <span name="multimethods">[multimethods][]</span>. Classes
came first and are the most popular style. With the rise of JavaScript (and to a
lesser extent [Lua][]), prototypes are more widely known than they used to be.
I'll talk more about those [later][]. For Lox, we're taking the, ahem, classic
approach.
面向对象编程有三大途径：类(classes)、[原型(prototypes)][prototypes]和[多方法(multimethods)][multimethods]。类排在第一位，是最流行的风格。随着JavaScript（其次是[Lua][]）的兴起，原型也比以前更加广为人知。稍后我们会更多地讨论这些问题。对于Lox，我们采取的是经典的方法。

[prototypes]: http://gameprogrammingpatterns.com/prototype.html
[multimethods]: https://en.wikipedia.org/wiki/Multiple_dispatch
[lua]: https://www.lua.org/pil/13.4.1.html
[later]: #design-note

<aside name="multimethods">

Multimethods are the approach you're least likely to be familiar with. I'd love
to talk more about them -- I designed [a hobby language][magpie] around them
once and they are *super rad* -- but there are only so many pages I can fit in.
If you'd like to learn more, take a look at [CLOS][] (the object system in
Common Lisp), [Dylan][], [Julia][], or [Raku][].
Multimethods是你最不可能熟悉的方法。我很想多谈论一下它们 -- 我曾经围绕它们设计了一个[业余语言][magpie]，它们特别棒 -- 但是我只能装下这么多页面了。如果你想了解更多，可以看看[CLOS][] (Common Lisp中的对象系统), [Dylan][], [Julia][], 或 [Raku][]。

[clos]: https://en.wikipedia.org/wiki/Common_Lisp_Object_System
[magpie]: http://magpie-lang.org/
[dylan]: https://opendylan.org/
[julia]: https://julialang.org/
[raku]: https://docs.raku.org/language/functions#Multi-dispatch

</aside>

Since you've written about a thousand lines of Java code with me already, I'm
assuming you don't need a detailed introduction to object orientation. The main
goal is to bundle data with the code that acts on it. Users do that by declaring
a *class* that:
既然你已经跟我一起编写了大约1000行Java代码，我假设你不需要对面向对象进行详细介绍。OOP的主要目标就是将数据与作用于数据的代码捆绑在一起。用户通过声明一个类来实现这一点：

<span name="circle"></span>

1. Exposes a *constructor* to create and initialize new *instances* of the
   class
  暴露*构造函数*以创建和初始化该类的新实例

1. Provides a way to store and access *fields* on instances
  提供在实例上存储和访问*字段*的方法。

1. Defines a set of *methods* shared by all instances of the class that
   operate on each instances' state.
  定义一组由类的所有实例共享的*方法*，这些方法对各个实例的状态进行操作。

That's about as minimal as it gets. Most object-oriented languages, all the way
back to Simula, also do inheritance to reuse behavior across classes. We'll add
that in the [next chapter][inheritance]. Even kicking that out, we still have a
lot to get through. This is a big chapter and everything doesn't quite come
together until we have all of the above pieces, so gather your stamina.
这大概是最低要求。大多数面向对象的语言（一直追溯到Simula），也都是通过继承来跨类重用行为。我们会在[下一章][inheritance]中添加该功能。即使剔除了这些，我们仍然有很多东西需要完成。这是一个很大的章节，直到我们完成上述所有内容之后，才能把所有东西整合到一起。所以请集中精力。

<aside name="circle">

<img src="image/classes/circle.png" alt="The relationships between classes, methods, instances, constructors, and fields." />

It's like the circle of life, *sans* Sir Elton John.
这就像生命的轮回，*除了* Elton John 爵士。

</aside>

[inheritance]: inheritance.html

## 类声明

Like we do, we're gonna start with syntax. A `class` statement introduces a new
name, so it lives in the `declaration` grammar rule.
跟之前一样，我们从语法开始。`class`语句引入了一个新名称，所以它应该在`declaration` 语法规则中。

```ebnf
declaration    → classDecl
               | funDecl
               | varDecl
               | statement ;

classDecl      → "class" IDENTIFIER "{" function* "}" ;
```

The new `classDecl` rule relies on the `function` rule we defined
[earlier][function rule]. To refresh your memory:
新的`classDecl`规则依赖于前面定义的`function`规则。复习一下：

[function rule]: functions.html#function-declarations

```ebnf
function       → IDENTIFIER "(" parameters? ")" block ;
parameters     → IDENTIFIER ( "," IDENTIFIER )* ;
```

In plain English, a class declaration is the `class` keyword, followed by the
class's name, then a curly-braced body. Inside that body is a list of method
declarations. Unlike function declarations, methods don't have a leading <span
name="fun">`fun`</span> keyword. Each method is a name, parameter list, and
body. Here's an example:
用简单的英语来说，类声明就是`class`关键字，后跟类的名称，然后是一对花括号包含的主体。在这个主体中，有一个方法声明的列表。与函数声明不同的是，方法没有前导的`fun`关键字。每个方法就是一个名称、参数列表和方法主体。下面是一个例子：

<aside name="fun">

Not that I'm trying to say methods aren't fun or anything.
我并不是想说方法不好玩什么的。

</aside>

```lox
class Breakfast {
  cook() {
    print "Eggs a-fryin'!";
  }

  serve(who) {
    print "Enjoy your breakfast, " + who + ".";
  }
}
```

Like most dynamically typed languages, fields are not explicitly listed in the
class declaration. Instances are loose bags of data and you can freely add
fields to them as you see fit using normal imperative code.
像大多数动态类型的语言一样，字段没有在类的声明中明确列出。实例是松散的数据包，你可以使用正常的命令式代码自由地向其中添加字段。

Over in our AST generator, the `classDecl` grammar rule gets its own statement
<span name="class-ast">node</span>.
在AST生成器中，`classDecl`语法规则有自己的语句节点。

^code class-ast (1 before, 1 after)

<aside name="class-ast">

The generated code for the new node is in [Appendix II][appendix-class].
新节点的生成代码见[附录 II][appendix-class]。

[appendix-class]: appendix-ii.html#class-statement

</aside>

It stores the class's name and the methods inside its body. Methods are
represented by the existing Stmt.Function class that we use for function
declaration AST nodes. That gives us all the bits of state that we need for a
method: name, parameter list, and body.
它存储了类的名称和其主体内的方法。方法使用现有的表示函数声明的Stmt.Function类来表示。这就为我们提供了一个方法所需的所有状态：名称、参数列表和方法体。

A class can appear anywhere a named declaration is allowed, triggered by the
leading `class` keyword.
类可以出现在任何允许名称声明的地方，由前导的`class`关键字来触发。

^code match-class (1 before, 1 after)

That calls out to:
进一步调用：

^code parse-class-declaration

There's more meat to this than most of the other parsing methods, but it roughly
follows the grammar. We've already consumed the `class` keyword, so we look for
the expected class name next, followed by the opening curly brace. Once inside
the body, we keep parsing method declarations until we hit the closing brace.
Each method declaration is parsed by a call to `function()`, which we defined
back in the [chapter where functions were introduced][functions].
这比其它大多数解析方法有更多的内容，但它大致上遵循了语法。我们已经使用了`class`关键字，所以我们接下来会查找预期的类名，然后是左花括号。一旦进入主体，我们就继续解析方法声明，直到碰到右花括号。每个方法声明是通过调用`function()`方法来解析的，我们在介绍函数的那一章中定义了该函数。

[functions]: functions.html

Like we do in any open-ended loop in the parser, we also check for hitting the
end of the file. That won't happen in correct code since a class should have a
closing brace at the end, but it ensures the parser doesn't get stuck in an
infinite loop if the user has a syntax error and forgets to correctly end the
class body.
就像我们在解析器中的所有开放式循环中的操作一样，我们也要检查是否到达文件结尾。这在正确的代码是不会发生的，因为类的结尾应该有一个右花括号，但它可以确保在用户出现语法错误而忘记正确结束类的主体时，解析器不会陷入无限循环。

We wrap the name and list of methods into a Stmt.Class node and we're done.
Previously, we would jump straight into the interpreter, but now we need to
plumb the node through the resolver first.
<span name="resolver_zh"></span>我们将名称和方法列表封装到Stmt.Class节点中，这样就完成了。以前，我们会直接进入解释器中，但是现在我们需要先进入分析器中对节点进行分析。

<aside name="resolver_zh">

译者注：为了区分parse和resolve，这里将resolver称为分析器，用于对代码中的变量进行分析

</aside>

^code resolver-visit-class

We aren't going to worry about resolving the methods themselves yet, so for now
all we need to do is declare the class using its name. It's not common to
declare a class as a local variable, but Lox permits it, so we need to handle it
correctly.
我们还不用担心针对方法本身的分析，我们目前需要做的是使用类的名称来声明这个类。将类声明为一个局部变量并不常见，但是Lox中允许这样做，所以我们需要正确处理。

Now we interpret the class declaration.
现在我们解释一下类的声明。

^code interpreter-visit-class

This looks similar to how we execute function declarations. We declare the
class's name in the current environment. Then we turn the class *syntax node*
into a LoxClass, the *runtime* representation of a class. We circle back and
store the class object in the variable we previously declared. That two-stage
variable binding process allows references to the class inside its own methods.
这看起来类似于我们执行函数声明的方式。我们在当前环境中声明该类的名称。然后我们把类的*语法节点*转换为LoxClass，即类的*运行时*表示。我们回过头来，将类对象存储在我们之前声明的变量中。这个二阶段的变量绑定过程允许在类的方法中引用其自身。

We will refine it throughout the chapter, but the first draft of LoxClass looks
like this:
我们会在整个章节中对其进行完善，但是LoxClass的初稿看起来如下：

^code lox-class

Literally a wrapper around a name. We don't even store the methods yet. Not
super useful, but it does have a `toString()` method so we can write a trivial
script and test that class objects are actually being parsed and executed.
字面上看，就是一个对name的包装。我们甚至还没有保存类中的方法。不算很有用，但是它确实有一个`toString()`方法，所以我们可以编写一个简单的脚本，测试类对象是否真的被解析和执行。

```lox
class DevonshireCream {
  serveOn() {
    return "Scones";
  }
}

print DevonshireCream; // Prints "DevonshireCream".
```

## 创建实例

We have classes, but they don't do anything yet. Lox doesn't have "static"
methods that you can call right on the class itself, so without actual
instances, classes are useless. Thus instances are the next step.
我们有了类，但是它们还不能做任何事。Lox没有可以直接在类本身调用的“静态”方法，所以如果没有实例，类是没有用的。因此，下一步就是实例化。

While some syntax and semantics are fairly standard across OOP languages, the
way you create new instances isn't. Ruby, following Smalltalk, creates instances
by calling a method on the class object itself, a <span
name="turtles">recursively</span> graceful approach. Some, like C++ and Java,
have a `new` keyword dedicated to birthing a new object. Python has you "call"
the class itself like a function. (JavaScript, ever weird, sort of does both.)
虽然一些语法和语义在OOP语言中是相当标准的，但创建新实例的方式并不是。Ruby，继Smalltalk之后，通过调用类对象本身的一个方法来创建实例，这是一种递归的优雅方法。有些语言，像C++和Java，有一个`new`关键字专门用来创建一个新的对象。Python让你像调用函数一样“调用”类本身。(JavaScript，永远都是那么奇怪，两者兼而有之)

<aside name="turtles">

In Smalltalk, even *classes* are created by calling methods on an existing
object, usually the desired superclass. It's sort of a turtles-all-the-way-down
thing. It ultimately bottoms out on a few magical classes like Object and
Metaclass that the runtime conjures into being *ex nihilo*.
在Smalltalk中，甚至连 *类* 也是通过现有对象（通常是所需的超类）的方法来创建的。有点像是一直向下龟缩。最后，它会在一些神奇的类上触底，比如Object和Metaclass，它们是运行时 *凭空创造* 出来的。

</aside>

I took a minimal approach with Lox. We already have class objects, and we
already have function calls, so we'll use call expressions on class objects to
create new instances. It's as if a class is a factory function that generates
instances of itself. This feels elegant to me, and also spares us the need to
introduce syntax like `new`. Therefore, we can skip past the front end straight
into the runtime.
我在Lox中采用了一种最简单的方法。我们已经有了类对象，也有了函数调用，所以我们直接使用类对象的调用表达式来创建新的实例。这就好像类是一个生产自身实例的工厂函数。这让我感觉很优雅，也不需要引入`new`这样的语法。因此，我们可以跳过前端直接进入运行时。

Right now, if you try this:
现在，如果你试着运行下面的代码：

```lox
class Bagel {}
Bagel();
```

You get a runtime error. `visitCallExpr()` checks to see if the called object
implements `LoxCallable` and reports an error since LoxClass doesn't. Not *yet*,
that is.
你会得到一个运行时错误。`visitCallExpr()`方法会检查被调用的对象是否实现了`LoxCallable` 接口，因为LoxClass没有实现所以会报错。只是目前还没有。

^code lox-class-callable (2 before, 1 after)

Implementing that interface requires two methods.
实现该接口需要两个方法。

^code lox-class-call-arity

The interesting one is `call()`. When you "call" a class, it instantiates a new
LoxInstance for the called class and returns it. The `arity()` method is how the
interpreter validates that you passed the right number of arguments to a
callable. For now, we'll say you can't pass any. When we get to user-defined
constructors, we'll revisit this.
有趣的是`call()`。当你“调用”一个类时，它会为被调用的类实例化一个新的LoxInstance并返回。`arity()` 方法是解释器用于验证你是否向callable中传入了正确数量的参数。现在，我们会说你不用传任何参数。当我们讨论用户自定义的构造函数时，我们再重新考虑这个问题。

That leads us to LoxInstance, the runtime representation of an instance of a Lox
class. Again, our first implementation starts small.
这就引出了LoxInstance，它是Lox类实例的运行时表示。同样，我们的第一个实现从小处着手。

^code lox-instance

Like LoxClass, it's pretty bare bones, but we're only getting started. If you
want to give it a try, here's a script to run:
和LoxClass一样，它也是相当简陋的，但我们才刚刚开始。如果你想测试一下，可以运行下面的脚本：

```lox
class Bagel {}
var bagel = Bagel();
print bagel; // Prints "Bagel instance".
```

This program doesn't do much, but it's starting to do *something*.
这段程序没有做太多事，但是已经开始做*一些事情*了。

## 实例属性

We have instances, so we should make them useful. We're at a fork in the road.
We could add behavior first -- methods -- or we could start with state --
properties. We're going to take the latter because, as we'll see, the two get
entangled in an interesting way and it will be easier to make sense of them if
we get properties working first.
我们有了实例，所以我们应该让它们发挥作用。我们正处于一个岔路口。我们可以首先添加行为（方法），或者我们可以先从状态（属性）开始。我们将选择后者，因为我们后面将会看到，这两者以一种有趣的方式纠缠在一起，如果我们先支持属性，就会更容易理解它们。

Lox follows JavaScript and Python in how it handles state. Every instance is an
open collection of named values. Methods on the instance's class can access and
modify properties, but so can <span name="outside">outside</span> code.
Properties are accessed using a `.` syntax.
Lox遵循了JavaScript和Python处理状态的方式。每个实例都是一个开放的命名值集合。实例类中的方法可以访问和修改属性，但外部代码也可以。属性通过`.`语法进行访问。

<aside name="outside">

Allowing code outside of the class to directly modify an object's fields goes
against the object-oriented credo that a class *encapsulates* state. Some
languages take a more principled stance. In Smalltalk, fields are accessed using
simple identifiers -- essentially, variables that are only in scope inside a
class's methods. Ruby uses `@` followed by a name to access a field in an
object. That syntax is only meaningful inside a method and always accesses state
on the current object.
允许类之外的代码直接修改对象的字段，这违背了面向对象的原则，即类封装状态。有些语言采取了更有原则的立场。在SmallTalk中，字段实际上是使用简单的标识符访问的，这些标识符是类方法作用域内的变量。Ruby使用@后跟名字来访问对象中的字段。这种语法只有在方法中才有意义，并且总是访问当前对象的状态。

Lox, for better or worse, isn't quite so pious about its OOP faith.
不管怎样，Lox对OOP的信仰并不是那么虔诚。

</aside>

```lox
someObject.someProperty
```

An expression followed by `.` and an identifier reads the property with that
name from the object the expression evaluates to. That dot has the same
precedence as the parentheses in a function call expression, so we slot it into
the grammar by replacing the existing `call` rule with:
一个后面跟着`.`和一个标识符的表达式，会从表达式计算出的对象中读取该名称对应的属性。这个点符号与函数调用表达式中的括号具有相同的优先级，所以我们要将该符号加入语法时，可以替换已有的`call`规则如下：

```ebnf
call           → primary ( "(" arguments? ")" | "." IDENTIFIER )* ;
```

After a primary expression, we allow a series of any mixture of parenthesized
calls and dotted property accesses. "Property access" is a mouthful, so from
here on out, we'll call these "get expressions".
在基本表达式之后，我们允许跟一系列括号调用和点属性访问的任何混合。属性访问有点拗口，所以自此以后，我们称其为“get表达式”。

### Get表达式

The <span name="get-ast">syntax tree node</span> is:
语法树节点是：

^code get-ast (1 before, 1 after)

<aside name="get-ast">

The generated code for the new node is in [Appendix II][appendix-get].
新节点的生成代码见[附录 II][appendix-get]。

[appendix-get]: appendix-ii.html#get-expression

</aside>

Following the grammar, the new parsing code goes in our existing `call()`
method.
按照语法，在现有的`call()`方法中加入新的解析代码。

^code parse-property (3 before, 4 after)

The outer `while` loop there corresponds to the `*` in the grammar rule. We zip
along the tokens building up a chain of calls and gets as we find parentheses
and dots, like so:
外面的`while`循环对应于语法规则中的`*`。随着查找括号和点，我们会沿着标记构建一系列的call和get，就像：

<img src="image/classes/zip.png" alt="Parsing a series of '.' and '()' expressions to an AST." />

Instances of the new Expr.Get node feed into the resolver.
新的Expr.Get节点实例会被送入分析器。

^code resolver-visit-get

OK, not much to that. Since properties are looked up <span
name="dispatch">dynamically</span>, they don't get resolved. During resolution,
we recurse only into the expression to the left of the dot. The actual property
access happens in the interpreter.
好吧，没什么好说的。因为属性是动态查找的，所以不会解析它们。在解析过程中，我们只递归到点符左边的表达式中。实际的属性访问发生在解释器中。

<aside name="dispatch">

You can literally see that property dispatch in Lox is dynamic since we don't
process the property name during the static resolution pass.
从字面上可以看出，Lox 中的属性分派是动态的，因为我们在静态解析传递过程中不处理属性名称。

</aside>

^code interpreter-visit-get

First, we evaluate the expression whose property is being accessed. In Lox, only
instances of classes have properties. If the object is some other type like a
number, invoking a getter on it is a runtime error.
首先，我们对属性被访问的表达式求值。在Lox中，只有类的实例才具有属性。如果对象是其它类型（如数字），则对其执行getter是运行时错误。

If the object is a LoxInstance, then we ask it to look up the property. It must
be time to give LoxInstance some actual state. A map will do fine.
如果该对象是LoxInstance，我们就要求它去查找该属性。现在必须给LoxInstance一些实际的状态了。一个map就行了。

^code lox-instance-fields (1 before, 2 after)

Each key in the map is a property name and the corresponding value is the
property's value. To look up a property on an instance:
map中的每个键是一个属性名称，对应的值就是该属性的值。查找实例中的一个属性：

^code lox-instance-get-property

<aside name="hidden">

Doing a hash table lookup for every field access is fast enough for many
language implementations, but not ideal. High performance VMs for languages like
JavaScript use sophisticated optimizations like "[hidden classes][]" to avoid
that overhead.
对于许多语言的实现来说，每次字段访问都进行哈希表查找已经足够快了，但这并不理想。JavaScript 等语言的高性能虚拟机使用 "[隐藏类][hidden classes]" 等复杂的优化方法来避免这种开销。

Paradoxically, many of the optimizations invented to make dynamic languages fast
rest on the observation that -- even in those languages -- most code is fairly
static in terms of the types of objects it works with and their fields.
自相矛盾的是，许多为使动态语言快速而发明的优化方法都是基于这样一种观点，即: 即使在这些语言中，大多数代码在对象类型及其字段方面都是相当静态的。

[hidden classes]: http://richardartoul.github.io/jekyll/update/2015/04/26/hidden-classes.html

</aside>

An interesting edge case we need to handle is what happens if the instance
doesn't *have* a property with the given name. We could silently return some
dummy value like `nil`, but my experience with languages like JavaScript is that
this behavior masks bugs more often than it does anything useful. Instead, we'll
make it a runtime error.
我们需要处理的一个有趣的边缘情况是，如果这个实例中*不包含*给定名称的属性，会发生什么。我们可以悄悄返回一些假值，如`nil`，但是根据我对JavaScript等语言的经验，这种行为只是掩盖了错误，而没有做任何有用的事。相反，我们将它作为一个运行时错误。

So the first thing we do is see if the instance actually has a field with the
given name. Only then do we return it. Otherwise, we raise an error.
因此，我们首先要做的就是看看这个实例中是否真的包含给定名称的字段。只有这样，我们才会返回其值。其它情况下，我们会引发一个错误。

Note how I switched from talking about "properties" to "fields". There is a
subtle difference between the two. Fields are named bits of state stored
directly in an instance. Properties are the named, uh, *things*, that a get
expression may return. Every field is a property, but as we'll see <span
name="foreshadowing">later</span>, not every property is a field.
注意我是如何从讨论“属性”转换到讨论“字段”的。这两者之间有一个微妙的区别。字段是直接保存在实例中的命名状态。属性是get表达式可能返回的已命名的*东西*。每个字段都是一个属性，但是正如我们稍后将看到的，并非每个属性都是一个字段。

<aside name="foreshadowing">

Ooh, foreshadowing. Spooky!
哦，看吧。真诡异!

</aside>

In theory, we can now read properties on objects. But since there's no way to
actually stuff any state into an instance, there are no fields to access. Before
we can test out reading, we must support writing.
理论上，我们现在可以读取对象的属性。但是由于没有办法将任何状态真正填充到实例中，所以也没有字段可以访问。在我们测试读取之前，我们需要先支持写入。

### Set表达式

Setters use the same syntax as getters, except they appear on the left side of
an assignment.
setter和getter使用相同的语法，区别只是它们出现在赋值表达式的左侧。

```lox
someObject.someProperty = value;
```

In grammar land, we extend the rule for assignment to allow dotted identifiers
on the left-hand side.
在语言方面，我们扩展了赋值规则，允许在左侧使用点标识符。

```ebnf
assignment     → ( call "." )? IDENTIFIER "=" assignment
               | logic_or ;
```

Unlike getters, setters don't chain. However, the reference to `call` allows any
high-precedence expression before the last dot, including any number of
*getters*, as in:
与getter不同，setter不使用链。但是，对`call` 规则的引用允许在最后的点符号之前出现任何高优先级的表达式，包括任何数量的*getters*，如：

<img src="image/classes/setter.png" alt="breakfast.omelette.filling.meat = ham" />

Note here that only the *last* part, the `.meat` is the *setter*. The
`.omelette` and `.filling` parts are both *get* expressions.
注意，这里只有最后一部分`.meat`是*setter*。`.omelette`和`.filling`部分都是*get*表达式。

Just as we have two separate AST nodes for variable access and variable
assignment, we need a <span name="set-ast">second setter node</span> to
complement our getter node.
就像我们有两个独立的AST节点用于变量访问和变量赋值一样，我们也需要一个setter节点来补充getter节点。

^code set-ast (1 before, 1 after)

<aside name="set-ast">

The generated code for the new node is in [Appendix II][appendix-set].
新节点的生成代码见[附录 II][appendix-set]。

[appendix-set]: appendix-ii.html#set-expression

</aside>

In case you don't remember, the way we handle assignment in the parser is a
little funny. We can't easily tell that a series of tokens is the left-hand side
of an assignment until we reach the `=`. Now that our assignment grammar rule
has `call` on the left side, which can expand to arbitrarily large expressions,
that final `=` may be many tokens away from the point where we need to know
we're parsing an assignment.
也许你不记得了，我们在解析器中处理赋值的方法有点奇怪。在遇到`=`之前，我们无法轻易判断一系列标记是否是一个赋值表达式的左侧部分。现在我们的赋值语法规则在左侧添加了`call`，它可以扩展为任意大的表达式，最后的`=`可能与我们需要知道是否正在解析赋值表达式的地方隔着很多标记。

Instead, the trick we do is parse the left-hand side as a normal expression.
Then, when we stumble onto the equal sign after it, we take the expression we
already parsed and transform it into the correct syntax tree node for the
assignment.
相对地，我们的技巧就是把左边的表达式作为一个正常表达式来解析。然后，当我们在后面发现等号时，我们就把已经解析的表达式转换为正确的赋值语法树节点。

We add another clause to that transformation to handle turning an Expr.Get
expression on the left into the corresponding Expr.Set.
我们在该转换中添加另一个子句，将左边的Expr.Get表达式转化为相应的Expr.Set表达式。

^code assign-set (1 before, 1 after)

That's parsing our syntax. We push that node through into the resolver.
这就是语法解析。我们将该节点推入分析器中。

^code resolver-visit-set

Again, like Expr.Get, the property itself is dynamically evaluated, so there's
nothing to resolve there. All we need to do is recurse into the two
subexpressions of Expr.Set, the object whose property is being set, and the
value it's being set to.
同样，像Expr.Get一样，属性本身是动态计算的，所以没有什么需要分析的。我们只需要递归到Expr.Set的两个子表达式中，即被设置属性的对象和它被设置的值。

That leads us to the interpreter.
这又会把我们引向解释器。

^code interpreter-visit-set

We evaluate the object whose property is being set and check to see if it's a
LoxInstance. If not, that's a runtime error. Otherwise, we evaluate the value
being set and store it on the instance. That relies on a new method in
LoxInstance.
我们先计算出被设置属性的对象，然后检查它是否是一个LoxInstance。如果不是，这就是一个运行时错误。否则，我们计算设置的值，并将其保存到该实例中。这一步依赖于LoxInstance中的一个新方法。

<aside name="order">

This is another semantic edge case. There are three distinct operations:

1. Evaluate the object.

2. Raise a runtime error if it's not an instance of a class.

3. Evaluate the value.

The order that those are performed in could be user visible, which means we need
to carefully specify it and ensure our implementations do these in the same
order.

这是另一种语义边缘情况。有三种不同的操作：

1. 评估对象。

2. 如果不是类的实例，则引发运行时错误。

3. 评估值。

执行这些操作的顺序可能是用户可见的，这意味着我们需要仔细敲定，并确保我们的实现以相同的顺序执行这些操作。

</aside>

^code lox-instance-set-property

No real magic here. We stuff the values straight into the Java map where fields
live. Since Lox allows freely creating new fields on instances, there's no need
to see if the key is already present.
这里没什么复杂的。我们把这些值之间塞入字段所在的Java map中。由于Lox允许在实例上自由创建新字段，所以不需要检查键是否已经存在。

## 类中的方法

You can create instances of classes and stuff data into them, but the class
itself doesn't really *do* anything. Instances are just maps and all instances
are more or less the same. To make them feel like instances *of classes*, we
need behavior -- methods.
你可以创建类的实例并将数据填入其中，但是类本身实际上并不能做任何事。实例只是一个map，而且所有的实例都是大同小异的。为了让它们更像是*类*的实例，我们需要行为 -- 方法。

Our helpful parser already parses method declarations, so we're good there. We
also don't need to add any new parser support for method *calls*. We already
have `.` (getters) and `()` (function calls). A "method call" simply chains
those together.
我们的解析器已经解析了方法声明，所以我们在这部分做的不错。我们也不需要为方法*调用*添加任何新的解析器支持。我们已经有了`.`(getter)和`()`(函数调用)。“方法调用”只是简单地将这些串在一起。

<img src="image/classes/method.png" alt="The syntax tree for 'object.method(argument)" />

That raises an interesting question. What happens when those two expressions are
pulled apart? Assuming that `method` in this example is a method on the class of
`object` and not a field on the instance, what should the following piece of
code do?
这引出了一个有趣的问题。当这两个表达式分开时会发生什么？假设这个例子中的方法`method`是`object`的类中的一个方法，而不是实例中的 一个字段，下面的代码应该做什么？

```lox
var m = object.method;
m(argument);
```

This program "looks up" the method and stores the result -- whatever that is --
in a variable and then calls that object later. Is this allowed? Can you treat a
method like it's a function on the instance?
这个程序会“查找”该方法，并将结果（不管是什么）存储到一个变量中，稍后会调用该对象。允许这样吗？你能将方法作为实例中的一个函数来对待吗？

What about the other direction?
另一个方向呢？

```lox
class Box {}

fun notMethod(argument) {
  print "called function with " + argument;
}

var box = Box();
box.function = notMethod;
box.function("argument");
```

This program creates an instance and then stores a function in a field on it.
Then it calls that function using the same syntax as a method call. Does that
work?
这个程序创建了一个实例，然后在它的一个字段中存储了一个函数。然后使用与方法调用相同的语法来调用该函数。这样做有用吗？

Different languages have different answers to these questions. One could write a
treatise on it. For Lox, we'll say the answer to both of these is yes, it does
work. We have a couple of reasons to justify that. For the second example --
calling a function stored in a field -- we want to support that because
first-class functions are useful and storing them in fields is a perfectly
normal thing to do.
不同的语言对这些问题有不同的答案。人们可以就此写一篇论文。对于Lox来说，这两个问题的答案都是肯定的，它确实有效。我们有几个理由来证明这一点。对于第二个例子 -- 调用存储在字段中的函数 -- 我们想要支持它，是因为头等函数是有用的，而且将它们存储在字段中是一件很正常的事情。

The first example is more obscure. One motivation is that users generally expect
to be able to hoist a subexpression out into a local variable without changing
the meaning of the program. You can take this:
第一个例子就比较晦涩了。一个场景是，用户通常希望能够在不改变程序含义的情况下，将子表达式赋值到一个局部变量中。你可以这样做：

```lox
breakfast(omelette.filledWith(cheese), sausage);
```

And turn it into this:
并将其变成这样：

```lox
var eggs = omelette.filledWith(cheese);
breakfast(eggs, sausage);
```

And it does the same thing. Likewise, since the `.` and the `()` in a method
call *are* two separate expressions, it seems you should be able to hoist the
*lookup* part into a variable and then call it <span
name="callback">later</span>. We need to think carefully about what the *thing*
you get when you look up a method is, and how it behaves, even in weird cases
like:
它做的是同样的事情。同样，由于方法调用中的`.`和`()`是两个独立的表达式，你似乎应该把查询部分提取到一个变量中，然后再调用它。我们需要仔细思考，当你查找一个方法时你得到的东西是什么，它如何作用，甚至是在一些奇怪的情况下，比如：

<aside name="callback">

A motivating use for this is callbacks. Often, you want to pass a callback whose
body simply invokes a method on some object. Being able to look up the method and
pass it directly saves you the chore of manually declaring a function to wrap
it. Compare this:
它的经典用途之一就是回调。通常，你想要传递一个回调函数，其主体只是调用某个对象上的一个方法。既然能够找到该方法并直接传递它，就省去了手动声明一个函数对其进行包装的麻烦工作。比较一下下面两段代码：

```lox
fun callback(a, b, c) {
  object.method(a, b, c);
}

takeCallback(callback);
```

With this:
以及:

```lox
takeCallback(object.method);
```

</aside>

```lox
class Person {
  sayName() {
    print this.name;
  }
}

var jane = Person();
jane.name = "Jane";

var method = jane.sayName;
method(); // ?
```

If you grab a handle to a method on some instance and call it later, does it
"remember" the instance it was pulled off from? Does `this` inside the method
still refer to that original object?
如果你在某个实例上获取了一个方法的句柄，并在稍后再调用它，它是否能“记住”它是从哪个实例中提取出来的？方法内部的`this`是否仍然指向原始的那个对象？

Here's a more pathological example to bend your brain:
下面有一个更变态的例子，可以摧毁你的大脑：

```lox
class Person {
  sayName() {
    print this.name;
  }
}

var jane = Person();
jane.name = "Jane";

var bill = Person();
bill.name = "Bill";

bill.sayName = jane.sayName;
bill.sayName(); // ?
```

Does that last line print "Bill" because that's the instance that we *called*
the method through, or "Jane" because it's the instance where we first grabbed
the method?
最后一行会因为*调用*方法的实体是bill而打印“Bill”，还是因为我们第一次获取方法的实例是jane而打印“Jane”。

Equivalent code in Lua and JavaScript would print "Bill". Those languages don't
really have a notion of "methods". Everything is sort of functions-in-fields, so
it's not clear that `jane` "owns" `sayName` any more than `bill` does.
在Lua和JavaScript中，同样的代码会打印 "Bill"。这些语言并没有真正的“方法”的概念。所有东西都类似于字段中的函数，所以并不清楚`jane` 是否更应该比`bill`“拥有”`sayName`。

Lox, though, has real class syntax so we do know which callable things are
methods and which are functions. Thus, like Python, C#, and others, we will have
methods "bind" `this` to the original instance when the method is first grabbed.
Python calls <span name="bound">these</span> **bound methods**.
不过，Lox有真正的类语法，所以我们确实知道哪些可调用的东西是方法，哪些是函数。因此，像Python、C#和其他语言一样，当方法第一次被获取时，我们会让方法与原始实例`this`进行 "绑定"。Python将这些绑定的方法称为**bound methods**（绑定方法）。

<aside name="bound">

I know, imaginative name, right?
我知道，富有想象力的名字，对吧？

</aside>

In practice, that's usually what you want. If you take a reference to a method
on some object so you can use it as a callback later, you want to remember the
instance it belonged to, even if that callback happens to be stored in a field
on some other object.
在实践中，这通常也是你想要的。如果你获取到了某个对象中一个方法的引用，这样你以后就可以把它作为一个回调函数使用，你想要记住它所属的实例，即使这个回调被存储在其它对象的字段中。

OK, that's a lot of semantics to load into your head. Forget about the edge
cases for a bit. We'll get back to those. For now, let's get basic method calls
working. We're already parsing the method declarations inside the class body, so
the next step is to resolve them.
好吧，这里有很多语义需要装到你的脑子里。暂时先不考虑那些边缘情况了，我们以后再讲。现在，让我们先把基本的方法调用做好。我们已经解析了类主体内的方法声明，所以下一步就是对其分析。

^code resolve-methods (1 before, 1 after)

<aside name="local">

Storing the function type in a local variable is pointless right now, but we'll
expand this code before too long and it will make more sense.
现在将函数类型保存到一个局部变量中是没有意义的，但我们稍后会扩展这段代码，到时它就有意义了。

</aside>

We iterate through the methods in the class body and call the
`resolveFunction()` method we wrote for handling function declarations already.
The only difference is that we pass in a new FunctionType enum value.
我们遍历类主体中的方法，并调用我们已经写好的用来处理函数声明的`resolveFunction()`方法。唯一的区别在于，我们传入了一个新的FunctionType枚举值。

^code function-type-method (1 before, 1 after)

That's going to be important when we resolve `this` expressions. For now, don't
worry about it. The interesting stuff is in the interpreter.
这一点在我们分析`this`表达式时很重要。现在还不用担心这个问题。有趣的部分在解释器中。

^code interpret-methods (1 before, 1 after)

When we interpret a class declaration statement, we turn the syntactic
representation of the class -- its AST node -- into its runtime representation.
Now, we need to do that for the methods contained in the class as well. Each
method declaration blossoms into a LoxFunction object.
当我们解释一个类声明语句时，我们把类的语法表示（其AST节点）变成它的运行时表示。现在，我们也需要对类中包含的方法进行这样的操作。每个方法声明都会变成一个LoxFunction对象。

We take all of those and wrap them up into a map, keyed by the method names.
That gets stored in LoxClass.
我们把所有这些都打包到一个map中，以方法名称作为键。这些数据存储在LoxClass中。

^code lox-class-methods (1 before, 3 after)

Where an instance stores state, the class stores behavior. LoxInstance has its
map of fields, and LoxClass gets a map of methods. Even though methods are
owned by the class, they are still accessed through instances of that class.
实例存储状态，类存储行为。LoxInstance包含字段的map，而LoxClass包含方法的map。虽然方法是归类所有，但仍然是通过类的实例来访问。

^code lox-instance-get-method (5 before, 2 after)

When looking up a property on an instance, if we don't <span
name="shadow">find</span> a matching field, we look for a method with that name
on the instance's class. If found, we return that. This is where the distinction
between "field" and "property" becomes meaningful. When accessing a property,
you might get a field -- a bit of state stored on the instance -- or you could
hit a method defined on the instance's class.
在实例上查找属性时，如果我们没有找到匹配的字段，我们就在实例的类中查找是否包含该名称的方法。如果找到，我们就返回该方法。这就是“字段”和“属性”之间的区别变得有意义的地方。当访问一个属性时，你可能会得到一个字段（存储在实例上的状态值），或者你会得到一个实例类中定义的方法。

The method is looked up using this:
方法是通过下面的代码进行查找的：

<aside name="shadow">

Looking for a field first implies that fields shadow methods, a subtle but
important semantic point.
首先寻找字段，意味着字段会遮蔽方法，这是一个微妙但重要的语义点。

</aside>

^code lox-class-find-method

You can probably guess this method is going to get more interesting later. For
now, a simple map lookup on the class's method table is enough to get us
started. Give it a try:
你大概能猜到这个方法后面会变得更有趣。但是现在，在类的方法表中进行简单的映射查询就足够了。试一下：

<span name="crunch"></span>

```lox
class Bacon {
  eat() {
    print "Crunch crunch crunch!";
  }
}

Bacon().eat(); // Prints "Crunch crunch crunch!".
```

<aside name="crunch">

Apologies if you prefer chewy bacon over crunchy. Feel free to adjust the script
to your taste.
如果您喜欢有嚼劲的培根而不是松脆的培根，请见谅。请根据自己的口味调整代码。

</aside>

## This

We can define both behavior and state on objects, but they aren't tied together
yet. Inside a method, we have no way to access the fields of the "current"
object -- the instance that the method was called on -- nor can we call other
methods on that same object.
我们可以在对象上定义行为和状态，但是它们并没有被绑定在一起。在一个方法中，我们没有办法访问“当前”对象（调用该方法的实例）的字段，也不能调用同一个对象的其它方法。

To get at that instance, it needs a <span name="i">name</span>. Smalltalk,
Ruby, and Swift use "self". Simula, C++, Java, and others use "this". Python
uses "self" by convention, but you can technically call it whatever you like.
为了获得这个实例，它需要一个名称。Smalltalk、Ruby和Swift使用 "self"。Simula、C++、Java等使用 "this"。Python按惯例使用 "self"，但从技术上讲，你可以随便叫它什么。

<aside name="i">

"I" would have been a great choice, but using "i" for loop variables predates
OOP and goes all the way back to Fortran. We are victims of the incidental
choices of our forebears.
使用 "I" 本来是个不错的选择，但使用 "i" 来表示循环变量早在 OOP 之前就有了，而且可以追溯到 Fortran。我们是前人偶然选择的牺牲品。

</aside>

For Lox, since we generally hew to Java-ish style, we'll go with "this". Inside
a method body, a `this` expression evaluates to the instance that the method was
called on. Or, more specifically, since methods are accessed and then invoked as
two steps, it will refer to the object that the method was *accessed* from.
对于Lox来说，因为我们通常遵循Java风格，我们会使用“this”。在方法体中，`this`表达式计算结果为调用该方法的实例。或者，更确切地说，由于方法是分为两个步骤进行访问和调用的，因此它会引用调用方法的对象。

That makes our job harder. Peep at:
这使得我们的工作更加困难。请看：

```lox
class Egotist {
  speak() {
    print this;
  }
}

var method = Egotist().speak;
method();
```

On the second-to-last line, we grab a reference to the `speak()` method off an
instance of the class. That returns a function, and that function needs to
remember the instance it was pulled off of so that *later*, on the last line, it
can still find it when the function is called.
在倒数第二行，我们从该类的一个实例中获取到了指向`speak()` 的引用。这个操作会返回一个函数，并且该函数需要记住它来自哪个实例，这样稍后在最后一行，当函数被调用时，它仍然可用找到对应实例。

We need to take `this` at the point that the method is accessed and attach it to
the function somehow so that it stays around as long as we need it to. Hmm... a
way to store some extra data that hangs around a function, eh? That sounds an
awful lot like a *closure*, doesn't it?
我们需要在方法被访问时获取到`this`，并将其附到函数上，这样当我们需要的时候它就一直存在。嗯…一种存储函数周围的额外数据的方法，嗯？听起来很像一个闭包，不是吗？

If we defined `this` as a sort of hidden variable in an environment that
surrounds the function returned when looking up a method, then uses of `this` in
the body would be able to find it later. LoxFunction already has the ability to
hold on to a surrounding environment, so we have the machinery we need.
如果我们把`this`定义为在查找方法时返回的函数外围环境中的一个隐藏变量，那么稍后在方法主体中使用`this`时就可以找到它了。LoxFunction已经具备了保持外围环境的能力，所以我们已经有了需要的机制。

Let's walk through an example to see how it works:
我们通过一个例子来看看它是如何工作的：

```lox
class Cake {
  taste() {
    var adjective = "delicious";
    print "The " + this.flavor + " cake is " + adjective + "!";
  }
}

var cake = Cake();
cake.flavor = "German chocolate";
cake.taste(); // Prints "The German chocolate cake is delicious!".
```

When we first evaluate the class definition, we create a LoxFunction for
`taste()`. Its closure is the environment surrounding the class, in this case
the global one. So the LoxFunction we store in the class's method map looks
like so:
当我们第一次执行类定义时，我们为`taste()`创建了一个LoxFunction。它的闭包是类外围的环境，在这个例子中就是全局环境。所以我们在类的方法map中保存的LoxFunction看起来像是这样的：

<img src="image/classes/closure.png" alt="The initial closure for the method." />

When we evaluate the `cake.taste` get expression, we create a new environment
that binds `this` to the object the method is accessed from (here, `cake`). Then
we make a *new* LoxFunction with the same code as the original one but using
that new environment as its closure.
当我们执行`cake.taste`这个get表达式时，我们会创建一个新的环境，其中将`this`绑定到了访问该方法的对象（这里是`cake`）。然后我们创建一个*新*的LoxFunction，它的代码与原始的代码相同，但是使用新环境作为其闭包。

<img src="image/classes/bound-method.png" alt="The new closure that binds 'this'." />

This is the LoxFunction that gets returned when evaluating the get expression
for the method name. When that function is later called by a `()` expression,
we create an environment for the method body as usual.
这个是在执行方法名的get表达式时返回的LoxFunction。当这个函数稍后被一个`()`表达式调用时，我们像往常一样为方法主体创建一个环境。

<img src="image/classes/call.png" alt="Calling the bound method and creating a new environment for the method body." />

The parent of the body environment is the environment we created earlier to bind
`this` to the current object. Thus any use of `this` inside the body
successfully resolves to that instance.
主体环境的父环境，也就是我们先前创建并在其中将`this`绑定到当前对象的那个环境。因此，在函数主体内使用`this`都可以成功解析到那个实例。

Reusing our environment code for implementing `this` also takes care of
interesting cases where methods and functions interact, like:
重用环境代码来实现`this`时，也需要注意方法和函数交互的情况，比如：

```lox
class Thing {
  getCallback() {
    fun localFunction() {
      print this;
    }

    return localFunction;
  }
}

var callback = Thing().getCallback();
callback();
```

In, say, JavaScript, it's common to return a callback from inside a method. That
callback may want to hang on to and retain access to the original object -- the
`this` value -- that the method was associated with. Our existing support for
closures and environment chains should do all this correctly.
例如，在JavaScript中，在一个方法中返回一个回调函数是很常见的。这个回调函数可能希望保留对方法所关联的原对象（`this`值）的访问。我们现有的对闭包和环境链的支持应该可以正确地做到这一点。

Let's code it up. The first step is adding <span name="this-ast">new
syntax</span> for `this`.
让我们把它写出来。第一步是为`this`添加新的语法。

^code this-ast (1 before, 1 after)

<aside name="this-ast">

The generated code for the new node is in [Appendix II][appendix-this].
新节点的生成代码见[附录 II][appendix-this]。

[appendix-this]: appendix-ii.html#this-expression

</aside>

Parsing is simple since it's a single token which our lexer already
recognizes as a reserved word.
解析很简单，因为它是已经被词法解析器当作关键字识别出来的单个词法标记。

^code parse-this (2 before, 2 after)

You can start to see how `this` works like a variable when we get to the
resolver.
当进入分析器后，就可以看到 `this` 是如何像变量一样工作的。

^code resolver-visit-this

We resolve it exactly like any other local variable using "this" as the name for
the "variable". Of course, that's not going to work right now, because "this"
*isn't* declared in any scope. Let's fix that over in `visitClassStmt()`.
我们使用`this`作为“变量”的名称，并像其它局部变量一样对其分析。当然，现在这是行不通的，因为“this”没有在任何作用域进行声明。我们在`visitClassStmt()`方法中解决这个问题。

^code resolver-begin-this-scope (2 before, 1 after)

Before we step in and start resolving the method bodies, we push a new scope and
define "this" in it as if it were a variable. Then, when we're done, we discard
that surrounding scope.
在我们开始分析方法体之前，我们推入一个新的作用域，并在其中像定义变量一样定义“this”。然后，当我们完成后，会丢弃这个外围作用域。

^code resolver-end-this-scope (2 before, 1 after)

Now, whenever a `this` expression is encountered (at least inside a method) it
will resolve to a "local variable" defined in an implicit scope just outside of
the block for the method body.
现在，只要遇到`this`表达式（至少是在方法内部），它就会解析为一个“局部变量”，该变量定义在方法体块之外的隐含作用域中。

The resolver has a new *scope* for `this`, so the interpreter needs to create a
corresponding *environment* for it. Remember, we always have to keep the
resolver's scope chains and the interpreter's linked environments in sync with
each other. At runtime, we create the environment after we find the method on
the instance. We replace the previous line of code that simply returned the
method's LoxFunction with this:
分析器对`this`有一个新的*作用域*，所以解释器需要为它创建一个对应的*环境*。记住，我们必须始终保持分析器的作用域链与解释器的链式环境保持同步。在运行时，我们在找到实例上的方法后创建环境。我们把之前那行直接返回方法对应LoxFunction的代码替换如下：

^code lox-instance-bind-method (1 before, 3 after)

Note the new call to `bind()`. That looks like so:
注意这里对`bind()`的新调用。该方法看起来是这样的：

^code bind-instance

There isn't much to it. We create a new environment nestled inside the method's
original closure. Sort of a closure-within-a-closure. When the method is called,
that will become the parent of the method body's environment.
这没什么好说的。我们基于方法的原始闭包创建了一个新的环境。就像是闭包内的闭包。当方法被调用时，它将变成方法体对应环境的父环境。

We declare "this" as a variable in that environment and bind it to the given
instance, the instance that the method is being accessed from. *Et voilà*, the
returned LoxFunction now carries around its own little persistent world where
"this" is bound to the object.
我们将`this`声明为该环境中的一个变量，并将其绑定到给定的实例（即方法被访问时的实例）上。就是这样，现在返回的LoxFunction带着它自己的小持久化世界，其中的“this”被绑定到对象上。

The remaining task is interpreting those `this` expressions. Similar to the
resolver, it is the same as interpreting a variable expression.
剩下的任务就是解释那些`this`表达式。与分析器类似，与解释变量表达式是一样的。

^code interpreter-visit-this

Go ahead and give it a try using that cake example from earlier. With less than
twenty lines of code, our interpreter handles `this` inside methods even in all
of the weird ways it can interact with nested classes, functions inside methods,
handles to methods, etc.
来吧，用前面那个蛋糕的例子试一试。通过添加不到20行代码，我们的解释器就能处理方法内部的`this`，甚至能以各种奇怪的方式与嵌套类、方法内部的函数、方法句柄等进行交互。

### Invalid uses of this  this的无效使用

Wait a minute. What happens if you try to use `this` *outside* of a method? What
about:
等一下，如果你尝试在方法之外使用`this`会怎么样？比如：

```lox
print this;
```

Or:

```lox
fun notAMethod() {
  print this;
}
```

There is no instance for `this` to point to if you're not in a method. We could
give it some default value like `nil` or make it a runtime error, but the user
has clearly made a mistake. The sooner they find and fix that mistake, the
happier they'll be.
如果你不在一个方法中，就没有可供`this`指向的实例。我们可以给它一些默认值如`nil`或者抛出一个运行时错误，但是用户显然犯了一个错误。他们越早发现并纠正这个错误，就会越高兴。

Our resolution pass is a fine place to detect this error statically. It already
detects `return` statements outside of functions. We'll do something similar for
`this`. In the vein of our existing FunctionType enum, we define a new ClassType
one.
我们的分析过程是一个静态检测这个错误的好地方。它已经检测了函数之外的`return`语句。我们可以针对`this`做一些类似的事情。在我们现有的FunctionType枚举的基础上，我们定义一个新的ClassType枚举。

^code class-type (1 before, 1 after)

Yes, it could be a Boolean. When we get to inheritance, it will get a third
value, hence the enum right now. We also add a corresponding field,
`currentClass`. Its value tells us if we are currently inside a class
declaration while traversing the syntax tree. It starts out `NONE` which means
we aren't in one.
是的，它可以是一个布尔值。当我们谈到继承时，它会扩展第三个值，因此使用了枚举。我们还添加了一个相应的字段`currentClass`。它的值告诉我们，在遍历语法树时，我们目前是否在一个类声明中。它一开始是`NONE`，意味着我们不在类中。

When we begin to resolve a class declaration, we change that.
当我们开始分析一个类声明时，我们会改变它。

^code set-current-class (1 before, 1 after)

As with `currentFunction`, we store the previous value of the field in a local
variable. This lets us piggyback onto the JVM to keep a stack of `currentClass`
values. That way we don't lose track of the previous value if one class nests
inside another.
与`currentFunction`一样，我们将字段的前一个值存储在一个局部变量中。这样我们可以在JVM中保持一个`currentClass`的栈。如果一个类嵌套在另一个类中，我们就不会丢失对前一个值的跟踪。

Once the methods have been resolved, we "pop" that stack by restoring the old
value.
一旦这么方法完成了分析，我们通过恢复旧值来“弹出”堆栈。

^code restore-current-class (2 before, 1 after)

When we resolve a `this` expression, the `currentClass` field gives us the bit
of data we need to report an error if the expression doesn't occur nestled
inside a method body.
当我们解析`this`表达式时，如果表达式没有出现在一个方法体内，`currentClass`就为我们提供了报告错误所需的数据。

^code this-outside-of-class (1 before, 1 after)

That should help users use `this` correctly, and it saves us from having to
handle misuse at runtime in the interpreter.
这应该能帮助用户正确地使用`this`，并且它使我们不必在解释器运行时中处理这个误用问题。

## 构造函数和初始化

We can do almost everything with classes now, and as we near the end of the
chapter we find ourselves strangely focused on a beginning. Methods and fields
let us encapsulate state and behavior together so that an object always *stays*
in a valid configuration. But how do we ensure a brand new object *starts* in a
good state?
我们现在几乎可以用类来做任何事情，而当我们接近本章结尾时，却发现自己奇怪地专注于开头。方法和字段让我们把状态和行为封装在一起，这样一个对象就能始终保持在有效的配置状态。但我们如何确保一个全新的对象是以良好的状态开始的？

For that, we need constructors. I find them one of the trickiest parts of a
language to design, and if you peer closely at most other languages, you'll see
<span name="cracks">cracks</span> around object construction where the seams of
the design don't quite fit together perfectly. Maybe there's something
intrinsically messy about the moment of birth.
为此，我们需要构造函数。我发现它们是语言设计中最棘手的部分之一，如果你仔细观察大多数其它语言，就会发现围绕着对象构造的缺陷，设计的接缝并不完全吻合。也许在一开始就存在本质上的混乱。

<aside name="cracks">

A few examples: In Java, even though final fields must be initialized, it is
still possible to read one *before* it has been. Exceptions -- a huge, complex
feature -- were added to C++ mainly as a way to emit errors from constructors.
举几个例子：在Java中，尽管final字段必须被初始化，但仍有可能在被初始化之前被读取。异常（一个庞大而复杂的特性）被添加到C++中主要是作为一种从构造函数发出错误的方式。

</aside>

"Constructing" an object is actually a pair of operations:
“构造”一个对象实际上是一对操作：

1.  The runtime <span name="allocate">*allocates*</span> the memory required for
    a fresh instance. In most languages, this operation is at a fundamental
    level beneath what user code is able to access.
    运行时为一个新的实例*分配*所需的内存。在多数语言中，这个操作是在用户代码可以访问的层面之下的基础层完成的。

    <aside name="allocate">

    C++'s "[placement new][]" is a rare example where the bowels of allocation
    are laid bare for the programmer to prod.
    C++中的 "[placement new][]" 是一个罕见的例子，在这种情况下，分配的内存被暴露出来供程序员使用。

    </aside>

2.  Then, a user-provided chunk of code is called which *initializes* the
    unformed object.
    然后，用户提供的一大块代码被调用，以初始化未成形的对象。

[placement new]: https://en.wikipedia.org/wiki/Placement_syntax

The latter is what we tend to think of when we hear "constructor", but the
language itself has usually done some groundwork for us before we get to that
point. In fact, our Lox interpreter already has that covered when it creates a
new LoxInstance object.
当我们听到“构造函数”时，我们往往会想到后者，但语言本身在此之前通常已经为我们做了一些基础工作。事实上，我们的Lox解释器在创建一个新的LoxInstance对象时已经涵盖了这一点。

We'll do the remaining part -- user-defined initialization -- now. Languages
have a variety of notations for the chunk of code that sets up a new object for
a class. C++, Java, and C# use a method whose name matches the class name. Ruby
and Python call it `init()`. The latter is nice and short, so we'll do that.
我们现在要做的是剩下的部分 -- 用户自定义的初始化。对于为类建立新对象的这块代码，不同的语言有不同的说法。C++、Java和C#使用一个名字与类名相匹配的方法。Ruby 和 Python 称之为 `init()`。后者又好又简短，所以我们采用它。

In LoxClass's implementation of LoxCallable, we add a few more lines.
在LoxClass的LoxCallable实现中，我们再增加几行。

^code lox-class-call-initializer (2 before, 1 after)

When a class is called, after the LoxInstance is created, we look for an "init"
method. If we find one, we immediately bind and invoke it just like a normal
method call. The argument list is forwarded along.
当一个类被调用时，在LoxInstance被创建后，我们会寻找一个 "init" 方法。如果我们找到了，我们就会立即绑定并调用它，就像普通的方法调用一样。参数列表直接透传。

That argument list means we also need to tweak how a class declares its arity.
这个参数列表意味着我们也需要调整类声明其元数的方式。

^code lox-initializer-arity (1 before, 1 after)

If there is an initializer, that method's arity determines how many arguments
you must pass when you call the class itself. We don't *require* a class to
define an initializer, though, as a convenience. If you don't have an
initializer, the arity is still zero.
如果有初始化方法，该方法的元数就决定了在调用类本身的时候需要传入多少个参数。但是，为了方便起见，我们并不要求类定义初始化方法。如果你没有初始化方法，元数仍然是0。

That's basically it. Since we bind the `init()` method before we call it, it has
access to `this` inside its body. That, along with the arguments passed to the
class, are all you need to be able to set up the new instance however you
desire.
基本上就是这样了。因为我们在调用`init()`方法之前已经将其绑定，所以它可以在方法体内访问`this`。这样，连同传递给类的参数，你就可以按照自己的意愿设置新实例了。

### 直接执行init()

As usual, exploring this new semantic territory rustles up a few weird
creatures. Consider:
像往常一样，探索这一新的语义领域会催生出一些奇怪的事物。考虑一下：

```lox
class Foo {
  init() {
    print this;
  }
}

var foo = Foo();
print foo.init();
```

Can you "re-initialize" an object by directly calling its `init()` method? If
you do, what does it return? A <span name="compromise">reasonable</span> answer
would be `nil` since that's what it appears the body returns.
你能否通过直接调用对象的`init()`方法对其进行“重新初始化”？如果可以，它的返回值是什么？一个合理的答案应该是`nil`，因为这是方法主体返回的内容。

However -- and I generally dislike compromising to satisfy the
implementation -- it will make clox's implementation of constructors much
easier if we say that `init()` methods always return `this`, even when
directly called. In order to keep jlox compatible with that, we add a little
special case code in LoxFunction.
然而，我通常不喜欢为满足实现而妥协，如果我们让`init()`方法总是返回`this`（即使是被直接调用时），它会使clox中的构造函数实现更加简单。为了保持jlox与之兼容，我们在LoxFunction中添加了一些针对特殊情况的代码。

<aside name="compromise">

Maybe "dislike" is too strong a claim. It's reasonable to have the constraints
and resources of your implementation affect the design of the language. There
are only so many hours in the day, and if a cut corner here or there lets you get
more features to users in less time, it may very well be a net win for their
happiness and productivity. The trick is figuring out *which* corners to cut
that won't cause your users and future self to curse your shortsightedness.
也许“不喜欢”这个说法太过激了。让语言实现的约束和资源影响语言的设计是合理的。一天只有这么多时间，如果在这里或那里偷工减料可以让你在更短的时间内为用户提供更多的功能，这可能会大大提高用户的幸福感和工作效率。诀窍在于，要弄清楚哪些弯路不会导致你的用户和未来的自己不会咒骂你的短视行为。

</aside>

^code return-this (2 before, 1 after)

If the function is an initializer, we override the actual return value and
forcibly return `this`. That relies on a new `isInitializer` field.
如果该函数是一个初始化方法，我们会覆盖实际的返回值并强行返回`this`。这个操作依赖于一个新的`isInitializer`字段。

^code is-initializer-field (2 before, 2 after)

We can't simply see if the name of the LoxFunction is "init" because the user
could have defined a *function* with that name. In that case, there *is* no
`this` to return. To avoid *that* weird edge case, we'll directly store whether
the LoxFunction represents an initializer method. That means we need to go back
and fix the few places where we create LoxFunctions.
我们不能简单地检查LoxFunction的名字是否为“init”，因为用户可能已经定义了一个同名的*函数*。在这种情况下，是没有`this`可供返回的。为了避免这种奇怪的边缘情况，我们将直接存储LoxFunction是否表示一个初始化方法。这意味着我们需要回头修正我们创建LoxFunctions的几个地方。

^code construct-function (1 before, 1 after)

For actual function declarations, `isInitializer` is always false. For methods,
we check the name.
对于实际的函数声明， `isInitializer`取值总是false。对于方法来说，我们检查其名称。

^code interpreter-method-initializer (1 before, 1 after)

And then in `bind()` where we create the closure that binds `this` to a method,
we pass along the original method's value.
然后在`bind()`方法，在创建闭包并将`this`绑定到新方法时，我们将原始方法的值传递给新方法。

^code lox-function-bind-with-initializer (1 before, 1 after)

### 从init()返回

We aren't out of the woods yet. We've been assuming that a user-written
initializer doesn't explicitly return a value because most constructors don't.
What should happen if a user tries:
我们还没有走出困境。我们一直假设用户编写的初始化方法不会显式地返回一个值，因为大多数构造函数都不会。如果用户尝试这样做会发生什么：

```lox
class Foo {
  init() {
    return "something else";
  }
}
```

It's definitely not going to do what they want, so we may as well make it a
static error. Back in the resolver, we add another case to FunctionType.
这肯定不会按照用户的期望执行，所以我们不妨把它作为一种静态错误。回到分析器中，我们为FunctionType添加另一种情况。

^code function-type-initializer (1 before, 1 after)

We use the visited method's name to determine if we're resolving an initializer
or not.
我们通过被访问方法的名称来确定我们是否在分析一个初始化方法。

^code resolver-initializer-type (1 before, 1 after)

When we later traverse into a `return` statement, we check that field and make
it an error to return a value from inside an `init()` method.
当我们稍后遍历`return`语句时，我们会检查该字段，如果从`init()`方法内部返回一个值时就抛出一个错误。

^code return-in-initializer (1 before, 1 after)

We're *still* not done. We statically disallow returning a *value* from an
initializer, but you can still use an empty early `return`.
我们*仍然*没有结束。我们静态地禁止了从初始化方法返回一个值，但是你仍然可用使用一个空的`return`。

```lox
class Foo {
  init() {
    return;
  }
}
```

That is actually kind of useful sometimes, so we don't want to disallow it
entirely. Instead, it should return `this` instead of `nil`. That's an easy fix
over in LoxFunction.
有时候这实际上是有用的，所以我们不想完全禁止它。相对地，它应该返回`this`而不是`nil`。这在LoxFunction中很容易解决。

^code early-return-this (1 before, 1 after)

If we're in an initializer and execute a `return` statement, instead of
returning the value (which will always be `nil`), we again return `this`.
如果我们在一个初始化方法中执行`return`语句时，我们仍然返回`this`，而不是返回值（该值始终是`nil`）。

Phew! That was a whole list of tasks but our reward is that our little
interpreter has grown an entire programming paradigm. Classes, methods, fields,
`this`, and constructors. Our baby language is looking awfully grown-up.
吁！这是一大堆任务，但是我们的收获是，我们的小解释器已经成长为一个完整的编程范式。类、方法、字段、`this`以及构造函数，我们的语言看起来已经非常成熟了。

<div class="challenges">

## Challenges

1.  We have methods on instances, but there is no way to define "static" methods
    that can be called directly on the class object itself. Add support for
    them. Use a `class` keyword preceding the method to indicate a static method
    that hangs off the class object.
    我们有实例上的方法，但是没有办法定义可以直接在类对象上调用的“静态”方法。添加对它们的支持，在方法之前使用`class`关键字指示该方法是一个挂载在类对象上的静态方法。

    ```lox
    class Math {
      class square(n) {
        return n * n;
      }
    }

    print Math.square(3); // Prints "9".
    ```

    You can solve this however you like, but the "[metaclasses][]" used by
    Smalltalk and Ruby are a particularly elegant approach. *Hint: Make LoxClass
    extend LoxInstance and go from there.*
    你可以用你喜欢的方式解决这问题，但是Smalltalk和Ruby使用的“[metaclasses](https://en.wikipedia.org/wiki/Metaclass)” 是一种特别优雅的方法。*提示：让LoxClass继承LoxInstance，然后开始实现。*

2.  Most modern languages support "getters" and "setters" -- members on a class
    that look like field reads and writes but that actually execute user-defined
    code. Extend Lox to support getter methods. These are declared without a
    parameter list. The body of the getter is executed when a property with that
    name is accessed.
    大多数现代语言都支持“getters”和“setters” -- 类中的成员，看起来像是字段的读写，但实际上执行的用户自定义的代码。扩展Lox以支持getter方法。这些方法在声明时没有参数列表。当访问具有该名称的属性时，会执行getter的主体。

    ```lox
    class Circle {
      init(radius) {
        this.radius = radius;
      }

      area {
        return 3.141592653 * this.radius * this.radius;
      }
    }

    var circle = Circle(4);
    print circle.area; // Prints roughly "50.2655".
    ```

3.  Python and JavaScript allow you to freely access an object's fields from
    outside of its own methods. Ruby and Smalltalk encapsulate instance state.
    Only methods on the class can access the raw fields, and it is up to the
    class to decide which state is exposed. Most statically typed languages
    offer modifiers like `private` and `public` to control which parts of a
    class are externally accessible on a per-member basis.
    Python和JavaScript允许你从对象自身的方法之外的地方自由访问对象的字段。Ruby和Smalltalk封装了实例状态。只有类上的方法可以访问原始字段，并且由类来决定哪些状态被暴露。大多数静态类型的语言都提供了像`private`和`public`这样的修饰符，以便按成员维度控制类的哪些部分可以被外部访问。

    What are the trade-offs between these approaches and why might a language
    prefer one or the other?
    这些方式之间的权衡是什么？为什么一门语言可能会更偏爱某一种方法？

[metaclasses]: https://en.wikipedia.org/wiki/Metaclass

</div>

<div class="design-note">

## Design Note: 原型与功率(Prototypes and Power)

In this chapter, we introduced two new runtime entities, LoxClass and
LoxInstance. The former is where behavior for objects lives, and the latter is
for state. What if you could define methods right on a single object, inside
LoxInstance? In that case, we wouldn't need LoxClass at all. LoxInstance would
be a complete package for defining the behavior and state of an object.

We'd still want some way, without classes, to reuse behavior across multiple
instances. We could let a LoxInstance [*delegate*][delegate] directly to another
LoxInstance to reuse its fields and methods, sort of like inheritance.

Users would model their program as a constellation of objects, some of which
delegate to each other to reflect commonality. Objects used as delegates
represent "canonical" or "prototypical" objects that others refine. The result
is a simpler runtime with only a single internal construct, LoxInstance.

That's where the name **[prototypes][proto]** comes from for this paradigm. It
was invented by David Ungar and Randall Smith in a language called [Self][].
They came up with it by starting with Smalltalk and following the above mental
exercise to see how much they could pare it down.

Prototypes were an academic curiosity for a long time, a fascinating one that
generated interesting research but didn't make a dent in the larger world of
programming. That is, until Brendan Eich crammed prototypes into JavaScript,
which then promptly took over the world. Many (many) <span
name="words">words</span> have been written about prototypes in JavaScript.
Whether that shows that prototypes are brilliant or confusing -- or both! -- is
an open question.

<aside name="words">

Including [more than a handful][prototypes] by yours truly.
其中[不乏][prototypes]你们的作品。

</aside>

I won't get into whether or not I think prototypes are a good idea for a
language. I've made languages that are [prototypal][finch] and
[class-based][wren], and my opinions of both are complex. What I want to discuss
is the role of *simplicity* in a language.

Prototypes are simpler than classes -- less code for the language implementer to
write, and fewer concepts for the user to learn and understand. Does that make
them better? We language nerds have a tendency to fetishize minimalism.
Personally, I think simplicity is only part of the equation. What we really want
to give the user is *power*, which I define as:

```text
power = breadth × ease ÷ complexity
```

None of these are precise numeric measures. I'm using math as analogy here, not
actual quantification.

*   **Breadth** is the range of different things the language lets you express.
    C has a lot of breadth -- it's been used for everything from operating
    systems to user applications to games. Domain-specific languages like
    AppleScript and Matlab have less breadth.

*   **Ease** is how little effort it takes to make the language do what you
    want. "Usability" might be another term, though it carries more baggage than
    I want to bring in. "Higher-level" languages tend to have more ease than
    "lower-level" ones. Most languages have a "grain" to them where some things
    feel easier to express than others.

*   **Complexity** is how big the language (including its runtime, core libraries,
    tools, ecosystem, etc.) is. People talk about how many pages are in a
    language's spec, or how many keywords it has. It's how much the user has to
    load into their wetware before they can be productive in the system. It is
    the antonym of simplicity.

[proto]: https://en.wikipedia.org/wiki/Prototype-based_programming

Reducing complexity *does* increase power. The smaller the denominator, the
larger the resulting value, so our intuition that simplicity is good is valid.
However, when reducing complexity, we must take care not to sacrifice breadth or
ease in the process, or the total power may go down. Java would be a strictly
*simpler* language if it removed strings, but it probably wouldn't handle text
manipulation tasks well, nor would it be as easy to get things done.

The art, then, is finding *accidental* complexity that can be omitted --
language features and interactions that don't carry their weight by increasing
the breadth or ease of using the language.

If users want to express their program in terms of categories of objects, then
baking classes into the language increases the ease of doing that, hopefully by
a large enough margin to pay for the added complexity. But if that isn't how
users are using your language, then by all means leave classes out.

</div>


<div class="design-note">

在本章中，我们引入了两个新的运行时实体，LoxClass和LoxInstance。前者是对象的行为所在，后者则是状态所在。如果你可以在LoxInstance的单个对象中定义方法，会怎么样？这种情况下，我们根本就不需要LoxClass。LoxInstance将是一个用于定义对象行为和状态的完整包。

我们仍然需要一些方法，在没有类的情况下，可以跨多个实例重用对象行为。我们可以让一个LoxInstance直接[委托](https://en.wikipedia.org/wiki/Prototype-based_programming#Delegation)给另一个LoxInstance来重用它的字段和方法，有点像继承。

用户可以将他们的程序建模为一组对象，其中一些对象相互委托以反映共性。用作委托的对象代表“典型”或“原型”对象，会被其它对象完善。结果就是会有一个更简单的运行时，只有一个内部结构LoxInstance。

这就是这种范式的名称“[原型](https://en.wikipedia.org/wiki/Prototype-based_programming)”的由来。它是由David Ungar和Randall Smith在一种叫做[Self](http://www.selflanguage.org/)的语言中发明的。他们从Smalltalk开始，按照上面的练习，看他们能把它缩减到什么程度，从而想到了这个方法。

长期以来，原型一直是学术上的探索，它是一个引人入胜的东西，也产生了有趣的研究，但是并没有在更大的编程世界中产生影响。直到Brendan Eich把原型塞进JavaScript，然后迅速风靡世界。关于JavaScript中的原型，人们已经写了很多（许多）<span
name="words_zh">文字</span>。这是否能够表明原型是出色的还是令人困惑的，或者兼而有之？这是一个开放的问题。

<aside name="words_zh">

其中[不乏][prototypes]你们的作品。

</aside>

我不会去讨论原型对于一门语言来说是不是一个好主意。基于原型和基于类的语言我都做过，我对两者的看法很复杂。我想讨论的是*简单性*在一门语言中的作用。

原型比类更简单 -- 语言实现者要编写的代码更少，语言用户要学习和理解的概念更少。这是否意味着它让语言变得更好呢？我们这些语言书呆子有一种迷恋极简主义的倾向。就我个人而言，我认为简单性只是一部分。我们真正想给用户的是功率，我将其定义为：

```text
power = breadth × ease ÷ complexity
功率 = 广度 × 易用性 ÷ 复杂性
```

这些都不是精确的数字度量。我这里用数学作比喻，而不是实际的量化。

* **广度**是语言可以表达的不同事物的范围。C语言具有很大的广度 -- 从操作系统到用户应用程序再到游戏，它被广泛使用。像AppleScript和Matlab这样的特定领域语言的广度相对较小。
* 易用性是指用户付出多少努力就可以用语言做想做的事。“可用性Usability”是另一个概念，它包含的内容比我想要表达的更多。“高级”语言往往比“低级”语言更容易使用。大多数语言都有一个核心，对它们来说，有些东西比其它的更容易表达。
* 复杂性是指语言的规模（包括其运行时、核心库、工具、生态等）有多大。人们谈论一种语言的规范有多少页，或者它有多少个关键词。这是指用户在使用系统之前，必须在先学习多少东西，才能产生效益。它是简单性的反义词。

降低复杂性确实可以提高功率，分母越小，得到的值就越大，所以我们直觉认为“简单的是好的”是对的。然而，在降低复杂性时，我们必须注意不要在这个过程中牺牲广度或易用性，否则总功率可能会下降。如果去掉字符串，Java将变成一种严格意义上的简单语言，但它可能无法很好地处理文本操作任务，也不会那么容易完成事情。

因此，关键就在于找到可以省略的意外复杂性，也就是哪些没有通过增加语言广度或语言易用性来体现其重要性的语言特性与交互。

如果用户想用对象的类别来表达他们的程序，那么在语言中加入类就能提高这类操作的便利性，希望能有足够大的提升幅度来弥补所增加的复杂性。但如果这不是用户使用您的语言的方式，那么无论如何都不要使用类。

</div>

[delegate]: https://en.wikipedia.org/wiki/Prototype-based_programming#Delegation
[prototypes]: http://gameprogrammingpatterns.com/prototype.html
[self]: http://www.selflanguage.org/
[finch]: http://finch.stuffwithstuff.com/
[wren]: http://wren.io/


