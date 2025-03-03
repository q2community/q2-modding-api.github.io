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

## bprintf / Broadcast_Print

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

## dprintf / Com_Print

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

## cprintf / Client_Print

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

## centerprintf / Center_Print

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

## error / Com_Error

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

## modelindex / soundindex / imageindex

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

Quake II provides a [[tagged memory allocator|tagged-memory]] for memory tracking & to prevent memory leaks from game modules if they are unable to shutdown properly. See the dedicated page for more information.

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

## cvar_set / cvar_forceset

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

