> When you are a Bear of Very Little Brain, and you Think of Things, you find
> sometimes that a Thing which seemed very Thingish inside you is quite
> different when it gets out into the open and has other people looking at it.
>
> 你要是一只脑子很小的熊，当你想事情的时候，你会发现，有时在你心里看起来很像回事的事情，当它展示出来，让别人看着它的时候，就完全不同了。
>
> <cite>A. A. Milne, <em>Winnie-the-Pooh</em></cite>
> <cite>A. A. 米尔恩, <em>小熊维尼</em></cite>

The past few chapters were huge, packed full of complex techniques and pages of
code. In this chapter, there's only one new concept to learn and a scattering of
straightforward code. You've earned a respite.
前面的几章篇幅很长，充满了复杂的技术和一页又一页的代码。在本章中，只需要学习一个新概念和一些简单的代码。你获得了喘息的机会。

Lox is <span name="unityped">dynamically</span> typed. A single variable can
hold a Boolean, number, or string at different points in time. At least, that's
the idea. Right now, in clox, all values are numbers. By the end of the chapter,
it will also support Booleans and `nil`. While those aren't super interesting,
they force us to figure out how our value representation can dynamically handle
different types.
Lox是动态类型的。一个变量可以在不同的时间点持有布尔值、数字或字符串。至少，我们的想法是如此。现在，在clox中，所有的值都是数字。到本章结束时，它还将支持布尔值和`nil`。虽然这些不是特别有趣，但它们迫使我们弄清楚值表示如何动态地处理不同类型。

<aside name="unityped">

There is a third category next to statically typed and dynamically typed:
**unityped**. In that paradigm, all variables have a single type, usually a
machine register integer. Unityped languages aren't common today, but some
Forths and BCPL, the language that inspired C, worked like this.

As of this moment, clox is unityped.

在静态类型和动态类型之外，还有第三类：单一类型（**unityped**）。在这种范式中，所有的变量都是一个类型，通常是一个机器寄存器整数。单一类型的语言在今天并不常见，但一些Forth派生语言和BCPL（启发了C的语言）是这样工作的。

从这一刻起，clox是单一类型的。

</aside>

## 带标签的联合体

The nice thing about working in C is that we can build our data structures from
the raw bits up. The bad thing is that we *have* to do that. C doesn't give you
much for free at compile time and even less at runtime. As far as C is
concerned, the universe is an undifferentiated array of bytes. It's up to us to
decide how many of those bytes to use and what they mean.
使用C语言工作的好处是，我们可以从最基础的比特位开始构建数据结构。坏处是，我们必须这样做。C语言在编译时并没有提供多少免费的东西，在运行时就更少了。对C语言来说，宇宙是一个无差别的字节数组。由我们来决定使用多少个字节以及它们的含义。

In order to choose a value representation, we need to answer two key questions:
为了选择一种值的表示形式，我们需要先回答两个关键问题：

1.  **How do we represent the type of a value?** If you try to, say, multiply a
    number by `true`, we need to detect that error at runtime and report it. In
    order to do that, we need to be able to tell what a value's type is.

    **我们如何表示一个值的类型？** 比如说，如果你将一个数字乘以`true`，我们需要在运行时检测到这个错误并报告它。为此，我们需要知道值的类型是什么？

2.  **How do we store the value itself?** We need to not only be able to tell
    that three is a number, but that it's different from the number four. I
    know, seems obvious, right? But we're operating at a level where it's good
    to spell these things out.

    **我们如何存储该值本身？** 我们不仅要能分辨出3是一个数字，还要能分辨出它与4是不同的。我知道，这是显而易见的对吧？但是在我们所讨论的层面，最好把这些事情说清楚。

Since we're not just designing this language but building it ourselves, when
answering these two questions we also have to keep in mind the implementer's
eternal quest: to do it *efficiently*.
因为我们不仅仅是设计这门语言，还要自己构建它，所以在回答这两个问题时，我们还必须牢记实现者们永恒的追求：*高效*地完成它。

Language hackers over the years have come up with a variety of clever ways to
pack the above information into as few bits as possible. For now, we'll start
with the simplest, classic solution: a **tagged union**. A value contains two
parts: a type "tag", and a payload for the actual value. To store the value's
type, we define an enum for each kind of value the VM supports.
多年来，语言黑客们想出了各种巧妙的方法，将上述信息打包成尽可能少的比特。现在，我们将从最简单、最经典的解决方案开始：**带标签的联合体**。一个值包含两个部分：一个类型“标签”，和一个实际值的有效载荷。为了存储值的类型，我们要为虚拟机支持的每一种值定义一个枚举。

^code value-type (2 before, 1 after)

<aside name="user-types">

The cases here cover each kind of value that has *built-in support in the VM*.
When we get to adding classes to the language, each class the user defines
doesn't need its own entry in this enum. As far as the VM is concerned, every
instance of a class is the same type: "instance".

In other words, this is the VM's notion of "type", not the user's.

这个案例中涵盖了*虚拟机中内置支持*的每一种值。等到我们在语言中添加类时，用户定义的每个类并不需要在这个枚举中添加对应的条目。对于虚拟机而言，一个类的每个实例都是相同的类型：“instance”。

换句话说，这是虚拟机中的“类型”概念，而不是用户的。

</aside>

For now, we have only a couple of cases, but this will grow as we add strings,
functions, and classes to clox. In addition to the type, we also need to store
the data for the value -- the `double` for a number, `true` or `false` for a
Boolean. We could define a struct with fields for each possible type.
现在，我们只有这几种情况，但随着我们向clox中添加字符串、函数和类，这里也会越来越多。除了类型之外，我们还需要存储值的数据 -- 数字是`double`值，Boolean是`true`或`false`。我们可以定义一个结构体，其中包含每种可能的类型所对应的字段。

<img src="image/types-of-values/struct.png" alt="A struct with two fields laid next to each other in memory." />

But this is a waste of memory. A value can't simultaneously be both a number and
a Boolean. So at any point in time, only one of those fields will be used. C
lets you optimize this by defining a <span name="sum">union</span>. A union
looks like a struct except that all of its fields overlap in memory.
但这是对内存的一种浪费。一个值不可能同时是数字和布尔值。所以在任何时候，这些字段中只有一个会被使用。C语言中允许定义联合体来优化这一点。联合体看起来很像是结构体，区别在于其中的所有字段在内存中是重叠的。

<aside name="sum">

If you're familiar with a language in the ML family, structs and unions in C
roughly mirror the difference between product and sum types, between tuples
and algebraic data types.

如果您熟悉 ML 系列语言，那么 C 语言中的结构体和联合体大致反映了乘积与总和、元组与代数数据类型之间的区别。

</aside>

<img src="image/types-of-values/union.png" alt="A union with two fields overlapping in memory." />

The size of a union is the size of its largest field. Since the fields all reuse
the same bits, you have to be very careful when working with them. If you store
data using one field and then access it using <span
name="reinterpret">another</span>, you will reinterpret what the underlying bits
mean.
联合体的大小就是其最大字段的大小。由于这些字段都复用相同的比特位，你在使用它们时必须要非常小心。如果你使用一个字段存储数据，然后用另一个字段访问数据，那你需要重新解释底层比特位的含义。

<aside name="reinterpret">

Using a union to interpret bits as different types is the quintessence of C. It
opens up a number of clever optimizations and lets you slice and dice each byte
of memory in ways that memory-safe languages disallow. But it is also wildly
unsafe and will happily saw your fingers off if you don't watch out.

使用联合体将比特位解释为不同类型是C语言的精髓。它提供了许多巧妙的优化，让你能够以内存安全型语言中不允许的方式对内存中的每个字节进行切分。但它也是非常不安全的，如果你不小心，它就可能会锯掉你的手指。

</aside>

As the name "tagged union" implies, our new value representation combines these
two parts into a single struct.
顾名思义，“带标签的联合体”说明，我们新的值表示形式中将这两部分合并成一个结构体。

^code value (2 before, 2 after)

There's a field for the type tag, and then a second field containing the union
of all of the underlying values. On a 64-bit machine with a typical C compiler,
the layout looks like this:
有一个字段用作类型标签，然后是第二个字段，一个包含所有底层值的联合体。在使用典型的C语言编译器的64位机器上，布局看起来如下：

<aside name="as">

A smart language hacker gave me the idea to use "as" for the name of the union
field because it reads nicely, almost like a cast, when you pull the various
values out.

一个聪明的语言黑客给了我一个想法，把“as”作为联合体字段名称，因为当你取出各种值时，读起来感觉很好，就像是强制转换一样。

</aside>

<img src="image/types-of-values/value.png" alt="The full value struct, with the type and as fields next to each other in memory." />

The four-byte type tag comes first, then the union. Most architectures prefer
values be aligned to their size. Since the union field contains an eight-byte
double, the compiler adds four bytes of <span name="pad">padding</span> after
the type field to keep that double on the nearest eight-byte boundary. That
means we're effectively spending eight bytes on the type tag, which only needs
to represent a number between zero and three. We could stuff the enum in a
smaller size, but all that would do is increase the padding.
首先是4字节的类型标签，然后是联合体。大多数体系结构都喜欢将值与它们的字长对齐。由于联合体字段中包含一个8字节的double值，所以编译器在类型字段后添加了4个字节的填充，以使该double值保持在最近的8字节边界上。这意味着我们实际在类型标签上花费了8个字节，而它只需要表示0到3之间的数字。我们可以把枚举放在一个占位更少的变量中，但这样做只会增加填充量。

<aside name="pad">

We could move the tag field *after* the union, but that doesn't help much
either. Whenever we create an array of Values -- which is where most of our
memory usage for Values will be -- the C compiler will insert that same padding
*between* each Value to keep the doubles aligned.

我们可以把标签字段移动到联合体字段之后，但这也没有多大帮助。每当我们创建一个Value数组时（这也是我们对Value的主要内存使用），C编译器都会在每个数值 *之间* 插入相同的填充，以保持双精度数对齐。

</aside>

So our Values are 16 bytes, which seems a little large. We'll improve it
[later][optimization]. In the meantime, they're still small enough to store on
the C stack and pass around by value. Lox's semantics allow that because the
only types we support so far are **immutable**. If we pass a copy of a Value
containing the number three to some function, we don't need to worry about the
caller seeing modifications to the value. You can't "modify" three. It's three
forever.
所以我们的Value是16个字节，这似乎有点大。我们稍后会改进它。同时，它们也足够小，可以存储在C语言的堆栈中，并按值传递。Lox的语义允许这样做，因为到目前为止我们只支持**不可变**类型。如果我们把一个包含数字3的Value的副本传递给某个函数，我们不需要担心调用者会看到对该值的修改。你不能“修改”3，它永远都是3。

[optimization]: optimization.html

## Lox的值和C语言的值

That's our new value representation, but we aren't done. Right now, the rest of
clox assumes Value is an alias for `double`. We have code that does a straight C
cast from one to the other. That code is all broken now. So sad.
这就是我们新的值表示形式，但是我们还没有做完。现在，clox的其余部分都假定了Value是`double`的别名。我们有一些代码是直接用C语言将一个值转换为另一个值。这些代码现在都被破坏了，好伤心。

With our new representation, a Value can *contain* a double, but it's not
*equivalent* to it. There is a mandatory conversion step to get from one to the
other. We need to go through the code and insert those conversions to get clox
working again.
在我们新的表示形式中，Value可以*包含*一个double值，但它并不等同于double类型。有一个强制性的转换步骤可以实现从一个值到另一个值的转换。我们需要遍历代码并插入这些转换步骤，以使clox重新工作。

We'll implement these conversions as a handful of macros, one for each type and
operation. First, to promote a native C value to a clox Value:
我们会用少量的宏来实现这些转换，每个宏对应一个类型和操作。首先，将原生的C值转换为clox Value：

^code value-macros (1 before, 2 after)

Each one of these takes a C value of the appropriate type and produces a Value
that has the correct type tag and contains the underlying value. This hoists
statically typed values up into clox's dynamically typed universe. In order to
*do* anything with a Value, though, we need to unpack it and get the C value
back out.
其中每个宏都接收一个适当类型的C值，并生成一个Value，其具有正确类型标签并包含底层的值。这就把静态类型的值提升到了clox的动态类型的世界。但是为了能对Value做任何操作，我们需要将其拆包并取出对应的C值。

^code as-macros (1 before, 2 after)

<aside name="as-null">

There's no `AS_NIL` macro because there is only one `nil` value, so a Value with
type `VAL_NIL` doesn't carry any extra data.

没有`AS_NIL`宏，因为只有一个`nil`值，所以一个类型为`VAL_NIL`的Value不会携带任何额外的数据。

</aside>

<span name="as-null">These</span> macros go in the opposite direction. Given a
Value of the right type, they unwrap it and return the corresponding raw C
value. The "right type" part is important! These macros directly access the
union fields. If we were to do something like:
这些宏的作用是反方向的。给定一个正确类型的Value，它们会将其解包并返回对应的原始C值。“正确类型”很重要！这些宏会直接访问联合体字段。如果我们要这样做：

```c
Value value = BOOL_VAL(true);
double number = AS_NUMBER(value);
```

Then we may open a smoldering portal to the Shadow Realm. It's not safe to use
any of the `AS_` macros unless we know the Value contains the appropriate type.
To that end, we define a last few macros to check a Value's type.
那我们可能会打开一个通往暗影王国的阴燃之门。除非我们知道Value包含适当的类型，否则使用任何的`AS_`宏都是不安全的。为此，我们定义最后几个宏来检查Value的类型。

^code is-macros (1 before, 2 after)

<span name="universe">These</span> macros return `true` if the Value has that
type. Any time we call one of the `AS_` macros, we need to guard it behind a
call to one of these first. With these eight macros, we can now safely shuttle
data between Lox's dynamic world and C's static one.
如果Value具有对应类型，这些宏会返回`true`。每当我们调用一个`AS_`宏时，我们都需要保证首先调用了这些宏。有了这8个宏，我们现在可以安全地在Lox的动态世界和C的静态世界之间传输数据了。

<aside name="universe">

<img src="image/types-of-values/universe.png" alt="The earthly C firmament with the Lox heavens above." />

The `_VAL` macros lift a C value into the heavens. The `AS_` macros bring it
back down.
`_VAL` 宏将 C 值带到到天上。`AS_` 宏则把它拉回来。

</aside>

## 动态类型数字

We've got our value representation and the tools to convert to and from it. All
that's left to get clox running again is to grind through the code and fix every
place where data moves across that boundary. This is one of those sections of
the book that isn't exactly mind-blowing, but I promised I'd show you every
single line of code, so here we are.
我们已经有了值的表示形式和转换的工具。要想让clox重新运行起来，剩下的工作就是仔细检查代码，修复每个数据跨边界传递的地方。这是本书中不太让人兴奋的章节之一，但我保证会给你展示每一行代码，所以我们开始吧。

The first values we create are the constants generated when we compile number
literals. After we convert the lexeme to a C double, we simply wrap it in a
Value before storing it in the constant table.
我们创建的第一个值是在编译数值字面量时生成的常量。在我们将词素转换为C语言的double之后，我们简单地将其包装在一个Value中，然后再存储到常量表中。

^code const-number-val (1 before, 1 after)

Over in the runtime, we have a function to print values.
在运行时，我们有一个函数来打印值。

^code print-number-value (1 before, 1 after)

Right before we send the Value to `printf()`, we unwrap it and extract the
double value. We'll revisit this function shortly to add the other types, but
let's get our existing code working first.
在我们将Value发送给`printf()`之前，我们将其拆装并提取出double值。我们很快会重新回顾这个函数并添加其它类型，但是我们先让现有的代码工作起来。

### 一元取负与运行时错误

The next simplest operation is unary negation. It pops a value off the stack,
negates it, and pushes the result. Now that we have other types of values, we
can't assume the operand is a number anymore. The user could just as well do:
接下来最简单的操作是一元取负。它会从栈中弹出一个值，对其取负，并将结果压入栈。现在我们有了其它类型的值，我们不能再假设操作数是一个数字。用户也可以这样做：

```lox
print -false; // Uh...
```

We need to handle that gracefully, which means it's time for *runtime errors*.
Before performing an operation that requires a certain type, we need to make
sure the Value *is* that type.
我们需要优雅地处理这个问题，这意味着是时候讨论运行时错误了。在执行需要特定类型的操作之前，我们需要确保Value是该类型。

For unary negation, the check looks like this:
对于一元取负来说，检查是这样的：

^code op-negate (1 before, 1 after)

First, we check to see if the Value on top of the stack is a number. If it's
not, we report the runtime error and <span name="halt">stop</span> the
interpreter. Otherwise, we keep going. Only after this validation do we unwrap
the operand, negate it, wrap the result and push it.
首先，我们检查栈顶的Value是否是一个数字。如果不是，则报告运行时错误并停止解释器。否则，我们就继续运行。只有在验证之后，我们才会拆装操作数，取负，将结果封装并压入栈。

<aside name="halt">

Lox's approach to error-handling is rather... *spare*. All errors are fatal and
immediately halt the interpreter. There's no way for user code to recover from
an error. If Lox were a real language, this is one of the first things I would
remedy.

Lox的错误处理方法是相当...... *简朴* 的。所有的错误都是致命的，会立即停止解释器。用户代码无法从错误中恢复。如果Lox是一种真正的语言，这是我首先要补救的事情之一。

</aside>

To access the Value, we use a new little function.
为了访问Value，我们使用一个新的小函数。

^code peek

It returns a Value from the stack but doesn't <span name="peek">pop</span> it.
The `distance` argument is how far down from the top of the stack to look: zero
is the top, one is one slot down, etc.
它从堆栈中返回一个Value，但是并不弹出它。`distance`参数是指要从堆栈顶部向下看多远：0是栈顶，1是下一个槽，以此类推。

<aside name="peek">

Why not just pop the operand and then validate it? We could do that. In later
chapters, it will be important to leave operands on the stack to ensure the
garbage collector can find them if a collection is triggered in the middle of
the operation. I do the same thing here mostly out of habit.

为什么不直接弹出操作数然后验证它呢？我们可以这么做。在后面的章节中，将操作数留在栈上是很重要的，可以确保在运行过程中触发垃圾收集时，垃圾收集器能够找到它们。我在这里做了同样的事情，主要是出于习惯。

</aside>

We report the runtime error using a new function that we'll get a lot of mileage
out of over the remainder of the book.
我们使用一个新函数来报告运行时错误，在本书的剩余部分，我们会从中得到很多的好处。

^code runtime-error

You've certainly *called* variadic functions -- ones that take a varying number
of arguments -- in C before: `printf()` is one. But you may not have *defined*
your own. This book isn't a C <span name="tutorial">tutorial</span>, so I'll
skim over it here, but basically the `...` and `va_list` stuff let us pass an
arbitrary number of arguments to `runtimeError()`. It forwards those on to
`vfprintf()`, which is the flavor of `printf()` that takes an explicit
`va_list`.
你以前肯定在C语言中调用过变参函数 -- 接受不同数量参数的函数：`printf()`就是其中之一。但你可能还没*定义*过自己的变参函数。这本书不是C语言教程，所以我在这里略过了，但是基本上是`...`和`va_list`让我们可以向`runtimeError()`传递任意数量的参数。它将这些参数转发给`vfprintf()`，这是`printf()`的一个变体，需要一个显式地`va_list`。

<aside name="tutorial">

If you are looking for a C tutorial, I love *[The C Programming Language][kr]*,
usually called "K&R" in honor of its authors. It's not entirely up to date, but
the quality of the writing more than makes up for it.

如果你在寻找一个C教程，我喜欢[C程序设计语言][kr]，通常被称为“K&R”，以纪念它的作者。它并不完全是最新的，但是写作质量足以弥补这一点。

[kr]: https://www.cs.princeton.edu/~bwk/cbook.html

</aside>

Callers can pass a format string to `runtimeError()` followed by a number of
arguments, just like they can when calling `printf()` directly. `runtimeError()`
then formats and prints those arguments. We won't take advantage of that in this
chapter, but later chapters will produce formatted runtime error messages that
contain other data.
调用者可以向`runtimeError()`传入一个格式化字符串，后跟一些参数，就像他们直接调用`printf()`一样。然后`runtimeError()`格式化并打印这些参数。在本章中我们不会利用这一点，但后面的章节中将生成包含其它数据的格式化运行时错误信息。

After we show the hopefully helpful error message, we tell the user which <span
name="stack">line</span> of their code was being executed when the error
occurred. Since we left the tokens behind in the compiler, we look up the line
in the debug information compiled into the chunk. If our compiler did its job
right, that corresponds to the line of source code that the bytecode was
compiled from.
在显示了希望有帮助的错误信息之后，我们还会告诉用户，当错误发生时正在执行代码中的哪一行。因为我们在编译器中留下了标识，所以我们可以从编译到字节码块中的调试信息中查找行号。如果我们的编译器正确完成了它的工作，就能对应到字节码被编译出来的那一行源代码。

We look into the chunk's debug line array using the current bytecode instruction
index *minus one*. That's because the interpreter advances past each instruction
before executing it. So, at the point that we call `runtimeError()`, the failed
instruction is the previous one.
我们使用当前字节码指令索引减1来查看字节码块的调试行数组。这是因为解释器在之前每条指令之前都会向前推进。所以，当我们调用 `runtimeError()`，失败的指令就是前一条。

<aside name="stack">

Just showing the immediate line where the error occurred doesn't provide much
context. Better would be a full stack trace. But we don't even have functions to
call yet, so there is no call stack to trace.

仅仅显示发生错误的那一行并不能提供太多的上下文信息。最后是提供完整的堆栈跟踪，但是我们目前甚至还没有函数调用，所以也没有调用堆栈可以跟踪。

</aside>

In order to use `va_list` and the macros for working with it, we need to bring
in a standard header.
为了使用`va_list`和相关的宏，我们需要引入一个标准头文件。

^code include-stdarg (1 after)

With this, our VM can not only do the right thing when we negate numbers (like
it used to before we broke it), but it also gracefully handles erroneous
attempts to negate other types (which we don't have yet, but still).
有了它，我们的虚拟机不仅可以在对数字取负时正确执行（原本就会这样做），而且还可以优雅地处理对其它类型取负的错误尝试（目前还没有，但仍然存在）。

### 二元数字运算符

We have our runtime error machinery in place now, so fixing the binary operators
is easier even though they're more complex. We support four binary operators
today: `+`, `-`, `*`, and `/`. The only difference between them is which
underlying C operator they use. To minimize redundant code between the four
operators, we wrapped up the commonality in a big preprocessor macro that takes
the operator token as a parameter.
我们现在已经有了运行时错误机制，所以修复二元运算符更容易，尽管它们更复杂。现在我们支持四种二元运算符：`+`、`-`、`*`和`/`。它们之间唯一的区别就是使用的是哪种底层C运算符。为了尽量减少这四个运算符之间的冗余代码，我们将它们的共性封装在一个大的预处理宏中，该宏以运算符标识作为参数。

That macro seemed like overkill a [few chapters ago][], but we get the benefit
from it today. It lets us add the necessary type checking and conversions in one
place.
这个宏在前几章中似乎是多余的，但现在我们却从中受益。它让我们可以在某个地方添加必要的类型检查和转换。

[few chapters ago]: a-virtual-machine.html#binary-operators

^code binary-op (1 before, 2 after)

Yeah, I realize that's a monster of a macro. It's not what I'd normally consider
good C practice, but let's roll with it. The changes are similar to what we did
for unary negate. First, we check that the two operands are both numbers. If
either isn't, we report a runtime error and yank the ejection seat lever.
是的，我知道这是一个巨大的宏。这不是我通常认为的好的C语言实践，但我们还是用它吧。这些调整与我们对一元取负所做的相似。首先，我们检查两个操作数是否都是数字。如果其中一个不是，我们就报告一个运行时错误，并拉下弹射座椅手柄。

If the operands are fine, we pop them both and unwrap them. Then we apply the
given operator, wrap the result, and push it back on the stack. Note that we
don't wrap the result by directly using `NUMBER_VAL()`. Instead, the wrapper to
use is passed in as a macro <span name="macro">parameter</span>. For our
existing arithmetic operators, the result is a number, so we pass in the
`NUMBER_VAL` macro.
如果操作数都没有问题，我们就把它们都弹出栈并进行拆装。然后我们应用给定的运算符，包装结果并将其压回栈中。注意，我们没有直接使用`NUMBER_VAL()`来包装结果。相反，我们要使用的包装器是作为宏参数传入的。对于我们现有的数字运算符来说，结果是一个数字，所以我们传入`NUMBER_VAL`宏。

<aside name="macro">

Did you know you can pass macros as parameters to macros? Now you do!

你知道可以将宏作为参数传递给宏吗？现在你知道了！

</aside>

^code op-arithmetic (1 before, 1 after)

Soon, I'll show you why we made the wrapping macro an argument.
很快，我就会告诉你为什么我们要将包装宏作为参数。

## 两个新类型

All of our existing clox code is back in working order. Finally, it's time to
add some new types. We've got a running numeric calculator that now does a
number of pointless paranoid runtime type checks. We can represent other types
internally, but there's no way for a user's program to ever create a Value of
one of those types.
我们现有的所有clox代码都恢复正常工作了。最后，是时候添加一些新类型了。我们有一个正在运行的数字计算器，它现在做了一些毫无意义的偏执的运行时类型检查。我们可以在内部表示其它类型，但用户的程序无法创建这些类型的Value。

Not until now, that is. We'll start by adding compiler support for the three new
literals: `true`, `false`, and `nil`. They're all pretty simple, so we'll do all
three in a single batch.
现在还不能。首先，我们向编译器添加对三个新字面量的支持：`true`、`false`、`nil`。它们都很简单，所以我们一次性完成这三个。

With number literals, we had to deal with the fact that there are billions of
possible numeric values. We attended to that by storing the literal's value in
the chunk's constant table and emitting a bytecode instruction that simply
loaded that constant. We could do the same thing for the new types. We'd store,
say, `true`, in the constant table, and use an `OP_CONSTANT` to read it out.
对于数字字面量，我们要面对这样一个事实：有数十亿个可能的数字值。为此，我们将字面量的值保存在字节码块的常量表中，并生成一个加载该常量的字节码指令。我们可以对这些新类型做同样的事。我们在常量表中存储值，比如`true`，并使用`OP_CONSTANT`来读取它。

But given that there are literally (heh) only three possible values we need to
worry about with these new types, it's gratuitous -- and <span
name="small">slow!</span> -- to waste a two-byte instruction and a constant
table entry on them. Instead, we'll define three dedicated instructions to push
each of these literals on the stack.
但是考虑到这些新类型实际上只有三种可能的值，这样做是没有必要的 -- 而且速度很慢！ -- 浪费了一个两字节的指令和常量表中的一个项。相反，我们会定义三个专用指令来将这些字面量压入栈中。

<aside name="small" class="bottom">

I'm not kidding about dedicated operations for certain constant values being
faster. A bytecode VM spends much of its execution time reading and decoding
instructions. The fewer, simpler instructions you need for a given piece of
behavior, the faster it goes. Short instructions dedicated to common operations
are a classic optimization.
我不是在开玩笑，对于某些常量值的专用操作会更快。字节码虚拟机的大部分执行时间都花在读取和解码指令上。对于一个特定的行为，你需要的指令越少、越简单，它就越快。专用于常见操作的短指令是一种典型的优化。

For example, the Java bytecode instruction set has dedicated instructions for
loading 0.0, 1.0, 2.0, and the integer values from -1 through 5. (This ends up
being a vestigial optimization given that most mature JVMs now JIT-compile the
bytecode to machine code before execution anyway.)
例如，Java字节码指令集中有专门的指令用于加载0.0、1.0、2.0以及从-1到5之间的整数。（考虑到大多数成熟的JVM在执行前都会对字节码进行JIT编译，这最终成为了一种残留的优化）

</aside>

^code literal-ops (1 before, 1 after)

Our scanner already treats `true`, `false`, and `nil` as keywords, so we can
skip right to the parser. With our table-based Pratt parser, we just need to
slot parser functions into the rows associated with those keyword token types.
We'll use the same function in all three slots. Here:
我们的扫描器已经将`true`、`false`和`nil`视为关键字，所以我们可以直接调到解析器。对于我们这个基于表格的Pratt解析器，只需要将解析器函数插入到与这些关键字标识类型相对应的行中。我们会在三个槽中使用相同的函数。这里：

^code table-false (1 before, 1 after)

Here:
这里:

^code table-true (1 before, 1 after)

And here:
还有这里:

^code table-nil (1 before, 1 after)

When the parser encounters `false`, `nil`, or `true`, in prefix position, it
calls this new parser function:
当解析器在前缀位置遇到`false`、`nil`或 `true`时，它会调用这个新的解析器函数：

^code parse-literal

Since `parsePrecedence()` has already consumed the keyword token, all we need to
do is output the proper instruction. We <span name="switch">figure</span> that
out based on the type of token we parsed. Our front end can now compile Boolean
and nil literals to bytecode. Moving down the execution pipeline, we reach the
interpreter.
因为`parsePrecedence()`已经消耗了关键字标识，我们需要做的就是输出正确的指令。我们根据解析出的标识的类型来确定指令。我们的前端现在可以将布尔值和`nil`字面量编译为字节码。沿着执行管道向下移动，我们就到了解释器。

<aside name="switch">

We could have used separate parser functions for each literal and saved
ourselves a switch but that felt needlessly verbose to me. I think it's mostly a
matter of taste.

我们本可以为每个字面意思使用单独的解析器函数，这样就省去了一个开关，但我觉得这样做太冗长了。我认为这主要是品味问题。

</aside>

^code interpret-literals (5 before, 1 after)

This is pretty self-explanatory. Each instruction summons the appropriate value
and pushes it onto the stack. We shouldn't forget our disassembler either.
这一点是不言而喻的。每条指令都会召唤出相应的值并将其压入堆栈。我们也不能忘记反汇编程序。

^code disassemble-literals (2 before, 1 after)

With this in place, we can run this Earth-shattering program:
有了这些，我们就可以运行这个惊天动地的程序：

```lox
true
```

Except that when the interpreter tries to print the result, it blows up. We need
to extend `printValue()` to handle the new types too:
只是当解释器试图打印结果时，就崩溃了。我们也需要扩展`printValue()`来处理新类型：

^code print-value (1 before, 1 after)

There we go! Now we have some new types. They just aren't very useful yet. Aside
from the literals, you can't really *do* anything with them. It will be a while
before `nil` comes into play, but we can start putting Booleans to work in the
logical operators.
我们继续！现在我们有了一些新的类型，只是它们目前还不是很有用。除了字面量之外，你无法真正对其做任何事。还需要一段时间`nil`才会发挥作用，但我们可以先让布尔值在逻辑运算符中发挥作用。

### 逻辑非和false

The simplest logical operator is our old exclamatory friend unary not.
最简单的逻辑运算符是我们充满感叹意味的老朋友一元取非。

```lox
print !true; // "false"
```

This new operation gets a new instruction.
这个新操作会有一条新指令。

^code not-op (1 before, 1 after)

We can reuse the `unary()` parser function we wrote for unary negation to
compile a not expression. We just need to slot it into the parsing table.
我们可以重用为一元取负所写的解析函数来编译一个逻辑非表达式。我们只需要将其插入到解析表格中。

^code table-not (1 before, 1 after)

Because I knew we were going to do this, the `unary()` function already has a
switch on the token type to figure out which bytecode instruction to output. We
merely add another case.
因为我之前已知道我们要这样做，`unary()`函数已经有了关于标识类型的switch语句，来判断要输出哪个字节码指令。我们只需要增加一个分支即可。

^code compile-not (1 before, 3 after)

That's it for the front end. Let's head over to the VM and conjure this
instruction into life.
前端就这样了。让我们去虚拟机那里，并将这个指令变成现实。

^code op-not (1 before, 1 after)

Like our previous unary operator, it pops the one operand, performs the
operation, and pushes the result. And, as we did there, we have to worry about
dynamic typing. Taking the logical not of `true` is easy, but there's nothing
preventing an unruly programmer from writing something like this:
跟之前的一元运算符一样，它会弹出一个操作数，执行操作，并将结果压入栈中。正如我们所做的那样，我们必须考虑动态类型。对`true`进行逻辑取非很容易，但没什么能阻止一个不守规矩的程序员写出这样的东西：

```lox
print !nil;
```

For unary minus, we made it an error to negate anything that isn't a <span
name="negate">number</span>. But Lox, like most scripting languages, is more
permissive when it comes to `!` and other contexts where a Boolean is expected.
The rule for how other types are handled is called "falsiness", and we implement
it here:
对于一元取负，我们把对任何非数字的东西进行取负当作一个错误。但是Lox，像大多数脚本语言一样，在涉及到`!`和其它期望出现布尔值的情况下，是比较宽容的。处理其它类型的规则被称为“falsiness”，我们在这里实现它：

<aside name="negate">

Now I can't help but try to figure out what it would mean to negate other types
of values. `nil` is probably its own negation, sort of like a weird pseudo-zero.
Negating a string could, uh, reverse it?

现在我忍不住想弄清楚，对其它类型的值取反意味着什么。`nil`可能有自己的非值，有点像一个奇怪的伪零。对字符串取反可以，emmm，反转？

</aside>

^code is-falsey

Lox follows Ruby in that `nil` and `false` are falsey and every other value
behaves like `true`. We've got a new instruction we can generate, so we also
need to be able to *un*generate it in the disassembler.
Lox遵循Ruby的规定，`nil`和`false`是假的，其它的值都表现为`true`。我们已经有了一条可以生成的新指令，所以我们也需要能够在反汇编程序中反生成它。

^code disassemble-not (2 before, 1 after)

### 相等与比较运算符

That wasn't too bad. Let's keep the momentum going and knock out the equality
and comparison operators too: `==`, `!=`, `<`, `>`, `<=`, and `>=`. That covers
all of the operators that return Boolean results except the logical operators
`and` and `or`. Since those need to short-circuit (basically do a little
control flow) we aren't ready for them yet.
还不算太糟。让我们继续保持这种势头，搞定相等与比较运算符： `==`，`!=`，`<`，`>`，`<=`和`>=`。这涵盖了所有会返回布尔值的运算符，除了逻辑运算符`and`和`or`。因为这些运算符需要短路计算（基本上是做一个小小的控制流），我们还没准备好。

Here are the new instructions for those operators:
下面是这些运算符对应的新指令：

^code comparison-ops (1 before, 1 after)

Wait, only three? What about `!=`, `<=`, and `>=`? We could create instructions
for those too. Honestly, the VM would execute faster if we did, so we *should*
do that if the goal is performance.
等一下，只有三个？那`!=`、`<=`和`>=`呢？我们也可以为它们创建指令。老实说，如果我们这样做，虚拟机的执行速度会更快。所以如果我们的目标是追求性能，那就*应该*这样做。

But my main goal is to teach you about bytecode compilers. I want you to start
internalizing the idea that the bytecode instructions don't need to closely
follow the user's source code. The VM has total freedom to use whatever
instruction set and code sequences it wants as long as they have the right
user-visible behavior.
但我的主要目标是教你有关字节码编译器的知识。我想要你开始内化一个想法：字节码指令不需要紧跟用户的源代码。虚拟机可以完全自由地使用它想要的任何指令集和代码序列，只要它们有正确的用户可见的行为。

The expression `a != b` has the same semantics as `!(a == b)`, so the compiler
is free to compile the former as if it were the latter. Instead of a dedicated
`OP_NOT_EQUAL` instruction, it can output an `OP_EQUAL` followed by an `OP_NOT`.
Likewise, `a <= b` is the <span name="same">same</span> as `!(a > b)` and `a >=
b` is `!(a < b)`. Thus, we only need three new instructions.
表达式`a!=b`与`!(a==b)`具有相同的语义，所以编译器可以自由地编译前者，就好像它是后者一样。它可以输出一条`OP_EQUAL`指令，之后是一条`OP_NOT`，而不是一条专用的`OP_NOT_EQUAL`指令。同样地，`a<=b`与`!(a>b)`相同，而`a>=b`与`!(a<b)`相同，所以我们只需要三条新指令。

<aside name="same" class="bottom">

*Is* `a <= b` always the same as `!(a > b)`? According to [IEEE 754][], all
comparison operators return false when an operand is NaN. That means `NaN <= 1`
is false and `NaN > 1` is also false. But our desugaring assumes the latter is
always the negation of the former.

`a<=b`总是与`!(a>b)`相同吗？根据IEEE 754标准，当操作数为NaN时，所有的比较运算符都返回假。这意味着`NaN <= 1`是假的，`NaN > 1`也是假的。但我们的脱糖操作假定了后者是前者的非值。

For the book, we won't get hung up on this, but these kinds of details will
matter in your real language implementations.

在本书中，我们不必纠结于此，但这些细节在你的真正的语言实现中会很重要。

[ieee 754]: https://en.wikipedia.org/wiki/IEEE_754

</aside>

Over in the parser, though, we do have six new operators to slot into the parse
table. We use the same `binary()` parser function from before. Here's the row
for `!=`:
不过，在解析器中，我们确实有6个新的操作符要加入到解析表中。我们使用与之前相同的`binary()`解析函数。下面是`!=`对应的行：

^code table-equal (1 before, 1 after)

The remaining five operators are a little farther down in the table.
其余五个运算符在表的最下方。

^code table-comparisons (1 before, 1 after)

Inside `binary()` we already have a switch to generate the right bytecode for
each token type. We add cases for the six new operators.
在`binary()`中，我们已经有了一个switch语句，为每种标识类型生成正确的字节码。我们为这六个新运算符添加分支。

^code comparison-operators (1 before, 1 after)

The `==`, `<`, and `>` operators output a single instruction. The others output
a pair of instructions, one to evalute the inverse operation, and then an
`OP_NOT` to flip the result. Six operators for the price of three instructions!
`==`、`<`和`>` 运算符输出单个指令。其它运算符则输出一对指令，一条用于计算逆运算，然后用`OP_NOT`来反转结果。仅仅使用三种指令就表达出了六种运算符的效果！

That means over in the VM, our job is simpler. Equality is the most general
operation.
这意味着在虚拟机中，我们的工作更简单了。相等是最普遍的操作。

^code interpret-equal (1 before, 1 after)

You can evaluate `==` on any pair of objects, even objects of different types.
There's enough complexity that it makes sense to shunt that logic over to a
separate function. That function always returns a C `bool`, so we can safely
wrap the result in a `BOOL_VAL`. The function relates to Values, so it lives
over in the "value" module.
你可以对任意一对对象执行`==`，即使这些对象是不同类型的。这有足够的复杂性，所以有必要把这个逻辑分流到一个单独的函数中。这个函数会一个C语言的`bool`值，所以我们可以安全地把结果包装在一个`BOLL_VAL`中。这个函数与Value有关，所以它位于“value”模块中。

^code values-equal-h (2 before, 1 after)

And here's the implementation:
下面是实现：

^code values-equal

First, we check the types. If the Values have <span
name="equal">different</span> types, they are definitely not equal. Otherwise,
we unwrap the two Values and compare them directly.
首先，我们检查类型。如果两个Value的类型不同，它们肯定不相等。否则，我们就把这两个Value拆装并直接进行比较。

<aside name="equal">

Some languages have "implicit conversions" where values of different types may
be considered equal if one can be converted to the other's type. For example,
the number 0 is equivalent to the string "0" in JavaScript. This looseness was a
large enough source of pain that JS added a separate "strict equality" operator,
`===`.
有些语言支持“隐式转换”，如果某个类型的值可以转换为另一个类型，那么这两种类型的值就可以被认为是相等的。举例来说，在JavaScript中，数字0等同于字符串“0”。这种松散性导致JS增加了一个单独的“严格相等”运算符，`===`。

PHP considers the strings "1" and "01" to be equivalent because both can be
converted to equivalent numbers, though the ultimate reason is because PHP was
designed by a Lovecraftian eldritch god to destroy the mind.
PHP认为字符串 "1" 和 "01" 是等价的，因为两者都可以转换成等价的数字，但是最根本的原因在于PHP是由Lovecraftian的邪神设计的，目的是摧毁人类心智。

Most dynamically typed languages that have separate integer and floating-point
number types consider values of different number types equal if the numeric
values are the same (so, say, 1.0 is equal to 1), though even that seemingly
innocuous convenience can bite the unwary.
大多数具有单独的整数和浮点数类型的动态类型语言认为，如果数值相同，则不同数字类型的值是相等的（所以说，1.0等于1），但即便是这种看似无害的便利，如果一不小心也会让人吃不消。

</aside>

For each value type, we have a separate case that handles comparing the value
itself. Given how similar the cases are, you might wonder why we can't simply
`memcmp()` the two Value structs and be done with it. The problem is that
because of padding and different-sized union fields, a Value contains unused
bits. C gives no guarantee about what is in those, so it's possible that two
equal Values actually differ in memory that isn't used.
对于每一种值类型，我们都有一个单独的case分支来处理值本身的比较。考虑到这些分支的相似性，你可能会想，为什么我们不能简单地对两个Value结构体进行`memcmp()`，然后就可以了。问题在于，因为填充以及联合体字段的大小不同，Value中会包含无用的比特位。C语言不能保证这些值是什么，所以两个相同的Value在未使用的内存中可能是完全不同的。

<img src="image/types-of-values/memcmp.png" alt="The memory respresentations of two equal values that differ in unused bytes." />

(You wouldn't believe how much pain I went through before learning this fact.)
(你无法想象在了解这个事实之前我经历了多少痛苦。)

Anyway, as we add more types to clox, this function will grow new cases. For
now, these three are sufficient. The other comparison operators are easier since
they work only on numbers.
总之，随着我们向clox中添加更多的类型，这个函数也会增加更多的case分支。就目前而言，这三个已经足够了。其它的比较运算符更简单，因为它们只处理数字。

^code interpret-comparison (3 before, 1 after)

We already extended the `BINARY_OP` macro to handle operators that return
non-numeric types. Now we get to use that. We pass in `BOOL_VAL` since the
result value type is Boolean. Otherwise, it's no different from plus or minus.
我们已经扩展了`BINARY_OP`宏，来处理返回非数字类型的运算符。现在我们要用到它了。因为结果值类型是布尔型，所以我们传入`BOOL_VAL`。除此之外，这与加减运算没有区别。

As always, the coda to today's aria is disassembling the new instructions.
与往常一样，今天的咏叹调的尾声是对新指令进行反汇编。

^code disassemble-comparison (2 before, 1 after)

With that, our numeric calculator has become something closer to a general
expression evaluator. Fire up clox and type in:
这样一来，我们的数字计算器就变得更接近于一个通用的表达式求值器。启动clox并输入：

```lox
!(5 - 4 > 3 * 2 == !nil)
```

OK, I'll admit that's maybe not the most *useful* expression, but we're making
progress. We have one missing built-in type with its own literal form: strings.
Those are much more complex because strings can vary in size. That tiny
difference turns out to have implications so large that we give strings [their
very own chapter][strings].
好吧，我承认这可能不是最*有用的*表达式，但我们正在取得进展。我们还缺少一种自带字面量形式的内置类型：字符串。它们要复杂得多，因为字符串的大小可以不同。这个微小的差异会产生巨大的影响，以至于我们给字符串单独开了一章。

[strings]: strings.html

<div class="challenges">

## Challenges

1. We could reduce our binary operators even further than we did here. Which
   other instructions can you eliminate, and how would the compiler cope with
   their absence?
   我们可以进一步简化二元操作符。还有哪些指令可以取消，编译器如何应对这些指令的缺失？

2. Conversely, we can improve the speed of our bytecode VM by adding more
   specific instructions that correspond to higher-level operations. What
   instructions would you define to speed up the kind of user code we added
   support for in this chapter?
   相反，我们可以通过添加更多对应于高级操作的专用指令来提高字节码虚拟机的速度。你会定义什么指令来加速我们在本章中添加的那种用户代码？

</div>


