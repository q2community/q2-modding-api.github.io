# Home

Welcome to the Quake II Modding wiki!

***

This wiki will document the modding process for Quake II; this specific wiki is targeting vanilla Quake II (3.2x DLL), Quake II re-release, and Quake II AngelScript. They all share the same framework, although there are minor differences between them.

Please read this page first, as it contains important information that the rest of the pages rely on. Afterwards, you should be able to jump into pretty much any other section.

Note that this wiki will assume you have some basic programming experience (we're not going to explain what a C string is, for instance).

Also, note that this wiki will guide you through the code that already exists, but won't take into account any additional stuff you might do with the code; for instance, you *could* modify vanilla or re-release to store C++ types in savable structures, but the wiki will assume you are using the code that we ship with.

## Icons

Since this wiki is covering three flavors of Quake II, bits of text may have icons in them to indicate notes, differences or particularities of one or more versions. You may see the following three icons used at various points, and their meanings are to their right:

* 🍦 Quake II 3.2x vanilla C code base
* ✨ Quake II re-release
* 🪽 AngelScript

You might see an entire section dedicated to a single release, (🪽 or perhaps a small note that only applies to one release).

## Terms

### memory

🍦✨ Any memory allocated from your mod should always go through the [tagged memory allocator](tagged-memory). This allows the engine to properly clean up memory if anything unexpected happens. You won't suffer any major consequences by using regular heap memory, but anything especially long term should be allocated using the above imports.

### handle / pointer

These two terms are interchangeable, and will refer to some pre-allocated chunk of memory somewhere. For instance, entities are pooled, and any given entity will 'point' to an entry in that chunk of memory.

### string

🍦✨ Strings that are stored in savable structures are C strings, and have to be allocated from [memory](#memory).
🪽 Strings are simple wrappers around `std::string` and need no special handling.

### server

The server is the engine portion of the code that runs the game. It [imports](server-imports) and [exports](server-exports) functions, constants, structs, and variables to communicate with the game module.

### edict / entity

`edict` and `entity` will often be used interchangeably, but they technically refer to two related concepts. `edict` is short for `entity dictionary`, and is a vestigial name left over from Quake; maps store key/value pairs of strings (a dictionary), and these pairs are then [parsed into an entity before they spawn](entity-lifecycle). An entity refers to the final entity pointer (🍦✨ `edict_t` 🪽 `ASEntity`).

In general, though, `edict` usually refers to the [server](#server)-visible portion of an entity, whereas `entity` refers to all of the additional data that the game adds on top of this. (🍦✨ `edict_t` keeps all of this data inside of it; see `edict_shared_t` for the server-visible fields, whose position & types **cannot be modified** by the game code.) (🪽 The data is split between `edict_t`, which is a thin handle to the server-visible fields, and `ASEntity`, which is where all of the game data goes into.)

### client / player

The client, as a concept, can refer to two different things contextually:

* The part of the engine (✨🪽 and the `cgame` portion of the game module) that runs for each player on their side. It is not connected to the server directly, and instead receives its updates through network packets to display the scene.
* A player (✨🪽 or bot) who has connected to the server. On the server & game side, similar to `edict_t`, the type `gclient_t` contains the data that is shared between the server & game. (🍦✨ The game can add more fields at the end of the client structure; only the top shared portion needs to match.) (🪽 `ASClient` is the wrapper type around `gclient_t`, and is where you add new fields into.)

While the game code refers to them both by the term `client`, I'll try to always refer to players as players in the documentation for clarity.

### spawn / spawning

Spawning is the term Quake games use for entities being created.

### free / freeing

Freeing is the term Quake games use for entities that are being destroyed. Technically, an entity does not get actually destroyed, but rather marked as free, and their slot can be re-used by other entities later.