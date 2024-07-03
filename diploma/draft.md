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

- Section 1.1 introduces us with the research in this field done so far;
- Section 1.2 is a theoretical part which explains some of the software development concepts related to game development;
- Section 2.1 compares Unity and Unreal Engine;
- Section 2.2 analyzes existing game engines;
- Section 2.3 explains the importance of a programming language;
- Section 2.4 evaluates the feasibility of creating games using a Zig language;
- Section 3 discusses our results and threats to their validity;
- Section 4 concludes.


### 1.1 Theoretical part

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
He proposes a "Runtime Engine Architecture", in which a game engine consists of 15 runtime components with a clear dependency hierarchy.
We take a close look at them and debate possible modifications in Section 2.2.
This book finds an optimal middle ground between a high-level, broad description of how those modules interact with each other, and their detailed implementations.
Most of the examples are written in C++ and are object-oriented in nature, however.

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
There are, however, other architectures used in proprietary mainstream engines, of which very little is known (Section 2.2).
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
[20] describes common problems faced during pre-production, production and post-production phases.

[1] explores whether game engines share similar characteristics with software frameworks by comparing 282 most popular open source engines and 282 most popular frameworks on GitHub.
"Game engines are slightly larger in terms of size and complexity and less popular and engaging than traditional frameworks.
The programming languages of game engines differ greatly from that of traditional frameworks.
Game engine projects have shorter histories with less releases.
Developers perceive game engines as different from traditional frameworks and claim that engines need special treatments."
Game developers often add scripting capabilities to their engines to ease the design and testing workflow,
meanwhile frameworks products are often written in the same programming languages.
They also explain that the reason for popularity of C family of languages is that "...engines must work close to the hardware and manage memory for performance.
Low-level, compiled languages allow developers to control fully the hardware and memory."
For C++ in particular, following features is what made it so dominant:
"abstraction, performance, memory management, platforms support, existing libraries, and community". [1 6.3]

Toftedahl and Engström in their work discovered that as of 2018 "Unity is the engine used in approximately 47 % of the games published on Itch.io."[9]
While on Steam it comes as second (13.2%) after Unreal Engine (25.6%).
They categorize game engines into 4 types:
- Core Game Engine: collection of product facing tools used to compile games to be executed on target platforms. e.g. id Tech 3, Unity core;
- Game Engine: a piece of software that contains a core engine and an arbitrary number of user facing tools e.g Raylib, Bevy;
- General Purpose Game Engine: a game engine targeted at a broad range of game genres e.g. Unity, Unreal, Godot;
- Special Purpose Game Engine: a game engine targeted at specific game genres e.g. GameMaker, Twine.

Since they did not quantify the exact amount of user facing tools required for a Game Engine, I don't think they performed a diligent job.
In theory, any single utility added to a Core Game Engine makes it a Game Engine.
Additionally, their definition of Core Game Engine is in contradiction with Gregory's "Runtime Engine Architecture" where Core files are "useful software utilities" [12, 1.6.6].
Unity core contains several packages including a renderer and UI [11], meanwhile Gregory has a "Low-level Renderer" and "Front End" separate from "Core".
As for the special purpose game engine, this can be considered as an engine that can only compile one kind of game but with different configurations.
In which case, a mod for Minecraft can be considered a game that was produced using Minecraft engine.

Before we move on to the research, let's brief ourselves with needed terminology and a short history.

### 1.2 Terminology

So what is a game engine? "John Carmack, and to a less degree John Romero, are credited for the creation and adoption of the term game engine.
In the early 90s, they created the first game engine to separate the concerns between the game code and its assets and to work collaboratively on the game as a team."[1]
A game became known as Doom, and it gave birth to the first-ever "modding" community.[12 1.3]
Thus, a new game could be made by simply changing game files and adding new art, environment, characters etc. inside a Doom's engine.
It is hard to draw a line and try to fit this engine into a taxonomy proposed by Toftedahl and Engström as
it is both a Special Purpose engine, that creates a game in a specific genre; and a Core engine, that provides useful software utilities.

A general definition for a game engine that [9] provides after inspecting some of the popular engine's websites is as follows:
"A framework for game development where a plethora of functions to support game development is gathered".

Jason Schreier claims that a word "engine" is a misnomer:
"An engine isn’t a single program or piece of technology — it’s a collection of software and tools that are changing constantly.
To say that Starfield and Fallout 76 are using the “same engine” because they might share an editor and other common traits
is like saying Indian and Chinese meals are identical because they both feature chicken and rice".[21]

Lastly, Politowsky et al. gathered multiple definitions of "game engine":
1) collection[s] of modules of simulation code that do not directly specify the game’s behavior (game logic) or game’s environment (level data);
2) "frameworks comprised of different tools, utilities, and interfaces that hide the low-level details of the implementations of games.
    Engines are extensible software that can be used as the foundations for many different games without major changes and are “software frameworks for game development”.
    They relieve developers so that they can focus on other aspects of game development” [1].

In brief, they all imply that an engine is foremost a collection of software tools.
A code that can be reused, and allows game developers to start working on an important problem instead of writing general utilities from scratch.
It is my belief that a "game engine" is an imaginary concept that is used as an umbrella term for software tools that help with creating video games.
Therefore, for the purpose of this paper, "game engine" will imply an ecosystem around software development, which helps with producing video games in one way or another.
Development environment potentially has a significant influence on the quality of a final product, and so cannot be ignored when designing a modern engine.

Video game modding (short for "modification") is the process of alteration by players or fans of one or more aspects of a video game, such as how it looks or behaves. [45]
Mods may range from small changes and tweaks to complete overhauls, and can extend the replay value and interest of the game.
We cover this topic in chapter 2.2.
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
For this and other purposes, an engine can integrate a scripting language.
Such language is more limiting than the language an engine is written in, but it allows skipping a compilation step every time there is a new change to the game.
Scripting language provides only a subset of features provided by an engine.
A scripting language can be virtually anything, from a simple custom parser to full-fledged languages like C#, Lua or Python.

Now with the theory covered, we take a look at some of the popular game engines, and gather more data from concurrent blog posts and articles.

## 2. Research

According to Toftedahl and Engström, Unreal and Unity are currently the most widely used game engines [9].
The worldwide demand for Unreal Engine skills is expected to grow 138% over 10 years. [28]
Therefore, it is, crucial to understand what makes both of those engines appealing towards game developers.

During my university years I've had the experience of making a game with following engines: Unity, UE4, Raylib, SDL2, OpenGL + glfw, Bevy, Godot and Mach.
Some of them are considered general purpose, while others only provide the most basic tools that allow to draw pixels onto the screen.

We begin this research by answering what set of features provided by Unity and Unreal appeals to game developers and make those features our guidelines.
After finding the most important ones, we take a look at open-source frameworks mentioned last paragraph and analyze them based on those guidelines.
The difference in analysis between Unity and Unreal and their open-source counterparts will be in our point of view.
First ones will be viewed from the point of view of a regular game developer, while the rest from both game developer and an engine developer perspective.
Then we compare the languages used in all of those engines and attempt to determine whether any of the language peculiarities have had a direct impact on engine's architecture.
The final result of this research is an engine prototype.

Creating a game engine is an integral part of a learning process:
"The only way for a developer to understand the way certain components work and communicate is to create his/her own computer game engine."[13]
The second-biggest reason why new engines are being developed is because of the learning purposes. With first reason needing more control over the environment.
The actual need to create a game is only on a third place [1].
Although this trivial reason only comes at a third place, as game developers, we must battle test our engines by making, however trivial, games with them.
Not ignoring artistic aspects of game development can help us while writing a game engine.
If we get too far into engine development without ever making games with it, we might discover that our engine cannot make games.
There is a difference between testing new features in an empty scene vs implementing them into something with actual gameplay, and distributing to other platforms. [24]

### 2.1 Unity and Unreal

There are many reasons why a certain product can gain popularity.
For starters, both Unity and Unreal have entered a market quite early.
Unity's first release was in 2005, and it received a lot of attention by being completely free to use, while first version of Unreal was announced back in 1998. [31]
20 years is ample time for a product to attract users and determine the hierarchy of consumer needs.
Nowadays, both engines are used for more than mere vide game creation.
Unreal is often being hailed as the future of filmmaking. Movie industry benefits from using it because working in real time makes things more dynamic.
One can quickly visualize scenes, frame shots and experiment with the film's "world" before committing to filming.
"Unreal Engine is an incredible catalyst for world building". [27]
Pre-visualisation is an area where efficiency is of utmost importance.
Results need to be handled quickly and artists must respond instantly to last-minute changes.
To achieve success in such environment, artists need tools that will deliver instant feedback.
Unity assists artists with their dedicated AR tools, which come pre-configured with good default options and can be used straight out of the box.
"This includes a purpose-built framework, Mars tools, and an XR Interaction Toolkit". [28]

Both products have helped with lowering the skill floor for beginner game developers, and this technical barrier only continues getting lower.
Not only new games are more graphically superior but also are much easier to make for less-technical artists.
"The trend is moving towards more and more automation of the process, and towards improving the quality of the product at the final stage." [29]
If this pattern proves to hold, it will mean that fewer games will be made from scratch, and more reusable assets (from code to art) will be recycled to make new games.
For a smooth experience of asset reuse, a solid marketplace will be needed.
"One of the biggest opportunities is how real-time engines facilitate exchanges via the marketplace". [29]
Indeed, one prominent advantage that both engines seem to possess is an online marketplace.
An artist can scroll a catalog of assets and make a game prototype in a short time span.

Despite the abundance of both free and paid 3D models available on the market, the world still needs more 3D designers.
"In the world of 3D design, demand is so high that there is more work to do than there are designers to do it". [30]
Perfect photorealism is becoming both more accessible and more common in design.
In the future, whether filmmakers will go for a traditional camera shoot or CGI implementation will be a question of budget. [30]
Consequentially, with the rise in demand for 3D art there also rises a demand for a software which allows working with it. 

#### Unity

In his overview of Unity, Gregory writes that the ease of development and cross-platform capabilities are its major strengths. [12 1.5.9]
Upon further investigation of the features of this engine, all the pros can be broken down into 3 rough categories: *portability, versatility and community*.

Most of the sources that construe Unity's popularity highlight how cross-platform support greatly assists during both production and post-production phases.
"...it [Unity] stands out due to its efficiency as well as its multitude of settings for publishing digital games across multiple platforms...
developers select and focus their efforts on developing code on a specific platform
without spending hours configuring implementations to make the application run on other platforms." [31]
Unity game can be run on more than 20 different platforms, including all major desktops, Android, iOS,
consoles like PlayStation 4 (including PS VR), PlayStation 5 (including PS VR2), Xbox One, Xbox Series S|X, Nintendo Switch and web browsers. [33]
Apart from being able to cross-compile a game, developers are also given a powerful suite of tools for analyzing and optimizing a game for each target platform. [12 1.5.9]

This might be the paramount reason why Unity has withstood the test of time and managed to retain their user base.
According to the data published by Unity Technologies, 71% of the mobile games are Unity-based. [25]
The ability to run a game on whichever device a player might possess significantly increases the chances of your game being picked up and played.
Especially as cross-platform games gain more and more popularity with each year. [34]
"The possibility of fully integrating games into web browsers, at a time when mobile devices have dominated the market, is probably the future of many digital games" [31]

Unity's versatility has extended its reach beyond regular game development,
finding application in the fields that require real-time simulation such as the automotive industry, architecture, healthcare, military and film production. [35]
A good example of a "do it all" software: a seemingly bottomless toolbox that contains the instruments useful for any professional in any industry.
There is no kind of game you cannot create with it, be it 2D, 2.5D, 3D, XR etc. [31]
Pixel art or photorealism, card game or platformer, fps or RPG. It does not matter.
Its user interface is simple enough to start prototyping without coding.
Coding part can be omitted entirely with visual scripting provided by Bolt. [25]
"All you should know is just how to make game art. Everything else can easily be done with Unity’s tools." [34]

Those tools include XR instruments (60% of AR and VR content is made with Unity [35]), Built-in Analytics (Real-time data and prebuilt dashboards).
Game Server Hosting, Matchmaker and Cloud Content Delivery allow developer to create multiplayer games and host them on a web server.
Unity Build Automation and Unity Version Control can take care of DevOps aspects of game development.
(47% of indies and 59% of midsize studios started to use DevOps to release quickly)
Asset Manager (in beta as of writing this paper) helps with managing a project's 3D assets and visualizing them in a web viewer. [34]
This is only a tip of an iceberg, and for a full list of available features you can always refer to the official documentation.
Not only those features are there, Unity also releases frequent updates accompanied by major fixes to software security vulnerabilities.
To top it all off, a scripting language C# provides even more versatility with the ability to support both the back-end (Azure SQL Server with .NET)
and the front-end (Asp.Net) of the application. [31]
Unity's adaptability to modern trends is simply fascinating and is overall a good textbook example on how to keep a game engine up to date.

Lastly, a reason that is not exclusive to the game development and can also be equally perceived as the consequence of the two previous factors is the community.
More specifically marketplace and learning materials.
Unity Asset Store offers a vast collection of assets, plugins, and tools that can be easily integrated into projects, saving time and enhancing development efficiency.
If there is some kind of tool one might need, someone has likely made it already and sells it for a cheap price on the asset store.
When starting a new project, teams don't need to build games from scratch as many of the prerequisites can be simply acquired from there
and be imported into a project in a matter of few clicks.
"It’s better to spend a hundred dollars on the Unity Asset Store instead of doing something that would cost twenty or thirty thousand dollars and two to three months to develop." [34]
By staying on the market for so long and being used in plenty of projects,
it is no surprise that abundance of tutorials and code snippets produced by helpful game developers can be found online.
The amount of learning materials for Unity exceeds those of any other engines out there, even Unreal. [32]
We will talk more about the importance of the community and game modding in a next chapter.

Now let's discuss Unity's shortcomings.
Biggest one stems from its desire to be versatile.
A freshly created, empty 2D project can weight up to 1.25 GB, and PackageCache folder up to 1 GB.
Although certain dependencies can be manually removed from "Packages/manifest.json",
most of the modules are deeply intertwined with Unity's core, and removing them would be unwise and could lead to bugs.
This bulkiness is fine for the modern hardware, but a resource-hungry behemoth is not ideal when a game needs to be run on an older hardware or power-efficient devices.
Unity was found to have issues with CPU and GPU consumption and modules related to rendering. [31]
Not everyone who wants to play video games can afford a modern computer or a phone.
Unity also doesn't support linking external libraries. [25] This factor cripples its possibility to be modular.
Additionally, with constant technological innovations, computing power can be placed into many unconventional targets like smart fridges, digital watches, thermostats etc.
But "While Unity's cross-platform support is a significant advantage, it can also lead to suboptimal performance on certain platforms.
Games developed in Unity may not perform as well as those developed using native game engines specifically designed for a particular platform." [32]
Being able to play a Snake game on a toaster would be a neat little feature worthy of buying it.

Moreover, despite showing a great adaptability, Unity Technologies's leadership sometimes make for-profit business decisions that are often not perceived well by the community,
such as one of their latest announcements about a "Runtime Fee", which will charge developers each time a game using the engine is downloaded. [36]
This decision was harshly rebuked by community and was eventually changed to apply only to games created with Unity Pro and Unity Enterprise.
Nonetheless, this company has shown themselves capable of trying to pull the rug out from under developers
and not hesitating to squeeze them into debt with per/install (released game) royalties and other loathsome shenanigans.
Problems like this are absent, as a rule, in open-source game engines, some of which we discuss later.

Lastly, Unity might suffer from an identity crisis.
It is supposed to be a "build anything" engine used by both indie and triple-A studios alike,
however many indie studios do not need "anything", they simply need the tools to make a game in a particular genre.
Trying to branch out into too many spaces at once instead of focusing on what they are good at is a sign of a capitalistic greed, and it might bite them in the future.
When the abundance of specialized software exists - software that is designed to excel at a small subset of features present in Unity,
why would a company that only need those specific features pick Unity?

#### Unreal Engine 5

Similar to Unity, Unreal Engine can be used for much broader purposes than just to create video games.
It is more fitting to think about it as a real-time digital creation platform used for creating games, visualizations, generating VFX and more. [37]
In particular, filming industry has been eagerly adopting it in recent years, being successfully used in numerous popular movies and TV series like
"Fallout", "Love, Death + Robots", "Mandalorian", "House of the Dragon" and many more. [38][27]
Virtual Camera system allows to control cameras inside the engine using an iPad Pro. [37]
Gregory claims that this engine has the best tools and richest engine feature sets in the industry, and that it can create virtually any kind of 3D game with stunning visuals. [12 1.5.2]
Unlike Unity, however, its huge arsenal of tools does not seem to be forcing a company to grow in every direction all at once.
Unreal's identity is strikingly clear - its job is to produce the best-looking graphics possible.
It is also mindful that plenty of AAA studio with large teams are using this engine and that team roles there are more diverse than in smaller studios, so the tools that are given
to the artists vary in degree of complexity but are rather artist-friendly, and the workflow for meshes and materials is similar to that of native 3D software like Blender or Maya.
Visual scripting in a form of Blueprints is an integral part of an engine's world editor (unlike visual scripting solutions in Unity that exist only as external plugins).
Built-in version control system allows to include a new team member into a project and easily merge new changes into a scene that a team is currently working on.
Arguably Epic Games' most famous project Fortnite not only showcases some of the extents of its engine, but also serves as a platform for game development with Unreal Editor for Fortnite (UEFN).
Games created with UEFN can be directly published to the Fortnite platform, reaching a built-in audience of millions. [40]

Undoubtedly, the primary driver behind Unreal Engine's widespread adoption is its ability to produce stunning visuals.
Let's delve deeper into these graphics-centric advancements.
Three of the most talked about technologies are *Lumen, Nanite and Virtual Shadow Maps*.
![Lumen](lumen.jpg)

Lumen is a global illumination and reflections system. It is fully dynamic which means there is no need for light baking.
Its "primary shipping target is to support large, open worlds running at 60 frames per second (FPS) on next-generation consoles." [41]
It does not work on PlayStation 4 and Xbox One. Lumen provides 2 methods for ray tracing: software ray tracing and hardware ray tracing.
Software ray tracing is more limiting as there are restrictions on what kind of geometry and materials can be used e.g. no skinned meshes.
Lumen Scene operates on the world around the camera, hence fast camera movement will cause Lumen Scene updating to fall behind where the camera is looking,
causing indirect lighting to pop in as it catches up.
This system works by parameterizing surfaces in a scene into Surface Cache which will be used to quickly look up lighting at ray hit points.
Material properties for each mesh in a scene are captured from multiple angles and populate Surface Cache.
Lumen calculates direct and indirect lighting for these surface positions, including sky lighting.
For example, light bouncing diffusely off a surface picks up the color of that surface and reflects the colored light onto other nearby surfaces, also known as color bleed.
Diffuse bounces are infinite, but Lumen Scene only covers 200 meters (m) from the camera position (when using software ray tracing).
For the fullest potential of Lumen, it is recommended to use hardware-accelerated ray tracing, but this raises a question whether Lumen's success lies with its software innovation
or purely from the utilization of new features provided by the latest hardware of graphics cards.

Nanite is a Level Of Detail (LOD) system. "A Nanite mesh is still essentially a triangle mesh at its core with a lot of level of detail and compression applied to its data." [41]
Nanite can handle orders of magnitude more triangles and instances than is possible for traditionally rendered geometry.
When mesh is first imported into a scene, it is analyzed and broken down into hierarchical clusters of triangle groups.
This technique that might be similar to Binary Space Partitioning.
"During rendering clusters are swapped on the fly at varying levels of detail based on the camera view
and connect perfectly without cracks to neighboring clusters within the same object." [41]
Data is streamed in on demand so that only visible detail needs to reside in memory.
Nanite runs in its own rendering pass that completely bypasses traditional draw calls.
My suspicion is that this process is similar to regular batching but with some extra complexity.
Overall, this addition to an engine allows to build extremely complex open worlds with astonishing amount of details.
Filming industry can abuse Nanite by importing high fidelity art sources like ZBrush sculpts and photogrammetry scans.
Although it is good for complex meshes when doing cinematography,
there are struggles with rendering a game at 60 FPS, in which case a manual optimization of LODs is preferred over relying on automatic LODs from Nanite. [42]

Virtual Shadow Maps (VSM) is a shadow mapping method used to deliver consistent, high-resolution shadowing.
They need to exist in order to match highly detailed Nanite geometry.
Their goal is to replace many stationary lights with a single, unified path.
At the core they are regular shadow maps but with high resolution (16k x 16k pixels).
However, in order to keep performance high at reasonable memory cost, VSMs are split into Pages (small tiles) that are 128x128 each.
Pages are allocated and rendered only as needed to shade on-screen pixels based on an analysis of the depth buffer.
The pages are cached between frames unless they are invalidated by moving objects or light, which further improves performance. [41]
Since VSMs rely on Nanite, they might suffer from similar performance issues.

There are many more fascinating tools and features in this engine like Datasmith (import entire pre-constructed scenes),
Niagara (VFX system), Control Rig (character animations), Unreal Insights (profiling)
and an elaborate online subsystem which aids with handling asynchronous communication with a variety of online services.
Unfortunately, the review of those modules as well as popular plugins is a topic on its own and deserves a separate paper.
We do take a brief, high level look at the architecture of its codebase, however, before exploring what drawbacks does this engine have.

Unreal's build system is called UnrealBuildTool. A game is a target built with it, and it comprises from C++ modules, that implement a certain area of functionality.
Code in each module can use other modules by referencing them in their build rules, which are C# scripts. This system is not much different from CMake, autotools, meson etc.
Modules are divided into 3 categories: Runtime, Editor functionality, and Developer tools.
Gameplay related functionality is spread throughout Runtime modules, some of the most commonly used are:

- Core: "a common framework for Unreal modules to communicate; a standard set of types, a math library, a container library, and a lot of the hardware abstraction" [41]
- CoreUObject: a base class for all managed objects that can be integrated with the editor. It's the central object in the whole object-oriented model of an engine.
- Engine: functionality associated with a game (game world, actors, characters, physics, special effects, meshes etc.)

Modules enforce good code separation, and although it does not necessarily make Unreal modular in a traditional sense,
developers are able to specify when specific modules need to be loaded and unloaded at runtime based on certain conditions.
Include What You Use (IWYU) option further helps with compilation speeds - every file includes only what it needs,
instead of including monolithic header files, such as Engine.h or UnrealEd.h and their corresponding source files.
As for the gameplay programming, scripts are written in C++ like an engine itself, and they use inheritance to expand the functionality of new objects (Actors).
There is a visual scripting language called Blueprints that allows to create classes, functions, and variables in the Unreal Editor.
Another option for scripting is Python, however it is still in experimental stage. It is a good option when one needs to automate his workflows within the Unreal Editor.

As for the drawbacks of this engine, perhaps the biggest one would be the consequence of trying to optimize toward the next generation graphics targets.
All those innovative systems listed earlier cannot be fully taken advantage of if a game is being run on moderately powerful graphics cards prior to 20 series. [43]
Out of all game engines we discuss in this paper, Unreal editor was the most resource hungry and caused frequent stuttering.
These performance issues make it hard to recommend this engine for developers with a limited budget and who want to target a variety of dated platforms.

Another noticeable issue arises when working with C++. Oftentimes writing scripts feels like writing a new dialect of C++
where one needs to learn specific macros and naming conventions.
A lot of Unreal's code has poor encapsulation. 
Many member variables are public in order to, presumable, support the editor and blueprint functionality.
Some public member functions should never be called by users, like the ones that manage the "lifetimes" of objects, but they are called by other modules of an engine.
This clutters the interface of many objects, making it difficult to figure out what are the important interface features of unfamiliar classes.
It is also difficult to make use of RAII due to how Unreal Engine handles the creation of UObject-derived objects.
Managing the state of the objects via initialization/uninitialization functions requires more caution compared to traditional C++,
and it is not uncommon for complex objects to have certain parts with a valid state while the rest is not.

Other cons are less problematic and are rather subjective. For example people on forums claim that the documentation is lacking.
However, what they imply is that either API reference is outdated (which is often the case for massive projects with frequent release cycles),
or that a manual does not explain certain game development concepts in enough details, which is hardly an engine's fault.
A steep learning curve is not a problem but a price that game developers need to pay to work on next generation games.
Tim Sweeney, a founder of Unreal Engine says "You can't treat ease-of-use as a stand-alone concept.
It's no good if the tools are super easy to get started with, so you can start building a game,
but they're super hard to finish the game with, because they impose workflow burdens or limited functionality." [39]
Workflow for 2D games is not great, but from the very start of this engine, it was focused on developing shooters or first-person games. [31]
2D is simply not its specialization to begin with.
A need to use C# for build scripts is a peculiar decision, but there is no universal way to build C++ applications and probably never will be,
this is a problem of the infrastructure around the language of choice and not of the engine.
Lastly a marketplace is big, and it is integrated very well into the engine and Epic store, but it certainly could be improved.
There are not enough filters to search for assets, and many of them are either unfinished, broken or have licensing issues which makes them impossible to use in commercial products.

#### Takeaways

Unity and Unreal stand as titans in the game development landscape.
Their long histories as free game engines make it a challenge for newcomers to enter the market.
Both engines implement a royalty system for commercially successful games (Unreal at 5% above $1 million USD in sales, and Unity with variable plans).
Source code of both engines is proprietary, but can be accessed nonetheless: Unreal on GitHub and Unity by paying for Enterprise or Industry editions.
Games made with Unreal seem to scale up better than Unity, as Epic appears to have built a more robust infrastructure around the whole process of game development
instead of simply giving developers the tools to make vide games.
Meanwhile, Unity struggles with handling AAA games with large landscapes and heavy on-screen elements. [34]
Yet it appears to be a better choice for newcomers, as the skill floor is lower and the amount of learning materials (mostly community generated) is higher.
It is also an engine of choice for serious games, as a final game can be easily exported to more platforms (including web browsers)
with a price of using more computational resources, and the diversity of possible game genres is superior to that of Unreal.

Neither engine, however, provides any meaningful tools to solve the problems of the pre-production phase of game development that were described in [20].
Requirements specifications for emotions, gameplay, aesthetics and immersion exist outside the engine, which makes developers not responsible for adhering to those requirements.
This opens an opportunity for incorporating high level game system description languages (external schema) into the engine,
which in its turn requires Game Design Documents to be an integral part of an engine.

Importantly to the objective of this paper, we distinguished 4 key elements responsible for the popularity of a game engine:

1. *Portability*. Developers want to work on games instead of figuring out how to port them to certain platforms.
The more target platforms a game engine support - the more likely it is to be used.
2. *Versatility*. Nobody knows how a final game will look like from the start and what systems will it need.
Having every possible tool available is a safety precaution that allows the flexibility during the production phase.
3. *Graphics*. Even when visual fidelity is not the primary goal, everybody wants their games to look beautiful.
4. *Community*. Games are not created in isolation from scratch.
A good engine needs to have a straightforward way of integrating community generated content, be it code snippets or assets.
For these purposes, it is crucial to have a first-class support for developing external plugins and assets via a marketplace.
Knowledge needs to be passed and re-applied, for purpose of which a game engine is served as a playground to battle test new ideas and techniques.

Now with those four criteria in mind, we compare some of the popular open-source engines as well as try to find what other points need to be considered when designing a game engine.

---
Draft
---

### 2.2. Engines analysis.

Before we begin our investigation, first we need to break down runtime architecture (RTEA) proposed by Gregory and define responsibilities of each module.
Then we compare and contrast architectures of open source game engines/frameworks mentioned at the start of last chapter with RTEA.
Afterwards, we expand the list of responsibilities of a theoretical uberengine and debate what kind of functionality should or should not be a part of an engine.
![rtea](rtea.png)

First distinction Gregory makes is separating a tool suite from runtime components. [12 1.6]
Tools include a software used for asset creation like Blender and Maya for 3D meshes and animations, Krita and Photoshop for textures, Ardour and Reaper for audio clips etc.
Those digital content creation (DCC) tools are clearly separate from the engine, and so are not a part of the RTEA.
The same goes for the version control system and instruments around a programming language (compiler, linker, IDE).
Yet it becomes less evident with other elements which ones should be a part of the RTEA and which ones should belong to a tool suite.
For example performance analysis tools (profilers) vary in degree of complexity:
some of them are standalone programs that only need an executable compiled without any extra flags (Valgrind),
while others are designed to be injected into code to profile certain parts of a program (Tracy).
Similarly, debugging can be done either with an executable using GDB, or a primitive console/line-drawing debugging can be performed manually for a quick and dirty visual feedback.
Even operating system submodule is ambiguous and cannot be easily classified into neither a tool suite nor RTEA,
as it provides a slew of drivers, system packages and a libc,
while simultaneously being the foremost important tool for any artist or developer and the core dependency of every content creation software.
Another element of a tool suite is the asset conditioning pipeline. It is the collection of operations performed on asset files to make them suitable for use in the engine
by either converting them into a standardized format, a custom format supported by the engine or a binary.
To put it simply, it is a file compression/decompression system.
The issue with this module is that many DCC software allow for a custom plugin integration that will help with exporting the right format,
eliminating the asset conditioning pipeline altogether, making this module entirely arbitrary depending on a game's complexity.
Resource management submodule baffles even Gregory himself with its ambiguity and leaves him confused as to where to put it.
It provides an interface for accessing different kinds of game assets, configuration files and other input data, making it a logical cog in the machinery of the engine.
However, to scale a game up painlessly, an engine needs a database in order to manage all metadata attached to the assets.
This database might take a shape of either an external relational database e.g. NoSQL or a manual management of text files,
which again makes it unclear whether it should belong to a tool suite or be a runtime component.
Lastly, the icing on the cake, a cherry on top, a roof for the house that is any modern game engine is the World Editor.
"The game world is where everything in a game engine comes together." [12 1.7.3]
Consequentially, to have a visual world editor an engine needs to be logically assembled, which makes this submodule to reside at the top of the dependency hierarchy.
It can be architected either as an independent piece of software, built on top of lower layer modules, or even be integrated into the game itself.
Weirdly, Gregory classify it as a part of the tool suite.

Runtime components are what makes up the rest of the typical engine, and normally they are structured in layers with a clear dependency hierarchy,
where an upper layer depends on a lower level.
Similarly to a tool suite, it is not always obvious which category a certain submodule belongs to.
Third-party SDKs and middleware (SDK) module lays at the lowest layer of RTEA, after an operating system (OS).
SDK is the collection of external libraries or APIs that an engine depends on to implement its other modules.
This might include container data structures and algorithms; abstraction over graphics APIs; collision and physics system; animation handling etc.
Platform Independent Layer (PLA) acts as a shield for the rest of the engine by wrapping platform specific functions into more general, platform-agnostic ones.
By doing so, the rest of the engine gets to use a consistent API across multiple targeted hardware platforms,
as well as having the ability to switch certain parts of SDK with different ones.
Core Systems' (COR) exact definition is vague: "useful software utilities", but some of its responsibilities are:
- Assertions (error checking);
- Memory management (malloc and free);
- Math;
- Common data structures and algorithms
- Random number generation
- Unit testing
- Document parsing
- etc.

Such ambiguity inadvertently raises a question whether a standard library provided by a language can be considered a Core engine system.
We discuss this possibility in Section 2.3.
Resource Manager (RES), as we mentioned earlier, is firstly arbitrary, since many engines tend to delegate the logic of accessing game assets to game programmers;
and secondly, usually in bigger engines for bigger games, resource management is divided between RTEA and a tool suite.
Those 4 components (SDK, PLA, COR, RES) and potentially OS form the base for any game engine, even in simple cases when entire functionality resides in a single file.

Another essential collection of modules can be grouped into a rendering engine.
Rendering is the most complex part of an engine and, similarly to base components, most often architected using dependency layers.
Low-Level Renderer (LLR) "encompasses all of the raw rendering facilities of the engine...[and] draws all of the geometry submitted to it". [12 1.6.8.1]
It can be perceived as Core Systems but for a rendering engine.
Scene Graph/Culling Optimizations (SGC) is a higher-level component that "limits the number of primitives submitted for rendering, based on some form of visibility determination".
Visual Effects (VFX) includes particle systems, light mapping, dynamic shadows, post efects etc.
Front End (FES) handles UI, aka displaying 2D graphics over 3D scene. This includes heads-up displays, in-game graphical user interface and other menus.

The rest of RTEA consists of smaller, more independent components: Profiling and Debugging tools (DEB), Physics and Collisions (PHY), Animation (SKA), Input handling system (HID),
Audio (AUD), Online multiplayer (OMP) and Gameplay Foundation system (GMP) which provides game developers with tools (often using a scripting language)
to implement player mechanics, define how game objects interact with each other, how to handle events, etc.

One last important component is Game-Specific Subsystems. Those are the concrete implementations of particular gameplay mechanics that are individual to every game,
and highly genre-dependent.
That's a component responsibility of which gradually shifts towards game designers.
"If a clear line could be drawn between the engine and the game, it would lie between the game-specific subsystems and the gameplay foundations layer". [12 1.6.16]
Therefore, this last piece lies outside RTEA, as those subsystems are genre-dependent and closely coupled with a final game, giving them their unique characteristics.

Despite the fact that Gregory lists in great detail potential submodules for each module, the biggest issue of RTEA arises from the inclusion of multiple SDKs that
act not as independent utilities but rather as mini frameworks encompassing the functionality of different components of RTEA.
In other words, unless an engine is written entirely from scratch, addition of external libraries yields the circular dependency of engine modules,
making it more challenging to truly modularize an engine.
Tight coupling between subsystems becomes inevitable as a project grows in size, but there is nothing supernatural about it, as was manifested in
"Visualizing Game Engine Subsystem Coupling Patterns" by Ullmann and the crew. [14]
The problem is not with the coupling per se, but rather with the lack of a universal description of this engine subsystems coupling.
When a single external library implements the functionality of multiple submodules, it is up to individual engine developers to decide how to manage a newly formed architecture.
As an example, let's take a look at Miniquad - a minimalistic graphics toolkit written in Rust.
Trying to fit it inside RTEA proves to be a cognitive challenge as there are 3 potential runtime components that might house this tool.
First is obviously SDK - an interface that allows an engine to talk to the hardware; second is PLA, as it provides a unified API across different platforms;
and lastly LLR, although not completely, because it will draw whatever geometry is submitted to it (geometry primitives are not included).
One reasonable solution is to design a universal game engine protocol that describes every single component that can possibly be a part of the engine,
and requiring the authors of software frameworks to use corresponding parts of this protocol as a template for their libraries.
Writing a game engine on top of a protocol and SDKs that implement that protocol would significantly help with managing the dependency tree
as well as with the future extension of the engine feature set, which consequently improves modding capabilities of the games built using that engine.




Quote research questions from [15] and try to answer them while doing a practical example.
- Terminology: The lack of a ‘game development’ language.
- What is a Game Engine: Where is the boundary between game and game engine?
- Game Genre: How do different genres affect the design of a game engine?
- Überengine.
- Design Dependencies: How do low-level issues affect top-level design?
- Best Practice: Are there specific design methods or architectural models that are used, or should be used, for the creation of a game engine?
Where's a line between a complete game and an engine that was used building it?
- Multiprocessing.


1. Raylib
2. SDL2
3. OpenGL + glfw (Vulkan, DX12)
4. Bevy (webgpu)

Bevy philosophy
https://github.com/bevyengine/bevy/discussions/8107
> Ideally the engine is written in the same language that games are.
> Being able to run an IDE "go to definition" command on a symbol in your game and hop directly into the engine source is an extremely powerful concept.

5. Godot

Godot engine describes itself as
"a feature-packed, cross-platform game engine to create 2D and 3D games from a unified interface.
It provides a comprehensive set of common tools, so that users can focus on making games without having to reinvent the wheel.
Games can be exported with one click to a number of platforms, including the major desktop platforms (Linux, macOS, Windows),
mobile platforms (Android, iOS), as well as Web-based platforms and consoles."
Essential, what Godot is a collection of common tools that are used to create games, that can handle the porting for a game developer.
Here, an engine is something, that contains software components, that can be re-used to make a new game.
In Gregory's architecture, porting responsibilities take Platform Independence Layer (PLA)
Godot is by far the most pleasant and complete General Purpose engine proposed by taxonomy.

6. Mach
7. Possible engine architectures: OOP, ECS, procedural
8. Modding
9. Code/asset reuse?


"Often authors present their own architecture as a de facto solution to their specific problem set"[15 p1]


"The establishment of clear boundaries between game engine (code) and game code" [15]
since game engine is a collection of software utilities, any code that is used to produced a final executable, from compiler to an animation editor, is - a game engine.


"...engine code, depth of simulation, and profiling as some of the highly domain-specific requirements that contribute to the difficulty of writing video games.
The author further breaks down engine code into mathematical and algorithmic knowledge and the wisdom to know how the algorithms will interact when coupled together."[7]

[39]

    You can't treat ease-of-use as a stand-alone concept.
    It's no good if the tools are super easy to get started with, so you can start building a game,
    but they're super hard to finish the game with, because they impose workflow burdens or limited functionality.
    (making tools easy to use has its consequences)

    "Big game companies like EA or Activision don't make the tools investment,
    they don't do the sort of long term thinking we do and realize that we have to make game development processes as efficient as possible."

     So many companies are now crippling their production process by building engines that are perfectly fine with tools that are not fine at all.
     It's always the tools that kill people.
     ...everybody should really make a conscientious decision to either fully invest in producing awesome tools for internal use, or not.
      It's not only for the 3d editor used to build your levels, but it's your build system,
      and it's your programming language, it's your production pipeline, it's the DCC tools you use, all of that.


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


https://hytale.com/news/2024/6/summer-2024-technical-explainer-hytale-s-entity-component-system-oPwpCAMdI

    They use flecs (written in C with a C++ API) as ECS foundation in a C++ engine.

    Most software (and engines) use OOP or entity-component architecture because problems can be broken down into familiar structures that can be reasoned about as objects:
    base character class that can be inherited by main player and add input handling or enemy with AI logic.

    Entity-component: entities (individual units) and components (combination of data and functionality).
    Unity can have multiple components. Difference from OOP is "composition over inheritance".
    Such structure is more modding friendly (changing components of NPC causes a new behavior) because it allows to compose a new entity without tinkering code.
    Scripting language can add functionality of adding new components that can be attached to existing entities.

    ECS takes it a step forward by decoupling functionality from components (data) into separate systems which match entities with defined sets of components and act upon them.
    This data layout is more efficient for the hardware:
    "taking advantage of CPU architecture, structuring data in a tightly-packed way to benefit from its locality in access patterns,
    and using those access patterns to parallelise as much logic as feasibly possible."

    "implementing a robust and performant ECS framework from the ground up is an incredibly challenging and time-consuming endeavor.
    There are countless different flavors of ECS, each with their own benefits and drawbacks..."

    Another reason to use flecs is no need to maintain it yourselves: it is battle-tested, receives frequent updates and bugfixes, and has a comprehensive suite of tests.

    "In many ways, an ECS is similar to a database, and Flecs makes full use of this fact"

    At times, some of these features can be challenging to reason about, but once understood and mastered, they provide extremely powerful game development tools.


Doom3 Source code review: https://fabiensanglard.net/doom3/index.php

"As one game developer pointed out to me this morning, even the ubiquitous Unreal Engine 4 is still built on a foundation
that started with the first Unreal, which came out in 1998."[21]


"To arrange the assets and models in a virtual world and improve the efficiency of rendering, game engines have integrated spatial data structures.
The most common are scene graphs, spatial partitioning, and bounding volume hierarchies (BVHs).
However, each game engine or 3D program has their own implementation of these spatial data structures."[18]


Modding and "the need of a uniform “game language”, but in a much wider context – involving the community aspects of communication as well"[9].
Adding to your game the ability to mod it is like making your game a special-purpose game engine. Minecraft Skyrim and Roblox.
Moding allows to create a new game without creating many assets or writing a lot of code.
"Bethesda does not release games, it releases modding toolkits" - u/Vellarain
"I don't know what game I'm playing anymore."
![Figure 2](mod.png)
A popular Elden Ring mod Seemless Co-op made a game director Hidetaka Miyazaki consider whether they want to implement a start-to-finish co-op experience in their future games.
https://www.pcgamer.com/games/rpg/fromsoftwares-says-elden-rings-seamless-co-op-mod-is-definitely-not-something-we-actively-oppose-and-may-even-consider-ideas-like-that-with-our-future-games/
wiki for acessing game information inside a game directly.

[34]

    The creators of GenieLabs say that it’s much easier for game developers to allow players to create game content by themselves rather than work for years on thousands of DLCs.
    Also, this approach helped to extend the lifespan of existing games by 33%

[40]

    ...if players love your islands, you’ll be eligible to receive payouts based on their engagement.

[44]

    3 traits that determine a game's modability:
    1. Support - how much modding support is built into a game; do developers provide any tools for this?
    2. Nature - sandbox/open world or a fixed path game? How suitable is game's environment to a new and experimental content?
    3. Passion - are gamers passionate about a gameto create new content/fixes for it? Unmodable game can still be reverse engineered (Dark Souls or Minecraft)
    If all 3 are present then a game becomes a its very own game engine.
    Doom (1 and 3, but 2 is meh)
    Even to this day, Doom moding community is very active, and have made truly huge variations of this game, from boxing to a lawn mower sim,
    from a trench warfare to an RPG fantasy.
    Minecraft (not 1, but 2 and 3) Infinite and random world make it easy to place a new content wothout conflicts.
    Simple progression = plenty of room for potential custom progres
    Skyrim (1, 2, 3) Creation kit = same tools used by devs. Open exploration/progression. Writers use it to experience a novel interactively (forgotten city), or whole team
    can use it to make a new game.
    Terraria (no 1, 2, 3) not many mods because vanila game is packed with content. Mods within it do not create brand new experiences.

    Author hypothesizes that a "large moding scenes might be the result of content vacum. Maybe a game can only achieve a modding heaven if its vanila base is broken."

Gregory's Game-Specific Subsystems is a moddable part of a game, that blends a game with its engine.



Data mining is used extensively to analyze how players are interacting with the game. [8]

Alternative architectures:
**Data-Driven Object System**: https://www.youtube.com/watch?v=Eb4-0M2a9xE&t=4s (2002)

    Goal: remove programmers from a picture as much as possible, keeping designers close to the game.
    Game object - piece of logical interactive content (tree, monster, character, triggers, bounding boxes)
    Game object system - construct and manage game objects.

    Usual OOP composition is never perfect because there are hundreds of ways to decompose the GO system problem into classes.
    It starts good, but as a game grows bigger and classes need to be expanded, logical chain gradually dissapears.
    Designers make decisions independently of engineering type structures and will ask for things that cut across engineering concerns.

    GOs are simply a database in a traditional sense. A list of components.
    Component system is flat, and each component is self-contained piece of game logic.
    Components are asembled into GOs to build complete objects. e.g component that can draw has model, texture ... then this component is added to e.g. tree or enemy. 
    Data preferably is in specialization tree (hierarchy) to be able to reuse data.
    External schema that describes GO hierarchy and some properties.

    Consists of 2 parts:
    Dynamic content - GOs created at runtime, die and go away. Componets are made inside scripts. Engine specific performance stuff is for C++.
    Static content - content database, data that goes to schema. Templates that determine how to construct objects from components.

    Parent componets can use OOP features like base class component that will handle messages, have constructor etc
    GO script component base(parent) component that used in scripting lang. Engine doesnt care C++ or script component.

Game Engine **Entity/Object Models**: https://www.youtube.com/watch?v=jjEsB611kxs&t=4s

    

Mention the importance of a multithreading module in the engine. So that doing concurrency becomes an app's own responsibility and not having to rely on OS thread.

Tips for VR https://blog.unity.com/games/accessible-vr-game-design-tips-from-owlchemy-labs

### 2.3. Programming languages analysis

During my university coursework I've had a hands on experience with C#, C, C++, Rust, JS and Zig. I will explain their pros and cons for game development.
We will also take a look at some other potential languages that might suit for game development.
Important to check the benchmarks [16][17].

As was described by [1] in Theoretical part, C and C++ became popular because of the access to the hardware memory.

“write-compile-run-debug” loop

[12 1.6.15.1]

    The game world model is intimately tied to a software object model, and this model can end up pervading the entire engine.
    The term software object model refers to the set of language features, policies and conventions used to implement a piece of object-oriented software.
    In the context of game engines, the software object model answers questions, such as:
    - Is your game engine designed in an object-oriented manner?
    - What language will you use? C? C++? Java? OCaml?
    - How will the static class hierarchy be organized? One giant monolithic hierarchy? Lots of loosely coupled components?
    - Will you use templates and policy-based design, or traditional polymorphism?
    - How are objects referenced? Straight old pointers? Smart pointers? Handles?
    - How will objects be uniquely identified? By address in memory only? By name? By a global unique identifier (GUID)?
    - How are the lifetimes of game objects managed?
    - How are the states of the game objects simulated over time?


https://c0de517e.com/013_web.htm

    "The art of programming close to the machine is disappearing. ... it's the silliness of divorcing programming from the craft of processing data.
    Of having built entire engineering practices, languages, systems, that created an unmanageable amount of complexity for no good reason.
    And then to add insult to injury, layering on top more tools in the foolish errand of trying to solve complexity with complexity."

    "The average big user-facing computing thing (program, website, game...) is a monster so big that nobody understands it end-to-end."

    Needles complexity, cargo-cult are introduced in production without any good reasons. Just because it's popular at a time.

    "...gamedev is closer to the hardware, sure, it is more mindful of it."

    Embrace the crap.
    "Any code written for a product is beholden to the product's needs, and that inevitably makes it ugly."
    "The art of engineering is managing the success of a product, crafting technology that is maximally useful to its purpose,
    keeping it ahead of the competition in the areas that differentiate it,
    and balancing the mountain of short-term needs (production pressure) with the long-term survival of the codebase."

    Game devs must learn from other fields.
    "I was "jealous" of web services mostly in terms of modularity.
    I have always espoused the benefits of live coding, hot-reloading (patching) and very few engines work that way.
    Moreover, few engines are even structured to have well-separated concerns."
    "game engines have theories of API development? Do engineers debate them? Do you start on a whiteboard sketching how subsystems will be divided?"
    Be open minded and don't be shy to learn even from "wasteful" Python.

    "nowadays it became popular for languages to come with their own package management ecosystems, again,
    privileging the idea of code reuse over code isolation, which is the only cure (imho) for complexity - tackling it with well defined runtime barriers."


https://c0de517e.com/014_future_engines.htm

    "The state of the art lies in touching as little memory as possible to initiate work.
    ...the art of a contemporary real-time engine is in how to thin down the inevitable communication from the CPU (typically, game logic, scene updates)
    to the command buffer generation."
    (Command buffers are the main mechanism for sending commands from the CPU to be executed on the GPU)

    "If we were to write an engine today, its role would be entirely around resource management, nothing else, on the CPU.
    Manage memory pools, swapping (streaming) resources in and out - effectively, the CPU knows about the world and its dynamics,
    and it constantly has to figure out what subset to expose to a GPU, in the most efficient way."

    "Memory traffic is not the only limit, if anything capacity is an even bigger issue.
    Compression will play a bigger role, in all its forms. Compressing assets for delivery, network streaming, for disk storage, in memory, in cache.
    ...virtual texturing, displacement mapping, scattering of geometry et al."

    "And low-end (a.k.a. most) mobile is a completely different ballgame, a landscape more uncharted than simply time-delayed compared to the gaming state of the art."


W. Zhang, D. Han, T. Kunz and K. M. Hansen, "Mobile Game Development: Object-Orientation or Not," 31st Annual International Computer Software and Applications Conference (COMPSAC 2007), Beijing, China, 2007, pp. 601-608, doi: 10.1109/COMPSAC.2007.151.

    Mobile games are one of the primary entertainment applications at present.
    Limited by scarce resources, such as memory, CPU, input and output, etc, mobile game development is more difficult than desktop application development,
    with performance as one of the top critical requirements.
    As object-oriented technology is the prevalent programming paradigm, most of the current mobile games are developed with object-orientation (OO) technologies.
    Intuitively OO is not a perfect paradigm for embedded software.
    Questions remain such as how OO and to what degree OO will affect the performance, executable file size,
    and how optimization strategies can improve the qualities of mobile game software.
    These questions are investigated in this paper within the mobile Role-Playing-Game (RPG) domain using five industrial mobile games developed with OO.
    We analyzed them and found excessive usage of OO features used for the development of mobile device applications (but normal for usual desktop applications).
    We then apply some optimization strategies along the way of structural programming.
    The experiment shows that the total jar file size of these five optimized games decreases 71%, the lines of codes decreases 59%,
    and the loading time of each optimized game decreases 22.73%, 34.62%, 25.79%, 24.65% and 16.70% respectively.
    Therefore, we conclude from our experiments that OO should be used with great care in the development of mobile games,
    and that structural programming can be a very competitive alternative.

TODO read
https://sandimetz.com/blog/2016/1/20/the-wrong-abstraction

    prefer duplication over the wrong abstraction

https://zserge.com/posts/better-c-benchmark

    UX of a programming language (how productive the developer is when using this or that language; ease of use; frustrations)

    Zig is surprisingly intuitive to a C coder and feels very simple (to concatenate strings one has to do everything manually - allocate the buffer, put strings there).
    A bit verbose, but explicit, predictable and easy to understand.
    The core of the language is so simple that it does not even use dynamic memory (need to carry the allocator around).
    Language doc - good, stdlib doc - bad.
    Smallest exe.

    Rust doc - comprehensive (a bit too much), tooling - modern and nice.
    Coding in Rust feels like puzzle solving. Could be fun and exciting as hobby, but not ergonomic.
    Largest exe.

    Go - productive (concise doc), but opinionated (not as much controll as Zig, buffered I/O, GC).
    Fastest build times.

    C++ extensive doc, many examples, powerful stdlib, lsp.
    Poor toolign - no derfault build system, no linting. Many flavors?


C++ started as "C with classes". Object-Oriented Programming is the paradigm on which C++ was started.
https://wordsandbuttons.online/the_real_cpp_killers.html

https://blog.sigma-star.io/2024/01/people-dont-understand-oop/

    “OOP to me means only messaging, local retention and protection and hiding of state-process, and extreme late-binding of all things.”

    Alan Kay (the guy who coined the term “Object Oriented Programming”)1

    OOP vs Functional programming vs procedural

    "Objects are collections of operations that share a state."

https://loglog.games/blog/leaving-rust-gamedev/

Cross-compilation. Windows and Unix. Wine and emulation.

What's Zig got that C, Rust and Go don't have? https://youtu.be/5_oqWE9otaE?si=_R5h5LKoFmP2wAGR

    Q:  New programming languages come into being as a reaction to what's missing in the marketplace.
        There's a burning reason why Zig needed to exist.
    A:  Some programs need to have a very close access to the hardware to run 

The effects of object-oriented technology on performance, executable file size, and optimization techniques for mobile games
and suggested that object-oriented technology should be used with great care because the structured programming in game development is highly competitive.[20 5.2.2.5]

Zig build system allows compiling a binary for a specific target hardware, which partly solves a problem of porting from [7 5.3].

Unlike other languages that tried to replace C by adding something new, Zig does the opposite - it removes as much as possible.

https://wikiless.tiekoetter.com/wiki/Comparison_of_programming_paradigms?lang=en

Note about Linux as a more pleasant developing environment.
Potential of NixOS and declarative OSes might help with reproducing development environment.
Any professional needs a sharp tool. A software developer needs to handle his editor the same way an NB player handles a basketball.
Benefits of modal editing and nvim.
"Open-source development platforms are available, but developers must customize them according to the required functionality." [20 5.2.2.3]
"...Windows was not meant for the type of development necessary for video games.
Microsoft Visual Studio was built for Visual Basic and C# mainly, not for C++, but the only viable C++ compiler and environment on Windows remains Visual C++." [7]

Cargo cult programming (practice of applying a design pattern or coding style blindly without understanding the reasons behind that design principle) and software development meta.

### 2.4. Practical example

I don't really consider myself a "programmer".

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

A note about the EUPL licencse.
"open-source is the right path to follow.
What appears to be a "finished" game can always be modded and reproduced into a new product. Thus, a game is also a game engine.
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

Game engine protocol describing all the components of the game engine. Requires a thorough research of existing game engines.

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
[22] Godot Docs v4.2 https://docs.godotengine.org/en/4.2/
[23] Stroustrup, Bjarne (7 March 2010). "Bjarne Stroustrup's FAQ: When was C++ invented?". https://web.archive.org/web/20160206214150/http://www.stroustrup.com/bs_faq.html#invention
[24] On the importance to make games during the game engine’s development. https://www.team-nutshell.dev/nutshellengine/articles/making-games-during-development.html
[25] https://kevurugames.com/blog/unity-what-makes-it-the-best-game-engine
[26] https://kevurugames.com/blog/unity-vs-unreal-engine-pros-and-cons
[27] https://www.creativebloq.com/features/use-unreal-engine-for-filmmaking
[28] https://www.creativebloq.com/features/why-you-need-to-learn-a-game-engine
[29] https://www.creativebloq.com/news/whats-the-future-for-game-engines
[30] https://www.itsnicethat.com/features/why-the-world-needs-more-3d-designers-adobe-report-231122
[31] Serious Games in Digital Gaming: A Comprehensive Review of Applications, Game Engines and Advancements 
[32] https://www.linkedin.com/pulse/unity-game-engine-pros-cons-developers-iman-irajdoost
[33] https://docs.unity3d.com/Manual/system-requirements.html
[34] https://retrostylegames.com/blog/is-unity-good
[35] https://www.arnia.com/what-makes-unity-so-popular-in-game-development
[36] https://www.theguardian.com/games/2023/sep/12/unity-engine-fees-backlash-response
[37] https://www.creativebloq.com/features/unreal-engine-5-everything-you-need-to-know
[38] https://www.unrealengine.com/en-US/uses/film-television
[39] https://web.archive.org/web/20180823012812/https://www.gamasutra.com/blogs/DavidLightbown/20180109/309414/Classic_Tools_Retrospective_Tim_Sweeney_on_the_first_version_of_the_Unreal_Editor.php
[40] https://www.unrealengine.com/en-US/uses/uefn-unreal-editor-for-fortnite
[41] Unreal Engine 5.4 Documentation https://dev.epicgames.com/documentation/en-us/unreal-engine/unreal-engine-5-4-documentation
[42] https://forums.unrealengine.com/t/nanite-performance-is-not-better-than-lods-test-results-fix-your-documentation-epic-youre-ruining-games/1263218?u=_solid_snake
[43] https://forums.unrealengine.com/t/new-ue5-4-feedback-start-working-on-and-investing-in-performance-innovations-for-actual-games/1164987
[44] Modding Heaven: When Games Become Engines. https://youtu.be/bmVsv15gjdg?si=edHuytBVkTX4jlEz
[45] Poor, Nathaniel (24 September 2013). "Computer game modders' motivations and sense of community: A mixed-methods approach". New Media & Society. 16 (8): 1249–1267. doi:10.1177/1461444813504266. S2CID 39280896
