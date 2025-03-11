# AngelScript

AngelScript is a scripting language created by `Andreas Jonsson`. It's a neat scripting language, and it's perfect for Quake II as it shares many of the concepts and syntax with the original C/C++ code base, while also being similar to QuakeC in some areas.

[You can read more about the scripting language itself at its main website](https://www.angelcode.com/).

For the actual language documentation, you should refer to [the official online documentation](https://www.angelcode.com/angelscript/sdk/docs/manual/index.html).

# Q2 AngelScript Runtime

The Quake II AngelScript Runtime is a game module that provides AngelScript scripting support. It's *not* a rewrite or remake of the original code; it's derived from the Q2PSX project, and is more of a port of the original to AngelScript. It is possible, however, to use this as a framework to make a more complex framework (for instance one that makes use of AngelScripts' object oriented features).

# Running an AS mod

To run an AngelScript-supported mod, currently all that's required is that the `scripts` subfolder is dropped into the same folder as the AngelScript runtime `game_x64.dll`. The game will then do the following:

* if the `q2as_use_game` cvar is 1 (the default), it will attempt to load the game module
  * all of the scripts inside of the `scripts/bgame` and `scripts/game` folders are loaded into the engine and built
  * in the currently available developer release, the text `SVAS` will be at the bottom-left of your screen if the AS runtime is loaded. 
* if the `q2as_use_cgame` cvar is 1 (the default), it will attempt to load the cgame module
  * all of the scripts inside of the `scripts/bgame` and `scripts/cgame` folders are loaded into the engine and built
  * in the currently available developer release, the text `CGAS` will be at the bottom-left of your screen if the AS runtime is loaded. 

You can edit files while the game is running, although there is currently no hot reload support. For single player/co-op mods, you can simulate a hot reload by using `save` and `load`. In deathmatch, however, you'll have to reload the map.

# Debugging

If you're running into issues with your mod, Q2AS provides a few tools to help track down any problems.

## Instrumentation

The DLL implements [Palanteer](https://github.com/dfeneyrou/palanteer), a simple low overhead library for recording profiling traces. You can download the executable from the GitHub repo above to inspect the traces.

To enable tracing, the `q2as_instrumentation` must be set to either 1, 2 or 3. It is a bitflag of which modules to instrument. `1` is the game, `2` is the cgame, and `3` will instrument both.

There's no on/off toggle, and it will instrument the entire life of your game session. `disconnect` or `quit` to stop the instrumentation.

> [!NOTE]
> These files can get *large*, be careful with disk space!

When finished, you can open the output file with Palanteer. It should output next to where the `.exe` file is for your engine.

## Debugger

The DLL implements [AngelScript UI Debugger](https://github.com/Paril/angelscript-ui-debugger), a sister project, which provides an easy-to-use debugger frontend.

When the runtime hits an exception, the debugger will automatically open to show you the exception state. Exceptions are not recoverable in AngelScript, and you will be dropped to console after closing the debugger when an exception is thrown. It gives you the ability to inspect the stack and locals & variables, though.

To use the debugger, you have a few options:
* call `debugbreak()` somewhere from inside your AngelScript code; this will force a breakpoint every time that line is hit
* set the `q2as_bp` cvar to either a function name (eg `fire_blaster`) or a file + line (eg `game/ai.as 672`). This will insert a breakpoint at the start of the function (or at the specified file/line combo) respectively.

> [!NOTE]
> The `q2as_bp` feature is not fully complete, but it should work for function names at the very least. It also doesn't auto-reset yet, so you may have to reset it back to `""` and then set it to your function again if you want to enter the same value that is already set in the cvar.

When a breakpoint is hit, you can then add more breakpoints in the debugger or use the Step features to move through code. As long as there is at least one breakpoint or step waiting to be hit, the debugger will not close and you can keep using it to add more breakpoints, etc. When not in broken state, though, you cannot inspect variables, as the memory state of the program has changed.