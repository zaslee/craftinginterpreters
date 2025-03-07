> Once we were blobs in the sea, and then fishes, and then lizards and rats and
> then monkeys, and hundreds of things in between. This hand was once a fin,
> this hand once had claws! In my human mouth I have the pointy teeth of a wolf
> and the chisel teeth of a rabbit and the grinding teeth of a cow! Our blood is
> as salty as the sea we used to live in! When we're frightened, the hair on our
> skin stands up, just like it did when we had fur. We are history! Everything
> we've ever been on the way to becoming us, we still are.
>
> 我们曾经是海里一团一团的东西，然后是鱼，然后是蜥蜴、老鼠、猴子，以及介于其间的数百种形态。这只手曾经是鳍，这只手曾经是爪子！在我人类的嘴里，有狼的尖牙，有兔子的凿齿，还有牛的磨牙！我们的血和我们曾经生活的大海一样咸！当我们受到惊吓时，我们皮肤上的毛发会竖起来，就像我们有毛时一样。我们就是历史！我们在成为我们的路上曾拥有的一切，我们仍然拥有。
>
> <cite>Terry Pratchett, <em>A Hat Full of Sky</em></cite>

Can you believe it? We've reached the last chapter of [Part II][]. We're almost
done with our first Lox interpreter. The [previous chapter][] was a big ball of
intertwined object-orientation features. I couldn't separate those from each
other, but I did manage to untangle one piece. In this chapter, we'll finish
off Lox's class support by adding inheritance.

[part ii]: a-tree-walk-interpreter.html
[previous chapter]: classes.html

Inheritance appears in object-oriented languages all the way back to the <span
name="inherited">first</span> one, [Simula][]. Early on, Kristen Nygaard and
Ole-Johan Dahl noticed commonalities across classes in the simulation programs
they wrote. Inheritance gave them a way to reuse the code for those similar
parts.
继承出现在面向对象语言中，可以追溯到第一种语言Simula。早些时候，克里斯汀 · 尼加德(Kristen Nygaard)和奥勒-约翰 · 达尔(Ole-Johan Dahl)注意到，在他们编写的模拟程序中，不同类之间存在共性。继承为他们提供了一种重用相似部分代码的方法。

[simula]: https://en.wikipedia.org/wiki/Simula

<aside name="inherited">

You could say all those other languages *inherited* it from Simula. Hey-ooo!
I'll, uh, see myself out.
可以说，所有其他语言都是从 Simula *继承* 来的。emmm，我先走了。

</aside>

## 超类和子类

Given that the concept is "inheritance", you would hope they would pick a
consistent metaphor and call them "parent" and "child" classes, but that would
be too easy. Way back when, C. A. R. Hoare coined the term "<span
name="subclass">subclass</span>" to refer to a record type that refines another
type. Simula borrowed that term to refer to a *class* that inherits from
another. I don't think it was until Smalltalk came along that someone flipped
the Latin prefix to get "superclass" to refer to the other side of the
relationship. From C++, you also hear "base" and "derived" classes. I'll mostly
stick with "superclass" and "subclass".
鉴于这个概念叫“继承”，你可能希望他们会选择一个一致的比喻，把类称为“父”类和“子”类，但这太简单了。早在很久以前，C. A. R. Hoare就创造了“subclass”这个术语，指的是完善另一种类型的记录类型。Simula借用了这个术语来指代一个继承自另一个类的类。我认为直到Smalltalk出现后，才有人将这个词的拉丁前缀取反义，用超类（superclass）指代这种关系的另一方。

<aside name="subclass">

"Super-" and "sub-" mean "above" and "below" in Latin, respectively. Picture an
inheritance tree like a family tree with the root at the top -- subclasses are
below their superclasses on the diagram. More generally, "sub-" refers to things
that refine or are contained by some more general concept. In zoology, a
subclass is a finer categorization of a larger class of living things.
"Super-" 和 "sub-" 在拉丁语中表示 "上面" 和 "下面" 。把继承树想象成一个根在顶部的家族树 -- 在这个图上，子类就在超类的下面。更一般地说，"sub-" 指的是细化或被更一般的概念所包含的事物。在动物学中，子类指的是对更大的生物类的一个精细分类。在集合论中，子集被一个更大的超集包含，该超集中包含子集的所有元素，可能还有更多元素。集合论和编程语言在类型论中相遇，就产生了 "超类" 和 "子类" 。在静态类型的面向对象语言中，一个子类通常也是其超类的一个子类。

In set theory, a subset is contained by a larger superset which has all of the
elements of the subset and possibly more. Set theory and programming languages
meet each other in type theory. There, you have "supertypes" and "subtypes".
在集合论中，一个子集被一个更大的超集所包含，这个超集拥有子集的所有元素，甚至可能更多。集合论和编程语言在类型理论中相遇。这里有 "超类" 和"子类"。

In statically typed object-oriented languages, a subclass is also often a
subtype of its superclass. Say we have a Doughnut superclass and a BostonCream
subclass. Every BostonCream is also an instance of Doughnut, but there may be
doughnut objects that are not BostonCreams (like Crullers).
在静态类型的面向对象语言中，子类通常也是其超类的子类。假设我们有一个 Doughnut 超类和一个 BostonCream 子类。每个 BostonCream 也是 Doughnut 的实例，但也可能存在不是 BostonCream 的甜甜圈对象（如: Crullers）。

Think of a type as the set of all values of that type. The set of all Doughnut
instances contains the set of all BostonCream instances since every BostonCream
is also a Doughnut. So BostonCream is a subclass, and a subtype, and its
instances are a subset. It all lines up.
将一个类型视为该类型所有值的集合。所有 Doughnut 实例的集合包含所有 BostonCream 实例的集合，因为每个 BostonCream 也是一个 Doughnut。因此，BostonCream 是一个子类，也是一个子类，它的实例也是一个子集。这一切都很吻合。

<img src="image/inheritance/doughnuts.png" alt="Boston cream &lt;: doughnut." />

</aside>

Our first step towards supporting inheritance in Lox is a way to specify a
superclass when declaring a class. There's a lot of variety in syntax for this.
C++ and C# place a `:` after the subclass's name, followed by the superclass
name. Java uses `extends` instead of the colon. Python puts the superclass(es)
in parentheses after the class name. Simula puts the superclass's name *before*
the `class` keyword.
我们在Lox中支持继承的第一步是找到声明类时指定超类的方法。这方面有很多不同的语法。C++和C#在子类的名字后面加一个`:`，然后是超类的名字。Java 使用 `extends` 而不是冒号。Python 则把超类放在类名后面的小括号里。Simula 把超类的名字放在关键字`class`之前。

This late in the game, I'd rather not add a new reserved word or token to the
lexer. We don't have `extends` or even `:`, so we'll follow Ruby and use a
less-than sign (`<`).
游戏已经到了后期，我宁愿不在词法分析器中添加新的保留字或标记。我们没有`extends`或`:`，所以我们遵循Ruby来使用小于号（`<`）。

```lox
class Doughnut {
  // General doughnut stuff...
}

class BostonCream < Doughnut {
  // Boston Cream-specific stuff...
}
```

To work this into the grammar, we add a new optional clause in our existing
`classDecl` rule.
为了在语法中实现这一点，我们在目前的`classDecl`规则中加一个新的可选子句。

```ebnf
classDecl      → "class" IDENTIFIER ( "<" IDENTIFIER )?
                 "{" function* "}" ;
```

After the class name, you can have a `<` followed by the superclass's name. The
superclass clause is optional because you don't *have* to have a superclass.
Unlike some other object-oriented languages like Java, Lox has no root "Object"
class that everything inherits from, so when you omit the superclass clause, the
class has *no* superclass, not even an implicit one.
在类的名称后面，可以有一个`<`，后跟超类的名称。超类子句是可选的，因为一个类不一定要有超类。与Java等面向对象的语言不同，Lox没有所有东西都继承的一个根“Object”类，所以当你省略超类子句时，该类就没有超类，甚至连隐含的都没有。

We want to capture this new syntax in the class declaration's AST node.
我们想在类声明的AST节点中捕捉这个新语法。

^code superclass-ast (1 before, 1 after)

You might be surprised that we store the superclass name as an Expr.Variable,
not a Token. The grammar restricts the superclass clause to a single identifier,
but at runtime, that identifier is evaluated as a variable access. Wrapping the
name in an Expr.Variable early on in the parser gives us an object that the
resolver can hang the resolution information off of.
你可能会惊讶，我们把超类的名字存为一个Expr.Variable，而不是一个Token。语法将一个超类子句限制为一个标识符，但是在运行时，这个标识符是当作变量访问来执行的。在解析器早期将名称封装在Expr.Variable内部，这样可以给我们提供一个对象，在分析器中可以将分析信息附加在其中。

The new parser code follows the grammar directly.
新的解析器代码直接遵循语法。

^code parse-superclass (1 before, 1 after)

Once we've (possibly) parsed a superclass declaration, we store it in the AST.
一旦我们（可能）解析到一个超类声明，就将其保存到AST节点中。

^code construct-class-ast (2 before, 1 after)

If we didn't parse a superclass clause, the superclass expression will be
`null`. We'll have to make sure the later passes check for that. The first of
those is the resolver.
如果我们没有解析到超类子句，超类表达式将是`null`。我们必须确保后面的操作会对其进行检查。首先是分析器。

^code resolve-superclass (1 before, 2 after)

The class declaration AST node has a new subexpression, so we traverse into and
resolve that. Since classes are usually declared at the top level, the
superclass name will most likely be a global variable, so this doesn't usually
do anything useful. However, Lox allows class declarations even inside blocks,
so it's possible the superclass name refers to a local variable. In that case,
we need to make sure it's resolved.
类声明的AST节点有一个新的子表达式，所以我们要遍历并分析它。因为类通常是在顶层声明的，超类的名称很可能是一个全局变量，所以这一步通常没有什么作用。然而，Lox运行在区块内的类声明，所以超类名称有可能指向一个局部变量。在那种情况下，我们需要保证能被它被分析。

Because even well-intentioned programmers sometimes write weird code, there's a
silly edge case we need to worry about while we're in here. Take a look at this:
即使是善意的程序员有时也会写出奇怪的代码，所以在这里我们需要考虑一个愚蠢的边缘情况。看看这个：

```lox
class Oops < Oops {}
```

There's no way this will do anything useful, and if we let the runtime try to
run this, it will break the expectation the interpreter has about there not
being cycles in the inheritance chain. The safest thing is to detect this case
statically and report it as an error.
这种代码不可能做什么有用的事情，如果我们尝试让运行时去执行它，将会打破解释器对继承链中没有循环的期望。最安全的做法是静态地检测这种情况，并将其作为一个错误报告出来。

^code inherit-self (2 before, 1 after)

Assuming the code resolves without error, the AST travels to the interpreter.
如果代码分析没有问题，AST节点就会被传递到解释器。

^code interpret-superclass (1 before, 1 after)

If the class has a superclass expression, we evaluate it. Since that could
potentially evaluate to some other kind of object, we have to check at runtime
that the thing we want to be the superclass is actually a class. Bad things
would happen if we allowed code like:
如果类中有超类表达式，我们就对其求值。因为我们可能会得到其它类型的对象，我们在运行时必须检查我们希望作为超类的对象是否确实是一个类。如果我们允许下面这样的代码，就会发生不好的事情：

```lox
var NotAClass = "I am totally not a class";

class Subclass < NotAClass {} // ?!
```

Assuming that check passes, we continue on. Executing a class declaration turns
the syntactic representation of a class -- its AST node -- into its runtime
representation, a LoxClass object. We need to plumb the superclass through to
that too. We pass the superclass to the constructor.
假设检查通过，我们继续。执行类声明语句会把类的语法表示（AST节点）转换为其运行时表示（一个LoxClass对象）。我们也需要把超类对象传入该类对象中。我们将超类传递给构造函数。

^code interpreter-construct-class (3 before, 1 after)

The constructor stores it in a field.
构造函数将它存储到一个字段中。

^code lox-class-constructor (1 after)

Which we declare here:
字段我们在这里声明：

^code lox-class-superclass-field (1 before, 1 after)

With that, we can define classes that are subclasses of other classes. Now, what
does having a superclass actually *do?*
有了这个，我们就可以定义一个类作为其它类的子类。现在，拥有一个超类究竟有什么用呢？

## 继承方法

Inheriting from another class means that everything that's <span
name="liskov">true</span> of the superclass should be true, more or less, of the
subclass. In statically typed languages, that carries a lot of implications. The
sub*class* must also be a sub*type*, and the memory layout is controlled so that
you can pass an instance of a subclass to a function expecting a superclass and
it can still access the inherited fields correctly.
继承自另一个类，意味着对于超类适用的一切，对于子类或多或少也应该适用。在静态类型的语言中，这包含了很多含义。子类也必须是一个子类，而且内存布局是可控的，这样你就可以把一个子类实例传递给一个期望超类的函数，而它仍然可以正确地访问继承的字段。

<aside name="liskov">

A fancier name for this hand-wavey guideline is the [*Liskov substitution
principle*][liskov]. Barbara Liskov introduced it in a keynote during the
formative period of object-oriented programming.
这种 "手把手" 的指导原则有一个更响亮的名字，叫做 "利斯科夫替换原则"（[*Liskov substitution
principle*][liskov]）。Barbara Liskov 在面向对象编程形成时期的一次主题演讲中介绍了这一原则。

[liskov]: https://en.wikipedia.org/wiki/Liskov_substitution_principle

</aside>

Lox is a dynamically typed language, so our requirements are much simpler.
Basically, it means that if you can call some method on an instance of the
superclass, you should be able to call that method when given an instance of the
subclass. In other words, methods are inherited from the superclass.
Lox是一种动态类型的语言，所以我们的要求要简单得多。基本上，这意味着如果你能在超类的实例上调用某些方法，那么当给你一个子类的实例时，你也应该能调用这个方法。换句话说，方法是从超类继承的。

This lines up with one of the goals of inheritance -- to give users a way to
reuse code across classes. Implementing this in our interpreter is
astonishingly easy.
这符合继承的目标之一 -- 为用户提供一种跨类重用代码的方式。在我们的解释器中实现这一点是非常容易的。

^code find-method-recurse-superclass (3 before, 1 after)

That's literally all there is to it. When we are looking up a method on an
instance, if we don't find it on the instance's class, we recurse up through the
superclass chain and look there. Give it a try:
这就是它的全部内容。当我们在一个实例上查找一个方法时，如果我们在实例的类中找不到它，就沿着超类继承链递归查找。试一下这个：

```lox
class Doughnut {
  cook() {
    print "Fry until golden brown.";
  }
}

class BostonCream < Doughnut {}

BostonCream().cook();
```

There we go, half of our inheritance features are complete with only three lines
of Java code.
好了，一半的继承特性只用了三行Java代码就完成了。

## 调用超类方法

In `findMethod()` we look for a method on the current class *before* walking up
the superclass chain. If a method with the same name exists in both the subclass
and the superclass, the subclass one takes precedence or **overrides** the
superclass method. Sort of like how variables in inner scopes shadow outer ones.
在`findMethod()`方法中，我们首先在当前类中查找，然后遍历超类链。如果在子类和超类中包含相同的方法，那么子类中的方法将优先于或**覆盖**超类的方法。这有点像内部作用域中的变量对外部作用域的遮蔽。

That's great if the subclass wants to *replace* some superclass behavior
completely. But, in practice, subclasses often want to *refine* the superclass's
behavior. They want to do a little work specific to the subclass, but also
execute the original superclass behavior too.
如果子类想要完全*替换*超类的某些行为，那就正好。但是，在实践中，子类通常想改进超类的行为。他们想要做一些专门针对子类的操作，但是也想要执行原来超类中的行为。

However, since the subclass has overridden the method, there's no way to refer
to the original one. If the subclass method tries to call it by name, it will
just recursively hit its own override. We need a way to say "Call this method,
but look for it directly on my superclass and ignore my override". Java uses
`super` for this, and we'll use that same syntax in Lox. Here is an example:
然而，由于子类已经重写了该方法，所有没有办法指向原始的方法。如果子类的方法试图通过名字来调用它，将会递归到自身的重写方法上。我们需要一种方式来表明“调用这个方法，但是要直接在我的超类上寻找，忽略我内部的重写方法”。Java中使用`super`实现这一点，我们在Lox中使用相同的语法。下面是一个例子：

```lox
class Doughnut {
  cook() {
    print "Fry until golden brown.";
  }
}

class BostonCream < Doughnut {
  cook() {
    super.cook();
    print "Pipe full of custard and coat with chocolate.";
  }
}

BostonCream().cook();
```

If you run this, it should print:
如果你运行该代码，应该打印出：

```text
Fry until golden brown.
Pipe full of custard and coat with chocolate.
```

We have a new expression form. The `super` keyword, followed by a dot and an
identifier, looks for a method with that name. Unlike calls on `this`, the search
starts at the superclass.
我们有了一个新的表达式形式。`super`关键字，后跟一个点和一个标识符，以使用该名称查找方法。与`this`调用不同，该搜索是从超类开始的。

### 语法

With `this`, the keyword works sort of like a magic variable, and the expression
is that one lone token. But with `super`, the subsequent `.` and property name
are inseparable parts of the `super` expression. You can't have a bare `super`
token all by itself.
在`this`使用中，关键字有点像一个魔法变量，而表达式是一个单独的标记。但是对于`super`，随后的`.`和属性名是`super`表达式不可分割的一部分。你不可能只有一个单独的`super`标记。

```lox
print super; // Syntax error.
```

So the new clause we add to the `primary` rule in our grammar includes the
property access as well.
因此，我们在语法中的`primary`规则添加新子句时要包含属性访问。

```ebnf
primary        → "true" | "false" | "nil" | "this"
               | NUMBER | STRING | IDENTIFIER | "(" expression ")"
               | "super" "." IDENTIFIER ;
```

Typically, a `super` expression is used for a method call, but, as with regular
methods, the argument list is *not* part of the expression. Instead, a super
*call* is a super *access* followed by a function call. Like other method calls,
you can get a handle to a superclass method and invoke it separately.
通常情况下，`super`表达式用于方法调用，但是，与普通方法一样，参数列表并不是表达式的一部分。相反，`super`调用是一个`super`属性访问，然后跟一个函数调用。与其它方法调用一样，你可以获得超类方法的句柄，然后单独运行它。

```lox
var method = super.cook;
method();
```

So the `super` expression itself contains only the token for the `super` keyword
and the name of the method being looked up. The corresponding <span
name="super-ast">syntax tree node</span> is thus:
因此，`super`表达式本身只包含`super`关键字和要查找的方法名称。对应的语法树节点为：

^code super-expr (1 before, 1 after)

<aside name="super-ast">

The generated code for the new node is in [Appendix II][appendix-super].
新节点的生成代码见[附录 II][appendix-super]。

[appendix-super]: appendix-ii.html#super-expression

</aside>

Following the grammar, the new parsing code goes inside our existing `primary()`
method.
按照语法，需要在我们现有的`primary`方法中添加新代码。

^code parse-super (2 before, 2 after)

A leading `super` keyword tells us we've hit a `super` expression. After that we
consume the expected `.` and method name.
开头的`super`关键字告诉我们遇到了一个`super`表达式，之后我们消费预期中的`.`和方法名称。

### 语义

Earlier, I said a `super` expression starts the method lookup from "the
superclass", but *which* superclass? The naïve answer is the superclass of
`this`, the object the surrounding method was called on. That coincidentally
produces the right behavior in a lot of cases, but that's not actually correct.
Gaze upon:
之前，我说过`super`表达式从“超类”开始查找方法，但是是哪个超类？一个不太成熟的答案是方法被调用时的外围对象`this`的超类。在很多情况下，这碰巧产生了正确的行为，但实际上这是不正确的。请看：

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
Translate this program to Java, C#, or C++ and it will print "A method", which
is what we want Lox to do too. When this program runs, inside the body of
`test()`, `this` is an instance of C. The superclass of C is B, but that is
*not* where the lookup should start. If it did, we would hit B's `method()`.
将这个程序转换为Java、c#或c++，它将输出“A method”，这也是我们希望Lox做的。当这个程序运行时，在`test`方法体中，`this`是C的一个实例，C是超类是B，但这不是查找应该开始的地方。如果是这样，我们就会命中B的`method()`。

Instead, lookup should start on the superclass of *the class containing the
`super` expression*. In this case, since `test()` is defined inside B, the
`super` expression inside it should start the lookup on *B*&rsquo;s superclass
-- A.
相反，查找应该从包含`super`表达式的类的超类开始。在这个例子中，由于`test()`是在B中定义的，它内部的`super`表达式应该在B的超类A中开始查找。

<span name="flow"></span>

<img src="image/inheritance/classes.png" alt="The call chain flowing through the classes." />

<aside name="flow">

The execution flow looks something like this:
执行流程看起来是这样的：

1. We call `test()` on an instance of C.
我们在C的一个实例上调用`test()`。

2. That enters the `test()` method inherited from B. That calls
   `super.method()`.
这就进入了从B中继承的`test()`方法，其中又会调用`super.method()`。

3. The superclass of B is A, so that chains to `method()` on A, and the program
   prints "A method".
   B的超类是A，所以链接到A中的`method()`，程序会打印出“A method”。

</aside>

Thus, in order to evaluate a `super` expression, we need access to the
superclass of the class definition surrounding the call. Alack and alas, at the
point in the interpreter where we are executing a `super` expression, we don't
have that easily available.
因此，为了对`super`表达式求值，我们需要访问围绕方法调用的类的超类。可惜的是，在解释器中执行`super`表达式的地方，我们并没有那么容易获得。

We *could* add a field to LoxFunction to store a reference to the LoxClass that
owns that method. The interpreter would keep a reference to the
currently executing LoxFunction so that we could look it up later when we hit a
`super` expression. From there, we'd get the LoxClass of the method, then its
superclass.
我们可以从LoxFunction添加一个字段，以存储指向拥有该方法的LoxClass的引用。解释器会保存当前正在执行的LoxFunction的引用，这样稍后在遇到`super`表达式时就可以找到它。从它开始，可以得到方法的LoxClass，然后找到它的超类。

That's a lot of plumbing. In the [last chapter][], we had a similar problem when
we needed to add support for `this`. In that case, we used our existing
environment and closure mechanism to store a reference to the current object.
Could we do something similar for storing the superclass<span
name="rhetorical">?</span> Well, I probably wouldn't be talking about it if the
answer was no, so... yes.
这需要很多管道。在上一章中，我们添加对`this`的支持时遇到了类似的问题。在那种情况下，我们使用已有的环境和闭包机制保存了指向当前对象的引用。那我们是否可以做类似的事情来存储超类？嗯，如果答案是否定的，我就不会问这个问题了，所以……是的。

<aside name="rhetorical">

Does anyone even like rhetorical questions?
有人喜欢反问句吗？

</aside>

[last chapter]: classes.html

One important difference is that we bound `this` when the method was *accessed*.
The same method can be called on different instances and each needs its own
`this`. With `super` expressions, the superclass is a fixed property of the
*class declaration itself*. Every time you evaluate some `super` expression, the
superclass is always the same.
一个重要的区别是，我们在方法被访问时绑定了`this`。同一个方法可以在不同的实例上被调用，而且每个实例都需要有自己的`this`。对于`super`表达式，超类是*类声明本身*的一个固定属性。每次对某个`super`表达式求值时，超类都是同一个。

That means we can create the environment for the superclass once, when the class
definition is executed. Immediately before we define the methods, we make a new
environment to bind the class's superclass to the name `super`.
这意味着我们可以在执行类定义时，为超类创建一个环境。在定义方法之前，我们创建一个新环境，将类的超类与名称`super`绑定。

<img src="image/inheritance/superclass.png" alt="The superclass environment." />

When we create the LoxFunction runtime representation for each method, that is
the environment they will capture in their closure. Later, when a method is
invoked and `this` is bound, the superclass environment becomes the parent for
the method's environment, like so:
当我们为每个方法创建LoxFunction运行时表示时，也就是这个方法闭包中获取的环境。之后，放方法被调用时会绑定`this`，超类环境会成为方法环境的父环境，就像这样：

<img src="image/inheritance/environments.png" alt="The environment chain including the superclass environment." />

That's a lot of machinery, but we'll get through it a step at a time. Before we
can get to creating the environment at runtime, we need to handle the
corresponding scope chain in the resolver.
这是一个复杂的机制，但是我们会一步一步完成它。在我们可以在运行时创建环境之前，我们需要在分析器中处理对应的作用域。

^code begin-super-scope (2 before, 2 after)

If the class declaration has a superclass, then we create a new scope
surrounding all of its methods. In that scope, we define the name "super". Once
we're done resolving the class's methods, we discard that scope.
如果该类声明有超类，那么我们就在其所有方法的外围创建一个新的作用域。在这个作用域中，我们会定义名称`super`。一旦我们完成了对该类中方法的分析，就丢弃这个作用域。

^code end-super-scope (2 before, 1 after)

It's a minor optimization, but we only create the superclass environment if the
class actually *has* a superclass. There's no point creating it when there isn't
a superclass since there'd be no superclass to store in it anyway.
这是一个小优化，但是我们只在类真的有超类时才会创建超类环境。在没有超类的情况下，创建超类环境是没有意义的，因为无论如何里面都不会存储超类。

With "super" defined in a scope chain, we are able to resolve the `super`
expression itself.
在作用域链中定义`super`后，我们就能够分析`super`表达式了。

^code resolve-super-expr

We resolve the `super` token exactly as if it were a variable. The resolution
stores the number of hops along the environment chain that the interpreter needs
to walk to find the environment where the superclass is stored.
我们把`super`标记当作一个变量进行分析。分析结果保存了解释器要在环境链上找到超类所在的环境需要的跳数。

This code is mirrored in the interpreter. When we evaluate a subclass
definition, we create a new environment.
这段代码在解释器中也有对应。当我们执行子类定义时，创建一个新环境。

^code begin-superclass-environment (6 before, 2 after)

Inside that environment, we store a reference to the superclass -- the actual
LoxClass object for the superclass which we have now that we are in the runtime.
Then we create the LoxFunctions for each method. Those will capture the current
environment -- the one where we just bound "super" -- as their closure, holding
on to the superclass like we need. Once that's done, we pop the environment.
在这个环境中，我们保存指向超类的引用 -- 即我们在运行时现在拥有的超类的实际LoxClass对象。然后我们为每个方法创建LoxFunction。这些函数将捕获当前环境（也就是我们刚刚绑定“super”的那个）作为其闭包，像我们需要的那样维系着超类。一旦这些完成，我们就弹出环境。

^code end-superclass-environment (2 before, 2 after)

We're ready to interpret `super` expressions themselves. There are a few moving
parts, so we'll build this method up in pieces.
我们现在已经准备好解释`super`表达式了。这会分为很多部分，所以我们逐步构建这个方法。

^code interpreter-visit-super

First, the work we've been leading up to. We look up the surrounding class's
superclass by looking up "super" in the proper environment.
首先，我们要做之前铺垫的工作。我们通过在适当环境中查找“super”来找到外围类的超类。

When we access a method, we also need to bind `this` to the object the method is
accessed from. In an expression like `doughnut.cook`, the object is whatever we
get from evaluating `doughnut`. In a `super` expression like `super.cook`, the
current object is implicitly the *same* current object that we're using. In
other words, `this`. Even though we are looking up the *method* on the
superclass, the *instance* is still `this`.
当我们访问方法时，还需要将`this`与访问该方法的对象进行绑定。在像`doughnut.cook`这样的表达式中，对象是我们通过对`doughnut`求值得到的内容。在像`super.cook`这样的`super`表达式中，当前对象隐式地与我们正使用的当前对象相同。换句话说，就是`this`。即使我们在超类中查找方法，*实例*仍然是`this`。

Unfortunately, inside the `super` expression, we don't have a convenient node
for the resolver to hang the number of hops to `this` on. Fortunately, we do
control the layout of the environment chains. The environment where "this" is
bound is always right inside the environment where we store "super".
不幸的是，在`super`表达式中，我们没有一个方便的节点可以让分析器将`this`对应的跳数保存起来。幸运的是，我们可以控制环境链的布局。绑定`this`的环境总是存储在保存`super`的环境中。

^code super-find-this (2 before, 1 after)

Offsetting the distance by one looks up "this" in that inner environment. I
admit this isn't the most <span name="elegant">elegant</span> code, but it
works.
将距离偏移1，在那个内部环境中查找“this”。我承认这个代码不是最优雅的，但是它是有效的。

<aside name="elegant">

Writing a book that includes every single line of code for a program means I
can't hide the hacks by leaving them as an "exercise for the reader".
写一本包含程序每一行代码的书，意味着我不能把充满黑科技的代码作为 "读者练习" 来隐藏它们。

</aside>

Now we're ready to look up and bind the method, starting at the superclass.
现在我们准备查找并绑定方法，从超类开始。

^code super-find-method (2 before, 1 after)

This is almost exactly like the code for looking up a method of a get
expression, except that we call `findMethod()` on the superclass instead of on
the class of the current object.
这几乎与查找get表达式方法的代码完全一样，区别在于，我们是在超类上调用`findMethod()` ，而不是在当前对象的类。

That's basically it. Except, of course, that we might *fail* to find the method.
So we check for that too.
基本上就是这样了。当然，除了我们可能找不到方法之外。所以，我们要对其检查。

^code super-no-method (2 before, 2 after)

There you have it! Take that BostonCream example earlier and give it a try.
Assuming you and I did everything right, it should fry it first, then stuff it
with cream.
这就对了！试着运行一下前面那个BostonCream的例子。如果你我都做对了，它的结果应该是：

### super的无效使用

As with previous language features, our implementation does the right thing when
the user writes correct code, but we haven't bulletproofed the intepreter
against bad code. In particular, consider:
像以前的语言特性一样，当用户写出正确的代码时，我们的语言实现也会做成正确的事情，但我们还没有在解释器中对错误代码进行防御。具体来说，考虑以下代码：

```lox
class Eclair {
  cook() {
    super.cook();
    print "Pipe full of crème pâtissière.";
  }
}
```

This class has a `super` expression, but no superclass. At runtime, the code for
evaluating `super` expressions assumes that "super" was successfully resolved
and will be found in the environment. That's going to fail here because there is
no surrounding environment for the superclass since there is no superclass. The
JVM will throw an exception and bring our interpreter to its knees.
这个类中有一个`super`表达式，但是没有超类。在运行时，计算`super`表达式的代码假定`super`已经被成功分析，并且可以在环境中找到超类。但是在这里会失败，因为没有超类，也就没有超类对应的外围环境。JVM会抛出一个异常，我们的解释器也会因此崩溃。

Heck, there are even simpler broken uses of super:
见鬼，还有更简单的`super`错误用法：

```lox
super.notEvenInAClass();
```

We could handle errors like these at runtime by checking to see if the lookup
of "super" succeeded. But we can tell statically -- just by looking at the
source code -- that Eclair has no superclass and thus no `super` expression will
work inside it. Likewise, in the second example, we know that the `super`
expression is not even inside a method body.
我们可以在运行时通过检查“super”是否查找成功而处理此类错误。但是我们可以只通过查看源代码静态地知道，Eclair没有超类，因此也就没有`super`表达式可以在其中生效。同样的，在第二个例子中，我们知道`super`表达式甚至不在方法体内。

Even though Lox is dynamically typed, that doesn't mean we want to defer
*everything* to runtime. If the user made a mistake, we'd like to help them find
it sooner rather than later. So we'll report these errors statically, in the
resolver.
尽管Lox是动态类型的，但这并不意味着我们要将一切都推迟到运行时。如果用户犯了错误，我们希望能帮助他们尽早发现，所以我们会在分析器中静态地报告这些错误。

First, we add a new case to the enum we use to keep track of what kind of class
is surrounding the current code being visited.
首先，在我们用来追踪当前访问代码外围类的类型的枚举中添加一个新值。

^code class-type-subclass (1 before, 1 after)

We'll use that to distinguish when we're inside a class that has a superclass
versus one that doesn't. When we resolve a class declaration, we set that if the
class is a subclass.
我们将用它来区分我们是否在一个有超类的类中。当我们分析一个类的声明时，如果该类是一个子类，我们就设置该值。

^code set-current-subclass (1 before, 1 after)

Then, when we resolve a `super` expression, we check to see that we are
currently inside a scope where that's allowed.
然后，当我们分析`super`表达式时，会检查当前是否在一个允许使用`super`表达式的作用域中。

^code invalid-super (1 before, 1 after)

If not -- oopsie! -- the user made a mistake.
如果不是，那就是用户出错了。

## 总结

We made it! That final bit of error handling is the last chunk of code needed to
complete our Java implementation of Lox. This is a real <span
name="superhero">accomplishment</span> and one you should be proud of. In the
past dozen chapters and a thousand or so lines of code, we have learned and
implemented...
我们成功了！最后的错误处理是完成Lox语言的Java实现所需的最后一块代码。这是一项真正的成就，你应该为此感到自豪。在过去的十几章和一千多行代码中，我们已经学习并实现了：

* [tokens and lexing][4], 标记与词法
* [abstract syntax trees][5], 抽象语法树
* [recursive descent parsing][6], 递归下降分析
* prefix and infix expressions, 前缀、中缀表达式
* runtime representation of objects, 对象的运行时表示
* [interpreting code using the Visitor pattern][7], 使用Visitor模式解释代码
* [lexical scope][8], 词法作用域
* environment chains for storing variables, 保存变量的环境链
* [control flow][9], 控制流
* [functions with parameters][10], 有参函数
* closures, 闭包
* [static variable resolution and error detection][11], 静态变量分析与错误检查
* [classes][12], 类
* constructors, 构造函数
* fields, 字段
* methods, and finally, 方法
* inheritance. 继承

[4]: scanning.html
[5]: representing-code.html
[6]: parsing-expressions.html
[7]: evaluating-expressions.html
[8]: statements-and-state.html
[9]: control-flow.html
[10]: functions.html
[11]: resolving-and-binding.html
[12]: classes.html

<aside name="superhero">

<img src="image/inheritance/superhero.png" alt="You, being your bad self." />

</aside>

We did all of that from scratch, with no external dependencies or magic tools.
Just you and I, our respective text editors, a couple of collection classes in
the Java standard library, and the JVM runtime.
所有这些都是我们从头开始做的，没有借助外部依赖和神奇工具。只有你和我，我们的文本编辑器，Java标准库中的几个集合类，以及JVM运行时。

This marks the end of Part II, but not the end of the book. Take a break. Maybe
write a few fun Lox programs and run them in your interpreter. (You may want to
add a few more native methods for things like reading user input.) When you're
refreshed and ready, we'll embark on our [next adventure][].
这标志着第二部分的结束，但不是这本书的结束。休息一下，也许可以编写几个Lox程序在你的解释器中运行一下（你可能需要添加一些本地方法来支持读取用户的输入等操作）。当你重新振作之后，我们将开始下一次冒险。

[next adventure]: a-bytecode-virtual-machine.html

<div class="challenges">

## Challenges

1.  Lox supports only *single inheritance* -- a class may have a single
    superclass and that's the only way to reuse methods across classes. Other
    languages have explored a variety of ways to more freely reuse and share
    capabilities across classes: mixins, traits, multiple inheritance, virtual
    inheritance, extension methods, etc.
    Lox只支持*单继承* -- 一个类可以有一个超类，这是唯一跨类复用方法的方式。其它语言中已经探索出了各种方法来更自由地跨类重用和共享功能：mixins, traits, multiple inheritance, virtual inheritance, extension methods, 等等。

    If you were to add some feature along these lines to Lox, which would you
    pick and why? If you're feeling courageous (and you should be at this
    point), go ahead and add it.
    如果你要在Lox中添加一些类似的功能，你会选择哪种，为什么？如果你有勇气的话（这时候你应该有勇气了），那就去添加它。

1.  In Lox, as in most other object-oriented languages, when looking up a
    method, we start at the bottom of the class hierarchy and work our way up --
    a subclass's method is preferred over a superclass's. In order to get to the
    superclass method from within an overriding method, you use `super`.
    在Lox中，与其它大多数面向对象语言一样，当查找一个方法时，我们从类的底层开始向上查找 -- 子类的方法优先于超类的方法。为了在覆盖方法中访问超类方法，你可以使用`super`。

    The language [BETA][] takes the [opposite approach][inner]. When you call a
    method, it starts at the *top* of the class hierarchy and works *down*. A
    superclass method wins over a subclass method. In order to get to the
    subclass method, the superclass method can call `inner`, which is sort of
    like the inverse of `super`. It chains to the next method down the
    hierarchy.
    [BEAT](https://beta.cs.au.dk/)语言采用了[相反的方法](http://journal.stuffwithstuff.com/2012/12/19/the-impoliteness-of-overriding-methods/)。当你调用一个方法时，它从类继承结构的顶层开始向下寻找。超类方法的优先级高于子类方法。为了访问子类的方法，超类方法可以调用`inner`，这有点像是`super`的反义词。它与继承层次结构中的下一级方法相连接。

    The superclass method controls when and where the subclass is allowed to
    refine its behavior. If the superclass method doesn't call `inner` at all,
    then the subclass has no way of overriding or modifying the superclass's
    behavior.
    超类方法控制着子类何时何地可以改进其行为。如果超类方法根本没有调用`inner`，那么子类就无法覆盖或修改超类的行为。

    Take out Lox's current overriding and `super` behavior and replace it with
    BETA's semantics. In short:
    去掉Lox目前的覆盖和`super`行为，用BEAT的语义来替换。简而言之：

    *   When calling a method on a class, prefer the method *highest* on the
        class's inheritance chain.
        当调用类上的方法时，优先选择类继承链中最高的方法。

    *   Inside the body of a method, a call to `inner` looks for a method with
        the same name in the nearest subclass along the inheritance chain
        between the class containing the `inner` and the class of `this`. If
        there is no matching method, the `inner` call does nothing.
        在方法体内部，`inner`调用会在继承链中包含`inner`的类和包含`this`的类之间，查找具有相同名称的最近的子类中的方法。如果没有匹配的方法，`inner`调用不做任何事情。

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
    这应该输出：

    ```text
    Fry until golden brown.
    Pipe full of custard and coat with chocolate.
    Place in a nice box.
    ```

1.  In the chapter where I introduced Lox, [I challenged you][challenge] to
    come up with a couple of features you think the language is missing. Now
    that you know how to build an interpreter, implement one of those features.
    在介绍Lox的那一章，我让你想出几个你认为该语言缺少的功能。现在你知道了如何构建一个解释器，请实现其中的一个功能。

[challenge]: the-lox-language.html#challenges
[inner]: http://journal.stuffwithstuff.com/2012/12/19/the-impoliteness-of-overriding-methods/
[beta]: https://beta.cs.au.dk/

</div>


