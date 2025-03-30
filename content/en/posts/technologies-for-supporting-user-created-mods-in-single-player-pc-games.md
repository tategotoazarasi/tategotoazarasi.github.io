---
date: '2025-03-27T17:33:26+08:00'
draft: true
title: 'Technologies for Supporting User-Created Modifications in Video Games'
tags: [ "game-modding","single-player-games","pc-gaming","modding-frameworks","scripting-languages","domain-specific-languages","lua","python","c-sharp","interpreted-scripts","compiled-mods","mod-apis","sandboxing","security","game-architecture","extensibility","mod-lifecycle","event-handling","paradox-games","rimworld","mount-and-blade-bannerlord","minecraft","software-engineering","user-created-content","performance-trade-offs","modding-technology","game-development","scripting-vs-compiled","mod-security","embedded-languages","dsl-design","game-extensibility","sandbox-strategies","case-studies","future-research","modding-tools","api-design","community-mods","gaming-platforms","code-execution" ]
---

## Abstract

Single-player PC games increasingly offer **modding frameworks** that allow players to extend or alter gameplay through
user-created modifications (mods). This thesis provides an in-depth examination of the technologies game developers
employ to support modding, focusing on the *developer-side* design of mod APIs and scripting frameworks. We compare the
use of general-purpose **embedded scripting languages** (such as Lua or Python) versus **domain-specific languages** (
DSLs) tailored for game modding, discussing advantages in flexibility and familiarity against performance and control.
We analyze whether mods are executed as interpreted scripts at runtime or compiled into binaries, detailing the
trade-offs in *development ease*, *execution speed*, and *security*. The lifecycle of mods within the game is explored –
when and how mods are loaded, and how mod code is triggered through game events or hooks. A significant emphasis is
placed on **sandboxing and security** strategies for mod execution, especially for compiled mods, covering technical
mechanisms to prevent malicious code execution (e.g. restricting file system or network access). We include
architectural diagrams and code examples to illustrate these concepts. Detailed case studies of several moddable games –
including Paradox Interactive’s grand strategy titles, *RimWorld*, *Mount & Blade II: Bannerlord*, and *Minecraft* – are
presented to contrast different approaches. Finally, we propose novel research directions from a computer science
perspective, such as safer mod runtime environments and automated mod verification. The scope is limited to
single-player PC game modding. The thesis is written in academic style, with formal structure, and cites relevant
literature including **Forsslund’s evaluation of Lua for game event handling**.

## Introduction

User-created modifications, colloquially known as mods, represent a foundational and dynamic element within the
contemporary PC gaming ecosystem. For decades, this practice has been integral, empowering players to significantly
alter or expand upon the content and mechanics of existing video games. This phenomenon extends far beyond simple
cosmetic tweaks; it encompasses the creation of entirely new gameplay systems, narrative arcs, and visual overhauls,
often developed by dedicated hobbyists functioning as end-user developers. The impact of modding is profound: it
demonstrably extends the commercial lifespan and cultural relevance of numerous titles, fostering vibrant, deeply
engaged communities that coalesce around shared creative endeavors and personalized play experiences. Indeed, the
historical significance of modding is underscored by the fact that some of the most commercially successful and
genre-defining games trace their origins directly back to ambitious modifications of earlier titles, illustrating the
potent capacity for innovation inherent in user-generated content within interactive entertainment.

Technically, a mod is not standalone software. It functions intrinsically as an extension of the original game's
codebase and data assets. Consequently, the potential for a thriving modding scene hinges critically on deliberate
architectural decisions made by the game's developers. To facilitate modding effectively, developers must proactively
design their games with extensibility as a core principle. This involves providing the necessary infrastructure—often in
the form of well-documented Application Programming Interfaces (APIs), specific "hooks" into the game's routines, or
robust scripting capabilities—that permits external, user-created code to safely and predictably interact with the
underlying game engine. This intentional design for modification transforms the game from a closed product into an open
platform for creativity.

From a software engineering standpoint, enabling game modding can be viewed as a specific instantiation of open software
extension. This concept resonates strongly with classic software engineering ideals, such as David Parnas's principles
regarding software designed for change and extension, as noted by Scacchi (2011) in the context of modding communities.
Many modern game development studios achieve this crucial extensibility through sophisticated techniques, prominently
featuring the use of embedded scripting languages and software product-line architectures. These approaches allow for
the modular integration of new content and features, often without necessitating modifications to the compiled core game
engine itself. By strategically exposing certain facets of the game logic via high-level scripts or dedicated mod APIs,
developers effectively empower end users to innovate, customize, and fundamentally reshape their gaming experience. This
dynamic has led some researchers to characterize modding as a unique form of end-user software engineering, effectively
blurring the traditional distinction between the game developer and the player.

Furthermore, the provision of official modding support by developers offers substantial advantages over scenarios where
modifications arise purely organically through player-driven reverse-engineering efforts. Official tools and APIs
typically lead to greater stability for mods, provide clearer and more accessible pathways for creators to interact with
game systems, and enable deeper, more seamless integration of user-generated content. Conversely, the absence of
sanctioned support often results in a fragmented and volatile modding landscape, characterized by pervasive
compatibility issues between different mods or across subsequent updates of the base game. The longevity and
functionality of mods in such environments frequently depend precariously on the continued, often arduous,
reverse-engineering efforts of a few dedicated community members. The early history of modding for Mojang's Minecraft
serves as a pertinent case study: the initial lack of a comprehensive official API necessitated reliance on
community-developed frameworks (like ModLoader and Forge), which, while ingenious, presented ongoing challenges in
maintaining compatibility as the core game rapidly evolved. In stark contrast, well-designed official modding APIs can
establish a far more robust and sustainable foundation, helping to ensure that modifications remain functional and
accessible over the long term, even amidst significant updates to the base game.

However, designing and implementing a robust, flexible, and secure modding system presents considerable technical and
strategic challenges for developers. A primary decision point involves the choice of languages or scripting systems to
be made available to mod authors. Should the modding interface leverage an established, general-purpose language
familiar to many programmers (such as Lua, Python, or C#), offering a potentially gentler learning curve and access to
existing libraries? Or should developers invest in creating a bespoke domain-specific language (DSL), tailored precisely
to the game's architecture and concepts, potentially offering greater control and optimization but requiring users to
learn a new syntax? This decision involves complex trade-offs concerning ease of use for modders, runtime performance
implications, integration complexity, and inherent security vulnerabilities.

Beyond language choice, developers must carefully engineer when and how mods are loaded into the game environment and
executed. Mods might be interpreted dynamically at runtime via an embedded scripting engine, offering flexibility but
potentially impacting performance. Alternatively, they could be pre-compiled into native machine code or an intermediate
language (like .NET bytecode) that the game loads directly, which can yield significant speed advantages but may
introduce complexities in the build process for modders and potential compatibility issues. Each approach carries
distinct pros and cons regarding execution speed, development convenience for modders, and the overall stability of the
modded game.

Perhaps the most critical challenge, particularly when mods involve executing arbitrary code, revolves around sandboxing
and security. Unchecked or poorly isolated mods possess the potential to destabilize the game client, cause crashes,
corrupt save data, or, in worst-case scenarios, execute malicious code that could compromise the player's entire system.
Consequently, implementing effective security measures is paramount. This thesis will delve deeply into various
techniques employed to mitigate these risks, including strategies such as carefully restricting the capabilities exposed
through mod APIs, employing virtual machines or other safe execution environments to isolate mod code, and implementing
permission systems for mods.

Therefore, this thesis undertakes a comprehensive investigation into the technological frameworks, architectural
patterns, and specific design decisions employed by video game developers to facilitate user-created modifications on
the PC platform. The primary objectives are multifaceted: to conduct a detailed analysis and comparison of various
approaches to modding API design; to examine the integration strategies for different programming and scripting
languages; to evaluate distinct methods of mod code execution (interpreted versus compiled); to scrutinize techniques
for effective sandboxing; and to assess the implementation of security measures designed to safeguard both the integrity
of the game and the security of the end-user's system. Ultimately, this work seeks not only to identify prevailing best
practices in the provision of official modding support but also to propose novel concepts and potential future research
directions that could lead to even safer, more powerful, and more accessible modding frameworks in the future.

The investigation is structured as follows. Section 2 will establish essential background knowledge regarding mod
frameworks and foundational concepts, including a comparative overview of scripting-based versus compiled mod paradigms.
Section 3 will engage in an in-depth discussion of the design considerations for modding interfaces: Section 3.1
compares the merits and drawbacks of embedded general-purpose scripting languages versus game-specific DSLs; Section 3.2
analyzes the trade-offs between interpreted versus compiled mod execution models; Section 3.3 describes various mod
loading mechanisms and lifecycle management strategies, including event triggers; and Section 3.4 provides a thorough
examination of sandboxing techniques and security strategies. Following this theoretical groundwork, Section 4 will
present illustrative case studies of several notable moddable PC games – specifically examining Paradox Interactive’s
grand strategy titles (such as the Europa Universalis and Crusader Kings series), Ludeon Studios’ colony simulator
RimWorld, TaleWorlds Entertainment’s action RPG Mount & Blade II: Bannerlord, and Mojang Studios’ ubiquitous sandbox
game Minecraft. These case studies will highlight how the theoretical concepts discussed in Section 3 are implemented,
adapted, and navigated in real-world development practice. Subsequently, Section 5 will discuss promising future
research directions, proposing novel ideas aimed at advancing the state-of-the-art in creating safer, more expressive,
and more performant modding frameworks. Finally, Section 6 will conclude the thesis, summarizing the key findings and
contributions. It is important to note that the scope of this discussion is deliberately limited to single-player game
modifications on the PC platform; the complexities introduced by multiplayer environments (e.g., anti-cheat mechanisms,
network synchronization) and console platforms (e.g., stringent certification processes, hardware limitations) present
distinct challenges that fall beyond the purview of this work. Throughout the analysis, a formal academic tone will be
maintained, utilizing precise terminology and citing relevant sources drawn from academic research, industry
publications, and official developer documentation where appropriate.

## Background and Key Concepts in Game Modding

The capacity for modification, commonly known as "modding," represents a significant aspect of contemporary video game
culture and longevity. Within this context, a mod is fundamentally a user-created package designed to alter a base
game’s existing data or runtime behavior. The scope of such modifications can vary dramatically, ranging from subtle
configuration adjustments and purely cosmetic asset replacements to the introduction of entirely new gameplay systems
articulated through sophisticated code. Common typologies include content mods that introduce new items, levels, or
characters; total conversions that fundamentally overhaul a game's setting, core ruleset, or even genre; and
modifications targeting the user interface for improved usability or aesthetics. A defining characteristic, however,
remains the mod's intrinsic dependency on the original game; it is not a standalone entity but rather content loaded and
processed by the base game's engine at runtime. This inherent dependency necessitates that the host game deliberately
provides architectural entry points, or hooks, enabling mods to integrate seamlessly. From a software architecture
perspective, therefore, mods function analogously to extensions or plugins within a larger host system, extending its
capabilities beyond the developers' original implementation.

Architecturally, enabling this extensibility draws parallels with traditional software engineering practices, which
often rely on mechanisms like plugin interfaces, callback systems, or integrated scripting environments. Games,
similarly, must expose a well-defined interface to accommodate mods. Two predominant approaches emerge in practice: (1)
Data-Driven modding, wherein the game dynamically reads external data files or scripts—often structured using custom
formats, Domain-Specific Languages (DSLs), or standard formats like JSON/XML—to configure or dictate game behavior. This
model is particularly prevalent for adding discrete content elements like items or quests. (2) Code-Driven modding,
which involves the game loading and executing user-supplied code, whether in the form of scripts processed by an
embedded interpreter or pre-compiled binary modules that directly interface with the game's Application Programming
Interface (API). It is common for sophisticated modding systems to employ a hybrid strategy, leveraging data-driven
techniques for content definition while supporting code-driven approaches for complex logic and behavioral
modifications.

Conceptually, beyond the technical implementation, the integration of modding capabilities can be understood through
several distinct models, each shaping the potential scope and impact of user creations. One prevalent model is additive
modding, which prioritizes allowing modders to introduce new content and features primarily through new files, without
directly modifying the original game assets. This philosophy, notably employed in many titles by Paradox Interactive,
inherently promotes better compatibility between disparate mods and significantly reduces the likelihood of conflicts
arising when multiple modifications attempt to alter the same core resources. By encouraging the creation of
supplementary files and directories that the game engine discovers and loads, developers can foster a more stable and
cooperative modding ecosystem.

In contrast, another conceptual model explicitly enables modders to override or replace existing game content. This
might involve substituting assets like textures or models, altering configuration files, replacing scripts, or even
modifying core logic components. While granting modders immense power to reshape the fundamental game experience, this
approach carries a significantly higher risk of compatibility issues. Conflicts frequently arise when multiple mods
target the same file or resource, or when updates to the base game alter the structure, format, or dependencies of the
files mods rely upon, potentially breaking existing modifications. A third, often complementary model focuses on
scripting and logic extension. Here, developers expose specific APIs, allowing modders to extend or customize game
behavior and functionality using provided scripting languages, thereby circumventing the need to directly manipulate the
engine's compiled codebase or sensitive internal structures. Finally, the most ambitious manifestation of modding is
arguably the total conversion, where modifications are so extensive that they radically transform the game's setting,
rules, aesthetics, and core mechanics, effectively creating an entirely new experience built upon the original game's
technological foundation. Such projects underscore the profound creative potential unlocked by robust and flexible
modding support.

Central to the implementation of code-driven modding is the critical decision regarding the language used for expressing
mod logic. One common strategy involves embedding a full-fledged scripting language directly into the game engine. Lua
is a frequently favored choice, renowned for its lightweight footprint, execution speed, and design centered around
embeddability. Other prominent languages like Python and JavaScript have also found application within various game
engines, offering different ecosystems and feature sets. The alternative path involves the creation of a domain-specific
language (DSL). A DSL, in this context, constitutes a custom, often simplified language crafted by the game developers
specifically for mod scripting or configuration tasks. In practice, game-centric DSLs often manifest as loosely
structured text files—sometimes resembling standard data formats like JSON or XML, other times adopting a unique
syntax—which are parsed and interpreted by a custom component within the game engine. These DSLs tend to possess a
deliberately limited scope, focusing intently on game-specific concepts such as triggering predefined events, applying
status effects, or defining item properties, rather than providing general-purpose programming constructs like complex
control flow or data structures. Each choice, between a general-purpose embedded language and a bespoke DSL, carries
distinct implications for development workflow, modder accessibility, expressive power, and potential performance
characteristics, demanding careful consideration during engine design.

Further distinguishing implementation strategies is the fundamental dichotomy concerning whether mod code is interpreted
dynamically at runtime or pre-compiled into a binary format before execution. Interpreted mods, typically distributed in
source code or an intermediate bytecode representation, are executed by a runtime environment—such as a scripting
virtual machine (VM) or Just-In-Time (JIT) compiler—integrated within the game process itself. This approach is
inherently linked to the use of scripting languages. Conversely, compiled mods are delivered as binary modules (e.g.,
Dynamic Link Libraries (DLLs) on Windows, or Java class files for JVM-based games) that the game loads directly into its
memory space, often allowing the mod code to execute at near-native speeds. Compiled approaches may permit modders to
use the same high-performance language as the core game (for instance, a game developed in C# supporting mods also
written in C#, compiled to standard .NET assemblies). The trade-offs between these interpreted and compiled models
involve a complex interplay between factors like runtime performance, ease of development and distribution,
cross-platform compatibility, and, critically, security and sandboxing capabilities, which we will explore further.

Effective support for any modding paradigm also hinges on the careful design and implementation of the mod loading
system. The game must possess a mechanism to reliably detect installed mods—commonly achieved by scanning a designated
mods directory within the game's installation path or user data folders, or managed via an external launcher application
where users explicitly enable or disable specific mods. Once detected, the loading process itself requires
consideration. Some games opt to load all enabled mods upfront during the initial startup sequence, integrating their
assets and logic before or during the main game initialization phase. Others might support more dynamic loading,
although true runtime hot-loading (adding or removing mods while the game is actively running) is less common due to its
inherent complexity and potential for instability. The precise order in which mods are loaded can become critically
important, especially when multiple mods modify overlapping aspects of the game or when explicit dependencies exist
between mods. Managing these dependencies robustly is a common challenge. Subsequent to loading, the mod's code must be
invoked at appropriate junctures in the game's execution flow. For example, a mod might need to execute initialization
logic when a new game session begins, perform updates on each frame or game tick, or react to specific in-game
occurrences like player actions or world events. To facilitate this, the game typically provides a system of hooks,
callbacks, or a general-purpose event subscription mechanism, defining the mod's lifecycle and interaction points.

A paramount concern, interwoven through all aspects of modding architecture, is security. Unlike the game's first-party
code developed in a controlled environment, mods originate from potentially untrusted third-party creators. This reality
introduces significant security risks. If mods are permitted to execute arbitrary code without constraints, a
maliciously crafted mod could potentially compromise the user's system—for example, by accessing sensitive files,
monitoring network traffic, installing malware—or simply destabilize the game itself, leading to crashes or
unpredictable behavior. While the threat profile differs slightly between single-player contexts (where the primary risk
is to the individual player's machine) and multiplayer scenarios (where mods might also be leveraged for unfair
advantages or cheating), the need for robust sandboxing of mod execution remains a critical consideration in all cases.
Effective sandboxing strategies can encompass various techniques, including strictly limiting the APIs accessible to mod
scripts (e.g., prohibiting direct file system or network access), running mod code within a heavily constrained virtual
environment or even a separate operating system process, or employing static code analysis tools to detect and reject
potentially dangerous code patterns before loading. Despite the availability of these techniques, many games adopt a
more pragmatic approach to security. They often rely heavily on community vigilance, moderation processes within mod
distribution platforms (like Steam Workshop), and user discretion to identify and avoid malicious mods, frequently
choosing not to implement stringent runtime sandboxing due to the associated performance overhead or development
complexity. This pragmatic reliance on external factors, however, is not foolproof; even seemingly secure scripting
environments can harbor vulnerabilities exploitable by malicious actors if not meticulously sandboxed, as demonstrated
by historical incidents such as security breaches involving malicious Dota 2 mods exploiting vulnerabilities in the
game's embedded JavaScript engine.

Ultimately, designing a game architecture that effectively enables rich and diverse modding capabilities necessitates
careful, deliberate planning from the project's inception. The underlying game engine should ideally be constructed with
extensibility as a core principle, often leveraging modular design patterns that facilitate the integration of external
content and logic. Unreal Engine's modular structure, allowing mods to be loaded as discrete dynamic plugins,
exemplifies such forethought. Furthermore, the widespread adoption of data-driven design philosophies—where significant
portions of game content, rules, and configurations are defined in external, human-readable data files (like XML or
JSON) rather than being hardcoded—can substantially lower the barrier to entry for modders. RimWorld serves as a
compelling case study, with its extensive use of XML definitions for virtually all game elements, making it remarkably
accessible for users to add new items, creatures, factions, and complex scenarios without needing to engage with
compiled code.

In conclusion, supporting a vibrant modding community requires developers to navigate a complex set of interconnected
challenges spanning conceptual models, architectural choices, language selection, runtime execution, loading mechanics,
and security enforcement. A delicate balance must constantly be struck between providing sufficient flexibility and
power to modders—exposing adequate functionality, utilizing familiar and accessible tools and languages, and allowing
for significant creative freedom—and maintaining the robustness, stability, performance, and security of the core game
experience for all users. The design of the modding framework itself is paramount, demanding careful consideration of
ease of use for creators of varying skill levels, extensibility to accommodate future innovation, stability across game
updates, inherent security against misuse, and minimal performance degradation. Successfully achieving this balance is
not merely a technical feat but a strategic investment in a game's long-term engagement and cultural impact.

## Designing Modding Frameworks: Languages and Execution Models

### Choosing the Foundation: Scripting Mechanisms in Modding Framework Design

A foundational decision confronting developers when designing a game's modding framework revolves around the choice of
scripting mechanism. This pivotal choice dictates how modders will interact with the game's systems, define new content,
and implement custom logic. Broadly, the options diverge into two primary paradigms: integrating a general-purpose
*embedded scripting language* or crafting a bespoke *domain-specific language (DSL)* tailored explicitly for the game's
modding needs. Each pathway carries profound and distinct implications, significantly shaping the mod developer
experience, the complexity of game engine integration, runtime performance characteristics, security considerations, and
ultimately, the scope and nature of the modding ecosystem that emerges around the game.

#### The Embedded Language Approach: Leveraging Existing Ecosystems

Many game developers opt to embed an existing, general-purpose language interpreter directly into their game engine,
effectively providing a *built-in scripting engine* for modders. This strategy leverages the maturity, feature sets, and
often widespread familiarity of established programming languages. Lua stands out as a prominent and historically
significant example, renowned for its successful integration into numerous influential titles across various genres,
including the intricate automation systems of **Factorio**, the user interface modifications in **World of Warcraft**,
the expansive modding capabilities of **Civilization IV**, and event scripting in Paradox Interactive's earlier titles
like *Europa Universalis III*. Lua's enduring appeal stems from its design philosophy: simplicity, a remarkably small
memory footprint, and execution speed, attributes that make it particularly well-suited for embedding within larger
C/C++ applications. As Forsslund (2013) succinctly noted, *“Lua is an embedded scripting language that has been used in
computer games since the 1990s and is known to have a small memory footprint and to be fast.”* Its lightweight nature
and straightforward C API facilitate integration, a key factor in its adoption. For instance, Oskar Forsslund's work
specifically involved integrating Lua into *Europa Universalis III* precisely for event scripting, demonstrating its
practical application even within complex simulation environments.

Beyond Lua, other general-purpose languages have also found niches in game modding. Python, celebrated for its
high-level, readable syntax and extensive standard library, powered aspects of game logic in titles like *Civilization
IV*. Its expressiveness makes it attractive for complex modding tasks, although potential performance characteristics,
especially in real-time contexts, and the dependency on a Python interpreter can be drawbacks. JavaScript, ubiquitous in
web development, is increasingly utilized within game engines, particularly for user interface development. Valve’s
Source 2 engine, for example, employs JavaScript for UI scripts (as seen in Dota 2’s Panorama UI), and the *Unity*
engine, while primarily C#-focused, supports HTML/JS for certain UI elements. However, JavaScript's performance in
computationally intensive game logic and managing security in untrusted mod contexts remain points of careful
consideration.

A particularly powerful option in the embedded category is C#, especially prevalent due to its tight integration with
popular engines like Unity and its use in titles such as *RimWorld* and *Cities: Skylines*. C# offers strong
performance (often benefiting from Just-In-Time compilation), a robust object-oriented feature set, and excellent
tooling support. Its capabilities enable modders to achieve deep, systemic modifications, altering core gameplay
mechanics, manipulating complex data structures, and even extending the game's editor itself. *RimWorld*, for instance,
relies heavily on C# for mods that introduce intricate new systems, while *Cities: Skylines* exposes a comprehensive
modding API based on C#. This approach, where mods are written in the same language as significant parts of the game
engine itself (similarly seen with C++ in Unreal Engine), begins to blur the lines, effectively turning the game's own
sophisticated API into the "language" for mods.

The primary advantage conferred by adopting a real programming language lies in its inherent **expressiveness and
familiarity**. Modders can potentially leverage existing programming knowledge, reducing the initial learning curve.
These languages come equipped with rich, pre-built feature sets, data structures, and standard libraries (though access
might be intentionally restricted for security). Furthermore, the mature runtimes often provide valuable development
tools like debuggers, profilers, and optimizing compilers (e.g., JIT compilers like LuaJIT), which can significantly
enhance the mod development workflow.

However, integrating a general-purpose language is not without significant challenges. A non-trivial integration effort
is required to embed the language runtime (the interpreter or virtual machine) and establish a robust communication
bridge – typically a **C API or binding layer** – between the game's native code (often C/C++) and the scripting
environment. This binding layer is critical for allowing scripts to safely query game state, trigger game actions, and
modify data without compromising stability or security. Developers must meticulously design this API, deciding precisely
which aspects of the game's internals to expose. This creates an ongoing maintenance burden: developers must anticipate
the future needs of modders or continuously extend the scripting API to enable new modding possibilities. A powerful
embedded language can also be "too powerful," presenting security risks if not carefully constrained within a sandbox.
Without restrictions, languages like Python or C# could potentially allow mods to perform undesirable actions like
arbitrary file system access or establishing network connections. Consequently, robust sandboxing mechanisms are
essential, often involving limiting access to standard libraries or providing custom, safer wrappers around sensitive
functions. The precise nature and limitations of the exposed API are critical design elements, as will be explored
further regarding sandboxing (e.g., in concepts analogous to Section 3.4's discussion).

Performance is another critical consideration. Embedded scripting languages almost invariably introduce some runtime
overhead compared to native compiled code. Each transition between the game engine's execution and the script's virtual
machine involves a context switch and potential data marshalling, which can become costly if performed frequently within
performance-critical loops. Moreover, interpreted or even JIT-compiled script code generally runs slower than highly
optimized native C++ code. Forsslund's (2013) empirical evaluation vividly illustrates this: rewriting *Europa
Universalis III*'s event trigger system in Lua resulted in a significant performance degradation, with total event
processing time becoming approximately **six times slower** than the original C++ implementation. While advancements
like JIT compilers (e.g., LuaJIT) can substantially mitigate this performance gap – Forsslund observed notable speedups
using LuaJIT compared to standard Lua – the fundamental trade-off remains. As Forsslund cautions, *“Embedded scripting
languages are often slower than the language you are using for your main program.”* Developers must therefore carefully
assess whether the gains in flexibility and modder empowerment justify the potential performance cost, a decision
heavily dependent on how frequently scripts are executed and how performance-sensitive those specific game systems are.

#### The Domain-Specific Language (DSL) Approach: Tailored Control and Optimization

As an alternative to embedding existing languages, some developers choose to design a custom scripting or configuration
language specifically for their game – a *domain-specific language (DSL)*. Unlike general-purpose languages intended for
a wide array of tasks, a DSL is narrowly focused, designed to express particular game concepts, events, behaviors, or
data configurations in a highly concise and targeted manner. Often, these DSLs are not Turing-complete programming
languages but rather structured data formats or declarative systems.

Paradox Interactive's proprietary Clausewitz engine, powering their grand strategy titles like *Europa Universalis*,
*Crusader Kings*, and *Hearts of Iron*, provides a quintessential example. This engine utilizes a **proprietary
scripting syntax** primarily for defining game events, diplomatic actions, character behaviors, mission trees, and
myriad other game elements. A typical snippet defining an event might resemble:

```plaintext
country_event = {
    id = "mymod.1"                # Unique identifier for the event
    title = "mymod.1.t"            # Localization key for display text
    picture = "GFX_evt_picture"    # Associated event image

    trigger = {                  # Conditions for the event to fire
        has_government = democratic
        NOT = { has_stability = 0.5 } # Example condition: stability below 0.5
    }

    immediate = {                # Effects that happen instantly when fired
        add_stability = -0.10
        country_modifier = { name = "recent_unrest" duration = 365 }
    }

    option = {                   # Player choices (optional)
        name = "mymod.1.a"       # Localization key for option A text
        ai_chance = { factor = 70 }
        add_political_power = 50
    }
}
```

This example, characteristic of Paradox's event scripting, showcases a relatively simple, declarative syntax. It's not
general-purpose code but a structured description of an event entity, defining its conditions (*trigger* block) and
consequences (*immediate* and *option* blocks). The game engine contains a specialized parser for these script files. It
interprets the DSL, often translating it into highly optimized internal data structures or directly into executable
logic within the C++ engine. Paradox's system is fundamentally *event-based* and heavily optimized for efficiently
filtering and processing a vast number of potential events based on game state changes. The DSL is tightly interwoven
with core game concepts; elements like trigger conditions (`has_government`) and game state scopes (e.g., implicit
understanding of `ROOT`, `FROM`, `owner`, `province` scopes) are directly understood by the parser and engine. Another
notable example, albeit closer to a traditional programming language in appearance, is Bethesda's **Papyrus** language,
used in *Skyrim*, *Fallout 4*, and subsequent titles. While possessing more procedural constructs than Clausewitz
scripts, Papyrus is custom-built for the Creation Engine, featuring a specific instruction set tightly coupled to game
objects and events, and lacking direct access to arbitrary operating system functions.

The primary benefits of the DSL approach lie in **fine-grained control and potential for significant optimization**.
Because the language is purpose-built for the game, developers can architect its syntax and semantics to align perfectly
with the engine's internal workings. This allows the engine to parse, interpret, and execute script-defined logic with
maximum efficiency. In the Paradox engine, for instance, the event scripts are parsed into internal representations (
like logical condition trees and effect lists) that the C++ core can evaluate extremely rapidly. Effectively, the DSL
often acts as a high-level configuration layer that the game compiles or interprets into highly performant internal code
or data structures. This direct translation yields substantial performance advantages, particularly crucial in
simulation-heavy games where thousands of rules or events might need evaluation each cycle. Forsslund's thesis again
provides key evidence here, highlighting that the original EU3 event system, based on parsed DSL scripts effectively
driving C++ logic, was **much faster** than the experimental embedded Lua implementation. Paradox's exploration of Lua
was driven by a desire for increased flexibility, but the performance trade-off proved significant; the Lua event
handling was not "nearly as fast" as the C++ equivalent, missing their target of staying within a 3x slowdown by
exhibiting closer to a ~6x slowdown in tests. Thus, for domains where performance is paramount and the scope of desired
modding activity is relatively well-defined (e.g., defining new events, items, or AI parameters rather than entirely new
game systems), a DSL can be a superior technical choice.

Furthermore, DSLs often provide **security and sandboxing by construction**. A carefully designed, limited DSL can
simply *omit* any constructs or operations that could pose security risks or lead to instability. Paradox's scripting
language, for example, has no built-in concepts for file I/O, network communication, or direct memory manipulation.
Scripts can only interact with the game world through a predefined, curated set of effects (`add_stability`,
`change_province_owner`, etc.) and query game state through specific triggers (`has_government`, `num_of_cities`, etc.).
This inherently sandboxed nature makes it extremely difficult for mods to "break out" of their intended boundaries or
execute malicious code. Modders cannot inject arbitrary binary code or call arbitrary system APIs if the language itself
provides no mechanism to do so. The game engine only exposes specific, controlled hooks. In essence, DSLs achieve
*security via limited expressiveness*, a stark contrast to the powerful, general-purpose nature of embedded languages
that necessitates explicit sandboxing measures. They can also be designed to enforce game rules and constraints directly
at the language level, further contributing to safer modifications. From a usability perspective, particularly for
non-programmers, a well-designed DSL using game-specific terminology can potentially be easier to learn and use for
common modding tasks compared to learning a full general-purpose language.

However, the DSL approach is not without its drawbacks. A significant challenge is the potentially **steep learning
curve**. Since each DSL is unique to its specific game or engine, modders must invest time in learning the intricacies
of its custom syntax, semantics, available commands, and often, its undocumented quirks (especially if the DSL is
proprietary and documentation is sparse). This contrasts sharply with leveraging a popular language like Lua or Python
where extensive tutorials, documentation, and community support already exist. Moreover, the tailored nature of a DSL
inherently means it is likely **less powerful and flexible** than a general-purpose language. Its limited scope might
restrict modders who envision implementing fundamentally new, complex systems or algorithms that the DSL designers did
not anticipate. For instance, a modder might find it impossible to implement a sophisticated new AI behavior or a
complex economic simulation model if the DSL lacks the necessary expressive power or primitives. In such scenarios,
modders may resort to **clever workarounds within the DSL's limitations, utilize external tools** to generate complex
script files, or, if the platform allows, turn to unofficial and often fragile methods like binary patching. Another
significant consideration is the **maintenance burden** placed squarely on the game developers. Every time the modding
community desires new functionality or expresses a need to interact with a previously inaccessible game system, the
developers might need to explicitly add new script commands, triggers, effects, or even modify the DSL's grammar and
parser. This iterative development cycle for the DSL itself can be resource-intensive, contrasting with an embedded
language where modders might creatively combine existing language features to achieve novel results without direct
developer intervention. As one developer noted regarding embedded languages, *"The problem with approaches like using
Lua is you’re limiting what modders have access to and you’re on the hook for anticipating their needs"* – this
constraint often applies even more strongly to the inherently more restricted scope of DSLs.

#### Hybrid Approaches and Evolving Paradigms

Recognizing the distinct advantages and disadvantages of each approach, some games employ a **hybrid strategy**. This
often involves using a simple DSL (frequently data files like XML, JSON, or YAML) for defining static data – such as
item properties, unit statistics, or localization text – while simultaneously providing an embedded scripting language (
like Lua or C#) for implementing more complex, dynamic logic and behaviors. *RimWorld*, for example, uses XML
extensively for defining things like creature types, items, and research projects, but relies on C# for mods that
introduce sophisticated new gameplay mechanics or AI. This tiered approach can offer a balanced solution, leveraging the
ease of use and structure of data formats for content definition, while providing the power of a full programming
language for procedural logic, thus using the right tool for each specific job.

Forsslund (2013) even proposed an intriguing workflow concept: using an embedded language like Lua for rapid prototyping
and development due to its flexibility, and then potentially translating or "compiling" the finalized scripts into the
game's more performant internal format (akin to a DSL execution pathway) for the released version of the mod or game
content. While not commonly implemented in practice due to the complexity of maintaining parallel systems and ensuring
accurate translation, this idea points towards potential future directions in bridging the flexibility-performance gap.

The landscape is further complicated by the rise of compiled mods, particularly with C# in Unity and C++ in Unreal
Engine. As mentioned earlier, when mods are written in the same high-performance language as the game engine and
compiled into dynamic link libraries (DLLs) or equivalent binary modules (as seen in *Mount & Blade II: Bannerlord*,
which supports both XML data files and C# DLL plugins), the distinction blurs. The game's own extensive API effectively
becomes the modding "language." This grants enormous power and performance potential but also introduces significant
security considerations related to executing potentially untrusted binary code and requires a higher level of technical
proficiency from modders.

#### Impact on the Modding Ecosystem and Concluding Thoughts

Ultimately, the choice of scripting mechanism profoundly shapes the modding landscape that develops around a game.
Paradox Interactive's consistent reliance on their proprietary Clausewitz scripting DSL (often complemented by Lua for
specific tasks in some titles) has cultivated a large, dedicated, and highly experienced modding community deeply adept
at manipulating intricate game rules, historical events, and vast amounts of game data primarily through structured text
files. This DSL-centric approach fosters a specific style of modding focused on content expansion and systemic tweaks
within the established framework. In contrast, *RimWorld*'s combination of accessible XML for data definition and
powerful C# for complex logic fosters a different kind of ecosystem, enabling mods ranging from simple item additions to
total gameplay overhauls, albeit with challenges arising from a less formally defined API that often necessitates
community-driven documentation and reverse engineering. *Mount & Blade II: Bannerlord*'s module system supporting both
XML and compiled C# offers immense capability but also highlights the security and stability challenges inherent in
allowing binary code execution. *Minecraft* presents yet another model, particularly the Java Edition, where the absence
of a strong official modding API led to the rise of powerful community-developed APIs like Forge and Fabric, built atop
Java. This showcases the incredible potential of community collaboration but also the inherent difficulties in
maintaining stability, compatibility, and security across a fragmented, community-led ecosystem. The Bedrock Edition of
*Minecraft*, conversely, utilizes a more controlled scripting API for its "add-ons," offering a safer and more
streamlined experience but arguably at the cost of the sheer flexibility found in the Java modding scene.

In summary, the decision between embedding a general-purpose language and designing a DSL involves navigating a complex
web of trade-offs. **Embedded languages** like Lua, Python, JavaScript, and C# offer **flexibility, expressiveness, and
potential familiarity**, leveraging existing ecosystems and tooling, but demand careful **integration, robust
sandboxing, and acceptance of potential performance overhead**. **DSLs**, exemplified by Paradox's scripts or Bethesda's
Papyrus, promise **tailor-made integration, optimized performance, and inherent security through limited scope**, but
impose a **steeper learning curve for a unique language, restrict modding possibilities, and place a continuous
maintenance burden on developers**. Hybrid approaches seek a middle ground, while the rise of compiled mods presents its
own set of opportunities and challenges. The optimal choice is context-dependent, hinging on factors such as the game's
genre and technical demands (e.g., performance sensitivity in simulations), the target audience for modders (casual
tweakers vs. experienced programmers), the desired depth and breadth of modifiability, and the development resources
available for building and maintaining the modding framework itself. Forsslund's findings regarding the Lua vs. C++
performance in EU3 remain a salient data point, underscoring the critical need to balance the allure of flexibility and
rapid development against the hard constraints of runtime efficiency in computationally intensive game environments.

### Interpreted vs. Compiled Mod Execution

A fundamental architectural decision in designing game modding capabilities revolves around the execution model for
user-generated code: specifically, whether mod code operates in an **interpreted** environment at runtime or is *
*compiled** ahead-of-time into binary formats. This choice represents a critical dimension in mod system design, often
correlating with, yet distinct from, the choice of programming language employed. For instance, a game might leverage
Lua, a language typically interpreted or Just-In-Time (JIT) compiled at runtime, or it might permit mods written in C++,
which necessitate compilation into platform-specific binaries like Dynamic Link Libraries (DLLs) before execution. Each
pathway carries profound and multifaceted implications, significantly influencing not only runtime **performance
characteristics** and system **security postures** but also shaping the entire **mod development workflow**,
accessibility, and long-term maintainability.

#### Interpreted Mods: Runtime Scripting Environments

The interpreted model involves executing mod logic that is not represented as native machine code within the primary
game executable. Instead, the game incorporates an interpreter or a virtual machine (VM) specifically designed to read
and execute mod scripts dynamically during gameplay. Classic implementations often embed scripting languages renowned
for their flexibility and ease of integration, such as Lua or Python. In this paradigm, mod files—frequently distributed
as human-readable text scripts or intermediate bytecode—are loaded by the game engine at runtime. The embedded
interpreter then processes these instructions sequentially or via bytecode execution.

A primary advantage of interpreted mods lies in their accessibility and ease of development. Mod creators can often
write and modify scripts using simple text editors, bypassing the need for complex compilation toolchains and build
processes, thereby enabling rapid iteration cycles. Distribution is also simplified, as plain text scripts are easily
shared. Furthermore, interpreted systems inherently facilitate cross-platform compatibility; a Lua script mod, for
example, can function identically on Windows, macOS, or Linux versions of the game, provided the game's embedded runtime
environment is consistent across platforms. Its execution depends solely on the game's integrated interpreter, not the
underlying operating system's binary format requirements.

From the game developer's standpoint, implementing an interpreted modding system necessitates integrating the chosen
language's runtime environment and meticulously exposing a controlled Application Programming Interface (API) or "hooks"
into the game's state and event system. When specific game events occur (e.g., a character interaction, completion of a
quest), the game engine triggers the execution of relevant mod code, perhaps by invoking functions like `lua_call` or
its equivalent in the chosen scripting language, passing necessary context and receiving results.

However, the runtime nature of interpretation introduces performance considerations. The **timing** and method of
interpretation are crucial. While rudimentary line-by-line interpretation on each execution is typically too slow for
practical game applications, most modern interpreted systems employ optimizations. Many, like standard Lua, compile
scripts into an intermediate bytecode representation once upon loading. This bytecode is then executed by the VM,
offering significantly better performance than raw interpretation. Despite this, inherent overhead remains compared to
native code execution. As empirical studies, such as Forsslund's measurements, have demonstrated, even with bytecode
compilation, the "bridging" layer between the game's native code (often C++) and the scripting language (like Lua)
introduces latency. This overhead stems partly from **context switching** between the native engine and the script VM,
and critically, from data marshaling – the process of converting and transferring data (like strings, numbers, or
complex game objects) between the distinct memory models and type systems of the native code and the scripting
environment (e.g., pushing data onto Lua's stack and converting return values back).

To further mitigate these performance bottlenecks, particularly for frequently executed code paths, **Just-In-Time (JIT)
compilation** techniques can be employed. Runtimes like LuaJIT dynamically compile performance-critical sections of
script bytecode into highly optimized native machine code during runtime. Forsslund's experiments highlighted that using
LuaJIT substantially improved the execution speed of Lua-based event handling, significantly closing the performance gap
compared to native C++ implementations (reducing overhead from ~6x slower towards a ~3x target). Similarly, Python-based
mod systems, seen in older titles like *Civilization*, could potentially leverage JIT compilers like PyPy. Nonetheless,
even sophisticated JIT compilation might not achieve parity with meticulously optimized, ahead-of-time compiled C++ code
for computationally intensive tasks. Consequently, game developers often design interpreted mod systems with
constraints, restricting scripts primarily to handling higher-level logic or responding to discrete events (e.g.,
triggered dialogues, turn completion in strategy games) rather than allowing them within performance-critical,
high-frequency loops like per-frame updates or physics calculations. For instance, strategy games like *Europa
Universalis IV* utilize interpreted scripts (using their domain-specific language) primarily for defining events and
decisions, parsing these text-based mod files upon game load.

#### Compiled Mods: Native or Bytecode Plugins

In contrast, the compiled mod model involves transforming mod code into binary formats that the game engine loads and
executes more directly. This approach manifests in several forms: mods compiled into the same native machine code as the
game engine itself (e.g., C or C++ mods compiled into DLLs on Windows or shared objects on Linux), or mods compiled into
an intermediate managed bytecode that a runtime environment integrated within the game (like the .NET CLR or Java
Virtual Machine) can execute (e.g., C# mods compiled to .NET Intermediate Language (IL) for games built with Unity or a
custom .NET engine, or Java mods compiled to .class files or JAR archives for Java-based games like *Minecraft*).
Regardless of the specific target format (native or bytecode), the defining characteristic is that the mod code is *not*
typically interpreted line-by-line by a game-specific scripting VM at runtime; instead, it executes as a more integral
part of the main program or within a powerful managed runtime.

![Comparison of mod integration approaches: interpreted scripting vs. compiled plugin](/images/b7eeae51-23a0-4541-9652-e0bdcb25ce10.png)

*Comparison of mod integration approaches: interpreted scripting vs. compiled plugin.*

The principal **advantages** of compiled mods are substantially higher **performance** and greater **power/flexibility
**. A mod compiled into a native plugin can, in principle, execute at speeds comparable to the original game code
because it operates at the same level, leveraging direct hardware access and optimized machine instructions. This
performance parity is crucial for ambitious mods aiming for significant gameplay alterations, such as total conversions,
complex new AI behaviors, or performance-intensive graphical enhancements, which might be computationally infeasible
within a slower scripting environment. Examples abound in the modding community: *Mount & Blade II: Bannerlord*, built
in C#, officially supports mods written in C# and compiled into DLLs loaded by the game. Similarly, *Minecraft*'s
extensive modding scene (via frameworks like Forge or Fabric) relies on mods written in Java and compiled into
bytecode (JAR files), while *Kerbal Space Program* (built on Unity/C#) uses compiled C# DLLs. This approach allows
modders to harness the full capabilities of the programming language, utilize extensive third-party libraries (subject
to the runtime environment), and interact more deeply with the game's internal systems via exposed APIs.

However, the compiled approach introduces significant challenges, particularly concerning **deployment, compatibility,
safety, and stability**. Since mods are compiled binaries, they often exhibit tight coupling with the specific version
of the game engine or its modding API they were built against. Game updates, even minor patches, can easily break
compiled mods if they alter the API contract, change internal data structures, or modify memory layouts (a common issue
with native C++ mods). This necessitates that modders potentially rebuild and re-release their mods after each game
update, increasing maintenance burden. Furthermore, the development process typically demands more sophisticated
tooling – including compilers, build systems, Integrated Development Environments (IDEs), and potentially official
Software Development Kits (SDKs) provided by the game developer. Creating a C# mod for *Bannerlord*, for example,
involves setting up a C# project, referencing specific game DLLs, writing code against the game's API, compiling it into
a DLL, and packaging it correctly – a considerably higher barrier to entry compared to editing a simple Lua script in a
text file.

Perhaps the most critical concern with compiled mods is **security and stability**. A compiled mod, especially a native
binary plugin, typically executes within the same process and memory space as the game itself, inheriting the same
privilege level. This means a buggy mod (e.g., containing a null pointer dereference, memory leak, or infinite loop) can
readily crash the entire game process, leading to instability for the end-user. More alarmingly, a maliciously crafted
compiled mod could potentially perform harmful actions on the user's system – reading sensitive files, writing or
deleting data, establishing network connections, or executing arbitrary code – essentially doing anything the user
running the game can do. By default, native compiled mods are often *not sandboxed*. While some argue that the
fundamental *detectability* of malicious intent might not inherently differ between compiled and interpreted code (as
both can be obfuscated), the practical reality is that the opaque, complex nature of compiled binaries often presents a
significantly greater challenge for security vetting, both for automated static/dynamic analysis tools and for manual
code review, compared to interpreted scripts where source code (or easily decompilable bytecode) might be more
accessible.

Recognizing these inherent dangers, developers and platform holders employ various **mitigation strategies**. **Code
signing**, where mod developers use digital certificates to sign their compiled binaries, can help establish
authenticity and accountability, though it doesn't guarantee safety. More robustly, **sandboxing** techniques aim to
execute mod code within a restricted environment, limiting its access to sensitive system resources, file system paths,
or network capabilities. However, implementing effective and performant sandboxing for native code plugins is
non-trivial. Static and dynamic code analysis tools can be integrated into modding platforms or launchers to scan for
known malicious patterns or suspicious behaviors, although sophisticated malware can evade detection. Ultimately, a
significant layer of defense relies on the vigilance of the **modding community** itself – platforms often depend on
users and experienced modders to vet mods, report suspicious activities, and build trust networks around reliable
creators. The explicit warning displayed by the *Bannerlord* launcher regarding the potential security risks associated
with loading community-created DLLs serves as a stark reminder of these dangers, underscoring the developer's awareness
and shifting some responsibility onto the user to exercise caution and only install mods from trusted sources.
Essentially, the choice for compiled mods often represents a trade-off: accepting increased security risks and
maintenance complexity in exchange for maximal performance and modding potential.

#### Hybrid Approaches and Future Directions

Seeking a balance between the performance of compiled code and the safety of interpreted scripts, hybrid or alternative
approaches exist and are areas of ongoing exploration. A notable middle ground involves using **managed bytecode
executed within a sandboxed virtual machine**. For instance, the *Roblox* platform allows user-generated content
including scripts written in Luau (a modified, sandboxed version of Lua), providing a controlled execution environment.
Historically, Java applets operated under a similar principle, running compiled Java bytecode within a security sandbox
in web browsers.

Looking forward, technologies like **WebAssembly (WASM)** present a compelling potential future for game modding. WASM
offers a binary instruction format designed as a portable compilation target, enabling near-native execution performance
across different platforms. Crucially, WASM runtimes are designed with security as a core principle, executing code
within a tightly controlled sandbox that explicitly defines and restricts access to host system functionalities.
Conceptually, allowing mods compiled to WASM modules could provide the speed benefits approaching native code while
retaining the strong security guarantees often associated with interpreted scripts. While not yet widely adopted in
mainstream game modding ecosystems, WASM is an area of active interest and research, potentially offering a way to
transcend the traditional dichotomy.

#### Conclusion: A Balancing Act

In summary, the decision between interpreted and compiled mod execution models involves a complex balancing act for game
developers. Interpreted systems generally offer greater ease of development, better cross-platform potential, and
simpler sandboxing capabilities, making them suitable for mods focused on content, configuration, or less
performance-critical logic. However, they typically incur a performance penalty. Compiled systems unlock significantly
higher performance and grant modders extensive power, enabling transformative and complex modifications, but they
introduce substantial challenges related to security, stability, compatibility, and development complexity. Many heavily
moddable PC games (*Skyrim*, *XCOM 2*, *Stardew Valley*, *Cities: Skylines*) effectively embrace or tolerate binary
mods, even if official support favors scripts, acknowledging the community's drive for deeper modification. Recognizing
this reality, some developers proactively design official compiled plugin systems (like *Bannerlord*'s) to provide a
supported pathway, contrasting with older approaches where binary modding was often an unofficial, less stable
endeavor (e.g., *Mount & Blade: Warband*). The choice profoundly impacts the entire modding ecosystem, shaping what
modders can create, how developers manage support and security, and ultimately, the end-user experience.

### Mod Loading and Game Lifecycle Integration

The integration of user-generated modifications, or "mods," represents a significant aspect of modern video game culture
and longevity. Understanding precisely **when and how mods are loaded and executed** is not merely a technical
curiosity; it is fundamental to ensuring that these additions integrate cleanly, function as intended, and coexist
harmoniously without disrupting the core game experience or conflicting destructively with one another. The process
involves a sophisticated orchestration of discovery, loading, initialization, and runtime execution, heavily dependent
on the game's underlying architecture and the specific modding support provided by its developers.

#### Mod Packaging, Discovery, and Metadata

The journey begins with how mods are packaged and discovered. Games with robust modding capabilities typically employ a
**module system**. A *module* serves as a self-contained package—often a distinct folder or archive file—containing all
the necessary assets, data files, and code for a specific mod. Crucially, each module usually includes a **metadata
descriptor**, such as a manifest file. This descriptor acts as an instruction manual for the game engine. For instance,
in *Mount & Blade II: Bannerlord*, each mod is treated as a "Module" defined by a `SubModule.xml` file. This file
explicitly informs the game about the components to load, which might include specific code assemblies (.dll files), new
graphical or audio assets, or custom game logic routines (subtasks).

When the game initializes (or sometimes when instructed to refresh its mod list), it systematically **scans designated
directories** (e.g., a `/Mods` folder within the game's installation) or utilizes official platform APIs, such as Steam
Workshop subscriptions, to **identify installed mod modules**. Once discovered, the game parses the metadata descriptor
for each mod to understand its requirements, dependencies, and the content it intends to introduce or modify.

#### The Loading Process: Timing and Content Integration

Following discovery and metadata parsing, the game proceeds to **load the actual content** of each enabled mod. This
loading process can vary significantly in its timing and methodology:

1. **Early Loading:** This is perhaps the most common approach, particularly for mods that introduce fundamental changes
   or need to be active from the very beginning of the game's execution. Mods are loaded during the initial startup
   sequence, before the main menu appears or a game world is loaded. *Bannerlord*'s reliance on `SubModule.xml` to
   direct the launcher which modules to load at startup is a prime example. Similarly, games like *Cities: Skylines*
   often compile C# mods during the initial game launch. This early integration is often necessary for modifications
   affecting core systems, adding new global mechanics, or altering the initial game setup.

2. **Runtime Loading:** Offering greater flexibility, runtime loading allows mods or specific pieces of mod content to
   be loaded (and sometimes unloaded) while the game is already running. This approach can significantly enhance the
   user experience by enabling features like joining modded multiplayer servers without pre-installing everything,
   downloading custom maps dynamically, or applying cosmetic changes on-the-fly, often without requiring a full game
   restart. Modern engines and frameworks facilitate this; for example, Unity engine games can leverage systems like
   Addressables, potentially combined with platforms like mod.io, to manage the delivery and integration of content
   during gameplay. However, implementing robust runtime loading is complex, as it requires carefully managing the
   injection and removal of assets and logic within an active simulation state.

3. **On-Demand Loading:** Representing a more granular approach, on-demand loading defers the loading of specific mod
   components until they are actually needed. This might occur when a player enters a specific area featuring modded
   content, interacts with a modded item, or triggers a particular event associated with the mod. The primary benefit is
   resource optimization – it avoids consuming memory and processing power for content that isn't currently relevant,
   potentially improving performance, especially in heavily modded scenarios.

Regardless of the timing strategy, the **content loading** itself involves integrating the mod's assets and code. For *
*data mods**, this typically means reading data files (e.g., XML, JSON, or proprietary formats defining new units,
items, quests, or dialogue) and merging this information into the game's internal databases or configuration structures.
Techniques vary: some games might simply add new entries, while others might allow mods to override existing entries
based on load order. For **asset mods**, loading involves incorporating new textures, models, sounds, or interface
elements, often leveraging a virtual file system where mod assets can override base game assets if they share the same
path or filename (as seen in Bethesda titles like *Skyrim* with `.bsa` archives and loose files). For **code mods** (
often scripts or compiled libraries like .NET DLLs), loading entails executing the script files or dynamically loading
the compiled assemblies into the game's process. *Kerbal Space Program*, for example, specifically looks for DLL files
in designated `Plugins/` directories within each mod's `GameData` folder and loads them using .NET reflection. Many
other Unity-based games follow similar patterns, sometimes unofficially leveraged by modders if the runtime's assembly
loading capabilities aren't locked down.

#### Initialization Phase: Setting the Stage

Simply loading a mod's code or data is often insufficient; an **initialization phase** is usually required to properly
integrate the mod into the game's systems *before* active gameplay begins. During this stage, mods typically register
their components with the game engine, set up necessary data structures, subscribe to events, or perform other essential
one-time setup tasks. Compiled mods might execute static constructors or have an explicitly defined `Init` method called
by the mod loader. Script mods might execute specific setup functions.

A highly structured example is found in *Minecraft Forge*, the popular modding framework for *Minecraft*. Forge defines
a detailed **mod lifecycle** with distinct event phases like *construction*, *preInitialization*, *initialization*, and
*postInitialization*. Mods use annotations to designate methods that should be called during each specific phase,
typically via an event bus system. This structured lifecycle ensures dependencies are met and registration occurs in a
predictable order – for example, ensuring all mods have registered their new items and blocks *before* any attempt is
made to load a world potentially containing them. Forge even parallelizes parts of this loading process across different
mods to reduce startup times for large modpacks, carefully managing synchronization between phases.

#### Runtime Execution: Triggering Mod Logic

Once loaded and initialized, mods need mechanisms to execute their logic during actual gameplay. This is achieved
through various **runtime hooks and triggering mechanisms** provided by the game or modding framework:

1. **Event-Driven Callbacks:** This is a very common pattern where the game engine defines and "fires" specific events
   corresponding to in-game occurrences (e.g., `OnGameStart`, `OnSaveLoaded`, `OnEntitySpawned`, `OnPlayerCraftedItem`,
   `OnButtonClicked`). Mods can register listener functions (callbacks) for these events. When an event occurs, the
   engine invokes all registered callbacks, often passing relevant contextual information (the event details).
   *Factorio*'s Lua scripting API heavily uses this via `script.on_event`, allowing mods to react precisely to actions
   like crafting or resource mining. Similarly, Paradox Interactive's games often use a system where mods define
   triggers based on game state conditions; the engine constantly evaluates these conditions and fires associated
   scripted effects when they are met. Forsslund's research highlights the efficacy of scripting languages like Lua in
   managing such event handling for mods.

2. **API Function Calls:** Modding APIs often expose a set of functions that mods can call directly to interact with the
   game engine. This allows mods to query game state, modify game data, spawn objects, or trigger specific game actions
   programmatically.

3. **Override Hooks:** Some frameworks allow mods to directly replace or augment core game functions. This might involve
   subclassing game classes and overriding virtual methods (if the game is designed with this extensibility) or using
   more advanced techniques. The Harmony library, widely used in the Unity C# modding scene (e.g., *RimWorld*, *Cities:
   Skylines*, *Bannerlord*), enables "patching" existing methods at runtime, allowing mods to run code *before* (
   prefix), *after* (postfix), or even entirely *instead of* (transpiler) the original game code. While powerful, this
   level of modification carries higher risks of conflicts and instability if not managed carefully.

4. **Configuration-Driven Logic:** In some systems, particularly data-driven games, mods can exert influence simply by
   providing configuration files. The game engine reads these files and interprets the defined rules or parameters to
   activate specific behaviors or modify existing ones. Paradox games extensively use this approach for defining events,
   decisions, national focuses, and more, allowing significant modification without complex coding.

5. **Periodic Execution (Tick/Update):** Certain modding systems allow mods to register functions that are executed
   repeatedly at regular intervals, such as every game frame (tick) or every simulated hour. This is necessary for mods
   that need continuous operation, like custom camera systems or ongoing background simulations. *The Sims 4*'s Python
   scripting support allows for such timed execution. However, developers must use this capability judiciously to avoid
   causing performance degradation.

6. **User Interface Interactions:** Mods frequently add new UI elements (buttons, windows, menus) or modify existing
   ones. User interaction with these modded UI components often serves as a direct trigger for the mod's functionality.

#### Managing Complexity: Conflicts, Dependencies, and Load Order

A critical challenge in any moddable game is managing the potential for **conflicts** when multiple mods attempt to
alter the same game data, assets, or code logic. The **load order** – the sequence in which mods are loaded and
initialized – becomes paramount. A common convention is that the mod loaded *last* takes precedence, effectively
overriding changes made by earlier mods affecting the same elements.

Game launchers or dedicated **mod managers** (like Vortex for Bethesda games and *Bannerlord*, or Irony Mod Manager for
Paradox titles) often provide users with tools to explicitly define this load order, helping to resolve conflicts or
achieve desired combinations of mod effects. Furthermore, mods frequently **depend on one another**. A mod might require
features or content added by another mod to function correctly. Good modding systems allow modders to declare these *
*dependencies** in their metadata (e.g., *Bannerlord*'s `SubModule.xml` allows specifying required modules). The mod
loader then uses this information to automatically sort the load order, ensuring dependencies are loaded before the mods
that require them, and warning the user if required dependencies are missing or incompatible. To minimize inadvertent
conflicts, mod developers often adopt best practices like using unique prefixes for their added identifiers (items,
functions, events) and preferring to add new content rather than directly overwriting vanilla game files where possible.

#### Robustness, Performance, and Conclusion

Finally, a well-designed mod loading system prioritizes **robustness and graceful error handling**. If a mod encounters
an error during loading, initialization, or runtime, the game should ideally catch the exception, log detailed
diagnostic information for the user and mod author, and potentially disable the faulty mod while allowing the game
itself to continue running, rather than crashing entirely.

Performance is also a key consideration. Loading dozens or hundreds of mods, especially large ones, can significantly
increase game startup times and impact runtime performance. Techniques like *Minecraft Forge*'s parallelized loading
phases and the careful use of on-demand loading or efficient runtime hooks are essential for maintaining a playable
experience in heavily modded environments.

In conclusion, the effective integration of mods hinges upon a carefully orchestrated lifecycle encompassing discovery,
timed loading (early, runtime, or on-demand), structured initialization, and a versatile set of runtime triggering
mechanisms. **Good design principles** for mod support emphasize clear points of entry and execution for mod code,
robust control over load order and concurrency, reliable dependency management, comprehensive error handling and
diagnostics, and careful consideration of performance impacts. Providing these capabilities allows developers to foster
vibrant modding communities that can significantly extend the life and appeal of their games, as will be further
explored through specific case studies of titles like *RimWorld* and *Minecraft*.

### Security and Sandboxing of Mods

The vibrant world of game modding, while offering unparalleled opportunities for creativity, customization, and extended
game life, introduces a significant and often underestimated security concern: the execution of **untrusted code**. By
enabling mod support, particularly through scripting or compiled binaries, game developers implicitly allow third-party
code, originating from potentially unknown or unverified sources, to run within the privileged context of their game's
process on the end-user's machine. This inherent vulnerability creates a pathway for potential misuse. In a worst-case
scenario, a maliciously crafted mod could exploit this access to perform a range of harmful actions, extending far
beyond simple game disruption; possibilities include corrupting save files, compromising user data, installing
persistent malware like keyloggers or ransomware, or incorporating the user's machine into a botnet. Even mods developed
with benign intent can inadvertently cause harm by introducing instability, creating resource contention leading to poor
performance, or conflicting with other mods or the game itself. Consequently, a critical consideration in the design and
implementation of any robust modding framework revolves around the strategy for isolating and controlling mod behavior –
specifically, how, and indeed whether, to effectively **sandbox** these external code components.

#### The Current Landscape: Trust, Community Curation, and Its Inherent Limitations

In contemporary practice, particularly within the sphere of single-player PC games, the primary approach to mod security
often leans heavily on non-technical mechanisms: **trust and community curation**. As summarized in a relevant GitHub
discussion concerning the risks of untrusted mod code, the prevailing attitude is that "Most games do not impose any
kind of security restrictions on mods and rely on community trust." This observation is empirically supported by the
modding ecosystems surrounding numerous popular titles. Games such as *The Elder Scrolls V: Skyrim*, *Minecraft* (Java
Edition), and *Mount & Blade II: Bannerlord*, for example, generally impose minimal technical restrictions on what mod
code, including compiled Dynamic Link Libraries (DLLs), is permitted to do. The underlying assumption is that users will
acquire mods from established, reputable sources – platforms like the Steam Workshop, Nexus Mods, or dedicated official
forums – which often feature community moderation, user ratings, and comments that can flag problematic content.
Furthermore, developers and platform holders may rely on users' existing antivirus software to act as a baseline
defense, potentially catching overtly malicious executables distributed within mod packages. Illustrating this,
following a concerning incident involving malware distributed via a *Cities: Skylines 2* mod, publisher Paradox
Interactive affirmed that mods uploaded to their platform undergo antivirus scanning as a precautionary measure.

However, this laissez-faire, trust-centric model is demonstrably not foolproof and carries inherent risks. History
provides sobering examples of malicious mods successfully circumventing community vigilance and platform screening. A
notable incident involved *Dota 2*, where mods exploited a vulnerability within the game engine itself (specifically,
the V8 JavaScript engine) to execute arbitrary code, effectively delivering a backdoor onto players' systems. This
highlights that threats may not only stem from intentionally malicious mod logic but also from exploiting underlying
engine weaknesses. In another instance, concerns were publicly raised within the *Mount & Blade II: Bannerlord*
community regarding the risks associated with allowing mods distributed as raw DLLs. Critics questioned the lack of a
safer, perhaps script-based, alternative, pointing out the potential for such mods to directly access process memory,
opening avenues for sophisticated malware that might evade simple detection. These incidents underscore the fragility of
reliance purely on trust and community oversight, emphasizing a growing need for more robust, technically enforced
sandboxing mechanisms, especially as modding becomes increasingly accessible and widespread, reaching less technically
sophisticated audiences.

#### Strategies for Sandboxing: A Spectrum of Technical Approaches

Recognizing the limitations of the trust model, developers and researchers have explored various technical strategies
for sandboxing mods, aiming to isolate them and mitigate potential harm. The core goal, analogous to the sandboxing
employed by web browsers to contain potentially malicious JavaScript, is to create a controlled environment that
restricts a mod's access to sensitive system resources and prevents it from interfering with the host game or operating
system in unauthorized ways. These strategies can be broadly categorized:

1. **Language-Level Restrictions (Scripting Languages):** When a game utilizes an embedded scripting language (e.g.,
   Lua, Python, JavaScript) for modding, developers gain a significant opportunity to implement fine-grained sandboxing.
   This is typically achieved by carefully **limiting the API exposed** to the mod scripts and controlling their
   execution environment. Lua, for instance, is well-suited for this, allowing the creation of sandboxed environments by
   manipulating its global table and selectively removing or replacing potentially dangerous standard library functions.
   A common practice is to disallow access to libraries like `io` (for arbitrary file input/output) and `os` (for
   executing operating system commands or interacting directly with the OS environment). Instead, the mod's interaction
   with the game world and system should be exclusively mediated through a carefully designed, game-specific API
   provided by the engine. *World of Warcraft*'s UI modding system serves as a prominent real-world example; Blizzard
   employs Lua but heavily restricts its capabilities. WoW mods cannot perform arbitrary file I/O or network calls; they
   are confined to interacting with the UI and gameplay functions exposed by Blizzard, with further restrictions applied
   contextually (e.g., blocking certain automated actions during combat to prevent cheating). This represents a
   *fine-grained sandboxing* approach tailored precisely to the game's requirements. Similar principles can be applied
   to Python, although its larger standard library and dynamic nature can make complete lockdown more challenging. A
   game could, for instance, launch a restricted Python interpreter with a limited set of available modules or
   experiment with removing built-in functions that could allow sandbox escapes. Godot Engine developers have also
   considered a "sandbox mode" specifically designed to restrict access to potentially dangerous OS-level functions from
   its GDScript language. The feasibility of such language-specific sandboxing, particularly for Lua, is further
   demonstrated in research exploring security solutions for BeamNG.drive modding.

2. **Operating System-Level Sandboxing:** Another category leverages features built directly into the host operating
   system to confine the mod's (or potentially the entire game's) execution. This approach treats the mod or game
   process as an untrusted entity and uses OS mechanisms to enforce boundaries. Technologies such as **containers** (
   e.g., Docker), mandatory access control frameworks like **AppArmor** or **SELinux** (common on Linux), or dedicated
   sandboxing APIs provided by the OS can restrict a process's privileges and limit its access to resources like the
   file system (e.g., only allowing writes to specific directories), network interfaces, inter-process communication
   channels, and hardware devices. Running mods within lightweight **virtual machines (VMs)** offers an even stronger
   degree of isolation, though often at a higher performance cost. A related concept involves sandboxing the *entire
   game* application. If a game were distributed through platforms enforcing stricter application models, such as
   Microsoft's Universal Windows Platform (UWP), it would operate within an OS-managed sandbox with inherently limited
   capabilities (e.g., restricted file system access). However, traditional PC games often require broad permissions for
   legitimate functions (saving files freely, interfacing with diverse hardware, interacting with overlay software),
   making wholesale adoption of heavily restrictive OS sandboxing models challenging and potentially detrimental to user
   experience or expected functionality.

3. **Software-Based and Engine-Integrated Sandboxing (Especially for Compiled Code):** Sandboxing **compiled mods** (
   e.g., C++ or C# DLLs) running within the same process as the game presents the most significant technical challenge.
   Unlike interpreted scripts, the behavior of compiled native code is much harder to analyze, predict, and control at
   runtime without substantial overhead or specialized runtime environments. Several techniques aim to address this:
    * **Process Isolation:** Inspired by modern web browser architecture (e.g., Chrome running tabs/extensions in
      separate, low-privilege processes), a game could theoretically run each compiled mod in its own isolated process.
      Communication between the mod process and the main game engine would occur via **Inter-Process Communication (IPC)
      ** mechanisms. While offering strong isolation, this approach necessitates a fundamental redesign of the game
      engine to manage shared state, handle IPC overhead, and essentially treat mods as microservices. The complexity
      and potential performance implications make this uncommon in current game development practice.
    * **Managed Runtimes with Security Features:** Platforms like .NET historically offered features such as **Code
      Access Security (CAS)**, which allowed developers to assign trust levels to assemblies and restrict their
      permissions (e.g., denying file I/O or network access). In theory, a .NET-based game (like those built with Unity,
      e.g., *RimWorld*, *Bannerlord*) could have loaded mod assemblies with partial trust. However, CAS is now largely
      deprecated, complex to manage correctly, and was never fully or reliably supported across all relevant platforms (
      like Mono, used by Unity). Consequently, it's rarely used for mod sandboxing today; mods loaded via frameworks
      like Harmony in Unity games typically run with the same full permissions as the game itself. The prevailing
      wisdom, echoed in technical discussions, is that securely hosting trusted and untrusted code *in the same process*
      is "really hard (bordering on impossible)" without robust, purpose-built runtime support.
    * **System Call Monitoring:** This technique involves intercepting and analyzing the system calls made by the mod's
      code at runtime. By examining requests for OS services (like opening files, initiating network connections, or
      accessing hardware), the game engine or a security layer could detect and potentially block attempts to access
      unauthorized resources or perform suspicious actions (e.g., connecting to unknown external servers).
    * **Memory Protection:** Employing hardware or software-based memory protection techniques, such as memory
      segmentation or page-level permissions, can help prevent mods from reading or writing memory outside their
      allocated regions, thus hindering attempts to tamper with the core game state or inject malicious code into other
      parts of the process.
    * **API Whitelisting and Static Analysis:** Instead of trying to block *bad* behavior, this approach focuses on
      allowing only *known good* behavior. It involves defining a limited, carefully curated API surface that mods are
      allowed to interact with. Access to any other internal engine functions or system libraries is denied by default.
      This can be coupled with **static analysis** of mod binaries before they are loaded. Tools like **Unbreakable**, a
      library developed for analyzing user-submitted .NET code for safety (used by SharpLab), work by maintaining a
      whitelist of permitted API calls and namespaces. Any assembly attempting to use disallowed APIs (e.g.,
      `System.IO`, `System.Net`) is rejected or flagged. A game developer could integrate such a tool to scan mod DLLs
      upon loading. While not a perfect runtime sandbox (determined attackers might use obfuscation, reflection, or
      other tricks to bypass static checks), it provides a significant improvement over blind trust. The main challenges
      include maintaining the whitelist, dealing with potential false positives/negatives, and deciding how to handle
      mods that fail the check (block entirely, or warn the user).
    * **User Consent and Granular Permissions:** A less stringent but potentially pragmatic approach involves **user
      prompts and explicit consent**. Similar to how mobile operating systems request permissions for apps (access
      contacts, location, etc.), a game could detect when a mod attempts a potentially sensitive action (e.g., writing a
      file outside the game's directories, making a network connection) and prompt the user for confirmation. This
      doesn't prevent malicious action if the user grants permission but raises awareness. A related idea is simply
      warning the user when loading a mod that hasn't been verified or uses potentially dangerous APIs (as identified by
      static analysis), shifting some responsibility to user caution. This constitutes a form of "user-consent sandbox."

#### Emerging Directions and Future Considerations: WASM and Custom Solutions

The ongoing challenge of securely integrating compiled mods has spurred interest in novel approaches. One particularly
promising direction is the use of **WebAssembly (WASM)** as a compilation target and runtime environment for mods. WASM
is an open standard designed from the ground up for executing untrusted code safely and efficiently, primarily within
web browsers but increasingly in other contexts. Mods could be written in various high-level languages (like C++, C#,
Rust) and compiled to WASM modules. The game engine would then host a WASM runtime (which itself is sandboxed), load the
mod module, and interact with it through a well-defined interface. By default, WASM code has no access to the host
system's capabilities (like file I/O or networking); it can only call functions explicitly provided by the host
environment (the game engine). This model offers a compelling combination of near-native performance (through JIT or AOT
compilation) and strong, built-in sandboxing. While challenges remain – such as designing rich and performant APIs
accessible from WASM, managing memory effectively between the host and WASM module, and tooling maturity – exploring
WASM-based modding frameworks represents a significant and potentially transformative research direction for enhancing
both safety and capability.

Beyond specific technologies, there is considerable potential in developing **custom sandboxing solutions** tailored
specifically to the unique requirements and common interaction patterns of game mods. Such solutions could potentially
leverage a deep understanding of typical mod functionalities to provide robust security without unduly hindering
legitimate creative expression. However, designing these custom solutions requires navigating a delicate **balance**:
the security mechanisms must be strong enough to mitigate real threats but flexible enough not to stifle modder
innovation. Overly restrictive sandboxes can easily frustrate modders and limit the scope of what can be achieved,
potentially undermining the very benefits modding brings. Furthermore, any sandboxing technique inevitably introduces
some **performance overhead**. This overhead must be carefully measured, managed, and minimized to ensure it does not
negatively impact the player's gaming experience, particularly in performance-sensitive real-time simulations.

#### Conclusion: Navigating the Path Towards Safer Modding

In summary, the prevailing approach to mod security in many single-player PC games prioritizes openness, performance,
and community trust over strict technical enforcement. While this has fostered vibrant modding communities, it leaves
users potentially exposed to risks from both malicious and unintentionally harmful code. The incidents involving malware
and exploits serve as stark reminders that trust alone is insufficient. Sandboxing techniques offer a path towards
mitigating these risks, with well-established and relatively feasible solutions available for scripted mods through
careful API design and environment control (as exemplified by WoW's Lua implementation or potentially restricted
environments in engines like Godot or Factorio).

The sandboxing of compiled binary mods remains a significantly harder problem within the traditional model of loading
them into the main game process. While OS-level sandboxing and true process isolation offer strong guarantees, they
often come with unacceptable complexity or performance trade-offs for typical game architectures. More pragmatic
approaches for compiled code include static analysis with API whitelisting (e.g., using tools conceptually similar to
Unbreakable), runtime monitoring (system calls, memory access), and user-centric measures like warnings and permission
prompts. Looking forward, emerging technologies like WebAssembly present a compelling opportunity to fundamentally
rethink mod execution, offering a potential pathway to achieve both strong security and high performance by design. The
contrasting approaches seen in games like Minecraft (unrestricted Java vs. sandboxed Bedrock Add-ons) or between
Paradox's controlled DSLs and Bannerlord's open DLL system highlight the diverse strategies currently employed and the
ongoing tension between flexibility, performance, and security. Ultimately, developing effective and balanced sandboxing
solutions is crucial for the future health and safety of the game modding ecosystem, ensuring that players can continue
to enjoy the creativity of modders with greater peace of mind.

## Case Studies of Modding Frameworks in Popular PC Games

To ground the discussion, we examine several games known for their mod support, each exemplifying different design
choices: (1) Paradox Interactive’s **grand strategy games** (e.g., *Europa Universalis*, *Crusader Kings* series), which
use a **proprietary DSL** and data-driven modding; (2) **RimWorld**, a storyteller simulation game built with Unity,
which supports **C# compiled mods** and extensive use of a patching library (Harmony); (3) **Mount & Blade II:
Bannerlord**, a sandbox RPG, which offers an official **module system with compiled .NET mods** and a rich engine API;
and (4) **Minecraft**, the sandbox block-building game, which despite initially lacking an official mod API developed a
robust **community-driven compiled mod ecosystem** in Java. Each illustrates different ends of the spectrum in language,
execution, and security.

### Case Study 1: Paradox Grand Strategy Games (Clausewitz Engine and Jomini)

Paradox’s grand strategy titles (such as *Europa Universalis IV*, *Crusader Kings III*, *Hearts of Iron IV*,
*Stellaris*, *Victoria 3*, etc.) are famous for their depth and also for being highly moddable. Paradox games follow a
consistent modding model rooted in their **Clausewitz engine**, with newer games incorporating a layer called **Jomini**
for shared functionality. Mods for these games are largely **data and script driven** using Paradox’s custom scripting
DSL, rather than writing arbitrary new code.

Mods for Paradox games are usually distributed as a set of text files (plus any assets) that override or add to the
game’s data. A mod is defined by a `.mod` file (a descriptor) that lists its contents and dependencies. The game’s
launcher loads these descriptors and merges the content at runtime.

The Clausewitz engine’s scripting is a prime example of a domain-specific language. It is *not Lua or Python*, but a
format unique to these games, optimized for describing game content. In CK2/CK3 or EU4, for example, mods can add: new *
*events** (random or triggered occurrences in game), **decisions** (options for the player), **focus trees** (in HOI4),
etc., all through plain text scripts. The language uses a key-value and nested-brace syntax that is fairly
human-readable. For instance, an event script might look like:

```plaintext
event = {
    id = mymod_flavor.1
    title = "New Festival"
    desc = "A new festival is held in the capital."
    trigger = {
        capital_scope = { prosperity > 50 }
    }
    option = {
        name = "Celebrate"
        effect = { treasury += -100 }
    }
    option = {
        name = "Decline"
        effect = { stability -= 1 }
    }
}
```

This pseudo-example shows the flavor: an `event` with an id, with conditions under `trigger` and two `option` choices
with different effects. The syntax is almost like writing a simple program, but it doesn’t allow arbitrary loops or
custom functions beyond what the engine defines. It’s a declarative style of scripting.

The Clausewitz engine parses these script files at game load. Internally, it likely transforms them into data structures
or C++ classes. As Forsslund (2013) described for EU3 (an earlier Clausewitz game), the trigger conditions form a tree
of evaluations in C++. Thus, when the game is running and needs to check if an event should fire, it’s running C++
code (or highly optimized checks) rather than interpreting text at that moment. This makes event execution fast, even if
thousands of events exist. The cost is front-loaded at load time (parsing).

The **Jomini** layer (introduced around *Imperator: Rome* and used in CK3, HOI4 updates, Vicky3) is meant to unify some
of this scripting functionality so that common features are easier to maintain across games. Jomini doesn’t replace the
scripting DSL but provides shared scripted systems (it’s named after a general, continuing Paradox’s tradition of naming
tech after historical figures like Clausewitz). For modders, Jomini might mean more consistency and possibly new
scripting capabilities across titles.

Importantly, Paradox modding does not allow modders to inject new binary code or call external libraries. Everything
must be done through the provided scripting hooks. This means modders cannot do things entirely outside the vision of
the original developers – they work within the sandbox of the DSL. If a desired feature isn’t exposed (say a modder
wants to write a new AI routine or complex algorithm), they cannot directly code it; they have to get creative with the
existing commands or hope Paradox expands the API in updates. This is a conscious design choice to keep things
deterministic (especially since some Paradox games have multiplayer sync to consider) and stable.

Paradox games typically load mods at game start. The user selects mods in the launcher, which then merges all the data.
There is often a load order mechanism (if two mods edit the same file, one will win out based on order). The games also
often allow total conversions which simply replace large data sets. Because it’s all data-driven, starting a game with
different mods essentially gives you a customized dataset for that session.

Once in-game, the *execution* of mod logic is event-based. Mods don’t “run” continuously; rather, the engine evaluates
triggers (like “on character action” or timed triggers like “mean_time_to_happen” in events) and then applies effects.
There isn’t, for example, a mod script that runs every frame – it’s all governed by game ticks and event polling. This
design is both for performance (no runaway mod scripts) and for keeping mod effects predictable.

As discussed, Paradox’s approach is inherently secure in the sense that modders cannot introduce arbitrary code. The
worst a mod can do is maybe create an event that intentionally ruins your game state (like sets all values to zero), but
it can’t access your disk or steal info. Recently, however, there was an incident of concern: a Cities: Skylines (the
original, not a Paradox-developed but Paradox-published title) mod was found with malware. That game (Cities: Skylines)
uses Unity and C#, and indeed that mod was a DLL with malicious code. Paradox responded by assuring users that on their
Paradox Mods platform, they virus-scan mods . For their in-house titles on Clausewitz, such an incident is unlikely
because those games don’t support DLL mods at all.

One challenge in Paradox’s model is that because mods can alter game behavior significantly (e.g., changing game rules),
if multiple mods conflict logically, the game might behave oddly. But that’s a design/content testing issue, not a
security one.

The modding community around Paradox games has created tools to handle the DSL (e.g., text editors with syntax
highlighting for Clausewitz script, merge tools). There are also de facto standards: for instance, using unique ID
namespaces to avoid event ID conflicts (mods prefix IDs with something like `mymod_` to not clash). Paradox provides
*Wikis* with documentation on the script terms (conditions, effects, etc.) and sometimes example mods.

This case study illustrates a **DSL-driven mod architecture**, prioritizing controlled extensibility over maximal
freedom. The benefits are stability and performance – even huge mods like total conversions run smoothly because they
mostly feed data into an engine built to handle it. The limitation is that modders can’t implement entirely new
mechanics that aren’t envisaged by the scripting language. In practice, Paradox continuously expands the script API
based on community feedback (for example, adding new trigger conditions or effects in patches to enable types of mods
that were previously impossible). This symbiosis works well for this genre and community, which is interested in
historical or fictional scenario overhauls, not so much in coding AI from scratch. The Forsslund thesis we cited earlier
directly came from Paradox’s investigation into using Lua; ultimately, they did not switch to Lua in their released
games, indicating they chose to stick with their performant DSL approach for now, likely because performance in their
large simulations is vital. It’s a case where a domain-specific solution has been refined over years to meet the needs
of modders within a certain scope.

### Case Study 2: *RimWorld* (Unity Engine with C# Mods and Harmony Patches)

*RimWorld* is a single-player sci-fi colony simulation game developed by Ludeon Studios. It’s built on Unity (written in
C#) and is known for an active modding scene. RimWorld’s modding approach is interesting because it straddles between
data-driven modding (for simple additions) and **full code modding in C#** for more complex changes. The game provides a
modding API and embraces the use of a community patching library called **Harmony** for altering game behavior.

RimWorld mods are typically organized as folders with a `About.xml` (metadata), any number of XML definition files, and
optionally a compiled assembly (`*.dll`) with C# code. The game has a Mods folder where users place mods (or they
subscribe via Steam Workshop and the game syncs them). The **load order** can be arranged by the player (some mods
require others to load first).

A lot of mod content in RimWorld is added via XML definitions. The game uses XML for defining things like items,
creatures (pawns), buildings, quests, etc. Modders can create new def files or patch existing ones via special XML patch
syntax. This is essentially a DSL approach for content: no code needed if you just want to add a new weapon or change a
tech tree. The XML files are processed at load, merging into the game’s databases.

For anything that goes beyond what XML can specify, modders can write C# code. RimWorld’s developers kindly released an
official modding API documentation (detailing classes and methods that mods can use or override). Mods that use code
will have a `.dll` built against RimWorld’s assembly references. When the game starts, it **loads all mod assemblies** (
likely via .NET `Assembly.Load` or Unity’s loading mechanisms). There is an entry point: if a mod assembly has a class
inheriting from `Verse.Mod` (a base class provided by RimWorld), the game will instantiate that class. This serves as a
mod’s initialization hook. In that class’s constructor, the mod can run any startup code (like reading settings,
applying Harmony patches, etc.).

Because RimWorld is written in C#, mods have essentially the same capability as the game’s own code – *there’s no
inherent sandbox*. Mods can access any public members of the game’s classes (and even private via reflection if they
want). The developers usually mark things `public` or provide hooks in expectation of mod use. For example, there are
virtual methods that mods can override by subclassing certain game classes.

One of the most significant aspects of RimWorld (and many Unity game mods) is the use of the **Harmony** library.
Harmony (by Andreas Pardeike) is a general library for .NET that allows runtime method patching – think of it as an
officially-sanctioned way to do what otherwise might be done by detouring or IL injection. With Harmony, a mod can
declare a patch on a method in the base game: you can prefix (run code before the original method), postfix (after it),
or even transpile (modify the IL of the method). This allows mods to change game behaviors without source code. For
instance, if the game has a method `GiveBirth` for colonists and a mod wants to add a chance of twins, the mod can
prefix patch `GiveBirth` to inject its logic (maybe calling the original method twice under certain conditions).
RimWorld modders heavily use this to alter core mechanics. The RimWorld developers explicitly permit and encourage this;
they include Harmony in their published game (some games even ship with Harmony to ensure all mods use the same
version).

Using Harmony, mods can effectively do **anything** in the game. This is immensely powerful – far beyond what Paradox
mods can do. Entire new systems (like multiplayer, which was added by a mod) or user interface overhauls have been
implemented by patching or adding to base functions. The downside is that it relies on the modders understanding the
game’s code. Also, there is a risk: if two mods patch the same method in incompatible ways, conflicts can occur (Harmony
does have some mechanisms to prioritize patches and detect conflicts, but it’s not foolproof).

C# code in mods runs at native speed (with JIT as usual for .NET). The presence of many Harmony patches does introduce
some overhead (because each original method call might trigger multiple prefix/postfix calls). In practice, this hasn’t
been a major problem for most mods, but poorly optimized mods can indeed slow down the game (for example, a mod that
runs heavy computations every tick). The responsibility is on mod authors to be mindful. RimWorld, being single-player,
allows the user to decide if the trade-off is worth it (e.g., they might accept a 10% slower game if the mod feature is
valuable).

The game loads mods before entering the main menu. It constructs each `Mod` subclass instance. Mods can also define
their own **settings** interface which the game will display in a Mods Settings menu automatically (via attributes in
code). During runtime, mods then operate as needed—some sit idle until a certain event (which they can get by patching
or by subscribing to update loops), others constantly run. There’s an update loop where Unity ticks each frame; mods
could, in theory, hook into that, but many game simulations in RimWorld happen on a tick system that mods can utilize (
e.g., a mod might patch the method that executes every in-game tick to attach its logic).

As noted, RimWorld mods run in-process with full permissions. There is *no sandbox*. A mod could delete your save files
or read your disk or send packets. Typically, RimWorld mods don’t do this (the community would spot it quickly).
Nonetheless, the game essentially trusts mods. If a mod crashes (throws an exception) and it’s not caught, it can bring
down the game or at least cause errors in the game’s state. RimWorld’s engine sometimes catches exceptions in mod code
and will display a red error message but continue running. For example, if a mod’s postfix on `GiveBirth` errors, the
game might catch it in a general try-catch around the event and just log it, allowing the game to continue (though the
mod’s effect might fail). Robust modding systems include lots of try-catch around mod hooks.

To illustrate, consider the **RimWorld Royalty DLC** integration: originally, certain features were only in the DLC, but
modders could check if the player has the DLC via the API and adapt. Mods often do something like:

```csharp
if (ModLister.HasActiveModWithName("Royalty")) {
    // use some Royalty-specific classes
}
```

The `ModLister` API is provided by the game to check loaded mods or DLCs . This shows how the game’s code itself offers
tools for mods to cooperate (e.g. not assume a DLC content exists unless DLC is there).

Another example: a mod might patch `IncidentWorker_Raid.TryExecute` (the function that spawns a raid event) to change
how raids work (maybe spawn friendly allies too). Using Harmony:

```csharp
[HarmonyPatch(typeof(IncidentWorker_Raid), "TryExecute")]
public static class Patch_RaidExecute {
    static void Prefix(ref IncidentParms parms) {
        // Modify the raid parameters before the base game executes it
        if (MyModSettings.doubleRaidSize) {
            parms.force = (int)(parms.force * 2);
        }
    }
}
```

This prefix runs before any raid is executed, and if the mod setting is toggled, it doubles the size of raids by
altering the parameter. The base game then continues, unaware that a mod tweaked its input. This kind of injection is
routine in RimWorld modding.

With great power comes great responsibility. The RimWorld community has developed conventions and libraries to manage
mod interactions. **HugsLib** is a library many mods use that provides common modding utilities (like a unified log,
update checking, etc.). The game’s community also uses a tool called **Harmony Debug** to inspect which mods patch what,
helping resolve conflicts when two mods try to patch the same thing differently.

The result is a vibrant mod ecosystem where you can transform RimWorld extensively – e.g., add multiplayer (the
“Zetrith’s Multiplayer” mod), change fundamental AI behaviors, introduce new UI screens, etc. The game effectively
becomes a platform. The developer (Tynan Sylvester) has embraced this, often incorporating popular mod ideas into
updates or allowing the mod scene to flourish without much interference.

This case shows a model almost opposite to Paradox’s: here modders can write actual program logic. The advantage is
creativity unbound – mods are only limited by code and imagination, not by developer-provided hooks alone. The
disadvantage is it requires programming skill and can lead to instability if mods misbehave. RimWorld’s solution is
community self-policing and good design of the base game to allow extension (exposing internal data structures,
providing an API, etc.). A statement from a developer on mod security encapsulated this approach: “If modding is
intrinsic to your game, script mods might be good for an accessible tier. Just know people will probably try to use
Harmony too unless you lock down the game” . RimWorld’s devs chose not to lock it down, thus mods freely use Harmony.

As an interesting note, in 2020 a RimWorld modder on GitHub asked about running untrusted code and one answer was
essentially: you can’t easily do that in Unity, better to go with the flow and just warn users . RimWorld implicitly
follows that philosophy: no sandbox, but also no known cases of malicious mods in its community. It relies on mod
distribution being relatively centralized (Steam Workshop and the official forum) where malicious actors would be
spotted.

### Case Study 3: *Mount & Blade II: Bannerlord* (Module System with .NET Plugins)

*Mount & Blade II: Bannerlord* is a medieval sandbox action-RPG with large battles, which released in Early Access in
2020 (full release 2022). It was eagerly anticipated not just for gameplay but also for its modding potential, given the
first Mount & Blade had a strong mod scene. Bannerlord’s developers provided official modding tools and documentation ,
and the game uses a **module-based mod system** running on .NET (specifically .NET Framework/Mono when it launched, now
possibly .NET Core). This case study highlights a designed-from-scratch mod support in a modern engine, with compiled
mods as the norm.

In Bannerlord, the base game itself is split into modules (each major aspect of the game is a module). Mods are
essentially additional modules. Each module (including the base ones and DLC) has a `SubModule.xml` which describes it .
Key information in this XML includes:

* Module ID, name, version.
* Dependencies on other modules.
* SubModule entries which can specify a `.DLL` to load, a type to instantiate for the module’s main class, and settings
  like whether it should load early or late.

When you launch Bannerlord, you use a launcher UI to select which modules (including mods) to enable. The game then
reads all `SubModule.xml` files. It respects dependencies and load order as per those files.

Bannerlord is built with a custom engine but much of the gameplay logic is in C# (the TaleWorlds library). They expose a
**C# API** for modders. Modders can reference the official TaleWorlds assemblies (like `TaleWorlds.CampaignSystem.dll`,
`TaleWorlds.Core.dll`, etc.) in their project to get access to game types such as `Hero`, `MobileParty`, `Campaign`.
Mods are compiled to DLLs targeting .NET. The `SubModule.xml` for a mod will specify the path to this DLL and the entry
point class. For example:

```xml

<SubModule>
	<Name>My Mod</Name>
	<DLLName>MyMod.dll</DLLName>
	<SubModuleClassType>MyModNamespace.MyModSubModule</SubModuleClassType>
	...
</SubModule>
```

This tells the engine to load `MyMod.dll` and instantiate `MyModNamespace.MyModSubModule` which presumably extends
`MBSubModuleBase` (an official base class). Indeed, Bannerlord’s mod API uses subclassing: a mod’s main class extends
`MBSubModuleBase` and overrides certain methods like `OnSubModuleLoad`, `OnGameStart`, `OnApplicationTick`, etc. The
engine calls these at appropriate
times  ([Getting Started | Bannerlord Documentation](https://docs.bannerlordmodding.com/_intro/getting-started.html#:~:text=)).
This is analogous to Unity’s MonoBehaviour or other game engines calling script methods on events.

Besides code, mods can include assets (like new models, textures) and data (XML files defining items, etc.). Bannerlord
uses XML for a lot of game data (similar to RimWorld, there are XMLs for defining items, etc.). Mods can provide their
own XMLs or patches to existing ones (the modding tools include an XML documentation). The `SubModule.xml` can list
additional XML resources the mod uses.

Since the game was built with modding in mind, TaleWorlds provided an official Editor (separate from the main game) to
create scenes, etc., and also an in-engine console for debugging. However, code mods typically require restarting the
game to test changes (though the developer console can reload some assets on the fly).

Bannerlord’s modding system does not sandbox mods. As discussed earlier, there was community concern about DLL mods
being a security risk . Indeed, Bannerlord mods have full access like any .NET code. A mod could use `System.IO.File` to
read/write files or connect to the internet. In practice, Bannerlord mods focus on game functionality (changing troop
stats, adding kingdoms, etc.), and malicious actions are not reported. The forum thread we saw mentioned a malware in
Cities:Skylines 2 (a different game) and warned that Bannerlord is similarly vulnerable due to DLL loading . As of
writing, TaleWorlds hasn’t added a scripting sandbox; they allow DLLs because the community expects that power for total
conversions (e.g., mods like “Realm of Thrones” which massively overhaul the game, or the “Bannerlord Online” mod which
adds multiplayer, all require deep code changes).

Running mods in C# has near-native performance (just normal .NET JIT). The risk is if a mod misbehaves. For example,
early in Bannerlord’s modding, many mods had to be updated whenever the game updated, because internal API changes would
break them (leading to crashes if a mod called a removed function). TaleWorlds tried to improve mod stability by marking
some APIs as stable. Still, compiled mods faced the classic problem: version compatibility. They eventually introduced
an official *Mod Compatibility Loader* that would allow older mods to run on newer versions if possible, or at least not
crash outright but disable if incompatible. Modders also often compile against the specific game version and note the
required version.

The Bannerlord engine calls the mod’s `MBSubModuleBase` methods at key points:

* `OnSubModuleLoad` – when the module is first loaded (even before the main menu).
* `OnBeforeInitialModuleScreenSetAsRoot` – before the main menu appears (some mods load UI here).
* `OnGameStart`, `OnGameEnd` – when a campaign or battle starts/ends.
* `OnMissionBehaviorInitialize` – each battle (mission) can have mod behaviors.
* `OnApplicationTick` – each frame tick (if needed).
* etc.

This structure encourages mods to hook into the life cycle rather than run arbitrarily. But mods can also subscribe to
game events or override classes by replacing the game’s class instances with their own (the engine often uses factories
or dependency injection that mods can influence).

A Bannerlord mod that, say, increases the party speed on roads might override a method in the pathfinding system, or
simpler: the game might have a data variable for road speed bonus which the mod’s XML adjusts. If not available in XML,
the mod could patch via code:

```csharp
public class MyModSubModule : MBSubModuleBase {
    protected override void OnGameStart(Game game, IGameStarter gameStarter) {
        base.OnGameStart(game, gameStarter);
        if (game.GameType is Campaign) {
            // Register a campaign behavior
            ((CampaignGameStarter)gameStarter).AddBehavior(new RoadSpeedBehavior);
        }
    }
}
public class RoadSpeedBehavior : CampaignBehaviorBase {
    public override void RegisterEvents{
        CampaignEvents.DailyTickEvent.AddNonSerializedListener(this, OnDailyTick);
    }
    private void OnDailyTick{
        foreach(var party in MobileParty.All) {
            if (party.IsOnRoad) {
                party.SpeedMultiplier *= 1.1f;
            }
        }
    }
    public override void SyncData(IDataStore dataStore) { } // for saving data, not needed here
}
```

This hypothetical example uses the game’s provided *CampaignEvents* to run code daily that boosts party speeds on roads.
The mod inserted a new *Behavior* into the campaign. Bannerlord’s API is built to allow adding such behaviors
dynamically. This illustrates a structured way to extend gameplay via provided hooks (AddBehavior).

Bannerlord’s approach is somewhat more managed than RimWorld’s free-for-all Harmony patching. Because the devs provided
explicit virtual methods and event subscriptions (as in the example), mods can often achieve goals without IL patching.
Harmony can still be used for Bannerlord (and indeed some mods might use it if needed), but a lot can be done with
official APIs. This reduces conflicts – multiple mods can add separate behaviors without clashing, whereas in RimWorld
if two mods patch the same method, there’s potential conflict.

Bannerlord modders share a community documentation repository and TaleWorlds actively updates the official API
docs ([Bannerlord Modding Documentation - bannerlord.com](<http://moddocs.bannerlord.com/#:~:text=Welcome> to the
Mount%26Blade II%3A,To access the “Frequently)). The developer modding forums are used for Q&A. Because the game is
complex, many mods are large projects themselves (some even on GitHub). The presence of an official modding toolkit (for
scene editing etc.) signals TaleWorlds’ commitment.

As of Nov 2024, after the cited forum post, there’s no indication Bannerlord switched to a scripting language only. The
suggestion in the forum “Why not use Lua or proprietary language?” likely stems from exactly the debate of this thesis –
trading mod power for security. TaleWorlds opted for power (compiled C# mods) to satisfy the hardcore mod community, at
the cost of potential security vulnerabilities. They possibly rely on platform scanning (Steam Workshop scanning mods,
etc.) and community moderation.

### Case Study 4: *Minecraft* (Java Edition Modding via Community Frameworks)

Minecraft’s case is unique because official mod support was absent in early years, and the community created its own
modding framework (Forge, and more recently Fabric/Quilt) to fill the gap. We include it because it represents one of
the most extensive mod ecosystems in PC gaming and highlights compiled mod injection in a Java environment. While
Minecraft is not *designed* for mods (originally), it has effectively become a platform due to these frameworks.

Forge (specifically for Minecraft: Java Edition) works by modifying the game’s Java bytecode to insert hooks and load
external mods. Essentially, Forge is a customized version of the Minecraft game engine that, at startup, will load
additional JAR files (mods) and incorporate them. In technical terms, Forge used to distribute a modified
`minecraft.jar` that included an event bus and a loader. Mods would be written in Java against an API that Forge
provided (which mimicked or wrapped internal game code). When launching, Forge does the following:

* It **decompiles** or uses an already deobfuscated version of the Minecraft code to apply patches (called “coremods”
  for low-level changes).
* It loads mod JARs and uses a custom classloader to inject them such that when Minecraft classes call certain methods,
  the mod code can run.
* It provides lifecycle events: mods have methods annotated for phases like preInit, init, postInit. Forge calls these
  in order for all mods .
* It has an event bus where mods can listen for game events (like a block broken event, a tick event, etc.).

For example, a simple Forge mod’s main class might look like:

```java
@Mod(modid="mymod", name="My Mod", version="1.0")
public class MyMod {
    @Mod.EventHandler
    public void preInit(FMLPreInitializationEvent event) {
        // Register items, blocks
        GameRegistry.register(new ItemMagicWand, ...);
    }
    @Mod.EventHandler
    public void init(FMLInitializationEvent event) {
        // Register event listeners
        MinecraftForge.EVENT_BUS.register(new MyEventHandler);
    }
}
```

Forge would construct `MyMod` and call these methods at the appropriate times. The mod author uses Forge’s
`GameRegistry` and `MinecraftForge.EVENT_BUS` to interact with the game. Underneath, these interact with Minecraft’s
code (e.g., adding an item to the item list).

Fabric is a newer, lightweight mod loader that avoids heavy patching. Instead, it uses the Mixins library to inject mod
code by modifying class bytecode at load time. It’s similar in concept but more decentralized; each mod can include
mixin definitions for which methods to modify. Fabric provides mappings and a lightweight API, but compared to Forge it
has less built-in abstraction – mods often directly call into deobfuscated Minecraft code through mappings.

Whether Forge or Fabric, mods are compiled Java code that run in the same JVM as Minecraft. They have full access to
Java’s capabilities. Mods often come with dependencies (shared libraries like “Minecraft Forge” or other mod libraries)
packaged in. The mod loading is handled by the loader (Forge or Fabric) which sets up the classpath.

There is no sandbox. Mods can do anything Java can (which is a lot, though the user must be running the game, so it
inherits user permissions). Java’s security manager could theoretically restrict mods, but it’s not used (and is now
removed in Java 17+ which Minecraft uses). In practice, just like other cases, trust is placed in mod sources (
CurseForge, Modrinth, etc.) to vet them. There have been malicious mods or plugins (especially on the server side with
Bukkit plugins in multiplayer), but the community usually reacts quickly.

Java mods run with near native speed (JIT optimized by the JVM). Sometimes having many mods can increase memory usage or
slow things due to extra logic, but the Minecraft community is accustomed to that and often the limiting factor is
Minecraft’s own single-threaded nature rather than mods.

Forge delineates mod phases to ensure order. For example, “preInit” is where mods register new blocks and items – by the
end of preInit, Forge has a list of all new items from all mods, so it can assign them IDs. “init” is when mods might
register recipes or inter-mod interactions. “postInit” is for any cross-mod adjustment after everything’s loaded. This
structured pipeline ensures that by the time the player is in the world, all mods have done setup. Forge also
parallelized mod loading to speed it up for large mod packs , since big packs can have hundreds of mods.

Minecraft (via Forge) has events like `PlayerInteractEvent` or `TickEvent` to which mods can subscribe (similar to how
Factorio mods use events ). This decouples mods from needing to patch every piece of code – instead of altering the
player interaction function, a mod can just listen for the interact event and execute alongside others.

One notable issue: a few years ago, a vulnerability in Log4j (a logging library included in Minecraft) made headlines (
“Log4Shell”). Malicious users could exploit it via chat messages on servers. While not directly a mod issue, it
highlighted that mods including libraries can introduce vulnerabilities. Also, some mods themselves had security holes.

Mojang (developers of Minecraft) did not officially support modding for the Java edition beyond providing an
*obfuscation mapping* for the community in later years. They focused on providing a **Data Pack** system (which allows
adding some content with JSON, and limited scripting with their command blocks) for light modding, and a separate
modding system for Bedrock (which uses sandboxed scripts in JSON and limited logic). But the Java modding via
Forge/Fabric remains unofficial yet extremely popular. It demonstrates how a community can implement a modding framework
that deals with all the concerns we’ve discussed: it basically built the scaffolding for mods to integrate (like load
order, event hooks, etc.) externally.

There are countless examples – from “Optifine” (optimizes graphics, adds features) to huge game-changing mods like
“IndustrialCraft” (adds machinery, power systems). These often add entirely new subsystems that interact with base game
mechanics. For example, IndustrialCraft adds electricity; it achieves this by introducing new block types and then
hooking into the game tick to update those blocks’ energy networks.

The community developed tools like ModLaunchers, debuggers, and integrated development environment (IDE) support with
mappings that let modders write code as if they had Minecraft’s source (because the community provides deobfuscation
mappings for each version).

It shows the *upper bound* of mod complexity – effectively, mods can rewrite the game. But it also shows the cost:
modders need to deal with game updates breaking things (since Mojang doesn’t keep an API stable for them), and
everything runs on trust. The success of Minecraft modding arguably influenced Microsoft/Mojang to incorporate safer
add-on systems in their other edition. But for PC single-player, the Java edition remains a paradise for modders willing
to dig into code or use frameworks created by others.

------

These case studies illustrate the spectrum from controlled, data-driven modding (Paradox) to open, code-driven modding (
RimWorld, Bannerlord, Minecraft). Each game’s approach was shaped by its engine technology, developer philosophy, and
community needs. Despite differences, common themes emerge: **providing clear entry points** for mods, handling mod
loading and dependencies, and balancing ease of use with power. Next, we discuss some forward-looking ideas inspired by
these examples and challenges.

## Future Directions and Research Proposals in Modding Frameworks

The state of mod support in single-player PC games has advanced greatly, but there remain open questions and
opportunities for improvement. From a computer science and systems architecture perspective, modding frameworks can
borrow ideas from programming language design, security engineering, and software product lines. In this section, we
propose and discuss some novel directions and research ideas for the future of modding technology:

**1. Safe but Performant Mod Sandboxing (WebAssembly or VM-based Mods):** One promising idea is to leverage *
*WebAssembly (WASM)** as a universal mod runtime. WebAssembly is designed to run code securely in a sandbox, with
near-native performance and a portable format. A game could define a mod API and supply modders with bindings in
multiple languages that compile to WASM (C++, Rust, even AssemblyScript). Mods would then be delivered as WASM modules.
The game’s engine would include a WASM runtime (which are increasingly efficient) to execute mod code. This offers
several benefits:

* *Security:* WASM bytecode is sandboxed by design – it cannot access anything not explicitly given by the host. The
  game could expose specific functions (for game data access) to the module and nothing more, effectively containing the
  mod’s capabilities. Memory safety is also guaranteed.
* *Multi-language support:* Modders could code in their preferred language (as long as it can compile to WASM) instead
  of being tied to whatever the game’s engine language is.
* *Performance:* WASM is faster than interpreted scripts and continues to improve. While there is overhead to calling
  between the host (game) and the WASM module, this could be optimized, and for heavy logic within the mod, the WASM JIT
  would handle it efficiently.

A research challenge here would be designing a **mod API schema** that can be expressed in a language-agnostic way. One
might define an IDL (Interface Definition Language) for mod<->game communication (similar to how Web APIs are defined
for JavaScript), then auto-generate binding code for both the game and mod side in various languages. Another challenge
is debugging and error reporting: developers would need tools to debug WASM mod code effectively (source maps could
help).

Some early explorations of WASM for game scripting exist, but applying it to user mods specifically is a ripe area for
development. It could combine the best of scripting (safe sandbox) and native mods (speed, capability).

**2. Static Analysis and Verification of Mods:** Taking inspiration from formal methods, one could apply static analysis
to mod code to ensure certain properties. For example, a static analyzer could scan a mod to guarantee it doesn’t call
prohibited APIs or doesn’t have obvious infinite loops that could hang the game. The earlier-mentioned **Unbreakable**
tool for .NET is a step in this direction (whitelisting safe calls) . A more advanced approach could use model checking
or symbolic execution on mod scripts to ensure they don’t violate certain conditions (like not altering memory they
shouldn’t). While full verification is hard (mods can be arbitrary code), targeted checks could catch common issues –
e.g., ensure a mod’s event handler executes within X milliseconds by analyzing its complexity (perhaps using abstract
interpretation).

Another angle is **capability-based security**: where mod code is analyzed for capabilities (file access, network
access, etc.) and is only allowed to run if it conforms to a policy. This could be integrated into mod distribution
platforms: for instance, the mod repository could display “This mod requests permission to access the file system” and
the game could enforce that or prompt the user.

**3. Unified Modding Layer / Standards:** Currently, each game reinvents mod support in its own way. A radical idea is
to develop a **standardized modding framework or middleware** that many games can adopt, especially those using common
engines. For example, an extension to Unity or Unreal that provides a built-in mod loader, scripting VM, and sandbox.
Unity already allows scripting in C#, but it doesn’t provide an out-of-the-box way to load external user scripts in a
built game. If engine makers provided a toggle for “moddability” that packages a mini runtime for user scripts and a
secure loader, it could simplify the work for each game studio (much like how Unity’s package system works, but for
untrusted packages). The challenge is that each game’s needs differ, so this would need to be highly customizable.
Perhaps a core library that handles the grunt work (loading mods, versioning, sandboxing) with hooks for game-specific
integration.

Academic work on plugin architectures (outside games) could inform this. For instance, OSGi for Java (a dynamic module
system) or MEF for .NET were about loading modules at runtime with controlled isolation. Adapting such systems to
high-performance game loops (maybe with real-time constraints) would be an interesting systems engineering problem.

**4. In-Game Mod Development Environments:** Another idea is to make mod development more accessible by providing
in-game or live development tools. Some modern games (like *Roblox* or *Dreams* on PlayStation) are essentially game
creation environments themselves. For traditional PC games, one could allow a mode where the game runs in a special
“editor” state and mods can be written and injected on the fly. This would entail an in-game code editor or a connection
to an external IDE that can hot-reload mod scripts. Hot reloading greatly speeds up iteration (some engines like Unreal
support hot reload for C++ during dev, but not exposed to end-user mods). There is research on live programming and
could it apply to mods? Potentially, yes: imagine a running game where you tweak a mod’s script and see the changes
immediately (with state preserved if possible). This would blur line between user and developer even more, but it aligns
with how mods essentially turn some players into co-developers.

**5. Modularity and Inter-Mod Communication:** As mod ecosystems grow, mods often need to interoperate. We see this with
Minecraft (where dozens of mods form a “modpack” and sometimes depend on each other) and Paradox mods (which might need
compatibility patches). A structured approach is to encourage **modular design**: smaller mods that do one thing and
expose APIs for other mods. Some frameworks already do this (e.g., Terraria’s tModLoader has a way for mods to call each
other’s methods). A formal research direction could be to define **mod composition languages** or dependency injection
mechanisms where mods can declare facets that others can extend. For instance, if Mod A adds a new resource type, Mod B
can find that at runtime and incorporate it without hardcoding to mod A – a bit like how browser extensions sometimes
detect and augment each other.

One concept could be a **mod event bus** not just for game events, but for mod events: mods broadcasting events to other
mods in a decoupled way. This would help large mod collections coordinate (like a mod that improves UI could announce “I
have altered the inventory UI” and other mods that add UI elements know to plug into the new one instead of the
original).

**6. Cloud-Based Mod Verification and Distribution:** We touched on scanning mods for malware; a further step is if the
game’s publisher provides a cloud service to run mods in a sandboxed environment (like a virtual machine) to observe
behavior before distributing. This is similar to how app stores vet apps. For instance, an automated system could load a
mod, simulate some game events, and monitor if the mod tries to perform disallowed operations. With advancements in
containerization, one could spin up an instrumented game server that runs the mod’s code in a controlled way to log
system calls. While heavy-weight, for certain platforms this could ensure safety. Possibly overkill for single-player
PC (where user ultimately consents to mods, unlike multi where server owners have to trust mods).

**7. AI-Assisted Modding:** Looking further ahead, artificial intelligence might assist both mod creation and
verification. AI could help modders write code (less a framework issue, more a tooling one), but also help analyze mod
interactions. For example, using machine learning to predict mod conflicts based on how they modify the game’s data or
to cluster mods by compatibility. Or AI bots could play a modded game to test stability and balance.

**8. Content and Behavior Separation:** Encourage designs where mod content (data) is separated from behavior (code)
with clear interfaces. For example, a game could let mods add new content purely via data (safe) and only allow code
when absolutely needed. Some modern games (like *Kenshi* or *Mount & Blade: Warband*) allow extensive modding purely
through data files and simple scripts, reserving code injection only for advanced mods (via community hooks). A
formalization of this could be a *tiered modding* system: Tier 1 mods can only use declarative changes (no code) – these
could even be allowed in multiplayer safely or on consoles; Tier 2 mods can include code but must use an approved API
subset; Tier 3 are full native code (with all the risks). The game or platform can then, for example, allow Tier 1 mods
freely, require warning for Tier 2, and disallow Tier 3 in certain contexts. This layered approach provides flexibility.
Forsslund hinted at something similar by suggesting using embedded scripts for prototyping and then converting them to
native for production for performance– analogous, one can imagine distributing a mod in a high-level form for safety and
optionally having a compiled plugin for those who want the extra features/performance.

In summary, future research and development in game modding can focus on making mods **safer, more interoperable, and
easier to create** without sacrificing the power that mod communities thrive on. The balance between **usability, speed,
and security** is a classic software engineering triad that here manifests as DSL vs language choice, interpreted vs
compiled, sandboxed vs free. Techniques from systems security (like sandboxing and capability control) and from
programming languages (like intermediate representations and dynamic loading) will be key to advancing mod frameworks.
Ideally, we will reach a point where players can enjoy rich mods with the confidence that their system is safe and
developers can integrate mod support with less fear of chaos or cheating. Bridging that gap is an exciting challenge
where academic insight and practical game development needs intersect.

## Conclusion

Supporting user-created mods in single-player PC games is a multifaceted challenge that lies at the intersection of
software architecture, programming language design, and security engineering. Game developers have explored a spectrum
of solutions, from controlled domain-specific scripting languages to full-fledged embedded programming environments.
This thesis reviewed those solutions and their implications in depth.

We saw that using **embedded scripting languages** (like Lua, Python, or C# in managed engines) can dramatically lower
the barrier for modders and increase flexibility, at the cost of runtime overhead and potential security exposure. Oskar
Forsslund’s empirical evaluation in *Europa Universalis III* quantified this trade-off: embedding Lua made mod scripting
easier and more extensible, but incurred roughly a six-fold slowdown in event execution compared to native C++. Such
results underscore that the choice of modding technology must align with a game’s performance constraints. Games with
heavy simulation might lean toward **DSLs and native integration** for speed – as Paradox’s Clausewitz engine does,
parsing game-specific scripts into efficient C++ logic. On the other hand, games prioritizing mod creativity and total
conversions (e.g. *Minecraft*, *RimWorld*) effectively hand the keys to modders by allowing direct code injection,
whether through official APIs or community loaders. This maximizes what mods can do, enabling thriving ecosystems of
content, but as we discussed, comes with minimal internal safeguards.

A recurring theme is that **developer support and tooling** can make or break a mod community. Games like *Mount & Blade
II: Bannerlord* demonstrate the value of providing official documentation and a structured module
system ([Getting Started | Bannerlord Documentation](https://docs.bannerlordmodding.com/_intro/getting-started.html#:~:text=)) ,
which channel modding efforts in a stable way. Meanwhile, the absence of official support in *Minecraft: Java Edition*
did not prevent modding but did push the community to develop complex frameworks (Forge, Fabric) to fill the void. These
frameworks essentially reverse-engineered mod support, implementing mod loading phases, event buses, and dependency
management . The success of such efforts suggests that certain principles – modularity, clear lifecycle events, and open
APIs – are somewhat universal in modding.

**Sandboxing and security** remain areas in need of improvement, as evidenced by incidents of malicious mods or
community concern . Our analysis indicates that most single-player games currently rely on social and platform
measures (curation, antivirus scanning ) rather than technical sandboxes to mitigate these risks. This is an
understandable pragmatism given the difficulty of isolating code in-process without major engine overhead . Yet, as mods
become more mainstream (even consoles now allow some mod-like content via curated stores), it will be increasingly
important to explore the future directions we proposed: using technologies like WebAssembly for a safer execution layer,
or static analysis to vet mods before they run. The ideal scenario is one where a user can download a mod without a
second thought about malware or system damage, akin to how mobile app stores have made users comfortable installing
third-party apps. Achieving that level of trust in PC mods will likely require industry collaboration on standards and
perhaps integration of OS-level sandbox features into game platforms.

From a **software architecture** standpoint, supporting mods essentially means designing a game as a *platform* rather
than a closed product. This involves additional upfront effort: defining extension points, maintaining stability across
game updates (so mods don’t all break), and sometimes accepting limits on optimization (to keep systems generic enough
for mods). The case studies illustrated differing approaches to this platform mindset. Paradox allows extensive modding
but confines it to the boundaries of what their engine was designed to simulate, thus preserving integrity. Ludeon
Studios (RimWorld) opened up almost everything, arguably turning debugging and compatibility over to the community’s
hands (which, fortunately, rose to the occasion with tools like Harmony). Neither approach is “wrong” – they reflect
design priorities and resource considerations.

We also highlighted **inter-mod cooperation** challenges. As mods accumulate, the chance of conflicts rises – one mod’s
change might inadvertently override another’s. Mechanisms like load order and dependency declarations are basic
solutions. A more advanced solution we discussed is establishing explicit mod interfaces or events for mods to
communicate, which can reduce direct conflicts. The modding scene has organically developed practices (like community
libraries and shared frameworks) to address these issues, but future mod frameworks could bake in support for such
patterns (e.g., a mod messaging system).

In conclusion, the technologies for mod support in single-player PC games have evolved significantly, but there is room
for further innovation. Developers must navigate the triad of **usability, performance, and security** when choosing how
to empower modders. The optimal solution is context-dependent: a heavily simulation-driven game might accept a custom
DSL with limited scope (emphasizing performance and safety), whereas an open-ended sandbox game might embed a full
programming environment (emphasizing flexibility). Importantly, as Forsslund’s work suggested, these choices can be
revisited over a game’s life cycle – one could imagine a hybrid approach where a game starts with easy script modding
during its active development, then moves to more optimized native mod integration for longevity.

The enduring popularity of mods and the creativity seen in communities serve as strong evidence that investing in mod
support yields dividends in player engagement and game longevity. As such, this is an area where academia and industry
can fruitfully collaborate. Academic research can propose new models for safe extensibility (as in other software
domains), and game developers can trial these in the real world, feeding back insights. With the rise of digital
distribution and platforms like Steam Workshop, delivering mods to players is easier than ever; now we must ensure
running those mods is seamless and secure. By combining robust architectural design, modern sandboxing techniques, and
supportive tools for mod developers, future games can usher in a new era of modding – one where players can truly
customize every aspect of their experience without concern, and where user innovations continue to push the boundaries
of what games can achieve.

**References**

* Forsslund, O. (2013). *Evaluating Lua for Use in Computer Game Event Handling*. Master’s Thesis, KTH Royal Institute
  of Technology, Stockholm.
* Scacchi, W. (2011). Modding as a Basis for Developing Game Systems. In *Proceedings of GAS’11 (Game Adaptation and
  Solvers Workshop)*, Honolulu, HI .
* PathogenDavid. (2020, Oct 26). Answer to “Running untrusted code (video game modding)”. *GitHub Roslyn Discussions*  .
* “Modding - Crusader Kings II Wiki.” (n.d.). *Paradox Wikis*. Retrieved from CK2 Wiki .
* “Mount & Blade II: Bannerlord Modding Documentation.” (2020). *TaleWorlds
  Entertainment* ([Getting Started | Bannerlord Documentation](https://docs.bannerlordmodding.com/_intro/getting-started.html#:~:text=)) .
* Vijayan, J. (2023, Feb 10). Malicious Game Mods Target Dota 2 Game Users. *Dark Reading* .
* Paradox Interactive Forums. (2024, Nov 4). *“Concerning about security vulnerability of Bannerlord modding”* (Forum
  thread) .
* A Weird Imagination. (2024, June 23). *A newbie’s introduction to Factorio modding* (Blog post) .
* Factorio Community Wiki. (2023). “Stages of Modding – Lifecycle Events.” Retrieved from Forge documentation .
* Paradox Interactive. (2023, Oct 31). *“Additional information regarding malware suspicion on mod ...”* (Forum post
  snippet) .
* Ludeon Studios. (2018). *RimWorld Modding Tutorial – Part 1: Basics*. (Forum and documentation references) .
* Reddit - r/paradoxplaza. (2017). *“Why is Paradox’s modding syntax so weird?”* (Discussion referencing Clausewitz
  DSL) .
* GitHub - Bannerlord Modding Community. (2022). *Bannerlord Documentation (community
  maintained)* ([Getting Started | Bannerlord Documentation](https://docs.bannerlordmodding.com/_intro/getting-started.html#:~:text=)).
* Check Point Research. (2020). *Gaming Engines: An Undetected Playground for Malware Loaders* (re: Godot/GDScript
  threat) .
* The Creative Assembly. (2019). *Total War: Warhammer Modding FAQ*. (Example of game with both script and data mods –
  reference, if needed).
* Factorio Wiki. (n.d.). “Events and script interface in Factorio mods.” *Factorio.com* .
