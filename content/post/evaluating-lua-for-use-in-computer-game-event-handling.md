---
title: "Evaluating Lua for Use in Computer Game Event Handling 评估Lua在计算机游戏事件处理中的应用"
date: 2024-02-11T22:00:36+08:00
draft: false
tags: ["thesis"]
math: false
---

*Oskar Forsslund*

*Stockholm, Sweden 2013*

*DD221X, Master's Thesis in Computer Science (30 ECTS credits)*

*Degree Program in Computer Science and Engineering 300 credits*

*Master Programme in Computer Science 120 credits*

*Royal Institute of Technology year 2013*

*Supervisor at CSC was Douglas Wikström*

*Examiner was Jens Lagergren*

*TRITA-CSC-E 2013:019*

*ISRN-KTH/CSC/E--13/019--SE*

*ISSN-1653-5715*

*Royal Institute of Technology*

*School of Computer Science and Communication*

*KTH Computer Science and Communication*

*SE-100 44 Stockholm, Sweden*

*URL:* [*www.kth.se/csc*](www.kth.se/csc)

## Abstract 摘要

This thesis rigorously investigates the differences in performance and development impact between the use of parsed scripts and embedded Lua scripts within the domain of in-game event evaluation in computer games. The focus of this study is an analytical comparison of the performance differences between these two methodologies, with a secondary focus on their respective impact on the development process.

本文深入探讨了在计算机游戏中，特别是在游戏事件评估领域，解析脚本与嵌入Lua脚本在性能表现及对开发过程影响方面的差异。研究着重对比了这两种技术手段在性能上的差别，并考察了它们对游戏开发流程的影响。

The scope of this research is limited to an investigation within the context of "Europa Universalis III" (EU3), a game developed by Paradox Development Studio (Paradox). The methodology involved the construction of a framework to facilitate the evaluation of in-game events via Lua scripts within EU3. The performance metrics of events executed through the Lua framework were then compared to those executed through the game's original event evaluation system.
Lua, an embedded scripting language used in video games since the 1990s, is known for its minimal memory footprint and accelerated execution speeds. Conversely, the traditional event evaluation mechanism in EU3 uses parsed script files executed as C++ code. The comparative analysis highlights the inherent advantages and limitations of each approach.

研究范围专门聚焦于“欧陆风云III”（EU3）这一Paradox Development Studio（Paradox）开发的游戏。通过建立一个框架，本研究旨在便捷地使用Lua脚本对EU3中的游戏事件进行评估。随后，将通过此Lua框架执行的事件的性能指标与游戏原生事件评估系统执行的事件进行了对比分析。Lua作为一种自九十年代起便广泛应用于视频游戏的嵌入式脚本语言，其最大的特点是占用内存少，执行速度快。而EU3的传统事件评估机制，则是通过解析作为C++代码执行的脚本文件。通过比较分析，本研究突显了每种方法的优势及局限性。

The use of an embedded scripting language such as Lua offers significant advantages, primarily in terms of increased flexibility and streamlined integration of new functionality. Conversely, parsed scripts exhibit superior performance metrics, underscoring a significant advantage in computational efficiency.

Lua等嵌入式脚本语言的使用带来了明显的好处，尤其是在提升灵活性和新功能集成流畅性方面。而解析脚本则在性能表现上展现了其优越性，特别是在计算效率方面展示了不可忽视的优势。

## Preface 前言

### Target Audience 目标读者

This thesis is primarily aimed at developers who are considering integrating Lua into C/C++ applications. It also has a broader appeal to those considering incorporating any scripting language into their software products. In addition, individuals with a keen interest in the performance aspects of Lua, despite the specific focus on its application within "Europa Universalis III" (EU3), may find this study of particular relevance.

本论文主旨在于向那些正考虑在C/C++应用程序中集成Lua的开发者们提供指导。同时，它对于想要在他们的软件产品中融入任何脚本语言的人士也同样具有吸引力。此外，对Lua性能方面感兴趣的读者，即便本研究主要聚焦于其在“欧陆风云III”（EU3）中的应用，也会发现这项研究格外贴切。

### Acknowledgements 致谢

Thanks to Jimmy Rönn for his indispensable guidance in navigating the complexities of EU3's event system and for being an exceptional sounding board.

在此，我特别感谢Jimmy Rönn对于我深入理解EU3事件系统复杂性提供的关键指导和卓越支持。

Thanks also to Mike Pall and the entire Lua mailing list community for their invaluable help and patience in dealing with numerous requests.

同样，我要感谢Mike Pall以及Lua邮件列表社区的全体成员，他们在处理无数请求时展现出的宝贵帮助与无尽耐心，对我而言至关重要。

Finally, I would like to express my sincere thanks to the staff at the Paradox office for their hospitality over the course of six months and for providing a pleasant and supportive working environment.

最后，我衷心感谢Paradox办公室的全体同仁。在过去的六个月里，他们的热情好客以及创造的友好支持性工作环境，为我的研究工作提供了极大的便利。

## Chapter 1 Background 背景

### 1.1 Problem 问题

#### 1.1.1 In-game Events 游戏内事件

##### An Overview of the Structure of Computer Games 计算机游戏结构概述

The term computer game encompasses a wide range of interactive digital programs, each of which is designed to present users with challenges and goals. Although perceptions of what constitutes a computer game may vary widely, a unifying characterization emerges when they are viewed as interactive programs that engage users through various forms of challenges, often toward the achievement of defined goals. This definition encompasses a wide range of game types, from rudimentary text-based interfaces that solicit direct responses to questions to expansive, graphically rich environments where players engage in complex adventures with a global community.

计算机游戏这个概念覆盖了广泛的互动数字程序，每一款游戏都旨在向玩家提出挑战并设定目标。虽然人们对计算机游戏的定义见仁见智，但一个共识是，这些都是旨在通过各式各样的挑战来吸引用户参与，并通常目标明确的互动式程序。无论是简单的文本界面游戏，要求玩家直接回答问题，还是那些让玩家在一个图形丰富的环境中与全球社区一同经历复杂冒险的游戏，这个定义都涵盖了它们。

Regardless of their complexity or subject matter, computer games share essential structural elements. These elements are shown in Figure 1.1, which illustrates a typical computer game architecture. The diagram illustrates the flow of interaction, beginning with the player's engagement through the user interface (UI). This interface can range from simple text prompts, similar to a command console, to sophisticated graphical user interfaces (GUIs) with audio components. Player input is passed to the game engine, which in turn interprets the input to modify the game state according to predefined rules.

不管游戏的复杂度或主题如何，所有计算机游戏都拥有一些基础的结构要素。如图1.1所示，展现了典型的计算机游戏架构，该图说明了从玩家通过用户界面参与开始的交互流程。用户界面可以是简单的文本提示，类似命令行控制台，也可以是集成了音频组件的复杂图形用户界面。玩家的输入将被传递给游戏引擎，游戏引擎进而根据预设规则解读这些输入，以改变游戏状态。

The game state encapsulates the current conditions within the game environment. It contains data that can be as simple as the player's responses or, as seen in "Europa Universalis III" (EU3), comprehensive details on the economic, political, industrial, and military aspects of all the nations within the game's universe. Saving a game preserves this complex state of the game for future play.

游戏状态包含了游戏环境中的当前情况，其数据可能非常简单，如玩家的选择反馈，或者如在“欧陆风云III”（EU3）中所展示的那样，包括游戏内所有国家在经济、政治、工业及军事各方面的细节信息。保存游戏的功能就是为了保留这种复杂的游戏状态，以便未来继续游玩。

![1.1](/home/tategotoazarasi/Documents/论文/Evaluating Lua for Use
in Computer Game Event Handling/1.1.png)

***Figure 1.1:** Typical structure of a modern computer game*

Artificial Intelligence (AI) also plays a vital role in game design. Although AI is typically associated with non-player characters, it essentially controls all aspects of the game that the player does not directly control. The AI operates by analyzing the current state of the game and applying a set of rules to determine its actions. These decisions are based on matching game conditions with predefined behaviors, with mechanisms in place to prioritize actions when multiple options are viable. This prioritization ensures that the AI chooses the most appropriate action from the available alternatives.

人工智能（AI）在游戏设计领域中占据着极其重要的地位。虽然AI通常被看作是与非玩家角色（NPC）紧密相关的技术，但实际上，它控制着游戏中玩家没有直接操作的所有内容。AI的运作机制是通过分析游戏的当前状态，并根据一系列规则来制定行动策略。这些决策是基于将游戏的实际情况与预设的行为模式进行匹配，而且在存在多个可行选项时，AI还会通过一定的机制来确定行动的优先级。这一优先级的确定，确保AI能从所有可能的选择中，选取最为恰当的行动方案。

Resulting changes in game state caused by AI decisions or player actions are communicated back to the player through the UI, completing the cycle of interaction and state evolution within the game.

AI决策或玩家动作所引发的游戏状态变更，将通过用户界面反馈给玩家，从而完成了游戏中交互与状态发展的完整循环。

##### In-game Events 游戏内事件

In-game events, as the name implies, are occurrences within the game environment that are activated by the game's artificial intelligence (AI) and result in changes to the state of the game. In Europa Universalis III (EU3), these events are perceived as seemingly random occurrences that have a significant impact on gameplay.

正如其名，游戏内事件是指在游戏环境中发生的、由游戏的人工智能（AI）触发的事件，这些事件会引起游戏状态的变化。在《欧陆风云III》（EU3）中，这类事件似乎随机出现，对游戏的玩法产生了深远的影响。

This study focuses on a critical component of in-game events known as the trigger mechanism. While several elements make up an event, the trigger is of primary interest for this analysis due to its intensive computational requirements during the event evaluation process. Other components, while essential to the execution of in-game events, do not play the central role in event evaluation that the trigger does.

本研究重点关注游戏内事件中一个关键的组成部分——触发机制。尽管一个事件包含多个要素，但由于在事件评估过程中对计算要求较高，触发机制成为了本分析的焦点。尽管其他组成部分对于游戏内事件的执行非常关键，但它们在事件评估中的作用并不如触发机制那样核心。

The trigger mechanism acts as a logical expression that is evaluated against the current game state to determine the feasibility of an event occurring. In the traditional system, where in-game events are derived from parsed script files, the trigger is constructed as a hierarchical logical structure similar to a tree. In this architecture, each leaf represents a state request, and each node represents a logical operator. The trigger evaluation process involves a recursive traversal of this tree, starting at the root node and aggregating the logical results.

触发机制充当一个根据当前游戏状态评估事件发生可能性的逻辑表达式。在那些游戏内事件源于解析脚本文件的传统系统中，触发器被设计成一个层级逻辑结构，类似于一棵树。在这种架构下，每个叶节点代表一个状态请求，每个分支节点代表一个逻辑运算符。触发器的评估过程包括从根节点开始对这棵树进行递归遍历，并汇总逻辑判断结果。

Trigger evaluation, and by extension, the entire event evaluation process, is performed in C++, a language known for its performance efficiency. The need for speed in this context is underscored by the large number of event evaluations that are performed during a game session, which must not adversely affect system performance in order to maintain an enjoyable game experience. However, the exclusive reliance on C++ for EU3's event-handling system introduces inevitable tradeoffs that will be explored in the following sections. For this performance-oriented study, a comparative analysis was conducted between the trigger evaluation times using Lua and those using EU3's existing C++-based event evaluation system.

触发器的评估，乃至整个事件评估过程，都是用C++来执行的，C++是一种以其执行效率而闻名的编程语言。在这个上下文中对速度的需求体现在游戏会话期间需要执行的大量事件评估必须不对系统性能产生负面影响，以保证玩家有一个愉快的游戏体验。然而，完全依赖C++进行EU3事件处理系统的设计引入了一些不可避免的权衡，这将在接下来的部分中进行探讨。为了这项以性能为中心的研究，我们对使用Lua的触发器评估时间与EU3现有的基于C++的事件评估系统进行了比较分析。

#### 1.1.2 Embedded Scripts vs. Parsed 嵌入式脚本与解析脚本

The juxtaposition of embedding a scripting language in a program versus using parsed script files to dictate program behavior can be distilled into a discourse of usability versus speed.

将脚本语言嵌入程序与使用解析脚本文件来指导程序行为的对比，实质上是围绕可用性与速度的辩论。

Embedding a scripting language provides the developer with a comprehensive programming toolkit that significantly reduces the need for additional coding compared to developing a custom parser. While the onus remains on the developer to delineate much of the terminal functionality, the adoption of an embedded scripting language can ease the development burden due to its inherently comprehensive nature. For example, integrating various integer comparison operations through parsed scripts requires either the implementation of multiple comparison functions or an increase in parser complexity to accommodate these comparisons. Conversely, an embedded scripting language inherently provides a set of basic functionalities, relegating the need for custom development to more sophisticated functionalities- a necessity regardless of the scripting methodology employed.

嵌入脚本语言赋予开发者一个全面的编程工具集，相较于开发一个自定义的解析器，这大大降低了额外编码的需求。尽管开发者仍需定义许多终端功能，但采用嵌入式脚本语言能够因其固有的全面性而减轻开发压力。例如，若通过解析脚本来集成各种整数比较操作，则需要实现多个比较函数，或者增加解析器的复杂度以支持这些比较。相比之下，嵌入式脚本语言天生就包含了一套基础功能，仅将对自定义开发的需求留给了更为复杂的功能——这是使用任何脚本方法时的必然需求。

However, the use of embedded scripting languages has its drawbacks, primarily in terms of performance. The execution of scripted functionality requires a transition from the main program to the scripts, which has two main drawbacks:

然而，使用嵌入式脚本语言也存在缺点，主要体现在性能方面。脚本功能的执行需要在主程序和脚本之间转换，这主要存在两个缺点：

- Alternating between execution of the main program and scripts introduces runtime overhead that is not present in parsed scripts. In addition, the transfer of information between these components introduces additional overhead, in contrast to a model where information is managed exclusively within the main program.

  主程序和脚本执行之间的切换引入了运行时开销，而这在解析脚本中是不存在的。此外，组件之间的信息传递也带来了额外开销，这与信息仅在主程序内部管理的模式形成了对比。

- Scripting languages typically have lower execution speeds than the languages used for the main program.

  脚本语言的执行速度通常低于主程序使用的语言。

Conversely, the parsing process associated with script files also introduces execution overhead. This overhead can be mitigated by using an embedded script that approximates the execution speed of the main program and is complemented by a fast compilation process. Nonetheless, the parsing overhead, which is primarily a one-time occurrence, warrants consideration, especially when the entire runtime of the program, rather than just the post-startup execution, is critical.

另一方面，与脚本文件相关的解析过程同样带来了执行开销。通过使用一个接近主程序执行速度的嵌入式脚本，并且辅以快速的编译过程，可以缓解这种开销。然而，解析开销主要是一次性发生的，特别是当整个程序的运行时间——而不仅仅是启动后的执行——变得至关重要时，这种开销值得被考虑。

### 1.2 Lua

This section provides a brief introduction to the Lua scripting language. Lua has been the choice of many video game developers due to its efficiency, power, and compact size. The purpose of this introduction is to provide a basic understanding of Lua in order to facilitate a better understanding of the discourse that follows. For those interested in a more in-depth exploration, there are many resources available, including the official Lua website.

本节将对Lua脚本语言进行简要介绍。Lua因其高效、功能强大且体积小巧，成为众多视频游戏开发者的首选。本介绍旨在提供对Lua的基础了解，帮助读者更好地理解后续的讨论。对于希望深入了解的读者，包括官方Lua网站在内的许多资源都可供参考。

#### 1.2.1 Overview Lua Lua概览

Conceived in the early 1990s by researchers at the Pontifical Catholic University of Rio de Janeiro, Brazil, Lua was designed to integrate seamlessly with C programs. Lua's complete source code is written in ANSI standard C and consists of approximately 17,000 lines. It features dynamic typing, automatic garbage collection, and the treatment of functions as first-class citizens. A distinctive feature of Lua is its extensive use of tables as the primary data structure, facilitating the nesting of tables within tables. The global namespace itself is implemented as a table, making variable assignments equivalent to modifying fields within that table.

Lua由巴西里约热内卢天主教大学的研究人员在1990年代早期创建，设计初衷是为了能够与C程序无缝集成。Lua的全部源代码采用ANSI标准C编写，代码总量约为17,000行。其特点包括动态类型系统、自动垃圾回收机制，以及将函数作为一等公民对待。Lua的一个独特之处在于其广泛使用表作为基础数据结构，支持表中嵌套表。全局命名空间本身也是作为一个表实现的，因此变量的赋值操作实质上是在修改该表中的字段。

Lua supports a variety of data types, including boolean, number, string, user data, function, thread, table, and nil. Unassigned variables are defaulted to nil. Indexing in Lua starts at 1. Initially, Lua did not provide native support for multi-threading, meaning that threads in Lua do not execute concurrently.

Lua支持包括布尔值、数字、字符串、用户数据、函数、线程、表及nil在内的多种数据类型。未初始化的变量默认为nil。Lua中的索引起始于1。最初，Lua并未原生支持多线程，这意味着Lua中的线程不能并行执行。

One notable aspect of Lua that proved invaluable in this research is its extensive string manipulation capabilities and support for regular expressions.

Lua在本研究中显示出的一个值得注意的方面是其广泛的字符串处理功能和对正则表达式的支持。

##### Lua Code Example Lua代码示例

To illustrate the syntax and capabilities of Lua, a sample Lua code is presented below. This particular script was designed to count the frequency of function usage within the EU3 event files. The script reads the `events/file` list file line by line and counts the occurrences of function names mentioned within the files specified on each line.

以下是一个Lua代码示例，用于展示Lua的语法和功能。这个特定脚本的设计目的是计算《欧陆风云III》事件文件中函数使用的频率。脚本会逐行读取`events/file`列表文件，并统计每一行所指定文件中提及的函数名称的出现次数。

```lua
funcs = {}

function list_funcs(str)
    for match in str:gmatch("%a[%a_]*%b()%s+>?=%s+%d+") do
        if not funcs[match] then
            funcs[match] = 1
        else
            funcs[match] = 1 + funcs[match]
        end
    end

    for match in str:gmatch("(%a[%a_]*%b())%s+[^>=]") do
        if not funcs[match] then
            funcs[match] = 1
        else
            funcs[match] = 1 + funcs[match]
        end
    end
end

for line in io.lines("events/filelist") do
    if line then
        local file = io.open("events/" .. line, "r")
        local content = file:read("*a")
        for match in content:gmatch("trigger%s+=%s+%b[]") do
            list_funcs(match)
        end
        for match in content:gmatch("trigger_prefix%s+=%s+%b[]") do
            list_funcs(match)
        end
        file:close()
    end

 file = io.open("funclist_unsort", "w")
 for k, v in pairs(funcs) do
     file:write(k .. "\t" .. v .. "\n")
 end
 file:close()
end
```

#### 1.2.2 Integration of Lua with C Lua与C的集成

As described in the previous discourse, Lua is designed to integrate seamlessly with C applications. This integration is achieved through the `lua_State` object, coupled with a comprehensive C application programming interface (API). The API provides functions for manipulating the `lua_State` and facilitates the execution of Lua scripts within it. The `lua_State` holds data from Lua scripts and manages the Lua stack, which is crucial for interactions between Lua and C.

如前所述，Lua被设计为能够与C应用程序无缝集成。这种集成是通过`lua_State`对象以及一个全面的C应用程序接口（API）来实现的。API提供了操作`lua_State`的函数，促进了Lua脚本在其内的执行。`lua_State`存储来自Lua脚本的数据，并管理Lua栈，这对于Lua与C之间的交互至关重要。

Calling a Lua function from C requires placing the desired function on the Lua stack, which can be achieved through methods such as `lua_getglobal` The arguments are then pushed onto the stack in the order in which they appear, culminating in the execution of `lua_call`. This function completes the call and removes the function and its arguments from the stack. It is then up to the Lua function to append any results to the stack in the order they were generated. Manipulating the stack, mainly through push and pop operations, also allows indexing from both ends. The index of the top element is synonymous with the total number of elements on the stack, following Lua's indexing convention of 1. Negative indices are used to reference the stack inversely, with `-1` indicating the top element; these are called pseudo-indexes.

从C中调用Lua函数，需要先将目标函数放置到Lua栈上，这可以通过`lua_getglobal`等方法完成。随后，参数按照它们出现的顺序被推入栈中，之后执行`lua_call`函数完成调用，并从栈中移除函数及其参数。接下来，Lua函数将按照生成的顺序将任何结果追加到栈上。通过推入和弹出操作操纵栈，同时允许从两端进行索引。栈顶元素的索引与栈上元素总数相同，遵循Lua的从1开始的索引约定。使用负索引可以反向引用栈，其中`-1`表示栈顶元素；这些被称为伪索引。

#### 1.2.3 LuaJIT and Its Utilization LuaJIT及其利用

For this project, LuaJIT was used to improve performance. LuaJIT, as described on its project homepage, serves as an alternative Lua compiler that improves performance at the expense of reduced portability. LuaJIT's portability limitations have not hindered this project, as it is compatible with most major platforms. The performance improvement achieved by LuaJIT is substantial, as will be discussed in Section 3.3.

在此项目中，采用LuaJIT来提升性能。LuaJIT，如其项目主页所述，是一个提高性能的Lua编译器替代品，但代价是降低了可移植性。LuaJIT的可移植性限制并未影响本项目，因为它与大多数主流平台兼容。LuaJIT带来的性能提升是显著的，这将在第3.3节中进一步讨论。

#### 1.2.4 Lua's Historical Embedment in Video Game Development Lua在视频游戏开发中的历史应用

The use of Lua in video game development is a phenomenon that has been around for a while. Its potential was quickly recognized by developers, leading to its integration into numerous widely played games, with "World of Warcraft" being a prime example. An examination of the extensive documentation of Lua's use in games available on the Lua user wiki reveals its frequent use for tasks such as user interface (UI) management, in-game event handling, game logic orchestration, and dynamic in-game parameter adjustment. This aligns closely with efforts at Paradox, particularly in the area of in-game event management.

Lua在视频游戏开发中的应用已经有一段时间了。开发者很快认识到了其潜力，导致Lua被集成进了许多广受欢迎的游戏中，“魔兽世界”便是一个突出的例子。通过检查Lua用户wiki上关于Lua在游戏中应用的丰富文档，可以发现Lua经常被用于用户界面（UI）管理、游戏内事件处理、游戏逻辑编排和动态游戏参数调整等任务。这与Paradox在游戏内事件管理领域的努力紧密相关。

## Chapter 2 Methodology 方法论

### 2.1 Workflow Implementation 工作流程实施

The first phase of this research involved the development of a preliminary version of the "Europa Universalis III" (EU3) event handling system using Lua, which was subsequently integrated into the EU3 game framework. This basic iteration aimed to verify the feasibility of using Lua for event evaluation purposes. The implementation phase involved the introduction of an alternative event evaluation mechanism within EU3 using Lua. This process required the creation of a set of functions within Lua to access the necessary in-game data, as well as the conversion of existing event scripts to be compatible with the Lua infrastructure.

本研究的首个阶段包括使用Lua开发《欧陆风云III》（EU3）事件处理系统的初版，并将其集成进EU3游戏框架中。这一初步迭代的目的是为了验证利用Lua进行事件评估的可能性。实施阶段包括使用Lua在EU3中引入了一种新的事件评估机制。这一过程需要创建一组Lua函数来访问游戏中所需的数据，同时将现有的事件脚本转换以适应Lua的基础架构。

Subsequent iterations focused on improving the efficiency of the system by identifying and refining areas of the code that were not performing optimally. This refinement process included both the evaluation algorithms and the structural organization of the events. A detailed examination of these scoring techniques and the significant improvements made is described in Section 3.2.

后续迭代的重点是通过识别和改善代码中性能表现不佳的部分来提升系统效率。这一改进过程涵盖了评估算法和事件的结构安排。第3.2节将详细讨论这些评分技巧及其显著的改善。

### 2.2 Testing Framework 测试框架

#### 2.2.1 Objective of Testing 测试目的

Testing was conducted exclusively on a single computer system embedded within the continuous development cycle. The primary purpose of the testing framework was to identify performance bottlenecks within the new event evaluation framework and quantify the effectiveness of the developed solution.

测试完全在单一计算机系统上进行，该系统嵌入于持续的开发周期中。测试框架的主要目的是识别新事件评估框架中的性能瓶颈，并对开发的解决方案的有效性进行量化。

The decision to limit testing to a single system was based on the goal of obtaining a preliminary comparison of the relative execution speeds of the two systems under consideration. The significant performance limitations of the developed framework would be consistent across different hardware architectures, facilitating the identification of significant inefficiencies.

选择仅在单一系统上进行测试的决策基于获得两个考虑中系统的相对执行速度初步比较的目标。开发框架的显著性能限制在不同硬件架构上应当是一致的，这有助于识别重大的效率问题。

A broader test environment would be advantageous for more granular analysis. However, for this project, the test configuration employed was deemed sufficient to provide the necessary insights.

对于更细致的分析，一个更广泛的测试环境将是有益的。然而，对于本项目而言，采用的测试配置被认为足够提供必要的洞察力。

#### 2.2.2 Testing Methodology 测试方法

The evaluation of the developed framework was performed within the game environment using an existing debugging and testing console integrated into the EU3 source code. This console was enhanced with event analysis functionality to allow a comparative analysis of execution speeds between the Lua-based system and the original implementation.

使用EU3源代码中集成的现有调试和测试控制台，在游戏环境内对开发的框架进行了评估。这个控制台被增强了事件分析功能，允许对基于Lua的系统与原始实现的执行速度进行比较分析。

Execution timings were recorded using high-resolution timing mechanisms available in the Windows operating environment. The event evaluation processes specific to each system were encapsulated within a timing framework that allowed the identification of events that exhibited performance discrepancies when evaluated using Lua.

使用Windows操作环境中可用的高分辨率计时机制记录执行时间。每个系统特有的事件评估过程被封装在一个计时框架中，使得能够识别在使用Lua评估时显示性能差异的事件。

After identifying events with performance discrepancies, further analysis was performed using the GlowCode profiling tool. This tool facilitated the precise identification of performance bottlenecks within the event evaluation process, providing insight into potential optimizations.

在识别出性能差异的事件后，使用GlowCode分析工具进行了更深入的分析。该工具帮助精确地识别事件评估过程中的性能瓶颈，提供了对潜在优化的见解。

GlowCode also provided a breakdown of the time allocation between Lua execution and C++ processing, providing a nuanced understanding of the performance dynamics between the two programming environments.

GlowCode还详细展示了Lua执行与C++处理之间的时间分配，提供了两种编程环境间性能动态的深入理解。

## Chapter 3 Results 结果

### 3.1 Implementation and Impact of Lua in Event Handling 在事件处理中实施Lua及其影响

##### Rationale for Integrating Lua 集成Lua的原因

Paradox's decision to explore integrating Lua into its development pipeline stems from a strategic evaluation to improve its event implementation process. Events are a central element of Paradox's game design, serving not only as a critical driver of gameplay but also significantly influencing the narrative and dynamics of the game environment. As such, an efficient and flexible event-handling system provides significant benefits to both the development team and broader business objectives.

Paradox探索将Lua集成进其开发流程的决定，源自于一项旨在改善事件实现过程的战略评估。事件是Paradox游戏设计中的核心元素，不仅是游戏玩法的关键推动力，也对游戏环境的叙事和动态产生显著影响。因此，一个高效且灵活的事件处理系统对开发团队及其更广泛的商业目标来说，带来了显著的益处。

Incorporating an embedded scripting language such as Lua offers two main advantages over traditional parsed scripting methods:

引入像Lua这样的嵌入式脚本语言，相较于传统的解析脚本方法，提供了两大主要优势：

- **Dynamic script modification**: Embedded scripting languages allow scripts to be modified and adjusted in real-time without having to reload an entire game or script. This feature dramatically streamlines the scripting process, allowing for rapid iteration and debugging of scripts, accelerating the development cycle.

  **动态脚本修改**：嵌入式脚本语言允许在不需重载整个游戏或脚本的情况下，对脚本进行实时修改和调整。这一特性极大地简化了脚本编写过程，使脚本的快速迭代和调试成为可能，从而加快了开发周期。

- **Advanced scripting capabilities**: Unlike parsed scripts, which serve primarily as directives to the game engine, embedded scripts operate as standalone mini-programs. This autonomy allows scripters to implement new functionality directly within the scripts themselves, fostering innovation and allowing new ideas to be immediately integrated into the game environment without waiting for engine updates. This capability is invaluable for prototyping and experimenting with new game mechanics.

  **高级脚本功能**：不同于主要作为游戏引擎指令的解析脚本，嵌入式脚本能作为独立的小型程序运行。这种自主性使脚本编写者能够直接在脚本中实现新功能，促进了创新，并使新思想能够立即被集成到游戏环境中，无需等待引擎的更新。这对于原型制作和测试新游戏机制至关重要。

However, the most important consideration when introducing a new event-handling system is the need to minimize any negative impact on system performance.

然而，在引入新的事件处理系统时，最重要的考量是需要尽量减少对系统性能的负面影响。

##### Event File Structure 事件文件结构

In the revised event handling system, event files are constructed as Lua scripts. Upon game initialization, these scripts populate the  `lua_State` with event-related data. Then, triggers for each event are stored within the `lua_State` as functions, each encapsulating a logical expression that evaluates the game state against the event's trigger conditions.

在修订后的事件处理系统中，事件文件以Lua脚本的形式构建。在游戏初始化时，这些脚本将事件相关的数据填充进`lua_State`。接着，每个事件的触发器以函数的形式存储在`lua_State`中，每个函数都封装了一个逻辑表达式，用于根据事件的触发条件评估游戏状态。

##### Methodology for Event Evaluation 事件评估方法

Evaluating an event's trigger within this framework involves calling the appropriate trigger function from C++ and providing relevant contextual information (such as references to specific countries or provinces). This approach simplifies the trigger evaluation process and allows for seamless integration between the game's C++ core and the Lua scripting layer.

在该框架中评估事件触发器涉及从C++调用相应的触发函数，并提供相关的上下文信息（例如，特定国家或省份的引用）。这种方式简化了触发器评估过程，并实现了游戏的C++核心与Lua脚本层之间的无缝集成。

### 3.2 Improvements 改进

#### 3.2.1 Efficiency Challenges with String Management 字符串管理的效率挑战

A critical oversight in the initial use of Lua for event evaluation was the inefficient management of strings. The original implementation suffered significant performance penalties due to the frequent creation and handling of strings, which are used extensively within the Europa Universalis III (EU3) framework for various identifiers and flags in the game's databases.

在最初使用Lua进行事件评估时，一个关键问题是字符串的管理效率低下。原始实施由于频繁创建和处理字符串而遭受了重大性能损失，而这些字符串在《欧陆风云III》（EU3）框架中被广泛用作各种标识符和游戏数据库中的标志。

Due to Lua's foundation in C, interoperability between Lua and C++ requires the use of the Lua stack, a construct inherently designed within the C language domain. Consequently, C++ string objects cannot be directly manipulated on the Lua stack. To bridge Lua scripts with C++, strings must be dynamically allocated in C++ to accommodate the C-based string representations derived from Lua, which imposes a significant performance overhead.

由于Lua是基于C的，Lua与C++之间的互操作性需要通过Lua栈来实现，这是一个在C语言域内设计的结构。因此，不能直接在Lua栈上操作C++的字符串对象。为了将Lua脚本与C++桥接，必须在C++中动态分配字符串，以容纳从Lua得出的基于C的字符串表示，这导致了显著的性能开销。

#### Strategic Improvements in String Handling 字符串处理的战略改进

To address this inefficiency, an innovative approach was taken to store and manage strings on the C++ side, identified by integer mappings. This method allowed integers to be passed between Lua and C++ instead of strings, resulting in a significant improvement in execution speed. While this solution introduces an additional layer of lookup for each string, it significantly mitigates the overhead associated with string generation at runtime.

为了解决这个效率问题，采取了一种创新的方法，在C++端通过整数映射来存储和管理字符串。这种方法允许在Lua和C++之间传递整数而非字符串，从而显著提高了执行速度。虽然这个解决方案为每个字符串引入了一个额外的查找层次，但它显著降低了运行时生成字符串的开销。

This challenge highlights a notable advantage of parsed scripts over embedded scripting languages. Parsed scripts inherently preprocess and store all necessary strings during the parsing phase, eliminating the need for runtime string generation and thus improving performance during the event evaluation process exemplified in the EU3 game engine. Alternatively, as demonstrated in this study, aligning data formats between the embedded scripting language and the host language (C++) is a viable strategy to circumvent the identified performance bottleneck.

这一挑战凸显了解析脚本相较于嵌入式脚本语言的一大优势。解析脚本在解析阶段就预处理并存储了所有必要的字符串，避免了运行时生成字符串的需求，从而在EU3游戏引擎中示例的事件评估过程中提升了性能。另一方面，正如本研究所示，对嵌入式脚本语言和宿主语言（C++）之间的数据格式进行对齐，是规避已识别性能瓶颈的一个有效策略。

The resolution of the string management inefficiency represents a critical refinement within the Lua-based event handling framework, illustrating the nuanced considerations essential to optimizing performance in game development. This enhancement not only underscores the importance of data format compatibility between scripting and host languages but also highlights the potential for embedded scripting languages to streamline game development processes when effectively integrated.

解决字符串管理效率低下问题代表了基于Lua的事件处理框架内的一个重要改进，展示了优化游戏开发性能所需的微妙考虑。这一改进不仅强调了脚本语言与宿主语言之间数据格式兼容性的重要性，也突出了嵌入式脚本语言在有效集成时简化游戏开发过程的潜力。

#### 3.2.2 Mitigating Transition Overheads 缓解转换开销

A key challenge in integrating Lua with C++ was to mitigate the performance overhead associated with transitions between the two languages. Two primary strategies were explored to address this issue: batch processing and preloading of information.

将Lua与C++结合使用的一个主要挑战在于缓解两种语言之间转换的性能开销。为了解决这个问题，我们探索了两种主要策略：批处理和信息预加载。

##### Batch Processing Approach 批处理方法

The principle of batch processing is based on the premise that consolidating multiple calls into a single transition between Lua and C++ would yield performance benefits. This approach involves running a collection of C++ functions called Lua with a list of functions as arguments. These functions are then executed sequentially within C++, with their return values aggregated and then returned to Lua. This method effectively reduces the frequency of Lua-to-C++ transitions.

批处理的核心原理是基于这样一个前提：将多个调用合并成一个从Lua到C++的单次转换，能够带来性能上的益处。这种方法涉及运行一系列C++函数，这些函数从Lua调用并带有一个函数列表作为参数。这些函数接着在C++中依序执行，它们的返回值被集合起来，然后一并返回给Lua。这个方法有效地降低了从Lua到C++转换的频率。

However, this strategy is limited by its inability to take advantage of the lazy evaluation mechanisms inherent in logical expressions. Lazy evaluation allows expression evaluation to be terminated as soon as the result can be determined without the need to process all parameters.

然而，这种策略的局限性在于它无法利用逻辑表达式中固有的惰性评估机制。惰性评估允许一旦能够确定结果就停止表达式的评估，而无需处理所有参数。

Consequently, batch processing, by requiring the pre-fetching of all data, could result in the retrieval of unnecessary information, thereby degrading performance in scenarios where not all data is used.

因此，由于批处理要求预先获取所有数据，它可能会导致检索到不必要的信息，从而在不需要使用所有数据的情况下降低性能。

Ultimately, the decision was made not to use batch as a general solution because it tended to underperform in cases where only a subset of the pre-fetched data is needed. The effectiveness of batch still depends on the judicious selection of operations to batch, a task of considerable complexity.

最终，由于在只需要部分预获取数据的情况下性能表现不佳，我们决定不将批处理作为一般性解决方案采用。批处理的有效性依赖于对哪些操作进行批处理的明智选择，这是一个颇具挑战性的任务。

##### Pre-loading Data 预加载数据

*Preloading* is an anticipatory approach that aims to identify and cache data that is frequently accessed during event evaluation. By storing this data for reuse within a single evaluation, the methodology seeks to reduce the cumulative number of function calls. Although comprehensive implementation across all EU3 events was limited by the large size of the game, targeted application of preloading demonstrated potential performance improvements.

*预加载数据*是一种前瞻性策略，目的是识别并缓存事件评估期间频繁访问的数据。通过在单次评估中存储并重用这些数据，该策略旨在减少函数调用的累计次数。虽然对所有EU3事件的全面实施受到游戏规模的限制，但针对性的预加载应用展示了潜在的性能改善。

As with batch processing, the effectiveness of preloading depends on the careful selection of the data to be cached. Preloading data that is ultimately unused can negatively impact performance, underscoring the importance of strategic data selection.

与批处理一样，预加载的有效性依赖于被缓存数据的精心选择。预加载最终未被使用的数据可能会对性能产生负面影响，这强调了战略性数据选择的重要性。

A hypothetical alternative to preloading could be localized caching within the evaluation process. While not pursued in this study due to the complexity and time constraints of the project, localized caching could provide a sophisticated means to further optimize performance by minimizing redundant data retrieval.

预加载的一个假设性替代方案可能是在评估过程中进行局部缓存。尽管由于项目的复杂性和时间限制，本研究未采用这一方法，但局部缓存可以提供一种进一步优化性能的复杂手段，通过最小化冗余数据的检索。

#### 3.2.3 Database Localization for Enhanced Performance 数据库本地化以提高性能

Splitting program execution between two languages requires strategic decisions regarding the allocation of computational tasks and data storage. An exploratory approach was taken to investigate the potential benefits of migrating databases from C++ to Lua to mitigate the performance overhead associated with cross-language interactions. This involved creating a Lua-based snapshot of a database and comparing event evaluation times with those of the traditional C++ database. Preliminary results indicated that localizing the database within Lua could improve the efficiency of event evaluation. However,  the overall impact on overall system performance remained inconclusive due to the partial implementation of the database functionality within the Lua environment. Fully integrating the game's database system into  Lua would require extensive modifications to Europa Universalis III, a  task beyond the scope and time constraints of this study. Nevertheless,  this experiment underscores the importance of thoroughly evaluating the capabilities and limitations of the chosen scripting language in order to optimize performance effectively.

在两种语言之间分割程序执行需要关于计算任务分配和数据存储的战略决策。采取了探索性方法来研究将数据库从C++迁移到Lua，以减轻跨语言交互带来的性能开销的潜在好处。这涉及创建一个基于Lua的数据库快照，并与传统C++数据库的事件评估时间进行比较。初步结果表明，将数据库本地化到Lua可以提升事件评估的效率。然而，由于数据库功能在Lua环境中只部分实现，对整体系统性能的影响仍不明确。完全将游戏的数据库系统集成到Lua中将需要对《欧陆风云III》进行广泛修改，这超出了本研究的范围和时间限制。尽管如此，这次实验强调了为有效优化性能，彻底评估所选脚本语言的能力和限制的重要性。

### 3.3 Comparative Analysis: Lua vs. LuaJIT 比较分析：Lua与LuaJIT

The investigation of the performance impact of integrating LuaJIT involved a direct comparison with the standard Lua libraries. The game was compiled using the LuaJIT libraries without any changes to the Lua event scripts or the C++ source code. The performance improvement due to LuaJIT was quantitatively assessed through rigorous testing, as shown in Figures 3.1 and 3.2.

对集成LuaJIT的性能影响的研究涉及与标准Lua库的直接比较。游戏使用LuaJIT库进行编译，而无需更改Lua事件脚本或C++源代码。通过严格测试定量评估了LuaJIT带来的性能改进。

![3.1](/home/tategotoazarasi/Documents/论文/Evaluating Lua for Use
in Computer Game Event Handling/3.1.png)

***Figure 3.1:** Difference in average evaluation time of triggers evaluated in C++, Lua and Lua with LuaJIT*

**Figure 3.1** illustrates the variance in average trigger evaluation times across the different setups. The methodology for calculating the average evaluation time involved timing the evaluation of each event under different Lua configurations, normalizing these times against a C++ benchmark, and then averaging these normalized values across all events. This figure illustrates the relative efficiency of the different configurations, highlighting the performance difference between standard Lua and LuaJIT-enhanced executions.

**图3.1** 显示了在不同配置下平均触发评估时间的变化。计算平均评估时间的方法涉及在不同Lua配置下对每个事件的评估进行计时，将这些时间相对于C++的基准进行标准化，然后计算所有事件的这些标准化值的平均值。该图表展示了不同配置的相对效率，并强调了标准Lua与LuaJIT增强执行之间的性能差异。

**Figure 3.2** presents a focused analysis of the performance gains achieved by the LuaJIT implementation. The improvement metric is derived from the formula: `1-(evaluation time with LuaJIT/evaluation time with regular Lua)`. This calculation averaged over all evaluated events, demonstrates the significant performance benefits provided by LuaJIT.

**图3.2** 对LuaJIT实施带来的性能提升进行了详细分析。改进指标是基于以下公式得出的：`1-(LuaJIT评估时间/常规Lua评估时间)`。这一计算在所有评估的事件上取平均，证明了LuaJIT提供的显著性能优势。

![3.2](/home/tategotoazarasi/Documents/论文/Evaluating Lua for Use
in Computer Game Event Handling/3.2.png)

***Figure 3.2:** Improvement in evaluation times of events using LuaJIT instead of the regular Lua compiler*

These results illustrate the significant improvement in event evaluation efficiency that can be achieved with LuaJIT and underscore the potential of advanced scripting language compilers in optimizing game development processes.

这些结果展示了使用LuaJIT能够实现事件评估效率的显著提升，同时强调了先进脚本语言编译器在优化游戏开发流程中的巨大潜力。

### 3.4 Results 结果

#### 3.4.1 Performance Evaluation 性能评估

##### Decision Criteria for Lua Integration Lua集成的决策标准

The decision to use Lua in a project depends mainly on the specific application of the scripting language. It is crucial to recognize that Lua's strength lies in something other than outperforming C++ in evaluating logical conditions applied to data managed in C++. The empirical results from the final configuration of the event evaluation system showed that event processing times ranged from 1.5 to 15 times slower than those of the original C++ setup. This discrepancy was expected, given the inherent overhead associated with bridging Lua and C++.

在项目中采用Lua主要取决于脚本语言的具体应用场景。关键在于认识到Lua的优势并不是在于比C++更高效地评估适用于C++管理的数据的逻辑条件。最终配置的事件评估系统的实证结果表明，事件处理时间比原始C++配置慢1.5到15倍。考虑到桥接Lua与C++固有的开销，这种差异是可以预见的。

##### Analysis of Performance Discrepancies 性能差异分析

The use of GlowCode for performance analysis sheds light on the primary factors contributing to slower execution times within the Lua framework. A pattern emerged when examining events with similar performance metrics relative to their C++ counterparts. Events that closely matched the performance of the original setup were predominantly executed within the C++ domain. Conversely, events with significantly slower execution times had a higher proportion of their processing within Lua.

利用GlowCode进行性能分析揭示了Lua框架执行时间较长的主要原因。在检查与其C++对应项性能指标相近的事件时，发现了一种模式。与原始配置性能接近的事件主要在C++领域内执行。相反，执行时间显著较长的事件在Lua中的处理占比更高。

This distinction underscores a fundamental aspect of the event evaluation process, which consists of three primary stages:

这种区别凸显了事件评估过程的一个基本方面，该过程主要包括三个阶段：

1. **Initialization and invocation**: The process begins with preparing and invoking the event from C++, which involves setting up arguments on the stack. This phase is common to all events.

   **初始化和调用**：过程以从C++准备和调用事件开始，这涉及到在栈上设置参数。这个阶段对所有事件都是通用的。

2. **Evaluation**: The core of event evaluation involves retrieving in-game values and performing comparisons. Each data retrieval represents a call to C++, so performance is affected by the frequency and complexity of these calls.

   **评估**：事件评估的核心是检索游戏内的值并进行比较。每次数据检索都代表了一次对C++的调用，因此性能受到这些调用频率和复杂性的影响。

3. **Result processing:** Finally, the result of the event analysis is passed back to C++, completing the process.

   **结果处理**：最终，事件分析的结果被传回C++，完成整个过程。

In pseudocode, this would look like this:

用伪代码表示，可能看起来像这样：

```pseudocode
Preparation and Lua call // static overhead
for every line in event condition do:
 fetch value from C++ side  // done by calling C++ function
 (compare to value) //not always done
return answer to C++ side // static overhead
```

The integration of Lua into the event handling framework of "Europa Universalis III" (EU3) has revealed a fundamental insight into the performance dynamics between Lua and C++. Each event evaluation involves an inherent overhead, both overall and specifically associated with calls to C++. For an event's runtime to be predominantly occupied by C++ execution, the functions that retrieve in-game values must have significant processing times, rendering the overhead introduced by Lua for these events nearly inconsequential.

将Lua集成进《欧陆风云III》（EU3）的事件处理框架中，揭示了Lua与C++之间性能动态的基本见解。每次事件评估都涉及固有的开销，无论是总体上的还是特别是与C++调用相关的。对于一个事件的运行时间主要由C++执行占据的情况，检索游戏内值的函数必须具有相当长的处理时间，使得Lua为这些事件引入的开销几乎可以忽略不计。

This observation is consistent with the empirical data collected, which indicates that the C++ functions used for data retrieval within Lua events closely mirror those used in traditional C++ event evaluation. Events that have comparable execution times to their C++ counterparts are characterized by data retrieval functions whose execution time minimizes the relative overhead of Lua. Conversely, events that are inherently fast in the original C++ framework tend to experience pronounced slowdowns when migrated to Lua due to the lightweight nature of their operations and the disproportionate impact of Lua's overhead.

这一观察与收集的实证数据相符，这些数据表明，用于Lua事件中数据检索的C++函数与传统C++事件评估中使用的函数非常相似。与其C++对应项具有可比执行时间的事件特征在于，数据检索函数的执行时间最小化了Lua的相对开销。反之，在原始C++框架中固有地快速执行的事件在迁移到Lua时会因为它们操作的轻量级和Lua开销的不成比例影响而经历显著的减慢。

Figures 3.3 and 3.4, although not visually depicted here, support this trend by showing that events with longer execution times in C++ tend to exhibit less pronounced relative slowdowns in Lua, highlighting the correlation between the complexity of C++ operations and the efficiency of Lua integration.

尽管没有在此图形描述，但图3.3和3.4支持这一趋势，显示了在C++中执行时间较长的事件在Lua中表现出较少的相对减慢，强调了C++操作复杂性与Lua集成效率之间的相关性。

##### Enhancements in Development Usability 开发可用性的增强

Moving to an embedded scripting language such as Lua offers significant advantages in terms of development flexibility and efficiency. The Lua-based implementation of EU3's event handling system exemplifies the streamlined process of creating new events once the necessary game state querying functions have been established. Lua's rich functionality enables the design of complex events beyond the capabilities of the original system, potentially facilitating the exploration of innovative game mechanics and narratives.

转向像Lua这样的嵌入式脚本语言，在开发灵活性和效率方面提供了重大优势。EU3的基于Lua的事件处理系统实现示例了一旦建立了必要的游戏状态查询功能，新事件的创建过程将会简化。Lua的丰富功能使设计超出原始系统能力的复杂事件成为可能，从而可能促进创新游戏机制和叙事的探索。

This usability benefit underscores Lua's intrinsic value as a game development tool. It provides a robust and flexible framework for extending game functionality beyond the constraints of traditional parsed scripting systems. The ability to dynamically create and modify events within Lua improves the development workflow and provides a compelling argument for its integration despite the performance tradeoffs identified in certain scenarios.

这种可用性好处突显了Lua作为游戏开发工具的固有价值。它为超越传统解析脚本系统限制的游戏功能扩展提供了一个强大而灵活的框架。能够在Lua中动态创建和修改事件的能力改善了开发流程，并为其集成提供了有力的论据，尽管在某些情境下发现了性能上的权衡。![3.3](/home/tategotoazarasi/Documents/论文/Evaluating Lua for Use
in Computer Game Event Handling/3.3.png)

***Figure 3.3:** Correlation between C++ runtime and relative runtime for country events*

![3.4](/home/tategotoazarasi/Documents/论文/Evaluating Lua for Use
in Computer Game Event Handling/3.4.png)

***Figure 3.4:** Correlation between C++ runtime and relative runtime for province events*

### 3.5 Conclusions 结论

The investigation into the integration of Lua into the "Europa Universalis III" (EU3) event-handling framework leads to two main conclusions:

将Lua集成进《欧陆风云III》（EU3）事件处理框架的研究引出了两个主要结论：

1. **Performance Differential**: Using Lua embedded in a C++ event handling environment does not achieve the same level of performance efficiency as a pure C++-based approach. This result underscores the inherent tradeoffs between the execution speed of embedded scripting languages and native code.

   **性能差异**：将Lua嵌入C++事件处理环境中，并未能达到纯C++方法相同的性能效率。这一发现强调了嵌入式脚本语言执行速度与原生代码之间固有的权衡。

2. **Advantages of Embedded Scripting Languages**: Despite the performance differences, using an embedded scripting language such as Lua offers significant advantages in terms of development flexibility and ease of use. By design, embedded scripts provide a more dynamic and extensible framework for event handling than traditional parsed scripts.

   **嵌入式脚本语言的优势**：尽管存在性能上的差异，但使用像Lua这样的嵌入式脚本语言在提高开发灵活性和易用性方面具有显著优势。从设计角度看，嵌入式脚本为事件处理提供了一个比传统解析脚本更动态、更可扩展的框架。

This dichotomy between speed and usability highlights the key considerations that developers must weigh when choosing a scripting approach for game development. In scenarios where speed of execution is paramount, parsed scripts provide the optimal solution. Conversely, when development efficiency and flexibility are priorities, the advantages of embedded scripting languages such as Lua become apparent.

速度与可用性之间的这种二元性凸显了开发者在选择游戏开发脚本策略时必须考虑的关键因素。在执行速度至关重要的场景中，解析脚本提供了最优解决方案。相反，当开发效率和灵活性成为重点时，像Lua这样的嵌入式脚本语言的优点则变得尤为突出。

Although the Lua-based event handling system did not meet all performance expectations, its ease of use and extensibility are compelling advantages. Further optimizations and innovative approaches could narrow the performance gap between Lua and native C++ implementations.

虽然基于Lua的事件处理系统未能完全达到性能预期，但其易用性和扩展性的显著优点不容忽视。通过进一步的优化和创新方法，有可能缩小Lua与原生C++实现之间的性能差距。

This exploration of the comparative merits of parsed versus embedded scripting in the realm of game event handling contributes to a broader understanding of the strategic choices available to developers. The results of this study illuminate the tradeoffs involved in using scripting languages for game development and provide insight into how best to balance the competing demands of performance efficiency and development agility.

这一对游戏事件处理中解析脚本与嵌入式脚本相对优势的探讨，为开发者可用的战略选择提供了更广泛的理解。本研究的结果揭示了使用脚本语言进行游戏开发涉及的权衡，并就如何在性能效率与开发敏捷性之间找到最佳平衡提供了洞见。

### 3.6 Future Work 未来工作

The investigation into the integration of Lua for event handling within "Europa Universalis III" (EU3) provides a foundation for further research and optimization efforts. Despite the observed sixfold increase in event evaluation time - approaching but not reaching Paradox's goal of a maximum threefold increase - there are several avenues for improving performance and expanding the application of Lua within game development contexts.

对Lua在《欧陆风云III》（EU3）事件处理中的集成进行的研究为进一步的研究和优化工作奠定了基础。尽管观察到事件评估时间增加了六倍——接近但未达到Paradox设定的最多三倍增加的目标——但仍有几种方法可以提升性能并扩展Lua在游戏开发中的应用。

##### Optimizing Data Storage with Lua 使用Lua优化数据存储

One promising approach is to localize in-game data within Lua, eliminating the performance overhead associated with cross-language calls to C++. Preliminary tests suggest a potential for performance improvement using this approach. This strategy is consistent with Roberto Ierusalimschy's findings, which emphasize the delicate balance between minimizing overhead and managing the speed tradeoffs inherent in language selection for specific functionality. Further exploration of the optimal distribution of computational tasks between Lua and C++ could yield significant efficiency gains.

一种有前景的方法是在Lua中本地化游戏数据，以消除与C++跨语言调用相关的性能开销。初步测试表明，采用这种方法有潜力提升性能。这种策略与Roberto Ierusalimschy的发现相一致，他强调了在最小化开销和管理语言选择固有的速度权衡之间保持微妙平衡的重要性。进一步探索Lua和C++之间计算任务的最优分配可能会带来显著的效率提升。

##### Comprehensive Event Evaluation in Lua 在Lua中全面评估事件

An extension of the data localization strategy is to perform the entire event evaluation process in Lua, requiring the migration of all in-game information to Lua. While this approach may reduce function call overhead, it faces similar challenges in terms of overall execution speed in Lua compared to C++. The feasibility and implications of this comprehensive Lua evaluation model deserve thorough investigation.

数据本地化策略的一个延伸是在Lua中执行整个事件评估过程，这要求将所有游戏内信息迁移到Lua。虽然这种方法可能减少函数调用的开销，但在Lua与C++的总体执行速度方面面临着相似的挑战。这种全面的Lua评估模式的可行性和影响值得彻底研究。

##### Dual Scripting Systems for Development Flexibility 双脚本系统以提高开发灵活性

A hybrid model that utilizes both parsed and embedded scripts provides an innovative solution for balancing development agility with runtime efficiency. During the development phase, an embedded scripting environment could facilitate rapid prototyping and iteration, allowing for immediate adjustments and enhancements to game scripts. Subsequently, these scripts could be converted into a format compatible with a parser for use in production, taking advantage of the speed benefits of parsed scripts while retaining the development benefits of embedded scripting.

一种结合解析脚本和嵌入式脚本的混合模型提供了一种平衡开发敏捷性与运行时效率的创新解决方案。在开发阶段，嵌入式脚本环境可以促进快速的原型设计和迭代，允许对游戏脚本进行即时调整和增强。随后，这些脚本可以转换为与解析器兼容的格式以便在生产中使用，既能利用解析脚本的速度优势，又能保留嵌入式脚本的开发优势。

This dual-system approach requires careful management to avoid codebase fragmentation but has the potential to significantly improve the scripting workflow and provide a versatile toolset for game developers.

这种双系统方法需要仔细的管理以避免代码库的碎片化，但有潜力显著改进脚本工作流程，并为游戏开发者提供一套多功能的工具。

## Chapter 4 Summary 总结

This thesis has begun a comprehensive analysis of the performance and development implications associated with integrating parsed and embedded scripts within the game event handling framework of the Paradox Development Studio, with Lua serving as the embedded scripting language of choice. The primary objective of this comparative study was to evaluate the performance metrics of these different setups, as well as to assess their impact on the game development process. Through the development and iterative refinement of a prototype, this research examined the operational efficiency and development flexibility offered by the embedded script setup in contrast to the traditional parsed script approach.

本论文展开了一项对Paradox Development Studio游戏事件处理框架中解析脚本与嵌入式脚本集成的性能和开发影响的全面分析，其中Lua被选作首选的嵌入式脚本语言。这项比较研究的主要目标旨在评估这些不同配置的性能指标及其对游戏开发流程的影响。通过开发并迭代改进原型，本研究探讨了嵌入式脚本设置相较于传统解析脚本方法所提供的操作效率和开发灵活性。

The results of this research show that although the Lua-based prototype did not match the performance efficiency of the original parsed script setup - with runtimes approximately 1.5 to 15 times slower - the embedded scripting approach significantly improved the development process. This improvement is attributed to the facilitation of new scripting functionality and a more streamlined script development workflow. The primary factor contributing to the extended runtimes within the Lua setup was identified as the overhead associated with the transition from C++ to Lua for event evaluation.

研究结果表明，尽管基于Lua的原型在性能效率上未能达到原始解析脚本配置的水平——运行时间大约慢了1.5到15倍——但嵌入式脚本方法显著优化了开发流程。这种改进得益于新脚本功能的引入以及更加流畅的脚本开发工作流程。在Lua配置中导致运行时间延长的主要因素被识别为与C++到Lua的事件评估转换相关的开销。

Suggestions for optimizing the performance of Lua-based event handling include strategies such as batching function calls and moving in-game information from C++ to Lua. These suggestions aim to mitigate the identified performance overheads, potentially narrowing the gap between Lua's execution speeds and the native C++ setup.

针对优化基于Lua的事件处理性能的建议包括批处理函数调用和将游戏内数据从C++迁移到Lua等策略。这些建议旨在缓解已识别的性能开销，可能缩小Lua执行速度与原生C++配置之间的差距。

This research highlights the tradeoffs between performance efficiency and development flexibility inherent in the choice between parsed and embedded scripting languages for in-game event handling. While parsed scripts offer superior performance, embedded scripts, as exemplified by Lua, provide a dynamic and extensible platform for game development.

本研究突出了游戏事件处理中选择解析脚本与嵌入式脚本语言之间在性能效率与开发灵活性之间必须权衡的内在问题。虽然解析脚本在性能上占优势，但嵌入式脚本，以Lua为例，为游戏开发提供了一个动态且可扩展的平台。

The results of this study underscore the need to choose a scripting approach that meets the specific needs and priorities of the game development project at hand, balancing the demands of operational efficiency with the benefits of development agility.

这项研究的结果强调了根据手头游戏开发项目的具体需求和优先级选择脚本方法的必要性，需要在操作效率的需求与开发敏捷性的好处之间找到平衡。

## Bibliography

[1] Official GlowCode homepage. <http://www.glowcode.com/>, December 2011.

[2] Official Lua homepage. <http://www.lua.org/>, December 2011.

[3] Roberto Ierusalimschy. Lua Programming Gems, chapter 2. Lua.Org, 2008.

[4] Official Lua mailing list. <http://www.lua.org/lua-l.html>, December 2011.

[5] Official Microsoft Support page: How To Use QueryPerformanceCounter to Time Code. <http://support.microsoft.com/kb/172338>, December 2011.

[6] Mike Pall. [www.luajit.org](http://www.luajit.org/), December 2011.

[7] Lua users' wiki on the uses of Lua. <http://lua-users.org/wiki/luauses>, December 2011.

## Appendix A Comparison of New and Original Events Files

### A New Event

```lua
province_event[949] = {
    trigger_prefix = [[
        owner = get_owner(p)
        controller = get_controller(p)
        function anp_func(p, anp)
            o2 = get_owner(anp)
            return controlled_by(o2, anp) and government(o2, 22)
        end
    ]],
    trigger = [[
        not (controlled_by(p, owner)) and
        not (government(owner, 22)) and
        government(controller, 22) and
        any_neighbor_province(p, anp_func) and
        war_exhaustion(owner) >= 5 and
        garrison(p) >= 1000 and
        has_siege(p, false)
    ]],
    mean_time_to_happen_months = 300,
    mean_time_to_happen_prefix = [[ owner = get_owner(p) ]],
    mean_time_to_happen_factor = {
        5.0, [[ luck(owner, true) ]],
        0.8, [[ has_owner_religion(p, false) ]]
    }
}
```

### An Original Event

```
province_event = {
    id = 949
    trigger = {
        NOT = { controlled_by = owner }
        NOT = { owner = { government = steppe_horde } }
        controller = { government = steppe_horde }
        any_neighbor_province = {
            controlled_by = owner
            owner = { government = steppe_horde }
        }
        war_exhaustion = 5
        garrison = 1000
        has_siege = no
    }
    mean_time_to_happen = {
        months = 300
        modifier = {
            factor = 5.0
            owner = { luck = yes }
        }
        modifier = {
            factor = 0.8
            has_owner_religion = no
        }
    }
    title = "EVTNAME746"
    desc = "EVTDESC746"
    option = {
        name = "EVTOPTA746"
        controller = { country_event = 747 }
    }
}
```

## Appendix B Test Code

```cpp
open_debug_file();
int event_nr = args[1].GetInt();
lua_State *L = initLuaEnvironment();
try {
    lua_getglobal(L, "load_province_event");
    lua_pushinteger(L, event_nr);
    lua_call(L, 1, 0);
} catch (luabind::error e) {
    HandleLuaError(e.state());
} catch (std::exception e) {
    ASSERT_FAIL(e.what());
}
AddLine(CString("---Check province trigger---"));
int count = 0;
int nr_of_prov = CCurrentGameState::AccessInstance()->GetMaxNumberOfProvinces();
AddLine(CString("Nr of provinces: ") + CString(nr_of_prov));
CProvince *prov;
std::ofstream file;
file.open("timing_province.txt");
file << std::fixed << std::setprecision(5);
__int64 ctr1 = 0, ctr2 = 0, freq = 0, tot = 0;
for (int j = 0; j < _nNrOfLoops; j++) {
    lua_settop(L, 0);
}
lua_gc(L, LUA_GCCOLLECT, 0);
QueryPerformanceCounter((LARGE_INTEGER *)&ctr1);
for (int i = 1; i < nr_of_prov; i++) {
    prov = &CCurrentGameState::AccessInstance()->GetProvince(i);
    if (prov->GetOwner().IsValid()) {
        try {
            // calls the evaluation function in Lua
            count += check_province(L, event_nr, prov, i, j);
        } catch (luabind::error e) {
            HandleLuaError(e.state());
        } catch (std::exception e) {
            ASSERT_FAIL(e.what());
        }
    }
}
QueryPerformanceCounter((LARGE_INTEGER *)&ctr2);
tot += (ctr2 - ctr1);
QueryPerformanceFrequency((LARGE_INTEGER *)&freq);
file << "Average " << (tot * 1000.0 / freq) / _nNrOfLoops << " msecs in lua (old style)\n";
AddLine(CString(" "));
AddLine(CString("Lua event version (old):"));
AddLine(CString("Average time over ") + CString(_nNrOfLoops) + CString(" loops was ") + CString((tot * 1000.0 / freq) / _nNrOfLoops) + CString(" msec."));
AddLine(CString("Nr of true triggers: ") + CString(count));

// CODE FOR TESTING ORIGINAL EVENTS
//***************************************************************************
CID id(ID_TYPE_EVENT, event_nr);
CEvent *pEvent = (CEvent*)CEvent::GetObjectFromID(id);
CSimpleRandom Random;
CEventScope scope(Random.GetInteger());
scope._Country = CNullTag();
CProvince *pProvince;
count = 0;
tot = 0;
for (int j = 0; j < _nNrOfLoops; j++) {
    QueryPerformanceCounter((LARGE_INTEGER *)&ctr1);
    for (int i = 1; i < nr_of_prov; i++) {
        try {
            pProvince = &CCurrentGameState::AccessInstance()->GetProvince(i);
            if (pProvince->GetOwner().IsValid()) {
                scope._nProvince = i;
                bool trigger_true = pEvent->GetTrigger().Evaluate(scope);
                if (j == 0 && trigger_true)
                    count++;
            }
        } catch (std::exception e) {
            ASSERT_FAIL(e.what());
        }
    }
    QueryPerformanceCounter((LARGE_INTEGER *)&ctr2);
    tot += (ctr2 - ctr1);
}
QueryPerformanceFrequency((LARGE_INTEGER *)&freq);
file << "Average " << (tot * 1000.0 / freq) / _nNrOfLoops << "\tmsecs in C++\n";
AddLine(CString(" "));
AddLine(CString("Original event version:"));
AddLine(CString("Average time over ") + CString(_nNrOfLoops) + CString(" loops was ") + CString((tot * 1000.0 / freq) / _nNrOfLoops) + CString(" msec."));
AddLine(CString("Nr of true triggers: ") + CString(count));
AddLine(CString(" "));

// ENDS HERE
//****************************************************************************
file.flush();
file.close();
lua_close(L);
close_debug_file();
```
