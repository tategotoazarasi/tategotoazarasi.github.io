---
date: '2025-03-27T17:33:26+08:00'
draft: false
summary: "Discover the tech behind single-player PC game mods, from scripting languages to security and future trends like WebAssembly."
title: 'Under the Hood: The Technologies Powering Your Favorite Game Mods'
tags: [ "game-modding","single-player-games","pc-gaming","modding-frameworks","scripting-languages","domain-specific-languages","lua","python","c-sharp","interpreted-scripts","compiled-mods","mod-apis","sandboxing","security","game-architecture","extensibility","mod-lifecycle","event-handling","paradox-games","rimworld","mount-and-blade-bannerlord","minecraft","software-engineering","user-created-content","performance-trade-offs","modding-technology","game-development","scripting-vs-compiled","mod-security","embedded-languages","dsl-design","game-extensibility","sandbox-strategies","case-studies","future-research","modding-tools","api-design","community-mods","gaming-platforms","code-execution" ]
---

If you've spent any time in the PC gaming world, you know mods. They're the lifeblood that keeps games fresh years after
release, the sparks of creativity that turn a fun experience into a personalized obsession, and sometimes, the
foundation for entirely new genres. Think about it: *Counter-Strike* started as a *Half-Life* mod, *Dota* emerged from
*Warcraft III*, and countless other innovations bubbled up from player communities tinkering with the games they
loved [1]. Modding isn't just about adding silly hats (though that's important too!); it's a powerful form of user
engagement, creativity, and even end-user software engineering [1].

But here's the thing: enabling this creative chaos isn't magic. It requires deliberate, often complex, technical
decisions from the game developers themselves. While some hardcore modders might reverse-engineer games without official
help, the most vibrant modding scenes often flourish where developers have intentionally built *support* for mods right
into the game's architecture.

Supporting mods properly is *hard*. Developers have to grapple with fundamental questions:

* What kind of tools or languages should modders use? Should it be something familiar like Python or Lua, or a custom
  language built just for the game?
* How will the mod code actually *run*? Will it be interpreted on the fly, or compiled into faster (but potentially
  riskier) binary code?
* How do we let mods interact with the game without crashing everything or, worse, opening up security holes?
* How do we balance giving modders enough power to be creative against maintaining game stability and performance?

This isn't just a technical puzzle; it's a strategic one that shapes the entire modding community around a game. Get it
right, and you foster decades of player loyalty and innovation (think *Skyrim* or *Minecraft*). Get it wrong, and you
might stifle creativity or end up with a buggy, insecure mess.

In this (rather extensive) post, we're going to unpack the technologies developers use on *their* side to make modding
possible for single-player PC games.

Our focus here is strictly on the *developer-side* technology for *single-player PC games*. Multiplayer introduces a
whole other layer of complexity (anti-cheat, synchronization, server hosting) that's beyond our scope today. Console
modding is also a different beast due to platform restrictions and certification processes.

So, let's pop the hood and see what makes modding tick.

## The Foundations: What Makes a Game Moddable?

Before we even talk about specific languages or execution methods, we need to understand that supporting mods isn't
usually an afterthought bolted onto a finished game. Truly moddable games are often designed with *extensibility* in
mind from the ground up.

### Designing for Extensibility

This idea isn't unique to games. Back in the day, software engineering pioneers like David Parnas talked about designing
software for change and extension [1]. The core principle is building systems in a modular way, with well-defined
interfaces between components, so that you can swap parts out or add new ones without breaking the whole thing.

Walt Scacchi, who has written extensively on game modding, framed it as a form of "open software extension" [1]. Games
that do this well often employ architectural patterns like:

#### Modular Design

Breaking the game down into distinct systems (AI, physics, UI, gameplay logic) that communicate through clear
interfaces. This makes it easier to expose specific parts to modders without revealing the entire messy internals. Think
of how Unreal Engine uses modules, which can facilitate plugin-based modding [2].

#### Data-Driven Architecture

Instead of hardcoding game rules, content, or parameters directly into the compiled code, developers store this
information in external files (like XML, JSON, YAML, or custom formats). The game engine reads these files at runtime to
configure itself. This is *huge* for modding because it means players can change significant aspects of the game simply
by editing or adding these data files, without needing to write any code. We'll see a great example of this with
*RimWorld*.

#### Software Product Lines

Some researchers view moddable games as instances of software product lines, where the base game is the core platform
and mods represent variations or features added to that platform [1]. Designing with this mindset encourages developers
to think about commonality and variability, isolating the core engine from the customizable game content.

### Data-Driven vs. Code-Driven Modding

This leads to a fundamental distinction in *how* mods interact with the game:

#### Data-Driven Modding

Mods primarily consist of data files that the game engine reads and interprets. This could be adding new items by
defining them in an XML file, creating new quests via configuration files, or adjusting weapon stats in a
spreadsheet-like format. The power here comes from the engine being designed to *read* this external data. Paradox games
are masters of this, allowing huge swathes of game logic and content to be defined in text files [3].

#### Code-Driven Modding

Mods include actual executable code (scripts or compiled binaries) that runs alongside the game's own code. This code
typically uses an Application Programming Interface (API) exposed by the game engine to query game state, react to
events, or modify game behavior. This allows for much deeper, more complex changes, like implementing entirely new
gameplay systems or altering core AI. Games like *RimWorld* (C# mods) [4] or *Mount & Blade II: Bannerlord* (C#
mods) [5] heavily rely on this.

Many games actually use a *hybrid* approach. They might allow simple content additions via data files (easy entry point
for beginners) and provide a scripting or code API for more advanced modders. *RimWorld* is a perfect example: add new
guns via XML, write complex AI behaviors in C# [4].

### The Modding API: The Gateway

Whether data-driven or code-driven, the core mechanism enabling mods is some form of interface provided by the
developers. This could be:

#### File Formats and Loaders

For data mods, the "API" is the specification of the data file formats (e.g., the structure of the XML definitions in
*RimWorld*) and the engine's ability to find and load these files from mod directories.

#### Scripting Hooks and Bindings

For script mods, the API consists of the functions and objects the scripting language can access to interact with the
game world (e.g., the Lua functions exposed by *Factorio* to let mods react to events like `on_player_crafted_item` [6].

#### Code Libraries and Interfaces

For compiled mods, the API is often a set of libraries (like DLLs or JARs) that mods link against, providing classes and
methods to interact with the engine (e.g., the `TaleWorlds.*.dll` assemblies that *Bannerlord* modders reference [5].

Designing a good modding API is an art. It needs to be:

#### Powerful enough

Expose enough functionality to allow meaningful mods.

#### Stable enough

Avoid changing constantly, which breaks existing mods with every game update.

#### Safe enough

Not expose functions that could easily crash the game or compromise security.

#### Documented enough

Modders need to know what's available and how to use it! (Though community reverse-engineering often fills gaps, as seen
in *RimWorld* [7].

With these foundational concepts in mind, let's dive into one of the biggest decisions developers face: what language
should mods be written in?

## Choosing the Right Tools: Languages for Modders

Okay, so you've decided to let players mod your game with code or scripts. Now what? Do you embed a popular language
like Lua or Python? Or do you invent your own special language just for your game? This choice has massive ripple
effects on performance, flexibility, security, and the kind of modding community that grows around your game.

### The Big Question: General-Purpose Language vs. Domain-Specific Language (DSL)?

This is the core dilemma. Let's break down the options.

#### Option 1: Embed a General-Purpose Language

This involves integrating an existing, off-the-shelf programming language interpreter or runtime into your game engine.
Modders then write their logic in that language.

##### Lua

The reigning champion of embedded game scripting. Why? It's small, fast (for a script), designed explicitly for
embedding in C/C++ applications, and relatively easy to learn [8]. Countless games use it, from *World of Warcraft*'s UI
mods to *Factorio*'s extensive modding system [9] to experiments in Paradox games [8]. Its C API makes it
straightforward (relatively speaking) to bridge the gap between the game's native code and the Lua scripts.

##### Python

Another popular choice, known for its readability and vast libraries. *Civilization IV* famously used Python for a lot
of its game logic scripting, allowing extensive modding. The downside? Python interpreters are generally heavier than
Lua's, and performance can be a concern for high-frequency tasks in games. Integration also requires careful handling of
the Global Interpreter Lock (GIL) if multithreading is involved.

##### C#

Particularly relevant for games built with the Unity engine. Since Unity itself uses C# for scripting, it's natural for
developers to expose parts of their C# codebase for modding. *RimWorld* [4], *Cities: Skylines* [10], and *Mount & Blade
II: Bannerlord* [5] all allow mods written as compiled C# assemblies (.DLLs). This provides immense power and
performance (thanks to the .NET JIT compiler) but comes with compilation hurdles and security headaches (more on that
later).

##### JavaScript

While primarily a web language, its ubiquity and mature engines (like V8) have led to its use in some game contexts,
especially for UI scripting. Valve's Source 2 engine, for example, uses JavaScript (specifically Panorama UI) for UI
elements in games like *Dota 2* [11]. Security is a major consideration here, as browser engines have incredibly
sophisticated sandboxes built over decades – something game engines usually lack.

##### Pros of General-Purpose Languages

###### Familiarity

Modders can leverage existing programming skills. The learning curve is lower if they already know Lua, Python, or C#.

###### Power & Expressiveness

These are real programming languages! Modders can implement complex algorithms, data structures, and logic that might be
impossible in a more limited system.

###### Ecosystem

Access to existing libraries (though often restricted for security), documentation, and development tools (debuggers,
IDEs).

##### Cons of General-Purpose Languages

###### Integration Complexity

Developers need to embed the interpreter/runtime, create bindings (the C/C++ API bridge), and carefully manage data
transfer between the game and the script environment. This binding layer can be tricky to get right and maintain.

###### Performance Overhead

Scripting is almost always slower than native compiled code. Oskar Forsslund's Master's thesis provides a stark example:
evaluating *Europa Universalis III*'s event triggers in Lua was roughly **six times slower** than the original C++
implementation that parsed custom script files [8]. The overhead comes from interpretation/JIT compilation itself,
but also significantly from the "context switching" and data marshaling required to call script functions from C++ and
vice-versa [8].

###### Security Risks

These languages are powerful. Unless carefully sandboxed, a mod script could potentially access the file system (
`io.open` in Lua), make network connections, or execute arbitrary OS commands (`os.execute` in Lua). Sandboxing requires
deliberately restricting the available functions and libraries, which takes effort.

###### API Surface

Developers need to decide *what* parts of the game to expose to the scripting language. They are constantly balancing
giving modders enough power versus overwhelming them or exposing internals that shouldn't be touched. As one developer
noted in a discussion, you're "on the hook for anticipating their needs" [12].

##### Mitigating Performance:

Just-In-Time (JIT) compilers can help bridge the performance gap. Forsslund found that using LuaJIT significantly sped
up the Lua event handling in his EU3 tests, getting closer to the performance target Paradox had set (though still
slower than native C++) [8]. However, even with a JIT, frequent calls across the native/script boundary can remain
costly. Developers often design around this by triggering scripts only for higher-level events rather than inside tight
loops that run every frame.

#### Option 2: Create a Domain-Specific Language (DSL)

Instead of embedding an existing language, some developers create their *own* custom language tailored specifically for
modding their game. These DSLs are often not full-fledged programming languages but rather specialized formats for
configuring game behavior or defining content.

##### Paradox Interactive's Clausewitz Scripting

This is the quintessential example. Games like *Europa Universalis IV*, *Crusader Kings III*, *Hearts of Iron IV*,
*Stellaris*, etc., use a proprietary scripting syntax (often in `.txt` files, despite sometimes involving Lua for
specific functions internally) to define almost everything: events, decisions, national focuses, technologies, AI
behavior, map data, etc. [3] [13]. It uses a nested key-value structure with braces, looking somewhat declarative.

Here's a *hypothetical* snippet resembling a Paradox event script:

```plaintext
# Example Event Definition in a Paradox-style DSL
country_event = {
    id = my_awesome_mod.101
    title = "MY_MOD_EVENT_101_TITLE" # Loc key
    desc = "MY_MOD_EVENT_101_DESC"  # Loc key

    picture = GFX_event_mymod_picture

    is_triggered_only = yes # Means it won't fire randomly

    trigger = {
        # Conditions for the event to be possible
        has_dlc = "My Awesome DLC" # Example check
        government = democracy
        NOT = { has_country_modifier = recently_had_event_101 }
    }

    mean_time_to_happen = {
        months = 120 # Average time for it to fire if conditions met
        modifier = {
            factor = 0.8 # Faster if...
            has_idea_group = economic_ideas
        }
    }

    immediate = {
        # Effects that happen instantly when the event fires
        add_stability = -1
        add_country_modifier = {
            name = "recently_had_event_101"
            duration = 3650 # 10 years
        }
    }

    option = {
        # First choice for the player
        name = "MY_MOD_EVENT_101_OPT_A" # Loc key
        ai_chance = { factor = 70 }
        add_treasury = 100
        add_prestige = 5
    }

    option = {
        # Second choice
        name = "MY_MOD_EVENT_101_OPT_B" # Loc key
        ai_chance = { factor = 30 }
        add_inflation = 2.0
    }
}
```

Notice how it's structured around game concepts (`trigger`, `mean_time_to_happen`, `option`, `effect` like
`add_stability`) rather than generic programming constructs like `if/else` or `for` loops (though the engine evaluates
these triggers logically).

##### Bethesda's Papyrus

Used in games like *Skyrim* and *Fallout 4*, Papyrus *looks* more like a traditional scripting language but is still a
DSL created specifically for Bethesda's Creation Engine. It's event-driven and designed for attaching scripts to objects
in the game world. While more powerful than Paradox's declarative style, it's still limited compared to a
general-purpose language (e.g., no direct file I/O for mods).

##### Pros of DSLs

###### Performance

DSLs can be heavily optimized. The game engine parses the DSL script (often at load time) and converts it into an
internal representation that can be executed very efficiently in native code (like C++). Forsslund's study confirmed
this: the original EU3 event system using parsed DSL scripts was *significantly faster* than his Lua prototype [8].
Paradox likely stuck with their DSL approach precisely because performance is critical in their complex simulations.

###### Safety by Design

Because the DSL only includes commands relevant to the game, it's inherently sandboxed. There's simply no syntax in
Paradox script for "delete file" or "connect to internet." The language itself restricts mods to interacting with the
game through predefined, controlled mechanisms (like specific triggers and effects). Security comes from *limited
expressiveness*.

###### Ease of Use (Potentially)

For non-programmers, a well-designed DSL that uses game-specific terminology might be easier to grasp than learning a
full programming language. Adding a new country event in Paradox script might feel more intuitive than writing
equivalent logic in Lua or C#.

###### Enforces Game Structure

DSLs can guide modders into creating content that fits the game's intended structure and rules.

##### Cons of DSLs

###### Learning Curve

Every DSL is unique. Modders have to learn a new, often proprietary, syntax and vocabulary for each game (or game
engine). Documentation might be sparse or community-driven.

###### Limited Power

DSLs are, by definition, domain-specific. Modders might hit a wall if they want to implement something complex or novel
that the DSL wasn't designed for. You can't easily write a new pathfinding algorithm or a complex economic simulation
using only Paradox event script commands. Modders sometimes have to resort to clever workarounds or request new features
from the developers.

###### Developer Burden

The game developers have to design, implement, document, and *maintain* the DSL and its parser/interpreter. If modders
need new functionality, developers might have to add new keywords or commands to the language itself, which can be
time-consuming.

#### Hybrid Approaches

Some games try to get the best of both worlds. They might use a simple data format or DSL for common tasks (like
defining items or basic quests) and embed a general-purpose language (like Lua) for more complex scripting needs (like
custom AI behaviors or intricate quest logic). This offers an easier entry point while still providing power for
advanced users. Forsslund even mused about using Lua for prototyping event logic due to its flexibility and then
potentially translating it back to the faster DSL format for release, though this seems rare in practice [8].

#### So, Which to Choose?

The decision often boils down to the game's specific needs and the developer's philosophy:

* If **performance** is absolutely critical (e.g., complex simulations running thousands of checks per second) and the
  types of mods expected are mostly content-focused within predictable boundaries, a **DSL** might be the better
  choice (Paradox).
* If **flexibility** and empowering modders to create truly novel systems is the priority, and the performance overhead
  is acceptable (or can be managed), an **embedded general-purpose language** is often preferred (*Factorio* with Lua,
  *RimWorld* with C#).
* If the game is built in an engine like Unity or uses .NET, allowing mods in the **same language (C#)** becomes a
  natural, powerful option, effectively turning the game's own codebase/API into the "language" for mods (*RimWorld*,
  *Bannerlord*).

Now that we've considered the *language* mods are written in, let's look at *how* that code actually gets executed by
the game.

## How Mods Run: Interpreted vs. Compiled Execution

This is another fundamental fork in the road for modding architecture. Does the game run mod code directly from source
files (or intermediate bytecode) at runtime, or does it load pre-compiled binary files (like DLLs)? This choice deeply
impacts performance, security, and the mod development workflow.

Let's visualize the difference conceptually (imagine a diagram here, as generating one directly is tricky):

### Interpreted Mod

1. Modder writes `MyMod.lua` (source code).
2. Player installs `MyMod.lua`.
3. Game starts, loads `MyMod.lua`.
4. Game embeds a Lua Virtual Machine (VM).
5. When needed, the Game Engine tells the Lua VM: "Run this function from `MyMod.lua`."
6. The Lua VM interprets (or JITs) the Lua code and executes it, calling back into the Game Engine API for game
   data/actions.

### Compiled Mod

1. Modder writes `MyMod.cs` (source code).
2. Modder uses a C# compiler (like `csc` or via Visual Studio) to build `MyMod.dll` (binary code).
3. Player installs `MyMod.dll`.
4. Game starts, uses the operating system or .NET runtime to load `MyMod.dll` directly into its own process memory.
5. When needed, the Game Engine directly calls functions within `MyMod.dll` (like methods in a C# class).
6. The code inside `MyMod.dll` runs as native (or JIT-compiled) code, calling back into the Game Engine API.

Now let's unpack the implications.

### Interpreted Mods (Runtime Scripting)

In this model, the mod code isn't native machine instructions when the player installs it. The game itself contains the
necessary machinery (an interpreter or a VM) to execute the mod scripts on the fly.

#### How it Works

The game loads the script files (e.g., `.lua`, `.py`). It might parse them line-by-line (very slow, rare nowadays) or,
more commonly, compile them into an intermediate bytecode format first (Lua does this automatically). Then, an
interpreter executes this bytecode. Sometimes, a Just-In-Time (JIT) compiler might even translate frequently used parts
of the bytecode into native machine code at runtime for better speed.

#### Examples

Games using Lua (like *Factorio*), Python (*Civ IV*), or potentially JavaScript. Even Paradox's DSLs are essentially
interpreted – the engine parses the `.txt` files and executes the logic they represent.

#### Pros of Interpreted Mods

##### Easier Development

Modders often just need a text editor. There's no separate compilation step. They can make changes and (sometimes) see
results quickly, leading to faster iteration.

##### Cross-Platform Compatibility

A Lua script mod will generally work identically on Windows, macOS, or Linux versions of the game, as long as the game
itself includes the Lua interpreter for each platform. The mod code is platform-agnostic.

##### Easier Sandboxing

As we discussed, the interpreter acts as a natural choke point. The game developer can control the environment the
interpreter provides to the script, removing dangerous functions or libraries (like file I/O or network access). This
makes sandboxing *much* more feasible than with compiled code.

#### Cons of Interpreted Mods

##### Performance

This is the big one. Even with bytecode compilation and JITs, interpreted code is generally slower than fully
pre-compiled native code. We saw Forsslund's 6x slowdown figure for Lua vs C++ [8]. The overhead comes from the
interpretation/JIT process itself and the cost of crossing the boundary between the native game engine and the scripting
VM (data marshaling, function call setup) [8]. This might be perfectly acceptable if mods only run occasionally for
non-critical tasks, but prohibitive if mods need to run complex logic every frame.

##### Requires Embedding Runtime

The game developer needs to integrate and ship the language interpreter/VM with the game.

### Compiled Mods (Native or Bytecode Plugins)

Here, mods are distributed as binary files (like `.dll` on Windows, `.so` on Linux, or `.jar` containing Java bytecode)
that the game loads directly into its process space.

#### How it Works

The game uses the operating system's dynamic library loading mechanism (like `LoadLibrary` on Windows) or a runtime
environment's assembly loading feature (like .NET's `Assembly.Load` or the Java Virtual Machine's classloader) to load
the mod's binary file. The code in the mod then runs essentially as part of the game itself.

#### Examples

*Mount & Blade II: Bannerlord* (C# DLLs loaded via .NET) [5], *RimWorld* (C# DLLs loaded via Unity/.NET), *Minecraft* (
Java JARs loaded by Forge/Fabric via the JVM), *Kerbal Space Program* (C# DLLs). Many games using engines like Unity or
Unreal might implicitly support compiled mods if modders can figure out how to get their compiled assemblies loaded,
even without official sanction.

#### Pros of Compiled Mods

##### Performance

This is the main advantage. Compiled code runs at or near native speed. A C# mod in *Bannerlord* or *RimWorld* executes
much like the game's own C# code. A C++ mod compiled to a DLL would run just as fast as the engine's C++. This enables
incredibly complex mods – total conversions, new physics systems, sophisticated AI – that might be computationally
infeasible with slower scripting languages.

##### Power & Flexibility

Modders typically get access to the full power of the language the mod is written in (C++, C#, Java). They can use
complex language features, interact more deeply with the engine's API (if exposed), and potentially link against
external libraries (though this adds complexity and risk).

#### Cons of Compiled Mods

##### Development Complexity

Modders need a proper development environment: a compiler, potentially the game's specific SDK or header files/library
references. The workflow involves writing code, compiling, packaging, and then testing in-game. This is a higher barrier
to entry than editing a script file.

##### Compatibility Issues

Compiled mods are often tightly coupled to a specific version of the game or engine API. When the game updates, internal
changes (like function signatures changing, classes being refactored, memory layouts shifting in C++) can easily break
compiled mods, requiring the mod author to update and recompile. This is a constant headache in communities like
*Minecraft* or *Bannerlord*.

##### Platform Dependence

A DLL compiled for Windows won't work on Linux or macOS. While managed runtimes like .NET and Java offer better
cross-platform potential (compile to intermediate bytecode), native C/C++ mods are inherently platform-specific.

##### Security Nightmare

This is the biggest drawback. By default, a compiled mod loaded into the game's process has the *exact same permissions*
as the game itself. If the game is running as the user, the mod can do *anything* the user can do: read/write arbitrary
files, connect to the internet, launch other processes, install malware, delete `system32` (okay, maybe not that easily,
but you get the idea). There is **no inherent sandbox**. We'll talk more about security later, but compiled mods
basically operate on trust [12].

##### Stability Risks

A bug in a compiled mod (like a null pointer dereference, an unhandled exception, or an infinite loop) can easily crash
the entire game process, not just the mod's operation.

### A Potential Middle Ground: Managed Runtimes and Future Tech

It's worth noting that compiled mods in managed languages like C# or Java occupy a slightly different space than native
C++ DLLs. The runtime environment (CLR for .NET, JVM for Java) does provide *some* layer of abstraction and safety (
e.g., memory safety, garbage collection). Historically, these runtimes also had security managers or code access
security systems (like .NET CAS) designed to run untrusted code with limited permissions [12]. However, these features
are largely deprecated or considered ineffective/too complex for robust sandboxing in modern versions, especially within
a single process [12]. So, in practice, C# mods in *RimWorld* or *Bannerlord* still run with full trust.

Looking ahead, technologies like **WebAssembly (WASM)** offer a tantalizing possibility. WASM is a binary instruction
format designed to be a portable compilation target for high-level languages, enabling deployment on the web for client
and server applications. Crucially, it's designed to run *safely* in a sandboxed environment, with near-native
performance. Could future games allow mods compiled to WASM? It might offer the speed benefits of compiled code with the
security benefits of a sandbox. This is an active area of interest we'll revisit in the "Future Directions" section.

### Summary Table: Interpreted vs. Compiled

| Feature            | Interpreted Mods (e.g., Lua, Python, DSLs)               | Compiled Mods (e.g., C# DLLs, Java JARs, C++ DLLs)           |
|--------------------|----------------------------------------------------------|--------------------------------------------------------------|
| **Performance**    | Generally Slower (VM overhead, boundary crossing)        | Generally Faster (Native or near-native speed)               |
| **Dev Ease**       | Easier (Text editor, no compile step, faster iteration)  | Harder (Compiler, SDK, build process needed)                 |
| **Flexibility**    | Limited by exposed API & language features               | High (Full language power, potentially external libraries)   |
| **Compatibility**  | Often better survives game updates (if API stable)       | Often breaks with game updates (tight coupling)              |
| **Cross-Platform** | Good (Script runs on any platform with game+VM)          | Poor (Native DLLs), Better (Managed bytecode like .NET/Java) |
| **Security**       | Easier to Sandbox (Control VM environment, limit API)    | Very Hard to Sandbox (Runs with full game permissions)       |
| **Stability**      | Errors might be caught by VM (less likely to crash game) | Errors can easily crash the entire game process              |

Developers must weigh these factors carefully. If they want to enable deep, complex mods and trust their community (or
implement external security measures), compiled mods offer the most power. If they prioritize accessibility, safety, and
easier maintenance, interpreted scripts are often the way to go. Many successful modding scenes exist at both ends of
this spectrum.

Now, assuming we've chosen a language and execution model, how does the game actually *find*, *load*, and *run* these
mods?

## Bringing Mods to Life: Loading and Lifecycle Integration

Okay, so players have downloaded some mods. How does the game actually know they exist, load their content and code, and
make sure they run at the right moments without stepping on each other's toes? This involves designing a robust mod
loading system and integrating mods into the game's lifecycle.

### Finding and Identifying Mods

First, the game needs to locate installed mods. Common strategies include:

#### Dedicated `Mods` Folder

The simplest approach. The game looks inside a specific folder (e.g., `My Documents/MyGame/Mods/` or
`<GameInstall>/Mods/`) for subfolders or archive files representing individual mods.

#### Launcher Manifests

Some games use a launcher application that manages mods. The launcher might maintain a list or configuration file
specifying which mods are enabled and where they are located. *Bannerlord*'s launcher does this, reading module
information before starting the game proper [5].

#### Platform Integration

Increasingly common is integration with platforms like Steam Workshop. Players subscribe to mods on Workshop, and the
Steam client downloads them to a specific location. The game then uses the Steam API to find and load subscribed mods.

Once found, each mod usually needs a **descriptor file** (sometimes called a manifest). This file contains metadata
about the mod, such as:

* Unique ID, Name, Author, Version
* Description
* Dependencies (other mods it requires)
* Load order hints
* Entry points (e.g., the main script file to run, or the DLL and class name to load).

*Bannerlord*'s `SubModule.xml` is a prime example of such a descriptor, containing all this information [5].
*RimWorld* uses an `About.xml` file for basic metadata [4].

### Loading Mods: Order Matters!

The game (or its launcher/mod manager) reads these descriptors, decides which mods are active, and then proceeds to load
them. This typically happens during game startup, before the main menu appears.

A critical aspect here is **load order**. If two mods modify the same game asset or piece of logic, which one "wins"?
Most systems adopt a "last loaded wins" rule. Mod A changes weapon damage to 10, Mod B loads later and changes it to
15 – the final damage will be 15.

This makes the *order* in which mods are loaded crucial for compatibility. Many games allow players to manually set the
load order through a mod manager UI (common in Bethesda games, Paradox games via launchers). The mod descriptor file
might also specify dependencies (e.g., "MyMod requires CoreLibraryMod version 1.2+"). The mod loader must respect these
dependencies, ensuring CoreLibraryMod is loaded *before* MyMod. If dependencies are missing or versions conflict, the
loader should ideally warn the user or disable the problematic mod. Tools like LOOT (Load Order Optimization Tool) for
Bethesda games automate the process of sorting mods based on known compatibility rules.

### The Mod Lifecycle: Initialization and Execution Hooks

Once a mod's files (data, scripts, binaries) are loaded into memory, its code often needs to run at specific points in
the game's lifecycle. A typical flow looks like this:

#### Load

The game loads the mod's assets and code.

#### Initialize

The game calls an initialization function or method in the mod. This is where the mod usually sets itself up, registers
things with the game engine, or applies patches.

#### Runtime Hooks

During gameplay, the game triggers the mod's code in response to specific events or at regular intervals.

#### Initialization:

##### Script Mods

Might have a specific `init()` function the engine calls after loading the script. Or the script might just run
top-to-bottom, registering event handlers as it goes.

##### Compiled Mods

Often have a designated entry point class. *Bannerlord* mods inherit from `MBSubModuleBase` and override methods like
`OnSubModuleLoad()` [5]. *RimWorld* mods can have a class inheriting from `Verse.Mod`, whose constructor acts as the
init hook [4]. *Minecraft Forge* defines a whole sequence of initialization events (`FMLPreInitializationEvent`,
`FMLInitializationEvent`, `FMLPostInitializationEvent`) that mods listen for, ensuring things happen in the right
order (e.g., all items registered before recipes) [14].

This initialization phase is crucial for mods to tell the game "I exist, and here's what I do." They might register new
item types, add UI elements, subscribe to game events, or (in the case of patching libraries like Harmony) apply their
modifications to the game's base code.

#### Runtime Execution Hooks

After initialization, how does mod code get triggered during actual gameplay? Developers provide various "hooks":

##### Event Callbacks / Subscriptions

This is very common. The game engine defines a set of events (e.g., `OnPlayerDamaged`, `OnQuestStarted`, `OnTick`,
`OnGuiRender`). Mods can register functions (callbacks) to be executed whenever a specific event occurs. The engine
manages firing these events and calling all subscribed mod functions, often passing event-specific data (like the amount
of damage taken, or the quest ID).

* *Factorio*'s `script.on_event(defines.events.EVENT_NAME, function(event_data) ... end)` is a classic example [6].
* Paradox's DSL works similarly; event blocks have `trigger` conditions the engine constantly checks, and `immediate` or
  `option` effects that run when triggered [15].
* *Minecraft Forge* has an extensive event bus (`MinecraftForge.EVENT_BUS`) covering hundreds of game actions.
* *Bannerlord*'s `CampaignEvents` system allows mods to subscribe to things like `DailyTickEvent`.

##### Method Overrides / Subclassing

If the game's architecture uses object-oriented principles heavily, it might allow mods to subclass existing game
classes and override virtual methods to change behavior. *Bannerlord* does this with its `CampaignBehaviorBase`,
allowing mods to add custom logic to the campaign loop.

##### Direct Patching (e.g., Harmony)

This is a more invasive but powerful technique, extremely popular in the Unity C# modding scene (*RimWorld*, *Kerbal
Space Program*, *Cities: Skylines*, sometimes *Bannerlord*). Libraries like Harmony allow mods to dynamically modify the
intermediate language (IL) bytecode of existing game methods *at runtime*. Mods can:

###### Prefix

Run code *before* the original method executes. Can modify arguments or even skip the original method entirely.

###### Postfix

Run code *after* the original method executes. Can access the return value and modify it.

###### Transpiler

Directly rewrite the IL instructions of the original method. Extremely powerful, but complex and fragile.

*RimWorld* mods use Harmony extensively to alter core game mechanics without needing the developers to provide explicit
hooks for everything [12]. While powerful, multiple mods patching the same method can lead to compatibility nightmares
if not carefully managed.

##### Tick/Update Hooks

Some systems allow mods to register a function that gets called every game frame or every simulation tick (e.g.,
`OnApplicationTick` in *Bannerlord*, or update loops in Unity). This is necessary for mods that need continuous
processing, but must be used cautiously to avoid performance degradation.

#### Handling Mod Errors:

What happens if a mod script has a bug or a compiled mod throws an exception? A robust modding framework should
anticipate this. Ideally, the game engine should wrap calls into mod code within error handlers (like `try-catch`
blocks). If a mod crashes:

* Log the error clearly, indicating which mod caused it.
* Prevent the error from crashing the entire game if possible.
* Potentially disable the faulty mod for the rest of the session.
* Notify the user about the issue.

Games like *RimWorld* are pretty good at catching mod errors and displaying a debug log window without necessarily
crashing, allowing the player to continue (though the game state might be compromised).

### Dynamic Loading (Hot Swapping)?

Can you install or uninstall mods while the game is running? Usually, no. Most games require a restart for mod changes
to take effect. Why? Because mods often need to integrate deeply during the initial loading phase (registering items,
patching code). Injecting or removing this integration into a live, running simulation state is extremely complex and
prone to errors. It's much simpler and safer to load everything upfront.

However, some systems are exploring partial runtime loading, especially for assets or data-driven content [16], but
runtime code injection/removal remains rare in mainstream modding.

An interesting related concept is **parallelized loading**. *Minecraft Forge*, facing long startup times with large
modpacks, implemented parallel loading for certain initialization phases to speed things up, carefully managing
dependencies and synchronization between stages [14].

### Resource Loading

Mods aren't just code; they often include assets (textures, models, sounds) and new data definitions. The modding
framework needs to handle loading these too. Common approaches include:

#### File Overrides

A simple method where mod files placed in the correct directory structure simply override the base game files with the
same name. Older games often used this. Fragile, as multiple mods overriding the same file causes conflicts.

#### Virtual File Systems / Archives

Games like *Skyrim* use archive files (`.bsa`) and a system where loose files in a `Data` folder (often managed by mod
managers) take precedence over archives, and load order determines which loose file wins if multiple mods provide the
same one.

#### Data Merging

For structured data (like lists of items or events), the engine might merge data from multiple mods. Paradox games
effectively do this, combining event files, localization strings, etc., from all active mods into the game's runtime
database.

#### API-Driven Loading

The modding API might provide functions for mods to explicitly load their own assets (e.g.,
`LoadTexture("mymod/textures/cool_gun.png")`) or register new data entries programmatically.

A well-designed system makes it clear how mod resources are integrated and how conflicts are resolved.

In essence, managing the mod lifecycle is about orchestrating the discovery, loading, initialization, and runtime
execution of potentially many independent pieces of user content, ensuring they play nicely together (as much as
possible) and integrate seamlessly into the game's flow.

Now, for the part that keeps developers up at night...

## The Elephant in the Room: Security and Sandboxing

Okay, let's talk security. We've established that mods, especially compiled ones, can be incredibly powerful. They run
code directly on the player's machine, often within the game's own process. What stops a malicious mod author from
slipping malware into their "Awesome New Sword" mod?

Honestly? In many cases, **not much technical enforcement**.

### The Current Reality: Trust, Community, and Hope

A recurring theme in discussions about mod security for single-player PC games is that the primary defense mechanism is
*not* a technical sandbox, but rather **community trust and platform curation**. A GitHub discussion involving
developers wrestling with this exact problem concluded: *"Most games do not impose any kind of security restrictions on
mods and rely on community trust."* [12].

This plays out in several ways:

#### Reputable Sources

Players are generally advised to download mods only from well-known platforms (Steam Workshop, Nexus Mods, official game
forums, reputable community sites like ModDB). These platforms often have moderation teams and community reporting
systems to catch malicious uploads.

#### Community Vetting

Popular mods are downloaded and used by thousands of players, including many technically savvy ones. If a mod started
doing suspicious things (like making weird network calls or messing with system files), it would likely be noticed and
reported quickly. Open-source mods are even easier to inspect.

#### Antivirus Software

Basic antivirus scanning might catch known malware signatures packaged within mod files [17]. After a malware scare
involving a *Cities: Skylines* mod, Paradox stated that mods uploaded to their Paradox Mods platform undergo antivirus
scanning [18].

#### Developer Warnings

Some developers explicitly warn players about the risks. The *Bannerlord* launcher, for example, displays a message
cautioning users about running unverified DLLs from mods.

**But is this enough?** History suggests maybe not. There have been notable incidents:

#### Dota 2 Mods (2023)

Malicious mods uploaded to the Steam Workshop exploited a vulnerability in the game's JavaScript engine (Panorama UI) to
gain remote code execution capabilities on users' machines [11]. This wasn't just a mod using legitimate APIs
maliciously; it was exploiting a flaw in the sandbox itself.

#### Minecraft Mods (Various)

The large, somewhat unregulated *Minecraft* modding scene has seen multiple instances of malware distributed through
mods on platforms like CurseForge, ranging from credential stealers to ransomware [19] [20].

#### Cities: Skylines Malware (2022)

A mod author was found distributing malware through several popular mods on the Steam Workshop.

#### Bannerlord Concerns

Even without specific major incidents (yet), the community has voiced unease about the inherent risk of loading
arbitrary DLLs, questioning why a potentially safer scripting approach wasn't chosen [21].

These cases highlight that relying solely on trust and community moderation isn't foolproof, especially as modding
becomes more mainstream and potentially attracts more malicious actors. So, what *technical* solutions exist or could be
used?

### Sandboxing Script Mods: The Easier Path

If your game uses an interpreted scripting language like Lua or Python, you have a much better chance of effectively
sandboxing mods. The interpreter itself provides a natural boundary.

#### How it Works

When the game engine initializes the scripting environment for a mod, it can deliberately *limit* the functions and
libraries available to that script.

* For **Lua**, this is relatively straightforward. The host application (the game) controls the global environment table
  that scripts run in. You can simply *not* load dangerous standard libraries like `io` (file input/output) and `os` (
  operating system commands). You can also replace or wrap built-in functions. The mod only gets access to the functions
  and game objects explicitly exposed through the C API bindings created by the developer.
* *World of Warcraft*'s UI modding system is a prime example of a heavily sandboxed Lua environment. Addons can
  manipulate the UI and query some game data, but they absolutely cannot access the local file system or make arbitrary
  network calls. Certain sensitive API calls are even restricted during combat to prevent automation ("protected
  functions").
* **Python** sandboxing is possible but generally considered harder due to the language's size and dynamic nature.
  Techniques involve using restricted execution modes, customizing the available built-in modules, or running the Python
  interpreter within an OS-level sandbox.

#### Effectiveness

API-level sandboxing for scripts is quite effective at preventing mods from directly causing harm *outside* the game (
like deleting files). It doesn't necessarily prevent mods from crashing the game or behaving badly *within* the game's
logic (e.g., infinite loops, excessive resource consumption), but it significantly contains the risk.

### Sandboxing Compiled Mods: The Herculean Task

This is where things get really difficult. Compiled code (C++, C#, Java bytecode) running in the same process as the
game has, by default, the same access rights. Truly isolating it is a major challenge.

#### Why it's Hard:

There's no natural interpreter boundary to control. The mod code is executing directly (or via a JIT) on the CPU.
Preventing it from making system calls (like opening files or network sockets) requires intervening at a lower level.

#### Approaches (Mostly Theoretical or Limited)

##### OS-Level Sandboxing

Run the *entire game process* within an operating system sandbox (like a container, Windows AppContainer, macOS App
Sandbox). This limits what the *whole game* (including mods) can do. While effective for security, it can be overly
restrictive, potentially breaking legitimate game features (like saving games anywhere, interacting with peripherals)
and might not be feasible for traditional PC game distribution models (e.g., Steam games usually run with full user
privileges).

##### Process Isolation:

Run each mod (or the modding subsystem) in a *separate process* with lower privileges. The main game process
communicates with the mod process via Inter-Process Communication (IPC). This is how web browsers sandbox
tabs/extensions [12].

###### Pros

Strong isolation. A crash in the mod process doesn't take down the game. OS enforces privilege separation.

###### Cons

Huge architectural complexity for the game engine (managing multiple processes, efficient IPC for game data).
Significant performance overhead due to IPC. Very few games attempt this for modding.

##### Managed Runtime Security (Deprecated/Ineffective)

As mentioned, .NET's Code Access Security (CAS) and Java's Security Manager were attempts to allow restricting
permissions for loaded assemblies/classes *within the same process*. However, CAS is deprecated and complex, and wasn't
fully supported in Mono/Unity anyway [12]. Modern consensus is that achieving reliable in-process sandboxing this way is
extremely difficult, if not impossible [12]. Unity games using Harmony definitely don't operate under any such
restrictions.

##### Static Analysis / API Whitelisting

Instead of trying to restrict at runtime, analyze the mod's binary code *before* loading it (or at load time).

Tools like **Unbreakable** (mentioned in the GitHub discussion, used by SharpLab) analyze .NET assemblies to check which
APIs they call [12]. It works by maintaining a *whitelist* of allowed namespaces/methods. If a mod tries to use anything
forbidden (like `System.IO` or `System.Net`), the analysis fails [12].

###### Pros

Can catch attempts to use obviously dangerous APIs without runtime overhead.

###### Cons

Not foolproof (clever attackers might obfuscate calls or use reflection). Requires maintaining the whitelist. Might have
false positives/negatives. Doesn't prevent logic bombs or resource exhaustion attacks.

###### Practical Use

A game could scan mod DLLs using such a tool and refuse to load mods that fail the check, or (more pragmatically) simply
*warn* the user that the mod uses potentially unsafe APIs and let them decide whether to proceed [12].

##### System Call Interception / Runtime Monitoring

Use techniques like API hooking or kernel-level monitoring to watch what the mod code *actually does* at runtime. If it
tries to make a forbidden system call (e.g., `CreateFile` outside its allowed directory), the monitor could block it or
prompt the user. This is complex to implement robustly and can have performance implications.

#### The Takeaway

Effectively sandboxing compiled mods running in the same process is *really hard* with current mainstream technologies.
Most games simply don't attempt it, accepting the risk and relying on the community/platform defenses.

### Minecraft: A Tale of Two Editions

Minecraft's approach is illustrative. The original **Java Edition** allows compiled Java mods (via Forge/Fabric) that
run with full JVM permissions – essentially unsandboxed. Mojang never implemented a robust security model for this. When
faced with bringing mods to platforms where security is paramount (consoles, mobile), they created **Bedrock Edition**,
which uses a completely different "Add-On" system. Bedrock Add-Ons are primarily data-driven (JSON files) with limited
scripting capabilities using a sandboxed JavaScript-like API. This severely restricts what mods can do compared to Java
Edition, but provides a much safer environment suitable for cross-platform play and curated marketplaces. It shows that
sometimes the solution to retrofitting security onto an open system is to create a separate, more restricted system
alongside it.

### Where Does This Leave Us?

For single-player PC games, the status quo is largely "modder/player beware," especially for games allowing compiled
mods. While outright malicious mods seem relatively rare compared to the sheer volume of mods available, the potential
risk is undeniable and incidents do happen. Scripting languages offer a much clearer path to technical sandboxing via
API control, and developers using them should absolutely leverage that capability. For compiled mods, the industry seems
to be waiting for better, more practical sandboxing technologies to emerge (like WASM?) or relying on platform-level
solutions (better Workshop scanning, OS sandboxing features).

Let's now see how these concepts play out in practice by looking at some specific games known for their modding scenes.

## Real-World Examples: Case Studies in Modding Tech

Theory is great, but let's see how different games have actually implemented mod support, embodying the choices and
trade-offs we've discussed.

### Case Study 1: Paradox Grand Strategy Games (Clausewitz/Jomini Engine) - The DSL Kings

Paradox Interactive's titles (*Europa Universalis IV*, *Crusader Kings III*, *Hearts of Iron IV*, *Stellaris*, *Victoria
3*) are legendary for their depth and equally legendary for their moddability. Total conversion mods that create
entirely new historical or fantasy settings are commonplace. How do they achieve this? Primarily through a **data-driven
approach using a proprietary Domain-Specific Language (DSL)**.

#### Engine & Language

These games run on the in-house Clausewitz engine (with newer games incorporating a shared layer called Jomini). Modding
is done almost entirely by editing or adding plain text files (`.txt`, `.yml`, sometimes `.lua` for specific scripting
tasks). These files use Paradox's unique scripting syntax to define everything from countries, characters, events,
decisions, technologies, graphics, and even AI weighting [3] [22]. It's a declarative, key-value based language
optimized for strategy game concepts. (See the DSL example snippet in the Languages section above).

#### Execution

The engine parses these script files at game startup. It doesn't interpret them line-by-line during gameplay. Instead,
it converts the logic (especially things like event triggers and AI weights) into an internal representation that can be
evaluated very quickly by the core C++ engine [8]. This is key to maintaining performance even with thousands of
potential events or complex AI calculations running in the background. The performance cost is front-loaded into the
initial game load time.

#### No Arbitrary Code

Crucially, mods cannot inject arbitrary compiled code (no DLLs). Modders are constrained to work within the vocabulary
and structure provided by the Paradox scripting DSL. If you want to do something the DSL doesn't support (like implement
a radically new UI paradigm or a fundamentally different economic model), you're generally out of luck unless Paradox
adds the necessary script commands in a future update.

#### Loading & Lifecycle

Mods are managed via the game's launcher, which reads `.mod` descriptor files. The launcher handles enabling mods and
setting load order. The game then merges data from all active mods at startup. During gameplay, mod logic is typically
triggered by the engine evaluating event conditions or AI decision factors based on the loaded scripts. Mods don't
usually run continuous code; they react to game state changes.

#### Security

Because mods are restricted to the DSL, they are inherently sandboxed from a system perspective. A Paradox mod can't
read your files or install malware. The worst it can do is mess up your game state (which can still be annoying!). This
design choice neatly sidesteps the security nightmare of compiled mods. (The *Cities: Skylines* malware incident
mentioned earlier involved a Paradox-published but not Paradox-developed game using the Unity engine with C# mods – a
completely different technical scenario).

#### Community & Tools

Paradox actively supports modding with official wikis documenting the script commands [15] and integrates mod
distribution via Paradox Mods and Steam Workshop [23]. The community has built extensive knowledge bases and tools (like
syntax highlighters and validation utilities) around the Clausewitz scripting language.

#### Why this approach?

It perfectly suits complex simulation games where performance and stability are paramount. It allows enormous content
and rule modifications within a controlled framework, fostering a huge modding scene focused on historical accuracy,
alternate history, or fantasy conversions. The limitations on arbitrary code are accepted as a reasonable trade-off for
stability and inherent safety. Forsslund's research showing their native script parsing outperformed embedded Lua likely
solidified their commitment to this DSL-centric approach [8].

### Case Study 2: *RimWorld* - XML, C#, and Harmony Mayhem

*RimWorld*, the sci-fi colony sim by Ludeon Studios, represents a different philosophy, embracing the power (and perils)
of compiled code within the popular Unity engine.

#### Engine & Language

Built on Unity, *RimWorld* uses C# for its core logic. Modding leverages this directly:

##### XML Definitions

A huge amount of game content (items, pawns, buildings, research projects, incidents, etc.) is defined in XML files.
This provides an accessible entry point for modders – adding a new rifle or alien race can often be done just by
creating new XML defs or patching existing ones [4] [24]. This is the data-driven part.

##### C# Assemblies

For anything more complex – new behaviors, UI changes, altered game mechanics – modders write code in C# and compile it
into .DLL assemblies. The game loads these DLLs at startup [4].

#### The "API" (and lack thereof)

While *RimWorld* exposes many of its C# classes and methods as public, it doesn't have a strictly defined, stable "
Modding API" in the traditional sense. Modders often need to decompile the game's assemblies (`Assembly-CSharp.dll`)
using tools like dnSpy or ILSpy to understand how the base game works and find the classes/methods they need to interact
with or modify [7]. Documentation is largely community-driven (like the RimWorld Wiki) [4].

#### Harmony Patching

This is the secret sauce (or Pandora's Box) of *RimWorld* modding. The game *includes* the Harmony library, which allows
mods to perform runtime IL patching of existing game methods (Prefix, Postfix, Transpiler) [12]. This means mods can
fundamentally alter almost *any* aspect of the game's behavior, even methods the developer never intended to be
moddable, without needing source code access. Want to change how colonists prioritize tasks? Patch the job assignment
methods. Want to add psychic space llamas? Patch the animal spawning logic. This provides incredible power and
flexibility.

*Example Harmony Patch (Conceptual):*

```csharp
using HarmonyLib;
using Verse; // RimWorld's core namespace

[HarmonyPatch(typeof(Pawn_JobTracker), "DetermineNextJob")] // Target the job finding method
public static class Patch_JobFinder {
    // Run *after* the original method finds a job
    static void Postfix(ref ThinkResult __result, Pawn ___pawn) {
        // If the original method found a job, and our mod wants pawns to prioritize hauling...
        if (__result.Job != null && ___pawn.story?.traits?.HasTrait(MyModDefOf.ObsessiveHauler) == true) {
            // ...maybe try to find a hauling job instead, even if it's lower priority originally.
            // (Actual logic would be more complex, finding hauling jobs etc.)
            // If we find a better hauling job, replace __result.Job with it.
        }
    }
}
```

#### Execution & Performance

C# mods run as JIT-compiled .NET code, offering good performance. However, heavy use of Harmony patching can introduce
overhead, as each patched method call might involve extra hops through prefix/postfix code. Poorly optimized mods,
especially those hooking into frequent updates, can definitely impact game speed.

#### Loading & Lifecycle

Mods are loaded from a Mods folder or Steam Workshop at startup. Load order is crucial and user-configurable. Mods with
code typically have a class inheriting from `Verse.Mod`; its constructor is called during initialization, often used to
apply Harmony patches or load settings. Runtime execution depends on what the mod does – event subscriptions (via
patching), Harmony patches triggering on specific method calls, or custom `Comp` (component) classes attached to game
objects that receive updates.

#### Security

There is **no sandbox**. RimWorld mods run with full .NET permissions within the game's process. A malicious mod could
theoretically do anything. The community relies entirely on trust, platform moderation (Steam Workshop), and the fact
that the developer (Tynan Sylvester) has fostered a generally positive and collaborative modding environment. Stability
is also a concern; conflicting Harmony patches or buggy mod code can cause errors or crashes, though RimWorld's error
handling often catches these and allows the game to continue (with a red error log).

#### Community & Tools

Despite the lack of a formal API, the community thrives, using decompilers, community wikis, shared libraries like
HugsLib (for common modding utilities), and tools to debug Harmony patch conflicts. The accessibility of XML combined
with the power of C#/Harmony has created one of the most vibrant modding scenes around.

#### Why this approach?

It maximizes modder freedom and creativity, leveraging the power of the Unity/C# environment. By including Harmony, the
developers essentially acknowledged that modders *would* find ways to patch the game anyway and provided a
standardized (if powerful) way to do it. The trade-off is a higher reliance on community knowledge, potential
instability from conflicts, and a complete lack of technical security enforcement.

### Case Study 3: *Mount & Blade II: Bannerlord* - Official Modules, Unofficial Risks

TaleWorlds Entertainment's *Mount & Blade II: Bannerlord*, a medieval sandbox RPG/strategy game, was developed with
modding explicitly in mind, offering official tools and a structured system based on .NET.

#### Engine & Language

*Bannerlord* uses a custom engine, but a significant portion of the game logic is written in C#. Modding follows suit,
primarily using C# compiled into DLLs, alongside XML for data definition [5] [25].

#### Module System

The game itself is structured into modules (Native, Sandbox, Storymode, etc.). Mods are simply additional modules. Each
module has a `SubModule.xml` descriptor file specifying its ID, version, dependencies, and crucially, the DLL to load
and the entry point class [5]. This provides a clear, structured way to organize and manage mods.

#### Official API & Tools

TaleWorlds provides official modding tools (including a scene editor) and relatively extensive API
documentation [26] [5]. Modders write C# code referencing the official `TaleWorlds.*.dll` assemblies. The API provides
base classes (like `MBSubModuleBase`) and event systems (`CampaignEvents`) for mods to hook into.

*Example Bannerlord Behavior (Conceptual):*

```csharp
using TaleWorlds.CampaignSystem;
using TaleWorlds.Core;
using TaleWorlds.MountAndBlade;

public class MyBanditModSubModule : MBSubModuleBase {
    // Called when the game starts a campaign
    protected override void OnGameStart(Game game, IGameStarter gameStarterObject) {
        if (game.GameType is Campaign) {
            CampaignGameStarter campaignStarter = (CampaignGameStarter)gameStarterObject;
            // Add our custom behavior to the campaign
            campaignStarter.AddBehavior(new EnhancedBanditBehavior());
        }
    }
}

// Our custom behavior logic
public class EnhancedBanditBehavior : CampaignBehaviorBase {
    public override void RegisterEvents() {
        // Subscribe to the hourly tick event
        CampaignEvents.HourlyTickEvent.AddNonSerializedListener(this, OnHourlyTick);
    }

    public override void SyncData(IDataStore dataStore) { /* Handle save/load */ }

    private void OnHourlyTick() {
        // Every hour, maybe make bandits smarter or more aggressive...
        // Access game state via Campaign.Current, MobileParty.All, etc.
    }
}
```

#### Execution & Performance

Mods are compiled C# DLLs loaded by the .NET runtime, running with excellent performance.

#### Loading & Lifecycle

The game launcher reads `SubModule.xml` files, resolves dependencies, and determines load order. Users can
enable/disable modules and adjust order in the launcher. The engine then loads the specified DLLs and calls methods on
the mod's `MBSubModuleBase` subclass at specific lifecycle points (`OnSubModuleLoad`, `OnGameStart`,
`OnApplicationTick`, etc.) [5]. Mods can then register for further events or add behaviors as needed.

#### Security

Like *RimWorld*, *Bannerlord* **does not sandbox** compiled mods. The DLLs run with full permissions. This has been a
point of discussion and concern within the community [21]. TaleWorlds relies on user caution and platform trust. The
official launcher explicitly warns about running unsigned code.

#### Stability

Compiled mods mean game updates can easily break compatibility. TaleWorlds has worked to improve API stability over
time, but modders often need to update their mods for new game versions. The structured module system and dependency
management help mitigate some conflicts compared to a free-for-all patching system. While Harmony *can* be used in
Bannerlord, many common modding tasks can be achieved via the official API hooks, potentially leading to fewer direct
method conflicts than in RimWorld.

#### Why this approach?

TaleWorlds aimed to provide powerful, official modding support from the start, recognizing the importance of mods to the
*Mount & Blade* franchise. They opted for compiled C# mods within a structured module system, providing official tools
and APIs. This enables deep modifications needed for total conversions (like popular *Game of Thrones* or *Warhammer*
mods) but sacrifices inherent security. It's a middle ground between Paradox's safe-but-limited DSL and RimWorld's
wild-west Harmony patching.

### Case Study 4: *Minecraft* (Java Edition) - Community Forged Power

*Minecraft* is a fascinating case because its massive modding scene emerged largely *without* official support in the
early days, forcing the community to build the infrastructure themselves.

#### Engine & Language

*Minecraft: Java Edition* is, unsurprisingly, written in Java. Modding involves writing Java code compiled into `.jar`
files.

#### Community Loaders (Forge & Fabric)

Since Mojang didn't provide a modding API for years, the community stepped in. **Minecraft Forge** became the dominant
mod loader. Forge works by patching the vanilla Minecraft Java bytecode at runtime to insert hooks, an event bus, and a
mod loading system. **Fabric** is a newer, more lightweight alternative that uses the **Mixin** library to apply
bytecode modifications more surgically. Both essentially create a moddable version of the game engine on the fly.

#### The "API"

Forge and Fabric provide APIs that modders code against. These APIs offer abstractions over the (often obfuscated)
internal Minecraft code, providing access to game objects, events, and registration systems. Modders typically need
mappings (provided by the community) to deobfuscate Minecraft's code during development.

*Example Forge Mod Structure (Conceptual):*

```java
import net.minecraftforge.fml.common.Mod;
import net.minecraftforge.fml.common.event.FMLInitializationEvent;
import net.minecraftforge.fml.common.event.FMLPreInitializationEvent;
import net.minecraftforge.common.MinecraftForge;
import net.minecraft.item.Item;
// ... other imports

@Mod(modid = MyMod.MODID, name = MyMod.NAME, version = MyMod.VERSION)
public class MyMod {
    public static final String MODID = "mymod";
    public static final String NAME = "My Awesome Mod";
    public static final String VERSION = "1.0";

    // Reference to the mod instance
    @Mod.Instance
    public static MyMod instance;

    // Example Item instance
    public static Item myAwesomeItem;

    @Mod.EventHandler
    public void preInit(FMLPreInitializationEvent event) {
        // Phase to register blocks, items, entities, etc.
        myAwesomeItem = new MyItem(); // Assume MyItem extends Item
        // ForgeRegistries.ITEMS.register(...); // Actual registration logic
        Log.info("My Mod PreInitialization complete.");
    }

    @Mod.EventHandler
    public void init(FMLInitializationEvent event) {
        // Phase to register recipes, event handlers, etc.
        MinecraftForge.EVENT_BUS.register(new MyEventHandler());
        Log.info("My Mod Initialization complete.");
    }

    // ... potentially other event handlers (PostInit, ServerStarting, etc.)
}
```

#### Execution & Performance

Mods are compiled Java bytecode running within the same Java Virtual Machine (JVM) as the game. Performance is generally
very good thanks to the JVM's JIT compiler, though large modpacks can strain CPU and memory resources due to the sheer
amount of added content and logic.

#### Loading & Lifecycle

Forge/Fabric scan a `mods` folder for JAR files. They handle dependencies and load order. They provide a structured
lifecycle with distinct phases (preInit, init, postInit) ensuring mods initialize in a coordinated manner [14]. Forge
even parallelized parts of this to speed up loading [14]. Mods typically interact with the game via event subscriptions
on the Forge/Fabric event bus or by registering their custom blocks/items/entities which then get handled by the
modified game loop.

#### Security

Zero technical sandboxing. Java mods run with full permissions within the JVM. The Java Security Manager *could* have
potentially been used, but wasn't, and is now deprecated anyway. The community relies heavily on downloading from
trusted sources (CurseForge, Modrinth) and community vigilance. However, as noted earlier, malware incidents *have*
occurred, highlighting the risks of this open, community-driven ecosystem [19]. The Log4Shell vulnerability also
impacted Minecraft and potentially mods using the vulnerable library.

#### Stability & Compatibility

Game updates frequently break mods, as Mojang doesn't maintain a stable API for the community loaders. Mod authors often
have to scramble to update their mods for new Minecraft versions. Compatibility between mods within large modpacks can
also be a significant challenge, requiring careful configuration and sometimes community-made compatibility patches.

#### Why this approach?

It wasn't really a deliberate *design* by Mojang, but rather an *emergent phenomenon*. The community desperately wanted
deep modding capabilities and built the tools to make it happen by reverse-engineering and patching the game. This
resulted in unparalleled creative freedom but also significant challenges in maintenance, stability, and security.
Mojang eventually provided obfuscation mappings to help the community, but largely left the Java modding scene to its
own devices while focusing on the safer, more limited Add-On system for Bedrock Edition.

These case studies showcase the spectrum: Paradox's controlled DSL garden; RimWorld's hybrid XML/C# approach amplified
by Harmony; Bannerlord's official but unsandboxed C# module system; and Minecraft's community-built Java powerhouse.
Each reflects different priorities and results in a distinct modding culture.

So, where does modding tech go from here?

## Looking Ahead: The Future of Modding Tech

The world of game modding technology isn't static. Developers, engine creators, and researchers are constantly exploring
ways to make modding safer, more powerful, and easier for both creators and players. Here are some promising directions
and research ideas, drawing inspiration from the challenges and successes we've seen:

### Safer, Performant Sandboxing: The WASM Hope (and others)

The holy grail is achieving the security of interpreted scripts with the performance of compiled code. **WebAssembly (
WASM)** keeps coming up as a potential solution.

#### The Idea

Games could define their modding API. Modders could write mods in various languages (C++, Rust, C#, Swift,
AssemblyScript) that compile to WASM modules. The game engine would then embed a WASM runtime (like Wasmer, Wasmtime, or
even browser engines) to execute these modules.

#### Why it's Promising

##### Security

WASM runs in a sandbox by default. It has no access to the host system (files, network, etc.) *unless* the host (the
game engine) explicitly provides functions (imports) to allow it. The engine could expose only the necessary game API
functions, creating a tightly controlled environment. Memory safety is also a core design principle.

##### Performance

WASM is designed for near-native performance, often using JIT compilation. While there's still a cost to calling between
the host and WASM, it's potentially much lower than traditional scripting VMs for complex computations *within* the mod.

##### Language Agnostic

Modders could potentially use their preferred language, broadening the pool of potential creators.

#### Challenges

Designing a robust, ergonomic, and performant API bridge between the game engine and WASM modules is non-trivial.
Debugging WASM code can be more complex. Tooling and engine integration are still evolving.

#### Status

While WASM is used for scripting in some contexts (e.g., some cloud platforms, even experimental game engine plugins),
widespread adoption for *user game modding* is still largely a research/future direction.

Other sandboxing avenues include improving OS-level containerization for games or exploring new language-level security
features in managed runtimes (though progress here seems slow for in-process scenarios).

### Smarter Verification: Static Analysis and Capabilities

Beyond runtime sandboxing, we can try to verify mods *before* they run.

#### Advanced Static Analysis

Tools like Unbreakable (for .NET) [12] demonstrate the concept of scanning code for calls to disallowed APIs. Future
tools could use more sophisticated techniques (symbolic execution, abstract interpretation) to check for more complex
properties, like potential infinite loops, excessive resource usage, or violations of specific game rules, without
actually running the code.

#### Capability-Based Security

Instead of just blacklisting bad things, analyze what *capabilities* a mod requires (e.g., "needs file access to save
settings," "needs network access to check for updates"). This information could be displayed to the user upon
installation ("This mod wants to access your network. Allow?"), similar to mobile app permissions. Mod platforms could
enforce policies based on declared capabilities.

### Standardization and Engine-Level Support

Could modding become a standard feature provided by major game engines like Unity and Unreal, rather than something each
developer has to implement from scratch?

#### Engine Modding Middleware

Imagine if Unity or Unreal offered a built-in, configurable modding framework – perhaps a secure scripting runtime (
maybe WASM-based?), a standard mod packaging format, and APIs for loading/managing mods. This could drastically lower
the barrier for developers to add mod support. Projects like `mod.io` are trying to provide cross-engine solutions for
mod distribution and management [16], but deeper engine integration could go further.

#### Standardized APIs?

Probably a long shot given the diversity of games, but perhaps standards could emerge for common modding tasks (e.g.,
asset loading, event handling), making it easier for modders to transfer skills between games using the same engine.

### Better Mod Development Experiences

Making modding easier and more interactive could unlock even more creativity.

#### In-Game Modding Tools

Some games are blurring the lines between playing and creating (*Roblox*, *Dreams*, *Core*). Could traditional games
offer better in-game tools for modding? Imagine live scripting environments where you can tweak mod code and see the
results instantly without restarting the game. Hot-reloading for scripts or even compiled code (which some engines
support during development) could be exposed to modders.

#### AI-Assisted Modding

AI code generation tools (like GitHub Copilot) can already help modders write boilerplate code. Future AI could
potentially help with debugging, optimizing mod performance, generating assets, or even suggesting ways to ensure
compatibility with other mods.

### Taming Complexity: Modularity and Inter-Mod Communication

As mod lists grow, managing interactions becomes key.

#### Designing for Modularity

Encourage mods to be small, focused, and expose clear APIs for other mods to use. This requires developers to provide
mechanisms for mods to discover and interact with each other safely.

#### Cross-Mod Event Buses

Expand the game's event system to allow mods to publish and subscribe to *custom* events, enabling decoupled
communication. Mod A could broadcast "NewResourceAdded" and Mod B could listen for it, without either needing direct
knowledge of the other.

#### Conflict Resolution Tools

Better tools (perhaps integrated into game launchers or mod managers) to automatically detect potential conflicts (e.g.,
two mods patching the same method, overriding the same data entry) and suggest solutions or load order adjustments.

### Enhanced Security via Platforms and Process

Leveraging the distribution platform for better security.

#### Cloud-Based Verification

Mod platforms (Steam Workshop, Nexus Mods, Paradox Mods) could implement automated cloud-based sandboxing and analysis.
Before a mod is publicly listed, it could be run in a secure environment to monitor its behavior (file access, network
calls, crashes). This is akin to how mobile app stores vet submissions.

#### Clearer Labeling

Platforms could require mods to declare their capabilities and display clear security warnings (e.g., "This mod uses
compiled code and runs with full permissions," "This mod only uses sandboxed scripts").

### Tiered Modding Support

Recognizing that not all mods need the same level of power or risk, developers could offer tiered support:

#### Tier 1 (Data/Config)

Safest level. Mods can only modify data files (XML, JSON) or use a very limited, declarative DSL. Suitable for content
additions, balance tweaks. Could potentially be allowed even in multiplayer or on consoles.

#### Tier 2 (Sandboxed Scripting)

Mods use an embedded scripting language (Lua, WASM) running within a strict sandbox, using only approved APIs. Allows
more complex logic but contained.

#### Tier 3 (Compiled Code)

Full power, full risk. Mods are compiled DLLs/JARs running unsandboxed. Requires explicit user consent and warnings.
Reserved for total conversions and deep system changes where performance is paramount.

This allows players and developers to choose the level of risk/reward they are comfortable with.

The future of modding tech likely involves combining several of these ideas – perhaps WASM for safe-but-performant code
execution, coupled with better platform-level verification and clearer user communication about the risks involved with
different types of mods.

## Wrapping Up: The Takeaway

Whew, that was a deep dive! Supporting user-created mods is clearly far more complex than just throwing a `Mods` folder
into the game directory. It involves fundamental choices about software architecture, language design, execution
environments, and security posture.

We've seen a spectrum of approaches:

* Paradox's controlled, performant, safe **DSL-driven** world.
* RimWorld's open, flexible, slightly chaotic **XML + C# + Harmony** ecosystem.
* Bannerlord's attempt at a structured, official **C# module system** (still lacking a sandbox).
* Minecraft's **community-built Java framework** rising from a lack of official support.

Each approach reflects different priorities and trade-offs between **modder flexibility, runtime performance,
development ease, and security/stability**. There's no single "right" answer; the best approach depends heavily on the
type of game, the engine technology, the performance budget, and the kind of modding community the developers want to
foster. Forsslund's work highlighted the stark performance cost of embedding Lua vs. native parsing in EU3 [8],
underscoring why performance-sensitive games might favor DSLs or compiled code, while games prioritizing creativity
might accept the overhead of scripts or the risks of DLLs.

Designing for moddability means designing a game as a **platform**. It requires extra effort in architecting for
extensibility, maintaining API stability, and providing documentation or tools. But the payoff – immense player
engagement, extended game lifespan, unexpected innovation – is often well worth the investment.

The biggest unresolved challenge remains **security**, especially for compiled mods. The current reliance on community
trust and platform vetting feels increasingly inadequate in the face of potential threats [21] [11]. Moving towards more
technically robust solutions like WASM-based execution, better static analysis, and clearer capability management seems
essential for the long-term health and safety of modding ecosystems.

Ultimately, the technologies supporting game mods are constantly evolving. It's an exciting space where the creativity
of players pushes the boundaries, and developers respond with new tools and frameworks. By understanding the technical
underpinnings, we can better appreciate the delicate balancing act developers perform and anticipate how modding might
become even more powerful, accessible, and secure in the future.

## References

* [1] W. Scacchi, ‘Modding as a basis for developing game systems’, in Proceedings of the 1st International Workshop on
  Games and Software Engineering, Waikiki, Honolulu HI USA: ACM, May 2011, pp. 5–8. doi: 10.1145/1984674.1984677.
* [2] ‘A Comprehensive Introduction to Unreal Engine Modding’. Sep. 25, 2024. [Online].
  Available: https://buckminsterfullerene02.github.io/dev-guide/
* [3] ‘Modding’, CK3 Wiki. [Online]. Available: https://ck3.paradoxwikis.com/Modding
* [4] ‘Modding Tutorials’, RimWorld Wiki. [Online]. Available: https://rimworldwiki.com/wiki/Modding_Tutorials
* [5] ‘Bannerlord Documentation’. [Online]. Available: https://docs.bannerlordmodding.com
* [6] ‘Tutorial:Scripting’, Factorio Wiki. [Online]. Available: https://wiki.factorio.com/Tutorial:Scripting
* [7] ‘A question about the modding API, the relationship between, C# and Xml thru .dll’, Ludeon Forums. [Online].
  Available: https://ludeon.com/forums/index.php?topic=29176.0
* [8] O. Forsslund, ‘Evaluating Lua for Usein Computer Game Event Handling’, Master of Science Thesis, KTH Royal
  Institute of Technology, Stockholm, Sweden, 2013. [Online].
  Available: https://www.diva-portal.org/smash/get/diva2:678986/FULLTEXT01.pdf
* [9] D. Perelman, ‘A newbie’s introduction to Factorio modding’, A Weird Imagination. [Online].
  Available: https://aweirdimagination.net/2024/06/23/a-newbies-introduction-to-factorio-modding/
* [10] ‘Modding API’, Cities: Skylines Wiki. [Online]. Available: https://skylines.paradoxwikis.com/Modding_API
* [11] J. Vijayan, ‘Malicious Game Mods Target Dota 2 Game Users’, Dark Reading. [Online].
  Available: https://www.darkreading.com/cloud-security/malicious-game-mods-target-dota-2-game-users
* [12] ‘Running untrusted code (video game modding)’, GitHub dotnet/roslyn Discussion. [Online].
  Available: https://github.com/dotnet/roslyn/discussions/48726
* [13] ‘Modding’, Europa Universalis 4 Wiki. [Online]. Available: https://eu4.paradoxwikis.com/Modding
* [14] ‘Stages of Modloading’, Forge Community Wiki. [Online].
  Available: https://forge.gemwire.uk/wiki/Stages_of_Modloading
* [15] ‘Scripting’, Crusader Kings II Wiki. [Online]. Available: https://ck2.paradoxwikis.com/Scripting
* [16] S. Reismanis, ‘Add mod support to a Unity game in 48 hours with mod.io’, Medium. [Online].
  Available: https://blog.mod.io/add-mod-support-to-a-unity-game-in-48-hours-with-mod-io-412a4346731
* [17] ‘Sandbox games and mods for security’, Steam Forums. [Online].
  Available: https://steamcommunity.com/discussions/forum/10/4625853880055444527/
* [18] ‘Additional information regarding malware suspicion on the Mod “Traffic” on Cities: Skylines II.’, Paradox
  Interactive Forums. [Online].
  Available: https://forum.paradoxplaza.com/forum/threads/additional-information-regarding-malware-suspicion-on-the-mod-traffic-on-cities-skylines-ii.1713439/
* [19] M. Szabó, ‘How Minecraft and game modding can undermine your security’, ESET Blog. [Online].
  Available: https://www.eset.com/blog/consumer/how-minecraft-and-game-modding-can-undermine-your-security/
* [20] V. Constantinescu, ‘Minecraft Mods Hit by Massive “BleedingPipe” Vulnerability, Leaving Thousands at Risk’,
  Bitdefender. [Online].
  Available: https://www.bitdefender.com/en-gb/blog/hotforsecurity/minecraft-mods-hit-by-massive-bleedingpipe-vulnerability-leaving-thousands-at-risk
* [21] ‘Concerning about security vulnerability of bannerlord modding’, TaleWorlds Forums. [Online].
  Available: https://forums.taleworlds.com/index.php?threads/concerning-about-security-vulnerability-of-bannerlord-modding.464024/
* [22] ‘Modding’, Stellaris Wiki. [Online]. Available: https://stellaris.paradoxwikis.com/Modding
* [23] ‘Paradox Mods’. [Online]. Available: https://mods.paradoxplaza.com
* [24] ‘User:Dninemfive’, RimWorld Wiki. [Online]. Available: https://rimworldwiki.com/wiki/User:Dninemfive
* [25] ‘Taleworlds Documentation’. [Online]. Available: https://moddocs.bannerlord.com/
* [26] ‘Official Modding Documentation’, TaleWorlds Forums. [Online].
  Available: https://forums.taleworlds.com/index.php?threads/official-modding-documentation.431644/
* [27] A. Buckwell, ‘Video Game Modding: What It Is and How to Get Started’, Acer Corner. [Online].
  Available: https://blog.acer.com/en/discussion/574/video-game-modding-what-it-is-and-how-to-get-started
* [28] A. Beskazalioglu, ‘Compiled and Interpreted Programming Languages: Advantages, Disadvantages, and Language
  Selection Guide for Projects’, Medium. [Online].
  Available: https://medium.com/@ahmetbeskazalioglu/compiled-and-interpreted-programming-languages-advantages-disadvantages-and-language-selection-b260ff8d2a50
* [29] A. Amador, ‘Gaming Engines: An Undetected Playground for Malware Loaders’, Check Point Research, Nov.
    2024. [Online].
          Available: https://research.checkpoint.com/2024/gaming-engines-an-undetected-playground-for-malware-loaders/
* [30] B. Francis, ‘The Risks of Video Game Mods: An Easy Way for Malware to Spread’, Dynacomp IT Solutions. [Online].
  Available: https://www.dynacompusa.com/post/the-risks-of-video-game-mods-an-easy-way-for-malware-to-spread
* [31] E. Leblond, ‘Godot sandbox & modding support’, GitHub godotengine/godot Issues. [Online].
  Available: https://github.com/godotengine/godot/issues/7753
* [32] D. J. Torrey, ‘Building a mod system for a game’, Medium. [Online].
  Available: https://medium.com/@davidjamestorreysr/building-a-mod-system-for-a-game-fd566b00759b
* [33] A. Ivora, ‘Securing the mod system of BeamNG.drive’, Master’s Thesis, Masaryk University, Brno, Czech,
    2023. [Online]. Available: https://is.muni.cz/th/x5p5j/?lang=en
* [34] ‘Is interpreted malware easier to detect than compiled malware?’, Information Security Stack Exchange. [Online].
  Available: https://security.stackexchange.com/q/33990
* [35] ‘Gaming Mods Security Risks’, Information Security Stack Exchange. [Online].
  Available: https://security.stackexchange.com/a/81230
