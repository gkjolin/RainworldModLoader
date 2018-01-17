![](https://i.imgur.com/KSE4URu.gif)

(Example: Slugcat Camo Mod by LodeRunner + Co-op Mod by OriginalSINe, both running through this mod loader. Check out [RainDB.net](http://www.raindb.net) for more released mods, which may some day be made compatible with this loader.)

# RainworldModLoader
Proof of concept Rainworld Mod Loader, almost at a state of being very useful.

If you are a mod developer, please do get in touch and provide feedback! I'm on the [Rainworld Discord](https://discordapp.com/invite/SBmHbpW) in #modding.

(This same approach could be used to create mod loaders for any Unity game, really.)

# Why?

The existing approach to modding Rainworld is to load the game's dotnet assembly using a tool called [dnSpy](https://github.com/0xd4d/dnSpy/), which lets you see decompiled source. You then have to edit the low level IL assembly code to do anything, deal with all sorts of cryptic errors, and after that your code is stuffed into the game DLL.

This workflow is not great for writing and maintaining comprehensive modifications.

# How Does It Work?

An injector program takes the vanilla game dll and injects a small mod loader routine. This only has to be done once to enable mod loading for an installed version of the game.

The mod loader will then load custom DLLs with mod-only code when the game starts up, and get the game to call into it.

The loader looks for mod assemblies in the Rainworld/Mods folder. Anything called *******Mod.dll* that contains an implementation of the modding interface gets loaded. Any static class that has a static void Initialize() and a name that ends with *******Mod* will have that method called.

The [Harmony](https://github.com/pardeike/Harmony/wiki) framework is currently used to inject your mod hooks into the game's code. Check out the provided example mod projects to see how it's done.

**Important: the exact way the mods are hooked into the game's many functions is still subject to change.**

Three big wins:

- The game can load multiple mods at once. (Providing they don't conflict, which requires some developer care. The modding API can't guarantee that any two mods go well together, but their developers can.)
- You can write your code in visual studio, in C#, organize it into separate DLLs, have dependencies, use source control, and so on.
- Mods and modding APIs don't need to redistribute copyrighted code in Assembly-CSharp, but can be locally applied as a patch to your game install with an easy to use patching program. The Hollow Knight modding community has [encountered and discussed this issue](https://gist.github.com/thejoshwolfe/db369bebf6518227c830fffee12ddbec), leading to a similar approach.

# Developing Mods

Check out this repository. Take note of the following:

Paths to important game folders are currently still hardcoded into the code and project setup. They will be different on your machine, so make sure to change the following:

* ModLoader.csproj -> Build/OutputPath
* MyModName.csproj -> Build/OutputPath (for any mod project in the solution)
* Inject.cs        -> public const string RootFolder, public const string AssemblyFolder

Injector also expects you to have made a backup of Assembly-CSharp.dll in the same folder, called Assembly-CSharp-Original.dll. It will do this automatically in the future.

You should now have a VS2017 solution that patches your game when ran, builds the mod assemblies and puts them in the game mod folder.

*Note: currently this project contains multiple example mods which will eventually migrate to their own repositories. They're purely there to help quickly iterate on the mod api design. You can delete them, overwrite them, or enable/disable them in the solution's build configuration.*

# Debugging

For debugging I still recommend using [dnSpy](https://github.com/0xd4d/dnSpy/wiki/Debugging-Unity-Games). If you load the game's assembly in there, and manually add your installed mod assemblies, and then launch the game through dnSpy's debug menu, its debugger works like a charm.

I've tried getting VS2017's debugging tools to work with the game, but haven't managed it yet. And besides, dnSpy's decompiled code is nicer to read.

The mod loader also automatically enables logging to consoleLog.txt and exceptionLog.txt, so all your Debug.Log, Debug.LogWarning and Debug.LogError calls ends up there.
