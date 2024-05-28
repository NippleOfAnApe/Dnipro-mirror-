# dnipro

A final thesis about " Zig as the Foundation for a Portable and Modular Game Engine" with code implementation.

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

Additionally, rebirth of handhelds with Nintendo Switch, Steam Deck and other platforms means that developers need to take more care about optimization
since the resources there are more limited compared to bulky PCs and consoles.
Although the semiconductor industry has been good at identifying near and long term bottlenecks of Moor's Law [3], some chip-makers claim that Moor's Law is dead,
justifying the price hike of graphics cards [4].
Thus, it is important for developers to find new ways for optimizing their games inside software, instead of relying on the hardware to handle the ever-increasing load.
Popular engines like Unreal and Unity allow for a faster development cycle with the cost of abstracting the low-level systems.

For these reasons, it is important to periodically evaluate the underlying tools and to look for possible improvements.
One of them could be switching to another language.
Most of the modern games are written in C++ and C [1] which allows developers to have more control over the hardware.
However, both of those languages are quite old, and were designed in a different age and for different devices that are common nowadays.

Game-development is a very complex process nowadays, especially in AAA studios. Generating the assets and writing the code is just a tine fraction off it.
Biggest problems that are faced during the process of game development are related to scope, feature creep, and cutting features.
Additionally, second most common kind of problem is related to technical issues [6].
Creation of a game starts from pre-production stage, with writing a game design document and scribbling the concept art.
I believe that it's during this period that a company needs to decide what kind of engine will be used based on the core gameplay features.
A right decision here might reduce the amount of technical problems by simply removing the redundant features of general-purpose engine like Unity.

In order to find possible improvements in current game development process, the following research questions will need to be answered:
Question #1: What is are main components of a game engine? Question #2: How does a programming language impact the game development process?

To answer them the following research tasks need to be performed:
1) analyze the architecture of popular game engines and identify common components;
2) compare C, C++, Rust, Zig and Odin for language features, semantics, and build system with other established languages;
3) analyze the suitability of Zig for game development;
4) demonstrate the feasibility with a practical example in a form of a simple 2D game.

The paper is structured as follows:

- Section 1 introduces the theoretical part by explaining some of the softwere development concepts related to game development, and the research in this field made so far.
- Section 2.1 give the analysis of existing game engines. Section 2.2 explains the importance of a programming language.
- Section 2.3 then demonstrates the feasibility of creating games using a modern programming language.
- Section 3 discusses our results and threats to their validity. Section 4 concludes.


# Theoretical part

Some of the academic papers that touch the topic of game engines mention that there is not enough research being done on the topic [6][10][11].
By the time of writing this paper, several works about game engines internal architecture have been published,
however, to the best of my knowledge, none has discussed the importance of a programming language.

A comprehensive book by Gregory [12] about the engine architecture is referenced in multiple studies.
He proposes a "Runtime Engine Architecture", in which a game engine consists of multiple runtime components with clear dependency hierarchy.
They are Audio (AUD), Core Systems (COR), Profiling and Debugging (DEB), Front End (FES), Gameplay Foundations (GMP), Human Interface Devices (HID),
Low-Level Renderer (LLR), Online Multiplayer (OMP), Collision and Physics (PHY), Platform Independence Layer (PLA), Resources (RES),
Third-party SDKs (SDK), Scene graph/culling optimizations (SGC), Skeletal Animation (SKA), Visual Effects (VFX).
He finds an optimal middle ground between a high-level, broad description of how those modules interact with each other, and their detailed implementations.
Most of the examples are written in C++ and are object-oriented by nature, however.

Ullmann et all [13] used this model as a target reference when exploring the architecture of three popular open-source game engines (Cocos2d-x, Godot, and Urho3D).
They added a separate World Editor (EDI) subsystem "because it [World Editor] directly impacts game developers’ work."
Godot was found to have all 16 out of 16 subsystems, while Cocos2d-x and Urcho3D 13 and 12 respectively.
However, the missing functionality was not absent, but rather scattered among other subsystems.
They also found which subsystems are more often coupled with one another: COR, SGC and LLR.
COR file are "useful software utilities" that are used throughout the system.
"Video games are highly dependent on visuals, and graphics are a cross-cutting concern, therefore it is no surprise that so many subsystems depend on LLR and SGC"[13]
Other commonly coupled systems were VFX, FES, RES and PLA.

In their next paper, they sampled 10 engines and found that the top-five subsystems were:
Core (COR), Low-Level Renderer (LLR), Resources (RES), World Editor (EDI) and Front End (FES) tied with Platform Independence Layer (PLA).
Those subsystems "act as a foundation for game engines because most of the other subsystems depend on them to implement their functionalities" [14].
A visualized dependency graph is shown in Figure 1 [subsystems.png].
There are, however, other architectures used in proprietary mainstream engines, of which very little is known. And the suggested architecture is only one of the possibilities.

[1] explores whether game engines share similar characteristics with software frameworks by comparing 282 most popular open source engines and 282 most popular frameworks on GitHub.
"Game engines are slightly larger in terms of size and complexity and less popular and engaging than traditional frameworks.
The programming languages of game engines differ greatly from that of traditional frameworks.
Game engine projects have shorter histories with less releases.
Developers perceive game engines as different from traditional frameworks and claim that engines need special treatments."
They also explain that the reason for popularity of C family of languages is that "...engines must work close to the hardware and manage memory for performance.
Low-level, compiled languages allow developers to control fully the hardware and memory."
Another common difference is that game developers often add scripting capabilities to their engines to ease the design and testing workflow.
Meanwhile, frameworks products are often written in the same programming languages.



    The theoretical part is inarguably an essential part of all types of research, including practical research, i.e. applied research papers.
    In this part, the author provides an overview of earlier treatments of the research problem established in the introduction and studies made
    and analyses the views of different authors by highlighting the pros and cons of different treatments and supporting them with arguments.
    In the case of practical research, this part is also used to explain the choice of a theory suitable for the planned solution
    and highlight the benefits of the chosen method compared with other options.
    The central terms used are also explained in this part of the paper.

1) explain terms [9]
Types of game engines (Core, gp, etc)
    "John Carmack, and to a less degree John Romero8, are credited for the creation and adoption of the term game engine.
    In the early 90s, they created the first game engine to separate the concerns between the game code and its assets and to work collaboratively on the game as a team."[1]
    "In 2002, Lewis and Jacobson defined game engines
    as “collection[s] of modules of simulation code that do not
    directly specify the game’s behavior (game logic) or game’s
    environment (level data)”. In 2007, Sherrod defined en-
    gines as frameworks comprised of different tools, utilities,
    and interfaces that hide the low-level details of the imple-
    mentations of games. Engines are extensible software that
    can be used as the foundations for many different games
    without major changes and are “software frameworks for
    game development”. They relieve developers so that they
    can focus on other aspects of game development”. In 2019,
    Toftedahl and Engström analysed and divided engines
    in four complementary types: (a) Core Game Engine, (b)
Game Engine, (c) General Purpose Game Engine, and (d)
    Special Purpose Game Engine."[1]
Low-level, high level language.
Language of engine vs language of scripting
2) lack of proper research [5], and related work
engine is building static meshes, not accounting for environment (spatial computing)
Need to datamine gamers to better understand the market. Emotional map [7]
Data mining is used extensively to analyze how players are interacting with the game. [8]
lack of information regarding the engines used in mainstream games [1]



# Research

During my university years I had an experience making games in following engines, so I will also describe what are Unity, UE5, Raylib, SDL2/3, Bevy, Godot and Mach.

### game engine analysis.

What kind of functionality is provided by those engines, that make a game developer pick this engine.
Quote research questions from "The Case for Research in Game Engine Architecture" and try to answer them while doing a practical example.
- Terminology: The lack of a ‘game development’ language.
- What is a Game Engine: Where is the boundary between game and game engine?
- Game Genre: How do different genres affect the design of a game engine?
- Design Dependencies: How do low-level issues affect top-level design?
- Best Practice: Are there specific design methods or architectural models that are used, or should be used, for the creation of a game engine?

### Programming languages

During my university coursework I've had a hands on experience with C#, C, C++, Rust, JS and Zig. I will explain thier pros and cons for game development.
Important to check the benchmarks [16][17].

Note about Linux as a more pleasant developing environment.

### Practical example

Find on https://github.com/zig-gamedev/zig-gamedev from a list of libs those that are required by an engine architecture proposed by Gregory.
Pull all the dependencies and compile without the errors.
Write 2D games: Snake and Breakout and an 3D scene with a player.
A note about the EUGPL licencse.


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
[11] A Holistic Look at Requirements Engineering in the Gaming Industry
[12] Jason Gregory. Game engine architecture. Taylor & Francis, CRC Press, Boca Raton, 3 edition, 2018. ISBN 978-1-138-03545-4.
[13] An Exploratory Approach for Game Engine Architecture Recovery
[14] Visualizing Game Engine Subsystem Coupling Patterns
[15] The Case for Research in Game Engine Architecture
[16] https://programming-language-benchmarks.vercel.app
[17] https://zserge.com/posts/better-c-benchmark
