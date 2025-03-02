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