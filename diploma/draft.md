# Zig as the Foundation for a Portable and Modular Game Engine

## Introduction

Until recent years, before Indie dev boom, most of the engines were proprietary and were tightly coupled to games that were built on top of them.
Developers who wanted to create a game had to do a thorough research about a target platform and write a lot of basic functionality that is now shipped in most engines by default.
Appearance of Steam and Xbox Live Arcade in mid two thousands was one of the stepping stones for indie game development as it "pulled underground indie devs up into the sun"
by taking care of the game distribution [2].
Naturally, this caused the rise in demand for game engines.
However, most of the engines at the time were proprietary, and companies behind them weren't keen on sharing their secrets, as the industry was still in its adolescence.
As a result, they were never properly researched [5].
Almost all relevant game engines are closed source which means that particularities on implementation of certain functionalities are not available to developers.
There are some books about engines, but they don't provide high level architecture overview, only low level implementation.
Most of the recent academic works agree that not a lot is known about the game development process, and most of the information about big and successful engines
come from postmortems, blogs or other media where individual developers share their experiences.
The fact that they are not thoroughly researched nor defined yet and is a source of much frustration. [1]

Additionally, rebirth of handhelds with Nintendo Switch, Steam Deck and other platforms means that developers need to take more care about the optimization
since the computing power of those devices is more limited compared to bulky PCs and consoles.
Although the semiconductor industry has been good at identifying near and long term bottlenecks of Moor's Law [3], some chip-makers claim that Moor's Law is dead,
justifying the price hike of graphics cards [4].
Thus, it is important for developers to find new ways for optimizing their games inside the software, instead of relying on the hardware to handle the ever-increasing load.
Popular engines like Unreal and Unity allow for a faster development cycle by limiting the developer's access to the engine's underlying systems.

For these reasons, it is important to periodically evaluate the underlying tools and to look for possible improvements.
One of them could be switching to another language.
Most of the modern games are written in C++ and C [1] which allows developers to have more control over the hardware.
However, both of those languages are quite old, and were designed during a different age and for different kind of hardware that is common nowadays.
(Dedicated graphic cards did not exist and 4 kilobytes of RAM was a norm)
"It is our belief that the manner in which low-level issues influence architectural design is intrinsically linked to the fact that technology changes at a very fast rate"[15]

Game-development is a very complex process nowadays, especially in AAA studios. Generating the assets and writing the code is just a tine fraction off it.
Biggest problems that are faced during the process of game development are related to scope, feature creep, and cutting features.
Additionally, second most common kind of problem faced in the game development is technical issues [6].
Creation of a game starts from pre-production stage, with writing a game design document and scribbling the concept art.
I believe that it's during this period when a company needs to decide what kind of engine will be used judging from the core gameplay features.
A right decision here might reduce the amount of technical problems by simply removing the redundant features of general-purpose engine like Unity.

Beneficially to Estonia, it is a good opportunity to make research in a field that is currently underperforming in Europe compared to US and Japan.
This could place Estonia into a dominant place in the sphere of game engine development.

In order to find possible improvements in current game development process, the following research questions will need to be answered:
Question #1: What are the most fundamental components of a game engine? Question #2: How does a programming language impact the game development process?

To answer them the following research tasks need to be performed:
1) analyze the architecture of popular game engines and identify common components;
2) compare C, C++, Rust, Zig and Odin for language features, semantics, package management and build system;
3) analyze the suitability of Zig for game development;
4) demonstrate the feasibility with a practical example in a form of a simple 2D game.

The paper is structured as follows:

- Section 1 introduces the theoretical part by explaining some of the software development concepts related to game development, and the research in this field done so far.
- Section 2.1 analyzes existing game engines;
- Section 2.2 explains the importance of a programming language;
- Section 2.3 evaluates the feasibility of creating games using a Zig language;
- Section 3 discusses our results and threats to their validity;
- Section 4 concludes.


## 1. Theoretical part

Most of the academic papers that touch the topic of game engines mention that there is not enough research being done on the topic [6][7][10][20].
"This lack of literature and research regarding game engine architectures is perplexing."[15 p1]
By the time of writing of this paper, several works about game engines internal architecture have been published,
however, to the best of my knowledge, none has discussed the importance of a programming language.

Anderson et al. do not perform a research, but introduce fundamental questions that need to be considered when designing the architecture of a game engine [15].
This is a great introductory resource that distills the current state of research of game engines.
They highlight the lack of proper terminology and game development "language", as well as the lack of clear boundaries between a game and its engine.
Authors encourage classifying common engine components, investigating the connection between low-level issues and top-level engine implementation,
and identifying the best practices in a field.
A paper also mentions an "Uberengine", a theoretical engine that allows to create any kind of game regardless of genre.

A comprehensive book by Gregory [12] about the engine architecture is referenced in multiple studies.
He proposes a "Runtime Engine Architecture", in which a game engine consists of multiple runtime components with clear dependency hierarchy.
They are Audio (AUD), Core Systems (COR), Profiling and Debugging (DEB), Front End (FES), Gameplay Foundations (GMP), Human Interface Devices (HID),
Low-Level Renderer (LLR), Online Multiplayer (OMP), Collision and Physics (PHY), Platform Independence Layer (PLA), Resources (RES),
Third-party SDKs (SDK), Scene graph/culling optimizations (SGC), Skeletal Animation (SKA), Visual Effects (VFX).
He finds an optimal middle ground between a high-level, broad description of how those modules interact with each other, and their detailed implementations.
Most of the examples are written in C++ and are object-oriented in nature, however.
The exact definition of Core Systems is vague: "useful software utilities". Some of its responsibilities are:
- Assertions(error checking);
- Memory management (malloc and free);
- Math library
- Common data structures and algorithms
- Random number generation
- Unit testing
- etc.

This leaves a question whether a standard library can be considered a Core engine system. We discuss this in Section 2.2.

Ullmann et al. [13] used this model as a target reference when exploring the architecture of three popular open-source game engines (Cocos2d-x, Godot, and Urho3D).
They added a separate World Editor (EDI) subsystem "because it [World Editor] directly impacts game developers’ work."
Godot was found to have all 16 out of 16 subsystems, while Cocos2d-x and Urcho3D 13 and 12 respectively.
However, the missing functionality was not absent, but rather scattered among other subsystems.
They also found which subsystems are more often coupled with one another: COR, SGC and LLR.
COR file are helper tools that are used throughout the system.
"Video games are highly dependent on visuals, and graphics are a cross-cutting concern, therefore it is no surprise that so many subsystems depend on LLR and SGC"[13]
Other commonly coupled systems were VFX, FES, RES and PLA.

In their next paper, they sampled 10 engines and found that the top-five subsystems were:
Core (COR), Low-Level Renderer (LLR), Resources (RES), World Editor (EDI) and Front End (FES) tied with Platform Independence Layer (PLA).
Those subsystems "act as a foundation for game engines because most of the other subsystems depend on them to implement their functionalities" [14].
A visualized dependency graph is shown in Figure 1.
There are, however, other architectures used in proprietary mainstream engines, of which very little is known (Section 2.1).
And the suggested architecture is only one of the possibilities.
![Figure 1](subsystems.png)

A breakdown of game development problems was performed by Politowski et all [6], where they determined that the second most common kind of problem is "technical", sitting at 12%.
First being "Design" problems - 13%, and third - "Team" with 10%.
Addressing the technical issues with game engines tackles the second most common kind of problem faced in a game-development industry.

A paper from Nederlands, written by Gunkel at all [18] describes the technical requirements of XR architecture
(Extended Reality, collective name for Virtual Reality, Augmented Reality and Mixed Reality).
One of the drawbacks of contemporary game engines is that they are designed to work with static meshes and not the environment around a user, which needs to be acquired in real-time.
This problem is being solved by Spatial Computing:
"widely autonomous operations to understand the environment and activities of the users and thus building the main enabler for any AR technology."
There is also a need for a metadata format that describes how and where to place an entity in a scene. The most promising one is glTF.
"GLTF is a royalty-free format that can both describe the composition of 2D and 3D media data, as well as directly compress assets in its binary format...
It is however yet to be seen if it can be established as a main entity placement descriptive format for XR".
Last point is about the Remote Rendering. To render a scene with proper lighting, shadows and, perhaps, ray-tracing, a substantial processing power is needed.
They conclude that game engines and multimedia orchestration "require a closer synergy, optimizing processes at various levels".

Mark O. Riedl [19] describes some of the common problems with AI in game engines.
He breaks down AI problems into 2 categories: pathfinding and decision-making; and introduces 2 strategies for decision-making: finite-state machines and behavior trees.
AI module takes the responsibility of datamining - analyzing how players interact with a game.
This data can be used to guide the development of patches, new content, or make business decisions.

Two papers explore game-development process as a whole.
[7] defines common problems during a project life-cycle, and similarly to Politkowski et al finds that the biggest contributor to the problems is management:
"...scope problems, for instance, feature creep - initial scope of the project is inadequately defined
and consequent development entails addition of new functionalities which becomes an issue for requirements management,"
Teams that tried incorporating Model-Based development report some success:
"model-driven development here is to abstract away the finer details of the game.
This is done with high-level models such as structure and behavior diagrams as well as control diagrams.
... it allows management to gain at least a rudimentary understanding of the game design
and help alleviate the communication problems encountered in previous development methodologies."
This probably means that the plague of scrum masters is inevitable.
Another important practice is using a Game Design Document.
[20] describes each stage in pre-production, production and post-production phases.

[1] explores whether game engines share similar characteristics with software frameworks by comparing 282 most popular open source engines and 282 most popular frameworks on GitHub.
"Game engines are slightly larger in terms of size and complexity and less popular and engaging than traditional frameworks.
The programming languages of game engines differ greatly from that of traditional frameworks.
Game engine projects have shorter histories with less releases.
Developers perceive game engines as different from traditional frameworks and claim that engines need special treatments."
Game developers often add scripting capabilities to their engines to ease the design and testing workflow,
meanwhile frameworks products are often written in the same programming languages.
They also explain that the reason for popularity of C family of languages is that "...engines must work close to the hardware and manage memory for performance.
Low-level, compiled languages allow developers to control fully the hardware and memory."[1 6.3]

Toftedahl and Engström in their work discovered that on 2018 "Unity is the engine used in approximately 47 % of the games published on Itch.io."[9]
While on Steam it comes as second (13.2%) after Unreal Engine (25.6%).
They categorize game engines into 4 types:
- Core Game Engine: collection of product facing tools used to compile games to be executed on target platforms. e.g. id Tech 3, Unity core;
- Game Engine: a piece of software that contains a core engine and an arbitrary number of user facing tools e.g Raylib, Bevy;
- General Purpose Game Engine: a game engine targeted at a broad range of game genres e.g. Unity, Unreal, Godot;
- Special Purpose Game Engine: a game engine targeted at specific game genres e.g. GameMaker, Twine.

Since they did not quantify the exact amount of user facing tools required for a Game Engine, I don't think they performed a proper job.
In theory, any single utility added to a Core Game Engine makes it a Game Engine.
Additionally, their definition of Core Game Engine is in contradiction with Gregory's "Runtime Engine Architecture" where Core files are "useful software utilities" [63, p.39].
Unity core contains several packages including a renderer and UI [11], meanwhile Gregory has a "Low-level Renderer" and "Front End" separate from "Core".
As for the special purpose game engine, this can be considered as an engine that can only compile one kind of game but with different configurations.
In which case, a mod for Minecraft can be considered a game that was produced using Minecraft engine.

Before we move on to the next part, let's brief ourselves with needed terminology and a short history.

#### Terminology

So what is a game engine? "John Carmack, and to a less degree John Romero, are credited for the creation and adoption of the term game engine.
In the early 90s, they created the first game engine to separate the concerns between the game code and its assets and to work collaboratively on the game as a team."[1]
A game became known as Doom, and it gave birth to the first-ever "modding" community.[12 1.3]
Thus, a new game could be made by simply adding new art, environment, characters etc. inside a Doom's engine.
It is hard to draw a line and try to fit this engine into a taxonomy proposed by Toftedahl and Engström:
it is both a Special Purpose engine, that creates a game in a specific genre; and a Core engine, that provides useful software utilities.

A general definition for a game engine that [9] provides after inspecting some of the popular engine's websites is as follows:
"A framework for game development where a plethora of functions to support game development is gathered".

Jason Schreier claims that a word "engine" is a misnomer:
"An engine isn’t a single program or piece of technology — it’s a collection of software and tools that are changing constantly.
To say that Starfield and Fallout 76 are using the “same engine” because they might share an editor and other common traits
is like saying Indian and Chinese meals are identical because they both feature chicken and rice".[21]

Lastly, Politowsky et al. gathered multiple definitions of "game engine":
"In 2002, Lewis and Jacobson defined game engines as
“collection[s] of modules of simulation code that do not directly specify the game’s behavior (game logic) or game’s environment (level data)”.
In 2007, Sherrod defined engines as frameworks comprised of different tools, utilities, and interfaces that hide the low-level details of the implementations of games.
Engines are extensible software that can be used as the foundations for many different games without major changes and are “software frameworks for game development”.
They relieve developers so that they can focus on other aspects of game development” [1].

In brief, they all imply that an engine is foremost a collection of software tools.
A code that can be reused, and allows game developers to start working on an important problem instead of writing general utilities from scratch.
It is my belief that a "game engine" is an imaginary concept that is used as an umbrella term for software tools that help with creating video games.
Therefore, for the purpose of this paper, "game engine" will imply an ecosystem around software development, which helps with producing video games in one way or another.
Development environment potentially has a significant influence on the quality of a final product, and so cannot be ignored when designing a modern engine.

During the analysis of programming languages, terms "low-level" and "high-level" will be used frequently.
There is no clear answer what makes a particular language high- or low-level, but it is good comparison tool.
A language like C can be perceived as low-level compared to Python, because a developer is responsible for memory management;
but on the other hand C might be treated as high-level compared to x86 assembly, because a developer does not work with registers directly.
Again, for the purpose of this paper, lower level will suggest that a language gives software engineers more control over the hardware.

GPU is a Graphics Processing Unit.
Unlike CPU, GPU can have some thousands of cores, but they are specialized to do floating point operations and do calculations in bulk.
Usually, to talk to a GPU, one needs to call graphics API like OpenGL, DirectX, Vulkan etc.
A renderer (or rendering engine) is a part of a game engine that is responsible for drawing pixels onto the screen.
A shader is a program that is sent to the GPU to be executed, and it "runs on each vertex or pixel in isolation"[22].

An important distinction we must establish is the language of engine vs language of scripting.
An engine needs to run a game fast. But game development cycle also needs to be fast.
For this purpose an engine can integrate a scripting language.
Such language is more limiting than the language an engine is written in, but it allows skipping a compilation step every time there is a new change to the game.
Scripting language provides only a subset of features provided by an engine.
A scripting language can be virtually anything, from a simple custom parser to full-fledged languages like C#, Lua or Python.

Now with the theory covered, we take a look at some of the popular game engines, and gather more data from concurrent blog posts and articles.

---
Draft
---

## 2. Research

According to Toftedahl and Engström, Unreal and Unity are currently the most widely used game engines [9].
It is impossible to analyze Unity's architecture due to a proprietary code base,
Unreal Engine source code can be accessed on Github, but due to the sheer complexity of this project, analysis of its codebase is outside the scope of this paper.
It is, however, important to understand what kind of features makes them appealing towards game developers.

Throughout my university studies I've had the experience of making a game with following engines: Unity, UE4, Raylib, SDL2, OpenGL + glfw, Bevy, Godot and Mach.
Some of them are considered general purpose, while others only provide the most basic tools that allow to draw pixels onto the screen.

Firstly, we answer what features from Unity and Unreal are the most appealing for game developers and make them a guideline for our game engine.


Therefore, we gather the inspiration for architecture from these.
The only way for a developer to understand the way certain components work and communicate is to create his/her own computer game engine [13].
Plus the second biggest reason why new engines are being created is because of the learning purposes. With first reason needing more control over the environment.
The actual need to create a game is only on third place [1].


My notes on Unity and UE5.

Why is everyone obsessed with Unreal Engine and Unity?
> The trend is moving towards more and more automation of the process, and towards improving the quality of the product at the final stage.
> Neural networks will actively help process large data sets and improve graphics variability,
>
> The technical barrier of entry continues to get lower, so the systems aren’t just visually superior but also easier for less-technical artists.
>
> One of the biggest opportunities is how real-time engines facilitate exchanges via the marketplace
https://www.creativebloq.com/news/whats-the-future-for-game-engines

Unreal Engine and Unity: why you need to learn a game engine
> The worldwide demand for Unreal Engine skills was projected to grow 138%
>
> One of the main areas that efficiency is of the essence is with pre-visualisation.
> Results must be delivered as fast as possible with artists responding quickly to last-minute changes.
> For artists to thrive in this environment, they need tools that will deliver instant feedback.
>
> This is evidenced in Unity’s dedicated AR tools, which are ready to go straight out of the box.
> This includes a purpose-built framework, Mars tools, and an XR Interaction Toolkit.
https://www.creativebloq.com/features/why-you-need-to-learn-a-game-engine

Is Unreal Engine the future of filmmaking?
> ...working in real time made things more dynamic.
>
> Pros: quickly visualise scenes, frame shots and experiment with the film's "world" before committing to filming.
>
> Unreal Engine is an incredible catalyst for world building
https://www.creativebloq.com/features/use-unreal-engine-for-filmmaking

Why the world needs more 3D designers
> In the world of 3D design, demand is so high that there is more work to do than there are designers to do it.
>
> Perfect photorealism will become more and more accessible, and whether to go for a conventional shoot or a CGI production will become almost only a matter of budget.
With the rise of pdemand in 3D art, there rises a demand for a software that allows to create it.
https://www.itsnicethat.com/features/why-the-world-needs-more-3d-designers-adobe-report-231122

Unity vs Unreal: Pros and Cons
> Unity + : cross-platform support.
> Unreal + : support for high-definition graphics (physics, VFX, lighting, etc.). Scripting with blueprints.
> Pros: many learning resources. Asset libraries.
> Cons: bulky and proprietary.
> Unity more popular then UE: beginer-friendly + bigger community.
https://kevurugames.com/blog/unity-vs-unreal-engine-pros-and-cons/

Unity - What makes it the best game engines?
> If we were to make a list of the best engines for beginners, Unity would be on top.
> Pros: cross-platform, simple UI, community, asset store, visual [scripting](https://unity.com/how-to/make-games-without-programming) like Bolt or Playmaker
> VR support https://blog.unity.com/technology/ar-vr-branded-content-uncover-new-ways-to-connect-with-your-audience
> Cons: less performance compared to narrowly focused engines, plugin support, large size, license.
https://kevurugames.com/blog/unity-what-makes-it-the-best-game-engine/

| | Unity | Unreal |
| :--: | :--: | :--: |
| Ease of use | beginer-friendly | steep learning curve |
| Scripting | C# easy | C++ hard but blueprints |
| Community | big | big but smaller than Unity |
| Indie | prefered | no. Resource-intensive |
| 2D | good | not prime |
| 3D | good | superb |
| Customization | customizable | Powerful but is seen as more rigid in terms of workflows |
| Updates | frequent | slower |

#### Unity

Unity’s primary design goals are ease of development and cross-platform
game deployment.
> Game engine architecture, Jason Gregory

> https://www.arnia.com/what-makes-unity-so-popular-in-game-development/
    Unity 1.0 was released in June 2005.
    The automotive industry, the architecture sector, healthcare, military, or the film industry, are just a few examples of domains that have been successfully adopting Unity.
    Unity is now supporting over 25 different platforms.
    Unity holds the supremacy for AR and VR as well, with over 60% of the developed content.
    Free version.
    Great Graphics.
    Play Mode Option (test and review the gameplay on the go)
    Unity Asset Store.
    Less Coding
    Strong Community (active forums, plenty of tutorials)
    Built-in Analytics (Real-time data and prebuilt dashboards)

> https://kevurugames.com/blog/unity-what-makes-it-the-best-game-engine
    71% of them (mobile games) are Unity-based according to Unity Technologies
    Unity consists of 3 main parts:
    - game engine (allows to create, test, and play 3D and 3D games and experiences in various environments);
    - application (combines design and user interface with graphics preview and playback controls);
    - code editor IDE (aka Integrated development environment that provides a separate text editor for writing code).
    Free version, provided that the annual income from the game does not exceed $100,000.
    + Unity is the best cross-platform option, covering all known modern platforms.
    + Community, tutorials, asset store.
    Component-based approach. Developer creates objects and adds various components to them.
    Visual scripting with Bolt (plugin from asset store).
    High system requirements for more advanced features (more fancy stuff - more resource consumption and no ability for optimization)
    Less performance compared to other narrowly focused gaming engines
    No support for links to external libraries.
    Large size and slow.
    Is Unity the best game engine? Answer this question yourself, because it all depends on how its functionality is suitable for the implementation of your ideas.
    The Unity gaming engine demonstrates excellent performance in creating absolutely diverse products.

> https://retrostylegames.com/blog/is-unity-good
    Cross-platform games become more and more popular every year. "Players want to be able to play on whatever device they have"
    Game Server Hosting, Matchmaker, and Cloud Content Delivery help studios enable cross-platform play.
    Unity Asset Store offers a vast collection of assets, plugins, and tools that can be easily integrated into projects, saving time and enhancing development efficiency.
    Absence of the need to start every project from scratch:
    "It’s better to spend a hundred dollars on the Unity Asset Store instead of doing something that would cost twenty or thirty thousand dollars and two to three months to develop."
    Datamining (LiveOps)
    Unity was always well-known for its fascinating adaptability to modern trends.
    All you should know is just how to make game art. Everything else can easily be done with Unity’s tools.
    Unity Build Automation and Unity Version Control - devops tools. 47% of indies and 59% of midsize studios started to use DevOps to release quickly.
    XR
    Asset Manager Beta (important piece generally!)
    Targeting mobile: "It’s quick to make, the potential audience is larger due to accessibility, and there’s less cost involved in making the prototype."
    Unity Reduces Game Development Timelines
    Keep up with AI trends:
    - Leonardo.Ai - creating textures, images, and even 3D models.
    - Polyhive: Texture 3D Assets with AI
    - AutoGen - generate dynamic and realistic skyboxes using AI
    - Shap-E - converting text input into 3D models
    The creators of GenieLabs say that it’s much easier for game developers to allow players to create game content by themselves rather than work for years on thousands of DLCs.
    Also, this approach helped to extend the lifespan of existing games by 33%
    With a wide community of developers utilizing Unity, finding skilled talent becomes more accessible.
    ...challenges with handling AAA games, large landscapes with heavy on-screen elements, and networked games.
    Also, it can restrict deep optimizations and might require workarounds for complex tasks.


#### Unreal doc
https://docs.unrealengine.com/5.3/en-US/API/QuickStart/
https://docs.unrealengine.com/5.3/en-US/programming-with-cplusplus-in-unreal-engine/
TLDR Very C++ object orientedness.

UnrealBuildTool builds the targets: game, editor, etc.
Each target is compiled from C++ modules, each implementing a particular area of functionality.
Your game is a target, and your game code is implemented in one or more modules.
Code in each module can use other modules by referencing them in their build rules.
Build rules for each module are given by C# scripts with the .build.cs extension.
Target rules are given by C# scripts with the .target.cs extension.

The ** Online Subsystem ** and its interfaces provide a common way to access the functionality of online services such as Steam, Xbox Live, Facebook, and so on.
Together with Online Service and built-in source control allows to seamlessly ship new updates and work on game changes incrementally without overhead.

C++ Wizard allows to graphically add/extend native C++ classes.

Modules are the basic building block of Unreal Engine's (UE) software architecture.
These encapsulate specific editor tools, runtime features, libraries, or other functionality in standalone units of code.
Modules supplied with the Unreal Engine are divided into three categories; the Runtime, functionality for the Editor, and Developer utilities.
Most gameplay programming just uses runtime modules, and the three most commonly encountered are Core, CoreUObject and Engine.
Modules enforce good code separation, compiled as separate compilation units, . Control when specific modules are loaded and unloaded at runtime. 
Modules can be included or excluded from your project based on certain conditions,

Include What You Use (IWYU) allows to only include the parts that your project uses (instead of full header + source) and increases build times.

The **Core** module provides a common framework for Unreal modules to communicate;
a standard set of types, a math library, a container library, and a lot of the hardware abstraction that allows Unreal to run on so many platforms.

The **CoreUObject** module defines UObject, the base class for all managed objects in Unreal.
Managed objects are key to integrating with the editor, for serialization, network replication, and runtime type information.

The **Engine** module contains functionality you'd associate with a game. The game world, actors, characters, physics and special effects are all defined here.

Many features are implemented using a plugin system, and are not part of an engine.
Overall this engine can do the vast majority of tasks that one would expect in a game engine.

**Supports both glTF and USD**

** Datasmith **
https://docs.unrealengine.com/5.3/en-US/datasmith-plugins-overview/
Collection of tools to import the entire pre-constructed scene into Unreal reusing the already built assets and layouts.
If your source scene contains multiple copies of the same geometry, Datasmith usually creates only one Static Mesh Asset for that object.
Changes you make to any information that Datasmith brings into Unreal from your source application are tracked as overrides.
It tracks the incremental changes from the first time it was imported in a scene, and can be reverted to any change.

** Interchange Framework **
Import and export framework. It is file format agnostic, asynchronous, customizable, and can be used at runtime.
When you reimport an asset that was previously imported using Interchange,
Unreal Engine remembers the pipeline stack and options that were used, and displays those options.

** Lumen **
https://docs.unrealengine.com/5.3/en-US/lumen-technical-details-in-unreal-engine/
Lumen is a fully dynamic global illumination and reflections system.
Lumen renders diffuse interreflection with infinite bounces and indirect specular reflections in large, detailed environments at scales ranging from millimeters to kilometers.
Lumen Global Illumination solves diffuse indirect lighting.
For example, light bouncing diffusely off a surface picks up the color of that surface and reflects the colored light onto other nearby surfaces —
creating an effect called color bleed.
Meshes in the scene also block indirect lighting, which also produces indirect shadowing.
Lumen provides infinite diffuse bounces.
Sky lighting is solved as part of Lumen's Final Gather process.
It includes sky shadowing, allowing indoor space to be much darker than outdoor lighting, providing a much more natural effect.
Lumen solves indirect specular, or reflections, for the full range of material roughness values.
Static light is not suported by Lumen, (like stationary, it located in the same place during gameplay, but unlike stationary, the color hue or intensity is unchangeable, they are built into the lightmap).

** Nanite **
It uses a new internal mesh format and rendering technology to render pixel scale detail and high object counts.
It intelligently does work on only the detail that can be perceived and no more.
Nanite's data format is also highly compressed, and supports fine-grained streaming with automatic level of detail.
https://docs.unrealengine.com/5.3/en-US/nanite-virtualized-geometry-in-unreal-engine/
A Nanite mesh is still essentially a triangle mesh at its core with a lot of level of detail and compression applied to its data.
On top of that, Nanite uses an entirely new system for rendering
Increase in geometry complexity. Directly import film-quality source arts, such as ZBrush sculpts and photogrammetry scans.
During import — meshes are analyzed and broken down into hierarchical clusters of triangle groups.
During rendering — clusters are swapped on the fly at varying levels of detail based on the camera view and connect perfectly without cracks to neighboring clusters within the same object.
Data is streamed in on demand so that only visible detail needs to reside in memory.
Nanite runs in its own rendering pass that completely bypasses traditional draw calls.

** Virtual Shadow Maps **
VSMs is the new shadow mapping method used to deliver consistent, high-resolution shadowing that works with film-quality assets and large, dynamically lit open worlds
Goals:
- Significantly increase shadow resolution to match highly detailed Nanite geometry
- Plausible soft shadows with reasonable, controllable performance costs
- Provide a simple solution that works by default with limited amounts of adjustment needed
- Replace the many Stationary Light shadowing techniques with a single, unified path
Virtual shadow maps are just very high-resolution shadow maps.
In their current implementation, they have a virtual resolution of 16k x 16k pixels.
Clipmaps are used to increase resolution further for Directional Lights.
To keep performance high at reasonable memory cost, VSMs split the shadow map into tiles (or Pages) that are 128x128 each.
Pages are allocated and rendered only as needed to shade on-screen pixels based on an analysis of the depth buffer.
The pages are cached between frames unless they are invalidated by moving objects or light, which further improves performance.

#### UE cons

Engines optimizes towards a new gen graphics targets. Instead developers need to have a controll over specific target optimization (or a library doing it).
https://forums.unrealengine.com/t/new-ue5-4-feedback-start-working-on-and-investing-in-performance-innovations-for-actual-games/1164987

Games, are being bottleneck by the source code engineer’s, not game studios(for the most part).
https://forums.unrealengine.com/t/new-ue5-4-feedback-start-working-on-and-investing-in-performance-innovations-for-actual-games/1164987/16

Heavy object-orientedness (will be discussed in next section)

Nanite is good for complex meshes when doing cinematography and not targeting 60fps. Manual optimization is better.
https://forums.unrealengine.com/t/nanite-performance-is-not-better-than-lods-test-results-fix-your-documentation-epic-youre-ruining-games/1263218?u=_solid_snake

Overall writing scripts feel like writing a new dialect of C++ where you need to learn new macros, naming conventions and the workflow.
This feat forces a programmer to debug the language itself and not the software that he/she writes.

C# for building/dding modules. Thus the need to learn another language.

### 2.1. Engines analysis.

"Often authors present their own architecture as a de facto solution to their specific problem set"[15 p1]

What kind of functionality is provided by those engines, that make a game developer pick this engine.
Quote research questions from [15] and try to answer them while doing a practical example.
- Terminology: The lack of a ‘game development’ language.
- What is a Game Engine: Where is the boundary between game and game engine?
- Game Genre: How do different genres affect the design of a game engine?
- Überengine.
- Design Dependencies: How do low-level issues affect top-level design?
- Best Practice: Are there specific design methods or architectural models that are used, or should be used, for the creation of a game engine?
Where's a line between a complete game and an engine that was used building it?
- Multiprocessing.

"The establishment of clear boundaries between game engine (code) and game code" [15]
since game engine is a collection of software utilities, any code that is used to produced a final executable, from compiler to an animation editor, is - a game engine.

Godot engine describes itself as
"a feature-packed, cross-platform game engine to create 2D and 3D games from a unified interface.
It provides a comprehensive set of common tools, so that users can focus on making games without having to reinvent the wheel.
Games can be exported with one click to a number of platforms, including the major desktop platforms (Linux, macOS, Windows),
mobile platforms (Android, iOS), as well as Web-based platforms and consoles."
Essential, what Godot is a collection of common tools that are used to create games, that can handle the porting for a game developer.
Here, an engine is something, that contains software components, that can be re-used to make a new game.
In Gregory's architecture, porting responsibilities take Platform Independence Layer (PLA)

"...engine code, depth of simulation, and profiling as some of the highly domain-specific requirements that contribute to the difficulty of writing video games.
The author further breaks down engine code into mathematical and algorithmic knowledge and the wisdom to know how the algorithms will interact when coupled together."[7]

Paradox's engine for strategy games like Hearts of Iron
https://venturebeat.com/pc-gaming/the-engine-behind-paradox-development-studios-future-games :

    Clausewitz [older engine] is a bunch of code that you can use to make games.
    You could use it to make a city-builder, a strategy game, an FPS … not that it would give you any tools for that, but you could...
    Jomini [new engine] is specifically for the top-down, map-based games.
    The pair are two halves of the same engine.
    The overais to try and share tech across as many of our projects as possible, so we can make bigger games, better games, faster.
    New Clausewitz-Jomini alliance is the inclusion of proper tools, which will help both designers and moders.
    For the designers, the new tools mean that they can focus on what they’re good at, specialities like art or writing.
    We’ve completely reworked the GUI system. Modders should be able to make super-fancy UIs without editing any code in the game.
    I think it comes down to that: We want to put as much power as we can into the hands of the people who are making content. They shouldn’t have to ask a coder for help.

Doom3 Source code review: https://fabiensanglard.net/doom3/index.php

"As one game developer pointed out to me this morning, even the ubiquitous Unreal Engine 4 is still built on a foundation
that started with the first Unreal, which came out in 1998."[21]

Raylib, SDL2, Bevy, Godot and Mach.
Godot is by far the most pleasant and complete General Purpose engine proposed by taxonomy.

WebGPU.

"To arrange the assets and models in a virtual world and improve the efficiency of rendering, game engines have integrated spatial data structures.
The most common are scene graphs, spatial partitioning, and bounding volume hierarchies (BVHs).
However, each game engine or 3D program has their own implementation of these spatial data structures."[18]

Modding and "the need of a uniform “game language”, but in a much wider context – involving the community aspects of communication as well"[9].
Adding to your game the ability to mod it is like making your game a special-purpose game engine. Minecraft Skyrim and Roblox.
"Bethesda does not release games, it releases modding toolkits" - u/Vellarain

Data mining is used extensively to analyze how players are interacting with the game. [8]

Alternative architectures:
- Data-Driven Object System: https://www.youtube.com/watch?v=Eb4-0M2a9xE&t=4s (2002)
- Game Engine Entity/Object Models: https://www.youtube.com/watch?v=jjEsB611kxs&t=4s

Mention the importance of a multithreading module in the engine. So that doing concurrenc becomes an app's own responsibility and not having to rely on OS thread.

Tips for VR https://blog.unity.com/games/accessible-vr-game-design-tips-from-owlchemy-labs

### 2.2. Programming languages analysis

During my university coursework I've had a hands on experience with C#, C, C++, Rust, JS and Zig. I will explain thier pros and cons for game development.
Important to check the benchmarks [16][17].

C++ started as "C with classes". Object-Oriented Programming is the paradigm on which C++ was started.
https://wordsandbuttons.online/the_real_cpp_killers.html

https://blog.sigma-star.io/2024/01/people-dont-understand-oop/

    “OOP to me means only messaging, local retention and protection and hiding of state-process, and extreme late-binding of all things.”

    Alan Kay (the guy who coined the term “Object Oriented Programming”)1

    OOP vs Functional programming vs procedural

    "Objects are collections of operations that share a state."

https://loglog.games/blog/leaving-rust-gamedev/

As was described by [1] in Theoretical part, C and C++ became popular because of the access to the hardware memory.

Cross-compilation. Windows and Unix. Wine and emulation.

What's Zig got that C, Rust and Go don't have? https://youtu.be/5_oqWE9otaE?si=_R5h5LKoFmP2wAGR

    Q:  New programming languages come into being as a reaction to what's missing in the marketplace.
        There's a burning reason why Zig needed to exist.
    A:  Some programs need to have a very close access to the hardware to run 

The effects of object-oriented technology on performance, executable file size, and optimization techniques for mobile games
and suggested that object-oriented technology should be used with great care because the structured programming in game development is highly competitive.[20 5.2.2.5]

Zig build system allows compiling a binary for a specific target hardware, which partly solves a problem of porting from [7 5.3].

Note about Linux as a more pleasant developing environment.
Potential of NixOS and declarative OSes might help with reproducing development environment.
Any professional needs a sharp tool. A software developer needs to handle his editor the same way an NB player handles a basketball.
Benefits of modal editing and nvim.
"Open-source development platforms are available, but developers must customize them according to the required functionality." [20 5.2.2.3]
"...Windows was not meant for the type of development necessary for video games.
Microsoft Visual Studio was built for Visual Basic and C# mainly, not for C++, but the only viable C++ compiler and environment on Windows remains Visual C++." [7]

Cargo cult programming and software development meta.

### 2.3. Practical example

I don't really consider myself a "programmer".

#### glTF vs USD

https://www.ec4labs.io/post/universal-scene-description-gltf-what-is-the-deal-here
glTF was designed specifically for real-time 3D graphics and is optimized for efficient storage and fast transfer of 3D assets.
On the other hand, USD was designed for large-scale, complex productions that require sophisticated data management and cross-application collaboration.
glTF is designed for fast performance and low overhead, making it ideal for use in real-time graphics and gaming.
USD, on the other hand, is designed to support large-scale, complex productions, and may have a higher overhead compared to glTF.

On the importance to make games during the game engine’s development.
https://www.team-nutshell.dev/nutshellengine/articles/making-games-during-development.html

Find on https://github.com/zig-gamedev/zig-gamedev from a list of libs those that are required by an engine architecture proposed by Gregory.
Pull all the dependencies and compile without the errors.
Write 2D games: Snake and Breakout and an 3D scene with a player.
Make them run on Windows.

A note about the EUGPL licencse.
"open-source is the right path to follow.
It democratizes and allow a soft learning curve for beginners. Also, it will allow the creation of more diverse games."[1 6.4]
[7] mentions domain specific knowledge required for engine development. Open source game/game engine with clearly distinct modules is the optimal environment for
a developer who can apply (or try to apply) domain-specific, deeply specialized knowledge.

## 3. Results

A game engine is primarily a collection of software tools. A swordsman needs to be proficient with a sword and make sure it's sharp.
Thus, the best game engine is the one that non-software developers don't even need to know anything about.
A 3D modeler is more productive with his 3D modelling software than with any sort of engine. The same goes for animators, artists, sound designers etc.

An engine becomes more useful when more games has been done with it.
So that a game can re-use the maximum amount of engine features that were introduced by a previous game in the same genre.
That's how Unity became so popular. As more people started to make games with it, more assets in marketplace and more tutorials on how to make games in it.

Development of a game should start from the highest point, not the lowest.
For example, a new shooter game should start from doing minor tweaks to the another "completed" shooter game, changing skybox, or skins of the enemies.
Then gradually, developers need to tap into underlying systems and bring more noticeable changes.
This will minimize the development cycle to the absolute minimum, since a game can be considered "completed" after each sprint.
Theseus ship in action. Repurposing an old ship vs building a new one.

### Discussion

Messaoudi, Simon, and Ksentini (2015) investigate the game 
engine Unity and from a technical standpoint measure its performance using a number 
of tests straining the CPU and GPU. The authors also discuss a difference between a 
game engine as a production tool and a game engine as the run-time executable that 
“drives” the game. They propose to use the term game engine to describe the 
executable, and the toolset that is used to build the games should be referred to as the 
“framework” (Messaoudi et al., 2015). Anderson et al. (2008) are also mentioning the 
misunderstanding in terminology and points to the fact that there is a lack of a “game 
development language”. One of the examples regarding terminology is the “game 
engine” term. Anderson et al. (2008) reflect on the boundaries of a game engine: (1) 
the game itself is confused with the game engine and (2) if the toolset is a part of the 
game engine. Other questions raised includes if it is “…possible to define a game 
engine independently of genre”, i.e. having a game engine that can be so flexible to 
accommodate for games in all genres – an “überengine” (Anderson et al., 2008, p. 229). 
O'Donnell (2014) also reflects upon the boundaries between games and engine, with a 
description of a game engine as “[t]he underlying software of the game. The engine is 
not a game; it is more basic than that. It provides a platform, to which more code, data 
and art assets are produced to make a game”. [9 p8]

Do we need to draw the boundaries between a game, an engine and a framework?

Standardization of engines and protocol?

## 4. Conclusion

Game Design document need to specify more information about the game systems, so that an engine has more information to work with.

## 5. References

[1] Are Game Engines Software Frameworks? A Three-perspective Study
[2] https://www.indiegamewebsite.com/2018/10/19/the-complete-history-of-indie-games
[3] https://download.intel.com/newsroom/2022/new-technologies/ann-kelleher-iedm-2022.pdf
[4] (https://www.marketwatch.com/story/moores-laws-dead-nvidia-ceo-jensen-says-in-justifying-gaming-card-price-hike-11663798618)
    Witkowski, Wallace (September 22, 2022). "'Moore's Law's dead,' Nvidia CEO Jensen Huang says in justifying gaming-card price hike". MarketWatch. Retrieved September 23, 2022.
[5] Game Development Software Engineering Process Life
[6] Dataset of Video Game Development Problems
[7] A Holistic Look at Requirements Engineering in the Gaming Industry
[8] A Python Engine for Teaching Artificial Intelligence in games
[9] A Taxonomy of Game Engines and the Tools that Drive the Industry
[10] Game Engine Comparative Anatomy
[11] https://docs.unity3d.com/Manual/pack-core.html
[12] Jason Gregory. Game engine architecture. Taylor & Francis, CRC Press, Boca Raton, 3 edition, 2018. ISBN 978-1-138-03545-4.
[13] An Exploratory Approach for Game Engine Architecture Recovery
[14] Visualizing Game Engine Subsystem Coupling Patterns
[15] The Case for Research in Game Engine Architecture
[16] https://programming-language-benchmarks.vercel.app
[17] https://zserge.com/posts/better-c-benchmark
[18] IMMERSIVE EXPERIENCES AND XR: A GAME ENGINE OR MULTIMEDIA STREAMING PROBLEM?
[19] A Python Engine for Teaching Artificial Intelligence in games
[20] Game Development Software Engineering Process Life Cycle: A Systematic Review
[21] The Controversy Over Bethesda's 'Game Engine' Is Misguided. https://archive.org/details/the-controversy-over-bethesdas-engine
[22] Godot Docs v4.2. https://docs.godotengine.org/en/4.2/
[23] Stroustrup, Bjarne (7 March 2010). "Bjarne Stroustrup's FAQ: When was C++ invented?". https://web.archive.org/web/20160206214150/http://www.stroustrup.com/bs_faq.html#invention
