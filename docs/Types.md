# Types

This page goes through the most common Quake II-specific types you'll come across in Quake II modding, and explains their purpose & how to manipulate them. This will cover the mathematical & engine-shared types first, and only briefly go into detail on specific types that are used in the vanilla Quake II mod experience.

## `vec3_t`

A 3d vector; it might represent a position, a unit vector, a vector * magnitude, or even Euler angles. There are several global functions to manipulate them - most of them have `vec` in their name somewhere (✨🪽 and member functions of vec3_t, which is where most of the globals were moved to).

🍦 Be careful when passing vectors as parameters, as the type is a simple typedef to a C array, and these have weird semantics.

## 🍦 `qboolean`

Quake II's initial source was written pre-C99 and didn't have a native boolean type. As a result, it uses a custom enum named `qboolean` with `false` and `true` values. In re-release, this was changed to the canonical `bool` type (which, in C, matches `stdbool.h`'s `_Bool`).

## time

Time in Quake II is represented as a number that is kept track of on the game side. The game [runs a level tick](Level-Lifecycle.md) which increases the current time of the level, and this value persists separately on levels (every new level always starts at a time of zero).

🍦 Times are represented as `float` **seconds** in most cases, and in a few other cases they are stored as `int` **frames**. A frame in Quake II is 100 milliseconds (10hz). The `FRAMETIME` macro contains the number of seconds in a frame (0.1) which can be used to convert to other units.

✨🪽 Times are represented as `int64` and stored in the `gtime_t` type. It contains several functions to create times from different units, as well as converting those times back into different components. (✨ You can also use the `_ms`, `_sec`, etc literal postfixes to easily create constants of units of time.) The game imports some useful constants from the server for timing, such as [`frame_time_s`](Server-Imports#frame_time_s) and [`frame_time_ms`](Server-Imports#frame_time_ms), to use for calculations.

## `print_type_t`

A print priority level, that tells the client how important the message is and whether it should be filtered or not.

In most engines, the `msg` userinfo cvar can be used to filter out certain types of messages that you don't want to receive.

🍦✨ The enum constants are globals prefixed with the string `PRINT_` to prevent name clashes.

✨🪽 There are also bitflags that can be mixed with any level type.

| Member | Description |
| --- | --- |
| LOW | Low priority messages, such as pickups. |
| MEDIUM | Medium priority messages, such as death messages. |
| HIGH | High priority messages; anything that is more important than the other two. |
| CHAT | A chat message. This message is also green and makes a specific sound on the client when displayed. |
| ✨🪽 TYPEWRITER | The message is displayed in the center of the screen, one character at a time like a typewriter. This is used for level goals in the re-release. |
| ✨🪽 CENTER | The message is displayed in the center of the screen all at once. |
| ✨🪽 TTS | Like `HIGH`, but will be spoken audibly if the engine supports text-to-speech. |
| ✨🪽 BROADCAST | Bitflag that can be mixed with any of the above types. If set, `Loc_Print` will deliver this message to all clients. |
| ✨🪽 NO_NOTIFY | Bitflag that can be mixed with any of the above types. If set, clients will only display this message in the console and not on the notification bar. |

## `area_solidity_t`

The type of area you want to grab from a call to [BoxEdicts](Server-Imports#BoxEdicts).

🍦✨ The enum constants are globals prefixed with the string `AREA_` to prevent name clashes.

| Member | Description |
| --- | --- |
| SOLID | Includes any entity that is `SOLID_BBOX` or `SOLID_BSP`. |
| TRIGGER | Includes any entity that is `SOLID_TRIGGER`. |

## `entity_state_t`

The entity state stores the data that is used for transmission to the client. It basically describes everything the client needs to render a given entity, as well as some other data that the server uses for things like collision.

| Member | Description |
| --- | --- |
| number | The numeric index of this entity. This should never be touched. |
| origin | The entity's position in the world. [Changing this requires re-linking the entity.](Entity-Lifecycle#linking) |
| angles | The entity's Euler angles. [Changing this may require re-linking the entity if it is a brush model.](Entity-Lifecycle#linking) |
| old_origin | The entity's previous in the world. This is mainly used for interpolating entities on the client side, but some types of entities (like `RF_BEAM`) will use this for a secondary position. |
| modelindex | [Model index](model-index) |
| modelindex2 | Secondary [model index](model-index) |
| modelindex3 | Third [model index](model-index) |
| modelindex4 | Fourth [model index](model-index) |
| frame | Index of frame used for animating sprites, brush model textures or models. It is sometimes also used to represent colors (like four palette indices for `RF_BEAM`) or overloaded for other purposes. |
| skinnum | Index of skin used for models. It is sometimes overloaded for other purposes. |
| effects | Bit flags of effects on this entity. See [effects_t](Types#effects_t). |
| renderfx | Bit flags of rendering effects on this entity. See [renderfx_t](Types#renderfx_t). |
| solid | Packed solidity data. This is managed automatically by the server, do not touch! |
| sound | [Sound index](sound-index) - looped sound that plays from this entity. |
| event | Event index; events are played once when set and the client receives it, and is reset to zero at the start of every frame. |
| ✨🪽 alpha | Alpha scalar. For backwards compatibility, a value of `0.0` is the same as a value of `1.0`. |
| ✨🪽 scale | Entity scale scalar. For backwards compatibility, a value of `0.0` is the same as a value of `1.0`. |
| ✨ instance_bits | For split-screen players, this controls the visibility of an entity for each split screen player. Managed by the server code, do not touch. |
| ✨🪽 loop_volume | Volume for looped sounds. For backwards compatibility, `0.0` will equal `1.0`. Note that as of Update 1, this field does not work correctly on the client side. |
| ✨&nbsp;🪽&nbsp;loop_attenuation | Volume for looped sounds. For backwards compatibility, `0.0` will equal `3.0`. A value of `-1` indicates no attenuation. |
| ✨ owner | Entity owner index; this is used to allow client prediction to properly handle collisions against solid entities that "own" us (such as teslas). It is automatically managed by the server; do not touch. |
| ✨🪽 old_frame | When `RF_OLD_FRAME_LERP` is set on `renderfx`, this value is transmitted over the network and forces interpolation from this frame to the current frame. It is useful when you wish to smooth a transition from an animation that might have ugly interpolation artifacts (like the Enforcer's head being shot off). |

## `edict_t`

Stores the data that is transmitted between the server & client, as well as various bits of data that the server uses for things like collision. (🍦✨ This struct is also where you add new members that you can use for whatever purpose on entities.)

| Member | Description |
| --- | --- |
| s | [Entity state; see entity_state_t](Types#entity_state_t) |
| client | [Pointer to client struct; see gclient_t](Types#gclient_t) |
| ✨🪽 sv | [State data for bots; see sv_entity_t](Types#sv_entity_t) |
| inuse | Whether the entity is currently active or not. See [Entity Lifecycle](entity-lifecycle). |
| 🍦 area | Pointers to other entities that are linked into the same world area. This is read-only; modifying this will have negative consequences. To check if an entity is linked into the world, check if prev or next is non-null. See [Entity Lifecycle](entity-lifecycle). |
| ✨🪽 linked | Whether the entity is currently linked into the world. See [Entity Lifecycle](entity-lifecycle). |
| linkcount | A running tally on how many times this entity has been linked. This can be useful to tell if an entity has been moved by something else, in which case you may want to run additional checks (for instance, to check if you've been moved by a platform), by comparing it against a previously-known value. |
| 🍦&nbsp;num_clusters | The number of clusters we are linked into. Read-only, not useful to the game code. |
| 🍦&nbsp;clusternums | The clusters we are linked into. Read-only, not useful to the game code. |
| 🍦&nbsp;headnode | The headnode we are linked into, if we overrun the cluster count. Read-only, not useful to the game code. |
| areanum, areanum2 | The current areas this entity is linked into. Entities can be linked into two areas, if they are straggling a water brush or standing inside of a door for instance. See [Areas](Areas). (🍦 this is different/unrelated to the `area` member) |
| svflags | A bit set of flags the server & game use for various things. See [svflags_t](Types#svflags_t) |
| mins, maxs | The entity's size in the world. This is an axis-aligned bounding box. [Changing this requires re-linking the entity.](Entity-Lifecycle#linking) |
| absmin, absmax | The entity's absolute box in the world, expanded by 1 on each axis. This is the same as (origin + mins) - { 1, 1, 1 } and (origins + maxs) + { 1, 1, 1 } respectively, *except* for brush models where this value may also be expanded to include rotation. This is read-only. |
| size | The entity's size (mins + maxs). This is read-only. |
| solid | The entity's solidity type. `TRIGGER` and `BBOX` are both usable by any entity, and simply link the entity into either trigger or solid area links, respectively. The `BSP` value can only be used on brush models (entities whose modelindex points to an inline BSP model, such as `*2`). [Changing this requires re-linking the entity.](Entity-Lifecycle#linking) |
| clipmask | The entity's content mask. The server does not use this field, it is only in this part of the structure for historical reasons. Physics routines of the game use this to control which entities this entity will touch when moving. |
| owner | The entity that owns this entity. When non-null, traces that have an `ignore` field will also ignore the owner (or ownee); for instance, shots fired from your own weapon are owned by you and ignore their owner when they move, which allows them to pass through you and vice versa. |

## `player_state_t`

Stores the information needed for the client to render a view.

| Member | Description |
| --- | --- |
| pmove | [Player movement state; see pmove_state_t](Types#pmove_state_t) |
| viewangles | 3D vector representing the player's view in Euler angles. |
| viewoffset | 3D vector representing an offset for the player's view. Used to add bob and other effects. |
| kick_angles | 3D vector the holds values for weapon kick angles used when calculating the view offset. |
| gunangles | 3D vector representing the angle of the gun the player is holding. Used to rotate the gun when moving and bobbing. |
| gunoffset | 3D vector representing an offset for the player's gun.  |
| gunindex | Index to the gun model being used. |
| ✨🪽 gunskin | Skin number used on the weapon. |
| gunframe | Frame index to current animation for the player's gun. |
| ✨🪽 gunrate | The gun framerate specified in hz. A value of 0 is equivalent of 10hz for backwards compatability. This is used to speed up framerate in case of powerups like haste. |
| 🍦&nbsp;blend<br>✨&nbsp;🪽&nbsp;screen_blend | Full-screen color change value, used for screen flashing. |
| ✨&nbsp;🪽&nbsp;damage_blend| New type of blend that only occurs around the edge of the screen; many effects are now using this rather than full-screen blend, for epilepsy reasons. |
| fov | Player's horizontal field of view. |
| rdflags | Flags attached to the player's refdef (refresh definition) that affect the entire scene. See [Player movement state; see refdef_flags_t](Types#refdef_flags_t) |
| stats | Values used for status bar updates; this is an array of 32 (✨🪽 64) `short`s transmitted to the client, which is then used by [the HUD layout string](HUD Layout) for rendering the HUD and other layouts |
| ✨🪽 team_id | Used for team-oriented gamemode to specify the player's team. |

## `cvar_flags_t`

Flags used to define different cvar behaviours. In 🍦 these flags are defined as constant values while in ✨🪽 it is an enum type.

🍦✨ The enum constants are globals prefixed with the string `CVAR_` to prevent name clashes.

| Flag Name | Description |
| --- | --- |
| ✨🪽 NOFLAGS | Representation for no flags; the same as zero. |
| ARCHIVE | Causes the cvar to be saved to the config file. |
| USERINFO | Sends the cvar value to the server as part of the player's userinfo (name, model, skin, etc). |
| SERVERINFO | Sends the cvar value to the client as part of server's info (game rules, max players, etc). |
| NOSET | Prevents the cvar to be changed from the console at all, but can still be modified from the command line. |
| LATCH | Prevents the cvar from taking effect immediately instead the value is latchedand applied after a map change or game restart. |
| ✨&nbsp;🪽&nbsp;USER_PROFILE | New flag that is only for the client/cgame; it indicates that a cvar is treated like userinfo, but not synced with the server. This is to allow cloud saved client-side config values that aren't part of userinfo. |

## `cvar_t`

Representation of a console variable. These are variables that is reachable through the in-game console and can be used to store, retrieve and modify engine, gameplay and user settings at runtime.

| Member | Description |
| --- | --- |
| name | Name of the cvar. |
| string<br>🪽 stringval | String representation of the cvar's value.<br/>(🪽⚠️ This creates a copy of the string, and **will** be expensive if called every frame. The value of the string should be cached, and `modified_count` used to see if it is modified from its old value.) |
| latched_string<br>🪽 latched_stringval| Used for latched values (Changes that take effect after restart).<br/>(🪽⚠️ This creates a copy of the string, and **will** be expensive if called every frame. The value of the string should be cached, and `modified_count` used to see if it is modified from its old value.) |
| flags | [Cvar flags; see cvar_flags_t](Types#cvar_flags_t)  |
| 🍦&nbsp;modified<br>✨&nbsp;🪽&nbsp;modified_count | 🍦boolean representing if this cvar has been changed or not.<br/>✨🪽 An integer that counts the number of times modified instead. This value will never be zero, which allows you to always check if the cvar has been modified even in its default state. |
| value | Floating point representation of the cvar's value. |
| ✨🪽 integer | Integral representation of the cvar's value. |
| 🪽 boolean | Property that returns true if the cvar is non-zero. |
| 🍦✨ next | Linked list pointer to the next cvar. This is internal; do not touch. |

## `contents_t`

Content flags are used to determine properties of solid geometry in the game world. These flags determine how entities interact with brushes, and are returned in [game imports that query the world, such as trace and pointcontents](Server-Imports). They are bitflags, and a brush can contain multiple contents. In 🍦 these flags are defined as constant values while in ✨🪽 it is an enum type.

🍦✨ The enum constants are globals prefixed with the string `CONTENTS_` to prevent name clashes.

> [!NOTE]
> The first 7 bits are mutually exclusive, and create visible surfaces. 

| Member | Description |
| --- | --- |
| ✨🪽 NONE | Representation for no flags; the same as zero. |
| SOLID | Solid surface, players and objects cannot move through it. |
| WINDOW | A transparent but solid surface, like glass. Players and objects cannot pass through it but does not completely block light. |
| AUX | Identical to MIST; historically used for detail objects. |
| LAVA | Lava surface; damages players and entities that touches it. |
| SLIME | Toxic slime surface; damages players and entities that touches it. |
| WATER | Water volumes; this allows swimming and other water physics. |
| MIST | Extra visible content type that, by default, has no interaction with entities. |
| ✨🪽 NO_WATERJUMP | Flag to make water not allow jumping; used for traps where waterjump should be disallowed. | 
| ✨🪽 PROJECTILECLIP | Blocks movement from entities with [svflags_t::PROJECTILE](Types#svflags_t) server flags. | 
| AREAPORTAL | Used for `func_areaportal`s. (✨🪽 also used as a special value by the game) |
| PLAYERCLIP | Blocks movement from entities with [svflags_t::PLAYER](Types#svflags_t) server flags. |
| MONSTERCLIP | Monsters include this flag in their clipmask. |
| CURRENT_0 | Flow direction used to define current direction of fluid or conveyor belts; flowing horizontally in direction of 0 degrees.|
| CURRENT_90 | Flowing horizontally in direction 90 degrees. |
| CURRENT_180 | Flowing horizontally in direction 180 degrees. |
| CURRENT_270 |  Flowing horizontally in direction 270 degrees. |
| CURRENT_UP | Fluid flow pushes entities upwards. |
| CURRENT_DOWN | Fluid flow pushes entities downwards. |
| ORIGIN | Used for rotting entities. | 
| MONSTER | Monsters include this flag in their clipmask when alive. |
| DEADMONSTER | Blocks movement from entities with [svflags_t::DEADMONSTER](Types#svflags_t) server flags. Used for things like gibs and dead monsters. (🍦 `fire_blaster` mentions that projectiles use this for prediction purposes, but this is untrue and was simply a misunderstanding on what this flag does) |
| DETAIL | Used for small decorative geometry that does not affect visibility calculations. |
| TRANSLUCENT | Used to specify if geometry is transparent. | 
| LADDER | Marks a ladder brush allowing the player to climb it. | 
| ✨🪽 PLAYER | Used only on player entities, allows tracing to exclude / include them. | 
| ✨🪽 PROJECTILE | Used only on projectiles, allows tracing to exclude / include them. |

There are also several built-in `MASK`s, which provide simple masks that are used in various routines in Q2, to prevent needing to type them out every time.

🪽 The mask enums are children of `contents_t`. This may change in the future.

| Constant | Description |
| --- | --- |
| MASK_ALL | Hits every content type. |
| MASK_SOLID | Only the fully solid contents (`SOLID` and `WINDOW`). |
| MASK_PLAYERSOLID | Content types that player movement considers solid (`SOLID`, `PLAYERCLIP`, `WINDOW`, `MONSTER`, ✨🪽 and `PLAYER`) |
| MASK_DEADSOLID | Content mask used for dead players (`SOLID`, `PLAYERCLIP`, `WINDOW`). |
| MASK_MONSTERSOLID | Content mask used for monster movement (`SOLID`, `MONSTERCLIP`, `WINDOW`, `MONSTER`, ✨🪽 and `PLAYER`). |
| MASK_WATER | Hits every liquid-like content type. |
| MASK_OPAQUE | Hits any surface the game considers opaque (`SOLID`, `SLIME`, `LAVA`). This isn't entirely always true, though, since slime and lava *can* be translucent, but it often isn't. This is mainly used for things like visibility routines. |
| MASK_SHOT | Content types hitscan shots can collide with (`SOLID`, `MONSTER`, `WINDOW`, `DEADMONSTER`, ✨🪽 and `PLAYER`) |
| MASK_CURRENT | Mask that includes all of the `CURRENT_` flags. |
| ✨🪽 MASK_BLOCK_SIGHT | Used by bot code; things the bots consider sight-blocking (`SOLID`, `LAVA`, `SLIME`, `MONSTER`, `PLAYER`) |
| ✨🪽 MASK_NAV_SOLID | Used by bot code; things the bots consider navigation-blocking (`SOLID`, `PLAYERCLIP`, `WINDOW`) |
| ✨🪽 MASK_LADDER_NAV_SOLID | Used by bot code; things the bots consider navigation-blocking for ladders (`SOLID`, `WINDOW`) |
| ✨🪽 MASK_WALK_NAV_SOLID | Used by bot code; things the bots consider navigation-walkable (`SOLID`, `PLAYERCLIP`, `WINDOW`, `MONSTERCLIP`) |
| ✨🪽 MASK_PROJECTILE | Used by projectiles for what contents they can collide with (`MASK_SHOT` and `PROJECTILECLIP`) |

## `surfflags_t`

Surface flags are used to determine properties for materials and textures applied to brushes. These flags controls lighting, physics, rendering effects and texture behaviour. They are defined as bitflags meaning a surface can contain multiple flags. In 🍦 these flags are defined as constant values while in ✨🪽 it is an enum type.

🍦✨ The enum constants are globals prefixed with the string `SURF_` to prevent name clashes.

| Member | Description |
| --- | --- |
|LIGHT | Light surface; indicates that this surface will emit light. |
| SLICK | Slippery surface; its only use in the base game is to have players be frictionless on them. |
| SKY | Sky surface; marks the surface as sky. |
| WARP | Warped surface; makes the texture distort dynamically, used for water, lava and other fluid surfaces. |
| TRANS33 | 33% transparent surface. |
| TRANS66 | 66% transparent surface. |
| FLOWING | Flowing surface; Causes the texture to scroll in a specific direction (determined by the brush's angle). Used for water current, lava flows and conveyor belts. |  
| NODRAW | Flag used when the surface should never be drawn. Used for invisible collision brushes. |
| ✨🪽 ALPHATEST | Enable alpha testing (on/off transparency) for the surface. |
| ✨🪽 N64_UV | Halves the texture size (Specific to N64). |
| ✨🪽 N64_SCROLL_X | Causes textures to scroll in the X axis (Specific to N64).  |
| ✨🪽 N64_SCROLL_Y | Causes textures to scroll in the Y axis (Specific to N64). |
| ✨🪽 N64_SCROLL_FLIP | Flips the scrolling axis (Specific to N64). |

## `cplane_t`

Collision plane which is fundamental to collision detection. Defines boundaries for brushes, clipping volumes and BSP nodes.

| Member | Description |
| --- | --- |
| normal | 3D vector representing the plane's normal. |
| dist | Distance from the origin to the plane on the normal vector. This defines the plane's position in the world. |
| type | Categorizes the plane based on which axis it aligns with. |
| signbits | Stores a bitmask for fast dot product to help define which side of the plane a point lies on. |
| 🍦✨ pad | Padding to ensure struct alignment, for performance reasons. Has no meaning. |

## `csurface_t`

Surface definition, stores surface properties used for rendering, collision detection and physics interaction.

| Member | Description |
| --- | --- |
| name | Name for the texture assigned to the surface. |
| flags | [Surface flags; see surfflags_t](Types#surfflags_t) |
| value | Material specific value. Only used for light emission in vanilla. |
| ✨🪽 id | Used by the client to index footstep sounds, offset by 1 if non-`null`, otherwise it's zero. |
| ✨🪽 material | Material name for this texinfo; this is defined by the `.mat` file attached to a particular texture. |

## `trace_t`

This struct is returned by value by the trace functions ([gi.trace](Server-Imports#trace)), and contains information about a box or point that is sweeped through the BSP and through entities.

| Member | Description |
| --- | --- |
| allsolid | If the trace is completely inside of a brush with the input content `mask`, this will be true.  |
| startsolid | If the trace started inside of a brush with the input content `mask` but was able to escape, this will be true (usually for partially-occluded collisions). |
| fraction | How far the trace moved between `start` and `end` of the trace before hitting something, as a fraction between `0.0` and `1.0`. If the value is `1.0`, nothing was hit (`endpos` will equal `end`). If the value is `0.0`, something was hit immediately (`endpos` will equal `start`). Any other value is interpolated between the two positions by this value. |
| endpos | The final position where the trace stopped. |
| plane | The surface normal at impact. |
| surface | The surface that was hit. ✨🪽 This value will never be `null`. |
| contents | The content flags of the brush that was hit; [Content flags; see contents_t](Types#contents_t) |
| ent | The entity that was hit (if there was one). This is very rarely (if ever) `null`, and will instead point to the `world` if nothing was hit. |
| ✨🪽 plane2 | When a trace impacts multiple places at destination the collision system will now require both of them; this is the 'second best' plane. |
| ✨🪽 surface2 | The second best surface hit. Will be `null` if a second surface was not hit. |

## `pmtype_t`

Defines different types of player movement states for the client-side movement prediction.

| Member | Description |
| --- | --- |
| NORMAL | Standard player movement. |
| ✨🪽 GRAPPLE | Used for grappling hook, not affected by gravity and is instead pulled towards `velocity`. |
| ✨🪽 NOCLIP | No clipping against anything; flying physics. This is what `SPECTATOR` represents in 🍦. |
| SPECTATOR | 🍦 No clipping mode, allows free movement with no gravity or collision. ✨🪽 Now cannot enter walls but can go through brush entities. |
| DEAD | Player is dead; prevents acceleration and turning but allows minor physics effects such as gravity. |
| GIB | Player has exploded into gibs using a smaller bounding box and no movement. |
| FREEZE | Player is completely frozen preventing all input and movement. |

## `pmflags_t`

Player movement flags used for prediction and physics. Mainly used to keep track of the player's movement state. They are defined as bitflags meaning a pmove_state_t can contain multiple flags. In 🍦 these flags are defined as constant values while in ✨🪽 it is an enum type.

| Member | Description |
| --- | --- |
| ✨🪽 NONE | Representation for no flags; the same as zero. |
| DUCKED | Player is crouching. |
| JUMP_HELD | Jump button is being held. |
| ON_GROUND | Player is on solid ground. |
| TIME_WATERJUMP | Water jump state; player is jumping out of water. |
| TIME_LAND | Small delay after landing when the player can jump again. |
| TIME_TELEPORT | Stops movement briefly after teleporting. |
| 🍦&nbsp;NO_PREDICTION<br>✨&nbsp;🪽&nbsp;NO_POSITIONAL_PREDICTION | Disables movement prediction. (Used for grappling hook). ✨🪽 Only disables prediction on origin, allowing angles to be predicted.|
| ✨🪽 ON_LADDER | Player is on a ladder. |
| ✨🪽 NO_ANGULAR_PREDICTION | Angular equivalent of `NO_POSITIONAL_PREDICTION`; disables angular prediction. |
| ✨🪽 IGNORE_PLAYER_COLLISION | Don't predict collision with other players. |
| ✨🪽 TIME_TRICK | If set then `pm_time` is the time remaining to start a 'trick jump', which is a canonized version of a bug in the vanilla movement code that allowed for quick multi-jumps. |

## `pmove_state_t`

Player's movement state.

| Member | Description |
| --- | --- |
| pm_type | [Player movement type; see pmtype_t](Types#pmtype_t) |
| origin | Player position. (🍦 these are compressed as shorts; divide by 8 to decode, multiply by 8 to decode) |
| velocity | Player velocity. (🍦 these are compressed as shorts; divide by 8 to decode, multiply by 8 to decode) |
| pm_flags | [Player movement flags; see pmflags_t](Types#pmflags_t) |
| pm_time | A time value, used for certain movement flags that affect movement over a short period of time. In 🍦 these are 8 milliseconds per 1 value (`/ 8` and `* 8` to encode and decode, respectively). In ✨🪽 these are just in milliseconds. |
| gravity | Current gravity value applied to the player. |
| delta_angles | Baseline angles. These describe the initial angle of the player. Since the server isn't in charge of the player's actual angles (the client is authoritative for them), this is the method of changing where the "rest position" is for angles, such as from spawning or teleporting. (🍦 these are compressed as shorts; use ANGLE2SHORT / SHORT2ANGLE to decode) |
| ✨🪽 viewheight | New field describing the viewheight (offset from the origin to the eye position); used for crouch prediction. |

## `button_t`

Button bits that is used to represent button states for a client. They are defined as bitflags meaning one `button_t` can represent multiple flags. In 🍦 these flags are defined as constant values while in ✨🪽 it is an enum type.

| Member | Description |
| --- | --- |
| ✨🪽 NONE | Representation for no flags; save as zero. |
| ATTACK |The fire/attack button is pressed. |
| USE | The use/interaction button is pressed. |
| ✨🪽 HOLSTER | Corresponds to new `+holster` command. |
| ✨🪽 JUMP | The jump button is pressed; replaces the `usercmd_t::upmove`.  |
| ✨🪽 CROUCH | The crouch button is pressed; replaces the `usercmd_t::upmove`. |
| ANY | Any button is pressed; used for general input detection. |

## `usercmd_t`

Usercommand that represents a player's input comands that is sent from the client to the server each frame.

| Member | Description |
| --- | --- |
| msec | The frame time in milliseconds since the last command. |
| buttons | [Button bitmask; see button_t](Types#button_t) |
| angles | View angles (yaw, pitch, roll). |
| forwardmove | Player movement along the forward axis; positive value means forward, negative value means backwards. |
| sidemove | Player movement along the left/right axis; positive value means right, negative value means left. |
| 🍦 upmove | Player movement along the vertical axis; positive means up or jumping, negative means down or crouching. |
| 🍦 impulse | Special action trigger. |
| 🍦 lightlevel | Light level at the players position; used for AI behaviour. |
| ✨🪽 server_frame | Tells the server which server frame that the input was depressed on; used for integrity checks and anti-lag hitscan. |

