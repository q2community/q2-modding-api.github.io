# Server Imports

Server imports (referred to as 'game imports' interchangeably) are the functions that the server provides to the game module, and allow the game module to interact with systems that the game does not have any direct control over.

We refer to these as *server* imports because we're writing from the POV of the game module (the game is importing functions from the server and exporting functions back to the server), and re-release has a client game (or cgame) module which also provides their own imports/exports.

> [!NOTE]
> The documentation here presents the functions as if they were globals; this is to simplify the documentation and not confuse programmers with the syntax of function pointers in C/C++.
>
> 🍦✨ All imports are accessed through the `gi` global, which is a static variable of `game_import_t` that contains pointers to all of the below functions.<br/>
> 🪽 All imports are global functions, prefixed with `gi_`.

# Printing

The API provides several functions to print messages to the server console & to other users.

✨🪽 These functions were renamed away from their legacy Quake counterparts, and now do not provide any sort of formatting support directly, instead relying on the game module to bring their own formatting library. This was to help support other languages as well as provide a free performance improvement when a string without any format specifiers were used. Localization is now also supported, and the `Loc` variants should always be used.

## Loc_Print ✨🪽

This is a new API provided for re-release which replaces all of the other functions. Several convenience wrappers are also provided to make calling this function a bit less painful.

The re-release supports localization, which works via wrapping `fmtlib` around an array of strings. Localization strings can refer to them by number (such as `{0} {1}`) to re-order them.

Technically, this function can also be used to send raw textual strings, but for those kinds of strings one can also use the legacy print functions without issue.

A [print level](Types#print_type_t) can be specified to change the priority level of the message, and to change behaviors of printing.

<!-- tabs:start -->

#### **✨**

```cpp
void Loc_Print(edict_t *ent, print_type_t level, const char *base, const char **args, size_t num_args);
void LocClient_Print(edict_t *e, print_type_t level, const char *base, ...);
void LocBroadcast_Print(print_type_t level, const char *base, ...);
void LocCenter_Print(edict_t *e, const char *base, ...);
```

When the varargs versions are used, a custom bespoke formatter is used to format simple types. Only floating point types, numeric types, and non-null `const char *` are allowed.

#### **🪽**

```cpp
void gi_Loc_Print(edict_t @ent, print_type_t printlevel, const string &in base);
void gi_Loc_Print(edict_t @ent, print_type_t printlevel, const string &in base, const ?&in...);
void gi_LocClient_Print(edict_t @, print_type_t printlevel, const string &in message);
void gi_LocClient_Print(edict_t @, print_type_t printlevel, const string &in fmt, const ?&in...);
void gi_LocBroadcast_Print(print_type_t printlevel, const string &in message);
void gi_LocBroadcast_Print(print_type_t printlevel, const string &in fmt, const ?&in...);
void gi_LocCenter_Print(edict_t @ent, const string &in message);
void gi_LocCenter_Print(edict_t @ent, const string &in fmt, const ?&in...);
```

When the varargs versions are used, `fmtlib` is used to format the arguments.

<!-- tabs:end -->

## bprintf<br/>Broadcast_Print

This function prints a message to all players. A [print level](Types#print_type_t) can be specified to change the priority level of the message (✨🪽 and to change behaviors of printing).

<!-- tabs:start -->

#### **🍦**

```cpp
void bprintf (int printlevel, char *fmt, ...);
```

This function uses C-style `sprintf` printing (see https://en.cppreference.com/w/cpp/io/c/fprintf).

#### **✨**

```cpp
void Broadcast_Print(print_type_t printlevel, const char *message);
```

> [!WARNING]
> This function is functionally deprecated and is only provided to make porting code from vanilla easier. [LocBroadcast_Print](#LocBroadcast_Print) should be used instead.

#### **🪽**

```cpp
void gi_Broadcast_Print(print_type_t printlevel, const string &in message);
void gi_Broadcast_Print(print_type_t printlevel, const string &in fmt, const ?&in...);
```

> [!WARNING]
> This function is functionally deprecated and is only provided to make porting code from vanilla easier. [LocBroadcast_Print](#LocBroadcast_Print) should be used instead.

When provided with three or more arguments, `gi_Broadcast_Print` uses `fmtlib` (https://fmt.dev/11.1/) (or `std::format`) formatting.

<!-- tabs:end -->

## dprintf<br/>Com_Print

This function prints a message solely to the server. It may be directed to the server console, a log file, etc.

<!-- tabs:start -->

#### **🍦**

```cpp
void dprintf (char *fmt, ...);
```

This function uses C-style `sprintf` formatting (see https://en.cppreference.com/w/cpp/io/c/fprintf).

#### **✨**

```cpp
void Com_Print(const char *msg);
void Com_PrintFmt(const char *format_str, ...)
```

`Com_PrintFmt` uses `fmtlib` (https://fmt.dev/11.1/) (or `std::format`) formatting, and is provided for convenience. 

#### **🪽**

```cpp
void gi_Com_Print(const string &in message);
void gi_Com_Print(const string &in fmt, const ?&in...);
```

When provided with two or more arguments, `gi_Com_Print` uses `fmtlib` (https://fmt.dev/11.1/) (or `std::format`) formatting.

<!-- tabs:end -->

## cprintf<br/>Client_Print

This function prints a message to a client (or the server console, if a null entity is provided). A [print level](Types#print_type_t) can be specified to change the priority level of the message (✨🪽 and to change behaviors of printing).

<!-- tabs:start -->

#### **🍦**

```cpp
void cprintf (edict_t *ent, int printlevel, char *fmt, ...);
```

This function uses C-style `sprintf` formatting (see https://en.cppreference.com/w/cpp/io/c/fprintf).

#### **✨**

```cpp
void Client_Print(edict_t *ent, print_type_t printlevel, const char *message);
```

> [!WARNING]
> This function is functionally deprecated and is only provided to make porting code from vanilla easier. [Loc_Print](#Loc_Print) should be used instead.

#### **🪽**

```cpp
void gi_Client_Print(edict_t @ent, print_type_t printlevel, const string &in message);
void gi_Client_Print(edict_t @ent, print_type_t printlevel, const string &in fmt, const ?&in...);
```

> [!WARNING]
> This function is functionally deprecated and is only provided to make porting code from vanilla easier. [Loc_Print](#Loc_Print) should be used instead.

When provided with four or more arguments, `gi_Client_Print` uses `fmtlib` (https://fmt.dev/11.1/) (or `std::format`) formatting.

<!-- tabs:end -->

## centerprintf<br/>Center_Print

This function prints a message to a client (or the server console, if a null entity is provided) in the center of their screen.

<!-- tabs:start -->

#### **🍦**

```cpp
void centerprintf (edict_t *ent, char *fmt, ...);
```

This function uses C-style `sprintf` formatting (see https://en.cppreference.com/w/cpp/io/c/fprintf).

#### **✨**

```cpp
void Center_Print(edict_t *ent, const char *message);
```

> [!WARNING]
> This function is functionally deprecated and is only provided to make porting code from vanilla easier. [Loc_Print](#Loc_Print) should be used instead.

#### **🪽**

```cpp
void gi_Center_Print(edict_t @ent, const string &in message);
void gi_Center_Print(edict_t @ent, const string &in fmt, const ?&in...);
```

> [!WARNING]
> This function is functionally deprecated and is only provided to make porting code from vanilla easier. [Loc_Print](#Loc_Print-✨🪽) should be used instead.

When provided with four or more arguments, `gi_Client_Print` uses `fmtlib` (https://fmt.dev/11.1/) (or `std::format`) formatting.

<!-- tabs:end -->

## error<br/>Com_Error

This function prints an error message to the server and ends the game session (drop to console).

<!-- tabs:start -->

#### **🍦**

```cpp
void error (char *fmt, ...);
```

This function uses C-style `sprintf` formatting (see https://en.cppreference.com/w/cpp/io/c/fprintf).

#### **✨**

```cpp
void Com_Error(const char *message);
void Com_ErrorFmt(const char *message, ...);
```

`Com_ErrorFmt` uses `fmtlib` (https://fmt.dev/11.1/) (or `std::format`) formatting, and is provided for convenience. 

#### **🪽**

```cpp
void gi_Com_Error(const string &in message);
void gi_Com_Error(const string &in fmt, const ?&in...);
```

When provided with two or more arguments, `gi_Com_Error` uses `fmtlib` (https://fmt.dev/11.1/) (or `std::format`) formatting.

<!-- tabs:end -->

# Sounds

Quake II provides a few imports for sending sound packets to players.

Sounds are constructed with a couple parameters:
* An entity to play a sound on. When an entity is provided, the sound will "follow" the entity if the entity is currently visible to the player, otherwise it will play at a single position.
* A fixed position to play a sound at. Positioned sounds can be given explicit origins.
* A [sound index](sound-index), the actual sound to play. This can not be zero.
* A volume to play the sound at, between 0 and 1 inclusive. A volume of 0 is technically legal although it will not be audible.
* An attenuation factor, between 0 and 4 inclusive. Sound attenuation determines the distance multiplier for sounds. For all values *except* `3.0` (`ATTN_STATIC`), the distance multiplier is `attenuation * 0.001`. For `3.0` (`ATTN_STATIC`), the multiplier is `attenuation * 0.0005` (`0.0015`); this effectively makes attenuations of `1.5` and `3.0` equal. An attenuation of `0.0` (`ATTN_NONE`) has no distance multiplier at all, and will be full volume across the entire level and have no spatialization (will always sound as if it is "inside" the listener).
* A time offset, between 0.0 and 0.255 inclusive. This is a timed delay - in seconds - applied to the sound. This is used for the shotgun, for instance, to cause a delay between the firing sound and the pumping sound.
* A sound channel, which consists of a 3-bit channel id and modifier flags. See [soundchan_t](Types#soundchan_t) for more information about these.

> [!NOTE]
> 🍦 When an explicit origin is not specified, sounds can sometimes be played at the wrong position if an entity is inside of a client's [PHS](PVS) but not their [PVS](PVS). If the entity has never been seen before, it will play at the world origin (`0 0 0`), otherwise it may play at the last position the client spotted the entity.

## sound

This function plays a sound either on the given entity or the world (if `ent` is `null`).

<!-- tabs:start -->

#### **🍦**

```cpp
void sound (edict_t *ent, int channel, int soundindex, float volume, float attenuation, float timeofs);
```

#### **✨**

```cpp
void sound (edict_t *ent, soundchan_t channel, int soundindex, float volume, float attenuation, float timeofs);
```

#### **🪽**

```cpp
void gi_sound(edict_t @ent, soundchan_t channel, int soundindex, float volume, float attenuation, float timeofs)
```

<!-- tabs:end -->

## positioned_sound

This function plays a sound either on the given entity or the world (if `ent` is `null`), and optionally at a fixed position in the world.

🍦 If the fixed position is `null`, this is functionally identical to [sound](#sound). This method of calling the function was removed in ✨🪽 as it is redundant; just call `sound`.

<!-- tabs:start -->

#### **🍦**

```cpp
void positioned_sound (vec3_t origin, edict_t *ent, int channel, int soundindex, float volume, float attenuation, float timeofs);
```

#### **✨**

```cpp
void positioned_sound (const vec3_t *origin, edict_t *ent, soundchan_t channel, int soundindex, float volume, float attenuation, float timeofs);
```

#### **🪽**

```cpp
void gi_positioned_sound(const vec3_t &in origin, edict_t @ent, soundchan_t channel, int soundindex, float volume, float attenuation, float timeofs)
```

<!-- tabs:end -->

## ✨🪽 local_sound

This function plays a sound directly to a given player rather than being broadcast to everybody. This is a convenience function to address a common need in Quake II modding, and is a replacement for directly constructing an `svc_sound` packet as was required in vanilla mods.

For information about the `dupe_key` parameter, see [unicast](#unicast).

The `target` parameter, when specified, indicates the player entity to send the sound effect to. For overloads without `target`, the sound is sent to `ent`.

<!-- tabs:start -->

#### **✨**

```cpp
void local_sound (edict_t *target, const vec3_t *origin, edict_t *ent, soundchan_t channel, int soundindex, float volume, float attenuation, float timeofs, uint32_t dupe_key);
void local_sound (edict_t *target, const vec3_t &origin, edict_t *ent, soundchan_t channel, int soundindex, float volume, float attenuation, float timeofs, uint32_t dupe_key = 0);
void local_sound (edict_t *target, edict_t *ent, soundchan_t channel, int soundindex, float volume, float attenuation, float timeofs, uint32_t dupe_key = 0);
void local_sound (const vec3_t &origin, edict_t *ent, soundchan_t channel, int soundindex, float volume, float attenuation, float timeofs, uint32_t dupe_key = 0);
void local_sound (edict_t *ent, soundchan_t channel, int soundindex, float volume, float attenuation, float timeofs, uint32_t dupe_key = 0);
```

#### **🪽**

```cpp
void gi_local_sound(edict_t @target, const vec3_t &in origin, edict_t @ent, soundchan_t channel, int soundindex, float volume, float attenuation, float timeofs, uint dupe_key);
void gi_local_sound(edict_t @target, edict_t @ent, soundchan_t channel, int soundindex, float volume, float attenuation, float timeofs, uint dupe_key);

// convenience overloads provided in `engine.as`
void gi_local_sound(edict_t @target, const vec3_t &in origin, edict_t @ent, soundchan_t channel, int soundindex, float volume, float attenuation, float timeofs);
void gi_local_sound(edict_t @target, edict_t @ent, soundchan_t channel, int soundindex, float volume, float attenuation, float timeofs);
void gi_local_sound(const vec3_t &in origin, edict_t @ent, soundchan_t channel, int soundindex, float volume, float attenuation, float timeofs, uint32 dupe_key = 0);
void gi_local_sound(edict_t @ent, soundchan_t channel, int soundindex, float volume, float attenuation, float timeofs, uint32 dupe_key = 0);
```

<!-- tabs:end -->

# Configstrings

Configstrings are special strings, synced with clients, that are used for multiple purposes. See [Configstrings](Configstrings) for more information about what these actually do.

> [!NOTE]
> To prevent making your mod version-specific, always use the [configstring constants](Types/#configstring_id_t) to refer to configstring indices.

## configstring

Write to a configstring.

<!-- tabs:start -->

#### **🍦**

```cpp
void configstring (int num, char *string);
```

#### **✨**

```cpp
void configstring (int num, const char *string);
```

#### **🪽**

```cpp
void gi_configstring(int, const string &in)
```

<!-- tabs:end -->

## ✨🪽 get_configstring

Fetch the current value assigned to the given configstring ID.

<!-- tabs:start -->

#### **✨**

```cpp
const char *get_configstring (int num);
```

#### **🪽**

```cpp
string gi_get_configstring(int num)
```

<!-- tabs:end -->

## modelindex<br/>soundindex<br/>imageindex

These three functions check if an index already exists for a given string, and if so, return that index, otherwise they create a new entry and return the new index.

Each index corresponds to an entry in their respective configstring tables. For more information about these values, see [Model Index](model-index), [Sound Index](sound-index) and [Image Index](image-index).

<!-- tabs:start -->

#### **🍦**

```cpp
void soundindex (char *string);
void modelindex (char *string);
void imageindex (char *string);
```

#### **✨**

```cpp
void soundindex (const char *string);
void modelindex (const char *string);
void imageindex (const char *string);
```

#### **🪽**

```cpp
int gi_soundindex(const string &in str);
int gi_modelindex(const string &in str);
int gi_imageindex(const string &in str);
```

<!-- tabs:end -->

# Memory

Quake II provides a [tagged memory allocator](tagged-memory) for memory tracking & to prevent memory leaks from game modules if they are unable to shutdown properly. See the dedicated page for more information.

## TagMalloc

Allocate a block of memory under the given tag.

<!-- tabs:start -->

#### **🍦**

```cpp
void *TagMalloc (int size, int tag);
```

#### **✨**

```cpp
void *TagMalloc(size_t size, int tag);
```

<!-- tabs:end -->

## TagFree

Free a block of memory previously returned from `TagMalloc`.

<!-- tabs:start -->

#### **🍦**

```cpp
void TagFree (void *block);
```

#### **✨**

```cpp
void TagFree(void *block);
```

<!-- tabs:end -->

## FreeTags

Free all blocks allocated under the specified tag.

<!-- tabs:start -->

#### **🍦**

```cpp
void FreeTags (int tag);
```

#### **✨**

```cpp
void FreeTags(int tag);
```

<!-- tabs:end -->

# Cvars

Cvars are console variables - named variables that contain a string value (and can also be quickly read back as float or int), and also contain a signal for when they are updated. See [cvar_t](Types#cvar_t) for more information about what you can do with the returned cvar pointers.

## cvar

This getter function (`cvar`) creates a new cvar if one does not already exist with the given name, but will not change the value of an existing cvar.

<!-- tabs:start -->

#### **🍦**

```cpp
cvar_t *cvar (char *var_name, char *value, int flags);
```

#### **✨**

```cpp
cvar_t *cvar(const char *var_name, const char *value, cvar_flags_t flags);
```

#### **🪽**

```cpp
cvar_t @gi_cvar(const string &in var_name, const string &in value, cvar_flags_t flags);
```

<!-- tabs:end -->

## cvar_set<br/>cvar_forceset

These two functions will change the value of a cvar; `cvar_forceset` will change the value of a latched cvar, which normally requires a server restart before they can be actually changed.

<!-- tabs:start -->

#### **🍦**

```cpp
cvar_t *cvar_set (char *var_name, char *value);
cvar_t *cvar_forceset (char *var_name, char *value);
```

#### **✨**

```cpp
cvar_t *cvar_set(const char *var_name, const char *value);
cvar_t *cvar_forceset(const char *var_name, const char *value);
```

#### **🪽**

```cpp
void gi_cvar_set(const string &in var_name, const string &in value);
void gi_cvar_forceset(const string &in var_name, const string &in value);
```

<!-- tabs:end -->

# Command Arguments

These functions are available under the context of [`ClientCommand`](Server-Exports#ClientCommand) and [`ServerCommand`](Server-Exports#ServerCommand).

Commands follow a basic tokenizer similar to map parsing, and supports quotes.

## argc

Returns the number of arguments that you can retrieve from `argv`. As an example, for the command `"give quad damage"`, it will return 3.

<!-- tabs:start -->

#### **🍦**

```cpp
int argc (void);
```

#### **✨**

```cpp
int argc();
```

#### **🪽**

```cpp
int gi_argc() nodiscard;
```

<!-- tabs:end -->

## args

Returns the command string **without the first argument**. As an example, for the command `"give quad damage"`, it will return `"quad damage"`.

<!-- tabs:start -->

#### **🍦**

```cpp
char *args (void);
```

#### **✨**

```cpp
const char *args();
```

#### **🪽**

```cpp
const string &gi_args() nodiscard;
```

<!-- tabs:end -->

## argv

Returns an individual token from the command string. As an example, for the command `"give quad damage"`, `argv(0)` will return `"give"`.

<!-- tabs:start -->

#### **🍦**

```cpp
char *argv (int n);
```

#### **✨**

```cpp
const char *argv(int n);
```

#### **🪽**

```cpp
const string &gi_argv(int n) nodiscard;
```

<!-- tabs:end -->

# AddCommandString

Pushes a string into the server's command buffer. Note that this string should end with `\n`.

<!-- tabs:start -->

#### **🍦**

```cpp
void AddCommandString (char *text);
```

#### **✨**

```cpp
void AddCommandString(const char *text);
```

#### **🪽**

```cpp
void gi_AddCommandString(const string &in fmt, const ?&in...);
```

When the varargs version is used, `fmtlib` is used to format the arguments.

<!-- tabs:end -->

# Collision Detection

Quake II provides several functions to test the state of the world for entities. This includes box tracing via a movement sweep function, an AABB gather function, as well as a fast point contents test.

> [!NOTE]
> For entities to be properly included in calls to these functions, [they must be linked](Entity-Lifecycle#linking) into the world. Changes to certain members (like origin) require a re-link, otherwise you may get false positives/negatives from these functions as well as visual glitches.

## trace

`trace` sweeps an axis-aligned box defined by `mins` and `maxs` through the world, from `start` to `end`.

If `passent` is non-`null`, the trace will pass through the entity specified in the trace, *as well as any entity that owns or is owned by this entity*. As an example, if a rocket is fired, it is owned by the player; traces done by the rocket will pass the rocket itself as this argument (as you don't want the trace to impact the rocket itself), which will also cause the rocket to ignore the player. If the player moves, it passes itself in as `passent`, which will cause it to ignore the rocket.

The `contentmask` is the mask of [contents_t](Types#contents_t) that should be impacted by the trace.

The trace is first sweeped against the world model. Then, every entity that crosses the sweeped box will be tested. The server does not directly know what the individual clipping mask is of each entity; instead, the [svflags_t](Types#svflags_t) of an entity determine what kind of contents they are. If the entity has the `svflags_t::DEADMONSTER` server flag and the `contentmask` does not include `contents_t::DEADMONSTER`, the entity will be skipped. (✨🪽 the same is true for `svflags_t::PLAYER` & `contents_t::PLAYER`, and `svflags_t::PROJECTILE` & `contents_t::PROJECTILE`, respectively). If the entity has a `solid_t` of `BBOX`, or a `solid_t` of `TRIGGER` with a non-BSP model, the entity's box is treated as if it was `contents_t::MONSTER`.

> [!NOTE]
> For an entity to be considered in traces at all, the trace *must* include `contents_t::MONSTER`; the constant name is a misnomer and would be better labelled as `ENTITY`. This might be confusing, since the documentation and source state that, for instance, `svflags_t::DEADMONSTER` is supposed to 'treat' the entity as `contents_t::DEADMONSTER`, but in reality this is just done as a filter. All non-bmodel entity boxes are always treated as `contents_t::MONSTER`.

> [!WARNING]
> 🍦 if `startsolid` or `allsolid` is true, a few members of the returned trace structure may be incorrect due to a bug in the entity testing code; `fraction`, `plane`, `surface` and `contents` may contain the results of a prior tested entity rather than the one that the trace actually got stuck inside. This bug was fixed in the re-release.

<!-- tabs:start -->

#### **🍦**

```cpp
trace_t	trace (vec3_t start, vec3_t mins, vec3_t maxs, vec3_t end, edict_t *passent, int contentmask);
```

If `mins` or `maxs` are `null`, they will use the same value as `vec3_origin` (all zeroes).

#### **✨**

```cpp
trace_t trace(const vec3_t &start, const vec3_t *mins, const vec3_t *maxs, const vec3_t &end, const edict_t *passent, contents_t contentmask);
trace_t trace(const vec3_t &start, const vec3_t &mins, const vec3_t &maxs, const vec3_t &end, const edict_t *passent, contents_t contentmask);
trace_t traceline(const vec3_t &start, const vec3_t &end, const edict_t *passent, contents_t contentmask);
```

If `mins` or `maxs` are `null`, they will use the same value as `vec3_origin` (all zeroes).

A convenience overload is provided for tracing a line (using `vec3_origin` as mins/maxs).

#### **🪽**

```cpp
trace_t gi_trace(const vec3_t &in start, const vec3_t &in mins, const vec3_t &in maxs, const vec3_t &in end, edict_t @passent, contents_t contentmask) nodiscard;
trace_t gi_traceline(const vec3_t &in start, const vec3_t &in end, edict_t @passent, contents_t contentmask) nodiscard;
```

A convenience overload is provided for tracing a line (using `vec3_origin` as mins/maxs).

<!-- tabs:end -->

## ✨🪽 clip

`clip` is a lower level trace function that allows you to sweep a box against a particular entity. This can be used to create custom collision detection routines that may not be possible with `trace` without a lot of extra work.

All of the details of [`trace`](#trace) apply, except that there is no `passent`; instead, you simply provide a box and it is clipped against the box (or brush model) of the entity passed in the first argument.

<!-- tabs:start -->

#### **✨**

```cpp
trace_t clip(edict_t *entity, const vec3_t &start, const vec3_t *mins, const vec3_t *maxs, const vec3_t &end, contents_t contentmask);
```

#### **🪽**

```cpp
trace_t gi_clip(edict_t @entity, const vec3_t &in start, const vec3_t &in mins, const vec3_t &in maxs, const vec3_t &in end, contents_t) nodiscard;
```

<!-- tabs:end -->

## pointcontents

This is a simple & fast function to return the [`contents_t`](Types#contents_t) mask of the world at a given location.

> [!NOTE]
> Due to an oversight, it is possible to get false positives from content types that don't cause BSP splits (such as `PLAYERCLIP`). The only way to guarantee that the position doesn't include that content type is to do a box trace at the same location.

<!-- tabs:start -->

#### **🍦**

```cpp
int	pointcontents (vec3_t point);
```

#### **✨**

```cpp
contents_t pointcontents(const vec3_t &point);
```

#### **🪽**

```cpp
contents_t gi_pointcontents(const vec3_t &in point) nodiscard;
```

<!-- tabs:end -->

## BoxEdicts

`BoxEdicts` is a fast way of gathering entities that intersect a given axis-aligned bounding box.

The `areatype` parameter indicates whether you wish to query a list of `SOLID` or `TRIGGER` entities; see [area_solidity_t](Types#area_solidity_t).

✨🪽 In re-release, this function was re-tooled to be more useful. If `filter` is null and `maxcount` is not 0, the behavior is identical to vanilla. Otherwise, some new rules apply:
* if `maxcount` is 0, the value of `list` is ignored and the function returns the number of entities that *would* have been captured into the list. This shouldn't be used for pre-allocation, however, as that requires two passes which is wasteful.
* if `filter` is non-`null`, the filter function will be called with the entity handle being tested against and the `filter_data` pointer passed into the function. The filter function returns a [BoxEdictsResult_t](Types#BoxEdictsResult_t), which dictates whether the entity should be put into the `list` and whether the search can be stopped early. The filter can also be used to dynamically capture results instead of trying to capture them into a fixed-size list.

<!-- tabs:start -->

#### **🍦**

```cpp
int BoxEdicts (vec3_t mins, vec3_t maxs, edict_t **list, int maxcount, int areatype);
```

The function will fill `list` with at most `maxcount` number of entities, and returns the total number of entities it found.

> [!WARNING]
> This function will only return a number up to `maxcount`; there is no way of knowing whether there were more entities that weren't added to the list.

#### **✨**

```cpp
size_t BoxEdicts(const vec3_t &mins, const vec3_t &maxs, edict_t **list, size_t maxcount, solidity_area_t areatype, BoxEdictsFilter_t filter, void *filter_data);
```

#### **🪽**

```cpp
uint gi_BoxEdicts(const vec3_t &in mins, const vec3_t &in maxs, array<edict_t@> @+list, uint maxcount, solidity_area_t areatype, BoxEdictsFilter_t @+filter, any @const+ filter_data, bool append);
```

Since arrays are a first-class feature compared to the C ABI, the array is directly passed in as a handle and written to from this function.

If `append` is true, the new entities are instead appended to the end of `list`; if it is false, the array is cleared before appending entities.

<!-- tabs:end -->

## 🍦 Pmove

In vanilla engines, the game imports the player movement function from the server, which is shared with the client code.

For more information about how to do this in re-release, see [Player Lifecycle](Player-Lifecycle).


<!-- tabs:start -->

#### **🍦**

```cpp
void Pmove (pmove_t *pmove);
```

<!-- tabs:end -->

# Entity Functions

## setmodel

In Quake II, this function is only really useful on brush models. It sets the modelindex to the correct value for a brush model `model` key from a map, and also sets the `mins`/`maxs` of the entity to the brush models' size and links it. For any other type of model, this is identical to directly setting `modelindex` to the string provided.

<!-- tabs:start -->

#### **🍦**

```cpp
void setmodel (edict_t *ent, char *name);
```

#### **✨**

```cpp
void setmodel(edict_t *ent, const char *name);
```

#### **🪽**

```cpp
void gi_setmodel(edict_t @ent, const string &in name);
```

<!-- tabs:end -->

## linkentity<br/>unlinkentity

These functions link and unlink the entity into the world, respectively. See [linking](Entity-Lifecycle#linking) for more information about what exactly this does.

<!-- tabs:start -->

#### **🍦**

```cpp
void linkentity (edict_t *ent);
void unlinkentity (edict_t *ent);
```

#### **✨**

```cpp
void linkentity(edict_t *ent);
void unlinkentity(edict_t *ent);
```

#### **🪽**

```cpp
void gi_linkentity(edict_t @ent);
void gi_unlinkentity(edict_t @ent);
```

<!-- tabs:end -->

## 🪽 find_by_str

For performance reasons, `G_Find` is not implemented on the scripting side. This function mimics the old functionality, and is a templated function so it can be used on different types in theory.

For more information about this function, see [Finding Entities](Useful-Functions#finding-entities).

<!-- tabs:start -->

#### **🪽**

```cpp
T @+find_by_str<T>(T @+from, const string &in member, const string &in value) nodiscard;
```

<!-- tabs:end -->

# Visibility<br/>Hearability

The game API provides a few functions to query or modify the visibility/hearability state of the world.

[See PVS for more information about how these mechanics work](PVS).

## inPVS<br/>inPHS

These functions return whether two given points can see or hear each other, respectively.

✨🪽 In addition, a new parameter was added to decide whether area portals are considered in this check or not.

<!-- tabs:start -->

#### **🍦**

```cpp
qboolean inPVS (vec3_t p1, vec3_t p2);
qboolean inPHS (vec3_t p1, vec3_t p2);
```

#### **✨**

```cpp
bool inPVS(const vec3_t &p1, const vec3_t &p2, bool portals);
bool inPHS(const vec3_t &p1, const vec3_t &p2, bool portals);
bool inPVS(const vec3_t &p1, const vec3_t &p2);
bool inPHS(const vec3_t &p1, const vec3_t &p2);
```

Two convenience functions are provided, which default `portals` to true, to match vanilla.

#### **🪽**

```cpp
bool gi_inPHS(const vec3_t &in p1, const vec3_t &in p2, bool portals) nodiscard;
bool gi_inPVS(const vec3_t &in p1, const vec3_t &in p2, bool portals) nodiscard;

// convenience overloads provided in `engine.as`

bool gi_inPHS(const vec3_t &in a, const vec3_t &in b) nodiscard;
bool gi_inPVS(const vec3_t &in a, const vec3_t &in b) nodiscard;
```

<!-- tabs:end -->

## AreasConnected

Returns `true` if the two area numbers are currently connected (flood into each other). If they are `false`, at least one areaportal between them is closed.

<!-- tabs:start -->

#### **🍦**

```cpp
qboolean AreasConnected (int area1, int area2);
```

#### **✨**

```cpp
bool AreasConnected(int area1, int area2);
```

#### **🪽**

```cpp
bool gi_AreasConnected(int area1, int area2) nodiscard;
```

<!-- tabs:end -->

## SetAreaPortalState

Toggle the state of the given area portal. By default, all area portals are closed.

<!-- tabs:start -->

#### **🍦**

```cpp
void SetAreaPortalState (int portalnum, qboolean open);
```

#### **✨**

```cpp
void SetAreaPortalState(int portalnum, bool open);
```

#### **🪽**

```cpp
void gi_SetAreaPortalState(int portalnum, bool open);
```

<!-- tabs:end -->

# ✨🪽 Constants

The API provides a few constants that correspond to features that cannot be changed during gameplay, instead of needing to calculate them yourself via the cvars.

## tick_rate

This corresponds to the `sv_tick_rate` cvar, and is the tick rate in hz.

<!-- tabs:start -->

#### **✨**

```cpp
uint32_t tick_rate;
```

#### **🪽**

```cpp
const uint gi_tick_rate;
```

<!-- tabs:end -->

## frame_time_s

This corresponds to the amount of time, in seconds, that a single game frame takes. In vanilla, this was the `FRAMETIME` macro.

<!-- tabs:start -->

#### **✨**

```cpp
float frame_time_s;
```

#### **🪽**

```cpp
const float gi_frame_time_s;
```

<!-- tabs:end -->

## frame_time_ms

This corresponds to the amount of time, in milliseconds, that a single game frame takes. In vanilla, this was the `FRAMETIME` macro multiplied by 1000.

<!-- tabs:start -->

#### **✨**

```cpp
uint32_t frame_time_ms;
```

#### **🪽**

```cpp
const uint gi_frame_time_ms;
```

<!-- tabs:end -->

# Networking

You can directly write packets to players to create effects and change certain client state, like the current layout string.

For information about all of the different network messages and packets you can actually write, see the dedicated [Networking](Networking) page.

## WriteChar<br/>WriteByte<br/>WriteShort<br/>WriteLong<br/>✨🪽 WriteFloat<br/>WriteString<br/>WritePosition<br/>WriteDir<br/>WriteAngle<br/>✨🪽 WriteEntity

These functions write the given type into the active buffer. To commit the current packet, use [unicast](#unicast) or [multicast](#multicast).

> [!NOTE]
> Position, Dir, Angle, and ✨🪽 Entity should always be used instead of trying to write those types byte-wise, as these functions will handle the encoding/decoding of these types (Position and Dir may be encoded differently, as an example).

<!-- tabs:start -->

#### **🍦**

```cpp
void WriteChar (int c);
void WriteByte (int c);
void WriteShort (int c);
void WriteLong (int c);
void WriteString (char *s);
void WritePosition (vec3_t pos);
void WriteDir (vec3_t pos);
void WriteAngle (float f);
```

> [!WARNING]
> 🍦 `WriteFloat` has a function, but was never implemented and should not be used.

#### **✨**

```cpp
void WriteChar(int c);
void WriteByte(int c);
void WriteShort(int c);
void WriteLong(int c);
void WriteFloat(float f);
void WriteString(const char *s);
void WritePosition(const vec3_t &pos);
void WriteDir(const vec3_t &pos);
void WriteAngle(float f);
void WriteEntity(const edict_t *e);
```

#### **🪽**

```cpp
void gi_WriteByte(int c);
void gi_WriteChar(int c);
void gi_WriteShort(int c);
void gi_WriteLong(int c);
void gi_WriteFloat(float f);
void gi_WriteString(const string &in s);
void gi_WritePosition(const vec3_t &in pos);
void gi_WriteDir(const vec3_t &in dir);
void gi_WriteAngle(float f);
void gi_WriteEntity(edict_t @ent);
```

<!-- tabs:end -->

## unicast

Send the current active buffer to a single entity & flush the buffer.

See [Reliability](Network#reliability) for information about the `reliable` parameter.

✨🪽 See [Dupe Key](Networking#dupe-key) for information about the `dupe_key` parameter.

<!-- tabs:start -->

#### **🍦**

```cpp
void unicast (edict_t *ent, qboolean reliable);
```

#### **✨**

```cpp
void unicast(edict_t *ent, bool reliable, uint32_t dupe_key);
void unicast(edict_t *ent, bool reliable);
```

A convenience overload is provided that uses no dupe_key.

#### **🪽**

```cpp
void gi_unicast(edict_t @ent, bool reliable, uint dupe_key);
void gi_unicast(edict_t @ent, bool reliable);
```

A convenience overload is provided that uses no dupe_key.

<!-- tabs:end -->

## multicast

Send the current active buffer to all clients & flush the buffer.

See [Reliability](Network#reliability) for information about the `reliable` parameter.

See [multicast_t](Types#multicast_t) for information about the `to` parameter.

<!-- tabs:start -->

#### **🍦**

```cpp
void multicast (vec3_t origin, multicast_t to);
```

#### **✨**

```cpp
void multicast(const vec3_t &origin, multicast_t to, bool reliable);
```

#### **🪽**

```cpp
void gi_multicast(const vec3_t &in origin, multicast_t to, bool reliable);
```

<!-- tabs:end -->

# Debugging

## ✨🪽 Draw_Line<br/>✨🪽 Draw_Point<br/>✨🪽 Draw_Circle<br/>✨🪽 Draw_Bounds<br/>✨🪽 Draw_Sphere<br/>✨🪽 Draw_OrientedWorldText<br/>✨🪽 Draw_StaticWorldText<br/>✨🪽 Draw_Cylinder<br/>✨🪽 Draw_Ray<br/>✨🪽 Draw_Arrow

Several functions are provided to draw simple wireframe lines in the world. These are super handy for debugging.

<!-- tabs:start -->

#### **✨**

```cpp
void Draw_Line(const vec3_t &start, const vec3_t &end, const rgba_t &color, const float lifeTime, const bool depthTest);
void Draw_Point(const vec3_t &point, const float size, const rgba_t &color, const float lifeTime, const bool depthTest);
void Draw_Circle(const vec3_t &origin, const float radius, const rgba_t &color, const float lifeTime, const bool depthTest);
void Draw_Bounds(const vec3_t &mins, const vec3_t &maxs, const rgba_t &color, const float lifeTime, const bool depthTest);
void Draw_Sphere(const vec3_t &origin, const float radius, const rgba_t &color, const float lifeTime, const bool depthTest);
void Draw_OrientedWorldText(const vec3_t &origin, const char * text, const rgba_t &color, const float size, const float lifeTime, const bool depthTest);
void Draw_StaticWorldText(const vec3_t &origin, const vec3_t &angles, const char * text, const rgba_t & color, const float size, const float lifeTime, const bool depthTest);
void Draw_Cylinder(const vec3_t &origin, const float halfHeight, const float radius, const rgba_t &color, const float lifeTime, const bool depthTest);
void Draw_Ray(const vec3_t &origin, const vec3_t &direction, const float length, const float size, const rgba_t &color, const float lifeTime, const bool depthTest);
void Draw_Arrow(const vec3_t &start, const vec3_t &end, const float size, const rgba_t & lineColor, const rgba_t & arrowColor, const float lifeTime, const bool depthTest);
```

#### **🪽**

```cpp
void gi_Draw_Line(const vec3_t &in start, const vec3_t &in end, const rgba_t &in color, float lifeTime, bool depthTest);
void gi_Draw_Point(const vec3_t &in point, float size, const rgba_t &in color, float lifeTime, bool depthTest);
void gi_Draw_Circle(const vec3_t &in origin, float size, const rgba_t &in color, float lifeTime, bool depthTest);
void gi_Draw_Bounds(const vec3_t &in mins, const vec3_t &in maxs, const rgba_t &in color, float lifeTime, bool depthTest);
void gi_Draw_Sphere(const vec3_t &in origin, float radius, const rgba_t &in color, float lifeTime, bool depthTest);
void gi_Draw_OrientedWorldText(const vec3_t &in origin, const string &in text, const rgba_t &in color, float size, float lifeTime, bool depthTest);
void gi_Draw_StaticWorldText(const vec3_t &in origin, const vec3_t &in angles, const string &in text, const rgba_t &in color, float size, float lifeTime, bool depthTest);
void gi_Draw_Cylinder(const vec3_t &in origin, float halfHeight, float radius, const rgba_t &in color, float lifeTime, bool depthTest);
void gi_Draw_Arrow(const vec3_t &in start, const vec3_t &in end, float size, const rgba_t &in lineColor, const rgba_t &in arrowColor, float lifeTime, bool depthTest);
```

<!-- tabs:end -->

## ✨🪽 SendToClipBoard

<!-- tabs:start -->

#### **✨**

```cpp
void SendToClipBoard(const char *text);
```

#### **🪽**

```cpp
void SendToClipBoard(const string &in text);
```

<!-- tabs:end -->

# Other Functions

## ✨🪽 ServerFrame

This function simply fetches the current server frame number. It's mainly useful to sync up time on the client, since the client can also retrieve the server frame number that it's currently rendering.

<!-- tabs:start -->

#### **✨**

```cpp
uint32_t ServerFrame();
```

#### **🪽**

```cpp
uint gi_ServerFrame() nodiscard;
```

<!-- tabs:end -->
