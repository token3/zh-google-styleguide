6. 其他 C++ 特性
----------------------------

6.1. 右值引用
~~~~~~~~~~~~~~~~~~~~~~

.. tip::

    仅在下面列出的某些特殊情况下使用右值引用.

定义:

	右值引用是一种能绑定到右值表达式的引用类型, 其语法与传统的引用语法相似. 例如, ``void f(string&& s)``; 声明了一个其参数是一个字符串的右值引用的函数.

    当标记“ ``&&`` ”应用于函数参数中的非限定模板参数时，将应用特殊的模板参数推导规则。这种引用称为转发引用.

优点:

	用于定义移动构造函数 (使用类的右值引用进行构造的函数) 使得移动一个值而非拷贝之成为可能. 例如, 如果 ``v1`` 是一个 ``vector<string>``, 则 ``auto v2(std::move(v1))`` 将很可能不再进行大量的数据复制而只是简单地进行指针操作, 在某些情况下这将带来大幅度的性能提升.
	
	右值引用使得编写通用的函数封装来转发其参数到另外一个函数成为可能, 无论其参数是否是临时对象都能正常工作.
	
	右值引用能实现可移动但不可拷贝的类型, 这一特性对那些在拷贝方面没有实际需求, 但有时又需要将它们作为函数参数传递或塞入容器的类型很有用.
	
	要高效率地使用某些标准库类型, 例如 ``std::unique_ptr``, ``std::move`` 是必需的.
	
缺点:
	
	右值引用是一个相对比较新的特性 (由 C++11 引入), 它尚未被广泛理解. 类似引用崩溃, 移动构造函数的自动推导这样的规则都是很复杂的.
	
结论:

	只在定义移动构造函数与移动赋值操作时使用右值引用, 不要使用 ``std::forward`` 功能函数. 你可能会使用 ``std::move`` 来表示将值从一个对象移动而不是复制到另一个对象. 

.. _function-overloading:

6.2. 函数重载
~~~~~~~~~~~~~~~~~~~~~~

.. tip::

    若要用好函数重载，最好能让读者一看调用点（call site）就胸有成竹，不用花心思猜测调用的重载函数到底是哪一种。该规则适用于构造函数。

定义:

    你可以编写一个参数类型为 ``const string&`` 的函数, 然后用另一个参数类型为 ``const char*`` 的函数重载它:

        .. code-block:: c++

            class MyClass {
                public:
                void Analyze(const string &text);
                void Analyze(const char *text, size_t textlen);
            };

优点:

    通过重载参数不同的同名函数, 令代码更加直观. 模板化代码需要重载, 同时为使用者带来便利.

缺点:

    如果函数单单靠不同的参数类型而重载（acgtyrant 注：这意味着参数数量不变），读者就得十分熟悉 C++ 五花八门的匹配规则，以了解匹配过程具体到底如何。另外，当派生类只重载了某个函数的部分变体，继承语义容易令人困惑。

结论:

    如果您打算重载一个函数, 可以试试改在函数名里加上参数信息。例如，用 ``AppendString()`` 和 ``AppendInt()`` 等， 而不是一口气重载多个 ``Append()``.

6.3. 缺省参数
~~~~~~~~~~~~~~~~~~~~~~

.. tip::

    我们不允许使用缺省函数参数，少数极端情况除外。尽可能改用函数重载。

优点:

    当您有依赖缺省参数的函数时，您也许偶尔会修改修改这些缺省参数。通过缺省参数，不用再为个别情况而特意定义一大堆函数了。与函数重载相比，缺省参数语法更为清晰，代码少，也很好地区分了「必选参数」和「可选参数」。

缺点:

    缺省参数会干扰函数指针，害得后者的函数签名（function signature）往往对不上所实际要调用的函数签名。即在一个现有函数添加缺省参数，就会改变它的类型，那么调用其地址的代码可能会出错，不过函数重载就没这问题了。此外，缺省参数会造成臃肿的代码，毕竟它们在每一个调用点（call site）都有重复（acgtyrant 注：我猜可能是因为调用函数的代码表面上看来省去了不少参数，但编译器在编译时还是会在每一个调用代码里统统补上所有默认实参信息，造成大量的重复）。函数重载正好相反，毕竟它们所谓的「缺省参数」只会出现在函数定义里。

结论:

    由于缺点并不是很严重，有些人依旧偏爱缺省参数胜于函数重载。所以除了以下情况，我们要求必须显式提供所有参数（acgtyrant 注：即不能再通过缺省参数来省略参数了）。

    其一，位于 ``.cc`` 文件里的静态函数或匿名空间函数，毕竟都只能在局部文件里调用该函数了。

    其二，可以在构造函数里用缺省参数，毕竟不可能取得它们的地址。

    其三，可以用来模拟变长数组。

        .. code-block:: c++

            // 通过空 AlphaNum 以支持四个形参
            string StrCat(const AlphaNum &a,
                          const AlphaNum &b = gEmptyAlphaNum,
                          const AlphaNum &c = gEmptyAlphaNum,
                          const AlphaNum &d = gEmptyAlphaNum);

6.4. 变长数组和 alloca()
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

.. tip::

    我们不允许使用变长数组和 ``alloca()``.

优点:

    变长数组具有浑然天成的语法. 变长数组和 ``alloca()`` 也都很高效.

缺点:

    变长数组和 ``alloca()`` 不是标准 C++ 的组成部分. 更重要的是, 它们根据数据大小动态分配堆栈内存, 会引起难以发现的内存越界 bugs: "在我的机器上运行的好好的, 发布后却莫名其妙的挂掉了".

结论:

    改用更安全的分配器（allocator），就像 ``std::vector`` 或 ``std::unique_ptr<T[]>``.

6.5. 友元
~~~~~~~~~~~~~~~~

.. tip::

    我们允许合理的使用友元类及友元函数.

通常友元应该定义在同一文件内, 避免代码读者跑到其它文件查找使用该私有成员的类. 经常用到友元的一个地方是将 ``FooBuilder`` 声明为 ``Foo`` 的友元, 以便 ``FooBuilder`` 正确构造 ``Foo`` 的内部状态, 而无需将该状态暴露出来. 某些情况下, 将一个单元测试类声明成待测类的友元会很方便.

友元扩大了 (但没有打破) 类的封装边界. 某些情况下, 相对于将类成员声明为 ``public``, 使用友元是更好的选择, 尤其是如果你只允许另一个类访问该类的私有成员时. 当然, 大多数类都只应该通过其提供的公有成员进行互操作.

.. _exceptions:

6.6. 异常
~~~~~~~~~~~~~~~~

.. tip::

    我们不使用 C++ 异常.

优点:

    - 异常允许应用高层决定如何处理在底层嵌套函数中「不可能发生」的失败（failures），不用管那些含糊且容易出错的错误代码（acgtyrant 注：error code, 我猜是Ｃ语言函数返回的非零 int 值）。

    - 很多现代语言都用异常。引入异常使得 C++ 与 Python, Java 以及其它类 C++ 的语言更一脉相承。

    - 有些第三方 C++ 库依赖异常，禁用异常就不好用了。

    - 异常是处理构造函数失败的唯一途径。虽然可以用工厂函数（acgtyrant 注：factory function, 出自 C++ 的一种设计模式，即「简单工厂模式」）或 ``Init()`` 方法代替异常, 但是前者要求在堆栈分配内存，后者会导致刚创建的实例处于 ”无效“ 状态。

    - 在测试框架里很好用。

缺点:

    - 在现有函数中添加 ``throw`` 语句时，您必须检查所有调用点。要么让所有调用点统统具备最低限度的异常安全保证，要么眼睁睁地看异常一路欢快地往上跑，最终中断掉整个程序。举例，``f()`` 调用 ``g()``, ``g()`` 又调用 ``h()``, 且 ``h`` 抛出的异常被 ``f`` 捕获。当心 ``g``, 否则会没妥善清理好。

    - 还有更常见的，异常会彻底扰乱程序的执行流程并难以判断，函数也许会在您意料不到的地方返回。您或许会加一大堆何时何处处理异常的规定来降低风险，然而开发者的记忆负担更重了。
    
    - 异常安全需要RAII和不同的编码实践. 要轻松编写出正确的异常安全代码需要大量的支持机制. 更进一步地说, 为了避免读者理解整个调用表, 异常安全必须隔绝从持续状态写到 "提交" 状态的逻辑. 这一点有利有弊 (因为你也许不得不为了隔离提交而混淆代码). 如果允许使用异常, 我们就不得不时刻关注这样的弊端, 即使有时它们并不值得.

    - 启用异常会增加二进制文件数据，延长编译时间（或许影响小），还可能加大地址空间的压力。

    - 滥用异常会变相鼓励开发者去捕捉不合时宜，或本来就已经没法恢复的「伪异常」。比如，用户的输入不符合格式要求时，也用不着抛异常。如此之类的伪异常列都列不完。

结论:

    从表面上看来，使用异常利大于弊, 尤其是在新项目中. 但是对于现有代码, 引入异常会牵连到所有相关代码. 如果新项目允许异常向外扩散, 在跟以前未使用异常的代码整合时也将是个麻烦. 因为 Google 现有的大多数 C++ 代码都没有异常处理, 引入带有异常处理的新代码相当困难.

    鉴于 Google 现有代码不接受异常, 在现有代码中使用异常比在新项目中使用的代价多少要大一些. 迁移过程比较慢, 也容易出错. 我们不相信异常的使用有效替代方案, 如错误代码, 断言等会造成严重负担.

    我们并不是基于哲学或道德层面反对使用异常, 而是在实践的基础上. 我们希望在 Google 使用我们自己的开源项目, 但项目中使用异常会为此带来不便, 因此我们也建议不要在 Google 的开源项目中使用异常. 如果我们需要把这些项目推倒重来显然不太现实.

    对于 Windows 代码来说, 有个 :ref:`特例 <windows-code>`.

(YuleFox 注: 对于异常处理, 显然不是短短几句话能够说清楚的, 以构造函数为例, 很多 C++ 书籍上都提到当构造失败时只有异常可以处理, Google 禁止使用异常这一点, 仅仅是为了自身的方便, 说大了, 无非是基于软件管理成本上, 实际使用中还是自己决定)

.. _RTTI:

6.7. 运行时类型识别
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
    TODO

.. tip::

    我们禁止使用 RTTI.

定义:

    RTTI 允许程序员在运行时识别 C++ 类对象的类型. 它通过使用 ``typeid`` 或者 ``dynamic_cast`` 完成.

优点:

	RTTI 的标准替代 (下面将描述) 需要对有问题的类层级进行修改或重构. 有时这样的修改并不是我们所想要的, 甚至是不可取的, 尤其是在一个已经广泛使用的或者成熟的代码中.
	
	RTTI 在某些单元测试中非常有用. 比如进行工厂类测试时, 用来验证一个新建对象是否为期望的动态类型. RTTI 对于管理对象和派生对象的关系也很有用.
	
	在考虑多个抽象对象时 RTTI 也很好用. 例如:
	
        .. code-block:: c++

            bool Base::Equal(Base* other) = 0;
            bool Derived::Equal(Base* other) {
              Derived* that = dynamic_cast<Derived*>(other);
              if (that == NULL)
                return false;
              ...
            }

缺点:

	在运行时判断类型通常意味着设计问题. 如果你需要在运行期间确定一个对象的类型, 这通常说明你需要考虑重新设计你的类.
	
	随意地使用 RTTI 会使你的代码难以维护. 它使得基于类型的判断树或者 switch 语句散布在代码各处. 如果以后要进行修改, 你就必须检查它们.

结论:

	RTTI 有合理的用途但是容易被滥用, 因此在使用时请务必注意. 在单元测试中可以使用 RTTI, 但是在其他代码中请尽量避免. 尤其是在新代码中, 使用 RTTI 前务必三思. 如果你的代码需要根据不同的对象类型执行不同的行为的话, 请考虑用以下的两种替代方案之一查询类型:
		
	虚函数可以根据子类类型的不同而执行不同代码. 这是把工作交给了对象本身去处理.
		
	如果这一工作需要在对象之外完成, 可以考虑使用双重分发的方案, 例如使用访问者设计模式. 这就能够在对象之外进行类型判断.
	
	如果程序能够保证给定的基类实例实际上都是某个派生类的实例, 那么就可以自由使用 dynamic_cast. 在这种情况下, 使用 dynamic_cast 也是一种替代方案.
	
	基于类型的判断树是一个很强的暗示, 它说明你的代码已经偏离正轨了. 不要像下面这样:
	
        .. code-block:: c++
        
            if (typeid(*data) == typeid(D1)) {
              ...
            } else if (typeid(*data) == typeid(D2)) {
              ...
            } else if (typeid(*data) == typeid(D3)) {
            ...
            
	一旦在类层级中加入新的子类, 像这样的代码往往会崩溃. 而且, 一旦某个子类的属性改变了, 你很难找到并修改所有受影响的代码块.
	
	不要去手工实现一个类似 RTTI 的方案. 反对 RTTI 的理由同样适用于这些方案, 比如带类型标签的类继承体系. 而且, 这些方案会掩盖你的真实意图.

6.8. 类型转换
~~~~~~~~~~~~~~~~~~~~~~

.. tip::

    使用 C++ 的类型转换, 如 ``static_cast<>()``. 不要使用 ``int y = (int)x`` 或 ``int y = int(x)`` 等转换方式;

定义:

    C++ 采用了有别于 C 的类型转换机制, 对转换操作进行归类.

优点:

    C 语言的类型转换问题在于模棱两可的操作; 有时是在做强制转换 (如 ``(int)3.5``), 有时是在做类型转换 (如 ``(int)"hello"``). 另外, C++ 的类型转换在查找时更醒目.

缺点:

    恶心的语法.

结论:

    不要使用 C 风格类型转换. 而应该使用 C++ 风格.

        - 用 ``static_cast`` 替代 C 风格的值转换, 或某个类指针需要明确的向上转换为父类指针时.
        - 用 ``const_cast`` 去掉 ``const`` 限定符.
        - 用 ``reinterpret_cast`` 指针类型和整型或其它指针之间进行不安全的相互转换. 仅在你对所做一切了然于心时使用.

    至于 ``dynamic_cast`` 参见 :ref:`RTTI`.

.. _streams:

6.9. 流
~~~~~~~~~~~~~~

.. tip::

    只在记录日志时使用流.

定义:

    流用来替代 ``printf()`` 和 ``scanf()``.

优点:

    有了流, 在打印时不需要关心对象的类型. 不用担心格式化字符串与参数列表不匹配 (虽然在 gcc 中使用 ``printf`` 也不存在这个问题). 流的构造和析构函数会自动打开和关闭对应的文件.

缺点:

    流使得 ``pread()`` 等功能函数很难执行. 如果不使用 ``printf`` 风格的格式化字符串, 某些格式化操作 (尤其是常用的格式字符串 ``%.*s``) 用流处理性能是很低的. 流不支持字符串操作符重新排序 (%1s), 而这一点对于软件国际化很有用.

结论:

    不要使用流, 除非是日志接口需要. 使用 ``printf`` 之类的代替.

    使用流还有很多利弊, 但代码一致性胜过一切. 不要在代码中使用流.

拓展讨论:

    对这一条规则存在一些争论, 这儿给出点深层次原因. 回想一下唯一性原则 (Only One Way): 我们希望在任何时候都只使用一种确定的 I/O 类型, 使代码在所有 I/O 处都保持一致. 因此, 我们不希望用户来决定是使用流还是 ``printf + read/write``. 相反, 我们应该决定到底用哪一种方式. 把日志作为特例是因为日志是一个非常独特的应用, 还有一些是历史原因.

    流的支持者们主张流是不二之选, 但观点并不是那么清晰有力. 他们指出的流的每个优势也都是其劣势. 流最大的优势是在输出时不需要关心打印对象的类型. 这是一个亮点. 同时, 也是一个不足: 你很容易用错类型, 而编译器不会报警. 使用流时容易造成的这类错误:

        .. code-block:: c++

            cout << this;   // 输出地址
            cout << *this;  // 输出值

    由于 ``<<`` 被重载, 编译器不会报错. 就因为这一点我们反对使用操作符重载.

    有人说 ``printf`` 的格式化丑陋不堪, 易读性差, 但流也好不到哪儿去. 看看下面两段代码吧, 实现相同的功能, 哪个更清晰?

        .. code-block:: c++

            cerr << "Error connecting to '" << foo->bar()->hostname.first
                 << ":" << foo->bar()->hostname.second << ": " << strerror(errno);

            fprintf(stderr, "Error connecting to '%s:%u: %s",
                    foo->bar()->hostname.first, foo->bar()->hostname.second,
                    strerror(errno));

    你可能会说, "把流封装一下就会比较好了", 这儿可以, 其他地方呢? 而且不要忘了, 我们的目标是使语言更紧凑, 而不是添加一些别人需要学习的新装备.

    每一种方式都是各有利弊, "没有最好, 只有更适合". 简单性原则告诫我们必须从中选择其一, 最后大多数决定采用 ``printf + read/write``.

6.10. 前置自增和自减
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

.. tip::

    对于迭代器和其他模板对象使用前缀形式 (``++i``) 的自增, 自减运算符.

定义:

    对于变量在自增 (``++i`` 或 ``i++``) 或自减 (``--i`` 或 ``i--``) 后表达式的值又没有没用到的情况下, 需要确定到底是使用前置还是后置的自增 (自减).

优点:

    不考虑返回值的话, 前置自增 (``++i``) 通常要比后置自增 (``i++``) 效率更高. 因为后置自增 (或自减) 需要对表达式的值 ``i`` 进行一次拷贝. 如果 ``i`` 是迭代器或其他非数值类型, 拷贝的代价是比较大的. 既然两种自增方式实现的功能一样, 为什么不总是使用前置自增呢?

缺点:

    在 C 开发中, 当表达式的值未被使用时, 传统的做法是使用后置自增, 特别是在 ``for`` 循环中. 有些人觉得后置自增更加易懂, 因为这很像自然语言, 主语 (``i``) 在谓语动词 (``++``) 前.

结论:

    对简单数值 (非对象), 两种都无所谓. 对迭代器和模板类型, 使用前置自增 (自减).

6.11. ``const`` 用法
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

.. tip::

    我们强烈建议你在任何可能的情况下都要使用 ``const``. 此外有时改用 C++11 推出的 constexpr 更好。

定义:

    在声明的变量或参数前加上关键字 ``const`` 用于指明变量值不可被篡改 (如 ``const int foo`` ). 为类中的函数加上 ``const`` 限定符表明该函数不会修改类成员变量的状态 (如 ``class Foo { int Bar(char c) const; };``).

优点:

    大家更容易理解如何使用变量. 编译器可以更好地进行类型检测, 相应地, 也能生成更好的代码. 人们对编写正确的代码更加自信, 因为他们知道所调用的函数被限定了能或不能修改变量值. 即使是在无锁的多线程编程中, 人们也知道什么样的函数是安全的.

缺点:

    ``const`` 是入侵性的: 如果你向一个函数传入 ``const`` 变量, 函数原型声明中也必须对应 ``const`` 参数 (否则变量需要 ``const_cast`` 类型转换), 在调用库函数时显得尤其麻烦.

结论:

    ``const`` 变量, 数据成员, 函数和参数为编译时类型检测增加了一层保障; 便于尽早发现错误. 因此, 我们强烈建议在任何可能的情况下使用 ``const``:

        - 如果函数不会修改你传入的引用或指针类型参数, 该参数应声明为 ``const``.
        - 尽可能将函数声明为 ``const``. 访问函数应该总是 ``const``. 其他不会修改任何数据成员, 未调用非 ``const`` 函数, 不会返回数据成员非 ``const`` 指针或引用的函数也应该声明成 ``const``.
        - 如果数据成员在对象构造之后不再发生变化, 可将其定义为 ``const``.

    然而, 也不要发了疯似的使用 ``const``. 像 ``const int * const * const x;`` 就有些过了, 虽然它非常精确的描述了常量 ``x``. 关注真正有帮助意义的信息: 前面的例子写成 ``const int** x`` 就够了.

    关键字 ``mutable`` 可以使用, 但是在多线程中是不安全的, 使用时首先要考虑线程安全.

``const`` 的位置:

    有人喜欢 ``int const *foo`` 形式, 不喜欢 ``const int* foo``, 他们认为前者更一致因此可读性也更好: 遵循了 ``const`` 总位于其描述的对象之后的原则. 但是一致性原则不适用于此, "不要过度使用" 的声明可以取消大部分你原本想保持的一致性. 将 ``const`` 放在前面才更易读, 因为在自然语言中形容词 (``const``) 是在名词 (``int``) 之前.

    这是说, 我们提倡但不强制 ``const`` 在前. 但要保持代码的一致性! (Yang.Y 注: 也就是不要在一些地方把 ``const`` 写在类型前面, 在其他地方又写在后面, 确定一种写法, 然后保持一致.)

6.12. ``constexpr`` 用法
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

.. tip::

    在 C++11 里，用 constexpr 来定义真正的常量，或实现常量初始化。

定义:

    变量可以被声明成 constexpr 以表示它是真正意义上的常量，即在编译时和运行时都不变。函数或构造函数也可以被声明成 constexpr, 以用来定义 constexpr 变量。

优点:

    如今 constexpr 就可以定义浮点式的真・常量，不用再依赖字面值了；也可以定义用户自定义类型上的常量；甚至也可以定义函数调用所返回的常量。

缺点:

    若过早把变量优化成 constexpr 变量，将来又要把它改为常规变量时，挺麻烦的；当前对constexpr函数和构造函数中允许的限制可能会导致这些定义中解决的方法模糊。

结论:

    靠 constexpr 特性，方才实现了 C++ 在接口上打造真正常量机制的可能。好好用 constexpr 来定义真・常量以及支持常量的函数。避免复杂的函数定义，以使其能够与constexpr一起使用。 千万别痴心妄想地想靠 constexpr 来强制代码「内联」。

6.13. 整型
~~~~~~~~~~~~~~~~~~

.. tip::

    C++ 内建整型中, 仅使用 ``int``. 如果程序中需要不同大小的变量, 可以使用 ``<stdint.h>`` 中长度精确的整型, 如 ``int16_t``.如果您的变量可能不小于 2^31 (2GiB), 就用 64 位变量比如 ``int64_t``. 此外要留意，哪怕您的值并不会超出 int 所能够表示的范围，在计算过程中也可能会溢出。所以拿不准时，干脆用更大的类型。

定义:

    C++ 没有指定整型的大小. 通常人们假定 ``short`` 是 16 位, ``int`` 是 32 位, ``long`` 是 32 位, ``long long`` 是 64 位.

优点:

    保持声明统一.

缺点:

    C++ 中整型大小因编译器和体系结构的不同而不同.

结论:

    ``<stdint.h>`` 定义了 ``int16_t``, ``uint32_t``, ``int64_t`` 等整型, 在需要确保整型大小时可以使用它们代替 ``short``, ``unsigned long long`` 等. 在 C 整型中, 只使用 ``int``. 在合适的情况下, 推荐使用标准类型如 ``size_t`` 和 ``ptrdiff_t``.

    如果已知整数不会太大, 我们常常会使用 ``int``, 如循环计数. 在类似的情况下使用原生类型 ``int``. 你可以认为 ``int`` 至少为 32 位, 但不要认为它会多于 ``32`` 位. 如果需要 64 位整型, 用 ``int64_t`` 或 ``uint64_t``.

    对于大整数, 使用 ``int64_t``.

    不要使用 ``uint32_t`` 等无符号整型, 除非你是在表示一个位组而不是一个数值, 或是你需要定义二进制补码溢出. 尤其是不要为了指出数值永不会为负, 而使用无符号类型. 相反, 你应该使用断言来保护数据.

    如果您的代码涉及容器返回的大小（size），确保其类型足以应付容器各种可能的用法。拿不准时，类型越大越好。

    小心整型类型转换和整型提升（acgtyrant 注：integer promotions, 比如 ``int`` 与 ``unsigned int`` 运算时，前者被提升为 ``unsigned int`` 而有可能溢出），总有意想不到的后果。

关于无符号整数:

    有些人, 包括一些教科书作者, 推荐使用无符号类型表示非负数. 这种做法试图达到自我文档化. 但是, 在 C 语言中, 这一优点被由其导致的 bug 所淹没. 看看下面的例子:

        .. code-block:: c++

            for (unsigned int i = foo.Length()-1; i >= 0; --i) ...

    上述循环永远不会退出! 有时 gcc 会发现该 bug 并报警, 但大部分情况下都不会. 类似的 bug 还会出现在比较有符合变量和无符号变量时. 主要是 C 的类型提升机制会致使无符号类型的行为出乎你的意料.

    因此, 使用断言来指出变量为非负数, 而不是使用无符号型!

6.14. 64 位下的可移植性
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

.. tip::

    代码应该对 64 位和 32 位系统友好. 处理打印, 比较, 结构体对齐时应切记:

- 对于某些类型, ``printf()`` 的指示符在 32 位和 64 位系统上可移植性不是很好. C99 标准定义了一些可移植的格式化指示符. 不幸的是, MSVC 7.1 并非全部支持, 而且标准中也有所遗漏, 所以有时我们不得不自己定义一个丑陋的版本 (头文件 ``inttypes.h`` 仿标准风格):

    .. code-block:: c++

        // printf macros for size_t, in the style of inttypes.h
        #ifdef _LP64
        #define __PRIS_PREFIX "z"
        #else
        #define __PRIS_PREFIX
        #endif

        // Use these macros after a % in a printf format string
        // to get correct 32/64 bit behavior, like this:
        // size_t size = records.size();
        // printf("%"PRIuS"\n", size);
        #define PRIdS __PRIS_PREFIX "d"
        #define PRIxS __PRIS_PREFIX "x"
        #define PRIuS __PRIS_PREFIX "u"
        #define PRIXS __PRIS_PREFIX "X"
        #define PRIoS __PRIS_PREFIX "o"


    +-------------------+---------------------+--------------------------+------------------+
    | 类型              | 不要使用            | 使用                     | 备注             |
    +===================+=====================+==========================+==================+
    | ``void *``        |                     |                          |                  |
    | (或其他指针类型)  | ``%lx``             | ``%p``                   |                  |
    +-------------------+---------------------+--------------------------+------------------+
    | ``int64_t``       | ``%qd, %lld``       | ``%"PRId64"``            |                  |
    +-------------------+---------------------+--------------------------+------------------+
    | ``uint64_t``      | ``%qu, %llu, %llx`` | ``%"PRIu64", %"PRIx64"`` |                  |
    +-------------------+---------------------+--------------------------+------------------+
    | ``size_t``        | ``%u``              | ``%"PRIuS", %"PRIxS"``   | C99 规定 ``%zu`` |
    +-------------------+---------------------+--------------------------+------------------+
    | ``ptrdiff_t``     | ``%d``              | ``%"PRIdS"``             | C99 规定 ``%zd`` |
    +-------------------+---------------------+--------------------------+------------------+

    注意 ``PRI*`` 宏会被编译器扩展为独立字符串. 因此如果使用非常量的格式化字符串, 需要将宏的值而不是宏名插入格式中. 使用 ``PRI*`` 宏同样可以在 ``%`` 后包含长度指示符. 例如, ``printf("x = %30"PRIuS"\n", x)`` 在 32 位 Linux 上将被展开为 ``printf("x = %30" "u" "\n", x)``, 编译器当成 ``printf("x = %30u\n", x)`` 处理 (Yang.Y 注: 这在 MSVC 6.0 上行不通, VC 6 编译器不会自动把引号间隔的多个字符串连接一个长字符串).

- 记住 ``sizeof(void *) != sizeof(int)``. 如果需要一个指针大小的整数要用 ``intptr_t``.

- 你要非常小心的对待结构体对齐, 尤其是要持久化到磁盘上的结构体 (Yang.Y 注: 持久化 - 将数据按字节流顺序保存在磁盘文件或数据库中). 在 64 位系统中, 任何含有 ``int64_t``/``uint64_t`` 成员的类/结构体, 缺省都以 8 字节在结尾对齐. 如果 32 位和 64 位代码要共用持久化的结构体, 需要确保两种体系结构下的结构体对齐一致. 大多数编译器都允许调整结构体对齐. gcc 中可使用 ``__attribute__((packed))``. MSVC 则提供了 ``#pragma pack()`` 和 ``__declspec(align())`` (YuleFox 注, 解决方案的项目属性里也可以直接设置).

- 创建 64 位常量时使用 LL 或 ULL 作为后缀, 如:

    .. code-block:: c++

        int64_t my_value = 0x123456789LL;
        uint64_t my_mask = 3ULL << 48;


- 如果你确实需要 32 位和 64 位系统具有不同代码, 可以使用 ``#ifdef _LP64`` 指令来切分 32/64 位代码. (尽量不要这么做, 如果非用不可, 尽量使修改局部化)

.. _preprocessor-macros:

6.15. 预处理宏
~~~~~~~~~~~~~~~~~~~~~~~~

.. tip::

    使用宏时要非常谨慎, 尽量以内联函数, 枚举和常量代替之.

宏意味着你和编译器看到的代码是不同的. 这可能会导致异常行为, 尤其因为宏具有全局作用域.

值得庆幸的是, C++ 中, 宏不像在 C 中那么必不可少. 以往用宏展开性能关键的代码, 现在可以用内联函数替代. 用宏表示常量可被 ``const`` 变量代替. 用宏 "缩写" 长变量名可被引用代替. 用宏进行条件编译... 这个, 千万别这么做, 会令测试更加痛苦 (``#define`` 防止头文件重包含当然是个特例).

宏可以做一些其他技术无法实现的事情, 在一些代码库 (尤其是底层库中) 可以看到宏的某些特性 (如用 ``#`` 字符串化, 用 ``##`` 连接等等). 但在使用前, 仔细考虑一下能不能不使用宏达到同样的目的.

下面给出的用法模式可以避免使用宏带来的问题; 如果你要宏, 尽可能遵守:

    - 不要在 ``.h`` 文件中定义宏.
    - 在马上要使用时才进行 ``#define``, 使用后要立即 ``#undef``.
    - 不要只是对已经存在的宏使用#undef，选择一个不会冲突的名称；
    - 不要试图使用展开后会导致 C++ 构造不稳定的宏, 不然也至少要附上文档说明其行为.
    - 不要用 ``##`` 处理函数，类和变量的名字。

6.16. 0, ``nullptr`` 和 ``NULL``
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

.. tip::

    指针使用  ``nullptr``，字符使用 ``'\0'`` (而不是 ``0`` 字面值)。

    对于指针 (地址值)，使用 ``nullptr``，因为这保证了类型安全。

    使用 ``'\0'`` 作为空字符。使用正确的类型使代码更具可读性。

6.17. sizeof
~~~~~~~~~~~~~~~~~~~~~~~~

.. tip::

    尽可能用 ``sizeof(varname)`` 代替 ``sizeof(type)``.

    使用 ``sizeof(varname)`` 是因为当代码中变量类型改变时会自动更新. 您或许会用 ``sizeof(type)`` 处理不涉及任何变量的代码，比如处理来自外部或内部的数据格式，这时用变量就不合适了。

    .. code-block:: c++

        Struct data;
        Struct data; memset(&data, 0, sizeof(data));

    .. warning::
        .. code-block:: c++

            memset(&data, 0, sizeof(Struct));

    .. code-block:: c++

        if (raw_size < sizeof(int)) {
            LOG(ERROR) << "compressed record not big enough for count: " << raw_size;
            return false;
        }

6.18. auto
~~~~~~~~~~~~~~~~~~~~

.. tip::

    用 ``auto`` 绕过烦琐的类型名，只要可读性好就继续用，别用在局部变量之外的地方。

定义：

    C++11 中，若变量被声明成 ``auto``, 那它的类型就会被自动匹配成初始化表达式的类型。您可以用 ``auto`` 来复制初始化或绑定引用。

    .. code-block:: c++

        vector<string> v;
        ...
        auto s1 = v[0];  // 创建一份 v[0] 的拷贝。
        const auto& s2 = v[0];  // s2 是 v[0] 的一个引用。

优点：

    C++ 类型名有时又长又臭，特别是涉及模板或命名空间的时候。就像：

    .. code-block:: c++

        sparse_hash_map<string, int>::iterator iter = m.find(val);

    返回类型好难读，代码目的也不够一目了然。重构其：

    .. code-block:: c++

        auto iter = m.find(val);

    好多了。

    没有 ``auto`` 的话，我们不得不在同一个表达式里写同一个类型名两次，无谓的重复，就像：

    .. code-block:: c++

        diagnostics::ErrorStatus* status = new diagnostics::ErrorStatus("xyz");

    有了 auto, 可以更方便地用中间变量，显式编写它们的类型轻松点。

缺点：

    类型够明显时，特别是初始化变量时，代码才会够一目了然。但以下就不一样了：

    .. code-block:: c++

        auto i = x.Lookup(key);

    看不出其类型是啥，x 的类型声明恐怕远在几百行之外了。

    程序员必须会区分 ``auto`` 和 ``const auto&`` 的不同之处，否则会复制错东西。

    auto 和 C++11 列表初始化的合体令人摸不着头脑：

    .. code-block:: c++

        auto x(3);  // 圆括号。
        auto y{3};  // 大括号。

    它们不是同一回事——``x`` 是 ``int``, ``y`` 则是 ``std::initializer_list<int>``. 其它一般不可见的代理类型（acgtyrant 注：normally-invisible proxy types, 它涉及到 C++ 鲜为人知的坑：`Why is vector<bool> not a STL container? <https://stackoverflow.com/a/17794965/1546088>`_）也有大同小异的陷阱。

    如果在接口里用 ``auto``, 比如声明头文件里的一个常量，那么只要仅仅因为程序员一时修改其值而导致类型变化的话——API 要翻天覆地了。

结论：

    ``auto`` 只能用在局部变量里用。别用在文件作用域变量，命名空间作用域变量和类数据成员里。永远别列表初始化 ``auto`` 变量。

    ``auto`` 还可以和 C++11 特性「尾置返回类型（trailing return type）」一起用，不过后者只能用在 lambda 表达式里。

.. _braced-initializer-list:

6.19. 列表初始化
~~~~~~~~~~~~~~~~~~~~~~~~~~~~

.. tip::

    你可以用列表初始化。

    早在 C++03 里，聚合类型（aggregate types）就已经可以被列表初始化了，比如数组和不自带构造函数的结构体：

    .. code-block:: c++

        struct Point { int x; int y; };
        Point p = {1, 2};

    C++11 中，该特性得到进一步的推广，任何对象类型都可以被列表初始化。示范如下：

    .. code-block:: c++

        // Vector 接收了一个初始化列表。
        vector<string> v{"foo", "bar"};

        // 不考虑细节上的微妙差别，大致上相同。
        // 您可以任选其一。
        vector<string> v = {"foo", "bar"};

        // 可以配合 new 一起用。
        auto p = new vector<string>{"foo", "bar"};

        // map 接收了一些 pair, 列表初始化大显神威。
        map<int, string> m = {{1, "one"}, {2, "2"}};

        // 初始化列表也可以用在返回类型上的隐式转换。
        vector<int> test_function() { return {1, 2, 3}; }

        // 初始化列表可迭代。
        for (int i : {-1, -2, -3}) {}

        // 在函数调用里用列表初始化。
        void TestFunction2(vector<int> v) {}
        TestFunction2({1, 2, 3});

    用户自定义类型也可以定义接收 ``std::initializer_list<T>`` 的构造函数和赋值运算符，以自动列表初始化：

    .. code-block:: c++

        class MyType {
         public:
          // std::initializer_list 专门接收 init 列表。
          // 得以值传递。
          MyType(std::initializer_list<int> init_list) {
            for (int i : init_list) append(i);
          }
          MyType& operator=(std::initializer_list<int> init_list) {
            clear();
            for (int i : init_list) append(i);
          }
        };
        MyType m{2, 3, 5, 7};

    最后，列表初始化也适用于常规数据类型的构造，哪怕没有接收 ``std::initializer_list<T>`` 的构造函数。

    .. code-block:: c++

        double d{1.23};
        // MyOtherType 没有 std::initializer_list 构造函数，
         // 直接上接收常规类型的构造函数。
        class MyOtherType {
         public:
          explicit MyOtherType(string);
          MyOtherType(int, string);
        };
        MyOtherType m = {1, "b"};
        // 不过如果构造函数是显式的（explict），您就不能用 `= {}` 了。
        MyOtherType m{"b"};

    千万别直接列表初始化 auto 变量，看下一句，估计没人看得懂：

    .. warning::
        .. code-block:: c++

            auto d = {1.23};        // d 即是 std::initializer_list<double>

    .. code-block:: c++

        auto d = double{1.23};  // 善哉 -- d 即为 double, 并非 std::initializer_list.

    至于格式化，参见 :ref:`braced-initializer-list-format`.

.. _lambda-expressions:

6.20. Lambda 表达式
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

.. tip::

    适当使用 lambda 表达式。当 lambda 将转移当前作用域时，首选显式捕获。

定义：

    Lambda 表达式是创建匿名函数对象的一种简易途径，常用于把函数当参数传，例如：

    .. code-block:: c++

        std::sort(v.begin(), v.end(), [](int x, int y) {
            return Weight(x) < Weight(y);
        });

    它们还允许通过名称显式或隐式使用默认捕获从封闭范围中捕获变量。显式捕获要求将每个变量作为值或引用捕获列出：

    .. code-block:: c++

        int weight = 3;
        int sum = 0;
        // Captures `weight` by value and `sum` by reference.
        std::for_each(v.begin(), v.end(), [weight, &sum](int x) {
          sum += weight * x;
        });
        
    默认捕获隐式捕获 lambda 正文中引用的任何变量，包括 this 是否使用了任何成员：

    .. code-block:: c++

        const std::vector<int> lookup_table = ...;
        std::vector<int> indices = ...;
        // Captures `lookup_table` by reference, sorts `indices` by the value
        // of the associated element in `lookup_table`.
        std::sort(indices.begin(), indices.end(), [&](int a, int b) {
          return lookup_table[a] < lookup_table[b];
        });
        
    变量捕获还可以具有显式初始值设定项，该初始值设定项可用于按值捕获仅移动变量，或用于普通引用或值捕获无法处理的其他情况：

    .. code-block:: c++

        std::unique_ptr<Foo> foo = ...;
        [foo = std::move(foo)] () {
            ...
        }

    此类捕获（通常称为“初始化捕获”或“广义 lambda 捕获”）实际上不需要从封闭作用域中“捕获”任何内容，甚至不需要从封闭作用域中具有名称;此语法是定义 Lambda 对象成员的完全通用方法：

    .. code-block:: c++

        [foo = std::vector<int>({1, 2, 3})] () {
          ...
        }

优点：

    * 传函数对象给 STL 算法，Lambda 最简易，可读性也好。
    * 适当使用默认捕获可以消除冗余，并突出显示默认捕获中的重要异常。
    * Lambda, ``std::functions`` 和 ``std::bind`` 可以搭配成通用回调机制（general purpose callback mechanism）；写接收有界函数为参数的函数也很容易了。

缺点：

    * lambda 中的变量捕获可能是悬空指针错误的根源，尤其是当 lambda 逃逸到当前作用域时。
    * 按值默认捕获可能会产生误导，因为它们不能防止悬空指针错误。按值捕获指针不会导致深度复制，因此它通常具有与按引用捕获相同的生存期问题。这在按值捕获 this 时尤其令人困惑，因为 的 this 用法通常是隐式的。
    * 捕获实际上声明了新变量（无论捕获是否具有初始值设定项），但它们看起来与 C++ 中的任何其他变量声明语法完全不同。特别是，变量的类型没有位置，甚至没有 auto 占位符（尽管初始化捕获可以间接指示它，例如，使用强制转换）。这甚至可能使得很难将它们识别为声明。
    * 初始化捕获本质上依赖于类型推导，并且存在许多与 auto 相同的缺点，但另一个问题是语法甚至没有提示读者正在进行推导。
    * lambda 的使用可能会失控;非常长的嵌套匿名函数会使代码更难理解。

结论：

    * 在适当的情况下使用 lambda 表达式，格式如下 `所述 <https://google.github.io/styleguide/cppguide.html#Formatting_Lambda_Expressions>`_。
    * 如果 lambda 可能离开当前作用域，则首选显式捕获。例如：

      .. code-block:: c++
  
          {
            Foo foo;
            ...
            executor->Schedule([&] { Frobnicate(foo); });
            ...
          }
          // BAD! The fact that the lambda makes use of a reference to `foo` and
          // possibly `this` (if `Frobnicate` is a member function) may not be
          // apparent on a cursory inspection. If the lambda is invoked after
          // the function returns, that would be bad, because both `foo`
          // and the enclosing object could have been destroyed.
  
      更喜欢写：
  
      .. code-block:: c++
  
          {
            Foo foo;
            ...
            executor->Schedule([&foo] { Frobnicate(foo); })
            ...
          }
          // BETTER - The compile will fail if `Frobnicate` is a member
          // function, and it's clearer that `foo` is dangerously captured by
          // reference.

    * 仅当 lambda 的生存期明显短于任何潜在捕获时，才使用默认的引用捕获 （ [&] ）。
    * 仅使用默认的按值 （ [=] ） 捕获作为绑定短 lambda 的几个变量的方法，其中捕获的变量集一目了然，并且不会导致隐式捕获 this 。（这意味着出现在非静态类成员函数中并在其正文中引用非静态类成员的 lambda 必须显式或 this 通过 [&] 捕获。不建议使用默认的按值捕获来编写长而复杂的 lambda。
    * 仅使用捕获来实际捕获封闭范围中的变量。不要将捕获与初始值设定项一起使用来引入新名称，或实质性地更改现有名称的含义。相反，以传统方式声明一个新变量，然后捕获它，或者避免使用 lambda 简写并显式定义函数对象。
    * 有关指定参数和返回类型的指导，请参阅 `类型推导 <https://google.github.io/styleguide/cppguide.html#Type_deduction>`_ 部分。

.. _template-metaprogramming:

6.21. 模板编程
~~~~~~~~~~~~~~~~~~~~~~~~~~~~
.. tip::
    不要使用复杂的模板编程

定义:

    模板编程指的是利用c++ 模板实例化机制是图灵完备性, 可以被用来实现编译时刻的类型判断的一系列编程技巧

优点:

    模板编程能够实现非常灵活的类型安全的接口和极好的性能, 一些常见的工具比如Google Test, std::tuple, std::function 和 Boost.Spirit. 这些工具如果没有模板是实现不了的

缺点:

    * 模板编程所使用的技巧对于使用c++不是很熟练的人是比较晦涩, 难懂的. 在复杂的地方使用模板的代码让人更不容易读懂, 并且debug 和 维护起来都很麻烦
    
    * 模板编程经常会导致编译出错的信息非常不友好: 在代码出错的时候, 即使这个接口非常的简单, 模板内部复杂的实现细节也会在出错信息显示. 导致这个编译出错信息看起来非常难以理解.
    * 大量的使用模板编程接口会让重构工具(Visual Assist X, Refactor for C++等等)更难发挥用途. 首先模板的代码会在很多上下文里面扩展开来, 所以很难确认重构对所有的这些展开的代码有用, 其次有些重构工具只对已经做过模板类型替换的代码的AST 有用. 因此重构工具对这些模板实现的原始代码并不有效, 很难找出哪些需要重构.

    
结论:

    * 模板编程有时候能够实现更简洁更易用的接口, 但是更多的时候却适得其反. 因此模板编程最好只用在少量的基础组件, 基础数据结构上, 因为模板带来的额外的维护成本会被大量的使用给分担掉

    * 在使用模板编程或者其他复杂的模板技巧的时候, 你一定要再三考虑一下. 考虑一下你们团队成员的平均水平是否能够读懂并且能够维护你写的模板代码.或者一个非c++ 程序员和一些只是在出错的时候偶尔看一下代码的人能够读懂这些错误信息或者能够跟踪函数的调用流程. 如果你使用递归的模板实例化, 或者类型列表, 或者元函数, 又或者表达式模板, 或者依赖SFINAE, 或者sizeof 的trick 手段来检查函数是否重载, 那么这说明你模板用的太多了, 这些模板太复杂了, 我们不推荐使用

    * 如果你使用模板编程, 你必须考虑尽可能的把复杂度最小化, 并且尽量不要让模板对外暴露. 你最好只在实现里面使用模板, 然后给用户暴露的接口里面并不使用模板, 这样能提高你的接口的可读性. 并且你应该在这些使用模板的代码上写尽可能详细的注释. 你的注释里面应该详细的包含这些代码是怎么用的, 这些模板生成出来的代码大概是什么样子的. 还需要额外注意在用户错误使用你的模板代码的时候需要输出更人性化的出错信息. 因为这些出错信息也是你的接口的一部分, 所以你的代码必须调整到这些错误信息在用户看起来应该是非常容易理解, 并且用户很容易知道如何修改这些错误

.. _boost:

6.22. Boost 库
~~~~~~~~~~~~~~~~~~~~~~~~~~~~

.. tip::

    只使用 Boost 中被认可的库.

定义:

    `Boost 库集 <https://www.boost.org/>`_ 是一个广受欢迎, 经过同行鉴定, 免费开源的 C++ 库集.

优点:

    Boost代码质量普遍较高, 可移植性好, 填补了 C++ 标准库很多空白, 如型别的特性, 更完善的绑定器, 更好的智能指针。

缺点:

    某些 Boost 库提倡的编程实践可读性差, 比如元编程和其他高级模板技术, 以及过度 "函数化" 的编程风格.

结论:

    为了向阅读和维护代码的人员提供更好的可读性, 我们只允许使用 Boost 一部分经认可的特性子集. 目前允许使用以下库:

        - `Call Traits <https://www.boost.org/doc/libs/1_58_0/libs/utility/call_traits.htm>`_ : ``boost/call_traits.hpp``

        - `Compressed Pair <https://www.boost.org/libs/utility/compressed_pair.htm>`_ : ``boost/compressed_pair.hpp``

        - `<The Boost Graph Library (BGL) <https://www.boost.org/doc/libs/1_58_0/libs/graph/doc/index.html>`_ : ``boost/graph``, except serialization (``adj_list_serialize.hpp``) and parallel/distributed algorithms and data structures(``boost/graph/parallel/*`` and ``boost/graph/distributed/*``)

        - `Property Map <https://www.boost.org/libs/property_map/>`_ : ``boost/property_map.hpp``

        - The part of `Iterator <https://www.boost.org/libs/iterator/>`_ that deals with defining iterators: ``boost/iterator/iterator_adaptor.hpp``, ``boost/iterator/iterator_facade.hpp``, and ``boost/function_output_iterator.hpp``

        - The part of `Polygon <https://www.boost.org/libs/polygon/>`_ that deals with Voronoi diagram construction and doesn't depend on the rest of Polygon: ``boost/polygon/voronoi_builder.hpp``, ``boost/polygon/voronoi_diagram.hpp``, and ``boost/polygon/voronoi_geometry_type.hpp``

        - `Bimap <https://www.boost.org/libs/bimap/>`_ : ``boost/bimap``

        - `Statistical Distributions and Functions <https://www.boost.org/libs/math/doc/html/dist.html>`_ : ``boost/math/distributions``

        - `Multi-index <https://www.boost.org/libs/multi_index/>`_ : ``boost/multi_index``

        - `Heap <https://www.boost.org/libs/heap/>`_ : ``boost/heap``

        - The flat containers from `Container <https://www.boost.org/libs/container/>`_: ``boost/container/flat_map``, and ``boost/container/flat_set``

    我们正在积极考虑增加其它 Boost 特性, 所以列表中的规则将不断变化.

    以下库可以用，但由于如今已经被 C++ 11 标准库取代，不再鼓励：

        - `Pointer Container <https://www.boost.org/libs/ptr_container/>`_ : ``boost/ptr_container``, 改用 `std::unique_ptr <https://en.cppreference.com/w/cpp/memory/unique_ptr>`_

        - `Array <https://www.boost.org/libs/array/>`_ : ``boost/array.hpp``, 改用 `std::array <https://en.cppreference.com/w/cpp/container/array>`_

6.23. C++11
~~~~~~~~~~~~~~~~~~~~~~

.. tip::

    适当用 C++11（前身是 C++0x）的库和语言扩展，在贵项目用 C++11 特性前三思可移植性。

定义：

    C++11 有众多语言和库上的 `变革 <https://en.wikipedia.org/wiki/C%2B%2B11>`_ 。

优点：

    在二〇一四年八月之前，C++11 一度是官方标准，被大多 C++ 编译器支持。它标准化很多我们早先就在用的 C++ 扩展，简化了不少操作，大大改善了性能和安全。

缺点：

    C++11 相对于前身，复杂极了：1300 页 vs 800 页！很多开发者也不怎么熟悉它。于是从长远来看，前者特性对代码可读性以及维护代价难以预估。我们说不准什么时候采纳其特性，特别是在被迫依赖老实工具的项目上。

    和 :ref:`boost` 一样，有些 C++11 扩展提倡实则对可读性有害的编程实践——就像去除冗余检查（比如类型名）以帮助读者，或是鼓励模板元编程等等。有些扩展在功能上与原有机制冲突，容易招致困惑以及迁移代价。

结论：

    C++11 特性除了个别情况下，可以用一用。除了本指南会有不少章节会加以讨若干 C++11 特性之外，以下特性最好不要用：

    - 尾置返回类型，比如用 ``auto foo() -> int`` 代替 ``int foo()``. 为了兼容于现有代码的声明风格。
    - 编译时合数 ``<ratio>``, 因为它涉及一个重模板的接口风格。
    - ``<cfenv>`` 和 ``<fenv.h>`` 头文件，因为编译器尚不支持。
    - 默认 lambda 捕获。

译者（acgtyrant）笔记
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

#. 实际上，`缺省参数会改变函数签名的前提是改变了它接收的参数数量 <https://www.zhihu.com/question/24439516/answer/27858964>`_，比如把 ``void a()`` 改成 ``void a(int b = 0)``, 开发者改变其代码的初衷也许是，在不改变「代码兼容性」的同时，又提供了可选 int 参数的余地，然而这终究会破坏函数指针上的兼容性，毕竟函数签名确实变了。
#. 此外把自带缺省参数的函数地址赋值给指针时，会丢失缺省参数信息。
#. 我还发现 `滥用缺省参数会害得读者光只看调用代码的话，会误以为其函数接受的参数数量比实际上还要少。 <https://www.zhihu.com/question/24439516/answer/27896004>`_
#. ``friend`` 实际上只对函数／类赋予了对其所在类的访问权限，并不是有效的声明语句。所以除了在头文件类内部写 friend 函数／类，还要在类作用域之外正式地声明一遍，最后在对应的 ``.cc`` 文件加以定义。
#. 本风格指南都强调了「友元应该定义在同一文件内，避免代码读者跑到其它文件查找使用该私有成员的类」。那么可以把其声明放在类声明所在的头文件，定义也放在类定义所在的文件。
#. 由于友元函数／类并不是类的一部分，自然也不会是类可调用的公有接口，于是我主张全集中放在类的尾部，即的数据成员之后，参考 :ref:`声明顺序 <declaration-order>` 。
#. `对使用 C++ 异常处理应具有怎样的态度？ <https://www.zhihu.com/question/22889420>`_ 非常值得一读。
#. 注意初始化 const 对象时，必须在初始化的同时值初始化。
#. 用断言代替无符号整型类型，深有启发。
#. auto 在涉及迭代器的循环语句里挺常用。
#. `Should the trailing return type syntax style become the default for new C++11 programs? <https://stackoverflow.com/questions/11215227/should-the-trailing-return-type-syntax-style-become-the-default-for-new-c11-pr>`_ 讨论了 auto 与尾置返回类型一起用的全新编码风格，值得一看。
