# Dnipro

A final thesis about "Zig as the Foundation for a Portable and Modular Game Engine" with code implementation.

# Introduction

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


# 1 Theoretical part

Most of the academic papers that touch the topic of game engines mention that there is not enough research being done on the topic [6][7][10][20].
"This lack of literature and research regarding game engine architectures is perplexing."[15 p1]
By the time of writing of this paper, several works about game engines internal architecture have been published,
however, to the best of my knowledge, none has discussed the importance of a programming language.

Anderson et al do not perform a research, but introduce fundamental questions that need to be considered when designing the architecture of a game engine [15].
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
- RNG
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
[7] defines common problems during a project life-cycle, and similarly to Politkowski et al finds that the biggest contributor to problems list is the management:
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

Toftedahl and Engström in their work [9] categorize game engines into 4 types:
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

So what is a game engine? "John Carmack, and to a less degree John Romero, are credited for the creation and adoption of the term game engine.
In the early 90s, they created the first game engine to separate the concerns between the game code and its assets and to work collaboratively on the game as a team."[1]
A game became known as Doom, and it gave birth to the first-ever "modding" community.[12 1.3]
Thus, a new game could be made by simply adding new art, environment, characters etc. inside a Doom's engine.
It is hard to draw a line and try to fit this engine into a taxonomy proposed by Toftedahl and Engström.
It is both a Special Purpose engine, that creates a game in a specific genre; and a Core engine, that provides useful software utilities.

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
However, for the purpose of this paper, "game engine" will also imply an ecosystem around software development, which helps with producing video games in one way or another.
Development environment potentially has a significant influence on the quality of a final product, and so cannot be ignored when designing a modern engine.

### Terminology

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

# 2 Research

During my university years I had an experience making games in following engines, so I will be describing what are Unity, UE5, Raylib, SDL2/3, Bevy, Godot and Mach.
The only way for a developer to understand the way certain components work and communicate is to create his/her own computer game engine [13].
Plus the second biggest reason why new engines are being created is because of the learning purposes. With first reason needing more control over the environment.
The actual need to create a game is only on third place [1].


### game engine analysis.

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
https://venturebeat.com/pc-gaming/the-engine-behind-paradox-development-studios-future-games/

"As one game developer pointed out to me this morning, even the ubiquitous Unreal Engine 4 is still built on a foundation
that started with the first Unreal, which came out in 1998."[21]

According to [9] Unreal and Unity are currently the most widely used comercial game engines.
What problems can a proper engine help to eliminate during Game Development Software Engineering Process Life Cycle[5]
My notes on Unity and UE5.

Brief summary of Raylib, SDL2, Bevy, Godot and Mach.
Godot is by far the most pleasant and complete General Purpose engine proposed by taxonomy.

WebGPU.

"To arrange the assets and models in a virtual world and improve the efficiency of rendering, game engines have integrated spatial data structures.
The most common are scene graphs, spatial partitioning, and bounding volume hierarchies (BVHs).
However, each game engine or 3D program has their own implementation of these spatial data structures."[18]

Modding and "the need of a uniform “game language”, but in a much wider context – involving the community aspects of communication as well"[9].
Adding to your game the ability to mod it is like making your game a special-purpose game engine. Minecraft Skyrim and Roblox.

Data mining is used extensively to analyze how players are interacting with the game. [8]

Alternative architectures:
- Data-Driven Object System: https://www.youtube.com/watch?v=Eb4-0M2a9xE&t=4s (2002)
- Game Engine Entity/Object Models: https://www.youtube.com/watch?v=jjEsB611kxs&t=4s

Multithreading

### Programming languages

During my university coursework I've had a hands on experience with C#, C, C++, Rust, JS and Zig. I will explain thier pros and cons for game development.
Important to check the benchmarks [16][17].

As was described by [1] in Theoretical part, C and C++ became popular because of the access to the hardware memory.

The effects of object-oriented technology on performance, executable file size, and optimization techniques for mobile games
and suggested that object-oriented technology should be used with great care because the structured programming in game development is highly competitive.[20 5.2.2.5]

Zig build system allows compiling a binary for a specific target hardware, which partly solves a problem of porting from [7 5.3].

Note about Linux as a more pleasant developing environment.
Any professional needs a sharp tool. A software developer needs to handle his editor the same way an NB player handles a basketball.
"Open-source development platforms are available, but developers must customize them according to the required functionality." [20 5.2.2.3]
"...Windows was not meant for the type of development necessary for video games.
Microsoft Visual Studio was built for Visual Basic and C# mainly, not for C++, but the only viable C++ compiler and environment on Windows remains Visual C++." [7]

Cargo cult programming and software development meta.

### Practical example

Find on https://github.com/zig-gamedev/zig-gamedev from a list of libs those that are required by an engine architecture proposed by Gregory.
Pull all the dependencies and compile without the errors.
Write 2D games: Snake and Breakout and an 3D scene with a player.
Make them run on Windows.

A note about the EUGPL licencse.
"open-source is the right path to follow.
It democratizes and allow a soft learning curve for beginners. Also, it will allow the creation of more diverse games."[1 6.4]
[7] mentions domain specific knowledge required for engine development. Open source game/game engine with clearly distinct modules is the optimal environment for
a developer who can apply (or try to apply) domain-specific, deeply specialized knowledge.

# Results

A game engine is primarily a collection of software tools. A swordsman needs to be proficient with a sword and make sure it's sharp.
Thus, the best game engine is the one that non-software developers don't even need to know anything about.
A 3D modeler is more productive with his 3D modelling software than with any sort of engine. The same goes for animators, artists, sound designers etc.

# Discussion

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

# Conclusion

Game Design document need to specify more information about the game systems, so that an engine has more information to work with.

# References


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
