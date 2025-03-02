Server imports (referred to as 'game imports' interchangeably) are the functions that the server provides to the game module, and allow the game module to interact with systems that the game does not have any direct control over.

We refer to these as *server* imports because we're writing from the POV of the game module (the game is importing functions from the server and exporting functions back to the server), and re-release has a client game (or cgame) module which also provides their own imports/exports.

> [!NOTE]
> The documentation here presents the functions as if they were globals; this is to simplify the documentation and not confuse programmers with the syntax of function pointers in C/C++.
>
> 🍦✨ All imports are accessed through the `gi` global, which is a static variable of `game_import_t` that contains pointers to all of the below functions.<br/>
> 🪽 All imports are global functions, prefixed with `gi_`.

# Printing

The API provides several functions to print messages to the server console & to other users.

✨🪽 These functions were renamed away from their legacy Quake counterparts, and now do not provide any sort of formatting support directly, instead relying on the game module to bring their own formatting library. This was to help support other languages as well as provide a free performance improvement when a string without any format specifiers were used.

## bprint / Broadcast_Print

This function prints a message to all players. A print type can be specified to change the priority level of the message.

```cpp
🍦 void bprintf (int printlevel, char *fmt, ...);
```

```cpp
✨ void Broadcast_Print(print_type_t printlevel, const char *message);
```

```clike
🪽 void gi_Broadcast_Print(print_type_t printlevel, const string &in message);
🪽 void gi_Broadcast_Print(print_type_t printlevel, const string &in fmt, const ?&in...);
```