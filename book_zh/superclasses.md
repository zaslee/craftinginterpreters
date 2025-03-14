> You can choose your friends but you sho' can't choose your family, an' they're
> still kin to you no matter whether you acknowledge &rsquo;em or not, and it
> makes you look right silly when you don't.
>
> 你可以选择你的朋友，但无法选择你的家庭，所以不管你承认与否，他们都是你的亲属，而且不承认会让你显得很蠢。
>
> <cite>Harper Lee, <em>To Kill a Mockingbird</em></cite>
> <cite>哈珀 · 李, <em>杀死一只知更鸟</em></cite>

This is the very last chapter where we add new functionality to our VM. We've
packed almost the entire Lox language in there already. All that remains is
inheriting methods and calling superclass methods. We have [another
chapter][optimization] after this one, but it introduces no new behavior. It
<span name="faster">only</span> makes existing stuff faster. Make it to the end
of this one, and you'll have a complete Lox implementation.
这是我们向虚拟机添加新功能的最后一章。我们已经把几乎所有的Lox语言都装进虚拟机中了。剩下的就是继承方法和调用超类方法。在本章之后[还有一章][optimization]，但是没有引入新的行为。它只是让现有的东西更快。坚持到本章结束，你将拥有一个完整的Lox实现。

<aside name="faster">

That "only" should not imply that making stuff faster isn't important! After
all, the whole purpose of our entire second virtual machine is better
performance over jlox. You could argue that *all* of the past fifteen chapters
are "optimization".
这个“只是”并不意味着加速不重要！毕竟，我们的第二个虚拟机的全部目的就是比jlox有更好的性能。你可以认为，前面的15章都是“优化”。

</aside>

[optimization]: optimization.html

Some of the material in this chapter will remind you of jlox. The way we resolve
super calls is pretty much the same, though viewed through clox's more complex
mechanism for storing state on the stack. But we have an entirely different,
much faster, way of handling inherited method calls this time around.
本章中的一些内容会让你想起jlox。我们解决超类调用的方式几乎是一样的，即便是从clox这种在栈中存储状态的更复杂的机制来看。但这次我们会用一种完全不同的、更快的方式来处理继承方法的调用。

## 继承方法

We'll kick things off with method inheritance since it's the simpler piece. To
refresh your memory, Lox inheritance syntax looks like this:
我们会从方法继承开始，因为它是比较简单的部分。为了恢复你的记忆，Lox的继承语法如下所示：

```lox
class Doughnut {
  cook() {
    print "Dunk in the fryer.";
  }
}

class Cruller < Doughnut {
  finish() {
    print "Glaze with icing.";
  }
}
```

Here, the Cruller class inherits from Doughnut and thus, instances of Cruller
inherit the `cook()` method. I don't know why I'm belaboring this. You know how
inheritance works. Let's start compiling the new syntax.
这里，Culler类继承自Doughnut，因此，Cruller的实例继承了`cook()`方法。我不明白我为什么要反复强调这个，你知道继承是怎么回事。让我们开始编译新语法。

^code compile-superclass (2 before, 1 after)

After we compile the class name, if the next token is a `<`, then we found a
superclass clause. We consume the superclass's identifier token, then call
`variable()`. That function takes the previously consumed token, treats it as a
variable reference, and emits code to load the variable's value. In other words,
it looks up the superclass by name and pushes it onto the stack.
在编译类名之后，如果下一个标识是`<`，那我们就找到了一个超类子句。我们消耗超类的标识符，然后调用`variable()`。该函数接受前面消耗的标识，将其视为变量引用，并发出代码来加载变量的值。换句话说，它通过名称查找超类并将其压入栈中。

After that, we call `namedVariable()` to load the subclass doing the inheriting
onto the stack, followed by an `OP_INHERIT` instruction. That instruction
wires up the superclass to the new subclass. In the last chapter, we defined an
`OP_METHOD` instruction to mutate an existing class object by adding a method to
its method table. This is similar -- the `OP_INHERIT` instruction takes an
existing class and applies the effect of inheritance to it.
之后，我们调用`namedVariable()`将进行继承的子类加载到栈中，接着是`OP_INHERIT`指令。该指令将超类与新的子类连接起来。在上一章中，我们定义了一条`OP_METHOD`指令，通过向已有类对象的方法表中添加方法来改变它。这里是类似的 -- `OP_INHERIT`指令接受一个现有的类，并对其应用继承的效果。

In the previous example, when the compiler works through this bit of syntax:
在前面的例子中，当编译器处理这些语法时：

```lox
class Cruller < Doughnut {
```

The result is this bytecode:
结果就是这个字节码：

<img src="image/superclasses/inherit-stack.png" alt="The series of bytecode instructions for a Cruller class inheriting from Doughnut." />

Before we implement the new `OP_INHERIT` instruction, we have an edge case to
detect.
在我们实现新的`OP_INHERIT`指令之前，还需要检测一个边界情况。

^code inherit-self (1 before, 1 after)

<span name="cycle">A</span> class cannot be its own superclass. Unless you have
access to a deranged nuclear physicist and a very heavily modified DeLorean, you
cannot inherit from yourself.
一个类不能成为它自己的超类。除非你能接触到一个核物理学家和一辆改装过的DeLorean汽车，否则你无法继承自己。

<aside name="cycle">

Interestingly, with the way we implement method inheritance, I don't think
allowing cycles would actually cause any problems in clox. It wouldn't do
anything *useful*, but I don't think it would cause a crash or infinite loop.
有趣的是，根据我们实现方法继承的方式，我认为允许循环实际上不会在clox中引起任何问题。它不会做任何 *有用* 的事情，但我认为它不会导致崩溃或无限循环。

</aside>

### 执行继承

Now onto the new instruction.
现在来看新指令。

^code inherit-op (1 before, 1 after)

There are no operands to worry about. The two values we need -- superclass and
subclass -- are both found on the stack. That means disassembling is easy.
不需要担心任何操作数。我们需要的两个值 -- 超类和子类 -- 都可以在栈中找到。这意味着反汇编很容易。

^code disassemble-inherit (1 before, 1 after)

The interpreter is where the action happens.
解释器是行为发生的地方。

^code interpret-inherit (1 before, 1 after)

From the top of the stack down, we have the subclass then the superclass. We
grab both of those and then do the inherit-y bit. This is where clox takes a
different path than jlox. In our first interpreter, each subclass stored a
reference to its superclass. On method access, if we didn't find the method in
the subclass's method table, we recursed through the inheritance chain looking
at each ancestor's method table until we found it.
从栈顶往下，我们依次有子类，然后是超类。我们获取这两个类，然后进行继承。这就是clox与jlox不同的地方。在我们的第一个解释器中，每个子类都存储了一个对其超类的引用。在访问方法时，如果我们没有在子类方法表中找到它，就通过继承链递归遍历每个祖先的方法表，直到找到该方法。

For example, calling `cook()` on an instance of Cruller sends jlox on this
journey:
例如，在Cruller的实例上调用`cook()`方法，jlox会这样做：

<img src="image/superclasses/jlox-resolve.png" alt="Resolving a call to cook() in an instance of Cruller means walking the superclass chain." />

That's a lot of work to perform during method *invocation* time. It's slow, and
worse, the farther an inherited method is up the ancestor chain, the slower it
gets. Not a great performance story.
在方法*调用*期间要做大量的工作。这很慢，而且更糟糕的是，继承的方法在祖先链上越远，它就越慢。这不是一个好的性能故事。

The new approach is much faster. When the subclass is declared, we copy all of
the inherited class's methods down into the subclass's own method table. Later,
when *calling* a method, any method inherited from a superclass will be found
right in the subclass's own method table. There is no extra runtime work needed
for inheritance at all. By the time the class is declared, the work is done.
This means inherited method calls are exactly as fast as normal method calls --
a <span name="two">single</span> hash table lookup.
新方法则要快得多。当子类被声明时，我们将继承类的所有方法复制到子类自己的方法表中。之后，当我们*调用*某个方法时，从超类继承的任何方法都可以在子类自己的方法表中找到。继承根本不需要做额外的运行时工作。当类被声明时，工作就完成了。这意味着继承的方法和普通方法调用一样快 -- 只需要一次哈希表查询。

<img src="image/superclasses/clox-resolve.png" alt="Resolving a call to cook() in an instance of Cruller which has the method in its own method table." />

<aside name="two">

Well, two hash table lookups, I guess. Because first we have to make sure a
field on the instance doesn't shadow the method.
好吧，我想应该是两次哈希查询。因为首先我们必须确保实例上的字段不会遮蔽方法。

</aside>

I've sometimes heard this technique called "copy-down inheritance". It's simple
and fast, but, like most optimizations, you get to use it only under certain
constraints. It works in Lox because Lox classes are *closed*. Once a class
declaration is finished executing, the set of methods for that class can never
change.
我有时听到这种技术被称为“向下复制继承”。它简单而快速，但是，与大多数优化一样，你只能在特定的约束条件下使用它。它适用于Lox，是因为Lox的类是*关闭*的。一旦某个类的声明执行完毕，该类的方法集就永远不能更改。

In languages like Ruby, Python, and JavaScript, it's possible to <span
name="monkey">crack</span> open an existing class and jam some new methods into
it or even remove them. That would break our optimization because if those
modifications happened to a superclass *after* the subclass declaration
executed, the subclass would not pick up those changes. That breaks a user's
expectation that inheritance always reflects the current state of the
superclass.
在Ruby、Python和JavaScript等语言中，可以打开一个现有的类，并将一些新方法加入其中，甚至删除方法。这会破坏我们的优化，因为如果这些修改在子类声明执行*之后*发生在超类上，子类就不会获得这些变化。这就打破了用户的期望，即继承总是反映超类的当前状态。

<aside name="monkey">

As you can imagine, changing the set of methods a class defines imperatively at
runtime can make it hard to reason about a program. It is a very powerful tool,
but also a dangerous tool.
可以想见，在运行时改变某个类中以命令式定义的方法集会使得对程序的推理变得困难。这是一个非常强大的工具，但也是一个危险的工具。

Those who find this tool maybe a little *too* dangerous gave it the unbecoming
name "monkey patching", or the even less decorous "duck punching".
那些认为这个工具可能有点太危险的人，给它取了个不伦不类的名字“猴子补丁”，或者是更不体面的“鸭子打洞”。

<img src="image/superclasses/monkey.png" alt="A monkey with an eyepatch, naturally." />

</aside>

Fortunately for us (but not for users who like the feature, I guess), Lox
doesn't let you patch monkeys or punch ducks, so we can safely apply this
optimization.
幸运的是（我猜对于喜欢这一特性的用户来说不算幸运），Lox不允许猴子补丁或鸭子打洞，所以我们可以安全的应用这种优化。

What about method overrides? Won't copying the superclass's methods into the
subclass's method table clash with the subclass's own methods? Fortunately, no.
We emit the `OP_INHERIT` after the `OP_CLASS` instruction that creates the
subclass but before any method declarations and `OP_METHOD` instructions have
been compiled. At the point that we copy the superclass's methods down, the
subclass's method table is empty. Any methods the subclass overrides will
overwrite those inherited entries in the table.
那方法重写呢？将超类的方法复制到子类的方法表中，不会与子类自己的方法发生冲突吗？幸运的是，不会。我们是在创建子类的`OP_CLASS`指令之后、但在任何方法声明和`OP_METHOD`指令被编译之前发出`OP_INHERIT`指令。当我们将超类的方法复制下来时，子类的方法表是空的。子类重写的任何方法都会覆盖表中那些继承的条目。

### 无效超类

Our implementation is simple and fast, which is just the way I like my VM code.
But it's not robust. Nothing prevents a user from inheriting from an object that
isn't a class at all:
我们的实现简单而快速，这正是我喜欢我的VM代码的原因。但它并不健壮。没有什么能阻止用户继承一个根本不是类的对象：

```lox
var NotClass = "So not a class";
class OhNo < NotClass {}
```

Obviously, no self-respecting programmer would write that, but we have to guard
against potential Lox users who have no self respect. A simple runtime check
fixes that.
显然，任何一个有自尊心的程序员都不会写这种东西，但我们必须堤防那些没有自尊心的潜在Lox用户。一个简单的运行时检查就可以解决这个问题。

^code inherit-non-class (1 before, 1 after)

If the value we loaded from the identifier in the superclass clause isn't an
ObjClass, we report a runtime error to let the user know what we think of them
and their code.
如果我们从超类子句的标识符中加载到的值不是ObjClass，就报告一个运行时错误，让用户知道我们对他们及其代码的看法。

## 存储超类

Did you notice that when we added method inheritance, we didn't actually add any
reference from a subclass to its superclass? After we copy the inherited methods
over, we forget the superclass entirely. We don't need to keep a handle on the
superclass, so we don't.
你是否注意到，在我们添加方法继承时，实际上并没有添加任何从子类指向超类的引用？我们把继承的方法复制到子类之后，就完全忘记了超类。我们不需要保存超类的句柄，所以我们没有这样做。

That won't be sufficient to support super calls. Since a subclass <span
name="may">may</span> override the superclass method, we need to be able to get
our hands on superclass method tables. Before we get to that mechanism, I want 
to refresh your memory on how super calls are statically resolved.
这不足以支持超类调用。因为子类可能会覆盖超类方法，我们需要能够获得超类方法表。在讨论这个机制之前，我想让你回忆一下如何静态解析超类调用。

<aside name="may">

"May" might not be a strong enough word. Presumably the method *has* been
overridden. Otherwise, why are you bothering to use `super` instead of just
calling it directly?
“可能”这个词也许不够有力。大概这个方法*已经*被重写了。否则，你为什么要费力地使用`super`而不是直接调用它呢？

</aside>

Back in the halcyon days of jlox, I showed you [this tricky example][example] to
explain the way super calls are dispatched:
回顾jlox的光辉岁月，我给你展示了这个棘手的示例，来解释超类调用的分派方式：

[example]: inheritance.html#semantics

```lox
class A {
  method() {
    print "A method";
  }
}

class B < A {
  method() {
    print "B method";
  }

  test() {
    super.method();
  }
}

class C < B {}

C().test();
```

Inside the body of the `test()` method, `this` is an instance of C. If super
calls were resolved relative to the superclass of the *receiver*, then we would
look in C's superclass, B. But super calls are resolved relative to the
superclass of the *surrounding class where the super call occurs*. In this case,
we are in B's `test()` method, so the superclass is A, and the program should
print "A method".
在`test()`方法的主体中，`this`是C的一个实例。如果超类调用是在*接收器*的超类中来解析的，那我们会在C的超类B中寻找方法。但是超类调用是在*发生超类调用的外围类*的超类中解析的。在本例中，我们在B的`test()`方法中，因此超类是A，程序应该打印“A method”。

This means that super calls are not resolved dynamically based on the runtime
instance. The superclass used to look up the method is a static -- practically
lexical -- property of where the call occurs. When we added inheritance to jlox,
we took advantage of that static aspect by storing the superclass in the same
Environment structure we used for all lexical scopes. Almost as if the
interpreter saw the above program like this:
这意味着超类调用不是根据运行时的实例进行动态解析的。用于查找方法的超类是调用发生位置的一个静态（实际上是词法）属性。当我们在jlox中添加继承时，我们利用了这种静态优势，将超类存储在我们用于所有词法作用域的同一个Environment结构中。就好像解释器看到的程序是这样的：

```lox
class A {
  method() {
    print "A method";
  }
}

var Bs_super = A;
class B < A {
  method() {
    print "B method";
  }

  test() {
    runtimeSuperCall(Bs_super, "method");
  }
}

var Cs_super = B;
class C < B {}

C().test();
```

Each subclass has a hidden variable storing a reference to its superclass.
Whenever we need to perform a super call, we access the superclass from that
variable and tell the runtime to start looking for methods there.
每个子类都有一个隐藏变量，用于存储对其超类的引用。当我们需要执行一个超类调用时，我们就从这个变量访问超类，并告诉运行时从那里开始查找方法。

We'll take the same path with clox. The difference is that instead of jlox's
heap-allocated Environment class, we have the bytecode VM's value stack and
upvalue system. The machinery is a little different, but the overall effect is
the same.
我们在clox中采用相同的方法。不同之处在于，我们使用的是字节码虚拟机的值栈和上值系统，而不是jlox的堆分配的Environment 类。机制有些不同，但总体效果是一样的。

### 超类局部变量

Our compiler already emits code to load the superclass onto the stack. Instead
of leaving that slot as a temporary, we create a new scope and make it a local
variable.
我们的编译器已经发出了将超类加载到栈中的代码。我们不将这个槽看作是临时的，而是创建一个新的作用域，并将其作为一个局部变量。

^code superclass-variable (2 before, 2 after)

Creating a new lexical scope ensures that if we declare two classes in the same
scope, each has a different local slot to store its superclass. Since we always
name this variable "super", if we didn't make a scope for each subclass, the
variables would collide.
创建一个新的词法作用域可以确保，如果我们在同一个作用域中声明两个类，每个类都有一个不同的局部槽来存储其超类。由于我们总是将该变量命名为“super”，如果我们不为每个子类创建作用域，那么这些变量就会发生冲突。

We name the variable "super" for the same reason we use "this" as the name of
the hidden local variable that `this` expressions resolve to: "super" is a
reserved word, which guarantees the compiler's hidden variable won't collide
with a user-defined one.
我们将该变量命名为“super”，与我们使用“this”作为`this`表达式解析得到的隐藏局部变量名称的原因相同：“super”是一个保留字，它可以保证编译器的隐藏变量不会与用户定义的变量发生冲突。

The difference is that when compiling `this` expressions, we conveniently have a
token sitting around whose lexeme is "this". We aren't so lucky here. Instead,
we add a little helper function to create a synthetic token for the given <span
name="constant">constant</span> string.
不同之处在于，在编译`this`表达式时，我们可以很方便地使用一个标识，词素是`this`。在这里我们就没那么幸运了。相对地，我们添加一个小的辅助函数，来为给定的常量字符串创建一个合成标识。

^code synthetic-token

<aside name="constant" class="bottom">

I say "constant string" because tokens don't do any memory management of their
lexeme. If we tried to use a heap-allocated string for this, we'd end up leaking
memory because it never gets freed. But the memory for C string literals lives
in the executable's constant data section and never needs to be freed, so we're
fine.
我说“常量字符串”是因为标识不对其词素做任何内存管理。如果我们试图使用堆分配的字符串，最终会泄漏内存，因为它永远不会被释放。但是，C语言字符串字面量的内存位于可执行文件的常量数据部分，永远不需要释放，所以我们这样没有问题。

</aside>

Since we opened a local scope for the superclass variable, we need to close it.
因为我们为超类变量打开了一个局部作用域，我们还需要关闭它。

^code end-superclass-scope (1 before, 2 after)

We pop the scope and discard the "super" variable after compiling the class body
and its methods. That way, the variable is accessible in all of the methods of
the subclass. It's a somewhat pointless optimization, but we create the scope
only if there *is* a superclass clause. Thus we need to close the scope only if
there is one.
在编译完类的主体及其方法后，我们会弹出作用域并丢弃“super”变量。这样，该变量在子类的所有方法中被都可以访问。这是一个有点无意义的优化，但我们只在有超类子句的情况下创建作用域。因此，只有在有超类的情况下，我们才需要关闭这个作用域。

To track that, we could declare a little local variable in `classDeclaration()`.
But soon, other functions in the compiler will need to know whether the
surrounding class is a subclass or not. So we may as well give our future selves
a hand and store this fact as a field in the ClassCompiler now.
为了记录是否有超类，我们可以在`classDeclaration()`中声明一个局部变量。但是很快，编译器中的其它函数需要知道外层的类是否是子类。所以我们不妨帮帮未来的自己，现在就把它作为一个字段存储在ClassCompiler中。

^code has-superclass (2 before, 1 after)

When we first initialize a ClassCompiler, we assume it is not a subclass.
当我们第一次初始化某个ClassCompiler时，我们假定它不是子类。

^code init-has-superclass (1 before, 1 after)

Then, if we see a superclass clause, we know we are compiling a subclass.
然后，如果看到超类子句，我们就知道正在编译一个子类。

^code set-has-superclass (1 before, 1 after)

This machinery gives us a mechanism at runtime to access the superclass object
of the surrounding subclass from within any of the subclass's methods -- simply
emit code to load the variable named "super". That variable is a local outside
of the method body, but our existing upvalue support enables the VM to capture
that local inside the body of the method or even in functions nested inside that
method.
这种机制在运行时为我们提供了一种方法，可以从子类的任何方法中访问外层子类的超类对象 -- 只需发出代码来加载名为“super”的变量。这个变量是方法主体之外的一个局部变量，但是我们现有的上值支持VM在方法主体内、甚至是嵌套方法内的函数中捕获该局部变量。

## 超类调用

With that runtime support in place, we are ready to implement super calls. As
usual, we go front to back, starting with the new syntax. A super call <span
name="last">begins</span>, naturally enough, with the `super` keyword.
有了这个运行时支持，我们就可以实现超类调用了。跟之前一样，我们从前端到后端，先从新语法开始。超类调用，自然是以`super`关键字开始。

<aside name="last">

This is it, friend. The very last entry you'll add to the parsing table.
就是这样，朋友，你要添加到解析表中的最后一项。

</aside>

^code table-super (1 before, 1 after)

When the expression parser lands on a `super` token, control jumps to a new
parsing function which starts off like so:
当表达式解析器落在一个`super`标识时，控制流会跳转到一个新的解析函数，该函数的开头是这样的：

^code super

This is pretty different from how we compiled `this` expressions. Unlike `this`,
a `super` <span name="token">token</span> is not a standalone expression.
Instead, the dot and method name following it are inseparable parts of the
syntax. However, the parenthesized argument list is separate. As with normal
method access, Lox supports getting a reference to a superclass method as a
closure without invoking it:
这与我们编译`this`表达式的方式很不一样。与`this`不同，`super`标识不是一个独立的表达式。相反，它后面的点和方法名称是语法中不可分割的部分。但是，括号内的参数列表是独立的。和普通的方法访问一样，Lox支持以闭包的方式获得对超类方法的引用，而不必调用它：

<aside name="token">

Hypothetical question: If a bare `super` token *was* an expression, what kind of
object would it evaluate to?
假设性问题：如果一个光秃秃的`super`标识 *是* 一个表达式，那么它会被计算为哪种对象呢？

</aside>

```lox
class A {
  method() {
    print "A";
  }
}

class B < A {
  method() {
    var closure = super.method;
    closure(); // Prints "A".
  }
}
```

In other words, Lox doesn't really have super *call* expressions, it has super
*access* expressions, which you can choose to immediately invoke if you want. So
when the compiler hits a `super` token, we consume the subsequent `.` token and
then look for a method name. Methods are looked up dynamically, so we use
`identifierConstant()` to take the lexeme of the method name token and store it
in the constant table just like we do for property access expressions.
换句话说，Lox并没有真正的超类*调用（call）*表达式，它有的是超类*访问（access）*表达式，如果你愿意，可以选择立即调用。因此，当编译器碰到一个`super`标识时，我们会消费后续的`.`标识，然后寻找一个方法名称。方法是动态查找的，所以我们使用`identifierConstant()`来获取方法名标识的词素，并将其存储在常量表中，就像我们对属性访问表达式所做的那样。

Here is what the compiler does after consuming those tokens:
下面是编译器在消费这些标识之后做的事情：

^code super-get (1 before, 1 after)

In order to access a *superclass method* on *the current instance*, the runtime
needs both the receiver *and* the superclass of the surrounding method's class.
The first `namedVariable()` call generates code to look up the current receiver
stored in the hidden variable "this" and push it onto the stack. The second
`namedVariable()` call emits code to look up the superclass from its "super"
variable and push that on top.
为了在*当前实例*上访问一个*超类方法*，运行时需要接收器*和*外围方法所在类的超类。第一个`namedVariable()`调用产生代码来查找存储在隐藏变量“this”中的当前接收器，并将其压入栈中。第二个`namedVariable()`调用产生代码，从它的“super”变量中查找超类，并将其推入栈顶。

Finally, we emit a new `OP_GET_SUPER` instruction with an operand for the
constant table index of the method name. That's a lot to hold in your head. To
make it tangible, consider this example program:
最后，我们发出一条新的`OP_GET_SUPER`指令，其操作数为方法名称的常量表索引。你脑子里装的东西太多了。为了使它具体化，请看下面的示例程序：

```lox
class Doughnut {
  cook() {
    print "Dunk in the fryer.";
    this.finish("sprinkles");
  }

  finish(ingredient) {
    print "Finish with " + ingredient;
  }
}

class Cruller < Doughnut {
  finish(ingredient) {
    // No sprinkles, always icing.
    super.finish("icing");
  }
}
```

The bytecode emitted for the `super.finish("icing")` expression looks and works
like this:
`super.finish("icing")`发出的字节码看起来像是这样的：

<img src="image/superclasses/super-instructions.png" alt="The series of bytecode instructions for calling super.finish()." />

The first three instructions give the runtime access to the three pieces of
information it needs to perform the super access:
前三条指令让运行时获得了执行超类访问时需要的三条信息：

1.  The first instruction loads **the instance** onto the stack.
2.  The second instruction loads **the superclass where the method is
    resolved**.
3.  Then the new `OP_GET_SUPER` instuction encodes **the name of the method to
    access** as an operand.


1.  第一条指令将**实例**加载到栈中。
2.  第二条指令加载了**将用于解析方法的超类**。
3.  然后，新的`OP_GET_SUPER`指令将**要访问的方法名称**编码为操作数。

The remaining instructions are the normal bytecode for evaluating an argument
list and calling a function.
剩下的指令是用于计算参数列表和调用函数的常规字节码。

We're almost ready to implement the new `OP_GET_SUPER` instruction in the
interpreter. But before we do, the compiler has some errors it is responsible
for reporting.
我们几乎已经准备好在解释器中实现新的`OP_GET_SUPER`指令了。但在此之前，编译器需要负责报告一些错误。

^code super-errors (1 before, 1 after)

A super call is meaningful only inside the body of a method (or in a function
nested inside a method), and only inside the method of a class that has a
superclass. We detect both of these cases using the value of `currentClass`. If
that's `NULL` or points to a class with no superclass, we report those errors.
超类调用只有在方法主体（或方法中嵌套的函数）中才有意义，而且只在具有超类的某个类的方法中才有意义。我们使用`currentClass`的值来检测这两种情况。如果它是`NULL`或者指向一个没有超类的类，我们就报告这些错误。

### 执行超类访问

Assuming the user didn't put a `super` expression where it's not allowed, their
code passes from the compiler over to the runtime. We've got ourselves a new
instruction.
假设用户没有在不允许的地方使用`super`表达式，他们的代码将从编译器传递到运行时。我们已经有了一个新指令。

^code get-super-op (1 before, 1 after)

We disassemble it like other opcodes that take a constant table index operand.
我们像对其它需要常量表索引操作数的操作码一样对它进行反汇编。

^code disassemble-get-super (1 before, 1 after)

You might anticipate something harder, but interpreting the new instruction is
similar to executing a normal property access.
你可能预想这是一件比较困难的事，但解释新指令与执行正常的属性访问类似。

^code interpret-get-super (1 before, 1 after)

As with properties, we read the method name from the
constant table. Then we pass that to `bindMethod()` which looks up the method in
the given class's method table and creates an ObjBoundMethod to bundle the
resulting closure to the current instance.
和属性一样，我们从常量表中读取方法名。然后我们将其传递给`bindMethod()`，该方法会在给定类的方法表中查找方法，并创建一个ObjBoundMethod将结果闭包与当前实例相绑定。

The key <span name="field">difference</span> is *which* class we pass to
`bindMethod()`. With a normal property access, we use the ObjInstances's own
class, which gives us the dynamic dispatch we want. For a super call, we don't
use the instance's class. Instead, we use the statically resolved superclass of
the containing class, which the compiler has conveniently ensured is sitting on
top of the stack waiting for us.
关键的区别在于将*哪个*类传递给`bindMethod()`。对于普通的属性访问，我们使用ObjInstances自己的类，这为我们提供了我们想要的动态分派。对于超类调用，我们不使用实例的类。相反，我们使用静态分析得到的外层类的超类，编译器已经确保它在栈顶等着我们。

We pop that superclass and pass it to `bindMethod()`, which correctly skips over
any overriding methods in any of the subclasses between that superclass and the
instance's own class. It also correctly includes any methods inherited by the
superclass from any of *its* superclasses.
我们弹出该超类并将其传递给`bindMethod()`，该方法会正确地跳过该超类与实例本身的类之间的任何子类覆写的方法。它还正确地包含了超类从其任何超类中继承的方法。

The rest of the behavior is the same. Popping the superclass leaves the instance
at the top of the stack. When `bindMethod()` succeeds, it pops the instance and
pushes the new bound method. Otherwise, it reports a runtime error and returns
`false`. In that case, we abort the interpreter.
其余的行为都是一样的。超类弹出栈使得实例位于栈顶。当`bindMethod()`成功时，它会弹出实例并压入新的已绑定方法。否则，它会报告一个运行时错误并返回`false`。在这种情况下，我们中止解释器。

<aside name="field">

Another difference compared to `OP_GET_PROPERTY` is that we don't try to look
for a shadowing field first. Fields are not inherited, so `super` expressions
always resolve to methods.
与`OP_GET_PROPERTY`相比的另一个区别是，我们不会先尝试寻找遮蔽字段。字段不会被继承，所以`super`表达式总是解析为方法。

If Lox were a prototype-based language that used *delegation* instead of
*inheritance*, then instead of one *class* inheriting from another *class*,
instances would inherit from ("delegate to") other instances. In that case,
fields *could* be inherited, and we would need to check for them here.
如果Lox是一种使用 *委托* 而不是 *继承* 的基于原型的语言，那么就不是一个 *类* 继承另一个 *类*，而是实例继承自（委托给）其它实例。在这种情况下，字段可以被继承，我们就需要在这里检查它们。

</aside>

### 更快的超类调用

We have superclass method accesses working now. And since the returned object is
an ObjBoundMethod that you can then invoke, we've got super *calls* working too.
Just like last chapter, we've reached a point where our VM has the complete,
correct semantics.
我们现在有了对超类方法的访问。由于返回的对象是一个你可以稍后调用的ObjBoundMethod，我们也就有了可用的超类*调用*。就像上一章一样，我们的虚拟机现在已经有了完整、正确的语义。

But, also like last chapter, it's pretty slow. Again, we're heap allocating an
ObjBoundMethod for each super call even though most of the time the very next
instruction is an `OP_CALL` that immediately unpacks that bound method, invokes
it, and then discards it. In fact, this is even more likely to be true for
super calls than for regular method calls. At least with method calls there is
a chance that the user is actually invoking a function stored in a field. With
super calls, you're *always* looking up a method. The only question is whether
you invoke it immediately or not.
但是，也和上一章一样，它很慢。同样，我们为每个超类调用在堆中分配了一个ObjBoundMethod，尽管大多数时候下一个指令就是`OP_CALL`，它会立即解包该已绑定方法，调用它，然后丢弃它。事实上，超类调用比普通方法调用更有可能出现这种情况。至少在方法调用中，用户有可能实际上在调用存储在字段中的函数。在超类调用中，你肯定是在查找一个方法。唯一的问题在于你是否立即调用它。

The compiler can certainly answer that question for itself if it sees a left
parenthesis after the superclass method name, so we'll go ahead and perform the
same optimization we did for method calls. Take out the two lines of code that
load the superclass and emit `OP_GET_SUPER`, and replace them with this:
如果编译器看到超类方法名称后面有一个左括号，它肯定能自己回答这个问题，所以我们会继续执行与方法调用相同的优化。去掉加载超类并发出`OP_GET_SUPER`的两行代码，替换为这个：

^code super-invoke (1 before, 1 after)

Now before we emit anything, we look for a parenthesized argument list. If we
find one, we compile that. Then we load the superclass. After that, we emit a
new `OP_SUPER_INVOKE` instruction. This <span
name="superinstruction">superinstruction</span> combines the behavior of
`OP_GET_SUPER` and `OP_CALL`, so it takes two operands: the constant table index
of the method name to look up and the number of arguments to pass to it.
现在，在我们发出任何代码之前，我们要寻找一个带括号的参数列表。如果找到了，我们就编译它，任何加载超类，之后，我们发出一条新的`OP_SUPER_INVOKE`指令。这个超级指令结合了`OP_GET_SUPER`和`OP_CALL`的行为，所以它需要两个操作数：待查找的方法名称和要传递给它的参数数量。

<aside name="superinstruction">

This is a particularly *super* superinstruction, if you get what I'm saying.
I... I'm sorry for this terrible joke.
如果你明白我的意思，这是一个 *特别* 超级的超级指示。我...... 我很抱歉开了这个糟糕的玩笑。

</aside>

Otherwise, if we don't find a `(`, we continue to compile the expression as a
super access like we did before and emit an `OP_GET_SUPER`.
否则，如果没有找到`(`，则继续像前面那样将表达式编译为一个超类访问，并发出一条`OP_GET_SUPER`指令。

Drifting down the compilation pipeline, our first stop is a new instruction.
沿着编译流水线向下，我们的第一站是一条新指令。

^code super-invoke-op (1 before, 1 after)

And just past that, its disassembler support.
在那之后，是它的反汇编器支持。

^code disassemble-super-invoke (1 before, 1 after)

A super invocation instruction has the same set of operands as `OP_INVOKE`, so
we reuse the same helper to disassemble it. Finally, the pipeline dumps us into
the interpreter.
超类调用指令具有与`OP_INVOKE`相同的操作数集，因此我们复用同一个辅助函数对其反汇编。最后，流水线将我们带到解释器中。

^code interpret-super-invoke (2 before, 1 after)

This handful of code is basically our implementation of `OP_INVOKE` mixed
together with a dash of `OP_GET_SUPER`. There are some differences in how the
stack is organized, though. With an unoptimized super call, the superclass is
popped and replaced by the ObjBoundMethod for the resolved function *before* the
arguments to the call are executed. This ensures that by the time the `OP_CALL`
is executed, the bound method is *under* the argument list, where the runtime
expects it to be for a closure call.
这一小段代码基本上是`OP_INVOKE`的实现，其中混杂了一点`OP_GET_SUPER`。不过，在堆栈的组织方式上有些不同。在未优化的超类调用中，超类会被弹出，并在调用的*参数*被执行之前替换为被解析函数的ObjBoundMethod。这确保了在`OP_CALL`执行时，已绑定方法在参数列表*之下*，也就是运行时期望闭包调用所在的位置。

With our optimized instructions, things are shuffled a bit:
在我们优化的指令中，事情有点被打乱：

<img src="image/superclasses/super-invoke.png" class="wide" alt="The series of bytecode instructions for calling super.finish() using OP_SUPER_INVOKE." />

Now resolving the superclass method is part of the *invocation*, so the
arguments need to already be on the stack at the point that we look up the
method. This means the superclass object is on top of the arguments.
现在，解析超类方法是执行的一部分，因此当我们查找方法时，参数需要已经在栈上。这意味着超类对象位于参数之上。

Aside from that, the behavior is roughly the same as an `OP_GET_SUPER` followed
by an `OP_CALL`. First, we pull out the method name and argument count operands.
Then we pop the superclass off the top of the stack so that we can look up the
method in its method table. This conveniently leaves the stack set up just right
for a method call.
除此之外，其行为与`OP_GET_SUPER`后跟`OP_CALL`大致相同。首先，我们取出方法名和参数数量两个操作数。然后我们从栈顶弹出超类，这样我们就可以在它的方法表中查找方法。这方便地将堆栈设置为适合方法调用的状态。

We pass the superclass, method name, and argument count to our existing
`invokeFromClass()` function. That function looks up the given method on the
given class and attempts to create a call to it with the given arity. If a
method could not be found, it returns `false`, and we bail out of the
interpreter. Otherwise, `invokeFromClass()` pushes a new CallFrame onto the call
stack for the method's closure. That invalidates the interpreter's cached
CallFrame pointer, so we refresh `frame`.
我们将超类、方法名和参数数量传递给现有的`invokeFromClass()`函数。该函数在给定的类上查找给定的方法，并尝试用给定的元数创建一个对它的调用。如果找不到某个方法，它就返回false，并退出解释器。否则，`invokeFromClass()`将一个新的CallFrame压入方法闭包的调用栈上。这会使解释器缓存的CallFrame指针失效，所以我们也要刷新`frame`。

## 一个完整的虚拟机

Take a look back at what we've created. By my count, we wrote around 2,500 lines
of fairly clean, straightforward C. That little program contains a complete
implementation of the -- quite high-level! -- Lox language, with a whole
precedence table full of expression types and a suite of control flow
statements. We implemented variables, functions, closures, classes, fields,
methods, and inheritance.
回顾一下我们创造了什么。根据我的计算，我们编写了大约2500行相当干净、简洁的C语言代码。这个小程序中包含了对Lox语言（相当高级）的完整实现，它有一个满是表达式类型的优先级表和一套控制流语句。我们实现了变量、函数、闭包、类、字段、方法和继承。

Even more impressive, our implementation is portable to any platform with a C
compiler, and is fast enough for real-world production use. We have a
single-pass bytecode compiler, a tight virtual machine interpreter for our
internal instruction set, compact object representations, a stack for storing
variables without heap allocation, and a precise garbage collector.
更令人印象深刻的是，我们的实现可以移植到任何带有C编译器的平台上，而且速度快到足以在实际生产中使用。我们有一个单遍字节码编译器，一个用于内部指令集的严格虚拟机解释器，紧凑的对象表示，一个用于存储变量而不需要堆分配的栈，以及一个精确的垃圾回收器。

If you go out and start poking around in the implementations of Lua, Python, or
Ruby, you will be surprised by how much of it now looks familiar to you. You
have seriously leveled up your knowledge of how programming languages work,
which in turn gives you a deeper understanding of programming itself. It's like
you used to be a race car driver, and now you can pop the hood and repair the
engine too.
如果你开始研究Lua、Python或Ruby的实现，你会惊讶于它们现在看起来有多熟悉。你已经真正提高了关于编程语言工作方式的知识水平，这反过来又使你对编程本身有了更深的理解。这就像你以前是个赛车手，现在你可以打开引擎盖，修改发动机了。

You can stop here if you like. The two implementations of Lox you have are
complete and full featured. You built the car and can drive it wherever you want
now. But if you are looking to have more fun tuning and tweaking for even
greater performance out on the track, there is one more chapter. We don't add
any new capabilities, but we roll in a couple of classic optimizations to
squeeze even more perf out. If that sounds fun, [keep reading][opt]...
如果你愿意，可以在这里停下来。你拥有的两个Lox实现是完整的、功能齐全的。你造了这俩车，现在可以把它开到你想去的地方。但是，如果你想获得更多改装与调整的乐趣，以期在赛道上获得更佳的性能，还有一个章节。我们没有增加任何新的功能，但我们推出了几个经典的优化，以挤压出更多的性能。如果这听起来很有趣，请继续读下去……

[opt]: optimization.html

<div class="challenges">

## Challenges

1.  A tenet of object-oriented programming is that a class should ensure new
    objects are in a valid state. In Lox, that means defining an initializer
    that populates the instance's fields. Inheritance complicates invariants
    because the instance must be in a valid state according to all of the
    classes in the object's inheritance chain.
  面向对象编程的一个原则是，类应该确保新对象处于有效状态。在Lox中，这意味着要定义一个填充实例字段的初始化器。继承使不变性复杂化，因为对于对象继承链中的所有类，实例必须处于有效状态。

    The easy part is remembering to call `super.init()` in each subclass's
    `init()` method. The harder part is fields. There is nothing preventing two
    classes in the inheritance chain from accidentally claiming the same field
    name. When this happens, they will step on each other's fields and possibly
    leave you with an instance in a broken state.
    简单的部分是记住在每个子类的`init()`方法中调用`super.init()`。比较难的部分是字段。没有什么方法可以防止继承链中的两个类意外地声明相同的字段名。当这种情况发生时，它们会互相干扰彼此的字段，并可能让你的实例处于崩溃状态。

    If Lox was your language, how would you address this, if at all? If you
    would change the language, implement your change.
    如果Lox是你的语言，你会如何解决这个问题？如果你想改变语言，请实现你的更改。

2.  Our copy-down inheritance optimization is valid only because Lox does not
    permit you to modify a class's methods after its declaration. This means we
    don't have to worry about the copied methods in the subclass getting out of
    sync with later changes to the superclass.
    我们的向下复制继承优化之所以有效，仅仅是因为Lox不允许在类声明之后修改它的方法。这意味着我们不必担心子类中复制的方法与后面对超类的修改不同步。

    Other languages, like Ruby, *do* allow classes to be modified after the
    fact. How do implementations of languages like that support class
    modification while keeping method resolution efficient?
    其它语言，如Ruby，确实允许在事后修改类。像这样的语言实现如何支持类的修改，同时保持方法解析的效率呢？

3.  In the [jlox chapter on inheritance][inheritance], we had a challenge to
    implement the BETA language's approach to method overriding. Solve the
    challenge again, but this time in clox. Here's the description of the
    previous challenge:
    在jlox关于继承的章节中，我们有一个习题，是实现BETA语言的方法重写。再次解决这个习题，但这次是在clox中。下面是对之前习题的描述：

    In Lox, as in most other object-oriented languages, when looking up a
    method, we start at the bottom of the class hierarchy and work our way up --
    a subclass's method is preferred over a superclass's. In order to get to the
    superclass method from within an overriding method, you use `super`.
    在Lox中，和其它大多数面向对象的语言一样，当查找一个方法时，我们从类层次结构的底部开始，然后向上查找 -- 子类的方法优于超类的方法。要想在子类方法中访问超类方法，可以使用`super`。

    The language [BETA][] takes the [opposite approach][inner]. When you call a
    method, it starts at the *top* of the class hierarchy and works *down*. A
    superclass method wins over a subclass method. In order to get to the
    subclass method, the superclass method can call `inner`, which is sort of
    like the inverse of `super`. It chains to the next method down the
    hierarchy.
    [BETA](https://beta.cs.au.dk/)语言则采取了[相反的方法](http://journal.stuffwithstuff.com/2012/12/19/the-impoliteness-of-overriding-methods/)。当你调用某个方法时，它从类层次结构的顶部开始向下运行。超类方法优于子类方法。要想访问子类方法，超类方法中可以调用`inner()`，这有点像是`super`的反义词。它会链接到层次结构中的下一个方法。

    The superclass method controls when and where the subclass is allowed to
    refine its behavior. If the superclass method doesn't call `inner` at all,
    then the subclass has no way of overriding or modifying the superclass's
    behavior.
    超类方法控制着子类何时何地被允许完善其行为。如果超类方法根本不调用`inner`，那么子类就没有办法覆写或修改超类的行为。

    Take out Lox's current overriding and `super` behavior, and replace it with
    BETA's semantics. In short:
    去掉Lox中当前的覆写和`super`行为，替换为BETA的语义。简而言之：

    *   When calling a method on a class, the method *highest* on the
        class's inheritance chain takes precedence.
        当调用某个类中的方法时，该类继承链上最高的方法优先。

    *   Inside the body of a method, a call to `inner` looks for a method with
        the same name in the nearest subclass along the inheritance chain
        between the class containing the `inner` and the class of `this`. If
        there is no matching method, the `inner` call does nothing.
        在方法体内部，对`inner`的调用，会沿着包含`inner`的类和`this`的类之间的继承链，在最近的子类中查找同名的方法。如果没有匹配的方法，`inner`调用就什么也不做。

    For example:
    举例来说：

    ```lox
    class Doughnut {
      cook() {
        print "Fry until golden brown.";
        inner();
        print "Place in a nice box.";
      }
    }

    class BostonCream < Doughnut {
      cook() {
        print "Pipe full of custard and coat with chocolate.";
      }
    }

    BostonCream().cook();
    ```

    This should print:
    这里应该打印：

    ```text
    Fry until golden brown.
    Pipe full of custard and coat with chocolate.
    Place in a nice box.
    ```

    Since clox is about not just implementing Lox, but doing so with good
    performance, this time around try to solve the challenge with an eye towards
    efficiency.
    因为clox不仅仅是实现Lox，而是要以良好的性能来实现，所以这次要尝试以效率为导向来解决这个问题。

[inheritance]: inheritance.html
[inner]: http://journal.stuffwithstuff.com/2012/12/19/the-impoliteness-of-overriding-methods/
[beta]: https://beta.cs.au.dk/

</div>


