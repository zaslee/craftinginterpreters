> I wanna, I wanna,<br />
> I wanna, I wanna,<br />
> I wanna be trash.<br />
>
> <cite>The Whip, &ldquo;Trash&rdquo;</cite>
> <cite>The Whip乐队的歌曲, &ldquo;Trash&rdquo;</cite>

We say Lox is a "high-level" language because it frees programmers from worrying
about details irrelevant to the problem they're solving. The user becomes an
executive, giving the machine abstract goals and letting the lowly computer
figure out how to get there.
我们说Lox是一种“高级”语言，因为它使得程序员不必担心那些与他们要解决的问题无关的细节。用户变成了执行者，给机器设定抽象的目标，让底层的计算机自己想办法实现目标。

Dynamic memory allocation is a perfect candidate for automation. It's necessary
for a working program, tedious to do by hand, and yet still error-prone. The
inevitable mistakes can be catastrophic, leading to crashes, memory corruption,
or security violations. It's the kind of risky-yet-boring work that machines
excel at over humans.
动态内存分配是自动化的最佳选择。这是一个基本可用的程序所必需的，手动操作很繁琐，而且很容易出错。不可避免的错误可能是灾难性的，会导致崩溃、内存损坏或安全漏洞。机器比人类更擅长这种既有风险又无聊的工作。

This is why Lox is a **managed language**, which means that the language
implementation manages memory allocation and freeing on the user's behalf. When
a user performs an operation that requires some dynamic memory, the VM
automatically allocates it. The programmer never worries about deallocating
anything. The machine ensures any memory the program is using sticks around as
long as needed.
这就是为什么Lox是一种**托管语言**，这意味着语言实现会代表用户管理内存的分配与释放。当用户执行某个需要动态内存的操作时，虚拟机会自动分配。程序员不用担心任何释放内存的事情。机器会确保程序使用的任意内存会在需要的时候存在。

Lox provides the illusion that the computer has an infinite amount of memory.
Users can allocate and allocate and allocate and never once think about where
all these bytes are coming from. Of course, computers do not yet *have* infinite
memory. So the way managed languages maintain this illusion is by going behind
the programmer's back and reclaiming memory that the program no longer needs.
The component that does this is called a **garbage <span
name="recycle">collector</span>**.
Lox提供了一种计算机拥有无限内存的错觉。用户可以不停地分配、分配、再分配，而不用考虑这些内存是从哪里来的。当然，计算机还没有无限的内存。因此，托管语言维持这种错觉的方式是背着程序员，回收程序不再需要的内存。实现这一点的组件被称为**垃圾回收器**。

<aside name="recycle">

Recycling would really be a better metaphor for this. The GC doesn't *throw
away* the memory, it reclaims it to be reused for new data. But managed
languages are older than Earth Day, so the inventors went with the analogy they
knew.
用 "循环利用" 来比喻这一点确实更恰当。GC 不会 *丢弃* 内存，而是回收内存，重新用于新数据。但托管语言的历史比地球日还要悠久，所以发明者们使用了他们熟悉的比喻。

<img src="image/garbage-collection/recycle.png" class="above" alt="A recycle bin full of bits." />

</aside>

## 可达性

This raises a surprisingly difficult question: how does a VM tell what memory is
*not* needed? Memory is only needed if it is read in the future, but short of
having a time machine, how can an implementation tell what code the program
*will* execute and which data it *will* use? Spoiler alert: VMs cannot travel
into the future. Instead, the language makes a <span
name="conservative">conservative</span> approximation: it considers a piece of
memory to still be in use if it *could possibly* be read in the future.
这就引出了一个非常困难的问题：虚拟机如何分辨哪些内存是不需要的？内存只有在未来被读取时才需要，但是如果没有时间机器，语言如何知道程序将执行哪些代码，使用哪些数据？剧透警告：虚拟机不能穿越到未来。相反，语言做了一个保守的估计：如果一块内存在未来*有可能*被读取，就认为它仍然在使用。

<aside name="conservative">

I'm using "conservative" in the general sense. There is such a thing as a
"conservative garbage collector" which means something more specific. All
garbage collectors are "conservative" in that they keep memory alive if it
*could* be accessed, instead of having a Magic 8-Ball that lets them more
precisely know what data *will* be accessed.
我用的是一般意义上的“保守”。有一种“保守的垃圾回收器”，它的意思更具体。所有的垃圾回收器都是“保守的”，如果内存*可以*被访问，它们就保持内存存活，而不是有一个魔力8号球，可以让它们更精确地知道哪些数据将被访问。

A **conservative GC** is a special kind of collector that considers any piece of
memory to be a pointer if the value in there looks like it could be an address.
This is in contrast to a **precise GC** -- which is what we'll implement -- that
knows exactly which words in memory are pointers and which store other kinds of
values like numbers or strings.
**保守的GC**是一种特殊的回收器，它认为如果任何一块内存中的值看起来可能是地址，那它就是一个指针。这与我们将要实现的**精确的GC**相反，精确GC知道内存中哪些数据是指针，哪些存储的是数字或字符串等其它类型的值。

</aside>

That sounds *too* conservative. Couldn't *any* bit of memory potentially be
read? Actually, no, at least not in a memory-safe language like Lox. Here's an
example:
这听起来*太*保守了。难道不是内存中的*任何*比特都可能被读取吗？事实上，不是，至少在Lox这样内存安全的语言中不是。下面是一个例子：

```lox
var a = "first value";
a = "updated";
// GC here.
print a;
```

Say we run the GC after the assignment has completed on the second line. The
string "first value" is still sitting in memory, but there is no way for the
user's program to ever get to it. Once `a` got reassigned, the program lost any
reference to that string. We can safely free it. A value is **reachable** if
there is some way for a user program to reference it. Otherwise, like the string
"first value" here, it is **unreachable**.
假设我们在完成第二行的赋值之后运行GC。字符串“first value”仍然在内存中，但是用户的程序没有办法访访问它。一旦`a`被重新赋值，程序就失去了对该字符串的任何引用，我们可以安全地释放它。如果用户程序可以通过某种方式引用一个值，这个值就是可达的。否则，就像这里的字符串“first value”一样，它是不可达的。

Many values can be directly accessed by the VM. Take a look at:
许多值可以被虚拟机直接访问。请看：

```lox
var global = "string";
{
  var local = "another";
  print global + local;
}
```

Pause the program right after the two strings have been concatenated but before
the `print` statement has executed. The VM can reach `"string"` by looking
through the global variable table and finding the entry for `global`. It can
find `"another"` by walking the value stack and hitting the slot for the local
variable `local`. It can even find the concatenated string `"stringanother"`
since that temporary value is also sitting on the VM's stack at the point when
we paused our program.
在两个字符串连接之后但是`print`语句执行之前暂停程序。虚拟机可以通过查看全局变量表，并查找`global`条目到达`"string"`。它可以通过遍历值栈，并找到局部变量`local`的栈槽来找到`"another"`。它甚至也可以找到连接后的字符串`"stringanother"`，因为在我们暂停程序的时候，这个临时值也在虚拟机的栈中。

All of these values are called **roots**. A root is any object that the VM can
reach directly without going through a reference in some other object. Most
roots are global variables or on the stack, but as we'll see, there are a couple
of other places the VM stores references to objects that it can find.
所有这些值都被称为**根**。根是虚拟机可以无需通过其它对象的引用而直接到达的任何对象。大多数根是全局变量或在栈上，但正如我们将看到的，还有其它一些地方，虚拟机会在其中存储它可以找到的对象的引用。

Other values can be found by going through a reference inside another value.
<span name="class">Fields</span> on instances of classes are the most obvious
case, but we don't have those yet. Even without those, our VM still has indirect
references. Consider:
其它值可以通过另一个值中的引用来找到。类实例中的字段是最明显的情况，但我们目前还没有类。即使没有这些，我们的虚拟机中仍然存在间接引用。考虑一下：

<aside name="class">

We'll get there [soon][classes], though!
不过，我们很快就会到达[那里][classes]！

[classes]: classes-and-instances.html

</aside>

```lox
fun makeClosure() {
  var a = "data";

  fun f() { print a; }
  return f;
}

{
  var closure = makeClosure();
  // GC here.
  closure();
}
```

Say we pause the program on the marked line and run the garbage collector. When
the collector is done and the program resumes, it will call the closure, which
will in turn print `"data"`. So the collector needs to *not* free that string.
But here's what the stack looks like when we pause the program:
假设我们在标记的行上暂停并运行垃圾回收器。当回收器完成、程序恢复时，它将会调用闭包，然后输出`"data"`。所以回收器需要*不释放*那个字符串。但当我们暂停程序时，栈是这样的：

<img src="image/garbage-collection/stack.png" alt="The stack, containing only the script and closure." />

The `"data"` string is nowhere on it. It has already been hoisted off the stack
and moved into the closed upvalue that the closure uses. The closure itself is
on the stack. But to get to the string, we need to trace through the closure and
its upvalue array. Since it *is* possible for the user's program to do that, all
of these indirectly accessible objects are also considered reachable.
`"data"`字符串并不在上面。它已经被从栈中提取出来，并移动到闭包所使用的关闭上值中。闭包本身在栈上。但是要得到字符串，我们需要跟踪闭包及其上值数组。因为用户的程序可能会这样做，所有这些可以间接访问的对象也被认为是可达的。

<img src="image/garbage-collection/reachable.png" class="wide" alt="All of the referenced objects from the closure, and the path to the 'data' string from the stack." />

This gives us an inductive definition of reachability:
这给了我们一个关于可达性的归纳定义：

*   All roots are reachable.
*   Any object referred to from a reachable object is itself reachable.


* 所有根都是可达的。
* 任何被某个可达对象引用的对象本身是可达的。

These are the values that are still "live" and need to stay in memory. Any value
that *doesn't* meet this definition is fair game for the collector to reap.
That recursive pair of rules hints at a recursive algorithm we can use to free
up unneeded memory:
这些是仍然“存活”、需要留在内存中的值。任何*不符合*这个定义的值，对于回收器来说都是可收割的猎物。这一对递归规则暗示了我们可以用一个递归算法来释放不需要的内存：

1.  Starting with the roots, traverse through object references to find the
    full set of reachable objects.
2.  Free all objects *not* in that set.


1. 从根开始，遍历对象引用，找到可达对象的完整集合。
2. 释放不在集合中的所有对象。

Many <span name="handbook">different</span> garbage collection algorithms are in
use today, but they all roughly follow that same structure. Some may interleave
the steps or mix them, but the two fundamental operations are there. They mostly
differ in *how* they perform each step.
如今有很多不同的垃圾回收算法，但是它们都大致遵循相同的结构。有些算法可能会将这些步骤进行交叉或混合，但这两个基本操作是存在的。不同算法的区别在于*如何*执行每个步骤。

<aside name="handbook">

If you want to explore other GC algorithms,
[*The Garbage Collection Handbook*][gc book] (Jones, et al.) is the canonical
reference. For a large book on such a deep, narrow topic, it is quite enjoyable
to read. Or perhaps I have a strange idea of fun.
如果你想探索其它的GC算法，[《垃圾回收手册》][gc book]是一本经典的参考书。对于这样一本深入浅出的大部头来说，它的阅读体验是相当愉快的。也许我对乐趣有种奇怪的看法。

[gc book]: http://gchandbook.org/

</aside>

## 标记-清除 垃圾回收

The first managed language was Lisp, the second "high-level" language to be
invented, right after Fortran. John McCarthy considered using manual memory
management or reference counting, but <span
name="procrastination">eventually</span> settled on (and coined) garbage
collection -- once the program was out of memory, it would go back and find
unused storage it could reclaim.
第一门托管语言是Lisp，它是继Fortran之后发明的第二种“高级”语言。John McCarthy曾考虑使用手动内存管理或引用计数，但最终还是选择（并创造了）垃圾回收 -- 一旦程序的内存用完了，它就会回去寻找可以回收的未使用的存储空间。

<aside name="procrastination">

In John McCarthy's "History of Lisp", he notes: "Once we decided on garbage
collection, its actual implementation could be postponed, because only toy
examples were being done." Our choice to procrastinate adding the GC to clox
follows in the footsteps of giants.
在John McCarthy的《Lisp的历史》中，他指出：“一旦我们决定进行垃圾回收，它的实际实现就会被推迟，因为当时只做了玩具性的例子。”我们选择推迟在clox中加入GC，是追随了巨人的脚步。

</aside>

He designed the very first, simplest garbage collection algorithm, called
**mark-and-sweep** or just **mark-sweep**. Its description fits in three short
paragraphs in the initial paper on Lisp. Despite its age and simplicity, the
same fundamental algorithm underlies many modern memory managers. Some corners
of CS seem to be timeless.
他设计了最早的、最简单的垃圾回收算法，被称为**标记并清除（mark-and-sweep）**，或者就叫**标记清除（mark-sweep）**。在最初的Lisp论文中，关于它的描述只有短短的三段。尽管它年代久远且简单，但许多现代内存管理器都使用了相同的基本算法。CS中的一些角落似乎是永恒的。

As the name implies, mark-sweep works in two phases:
顾名思义，标记-清除分两个阶段工作：

*   **Marking:** We start with the roots and traverse or <span
    name="trace">*trace*</span> through all of the objects those roots refer to.
    This is a classic graph traversal of all of the reachable objects. Each time
    we visit an object, we *mark* it in some way. (Implementations differ in how
    they record the mark.)
*   **Sweeping:** Once the mark phase completes, every reachable object
    in the heap has been marked. That means any unmarked object is unreachable and
    ripe for reclamation. We go through all the unmarked objects and free each
    one.


*   **标记**：我们从根开始，遍历或跟踪这些根所引用的所有对象。这是对所有可达对象的经典图式遍历。每次我们访问一个对象时，我们都用某种方式来标记它。（不同的实现方式，记录标记的方法也不同）
*   **清除**：一旦标记阶段完成，堆中的每个可达对象都被标记了。这意味着任何未被标记的对象都是不可达的，可以被回收。我们遍历所有未被标记的对象，并逐个释放。

It looks something like this:
它看起来像是这样的：

<img src="image/garbage-collection/mark-sweep.png" class="wide" alt="Starting from a graph of objects, first the reachable ones are marked, the remaining are swept, and then only the reachable remain." />

<aside name="trace">

A **tracing garbage collector** is any algorithm that traces through the graph
of object references. This is in contrast with reference counting, which has a
different strategy for tracking the reachable objects.
**跟踪式垃圾回收器**是指任何通过对象引用图来追踪的算法。这与引用计数相反，后者使用不同的策略来追踪可达对象。

</aside>

That's what we're gonna implement. Whenever we decide it's time to reclaim some
bytes, we'll trace everything and mark all the reachable objects, free what
didn't get marked, and then resume the user's program.
这就是我们要实现的。每当我们决定是时候回收一些字节的时候，我们就会跟踪一切，并标记所有可达的对象，释放没有被标记的对象，然后恢复用户的程序。

### 垃圾回收

This entire chapter is about implementing this one <span
name="one">function</span>:
整个章节都是关于实现这一个函数的：

<aside name="one">

Of course, we'll end up adding a bunch of helper functions too.
当然，我们最终也会添加一些辅助函数。

</aside>

^code collect-garbage-h (1 before, 1 after)

We'll work our way up to a full implementation starting with this empty shell:
我们会从这个空壳开始逐步完整实现：

^code collect-garbage

The first question you might ask is, When does this function get called? It
turns out that's a subtle question that we'll spend some time on later in the
chapter. For now we'll sidestep the issue and build ourselves a handy diagnostic
tool in the process.
你可能会问的第一个问题是，这个函数在什么时候被调用？事实证明，这是一个微妙的问题，我们会在后面的章节中花些时间讨论。现在，我们先回避这个问题，并在这个过程中为自己构建一个方便的诊断工具。

^code define-stress-gc (1 before, 2 after)

We'll add an optional "stress test" mode for the garbage collector. When this
flag is defined, the GC runs as often as it possibly can. This is, obviously,
horrendous for performance. But it's great for flushing out memory management
bugs that occur only when a GC is triggered at just the right moment. If *every*
moment triggers a GC, you're likely to find those bugs.
我们将为垃圾回收器添加一个可选的“压力测试”模式。当定义这个标志后，GC就会尽可能频繁地运行。显然，这对性能来说是很糟糕的。但它对于清除内存管理bug很有帮助，这些bug只有在适当的时候触发GC时才会出现。如果每时每刻都触发GC，那你很可能会找到这些bug。

^code call-collect (1 before, 1 after)

Whenever we call `reallocate()` to acquire more memory, we force a collection to
run. The if check is because `reallocate()` is also called to free or shrink an
allocation. We don't want to trigger a GC for that -- in particular because the
GC itself will call `reallocate()` to free memory.
每当我们调用`reallocate()`来获取更多内存时，都会强制运行一次回收。这个`if`检查是因为，在释放或收缩分配的内存时也会调用`reallocate()`。我们不希望在这种时候触发GC -- 特别是因为GC本身也会调用`reallocate()`来释放内存。

Collecting right before <span name="demand">allocation</span> is the classic way
to wire a GC into a VM. You're already calling into the memory manager, so it's
an easy place to hook in the code. Also, allocation is the only time when you
really *need* some freed up memory so that you can reuse it. If you *don't* use
allocation to trigger a GC, you have to make sure every possible place in code
where you can loop and allocate memory also has a way to trigger the collector.
Otherwise, the VM can get into a starved state where it needs more memory but
never collects any.
在分配之前进行回收是将GC引入虚拟机的经典方式。你已经在调用内存管理器了，所以这是个很容易挂接代码的地方。另外，分配是唯一你真的*需要*一些释放出来的内存的时候，这样你就可以重新使用它。如果*不*使用分配来触发GC，则必须确保代码中每个可以循环和分配内存的地方都有触发回收器的方法。否则，虚拟机会进入饥饿状态，它需要更多的内存，但却没有回收到任何内存。

<aside name="demand">

More sophisticated collectors might run on a separate thread or be interleaved
periodically during program execution -- often at function call boundaries or
when a backward jump occurs.
更复杂的回收器可能运行在单独的线程上，或者在程序执行过程中定期交错运行 -- 通常是在函数调用边界处或发生后向跳转时。

</aside>

### 调试日志

While we're on the subject of diagnostics, let's put some more in. A real
challenge I've found with garbage collectors is that they are opaque. We've been
running lots of Lox programs just fine without any GC *at all* so far. Once we
add one, how do we tell if it's doing anything useful? Can we tell only if we
write programs that plow through acres of memory? How do we debug that?
既然我们在讨论诊断的问题，那我们再加入一些内容。我发现垃圾回收器的一个真正的挑战在于它们是不透明的。到目前为止，我们已经在没有任何GC的情况下运行了很多Lox程序。一旦我们添加了GC，我们如何知道它是否在做有用的事情？只有当我们编写的程序耗费了大量的内存时，我们才能知道吗？我们该如何调试呢？

An easy way to shine a light into the GC's inner workings is with some logging.
了解GC内部工作的一种简单方式是进行一些日志记录。

^code define-log-gc (1 before, 2 after)

When this is enabled, clox prints information to the console when it does
something with dynamic memory.
启用这个功能后，当clox使用动态内存执行某些操作时，会将信息打印到控制台。

We need a couple of includes.
我们需要一些引入。

^code debug-log-includes (1 before, 2 after)

We don't have a collector yet, but we can start putting in some of the logging
now. We'll want to know when a collection run starts.
我们还没有回收器，但我们现在可以开始添加一些日志记录。我们想要知道回收是在何时开始的。

^code log-before-collect (1 before, 1 after)

Eventually we will log some other operations during the collection, so we'll
also want to know when the show's over.
最终，我们会在回收过程中记录一些其它操作，因此我们也想知道回收什么时候结束。

^code log-after-collect (2 before, 1 after)

We don't have any code for the collector yet, but we do have functions for
allocating and freeing, so we can instrument those now.
我们还没有关于回收器的任何代码，但是我们有分配和释放的函数，所以我们现在可以对这些函数进行检测。

^code debug-log-allocate (1 before, 1 after)

And at the end of an object's lifespan:
在对象的生命周期结束时：

^code log-free-object (1 before, 1 after)

With these two flags, we should be able to see that we're making progress as we
work through the rest of the chapter.
有了这两个标志，我们应该能够看到我们在本章其余部分的学习中取得了进展。

## 标记 Root

Objects are scattered across the heap like stars in the inky night sky. A
reference from one object to another forms a connection, and these
constellations are the graph that the mark phase traverses. Marking begins at
the roots.
对象就像漆黑夜空中的星星一样散落在堆中。从一个对象到另一个对象的引用形成了一种连接，而这些星座就是标记阶段需要遍历的图。标记是从根开始的。

^code call-mark-roots (3 before, 2 after)

Most roots are local variables or temporaries sitting right in the VM's stack,
so we start by walking that.
大多数根是虚拟机栈中的局部变量或临时变量，因此我们从遍历栈开始：

^code mark-roots

To mark a Lox value, we use this new function:
为了标记Lox值，我们使用这个新函数：

^code mark-value-h (1 before, 1 after)

Its implementation is here:
它的实现在这里：

^code mark-value

Some Lox values -- numbers, Booleans, and `nil` -- are stored directly inline in
Value and require no heap allocation. The garbage collector doesn't need to
worry about them at all, so the first thing we do is ensure that the value is an
actual heap object. If so, the real work happens in this function:
一些Lox值（数字、布尔值和`nil`）直接内联存储在Value中，不需要堆分配。垃圾回收器根本不需要担心这些，因此我们要做的第一件事是确保值是一个真正的堆对象。如果是这样，真正的工作就发生在这个函数中：

^code mark-object-h (1 before, 1 after)

Which is defined here:
下面是定义：

^code mark-object

The `NULL` check is unnecessary when called from `markValue()`. A Lox Value that
is some kind of Obj type will always have a valid pointer. But later we will
call this function directly from other code, and in some of those places, the
object being pointed to is optional.
从`markValue()`中调用时，`NULL`检查是不必要的。某种Obj类型的Lox Value一定会有一个有效的指针。但稍后我们将从其它代码中直接调用这个函数，在其中一些地方，被指向的对象是可选的。

Assuming we do have a valid object, we mark it by setting a flag. That new field
lives in the Obj header struct all objects share.
假设我们确实有一个有效的对象，我们通过设置一个标志来标记它。这个新字段存在于所有对象共享的Obj头中。

^code is-marked-field (1 before, 1 after)

Every new object begins life unmarked because we haven't yet determined if it is
reachable or not.
每个新对象在开始时都没有标记，因为我们还不确定它是否可达。

^code init-is-marked (1 before, 2 after)

Before we go any farther, let's add some logging to `markObject()`.
在进一步讨论之前，我们先在`markObject()`中添加一些日志。

^code log-mark-object (2 before, 1 after)

This way we can see what the mark phase is doing. Marking the stack takes care
of local variables and temporaries. The other main source of roots are the
global variables.
这样我们就可以看到标记阶段在做什么。对栈进行标记可以处理局部变量和临时变量。另一个根的主要来源就是全局变量。

^code mark-globals (2 before, 1 after)

Those live in a hash table owned by the VM, so we'll declare another helper
function for marking all of the objects in a table.
它们位于VM拥有的一个哈希表中，因此我们会声明另一个辅助函数来标记表中的所有对象。

^code mark-table-h (2 before, 2 after)

We implement that in the "table" module here:
我们在“table”模块中实现它：

^code mark-table

Pretty straightforward. We walk the entry array. For each one, we mark its
value. We also mark the key strings for each entry since the GC manages those
strings too.
非常简单。我们遍历条目数组。对于每个条目，我们标记其值。我们还会标记每个条目的键字符串，因为GC也要管理这些字符串。

### 不明显的 Root

Those cover the roots that we typically think of -- the values that are
obviously reachable because they're stored in variables the user's program can
see. But the VM has a few of its own hidey-holes where it squirrels away
references to values that it directly accesses.
这些覆盖了我们通常认为的根 -- 那些明显可达的值，因为它们存储在用户程序可以看到的变量中。但是虚拟机也有一些自己的藏身之所，可以隐藏对直接访问的值的引用。

Most function call state lives in the value stack, but the VM maintains a
separate stack of CallFrames. Each CallFrame contains a pointer to the closure
being called. The VM uses those pointers to access constants and upvalues, so
those closures need to be kept around too.
大多数函数调用状态都存在于值栈中，但是虚拟机维护了一个单独的CallFrame栈。每个CallFrame都包含一个指向被调用闭包的指针。VM使用这些指针来访问常量和上值，所以这些闭包也需要被保留下来。

^code mark-closures (1 before, 2 after)

Speaking of upvalues, the open upvalue list is another set of values that the
VM can directly reach.
说到上值，开放上值列表是VM可以直接访问的另一组值。

^code mark-open-upvalues (3 before, 2 after)

Remember also that a collection can begin during *any* allocation. Those
allocations don't just happen while the user's program is running. The compiler
itself periodically grabs memory from the heap for literals and the constant
table. If the GC runs while we're in the middle of compiling, then any values
the compiler directly accesses need to be treated as roots too.
还要记住，回收可能会在*任何*分配期间开始。这些分配并不仅仅是在用户程序运行的时候发生。编译器本身会定期从堆中获取内存，用于存储字面量和常量表。如果GC在编译期间运行，那么编译器直接访问的任何值也需要被当作根来处理。

To keep the compiler module cleanly separated from the rest of the VM, we'll do
that in a separate function.
为了保持编译器模块与虚拟机的其它部分完全分离，我们在一个单独的函数中完成这一工作。

^code call-mark-compiler-roots (1 before, 1 after)

It's declared here:
它是在这里声明的：

^code mark-compiler-roots-h (1 before, 2 after)

Which means the "memory" module needs an include.
这意味着“memory”模块需要引入头文件。

^code memory-include-compiler (2 before, 1 after)

And the definition is over in the "compiler" module.
定义在“compiler”模块中。

^code mark-compiler-roots

Fortunately, the compiler doesn't have too many values that it hangs on to. The
only object it uses is the ObjFunction it is compiling into. Since function
declarations can nest, the compiler has a linked list of those and we walk the
whole list.
幸运的是，编译器并没有太多挂载的值。它唯一使用的对象是它正在编译的ObjFunction。由于函数声明可以嵌套，编译器有一个函数声明的链表，我们遍历整个列表。

Since the "compiler" module is calling `markObject()`, it also needs an include.
因为“compiler”模块会调用`markObject()`，也需要引入。

^code compiler-include-memory (1 before, 1 after)

Those are all the roots. After running this, every object that the VM -- runtime
and compiler -- can get to *without* going through some other object has its
mark bit set.
这些就是所有的根。运行这段程序后，虚拟机（运行时和编译器）无需通过其它对象就可以达到的每个对象，其标记位都被设置了。

## 跟踪对象引用

The next step in the marking process is tracing through the graph of references
between objects to find the indirectly reachable values. We don't have instances
with fields yet, so there aren't many objects that contain references, but we do
have <span name="some">some</span>. In particular, ObjClosure has the list of
ObjUpvalues it closes over as well as a reference to the raw ObjFunction that it
wraps. ObjFunction, in turn, has a constant table containing references to all
of the literals created in the function's body. This is enough to build a fairly
complex web of objects for our collector to crawl through.
标记过程的下一步是跟踪对象之间的引用图，找到间接可达的对象。我们现在还没有带有字段的实例，因此包含引用的对象不多，但确实有一些。特别的，ObjClosure拥有它所关闭的ObjUpvalue列表，以及它所包装的指向原始ObjFunction的引用。反过来，ObjFunction有一个常量表，包含函数体中创建的所有字面量的引用。这足以构建一个相当复杂的对象网络，供回收器爬取。

<aside name="some">

I slotted this chapter into the book right here specifically *because* we now
have closures which give us interesting objects for the garbage collector to
process.
我把这一章安排在这里，特别是因为我们现在有了闭包，它给我们提供了有趣的对象，让垃圾回收器来处理。

</aside>

Now it's time to implement that traversal. We can go breadth-first, depth-first,
or in some other order. Since we just need to find the *set* of all reachable
objects, the order we visit them <span name="dfs">mostly</span> doesn't matter.
现在是时候实现遍历了。我们可以按照广度优先、深度优先或其它顺序进行遍历。因为我们只需要找到所有可达对象的集合，所以访问它们的顺序几乎没有影响。

<aside name="dfs">

I say "mostly" because some garbage collectors move objects in the order that
they are visited, so traversal order determines which objects end up adjacent in
memory. That impacts performance because the CPU uses locality to determine
which memory to preload into the caches.
我说“几乎”是因为有些垃圾回收器是按照对象被访问的顺序来移动对象的，所以遍历顺序决定了哪些对象最终会在内存中相邻。这会影响性能，因为CPU使用位置来决定哪些内存要预加载到缓存中。

Even when traversal order does matter, it's not clear which order is *best*.
It's very difficult to determine which order objects will be used in in the
future, so it's hard for the GC to know which order will help performance.
即便在遍历顺序很重要的时候，也不清楚哪种顺序是最好的。很难确定对象在未来会以何种顺序被使用，因此GC很难知道哪种顺序有助于提高性能。

</aside>

### 三色抽象

As the collector wanders through the graph of objects, we need to make sure it
doesn't lose track of where it is or get stuck going in circles. This is
particularly a concern for advanced implementations like incremental GCs that
interleave marking with running pieces of the user's program. The collector
needs to be able to pause and then pick up where it left off later.
当回收器在对象图中漫游时，我们需要确保它不会失去对其位置的跟踪或者陷入循环。这对于像增量GC这样的高级实现来说尤其值得关注，因为增量GC将标记与用户程序的的运行部分交织在一起。回收器需要能够暂停，稍后在停止的地方重新开始。

To help us soft-brained humans reason about this complex process, VM hackers
came up with a metaphor called the <span name="color"></span>**tricolor
abstraction**. Each object has a conceptual "color" that tracks what state the
object is in, and what work is left to do.
为了帮助我们这些愚蠢的人类理解这个复杂的过程，虚拟机专家们想出了一个称为**三色抽象**的比喻。每个对象都有一个概念上的“颜色”，用于追踪对象处于什么状态，以及还需要做什么工作。

<aside name="color">

Advanced garbage collection algorithms often add other colors to the
abstraction. I've seen multiple shades of gray, and even purple in some designs.
My puce-chartreuse-fuchsia-malachite collector paper was, alas, not accepted for
publication.
高级的垃圾回收算法经常为这个抽象概念加入其它颜色。我见过多种深浅不一的灰色，甚至在一些设计中见过紫色。可惜的是，我的黄绿色-紫红色-孔雀石回收器论文没有被接受发表。

</aside>

*   **<img src="image/garbage-collection/white.png" alt="A white circle."
    class="dot" /> White:** At the beginning of a garbage collection, every
    object is white. This color means we have not reached or processed the
    object at all.
*   **<img src="image/garbage-collection/gray.png" alt="A gray circle."
    class="dot" /> Gray:** During marking, when we first reach an object, we
    darken it gray. This color means we know the object itself is reachable and
    should not be collected. But we have not yet traced *through* it to see what
    *other* objects it references. In graph algorithm terms, this is the
    *worklist* -- the set of objects we know about but haven't processed yet.
*   **<img src="image/garbage-collection/black.png" alt="A black circle."
    class="dot" /> Black:** When
    we take a gray object and mark all of the objects it references, we then
    turn the gray object black. This color means the mark phase is done
    processing that object.


*   **<img src="image/garbage-collection/white.png" alt="A white circle."
    class="dot" /> 白色:** 在垃圾回收的开始阶段，每个对象都是白色的。这种颜色意味着我们根本没有达到或处理该对象。
*   **<img src="image/garbage-collection/gray.png" alt="A gray circle."
    class="dot" /> 灰色:** 在标记过程中，当我们第一次达到某个对象时，我们将其染为灰色。这种颜色意味着我们知道这个对象本身是可达的，不应该被收集。但我们还没有*通过*它来跟踪它引用的*其它*对象。用图算法的术语来说，这就是*工作列表（worklist）* -- 我们知道但还没有被处理的对象集合。
*   **<img src="image/garbage-collection/black.png" alt="A black circle."
    class="dot" /> 黑色:** 当我们接受一个灰色对象，并将其引用的所有对象全部标记后，我们就把这个灰色对象变为黑色。这种颜色意味着标记阶段已经完成了对该对象的处理。

In terms of that abstraction, the marking process now looks like this:
从这个抽象的角度看，标记过程新增看起来是这样的：

1.  Start off with all objects white.
    开始时，所有对象都是白色的。

2.  Find all the roots and mark them gray.
    找到所有的根，将它们标记为灰色。

3.  Repeat as long as there are still gray objects:
    只要还存在灰色对象，就重复此过程：

    1.  Pick a gray object. Turn any white objects that the object mentions to
        gray.
        选择一个灰色对象。将该对象引用的所有白色对象标记为灰色。

    2.  Mark the original gray object black.
        将原来的灰色对象标记为黑色。

I find it helps to visualize this. You have a web of objects with references
between them. Initially, they are all little white dots. Off to the side are
some incoming edges from the VM that point to the roots. Those roots turn gray.
Then each gray object's siblings turn gray while the object itself turns black.
The full effect is a gray wavefront that passes through the graph, leaving a
field of reachable black objects behind it. Unreachable objects are not touched
by the wavefront and stay white.
我发现把它可视化很有帮助。你有一个对象网络，对象之间有引用。最初，它们都是小白点。旁边是一些虚拟机的传入边，这些边指向根。这些根变成了灰色。然后每个灰色对象的兄弟节点变成灰色，而该对象本身变成黑色。完整的效果是一个灰色波前穿过图，在它后面留下一个可达的黑色对象区域。不可达对象不会被波前触及，并保持白色。

<img src="image/garbage-collection/tricolor-trace.png" class="wide" alt="A gray wavefront working through a graph of nodes." />

At the <span name="invariant">end</span>, you're left with a sea of reached,
black objects sprinkled with islands of white objects that can be swept up and
freed. Once the unreachable objects are freed, the remaining objects -- all
black -- are reset to white for the next garbage collection cycle.
最后，你会看到一片可达的、黑色对象组成的海洋，其中点缀着可以清除和释放的白色对象组成的岛屿。一旦不可达的对象被释放，剩下的对象（全部为黑色）会被重置为白色，以便在下一个垃圾收集周期使用。

<aside name="invariant">

Note that at every step of this process no black node ever points to a white
node. This property is called the **tricolor invariant**. The traversal process
maintains this invariant to ensure that no reachable object is ever collected.
请注意，在此过程的每一步，都没有黑色节点指向白色节点。这个属性被称为**三色不变性**。变量过程中保持这一不变性，以确保没有任何可达对象被回收。

</aside>

### 灰色对象的工作列表

In our implementation we have already marked the roots. They're all gray. The
next step is to start picking them and traversing their references. But we don't
have any easy way to find them. We set a field on the object, but that's it. We
don't want to have to traverse the entire object list looking for objects with
that field set.
在我们的实现中，我们已经对根进行了标记。它们都是灰色的。下一步是开始挑选灰色对象并遍历其引用。但是我们没有任何简单的方法来查找灰色对象。我们在对象上设置了一个字段，但也仅此而已。我们不希望遍历整个对象列表来查找设置了该字段的对象。

Instead, we'll create a separate worklist to keep track of all of the gray
objects. When an object turns gray, in addition to setting the mark field we'll
also add it to the worklist.
相反，我们创建一个单独的工作列表来跟踪所有的灰色对象。当某个对象变成灰色时，除了设置标记字段外，我们还会将它添加到工作列表中。

^code add-to-gray-stack (1 before, 1 after)

We could use any kind of data structure that lets us put items in and take them
out easily. I picked a stack because that's the simplest to implement with a
dynamic array in C. It works mostly like other dynamic arrays we've built in
Lox, *except*, note that it calls the *system* `realloc()` function and not our
own `reallocate()` wrapper. The memory for the gray stack itself is *not*
managed by the garbage collector. We don't want growing the gray stack during a
GC to cause the GC to recursively start a new GC. That could tear a hole in the
space-time continuum.
我们可以使用任何类型的数据结构，让我们可以轻松地放入和取出项目。我选择了栈，因为这是用C语言实现动态数组最简单的方法。它的工作原理与我们在Lox中构建的其它动态数组基本相同，*除了一点*，要注意它调用了*系统*的`realloc()`函数，而不是我们自己包装的`reallocate()`。灰色对象栈本身的内存是*不被*垃圾回收器管理的。我们不希望因为GC过程中增加灰色对象栈，导致GC递归地发起一个新的GC。这可能会在时空连续体上撕开一个洞。

We'll manage its memory ourselves, explicitly. The VM owns the gray stack.
我们会自己显式地管理它的内存。VM拥有这个灰色栈。

^code vm-gray-stack (1 before, 1 after)

It starts out empty.
开始时是空的。

^code init-gray-stack (1 before, 2 after)

And we need to free it when the VM shuts down.
当VM关闭时，我们需要释放它。

^code free-gray-stack (2 before, 1 after)

<span name="robust">We</span> take full responsibility for this array. That
includes allocation failure. If we can't create or grow the gray stack, then we
can't finish the garbage collection. This is bad news for the VM, but
fortunately rare since the gray stack tends to be pretty small. It would be nice
to do something more graceful, but to keep the code in this book simple, we just
abort.
我们对这个数组负担全部责任，其中包括分配失败。如果我们不能创建或扩张灰色栈，那我们就无法完成垃圾回收。这对VM来说是个坏消息，但幸运的是这很少发生，因为灰色栈往往是非常小的。如果能做得更优雅一些就好了，但是为了保持本书中的代码简单，我们就停在这里吧。

<aside name="robust">

To be more robust, we can allocate a "rainy day fund" block of memory when we
start the VM. If the gray stack allocation fails, we free the rainy day block
and try again. That may give us enough wiggle room on the heap to create the
gray stack, finish the GC, and free up more memory.
为了更加健壮，我们可以在启动虚拟机时分配一个“雨天基金”内存块。如果灰色栈分配失败，我们就释放这个块并重新尝试。这可能会为我们在堆上提供足够的空间来创建灰色栈，完成GC并释放更多内存。

</aside>

^code exit-gray-stack (2 before, 1 after)

### 处理灰色对象

OK, now when we're done marking the roots, we have both set a bunch of fields
and filled our work list with objects to chew through. It's time for the next
phase.
好了，现在我们在完成对根的标记后，既设置了一堆字段，也用待处理的对象填满了我们的工作列表。是时候进入下一阶段了。

^code call-trace-references (1 before, 2 after)

Here's the implementation:
下面是其实现：

^code trace-references

It's as close to that textual algorithm as you can get. Until the stack empties,
we keep pulling out gray objects, traversing their references, and then marking
them black. Traversing an object's references may turn up new white objects that
get marked gray and added to the stack. So this function swings back and forth
between turning white objects gray and gray objects black, gradually advancing
the entire wavefront forward.
这与文本描述的算法已经尽可能接近了。在栈清空之前，我们会不断取出灰色对象，遍历它们的引用，然后将它们标记为黑色。遍历某个对象的引用可能会出现新的白色对象，这些对象被标记为灰色并添加到栈中。所以这个函数在把白色对象变成灰色和把灰色对象变成黑色之间来回摆动，逐渐把整个波前向前推进。

Here's where we traverse a single object's references:
下面是我们遍历某个对象的引用的地方：

^code blacken-object

Each object <span name="leaf">kind</span> has different fields that might
reference other objects, so we need a specific blob of code for each type. We
start with the easy ones -- strings and native function objects contain no
outgoing references so there is nothing to traverse.
每种对象类型都有不同的可能引用其它对象的字段，因此我们需要为每种类型编写一块特定的代码。我们从简单的开始 -- 字符串和本地函数对象不包含向外的引用，因此没有任何东西需要遍历。

<aside name="leaf">

An easy optimization we could do in `markObject()` is to skip adding strings and
native functions to the gray stack at all since we know they don't need to be
processed. Instead, they could darken from white straight to black.
我们可以在`markObject()`中做一个简单的优化，就是不要向灰色栈中添加字符串和本地函数，因为我们知道它们不需要处理。相对地，它们可以从白色直接变黑。

</aside>

Note that we don't set any state in the traversed object itself. There is no
direct encoding of "black" in the object's state. A black object is any object
whose `isMarked` field is <span name="field">set</span> and that is no longer in
the gray stack.
注意，我们没有在已被遍历的对象本身中设置任何状态。在对象的状态中，没有对“black”的直接编码。黑色对象是`isMarked`字段被设置且不再位于灰色栈中的任何对象。

<aside name="field">

You may rightly wonder why we have the `isMarked` field at all. All in good
time, friend.
你可能正在好奇为什么要有`isMarked`字段。别急，朋友。

</aside>

Now let's start adding in the other object types. The simplest is upvalues.
现在让我们开始添加其它的对象类型。最简单的是上值。

^code blacken-upvalue (2 before, 1 after)

When an upvalue is closed, it contains a reference to the closed-over value.
Since the value is no longer on the stack, we need to make sure we trace the
reference to it from the upvalue.
当某个上值被关闭后，它包含一个指向关闭值的引用。由于该值不在栈上，我们需要确保从上值中跟踪对它的引用。

Next are functions.
接下来是函数。

^code blacken-function (1 before, 1 after)

Each function has a reference to an ObjString containing the function's name.
More importantly, the function has a constant table packed full of references to
other objects. We trace all of those using this helper:
每个函数都有一个对包含函数名称的ObjString 的引用。更重要的是，函数有一个常量表，其中充满了对其它对象的引用。我们使用这个辅助函数来跟踪它们：

^code mark-array

The last object type we have now -- we'll add more in later chapters -- is
closures.
我们现在拥有的最后一种对象类型（我们会在后面的章节中添加更多）是闭包。

^code blacken-closure (1 before, 1 after)

Each closure has a reference to the bare function it wraps, as well as an array
of pointers to the upvalues it captures. We trace all of those.
每个闭包都有一个指向其包装的裸函数的引用，以及一个指向它所捕获的上值的指针数组。我们要跟踪所有这些。

That's the basic mechanism for processing a gray object, but there are two loose
ends to tie up. First, some logging.
这就是处理灰色对象的基本机制，但还有两个未解决的问题。首先，是一些日志记录。

^code log-blacken-object (1 before, 1 after)

This way, we can watch the tracing percolate through the object graph. Speaking
of which, note that I said *graph*. References between objects are directed, but
that doesn't mean they're *acyclic!* It's entirely possible to have cycles of
objects. When that happens, we need to ensure our collector doesn't get stuck in
an infinite loop as it continually re-adds the same series of objects to the
gray stack.
这样一来，我们就可以观察到跟踪操作在对象图中的渗入情况。说到这里，请注意，我说的是图。对象之间的引用是有方向的，但这并不意味着它们是无循环的！完全有可能出现对象的循环。当这种情况发生时，我们需要确保，我们的回收器不会因为持续将同一批对象添加到灰色堆栈而陷入无限循环。

The fix is easy.
解决方法很简单。

^code check-is-marked (1 before, 1 after)

If the object is already marked, we don't mark it again and thus don't add it to
the gray stack. This ensures that an already-gray object is not redundantly
added and that a black object is not inadvertently turned back to gray. In other
words, it keeps the wavefront moving forward through only the white objects.
如果对象已经被标记，我们就不会再标记它，因此也不会把它添加到灰色栈中。这就保证了已经是灰色的对象不会被重复添加，而且黑色对象不会无意中变回灰色。换句话说，它使得波前只通过白色对象向前移动。

## 清除未使用的对象

When the loop in `traceReferences()` exits, we have processed all the objects we
could get our hands on. The gray stack is empty, and every object in the heap is
either black or white. The black objects are reachable, and we want to hang on to
them. Anything still white never got touched by the trace and is thus garbage.
All that's left is to reclaim them.
当`traceReferences()`中的循环退出时，我们已经处理了所有能接触到的对象。灰色栈是空的，堆中的每个对象不是黑色就是白色。黑色对象是可达的，我们想要抓住它们。任何仍然是白色的对象都没有被追踪器接触过，因此是垃圾。剩下的就是回收它们了。

^code call-sweep (1 before, 2 after)

All of the logic lives in one function.
所有的逻辑都在一个函数中。

^code sweep

I know that's kind of a lot of code and pointer shenanigans, but there isn't
much to it once you work through it. The outer `while` loop walks the linked
list of every object in the heap, checking their mark bits. If an object is
marked (black), we leave it alone and continue past it. If it is unmarked
(white), we unlink it from the list and free it using the `freeObject()`
function we already wrote.
我知道这有点像是一堆代码和指针的诡计，不过一旦你完成了，就没什么好说的。外层的`while`循环会遍历堆中每个对象组成的链表，检查它们的标记位。如果某个对象被标记（黑色），我们就不管它，继续进行。如果它没有被标记（白色），我们将它从链表中断开，并使用我们已经写好的`freeObject()`函数释放它。

<img src="image/garbage-collection/unlink.png" alt="A recycle bin full of bits." />

Most of the other code in here deals with the fact that removing a node from a
singly linked list is cumbersome. We have to continuously remember the previous
node so we can unlink its next pointer, and we have to handle the edge case
where we are freeing the first node. But, otherwise, it's pretty simple --
delete every node in a linked list that doesn't have a bit set in it.
这里大多数其它代码都在处理这样一个事实：从单链表中删除节点非常麻烦。我们必须不断地记住前一个节点，这样我们才能断开它的next指针，而且我们还必须处理释放第一个节点这种边界情况。但是，除此之外，它非常简单 -- 删除链表中没有设置标记位的每个节点。

There's one little addition:
还有一点需要补充：

^code unmark (1 before, 1 after)

After `sweep()` completes, the only remaining objects are the live black ones
with their mark bits set. That's correct, but when the *next* collection cycle
starts, we need every object to be white. So whenever we reach a black object,
we go ahead and clear the bit now in anticipation of the next run.
在`sweep()`完成后，仅剩下的对象是带有标记位的活跃黑色对象。这是正确的，但在*下一个*回收周期开始时，我们需要每个对象都是白色的。因此，每当我们碰到黑色对象时，我们就继续并清除标记位，为下一轮作业做好准备。

### 弱引用与字符串池

We are almost done collecting. There is one remaining corner of the VM that has
some unusual requirements around memory. Recall that when we added strings to
clox we made the VM intern them all. That means the VM has a hash table
containing a pointer to every single string in the heap. The VM uses this to
de-duplicate strings.
我们差不多已经回收完毕了。虚拟机中还有一个剩余的角落，它对内存有着一些不寻常的要求。回想一下，我们在clox中添加字符串的时，我们让虚拟机对所有字符串进行驻留。这意味着VM拥有一个哈希表，其中包含指向堆中每个字符串的指针。虚拟机使用它来对字符串去重。

During the mark phase, we deliberately did *not* treat the VM's string table as
a source of roots. If we had, no <span name="intern">string</span> would *ever*
be collected. The string table would grow and grow and never yield a single byte
of memory back to the operating system. That would be bad.
在标记阶段，我们故意*不将*虚拟机的字符串表作为根的来源。如果我们这样做，就不会有字符串被回收。字符串表会不断增长，并且永远不会向操作系统让出一比特的内存。那就糟糕了。

<aside name="intern">

This can be a real problem. Java does not intern *all* strings, but it does
intern string *literals*. It also provides an API to add strings to the string
table. For many years, the capacity of that table was fixed, and strings added
to it could never be removed. If users weren't careful about their use of
`String.intern()`, they could run out of memory and crash.
这可能是一个真正的问题。Java并没有驻留*所有*字符串，但它确实驻留了字符串*字面量*。它还提供了向字符串表添加字符串的API。多年以来，该表的容量是固定的，添加到其中的字符串永远无法被删除。如果用户不谨慎使用`String.intern()`，他们可能会耗尽内存导致崩溃。

Ruby had a similar problem for years where symbols -- interned string-like
values -- were not garbage collected. Both eventually enabled the GC to collect
these strings.
Ruby多年以来也存在类似的问题，符号（驻留的类似字符串的值）不会被垃圾回收。两者最终都启用了GC来回收这些字符串。

</aside>

At the same time, if we *do* let the GC free strings, then the VM's string table
will be left with dangling pointers to freed memory. That would be even worse.
同时，如果我们真的让GC释放字符串，那么VM的字符串表就会留下指向已释放内存的悬空指针。那就更糟糕了。

The string table is special and we need special support for it. In particular,
it needs a special kind of reference. The table should be able to refer to a
string, but that link should not be considered a root when determining
reachability. That implies that the referenced object can be freed. When that
happens, the dangling reference must be fixed too, sort of like a magic,
self-clearing pointer. This particular set of semantics comes up frequently
enough that it has a name: a [**weak reference**][weak].
字符串表是很特殊的，我们需要对它进行特殊的支持。特别是，它需要一种特殊的引用。这个表应该能够引用字符串，但在确定可达性时，不应该将该链接视为根。这意味着被引用的对象也可以被释放。当这种情况发生时，悬空的引用也必须被修正，有点像一个神奇的、自我清除的指针。这组特定的语义出现得非常频繁，所以它有一个名字：**[弱引用](https://en.wikipedia.org/wiki/Weak_reference)**。

[weak]: https://en.wikipedia.org/wiki/Weak_reference

We have already implicitly implemented half of the string table's unique
behavior by virtue of the fact that we *don't* traverse it during marking. That
means it doesn't force strings to be reachable. The remaining piece is clearing
out any dangling pointers for strings that are freed.
我们已经隐式地实现了一半的字符串表的独特行为，因为我们在标记阶段没有遍历它。这意味着它不强制要求字符串可达。剩下的部分就是清除任何指向被释放字符串的悬空指针。

To remove references to unreachable strings, we need to know which strings *are*
unreachable. We don't know that until after the mark phase has completed. But we
can't wait until after the sweep phase is done because by then the objects --
and their mark bits -- are no longer around to check. So the right time is
exactly between the marking and sweeping phases.
为了删除对不可达字符串的引用，我们需要知道哪些字符串不可达。在标记阶段完成之后，我们才能知道这一点。但是我们不能等到清除阶段完成之后，因为到那时对象（以及它们的标记位）已经无法再检查了。因此，正确的时机正好是在标记和清除阶段之间。

^code sweep-strings (1 before, 1 after)

The logic for removing the about-to-be-deleted strings exists in a new function
in the "table" module.
清除即将被删除的字符串的逻辑存在于“table”模块中的一个新函数中。

^code table-remove-white-h (2 before, 2 after)

The implementation is here:
实现在这里：

^code table-remove-white

We walk every entry in the table. The string intern table uses only the key of
each entry -- it's basically a hash *set* not a hash *map*. If the key string
object's mark bit is not set, then it is a white object that is moments from
being swept away. We delete it from the hash table first and thus ensure we
won't see any dangling pointers.
我们遍历表中的每一项。字符串驻留表只使用了每一项的键 -- 它基本上是一个HashSet而不是HashMap。如果键字符串对象的标记位没有被设置，那么它就是一个白色对象，很快就会被清除。我们首先从哈希表中删除它，从而确保不会看到任何悬空指针。

## 何时回收

We have a fully functioning mark-sweep garbage collector now. When the stress
testing flag is enabled, it gets called all the time, and with the logging
enabled too, we can watch it do its thing and see that it is indeed reclaiming
memory. But, when the stress testing flag is off, it never runs at all. It's
time to decide when the collector should be invoked during normal program
execution.
我们现在有了一个功能完备的标记-清除垃圾回收器。当压力测试标志启用时，它会一直被调用，而且在日志功能也被启用的情况下，我们可以观察到它正在工作，并看到它确实在回收内存。但是，当压力测试标志关闭时，它根本就不会运行。现在是时候决定，在正常的程序执行过程中，何时应该调用回收器。

As far as I can tell, this question is poorly answered by the literature. When
garbage collectors were first invented, computers had a tiny, fixed amount of
memory. Many of the early GC papers assumed that you set aside a few thousand
words of memory -- in other words, most of it -- and invoked the collector
whenever you ran out. Simple.
据我所知，这个问题在文献中没有得到很好的回答。在垃圾回收器刚被发明出来的时候，计算机只有一个很小的、固定大小的内存。许多早期的GC论文假定你预留了几千个字的内存（换句话说，其中大部分是这样），并在内存用完时调用回收器。这很简单。

Modern machines have gigs of physical RAM, hidden behind the operating system's
even larger virtual memory abstraction, which is shared among a slew of other
programs all fighting for their chunk of memory. The operating system will let
your program request as much as it wants and then page in and out from the disc
when physical memory gets full. You never really "run out" of memory, you just
get slower and slower.
现代计算机拥有数以G计的物理内存，而操作系统在其基础上提供了更多的虚拟内存抽象，这些物理内存是由一系列其它程序共享的，所有程序都在争夺自己的那块内存。操作系统会允许你的程序尽可能多地申请内存，然后当物理内存满时会利用磁盘进行页面换入换出。你永远不会真的“耗尽”内存，只是变得越来越慢。

### 延迟和吞吐量

It no longer makes sense to wait until you "have to", to run the GC, so we need
a more subtle timing strategy. To reason about this more precisely, it's time to
introduce two fundamental numbers used when measuring a memory manager's
performance: *throughput* and *latency*.
等到“不得不做”的时候再去运行GC，就没有意义了，因此我们需要一种更巧妙的选时策略。为了更精确地解释这个问题，现在应该引入在度量内存管理器性能时使用的两个基本数值：*吞吐量*和*延迟*。

Every managed language pays a performance price compared to explicit,
user-authored deallocation. The time spent actually freeing memory is the same,
but the GC spends cycles figuring out *which* memory to free. That is time *not*
spent running the user's code and doing useful work. In our implementation,
that's the entirety of the mark phase. The goal of a sophisticated garbage
collector is to minimize that overhead.
与显式的、用户自发的释放内存相比，每一种托管语言都要付出性能代价。实际释放内存所花费的时间是相同的，但是GC花费了一些周期来计算要释放*哪些*内存。这些时间没有花在运行用户的代码和做有用的工作。在我们的实现中，这就是整个标记阶段。复杂的垃圾回收器的模板就是使这种开销最小化。

There are two key metrics we can use to understand that cost better:
我们可以使用这两个关键指标来更好地理解成本：

*   **Throughput** is the total fraction of time spent running user code versus
    doing garbage collection work. Say you run a clox program for ten seconds
    and it spends a second of that inside `collectGarbage()`. That means the
    throughput is 90% -- it spent 90% of the time running the program and 10%
    on GC overhead.
    **吞吐量**是指运行用户代码的时间与执行垃圾回收工作所花费的时间的总比例。假设你运行一个clox程序10秒钟，其中有1秒花在`collectGarbage()`中。这意味是吞吐量是90% -- 它花费了90%的时间运行程序，10%的时间用于GC开销。

    Throughput is the most fundamental measure because it tracks the total cost
    of collection overhead. All else being equal, you want to maximize
    throughput. Up until this chapter, clox had no GC at all and thus <span
    name="hundred">100%</span> throughput. That's pretty hard to beat. Of
    course, it came at the slight expense of potentially running out of memory
    and crashing if the user's program ran long enough. You can look at the goal
    of a GC as fixing that "glitch" while sacrificing as little throughput as
    possible.
    吞吐量是最基本的度量值，因为它跟踪的是回收开销的总成本。在其它条件相同的情况下，你会希望最大化吞吐量。在本章之前，clox完全没有GC，因此吞吐量为100%。这是很难做到的。当然，它的代价是，如果用户的程序运行时间足够长的话，可能会导致内存耗尽和程序崩溃。你可以把GC的目标看作是修复这个“小故障”，同时以牺牲尽可能少的吞吐量为代价。

<aside name="hundred">

Well, not *exactly* 100%. It did still put the allocated objects into a linked
list, so there was some tiny overhead for setting those pointers.
嗯，不完全是100%。它仍然将分配的对象放入了一个链表中，所以在设置这些指针时有一些微小的开销。

</aside>

*   **Latency** is the longest *continuous* chunk of time where the user's
    program is completely paused while garbage collection happens. It's a
    measure of how "chunky" the collector is. Latency is an entirely different
    metric than throughput.
    **延迟**是指当垃圾回收发生时，用户的程序完全暂停的最长连续时间块。这是衡量回收器“笨重”程度的指标。延迟是一个与吞吐量完全不同的指标。

    Consider two runs of a clox program that both take ten seconds. In the first
    run, the GC kicks in once and spends a solid second in `collectGarbage()` in
    one massive collection. In the second run, the GC gets invoked five times,
    each for a fifth of a second. The *total* amount of time spent collecting is
    still a second, so the throughput is 90% in both cases. But in the second
    run, the latency is only 1/5th of a second, five times less than in the
    first.
    考虑一下，一个程序的两次运行都花费了10秒。第一次运行时，GC启动了一次，并在`collectGarbage()`中花费了整整1秒钟进行了一次大规模的回收。在第二次运行中，GC被调用了五次，每次调用1/5秒。回收所花费的总时间仍然是1秒，所以这两种情况下的吞吐量都是90%。但是在第二次运行中，延迟只有1/5秒，比第一次少了5倍。

<span name="latency"></span>

<img src="image/garbage-collection/latency-throughput.png" alt="A bar representing execution time with slices for running user code and running the GC. The largest GC slice is latency. The size of all of the user code slices is throughput." />

<aside name="latency">

The bar represents the execution of a program, divided into time spent running
user code and time spent in the GC. The size of the largest single slice of time
running the GC is the latency. The size of all of the user code slices added up
is the throughput.
每个条带表示程序的执行，分为运行用户代码的时间和在GC中花费的时间。运行GC的最大单个时间片的大小就是延迟。所有用户代码片的大小相加就是吞吐量。

</aside>

If you like analogies, imagine your program is a bakery selling fresh-baked
bread to customers. Throughput is the total number of warm, crusty baguettes you
can serve to customers in a single day. Latency is how long the unluckiest
customer has to wait in line before they get served.
如果你喜欢打比方，可以将你的程序想象成一家面包店，向顾客出售新鲜出炉的面包。吞吐量是指你在一天内可以为顾客提供的温暖结皮的法棍的总数。延迟是指最不走运的顾客在得到服务之前需要排队等候多长时间。

<span name="dishwasher">Running</span> the garbage collector is like shutting
down the bakery temporarily to go through all of the dishes, sort out the dirty
from the clean, and then wash the used ones. In our analogy, we don't have
dedicated dishwashers, so while this is going on, no baking is happening. The
baker is washing up.
运行垃圾回收器就像暂时关闭面包店，去检查所有的盘子，把脏的和干净的分开，然后把用过的洗掉。在我们的比喻中，我们没有专门的洗碗机，所以在这个过程中，没有烘焙发生。面包师正在清洗。

<aside name="dishwasher">

If each person represents a thread, then an obvious optimization is to have
separate threads running garbage collection, giving you a **concurrent garbage
collector**. In other words, hire some dishwashers to clean while others bake.
This is how very sophisticated GCs work because it does let the bakers
-- the worker threads -- keep running user code with little interruption.
如果每个人代表一个线程，那么一个明显的优化就是让单独的线程进行垃圾回收，提供一个**并发垃圾回收器**。换句话说，在其他人烘焙的时候，雇佣一些洗碗工来清洗。这就是非常复杂的GC工作方式，因为它确实允许烘焙师（工作线程）在几乎没有中断的情况下持续运行用户代码。

However, coordination is required. You don't want a dishwasher grabbing a bowl
out of a baker's hands! This coordination adds overhead and a lot of complexity.
Concurrent collectors are fast, but challenging to implement correctly.
但是，协调是必须的。你不会想让洗碗工从面包师手中抢走碗吧！这种协调增加了开销和大量的复杂性。并发回收器速度很快，但要正确实现却很有挑战性。

<img src="image/garbage-collection/baguette.png" class="above" alt="Un baguette." />

</aside>

Selling fewer loaves of bread a day is bad, and making any particular customer
sit and wait while you clean all the dishes is too. The goal is to maximize
throughput and minimize latency, but there is no free lunch, even inside a
bakery. Garbage collectors make different trade-offs between how much throughput
they sacrifice and latency they tolerate.
每天卖出更少的面包是糟糕的，让任何一个顾客坐着等你洗完所有的盘子也是如此。我们的目标是最大化吞吐量和最小化延迟，但是没有免费的午餐，即使是在面包店里。不同垃圾回收器在牺牲多少吞吐量和容忍多大延迟之间做出了不同的权衡。

Being able to make these trade-offs is useful because different user programs
have different needs. An overnight batch job that is generating a report from a
terabyte of data just needs to get as much work done as fast as possible.
Throughput is queen. Meanwhile, an app running on a user's smartphone needs to
always respond immediately to user input so that dragging on the screen feels
<span name="butter">buttery</span> smooth. The app can't freeze for a few
seconds while the GC mucks around in the heap.
能够进行这些权衡是很有用的，因为不同的用户程序有不同的需求。一个从TB级数据中生成报告的夜间批处理作业，只需要尽可能快地完成尽可能多的工作。吞吐量为王。与此同时，在用户智能手机上运行的应用程序需要总是对用户输入立即做出响应，这样才能让用户在屏幕上拖拽时感觉非常流畅。应用程序不能因为GC在堆中乱翻而冻结几秒钟。

<aside name="butter">

Clearly the baking analogy is going to my head.
显然，烘焙的比喻让我头晕目眩。

</aside>

As a garbage collector author, you control some of the trade-off between
throughput and latency by your choice of collection algorithm. But even within a
single algorithm, we have a lot of control over *how frequently* the collector
runs.
作为一个垃圾回收器作者，你可以通过选择收集算法来控制吞吐量和延迟之间的一些权衡。但即使在单一的算法中，我们也可以对回收器的运行频率有很大的控制。

Our collector is a <span name="incremental">**stop-the-world GC**</span> which
means the user's program is paused until the entire garbage collection process
has completed. If we wait a long time before we run the collector, then a large
number of dead objects will accumulate. That leads to a very long pause while
the collector runs, and thus high latency. So, clearly, we want to run the
collector really frequently.
我们的回收器是一个**stop-the-world GC**，这意味着会暂停用户的程序，直到垃圾回收过程完成。如果我们在运行回收器之前等待很长时间，那么将会积累大量的死亡对象。这会导致回收器在运行时会出现很长时间的停顿，从而导致高延迟。所以，很明显，我们希望频繁地运行回收器。

<aside name="incremental">

In contrast, an **incremental garbage collector** can do a little collection,
then run some user code, then collect a little more, and so on.
相比之下，**增量式垃圾回收器**可以做一点回收工作，然后运行一些用户代码，然后再做一点回收工作，以此类推。

</aside>

But every time the collector runs, it spends some time visiting live objects.
That doesn't really *do* anything useful (aside from ensuring that they don't
incorrectly get deleted). Time visiting live objects is time not freeing memory
and also time not running user code. If you run the GC *really* frequently, then
the user's program doesn't have enough time to even generate new garbage for the
VM to collect. The VM will spend all of its time obsessively revisiting the same
set of live objects over and over, and throughput will suffer. So, clearly, we
want to run the collector really *in*frequently.
但是每次回收器运行时，它都要花一些时间来访问活动对象。这其实并没有什么用处（除了确保它们不会被错误地删除之外）。访问活动对象的时间是没有释放内存的时间，也是没有运行用户代码的时间。如果你*真的*非常频繁地运行GC，那么用户的程序甚至没有足够的时间生成新的垃圾供VM回收。VM会花费所有的时间反复访问相同的活动对象，吞吐量将会受到影响。所以，很明显，我们也不希望频繁地运行回收器。

In fact, we want something in the middle, and the frequency of when the
collector runs is one of our main knobs for tuning the trade-off between latency
and throughput.
事实上，我们想要的是介于两者之间的东西，而回收器的运行频率是我们调整延迟和吞吐量之间权衡的主要因素之一。

### 自适应堆

We want our GC to run frequently enough to minimize latency but infrequently
enough to maintain decent throughput. But how do we find the balance between
these when we have no idea how much memory the user's program needs and how
often it allocates? We could pawn the problem onto the user and force them to
pick by exposing GC tuning parameters. Many VMs do this. But if we, the GC
authors, don't know how to tune it well, odds are good most users won't either.
They deserve a reasonable default behavior.
我们希望GC运行得足够频繁，以最小化延迟，但又不能太频繁，以维持良好的吞吐量。但是，当我们不知道用户程序需要多少内存以及内存分配的频率时，我们如何在两者之间找到平衡呢？我们可以把问题推给用户，并通过暴露GC调整参数来迫使他们进行选择。许多虚拟机都是这样做的。但是，如果我们这些GC的作者都不知道如何很好地调优回收器，那么大多数用户可能也不知道。他们理应得到一个合理的默认行为。

I'll be honest with you, this is not my area of expertise. I've talked to a
number of professional GC hackers -- this is something you can build an entire
career on -- and read a lot of the literature, and all of the answers I got
were... vague. The strategy I ended up picking is common, pretty simple, and (I
hope!) good enough for most uses.
说实话，这不是我的专业领域。我曾经和一些专业的GC专家交谈过（GC是一项可以投入整个职业生涯的东西），并且阅读了大量的文献，我得到的所有答案都是……模糊的。我最终选择的策略很常见，也很简单，而且（我希望！）对大多数用途来说足够好。

The idea is that the collector frequency automatically adjusts based on the live
size of the heap. We track the total number of bytes of managed memory that the
VM has allocated. When it goes above some threshold, we trigger a GC. After
that, we note how many bytes of memory remain -- how many were *not* freed. Then
we adjust the threshold to some value larger than that.
其思想是，回收器的频率根据堆的大小自动调整。我们根据虚拟机已分配的托管内存的总字节数。当它超过某个阈值时，我们就触发一次GC。在那之后，我们关注一下有多少字节保留下来 -- 多少没有被释放。然后我们将阈值调整为比它更大的某个值。

The result is that as the amount of live memory increases, we collect less
frequently in order to avoid sacrificing throughput by re-traversing the growing
pile of live objects. As the amount of live memory goes down, we collect more
frequently so that we don't lose too much latency by waiting too long.
其结果是，随着活动内存数量的增加，我们回收的频率会降低，以避免因为重新遍历不断增长的活动对象而牺牲吞吐量。随着活动内存数量的减少，我们会更频繁地收集，这样我们就不会因为等待时间过长而造成太多的延迟。

The implementation requires two new bookkeeping fields in the VM.
这个实现需要在虚拟机中设置两个新的簿记字段。

^code vm-fields (1 before, 1 after)

The first is a running total of the number of bytes of managed memory the VM has
allocated. The second is the threshold that triggers the next collection. We
initialize them when the VM starts up.
第一个是虚拟机已分配的托管内存实时字节总数。第二个是触发下一次回收的阈值。我们在虚拟机启动时初始化它们。

^code init-gc-fields (1 before, 2 after)

The starting threshold here is <span name="lab">arbitrary</span>. It's similar
to the initial capacity we picked for our various dynamic arrays. The goal is to
not trigger the first few GCs *too* quickly but also to not wait too long. If we
had some real-world Lox programs, we could profile those to tune this. But since
all we have are toy programs, I just picked a number.
这里的起始阈值是任意的。它类似于我们为各种动态数据选择的初始容量。我们的目标是不要太快触发最初的几次GC，但是也不要等得太久。如果我们有一些真实的Lox程序，我们可以对程序进行剖析来调整这个参数。但是因为我们写的都是一些玩具程序，我只是随意选了一个数字。

<aside name="lab">

A challenge with learning garbage collectors is that it's *very* hard to
discover the best practices in an isolated lab environment. You don't see how a
collector actually performs unless you run it on the kind of large, messy
real-world programs it is actually intended for. It's like tuning a rally car
-- you need to take it out on the course.
学习垃圾回收器的一个挑战是，在孤立的实验室环境中很难发现最佳实践。除非你在大型的、混乱的真实世界的程序上运行回收器，否则你无法看到它的实际表现。这就像调校一辆拉力赛车 -- 你需要把它带到赛道上。

</aside>

Every time we allocate or free some memory, we adjust the counter by that delta.
每当我们分配或释放一些内存时，我们就根据差值来调整计数器。

^code updated-bytes-allocated (1 before, 1 after)

When the total crosses the limit, we run the collector.
当总数超过限制时，我们运行回收器。

^code collect-on-next (2 before, 1 after)

Now, finally, our garbage collector actually does something when the user runs a
program without our hidden diagnostic flag enabled. The sweep phase frees
objects by calling `reallocate()`, which lowers the value of `bytesAllocated`,
so after the collection completes, we know how many live bytes remain. We adjust
the threshold of the next GC based on that.
现在，终于，即便用户运行一个没有启用隐藏诊断标志的程序时，我们的垃圾回收器实际上也做了一些事情。扫描阶段通过调用`reallocate()`释放对象，这会降低`bytesAllocated`的值，所以在收集完成后，我们知道还有多少活动字节。我们在此基础上调整下一次GC的阈值。

^code update-next-gc (1 before, 2 after)

The threshold is a multiple of the heap size. This way, as the amount of memory
the program uses grows, the threshold moves farther out to limit the total time
spent re-traversing the larger live set. Like other numbers in this chapter, the
scaling factor is basically arbitrary.
该阈值是堆大小的倍数。这样一来，随着程序使用的内存量的增加长，阈值会向上移动。以限制重新遍历更大的活动集合所花费的总时间。和本章中的其它数字一样，比例因子基本上是任意的。

^code heap-grow-factor (1 before, 2 after)

You'd want to tune this in your implementation once you had some real programs
to benchmark it on. Right now, we can at least log some of the statistics that
we have. We capture the heap size before the collection.
一旦你有了一些真正的程序来对其进行基准测试，你就需要在实现中对该参数进行调优。现在，我们至少可以记录一些统计数据。我们在回收之前捕获堆的大小。

^code log-before-size (1 before, 1 after)

And then print the results at the end.
最后把结果打印出来。

^code log-collected-amount (1 before, 1 after)

This way we can see how much the garbage collector accomplished while it ran.
这样，我们就可以看到垃圾回收器在运行时完成了多少任务。

## 垃圾回收Bug

In theory, we are all done now. We have a GC. It kicks in periodically, collects
what it can, and leaves the rest. If this were a typical textbook, we would wipe
the dust from our hands and bask in the soft glow of the flawless marble edifice
we have created.
理论上讲，我们现在已经完成了。我们有了一个GC，它周期性启动，回收可以回收的东西，并留下其余的东西。如果这是一本典型的教科书，我们会擦掉手上的灰尘，沉浸在我们所创造的完美无瑕的大理石建筑的柔和光芒中。

But I aim to teach you not just the theory of programming languages but the
sometimes painful reality. I am going to roll over a rotten log and show you the
nasty bugs that live under it, and garbage collector bugs really are some of the
grossest invertebrates out there.
但是，我的目的不仅仅是教授编程语言的理论，还要教你有时令人痛苦的现实。我要掀开一根烂木头，向你展示生活在下面的讨厌的虫子，垃圾回收器虫(垃圾回收器bug)真的是世界上最恶心的无脊椎动物之一。

The collector's job is to free dead objects and preserve live ones. Mistakes are
easy to make in both directions. If the VM fails to free objects that aren't
needed, it slowly leaks memory. If it frees an object that is in use, the user's
program can access invalid memory. These failures often don't immediately cause
a crash, which makes it hard for us to trace backward in time to find the bug.
回收器的工作是释放已死对象并保留活动对象。在这两个方面都很容易出现错误。如果虚拟机不能释放不需要的对象，就会慢慢地泄露内存。如果它释放了一个正在使用的对象，用户的程序就会访问无效的内存。这些故障通常不会立即导致崩溃，这使得我们很难即时追溯以找到错误。

This is made harder by the fact that we don't know when the collector will run.
Any call that eventually allocates some memory is a place in the VM where a
collection could happen. It's like musical chairs. At any point, the GC might
stop the music. Every single heap-allocated object that we want to keep needs to
find a chair quickly -- get marked as a root or stored as a reference in some
other object -- before the sweep phase comes to kick it out of the game.
由于我们不知道回收器何时会运行，这就更加困难了。任何发生内存分配的地方恰好可能是发生回收的地方。这就像抢椅子游戏。在任何时候，GC都可能停止音乐。我们想保留的每一个堆分配对象都需要快速找到一个椅子（被标记为根或作为引用保存在其它对象中），在清除阶段将其踢出游戏之前。

How is it possible for the VM to use an object later -- one that the GC itself
doesn't see? How can the VM find it? The most common answer is through a pointer
stored in some local variable on the C stack. The GC walks the *VM's* value and
CallFrame stacks, but the C stack is <span name="c">hidden</span> to it.
VM怎么可能会在稍后使用一个GC自己都看不到的对象呢？VM如何找到它？最常见的答案是通过存储在C栈中的一些局部变量。GC会遍历VM的值和CallFrame栈，但C的栈对它来说是隐藏的。

<aside name="c">

Our GC can't find addresses in the C stack, but many can. Conservative garbage
collectors look all through memory, including the native stack. The most
well-known of this variety is the [**Boehm–Demers–Weiser garbage
collector**][boehm], usually just called the "Boehm collector". (The shortest
path to fame in CS is a last name that's alphabetically early so that it shows
up first in sorted lists of names.)
我们的GC无法在C栈中查找地址，但很多GC可以。保守的垃圾回收器会查看所有内存，包括本机堆栈。这类垃圾回收器中最著名的是[**Boehm–Demers–Weiser垃圾回收器** ][boehm]，通常就叫作“Boehm 回收器”。（在CS中，成名的捷径是姓氏在字母顺序上靠前，这样就能在排序的名字列表中出现在第一位）

[boehm]: https://en.wikipedia.org/wiki/Boehm_garbage_collector

Many precise GCs walk the C stack too. Even those have to be careful about
pointers to live objects that exist only in *CPU registers*.
许多精确GC也在C栈中遍历。即便是这些GC，也必须对指向仅存于CPU寄存器中的活动对象的指针加以注意。

</aside>

In previous chapters, we wrote seemingly pointless code that pushed an object
onto the VM's value stack, did a little work, and then popped it right back off.
Most times, I said this was for the GC's benefit. Now you see why. The code
between pushing and popping potentially allocates memory and thus can trigger a
GC. We had to make sure the object was on the value stack so that the
collector's mark phase would find it and keep it alive.
在前面的章节中，我们编写了一些看似无意义的代码，将一个对象推到VM的值栈上，执行一些操作，然后又把它弹了出来。大多数时候，我说这是为了便于GC。现在你知道为什么了。压入和弹出之间的代码可能会分配内存，因此可能会触发GC。我们必须确保对象在值栈上，这样回收器的标记阶段才能找到它并保持它存活。

I wrote the entire clox implementation before splitting it into chapters and
writing the prose, so I had plenty of time to find all of these corners and
flush out most of these bugs. The stress testing code we put in at the beginning
of this chapter and a pretty good test suite were very helpful.
在把整个clox拆分为不同章节并编写文章之前，我已经写完了整个clox实现，因此我有足够的时间来找到这些角落，并清除大部分的bug。我们在本章开始时放入的压力测试代码和一个相当好的测试套件都非常有帮助。

But I fixed only *most* of them. I left a couple in because I want to give you a
hint of what it's like to encounter these bugs in the wild. If you enable the
stress test flag and run some toy Lox programs, you can probably stumble onto a
few. Give it a try and *see if you can fix any yourself*.
但我只修复了其中的大部分。我留下了几个，因为我想给你一些提示，告诉你在野外遇到这些虫子是什么感觉。如果你启用压力测试标志并运行一些玩具Lox程序，你可能会偶然发现一些。试一试，看看你是否能自己解决问题。


### 添加到常量表中

You are very likely to hit the first bug. The constant table each chunk owns is
a dynamic array. When the compiler adds a new constant to the current function's
table, that array may need to grow. The constant itself may also be some
heap-allocated object like a string or a nested function.
你很有可能会碰到第一个bug。每个块拥有的常量表是一个动态数组。当编译器向当前函数的表中添加一个新常量时，这个数组可能需要增长。常量本身也可以是一些堆分配的对象，如字符串或嵌套函数。

The new object being added to the constant table is passed to `addConstant()`.
At that moment, the object can be found only in the parameter to that function
on the C stack. That function appends the object to the constant table. If the
table doesn't have enough capacity and needs to grow, it calls `reallocate()`.
That in turn triggers a GC, which fails to mark the new constant object and
thus sweeps it right before we have a chance to add it to the table. Crash.
待添加到常量表的新对象会被传递给`addConstant()`。此时，该对象只能在C栈上该函数的形参中找到。该函数将对象追加到常量表中。如果表中没有足够的容量并且需要增长，它会调用`reallocate()`。这反过来又触发了一次GC，它无法标记新的常量对象，因此在我们有机会将该对象添加到常量表之前便将其清除了。崩溃。

The fix, as you've seen in other places, is to push the constant onto the stack
temporarily.
正如你在其它地方所看到的，解决方法是将常量临时推入栈中。

^code add-constant-push (1 before, 1 after)

Once the constant table contains the object, we pop it off the stack.
一旦常量表中有了该对象，我们就将其从栈中弹出。

^code add-constant-pop (1 before, 1 after)

When the GC is marking roots, it walks the chain of compilers and marks each of
their functions, so the new constant is reachable now. We do need an include
to call into the VM from the "chunk" module.
当GC标记根时，它会遍历编译器链并标记它们的每个函数，因此现在新的常量是可达的。我们确实需要引入头文件开从“chunk”模块调用到VM中。

^code chunk-include-vm (1 before, 2 after)

### 驻留字符串

Here's another similar one. All strings are interned in clox, so whenever we
create a new string, we also add it to the intern table. You can see where this
is going. Since the string is brand new, it isn't reachable anywhere. And
resizing the string pool can trigger a collection. Again, we go ahead and stash
the string on the stack first.
下面是另一个类似的例子。所有字符串在clox都是驻留的，因此每当创建一个新的字符串时，我们也会将其添加到驻留表中。你知道将会发生什么。因为字符串是全新的，所以它在任何地方都是不可达的。调整字符串池的大小会触发一次回收。同样，我们先去把字符串藏在栈上。

^code push-string (2 before, 1 after)

And then pop it back off once it's safely nestled in the table.
等它稳稳地进入表中，再把它弹出来。

^code pop-string (1 before, 2 after)

This ensures the string is safe while the table is being resized. Once it
survives that, `allocateString()` will return it to some caller which can then
take responsibility for ensuring the string is still reachable before the next
heap allocation occurs.
这确保了在调整表大小时字符串是安全的。一旦它存活下来，`allocateString()`会把它返回给某个调用者，随后调用者负责确保，在下一次堆分配之前字符串仍然是可达的。

### 连接字符串

One last example: Over in the interpreter, the `OP_ADD` instruction can be used
to concatenate two strings. As it does with numbers, it pops the two operands
from the stack, computes the result, and pushes that new value back onto the
stack. For numbers that's perfectly safe.
最后一个例子：在解释器中，`OP_ADD`指令可以用来连接两个字符串。就像处理数字一样，它会从栈中取出两个操作数，计算结果，并将新值压入栈中。对于数字来说，这是绝对安全的。

But concatenating two strings requires allocating a new character array on the
heap, which can in turn trigger a GC. Since we've already popped the operand
strings by that point, they can potentially be missed by the mark phase and get
swept away. Instead of popping them off the stack eagerly, we peek them.
但是连接两个字符串需要在堆中分配一个新的字符数组，这又会触发一次GC。因为此时我们已经弹出了操作数字符串，它们可能被标记阶段遗漏并被清除。我们不急于从栈中弹出这些字符串，而只是查看一下它们。

^code concatenate-peek (1 before, 2 after)

That way, they are still hanging out on the stack when we create the result
string. Once that's done, we can safely pop them off and replace them with the
result.
这样，当我们创建结果字符串时，它们仍然挂在栈上。一旦完成操作，我们就可以放心的将它们弹出，并用结果字符串替换它们。

^code concatenate-pop (1 before, 1 after)

Those were all pretty easy, especially because I *showed* you where the fix was.
In practice, *finding* them is the hard part. All you see is an object that
*should* be there but isn't. It's not like other bugs where you're looking for
the code that *causes* some problem. You're looking for the *absence* of code
which fails to *prevent* a problem, and that's a much harder search.
这些都很简单，特别是因为我告诉了你解决方法在哪里。实际上，*找到*它们才是困难的部分。你所看到的只是一个*本该*存在但却不存在的对象。它不像其它错误那样，你需要找的是*导致*某些问题的代码。这里你要找的是那些无法防止问题发生的代码*缺失*，而这是一个更困难的搜索。

But, for now at least, you can rest easy. As far as I know, we've found all of
the collection bugs in clox, and now we have a working, robust, self-tuning,
mark-sweep garbage collector.
但是，至少现在，你可以放心了。据我所知，我们已经找到了clox中的所有回收错误，现在我们有了一个有效的、强大的、自我调整的标记-清除垃圾回收器。

<div class="challenges">

## Challenges

1.  The Obj header struct at the top of each object now has three fields:
    `type`, `isMarked`, and `next`. How much memory do those take up (on your
    machine)? Can you come up with something more compact? Is there a runtime
    cost to doing so?
    每个对象顶部的Obj头结构体现在有三个字段：`type`，`isMarked`和`next`。它们（在你的机器上）占用了多少内存？你能想出更紧凑的办法吗？这样做是否有运行时成本？

1.  When the sweep phase traverses a live object, it clears the `isMarked`
    field to prepare it for the next collection cycle. Can you come up with a
    more efficient approach?
    当清除阶段遍历某个活动对象时，它会清除`isMarked`字段，以便为下一个回收周期做好准备。你能想出一个更有效的方法吗？

1.  Mark-sweep is only one of a variety of garbage collection algorithms out
    there. Explore those by replacing or augmenting the current collector with
    another one. Good candidates to consider are reference counting, Cheney's
    algorithm, or the Lisp 2 mark-compact algorithm.
    标记-清除只是众多垃圾回收算法中的一种。通过用另一种回收器来替换或增强当前的回收器来探索这些算法。可以考虑引用计数、Cheney算法或Lisp 2标记-压缩算法。
</div>

<div class="design-note">

## Design Note: 分代回收器(Generational Collectors)

A collector loses throughput if it spends a long time re-visiting objects that
are still alive. But it can increase latency if it avoids collecting and
accumulates a large pile of garbage to wade through. If only there were some way
to tell which objects were likely to be long-lived and which weren't. Then the
GC could avoid revisiting the long-lived ones as often and clean up the
ephemeral ones more frequently.

It turns out there kind of is. Many years ago, GC researchers gathered metrics
on the lifetime of objects in real-world running programs. They tracked every
object when it was allocated, and eventually when it was no longer needed, and
then graphed out how long objects tended to live.

They discovered something they called the **generational hypothesis**, or the
much less tactful term **infant mortality**. Their observation was that most
objects are very short-lived but once they survive beyond a certain age, they
tend to stick around quite a long time. The longer an object *has* lived, the
longer it likely will *continue* to live. This observation is powerful because
it gave them a handle on how to partition objects into groups that benefit from
frequent collections and those that don't.

They designed a technique called **generational garbage collection**. It works
like this: Every time a new object is allocated, it goes into a special,
relatively small region of the heap called the "nursery". Since objects tend to
die young, the garbage collector is invoked <span
name="nursery">frequently</span> over the objects just in this region.

<aside name="nursery">

Nurseries are also usually managed using a copying collector which is faster at
allocating and freeing objects than a mark-sweep collector.
nursery通常也是要复制回收器进行管理，它在分配和释放对象方面比标记-清除回收器更快。

</aside>

Each time the GC runs over the nursery is called a "generation". Any objects
that are no longer needed get freed. Those that survive are now considered one
generation older, and the GC tracks this for each object. If an object survives
a certain number of generations -- often just a single collection -- it gets
*tenured*. At this point, it is copied out of the nursery into a much larger
heap region for long-lived objects. The garbage collector runs over that region
too, but much less frequently since odds are good that most of those objects
will still be alive.

Generational collectors are a beautiful marriage of empirical data -- the
observation that object lifetimes are *not* evenly distributed -- and clever
algorithm design that takes advantage of that fact. They're also conceptually
quite simple. You can think of one as just two separately tuned GCs and a pretty
simple policy for moving objects from one to the other.

</div>

<div class="design-note">

如果回收器花费很长时间重新访问仍然活动的对象，则会损失吞吐量。但是，如果它避免了回收并积累了一大堆需要处理的垃圾，就会增加延迟。要是能有某种办法可以告诉我们哪些对象可能是长寿的以及哪些对象不是就好了。这样GC就可以避免频繁地重新访问寿命较长的数据，而更频繁地清理那些短暂寿命短暂的对象。

事实证明，确实如此。许多年前，GC研究人员收集了关于真实运行程序中对象生命周期的指标。他们跟踪了每个对象被分配时，以及它最终不再需要时的情况，然后用图表显示出对象的寿命。

他们发现了一种被称为“**代际假说**”的东西，或者是一个不太委婉的术语“**早夭**”。他们的观察结果是，大多数对象的寿命都很短，但是一旦它们存活超过了一定的年龄，它们往往会存活相当长的时间。一个对象*已经*存活的时间越长，它将*继续*存活的时间就越长。这一观察结果非常有说服力，因为这为他们提供了将对象划分为频繁回收的群体和不频繁回收群体的方法。

他们设计了一种叫作**分代垃圾回收**的技术。它的工作原理是这样的：每次分配一个新对象时，它会进入堆中一个特殊的、相对较小的区域，称为“nursery”（意为托儿所）。由于对象倾向于早夭，所以垃圾回收器会在这个区域中的对象上被 <span
name="nursery_zh">频繁</span>调用。

<aside name="nursery_zh">

nursery通常也是要复制回收器进行管理，它在分配和释放对象方面比标记-清除回收器更快。

</aside>

GC在nursery的每次运行都被称为“一代”。任何不再需要的对象都会被释放。那些存活下来对象现在被认为老了一代，GC会为每个对象记录这一属性。如果一个对象存活了一定数量的代（通常只是一次回收），它就会被永久保留。此时，将它从nursery中复制处理，放入一个更大的、用于存放 长寿命对象的堆区域。垃圾回收器也会在这个区域内运行，但频率要低得多，因为这些对象中的大部分都很有可能还活着。

分代回收器是经验数据（观察到对象生命周期不是均匀分布的）以及利用这一事实的聪明算法设计的完美结合。它们在概念上也很简单。你可以把它看作是两个单独调优的GC和把对象从一个区域移到另一个区域的一个非常简单的策略。

</div>
