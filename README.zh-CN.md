﻿# Unilang

　　查看本文档内容的不同语言版本：

* [English](README.md)
* **简体中文**

© 2020-2022 统信软件技术有限公司

　　本仓库维护名为 **Unilang** 新编程语言，包括相关的文档和参考实现——一个解释器。

　　解释器的构建和使用参见以下各节。

# 关于新语言

　　Unilang 是为适应更有效和灵活开发桌面环境应用的提出的通用目的编程语言项目。

## 缘起

　　当前桌面应用开发已有许多选项，存在各自的优势和不足：

* [Qt](https://www.qt.io/) 代表的 C/C++ 本机应用开发方案是许多 Linux 桌面系统应用的主流方案。
	* C/C++ 具有成熟的语言标准和实现，以及丰富的开发资源，包括具有较好的厂商中立性的多个语言实现；但同时存在较难学习、项目开发周期往往较长、成本较高的问题。其中大多数全局问题难以短期改善。
		* C/C++ 是最具有可移植性的工业语言的代表。但是，广泛依赖的特性并非被语言标准化，同时依赖很多底层系统的细节，如热加载。
		* C/C++ 的库的资源丰富，但包管理、持续集成和二进制分发兼容性等问题长期以来无法有效解决，碎片化严重，不利于快速部署。
		* C/C++ 作为静态语言，在一些场景下开发效率较低。作为静态类型语言，并不具有非常健壮的类型系统，对开发体验改进有限。
		* C/C++ 能显式管理资源，允许开发大规模的高性能 GUI 应用。但相对地，容易误用，正确实现对开发者的要求较高。
	* Qt 自身具有良好的可移植性，能适应众多主流桌面平台。
		* 但是，因为语言的局限性等技术问题，Qt 还需要依赖专用的语言扩展（而非标准 C++ ），在语言层次上的可移植性和可扩展性相对较差。
		* 相对其它 C/C++ 程序，Qt 部署需要较多的空间。不过，若成为系统库，这个问题影响相对较小。
* [Electron](https://www.electronjs.org/) 代表的非本机和动态语言运行时为基础的开发方案是另一类较主流的方案。
	* 使用流行的动态语言能克服一些静态语言不够灵活的问题，但有时难以保障质量。
	* 依赖 [GC](https://en.wikipedia.org/wiki/Garbage_collection_%28computer_science%29) 不需要显式管理内存，减小了一些开发难度，但大多数开发者难以有效优化运行时机制，而容易造成内存泄漏等问题极大影响 GUI 应用的质量和开发体验。
	* 往往需要部署巨大的运行时。
	* 一些运行时的实现可能有冷启动的性能问题。
* [PySide](https://wiki.qt.io/PySide) 代表的本机和动态语言混合方案能解决以上两类方案的部分问题。
	* 但是，这类方案不能自动解决本机语言和动态语言自身带来的问题。
	* 这同时要求开发者对基础方案均有了解，并不保证更易用。否则，一旦使用不当，则可能集成两者的缺陷，而非优势。
* [Flutter](https://flutter.dev/) 代表的移动端解决方案也正在向桌面移植。
	* 桌面端相对不够成熟。
	* 因为通常也具有动态语言，同时具有部分其它动态语言运行时的方案类似的问题。

	从更高层的结构角度来看，不同类型的 GUI 方案也各自存在不同的架构意义上的技术局限性，也极大地限制了真正的通用方案的选择余地：

* 和传统的*保留模式(retained mode)* GUI 相比，[Dear ImGUI](https://github.com/ocornut/imgui) 为代表的[*立即模式(immediate mode)* GUI](https://en.wikipedia.org/wiki/Immediate_mode_GUI) 缺乏 GUI 作为实体的抽象，不能很好地对应传统的 [WIMP 隐喻](https://en.wikipedia.org/wiki/WIMP_%28computing%29)。
	* 所谓的[立即模式](https://en.wikipedia.org/wiki/Immediate_mode_%28computer_graphics%29)原本专注于图形渲染，对 UI 交互缺乏关注，因此即便实现了良好的显示输出，仍需要大量工作改善交互性。
	* 因为对中间状态的简化，立即模式基本上难以扩展而实现 UI 自动化接口。
* 依赖随系统提供的“本机”方案难以克服底层实现的具体功能局限。
	* 例如，[Win32 窗口样式 `WS_EX_LAYERED` 在 Windows 8 之前仅在顶层窗口而不在子窗口中受支持](https://docs.microsoft.com/windows/win32/winmsg/extended-window-styles)。
	* 考虑到和系统实现的耦合性，这实际上意味着依赖系统提供实现的坚持“原生”的策略的方案（如 [wxWidgets](https://www.wxwidgets.org/) ），不可能可靠地提供一致的、可移植的用户体验，甚至是同一个系统的不同版本中。
	* 这种不便甚至在系统厂商内部被承认。
		* 封装 Win32 的 [MFC](https://docs.microsoft.com/cpp/mfc/mfc-desktop-applications) 和 [WinForms](https://docs.microsoft.com/dotnet/desktop/winforms) 日趋淘汰，被所谓的重新实现绘制逻辑的 direct UI（意为不依赖 Win32 `HWND`）如 [WPF](https://docs.microsoft.com/dotnet/desktop/wpf) 等普遍取代。
		* 相对其它方案，[WinUI](https://microsoft.github.io/microsoft-ui-xaml/) 很早（自 WinUI 2 ）就直接放弃了对（不那么）旧的版本的操作系统的支持，对旧版本系统的依赖也是一个因素。
* Web 图形客户端（浏览器）为基础的 GUI 具有良好的可移植性和灵活性，但是有其它一些特有问题：
	* Web 实现的灵活性主要通过客户端语言的有限组合，具体即 [JavaScript](https://www.ecma-international.org/publications-and-standards/standards/ecma-262/) 、[CSS](https://www.w3.org/TR/CSS/#css) 和 [HTML](https://html.spec.whatwg.org/multipage/) 。这些语言的缺陷会长期存在。
		* [WebAssembly](https://webassembly.org/) 是一个补充，可预见的未来不能取代 JavaScript 的地位，更无法替代 JavaScript 以外的其它 DSL 。
		* 这些技术并非能确保良好支持本机应用的开发。
			* 历史上，HTML 呈现 Web 中的静态的文档（称为“页面”）而不是交互式的动态程序而设计。
			* JavaScript 和 CSS 角色上把静态页面转换为动态内容的客户端补丁，在早期也曾经功能严重受限（因此有 [Flash](https://en.wikipedia.org/wiki/Adobe_Flash) 等其它技术的流行）。
			* 即便 [DOM](https://dom.spec.whatwg.org/) 等标准化的技术规格能简化大量实现细节，跨浏览器不兼容时有发生。所幸 JavaScript 作为动态语言易于通过 [shim](https://en.wikipedia.org/wiki/Shim_%28computing%29) （而无需重新部署运行环境）等方式减少问题，但这至少以适配工作量为代价，而且不总是可行。例如，缺乏对 [ES6 的 PTC(proper tail call)](https://262.ecma-international.org/6.0/#sec-tail-position-calls) 的支持基本不能通过不修改底层实现解决。
	* 实际的实现极端复杂，明显比其它方案更加难以定制。
		* 浏览器的核心部分（实现 HTML 和 CSS 等的排版引擎和 JavaScript 等语言的运行时）是高度封装的集成组件，大多使用 C/C++ 实现，却比几乎所有 C/C++ 程序更难修改和裁剪。
		* 因此，基于 Web 的 GUI 方案，基本只能捆绑这些原生组件而不能做出大的改动。即便应用不依赖的功能也不便在部署前被移除。除非被系统分发，这将严重使程序的体积膨胀。
	* 因为传统因素（如对安全性的担忧），Web 程序和本机原生环境的交互受到限制，开发桌面应用可能需要不少附加工作量。
* 依赖不同其它组件的混合框架存在对应的路径依赖问题。
	* 如依赖系统提供的本机 GUI 实现的框架，会遇到上述本机方案的问题。
	* 依赖 Web 的框架（如 Electron、[Cordova](https://cordova.apache.org/)、[Tauri](https://tauri.app/) ），会有上述基于 Web 方案的问题。
	* 如果都依赖，类似的局限性最终都会被共同继承。
	* 不过，如果只是把这些目标作为可选的输出配置之一（例如，把 WebAssembly 作为生成目标），则不会有这些问题。
* 以 Qt 为代表的本机而非操作系统原生的传统 GUI 框架，鲜有和上述问题能相提并论的全局架构缺陷，但在实现架构和 API 设计上仍然存在诸多问题，开发体验也不尽如人意。

　　所以，没有任何现有方案能兼顾各种不同的问题而成为没有疑义的桌面开发首选方案。

　　以上问题中，有相当一部分（性能、部署难度、可移植性）和语言直接相关。考察其中语言部分的问题，我们发现，现有语言不足以兼顾所有这些主要问题，因为：

* C、C++、ECMAScript 等最流行的一些标准化语言具有沉重的历史包袱，且不具有足够扩展语言自身的能力以兼顾这些需求。
* Dart 等专为类似方案设计的语言在一些基本设计上的决策（如依赖全局 GC ）使之无法完全适合一些重要场景。
* 其它的一些通用目的语言，如 [Rust](https://www.rust-lang.org/) 和 [Go](https://golang.org) ，并没有配套提出 GUI 解决方案。
	* 尽管存在一些第三方 GUI 项目，它们的设计中的优势也并不能很好地适应桌面应用开发的需求。
	* 语言社区也没有以开发桌面应用为主要重点推进。

　　我们迫切希望有一种新的语言解决以上所有痛点。但是，仅仅提供一种新的语言设计和实现是不够的。新的语言不是自动解决遗留问题的魔法——特别是考虑到市面上并不缺乏“新”的编程语言，却仍未满足需求的现状。

　　造成这种局面的一个技术理由是，许多设计过于专注具体需求而缺乏考虑语言长期演进的普遍因素，在预期目标领域之外的适用性急剧下降，不够通用，或在平衡通用性和复杂性上失败了。这使得应用领域和预期略有偏差、暴露原有设计的局限性时，用户即便懂得如何改进一个语言，也会在语言二次开发上遇到现实可行的困难，而被迫放弃。

　　若提出新的选项而不能避免这种状况，只会更加阻碍问题的解决。因此，在能满足需求的基础上，我们希望新的语言能以更深刻的方式真正地实现通用性——通过减少为个别问题领域准备的原生的*特设的(ad-hoc)* 特性，而以更普遍的基本特性集取而代之的方式。

> Programming languages should be designed not by piling feature on top of feature, but by removing the weaknesses and restrictions that make additional features appear necessary.

<p align="right">—— <a href="https://schemers.org/Documents/Standards/">R<sup>n</sup>RS</a> & <a href="https://ftp.cs.wpi.edu/pub/techreports/pdf/05-07.pdf">R<sup>-1</sup>RK</a></p>

## 新语言的必要性

　　好吧，但是为什么要用一个**新**语言呢？

　　我们知道，编程语言的大多数属性是通过语言规则指定的*特性(feature)* 提供的。此外，通过故意缺乏一组特定的特性（即*故障特性(misfeature) *），可以保证某些其他所需的特性。

　　用户想要的是特性，而不是错误特性。事实上，用户所需的具体特性和错误特性集并不是完全固定的。向当前编程语言添加新特性可能很容易。然而，由于（不）兼容性方面的风险，从语言中移除错误特性几乎总是相当困难的（如果不是不可能的话），除非语言从设计上就预期了这种移除。

　　决定一个特性是否满足我们的需求，需要考虑许多技术问题。其他情形可能需要不同的决策。所以，公共的特性集可能出人意料地小。但是几乎所有工业语言都提供得太多了，而那些多余的特性又不容易按需被用户（而非语言设计者）添加回来。这种情形意味着许多错误特性是不可避免的。

　　出于实用原因，我们决定不改变当前的语言（这可能会增加语言的复杂性，并给一些用户带来更多的不功能），而是重新发明我们自己的轮子，以解决问题。

　　我们也更倾向于只使用**一种**语言来满足需求。否则，我们也可以有几种语言。这些语言自然适用于不同的问题领域，即*领域特定语言(DSL)* 。

　　当这些不同领域共享很少的工作时，后者可能运作良好。另一方面，根据我们的经验，对于由一些相关子域组成的稍大的问题域，维护不同的 DSL 是一种痛苦。互操作性方面也存在不必要的成本。即使我们想以不同的工具集专门用于不同的问题域，也应在不同的语言之间复用更多的公共设计和实现。它们在设计上不应完全独立。换句话说，DSL 的替代品最好是通用语言的方言。

　　一些用户可能会关心这里的“一个”语言的感官体验。的确，一种唯一的、通用的*语法(syntax)* 很难跨不同的问题领域，因为语法上存在冲突需求：我们可能需要完全不同的视觉风格来实现不同的目的。但实际上，没有任何定律排除在一种语言中同时支持多组具体语法的可能性。

　　因此，语言的语法不应是问题。如果语法不令人满意，那就改变它。这是由用户而非语言的设计者完成的。这当然会有一些成本，但一旦已有正确的底层语言设计，改动的代价就应比设计和实现完全不同的语言代价更小。

## 方法论

　　如前所述，要设计一种真正通用的语言，我们不希望将特性堆砌在现有语言之上。

　　我们没有直接指定丰富的特性集，而是首先提供了*基础语言*，它只具有足够基本的特性。这些特性可用于以库 API 的形式实现其它特性。这里，语言被*派生(derive)* 。

　　基础语言是专门为易于集成而设计的。换句话说，除了公共特征集之外，它还服务于语言派生的问题域，这被大多数编程语言所忽略。与大多数（如果不是所有）主流编程语言相比，它在这方面应该相对容易使用。具体来讲，基础语言在没有其他语言中许多流行的核心语言特性（如类型系统）的情况下仍具有相当可用性。这大大降低了所有语言派生中不成熟设计蕴含的错误特性的风险。

　　我们理解用户可能需要本机语言设计中未原生提供的开箱即用的特性。这通过允许语言的基本特性更可编程来解决，即大多数特性都在库中。

　　我们也鼓励用户添加来自当前语言的新功能集，并以库的形式为我们提供贡献。

　　这种方法类似较小粒度锁改进争用，提高了语言进化过程中潜在的并行性。我们已经确定了一些主要程序语言在开发过程中的具有的较高的通信成本，并试图以这种方式防止一些可预见的问题。这可比任何替代方法更有效。

## 如何实现 GUI ？

　　GUI 是一个现实的包含大量非平凡编程主要的问题领域。这个领域同时相当重要和复杂。特别地，它有很多不同的问题子领域，而许多解决方案已经选择了不同 DSL 的混合使用。因此，这对实验我们的方法论而言，是一个很好的例子。

　　由于问题的内稟复杂性，我们不期望万能的解决方案能够满足所有需求。相对地，我们有不同的计划来改善现状。

　　在所有计划中，语言的动态性起着重要作用。我们认为，对于部件布局、层次结构、运行时对象内省(inspection) 和热重载等特性，最好是动态的。在静态语言中实现它们的任何努力都需要首先实现程序的动态描述，如果它是可重用的，则本质上实现动态语言。避免了需要选择不同的解决方案来解决问题，这是对许多技术的改进。

　　即使与已经使用某些动态语言的当前解决方案相比，也有更多不同的改进：

* 只需要一种语言，而不是多种语言的混合。
* 自定义和扩展专门用于任何领域（包括 GUI ）的语言功能更容易。
* 以精确的生命周期管理资源更容易，因为不需要GC。
* 最终用户的定制可能更容易，因为为最终用户嵌入 DSL（如果需要）可以共享语言实现的大部分语义属性，且可能只需要对语法进行少量调整。

　　（此处待添加更多具体工作。）

## 特色

　　Unilang 是为了统筹解决现有不足的新的方案中的语言部分，主要特色有：

* 作为动态语言，提供相对其它语言更强的语言层次上的可扩展性。
	* 大多数语言中需要修改语言核心规则提供的特性，在 Unilang 中预期只需要用户使用 Unilang 语言编写的库解决，例如静态类型检查可以通过用户程序提供。
		* 通过用户定制语言的功能，可以有效限制非预期的动态特性，**最终得到和大多数静态语言接近的开发体验上的优势，同时避免静态语言核心规则带来的不便**。
		* 允许在已部署 Unilang 程序的环境中通过添加库补全现有语言特性，而不需要重新部署工具链的实现。
		* 提供一个基础语言，并以库的形式扩展这个语言而得到实用的特性集。库预期由本项目和用户提供。
	* 类似 C 和 C++ 而不同于 [Java](https://docs.oracle.com/javase/specs/jls/se18/html/jls-1.html) ，不明确要求或假定翻译和执行的具体形式。实现使用编译、解释和何种映像格式加载等实现细节对核心语言规则透明。
	* 不预设如 C 和 C++ 那样明确的[*翻译阶段(phases of translation)*](http://eel.is/c++draft/lex.phases) 。不需要单独阶段展开的宏——配合支持一等环境的函数即可取代宏。
	* 支持[同像性(homoiconicity)](https://en.wikipedia.org/wiki/Homoiconicity) ，允许代码即数据(code as data) 的方式编程。
	* 函数是[一等对象(first-class object)](https://en.wikipedia.org/wiki/First-class_citizen) 。
	* *环境(environment)* 对变量绑定具有所有权。支持作为一等对象的*一等环境(first-class environment)* 。
* 支持类似 C++ 的对象模型和（当前不被检查的）不安全所有权语义。
	* 和 C# 或 Rust 等不同，不提供一种特设的 `unsafe` 关键字标记“不安全”的代码段落，最基础的特性默认是“不安全”的。
	* 安全性并非由语言唯一地定义，允许用户通过扩展类型系统等方式实现自定义的不同种和程度的安全性。
* 不要求全局 GC ，同时语言的一个子集允许和 C++ 同等层次的“不安全”但能确保确定性的资源分配。
	* 没有原生提供针对不安全操作的静态检查，但是语言的可扩展性允许直接实现类型系统或者自动证明更强的内存安全。未来可能作为库一并提供。
	* 语言规则仍然允许引入依赖 GC 的互操作。特别地，允许引入多个非全局的 GC 示例。
* 支持正式意义上的 [PTC](https://www.researchgate.net/profile/William_Clinger/publication/2728133_Proper_Tail_Recursion_and_Space_Efficiency/links/02e7e53624927461c8000000/Proper-Tail-Recursion-and-Space-Efficiency.pdf) ，而不需要用户程序内对栈溢出等未定义行为进行变通。
	* 主流语言中，没有依赖全局 GC 的语言实现都没有提供类似的保证。
* 使用隐式的[*潜在类型(latent typing)*](https://en.wikipedia.org/wiki/Latent_typing) 而非显式的[*清单类型(manifest typing)*](https://en.wikipedia.org/wiki/Manifest_typing) 。
	* 这自然地避免用户扩展的类型系统和原生规则冲突，而保持可扩展性。
		* 在扩展前，作为实现细节，已允许蕴含[*类型推断(type inference)*](https://en.wikipedia.org/wiki/Type_inference) 消除一些类型检查而不影响程序语义。
		* 允许用户程序扩展*类型标注(type annotation)* 的语法和相关检查。
	* 表达式和 C++ 类似但略有不同的[*值类别(value category)*](http://eel.is/c++draft/basic.lval) ；但和 C++ 不同，不是静态确定的表达式的属性，而是跟随对象的动态元数据。
	* 类似 C++ 的 `const` 类型限定符，通过左值引用的对象允许标记为不可修改（只读），而不是如 Rust 等语言默认约定值*不可变(immutable)* 。
	* 类似 C++ 的*消亡值(xvalue)* ，通过左值引用的对象允许标记唯一，而允许其中的资源被转移。
* **原理** 以上代表性的选型决策中，一个共通的方法是比较不同方向的扩展之间的技术可行性——并选取容易扩展的选项。否则，即便可行，也有许多本应避免的无效的工作。
	* 设计一个静态语言，然后添加一些规则把它伪装成具有足够动态特性的动态语言，远远难于在动态语言上添加规则而得到静态语言的特性集。
		* 因此，基础语言首先被设计为动态语言。
	* 从一个标记放弃某种保证的上下文中添加证明恢复某种保证（且不和其它保证冲突），比从一个已知不具有保证的上下文添加证明以确保保证更困难。
		* 例如，用 `unsafe` 等特设语法标记“不安全”的语言中，通常会放弃语言定义的任意安全保证，而不能选择保留其中的一部分。即便忽略这个问题，语言也缺乏机制允许用户提供更严格的保证。
			* 因此，基础语言默认是不安全的。
		* 再如，默认不可变的数据结构虽然能保证 [const correctness](https://isocpp.org/wiki/faq/const-correctness) 这样的“正确性”（一种保持被限定的不可变性质不被丢弃的[*类型安全性(type safety)*](https://en.wikipedia.org/wiki/Type_safety)），却忽略对“不可变”的定义描述不充分而不能让用户程序扩展的问题——很多情形下，不可变仅仅需要是一种等价关系，而并非不可修改。
			* 这可能导致具体的不可修改性被滥用，例如 C++ 标准库关联容器的键类型实际上不需要符合 C++ 的 `const` ，因为键的不可变确切地由[比较对象导出的等价关系](http://www.eel.is/c++draft/associative.reqmts#general-3)定义，但类型系统无法区别两种情形。这过度地限制了键上的本应允许的操作。
				* 这里用 `const_cast` 这样的不安全转换取消 `const` 引入的类型安全保证并自行假定不会破坏不可变性，是个无奈的变通（“更困难”的情形，且无法恢复类型安全性而效果更差）。
			* 默认不可变的类型系统，如 Rust 的设计，则更根本地在类型组合构造上阻止了扩展的方向。
				* 这蕴含不可变性只有一种，除非修改类型系统设计，放弃原有的不可变定义并重新引入类似 C++ `const` 的限定符机制（“更困难”的情形）。
			* 这也限制了现有实现中的[*常量传播(const propagation)*](https://zh.wikipedia.org/zh-cn/%E5%B8%B8%E6%95%B8%E6%8A%98%E7%96%8A#%E5%B8%B8%E6%95%B8%E5%82%B3%E6%92%AD) 的优化范围，因为原则上这里的“常量”只关心替换能保持变换前后的语义等价性(semantic-preserving)（对 C++ ，这是关于[抽象机语义](http://www.eel.is/c++draft/intro.compliance#intro.abstract-1)上的），而不在意具体的值是否相等。
				* 若语言允许用户表达“一些具有不同表示的值被视为等价”，则优化的适应性会自然地扩展。
			* 因此，基础语言中的对象默认可变。
	* 在一个已经要求全局 GC 的语言中排除 GC ，远远难于在不依赖 GC 的语言规则基础上添加和 GC 交互的能力（特别是 GC 允许用户定制时）。
		* 因此，语言首先排除对全局 GC 的依赖。
	* 在一个没有 PTC 保证的语言实现中添加扩展基本是不可行的，除非重新实现最核心的求值规则在内的逻辑（例如，再添加一个执行引擎）。
		* 因此，语言首先要求 PTC 以使其实现有足够的可用性，而不鼓励嵌套的不可靠的实现。
		* 注意对 PTC 不可靠的实现方式在其它方面仍可能很成功（如翻译 ECMAScript 方言的 [Babel](https://babeljs.io/) ）。因此，大多数其它特性并没有（也不需要有）如 PTC 需要基本规则明确担保的地位。
* 和 C++ 具有良好的互操作性。
	* 当前解释器（运行时）使用 C++ 实现。
	* 结合对象模型，能确保 Unilang 对象和 C++ 对象的对应。
	* 语言绑定主要关注已知 ABI 的 C/C++ API 。

　　为保持通用性，Unilang 不内建提供 GUI 功能，而通过库提供相关 API 。当前计划中，Unilang 将会支持基于 Qt 的绑定的库，以便衔接过渡现有的一些桌面应用项目。Unilang 的语言设计保持足够的抽象能力和可扩展性，允许在未来直接实现 GUI 框架。

# 文档说明

　　本项目包含以下文档：

* `README` ：本文档，介绍项目的整体状况，使用方法和支持的主要功能，附更新记录。
	* 预期提供给所有对本项目有兴趣的读者。
* [语言规范文档](doc/Language.zh-CN.md)：正式的语言规范，包含实现的基准、要求支持的特性和部分解释性说明。
	* 供项目的贡献者、语言及其实现最终用户（开发者）参考。
	* 是确定语言设计和语言实现（解释器和库代码）存在缺陷的主要依据。
* [解释器实现文档](doc/Interpreter.zh-CN.md)：描述解释器中一般不作为语言特性公开的设计以及对应的实现，也包含一些和项目演进和设计决策相关的原理说明。
	* 预期给项目的维护者（解释器的实现者）和对改进语言设计有兴趣的读者参考。
* [语言介绍文档](doc/Introduction.zh-CN.md)：介绍语言的使用方式。
	* 预期作为语言首选的入门文档。
	* 建议所有本项目（语言、解释器和库）的用户阅读。
* [特性介绍文档](doc/Features.zh-CN.md)：补充语言介绍文档，列出特性列表，介绍其主要使用方式。
	* 预期给语言的最终用户（开发者）参考。
	* 建议需要对语言的使用深入了解和需要提议新语言特性的用户阅读。

　　项目的贡献者一般应能确定以上文档中的内容和对应实现的修改（若存在）的关联性。

　　若文档的内容之间存在逻辑矛盾或者以下不一致，请联系维护者报告缺陷。

## 一致性

　　文档不应当提供不符合被描述的对象的误导性信息。

　　文档通常应保持和本仓库中的其它部分（包括作为参考实现的解释器及标准库代码）一致。

　　但是，本项目在早期开发阶段，部分文档的一致性的要求可能存在差异：

* 语言实现*符合(conform)* 语言规范。
* 在每个（以 Git 标签）发布的版本之间，解释器实现文档、语言介绍文档和特性介绍文档可能与当前提交的开发版本（在 `master` 分支）有限的不一致。
* 语言介绍文档和特性介绍文档（相对语言规范）不完整，即并非所有特性都被涵盖。
* 其它情形下，所有其它内容符合 `README` 中的要求。

　　不满足以上要求的文档不一致应当视为文档或被文档描述的其它对象的缺陷。

# 构建

　　本项目支持不同方法构建。

　　支持的宿主环境为 MSYS2 MinGW32 和 Linux 。

　　以下使用版本库根目录作为当前工作目录。

## 构建环境依赖

　　一些外部依赖项的源代码在版本库及 git 子模块中提供。

　　构建环境依赖以下环境工具：

* `git`
* `bash`
* [GNU coreutils](https://www.gnu.org/software/coreutils/)
* 支持 ISO C++ 11 的 G++ 和与之兼容的 GNU binutils

　　可选依赖：

* 可使用 Clang++ 代替 G++ 。

　　构建使用外部二进制依赖和相关工具：

* libffi
* LLVM 7
	* `llvm-config`
* Qt 5
* `pkg-config`

　　安装构建环境依赖的包管理器命令行举例：

```
# Some dependencies may have been preinstalled.
# MSYS2
pacman -S --needed bash coreutils git mingw-w64-x86_64-gcc mingw-w64-x86_64-binutils mingw-w64-x86_64-libffi mingw-w64-x86_64-llvm mingw-w64-x86_64-pkgconf mingw-w64-x86_64-qt5-base mingw-w64-x86_64-qt5-declarative
# Arch Linux
sudo pacman -S --needed bash coreutils git gcc binutils libffi pkgconf qt5-base qt5-declarative
yay -S llvm70 # Or some other AUR frontend command.
# Debian (buster/bullseye)/Ubuntu (bionic-updates/focal)/Deepin
sudo apt install bash coreutils git g++ libffi-dev llvm-7-dev pkg-config qtbase5-dev qtdeclarative5-dev
```

　　若系统不提供 LLVM 7 包，可能需要自行构建。本项目中，环境变量 `USE_LLVM_PREFIX` 指定自定义的 LLVM 的安装路径前缀，被脚本按需使用。

　　另见以下的[环境配置](#环境配置)安装更多可选的依赖。

### Qt 环境要求和假设

* 使用 [Itanium C++ ABI](https://itanium-cxx-abi.github.io/cxx-abi/abi.html) 。
* 不支持 `QT_NAMESPACE` 。
* 直接依赖文件系统中的安装的 Qt 文件。
* 以下依赖项应当被 `pkg-config` 找到：
	* `Qt5Widgets`
	* `Qt5Quick`

## 构建环境更新

　　构建之前，在版本库根目录运行以下命令确保外部依赖项：

```
git submodule update --init
```

　　若实际发生更新，且之前执行过 `install-sbuild.sh` 脚本，需清理补丁标记文件以确保再次执行这个脚本时能继续正确地处理源代码：

```
rm -f 3rdparty/.patched
```

　　使用以下 `git` 命令也能清理文件：

```
git clean -f -X 3rdparty
```

## 使用直接构建脚本

　　运行脚本 `build.sh` 直接构建，在当前工作目录输出可执行文件：

```
./build.sh
```
　　默认使用 `g++` 。环境变量 `CXX` 可指定要使用的其它替代，如：

```
env CXX=clang++ ./build.sh
```

　　默认使用 `-std=c++11 -Wall -Wextra -g` 编译器选项。类似地，使用环境变量 `CXXFLAGS` 可替代默认值。

　　这个脚本使用 shell 命令行调用 `$CXX` 指定的编译器驱动，不支持并行构建，可能较慢。

　　优点是不需要进一步配置环境即可使用。适合一次性测试和部署。

## 使用外部工具构建脚本

　　利用[外部工具的脚本](https://frankhb.github.io/YSLib-book/Tools/Scripts.zh-CN.html)，可支持更多的构建配置。这个方式相比直接构建脚本更适合开发。

　　当前 Linux 平台只支持 x86_64 宿主架构。

　　以下设并行构建任务数 `$(nproc)` 。可在命令中单独指定其它正整数的值代替。

### 环境配置

　　配置环境完成工具和依赖项（包括动态库）的安装，仅需一次。（但更新子模块后一般建议重新配置。）

　　安装的文件由 `3rdparty/YSLib` 中的源代码构建。

　　对 Linux 平台构建目标，首先需确保构建过程使用的外部依赖被安装：

* [freetype2](https://freetype.org/)

　　例如，使用包管理器：

```
# Arch Linux
sudo pacman -S freetype2 --needed
# Debian/Ubuntu/Deepin
sudo apt install libfreetype6-dev
```

　　为以下脚本中自动更新二进制依赖和对源文件补丁，需要以下依赖：

* `wget`
* `7za`
* `sed`（避免可能破坏行尾的 Win32 版本）

　　例如，使用包管理器：

```
# MSYS2
# XXX: Do not use mingw-w64-x86_64-sed to ensure the EOL characters as-is.
pacman -S --needed mingw-w64-x86_64-wget p7zip sed
# Arch Linux
sudo pacman -S --needed wget p7zip sed
yay -S llvm70 # Or some other AUR frontend command.
# Debian (buster/bullseye)/Ubuntu (bionic-updates/focal)/Deepin
sudo apt install wget p7zip-full sed
```

　　运行脚本 `./install-sbuild.sh` 安装外部工具和库。脚本更新预编译的二进制依赖之后，构建和部署工具和库。其中，二进制依赖直接被部署到源码树中。当前二进制依赖只支持 `x86_64-linux-gnu` 。本项目构建输出的文件分发时不需要依赖其中的二进制文件。

**注释** 脚本安装的二进制依赖可能会随构建环境更新改变，但当前本项目保证不依赖其中可能存在的二进制不兼容的部分。因此，二进制依赖的更新是可选的。但是，在构建环境更新后，一般仍需再次运行脚本配置环境，以确保覆盖安装外部工具和（非二进制依赖形式分发的）库的最新版本。其中，若二进制依赖文件不再在脚本预期的部署位置中存在，脚本会从网络重新获取最新版本的二进制依赖。

　　以下环境变量控制脚本的行为：

* `SHBuild_BuildOpt` ：构建选项。默认值为 `-xj,$(nproc)` ，其中 `$(nproc)` 是并行构建任务数。可调整 `$(nproc)` 为其它正整数。
* `SHBuild_SysRoot` ：安装根目录。默认指定值指定目录 `"3rdparty/YSLib/sysroot"` 。
* `SHBuild_BuildDir` ：中间文件安装的目录。默认值指定目录 `"3rdparty/YSLib/build"` 。
* `SHBuild_Rebuild_S1` ：非空值指定重新构建 [stage 1 SHBuild](https://frankhb.github.io/YSLib-book/Tools/SHBuild.zh-CN.html#%E5%A4%9A%E9%98%B6%E6%AE%B5%E6%9E%84%E5%BB%BA)（较慢）。
	* **注意** 构建环境更新 `3rdparty/YSLib/Tools/Scripts` 的文件后，需指定此环境变量为非空值，以避免可能和更新后的文件不兼容的问题。
	* 其它情形不必要。建议忽略，以提升安装时的构建性能。

　　使用安装的二进制工具和动态库需配置路径，如下：

```
# Configure PATH.
export PATH=$(realpath "$SHBuild_SysRoot/usr/bin"):$PATH
# Configure LD_LIBRARY_PATH (reqiured for Linux with non-default search path).
export LD_LIBRARY_PATH=$(realpath "$SHBuild_SysRoot/usr/lib"):$LD_LIBRARY_PATH
```

　　以上 `export` 命令的逻辑可放到 shell 启动脚本（如 `.bash_profile` ）中而不需重复配置。

### 构建命令

　　配置环境后，运行脚本 `sbuild.sh` 构建。

　　和直接构建脚本相比，支持并行构建，且支持不同的配置，如：

```
./sbuild.sh release-static -xj,$(nproc)
```

　　则默认在 `build/.release-static` 目录下输出构建的文件。为避免和中间目录冲突，输出的可执行文件后缀名统一为 `.exe` 。

　　此处 `release-static` 是**配置名称**。

　　设非空的配置名称为 `$CONF` 。当 `$SHBuild_BuildDir` 非空时输出文件目录是 `SHBuild_BuildDir/.$CONF` ；否则，输出文件目录是 `build/.$CONF` 。

　　当 `$CONF` 前缀为 `debug` 时，使用调试版本的库（已在先前的构建环境安装步骤中从 `3rdparty` 的源代码构建），否则使用非调试版本的库。当 `$CONF` 后缀为 `static` 时，使用静态库，否则使用动态库。使用动态库的可执行文件依赖先前设置的 `LD_LIBRARY_PATH` 路径下的动态库文件。

　　运行直接构建脚本使链接静态库，大致相当于此处使用非 debug 静态库构建。

# 运行

## 运行环境

　　使用上述动态库配置构建的解释器可执行文件在运行时依赖对应的动态库文件。此时，需确保对应的库文件能被系统搜索到（以下运行环境配置已在前述的开发环境配置中包含），如：

```
# MinGW32
export PATH=$(realpath "$SHBuild_SysRoot/usr/bin"):$PATH
```

```
# Linux
export LD_LIBRARY_PATH=$(realpath "$SHBuild_SysRoot/usr/lib"):$LD_LIBRARY_PATH
```

　　若使用系统包管理器以外的方式安装 LLVM 运行时库到非默认位置，类似添加 LLVM 的路径，如：

```
# Linux
export LD_LIBRARY_PATH=/opt/llvm70/lib:$LD_LIBRARY_PATH
```

　　以上 Linux 配置的 `LD_LIBRARY_PATH` 也可通过 [`ldconfig`](https://man7.org/linux/man-pages/man8/ldconfig.8.html) 等其它方式代替。

　　使用静态链接构建的版本不需要这样的运行环境配置；不过 LLVM 通常使用动态库。

**注意** 非脚本配置的外部二进制依赖项可能不兼容，需要通过系统包管理器等方式部署，依赖这些库导致解释器最终的二进制文件不保证跨系统环境（如不同 Linux 发行版）之间可移植。

## 运行解释器

　　运行解释器可执行文件直接进入交互模式运行 REPL ；或在命令行指定一个脚本，进入脚本模式执行脚本中的源程序。脚本名称 `-` 被视为标准输入。

　　运行解释器时使用命令行选项 `-e` 可在进入交互模式或脚本模式前直接求值字符串参数。选项 `-e` 可以使用多次，每个选项后具有一个命令行参数，这些参数字符串被作为 Unilang 源代码顺序求值。

　　解释器命令行支持 POSIX 约定，在命令行参数 `--` 之后的其它参数不被解释为选项。这允许指定和选项重名的脚本文件。

　　命令行选项 `-h` 或 `--help` 显示解释器命令行的帮助。

　　解释器处理以下可选环境变量：

* `ECHO`：非空值启用 REPL 回显。这确保解释器在每个交互会话后输出求值结果。
* `UNILANG_NO_JIT`：非空值停用基于 JIT 编译的代码执行优化，使用纯解释器。
* `UNILANG_NO_SRCINFO`：非空值停用用于诊断消息输出的从源文件取得的源代码信息。源文件名仍被诊断消息使用。
* `UNILANG_PATH`：指定库加载路径。详见[语言规范](doc/Language.zh-CN.md)对标准库函数 `load` 的说明以及[解释器实现](doc/Interpreter.zh-CN.md)对标准库模块操作的说明。

　　除使用选项 `-e` ，配合外部的 `echo` 命令，也可支持非交互式输入，如：

```
echo 'display "Hello world."; () newline' | ./unilang
```

### Qt Demo

　　示例中包含使用 QtWidgets 的程序。

```
./unilang demo/qt.txt
```

　　等价的 Python 实现参考 `demo/qt.py` 。

　　另一个使用 QtQuick 的示例类似 Qt 官方 `qmlscene` 工具的最小化版本：

```
./unilang demo/qml.txt
```

　　这个示例加载相对当前工作目录的源文件 `demo/hello.qml` 。当前工作目录可以是存储库的根目录。

### Quicksort demo

```
./unilang demo/quicksort.txt
```

## 运行测试脚本

　　文件 `test.sh` 是测试脚本。可以直接运行测试用例。脚本在其中调用解释器。

　　测试用例直接在脚本代码中指定，包括调用解释器运行测试程序 `test.txt`。在 REPL 中 `load "test.txt"` 也可加载测试程序。

　　脚本以下支持环境变量：

* `UNILANG` ：指定解释器可执行文件路径，默认为 `./unilang` 。
* `PTC` ：非空时，运行 PTC 测试用例。手动终止进程后结束用例。在此期间，正确的 PTC 实现可确保最终内存占用不随时间增长。

　　使用 `sbuild.sh` 构建的可执行文件不在当前目录。可使用类似以下的 `bash` 命令调用：

```
UNILANG=build/.debug/unilang.exe ./test.sh
```

# 支持的语言特性

　　语言特性可参照[《Unilang 介绍》](doc/Introduction.zh-CN.md)中的例子（尚未完全支持）和[特性清单](doc/Features.zh-CN.md)。

　　另见[语言规范](doc/Language.zh-CN.md)和[解释器设计和实现文档](Interpreter.zh-CN.md)。

## 已知问题

　　不精确数使用 C++ 标准库 `<cstdio>` 兼容格式输出，可能在非默认区域(locale) 设置中输出非预期的格式，如：

* 小数点不是 `.` 。
* 数值中出现小数点、符号和指数字符以外的非数字分隔符。

　　当前版本在非默认区域下不确保这些输出能被作为 Unilang 数值字面量解析。

# 贡献指南

　　我们鼓励您报告问题和贡献修改。

　　基本事项参见[开发者代码贡献指南](https://github.com/linuxdeepin/developer-center/wiki/Contribution-Guidelines-for-Developers)。

　　但是，以下可能**不同**的规则优先适用于本项目：

* 文档中的正式条款使用 [ISO/IEC Directive 2](https://www.iso.org/sites/directives/current/part2/index.xhtml) 规定的动词用法。
	* **注释** 这比 [RFC 2119](https://www.ietf.org/rfc/rfc2119.txt) 更严格。
* 版本元数据：
	* 版本应当符合[语义化版本](https://semver.org/lang/zh-CN/)中的约定。
	* 显式的版本号（包括在 Git 标签中的）由维护者更新。
* 提交消息：
	* 若改动仅限非英文内容，应当使用对应的非英文消息。否则，应当使用英文消息。
	* 不要求使用[约定式提交](https://www.conventionalcommits.org/zh-hans/v1.0.0/)。
		* 标题的格式为 `[可选的范围: ]<描述>`。
			* **注释** 范围没有括号；可选的字符序列 `: ` 仍然要求是英文标点，从属于可选的范围描述中。
			* 范围应指定为仓库下的目录。仅当改动范围不能被归类为仓库的顶层目录时视为全局的，此时范围应被省略。
		* 标题和正文中的描述应为陈述句。
		* 正文和脚注的其它格式和约定式提交相同，除了不要求 **BREAKING CHANGE** 。
		* **原理**
			* 和广泛使用的 [Git trailers](https://git-scm.com/docs/git-interpret-trailers) 不同，大多数流行的项目都没有使用约定式提交。
			* 当前本项目没有部署依赖约定式提交的自动化工具，不会提取*类型*。
			* 约定式提交预设的*类型(type)* 不总是能保证满足符合语义化版本的使用约定。
			* 没有明确约定如何指定的其它类型的规则时，出现在标题中的类型并不提升可读性。
	* 段落内部不应当具有冗余的硬回车。
* 文本文件：
	* 可执行的文本形式的脚本源文件应当具有*可执行*的权限，可选使用 [shebang](https://zh.wikipedia.org/zh-cn/Shebang)。
	* 其它文本形式的脚本代码不具有*可执行*权限，不使用 shebang 。
		* **注释** 当前 Unilang 源程序不支持 shebang ，需脚本解释器包装调用。这可能在以后改变。
	* 已带有 shebang 或者其它文件头的文本文件或仅以二进制形式使用的文本文件不应当带有 BOM 。
	* 如有可能，其它情形的文本文件默认使用 UTF-8 + BOM 编码和 CR+LF 行尾。
		* **注释** 这保持足够的文本*文件*元数据、确保跨编辑器的可移植性，同时能在默认配置下检查出误用。
	* 除了许可证和外部文档外，如有可能，行尾不带有冗余的空白符，文件结尾保持两个空行。
	* 除字面地引用的文本，不使用超过一个空白符表示一个缩进，除非语言中有其它首选形式。
		* **原理** 一个缩进内部默认不应能插入字符。
		* **注释** 一般地，水平文本除东亚语言悬挂缩进中使用的 2 个 U+3000 构成的序列外，缩进仅应使用水平制表符。
	* 对齐应使用 ` ` 。
		* **原理** 相对其它字符，这便于预测在通常的等宽字体设置下的视觉效果且具有较好的兼容性。
* 源代码：
	* 作为程序源代码的文本文件每行应不超过 80 字符，但 URL 等有必要确保内容在一整行的情形例外。
	* C++ ：
		* 语言标准：
			* 使用 ISO C++11 ，排除不和之后语言标准版本中不兼容的特性。
				* **注释** 例如，ISO C++20 修改了关于 *lambda-expression* 默认捕获和 `u8` 字面量的规则，使一些代码含义可能不同而不应当在此被使用。
			* 代码应兼容 G++/Clang++ `-std=c++11 -pedantic-errors` 。
		* 词法风格：
			* **注释** 文本文件的缩进和对齐规则在此被继承。
			* `{` 和 `}` 应单独占一行，但构成 *braced-init-list* 或 *lambda-expression* 的最外层块时除外。
				* **原理** 为了兼顾视觉上容易索引和紧凑。
			* 标识符前缀 `INC_Unilang_` 保留给头文件守卫宏。
			* 除宏名，公开的 API 的标识符应当使用 `PascalCase` 。
		* 名称查找：
			* 不应在接口设计没有明确要求的情形使用 `namespace` 指令（仅允许在实现内部使用 `using namespace` ）。
			* 应当注意 ADL 。仅在不引起歧义时使用非限定名称。
		* 类型名称：
			* 不应当使用 `typedef` 。
				* **注释** 使用 `using` 代替。
			* 函数声明中的函数类型名称中的第一个标志符后应换行。
	* Markdown ：代码应当符合 [GitHub Flavor Markdown](https://github.github.com/gfm/) 。
	* Doxygen（当前未使用）：
		* 在命令中，应当使用 `\` 而不是 `@` 。
		* 如有可能，不应当连用的 `*` 和 `/` 。
			* **注释** 例如，使用 `//!` 而不是 `///` 。
* 本地化：
	* 当前本项目不支持文档以外的本地化资源。
	* 本项目中的程序不和系统区域设置交互（另见[已知问题](#已知问题)），用户界面消息视同区域 `en-US` 。
	* **注释** 以上限制预期将在以后移除。
	* 多语言超链接可能被添加到文档的翻译版本顶端。
		* 若链接存在，则所有翻译版本都应当具有相应的链接，除了当前语言的翻译版本可能不是链接且具有不同的显示样式。
		* **原理** 因为当前没有普遍接受的、开箱即用的原生语言切换支持，文档可能需要维护显式的链接以切换不同语言的显示。
		* **NOTE** 另见相关讨论：[[1]](https://github.com/github/markup/issues/899)、[[2]](https://github.com/github/cmark-gfm/issues/243)和[[3]](https://talk.commonmark.org/t/adding-lang-lang-syntax/3998)。
	* 本仓库中的文档的翻译版本可如同代码一样被添加。翻译版本的文件名应当符合以下规则：
		* 每个文件应具有一种通过语言代码指示的明确的语言，或默认为 `en-US` 。
		* 若存在语言代码，它应当和前缀的 `.` 一并出现在文件名中的在扩展名（及前缀 `.` 前，若存在）前一次。
		* 语言代码应当由 [ISO 639-1](https://en.wikipedia.org/wiki/ISO_639-1) 代码、`-` 和 [ISO 3166-1](https://en.wikipedia.org/wiki/ISO_3166-1) 代码的序列构成。
		* 在语言代码中应当使用标准建议的大小写，即 ISO 639-1 的小写和 ISO 3166-1 的大写。
		* **注释** 语言代码的形式符合 [IETF 语言标签](https://en.wikipedia.org/wiki/IETF_language_tag)中的建议，具体地，即 [RFC 5646](https://www.rfc-editor.org/rfc/rfc5646.html) 的“语言”和“区域”语法元素。
		* **注释** `en-US` 版本的文件名省略 `.en-US` 。例如，`README.md` 是 `en-US` 区域的翻译版本的文件名。其对应 `zh-CN` 版本的文件名是 `README.zh-CN.md` 。

# 许可证

　　Unilang 在 [BSD-2-Clause-Patent](https://spdx.org/licenses/BSD-2-Clause-Patent.html) 下发布。

