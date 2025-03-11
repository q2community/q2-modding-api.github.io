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
| ✨🪽 viewheight | Player's viewheight (offset from the origin to the eye position); used for crouch prediction. |

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
| 🍦 impulse | A vestigial from Quake, referring to the `impulse` passed from the last impulse cmd. Not used by the game; use `ClientCommand` commands instead. |
| 🍦 lightlevel | Light level at the players position; used for AI behaviour. |
| ✨🪽 server_frame | Tells the server which server frame that the input was depressed on; used for integrity checks and anti-lag hitscan. |

## `water_level_t`

New waterlevel type (added in ✨🪽) that is used to give names to the different water levels that in 🍦 had no names. In 🍦 these are hardcoded as magic numbers rather than referred to by name.

| Value | Member | Description |
| --- | --- | --- |
| 0 | NONE | Not touching water. |
| 1 | FEET | Water is at feet level. |
| 2 | WAIST | Water is at waist level.  |
| 3 | UNDER | Entity is completely under water. |

## `refdef_flags_t`

Refresh definition flags that affect the entire scene. They are defined as bitflags meaning one `refdef_flags_t` can represent multiple flags. In 🍦 these flags are defined as constant values while in ✨🪽 it is an enum type.

| Member | Description |
| --- | --- |
| ✨🪽 NONE | Representation for no flags; save as zero. |
| UNDERWATER | When the player is underwater this flag warps the screen in order to create a distortion effect. |
| NOWORLDMODEL | Prevents rendering world geometry; used in menus and some cutscenes. Not useful to the game code. |
| IRGOGGLES | Infrared goggles effect. |
| UVGOGGLES | Ultraviolet goggles effect. Unused. |
| ✨🪽 NO_WEAPON_LERP | Used to temporarily disable interpolation on weapons. |

## `pmove_t`

Player movement state, used for player movement and collision detection. This type contains the player's current movement state, input commands and movement results.

| Member | Description |
| --- | --- |
| s | Input & output variable. [Player movement state; see pmove_state_t](Types#pmove_state_t) |
| cmd | Input variable. [User command; see usercmd_t](Types#usercmd_t) |
| snapinitial | Input variable. Set to true if this is an 'initial position' (when state has been reset, essentially, like on respawn). |
| 🍦 numtouch | Output variable. Number of entities that the player touched. |
| 🍦 touchents | Output variable. Array of entities that the player collided with. |
| ✨ touch | Output variable. [Touch list; see touch_list_t](Types#touch_list_t) |
| viewangles | Player's view angles. |
| 🍦 viewheight | Output variable. The viewheight (offset from the origin to the eye position); used for crouching. (✨🪽 moved this to `pmove_state_t`) |
| mins, maxs | Output variables. The entity's size in the world. This is an axis-aligned bounding box. |
| groundentity | Output variable. The entity that the player is standing on. |
| ✨🪽 groundplane | Output variable. [Collision plane; see cplane_t](Types#cplane_t) |
| watertype | Type of liquid the player is standing on?. |
| waterlevel | [Water level; see water_level_t](Types#water_level_t) |
| trace() | Collision detection function callback. |
| pointcontents() | Function callback to check the material at a point. |
| ✨🪽 player | Input variable. An opaque handle to an [edict; see edict_t](Types#edict_t) that refers to the current player. This is passed back to the `trace` function. |
| ✨🪽 clip() | World clipping function callback. |
| ✨🪽 viewoffset | Input variable. Player's view offset. |
| ✨🪽 screen_blend | Output variable containing the full-screen blend to apply to the view. |
| ✨🪽 rdflags | Output variable. [Refresh definition flags; see refdef_flags_t](Types#refdef_flags_t) |
| ✨🪽 jump_sound | Output variable to tell the game to play a jumping sound. |
| ✨🪽 step_clip | Output variable; if we stepped up onto a step via a jump, this is set to true. Helps client prediction avoid a harsh snap. |
| ✨🪽 impact_delta | Output variable; impact delta used for falling damage. |

> [!NOTE]
> 🪽 Due to limitations, the `touch`/`touchents`/`numtouch` members are instead member functions of `pmove_t`: `touch_length`, `touch_push_back`, `touch_get` and `touch_clear`.

## `effects_t`

Visual effects that are applied to entities. They are defined as bitflags meaning one `effects_t` can represent multiple flags. In 🍦 these flags are defined as constant values while in ✨🪽 it is an enum type.

| Member | Description |
| --- | --- |
| ✨🪽 NONE | Representation for no flags; same as zero. |
| ROTATE | Rotation effect for power-ups and items. |
| GIB | Leaves a blood trail. |
| ✨🪽 BOB | Weapon bobbing effect. |
| BLASTER | Blaster shot glow and trail. |
| ROCKET | ROcket projective glow and trail. |
| GRENADE | Grenade projectile. |
| HYPERBLASTER | Hyperblaster glow and trail. |
| BFG | BFG green glow and energy effect. |
| COLOR_SHELL | Colored glow effect. |
| POWERSCREEN | Energy shield effect. ✨🪽 This effect uses a different model and is scaled to the monster's size. |
| ANIM01 | Automatic animation cycling effect. Cycles between frames 0 and 1 at 2Hz. |
| ANIM23 | Cycles between frames 2 and 3 at 2Hz. |
| ANIM_ALL | Cycles through all frames at 2Hz. |
| ANIM_ALLFAST | Cycles through all frames at 10hz. |
| FLIES | Fly particle effect. |
| QUAD | Quad damage effect. |
| PENT | Pentagram effect used for invulnerability. |
| TELEPORTER | Teleportation burst effect. |
| FLAG1 | CTF red flag effect. |
| FLAG2 | CTF blue flag effect. |
| IONRIPPER | Ion Ripper projectile trail. |
| GREENGIB | Green colored gibs. |
| BLUEHYPERBLASTER | Blue colored hyperblaster glow and trail. |
| SPINNINGLIGHTS | Rotation lights. |
| PLASMA | Plasma based weapon glow and trail. |
| TRAP | Visual effect for traps. |
| TRACKER | Used for homing projectiles. |
| DOUBLE | Double damage effect. |
| SPHERETRANS | Partially transparent effect. |
| TAGTRAIL | Special projectile trail. |
| HALF_DAMAGE | Half damage effect. |
| TRACKERTRAIL | Homing projectile damage effect. |
| ✨🪽 DUALFIRE | Similar to `QUAD` but for dualfire damage. |
| ✨🪽 HOLOGRAM | Used for the N64 hologram effect. |
| ✨🪽 FLASHLIGHT | Marks the entity to have a Flashlight like effect. |
| ✨🪽 BARREL_EXPLODING | Used before an explobox explodes, emits steam particles from the barrel. |
| ✨🪽 TELEPORTER2 | Used for N64 teleporter. |
| ✨🪽 GRENADE_LIGHT | Small light around monster grenades. |

> [!ATTENTION]
> TODO: Document special values/masks

## `renderfx_t`

Special render effects for entities. They are defined as bitflags meaning one `renderfx_t` can represent multiple flags. In 🍦 these flags are defined as constant values while in ✨🪽 it is an enum type.

| Member | Description |
| --- | --- |
| ✨🪽 NONE | Representation for no flags; save as zero. |
| MINLIGHT | Ensures entity always has some lighting applied to it. |
| VIEWERMODEL | Prevents entity from being seen from the player's eyes; can still be seen from reflections. |
| WEAPONMODEL | The opposite of `VIEWERMODEL` this entity is only drawn from the player's view. |
| FULLBRIGHT | Makes the entity always fully lit; ignores ambient lighting. |
| DEPTHHACK | Adjust Z-buffer depth for viewmodels preventing them to clip through walls. |
| TRANSLUCENT | Makes the entity semi-transparent. |
| 🍦FRAMELERP<br>✨&nbsp;🪽&nbsp;NO_ORIGIN_LERP | Disables origin interpolation. |
| BEAM | Marks the entity as a beam effect; used for lasers and lighting. ✨🪽 Can now create custom segmented beams by setting a non-one modelindex on beams. |
| CUSTOMSKIN | Use custom skin texture from the `image_precache`. |
| GLOW | Applies a pulsing glow effect. |
| SHELL_RED | Adds a red energy shell effect. |
| SHELL_GREEN | Adds a green energy shell effect. |
| SHELL_BLUE | Adds a blue energy shell effect. |
| ✨🪽NOSHADOW | Marks entity to not have a shadow. |
| ✨🪽CASTSHADOW | Used for dynamic lights, to tell it it is a shadow-caster. |
| IR_VISIBLE | Entity is visible through infrared goggles. |
| SHELL_DOUBLE | Adds both red and blue shell effects. |
| SHELL_HALF_DAM | Indicates half-damage protection. |
| USE_DISGUISE | Marks the entity as using a disguise. |
| ✨🪽 SHELL_LITE_GREEN | Equivalent shell color for `DUALFIRE`. |
| ✨🪽 CUSTOM_LIGHT | Creates custom dynamic light at the position of the object. |
| ✨🪽 FLARE | Marks entity to be rendered as a flare instead of the usual entity rendering. |
| ✨🪽 OLD_FRAME_LERP | Signals to the client that `s.old_frame` should be used for the next frame and respected by the client. |
| ✨🪽 DOT_SHADOW | Draw a blob shadow underneath the entity. |
| ✨🪽 LOW_PRIORITY | Marks the entity as low priority. If the renderer runs out of entity slots, this entity can be replaced. |
| ✨🪽 NO_LOD | Only use high quality models if available (do not fall back to MD2s for LOD). |
| ✨🪽 NO_STEREO | Stereo sound is disabled on the entity. |
| ✨🪽 STAIR_STEP | Marks the entity as they stepped on stairs; causes their Z change from previous frame to interpolate at 10hz, similar to how the player view handles stairs. |
| ✨🪽 FLARE_LOCK_ANGLE | Used in flare rendering to cause the flare to not rotate towards the viewer. |

> [!ATTENTION]
> TODO: Document special values/masks

## `player_muzzle_t`

Player muzzle effects. In 🍦 these flags are defined as constant values while in ✨🪽 it is an enum type.

| Member | Description |
| --- | --- |
| ✨🪽 NONE | Representation for no flags; save as zero. |
| BLASTER | Blaster shot. |
| MACHINEGUN | Machine gun muzzle flash. |
| SHOTGUN | Shotgun muzzle flash. |
| CHAINGUN1 | Chaingun fire; first stage with slow fire rate. |
| CHAINGUN2 | Chainfun fire; second stage with medium fire rate. |
| CHAINGUN3 | Chaingun fire; third stage with full fire rate. |
| RAILGUN | Railgun shot effect. |
| ROCKET | Rocket launcher muzzle flash. |
| GRENADE | Grenade launcher muzzle flash. |
| LOGIN | Effect when a player spawns into the game. |
| LOGOUT | Effect when a player leaves the game. |
| RESPAWN | Effect when player respawns after dying. |
| BFG | BFG muzzle flash. |
| SSHOTGUN | Super shotgun muzzle flash. |
| HYPERBLASTER | Hyperblaster muzzle flash. |
| ITEMRESPAWN | Item respawn effect. |
| IONRIPPER | Ion ripper muzzle flash. |
| BLUEHYPERBLASTER | Alternative hyperblaster muzzle flash. |
| PHALANX | Phalanx cannon muzzle flash. |
| ✨🪽 BFG2 | Secondary muzzle flash for BFG (when the fire frame occurs). |
| ✨🪽 PHALANX2 | Secondary muzzle flash for the Phalanx (right barrel). |
| SILENCED | Flag to suppress muzzle flash for silenced weapons. |
| ETF_RIFLE | ETF rifle muzzle flash. |
| 🍦UNUSED<br>✨🪽 PROX | Prox launcher muzzle flash. |
| 🍦SHOTGUN2<br>✨🪽ETF_RIFLE2 | Second barrel of the ETF rifle muzzle flash. Unused in vanilla. |
| HEATBEAM | Heat beam lazer muzzle flash. |
| BLASTER2 | Unused blaster muzzle flash. |
| TRACKER | Disruptor projectile muzzle flash. |
| NUKE1 | Nuclear weapon; stage 1 explosion flash. |
| NUKE2 | Nuclear weapon; stage 2 explosion flash. |
| NUKE4 | Nuclear weapon; stage 4 explosion flash. |
| NUKE8 | Nuclear weapon; stage 8 explosion flash. |

## `monster_muzzleflash_id_t`

Monster muzzle effects. In 🍦 these flags are defined as constant values while in ✨🪽 it is an enum type.

| Member | Description |
| --- | --- |
| ✨🪽 UNUSED_0 | Not used; save as zero. |
| MZ2_TANK_BLASTER_1 - 3 | Tank blaster muzzle position. |
| MZ2_TANK_MACHINEGUN_1 - 19 | Tank machinegun muzzle position. |
| MZ2_TANK_ROCKET_1 - 3 | Tank rocket muzzle position. |
| MZ2_INFANTRY_MACHINEGUN_1 - 13 | Infantry machinegun muzzle flash position. |
| MZ2_SOLDIER_BLASTER_1 - 8 | Soldier blaster muzzle position. |
| MZ2_SOLDIER_SHOTGUN_1 - 8 | Soldier shotgun muzzle position. |
| MZ2_SOLDIER_MACHINEGUN_1 - 8 | Soldier machinegun muzzle position. |
| MZ2_GUNNER_MACHINEGUN_1 - 8 | Gunner machinegun muzzle position. |
| MZ2_GUNNER_GRENADE_1 - 4 | Gunner grenade launcher muzzle position. |
| MZ2_CHICK_ROCKET_1 | Chick rocket launcher muzzle flash. |
| MZ2_FLYER_BLASTER_1 - 2 | Flyer blaster muzzle flash position. |
| MZ2_MEDIC_BLASTER_1 - 2 | Medic blaster muzzle flash position. |
| MZ2_GLADIATOR_RAILGUN_1 | Gladiator railgun muzzle flash. |
| MZ2_HOVER_BLASTER_1 | Hover blaster muzzle flash. |
| MZ2_SUPERTANK_MACHINEGUN_1 - 6 | Super tank machinegun muzzle flash position. |
| MZ2_SUPERTANK_ROCKET_1 - 3 | Super tank rocket launcher muzzle flash position. |
| MZ2_BOSS2_MACHINEGUN_L1 - L5 | Boss 2 left-side machinegun muzzle flash position. |
| MZ2_BOSS2_ROCKET_1 - 4 | Boss 2 rocket launcher muzzle flash positions. |
| MZ2_BOSS2_MACHINEGUN_R1 - R5 | Boss 2 right-side machinegun muzzle flash position. |
| MZ2_MAKRON_BFG | Makrok BGF muzzle flash. |
| MZ2_MAKRON_BLASTER_1 - 17 | Makron blaster muzzle flash position. |
| MZ2_MAKRON_RAILGUN_1 | Makron railgun muzzle flash. |
| MZ2_JORG_MACHINEGUN_L1 - L6 | Jorg left-side machinegun muzzle flash position. |
| MZ2_JORG_MACHINEGUN_R1 - R6 | Jorg right-side machinegun muzzle flash position. |
| MZ2_JORG_BFG_1 | Jorg BFG muzzle flash. |
| MZ2_CARRIER_MACHINEGUN_L1 - L2 | Carrier left-side machinegun muzzle flash position. |
| MZ2_CARRIER_MACHINEGUN_R1 - R2 | Carrier right-side machinegun muzzle flash position. |
| MZ2_CARRIER_GRENADE | Carrier grenade launcher muzzle flash. |
| MZ2_CARRIER_RAILGUN | Carrier railgun muzzle flash. |
| MZ2_CARRIER_ROCKET_1 - 4 | Carrier rocket launcher muzzle flash position. |
| MZ2_WIDOW_DISRUPTOR | Widow disruptor muzzle flash. |
| MZ2_WIDOW_BLASTER | Widow blaster muzzle flash. |
| MZ2_WIDOW_RAIL | Widow railgun muzzle flash. |
| MZ2_WIDOW_PLASMABEAM | Widow plasma beam muzzle flash. |
| MZ2_WIDOW_RAIL_LEFT | Widow left-side railgun muzzle flash. |
| MZ2_WIDOW_RAIL_RIGHT | Widow right-side railgun muzzle flash. |
| MZ2_WIDOW_BLASTER_SWEEP1 - 9 | Widow blaster sweeping muzzle flash position. |
| MZ2_WIDOW_BLASTER_100 - 0 - 70L | Widow's various blaster positions. |
| MZ2_WIDOW2_BEAMER_1 - 5 | Widow2 beamer muzzle flash positions. |
| MZ2_WIDOW2_BEAM_SWEEP_1 - 11 | Widow2 beamer sweeping muzzle flash positions. |
| ✨🪽 MZ2_SOLDIER_RIPPER_1 - 8 | Soldier ripper muzzle flash positions. |
| ✨🪽 MZ2_SOLDIER_HYPERGUN_1 - 8 | Soldier hyperblaster muzzle flash positions. |
| ✨🪽 MZ2_GUARDIAN_BLASTER | PSX Guardian blaster muzzle flash. |
| ✨🪽 MZ2_ARACHNID_RAIL1 - _UP2 | PSX Arachnid rail muzzle flashes.|
| ✨🪽 MZ2_INFANTRY_MACHINEGUN_14 - 21 | Infantry run-attack muzzle flash. |
| ✨🪽 MZ2_GUNCMDR_CHAINGUN_1 - 2 | Gunner commander chaingun muzzle flash position. |
| ✨🪽 MZ2_GUNCMDR_GRENADE_MORTAR_1 - 3 | Gunner commander grenade mortar muzzle flash positions. |
| ✨🪽 MZ2_GUNCMDR_GRENADE_FRONT_1 - 3 | Gunner commander grenade launcher front muzzle flash positions. |
| ✨🪽 MZ2_GUNCMDR_GRENADE_CROUCH_1 - 3 | Gunner commander grenade launcehr crouching muzzle flash position. |
| ✨🪽 MZ2_SOLDIER_BLASTER_9 | Soldier blaster prone muzzle flash position. |
| ✨🪽 MZ2_SOLDIER_SHOTGUN_9 | Soldier shotgun prone muzzle flash position. |
| ✨🪽 MZ2_SOLDIER_MACHINEGUN_9 | Soldier machinegun prone muzzle flash position. |
| ✨🪽 MZ2_SOLDIER_RIPPER_9 | Soldier ripper prone muzzle flash position. |
| ✨🪽 MZ2_SOLDIER_HYPERGUN_9 | Soldier hypergun prone muzzle flash position. |
| ✨🪽 MZ2_GUNNER_GRENADE2_1 - 4 | Alternative firing animation for gunner grenade launcher. |
| ✨🪽 MZ2_INFANTRY_MACHINEGUN_22 | Alternative firing animation for infantry machinegun. |
| ✨🪽 MZ2_SUPERTANK_GRENADE_1 - 2 | Supertank grenade launcher muzzle flash position. |
| ✨🪽 MZ2_HOVER_BLASTER_2 | Hover blaster other side muzzle flash. |
| ✨🪽 MZ2_DAEDALUS_BLASTER_2 | Daedalus other side blaster muzzle flash. |
| ✨🪽 MZ2_MEDIC_HYPERBLASTER1_1 - 12 | Medic hyperblaster sweed muzzle flash positions. |
| ✨🪽 MZ2_MEDIC_HYPERBLASTER2_1 - 12 | Medic commander hyperblaster sweed muzzle flash positions. |
| ✨🪽 MZ2_LAST | Only used internally for compile-time checks. |

## `temp_event_t`

Temporary entity events, enum that defines various temporary, short-lived entity events; usually visual and gameplay feedback such as gunshots, explosions or special effects.

| Member | Description |
| --- | --- |
| GUNSHOT | Visual effect for gunshots hitting a surface. |
| BLOOD | Blood effect when player or a monster is hit. |
| BLASTER | Projectile effect from a blaster weapon. |
| RAILTRAIL | Visual effect for railgun trail. |
| SHOTGUN | Visual effect for shotgun pellet. |
| EXPLOSION1 | Standard explosion effect type 1. |
| EXPLOSION2 | Standard explosion effect type 2. |
| ROCKET_EXPLOSION | Visual effect from rocket explosion. |
| GRENADE_EXPLOSION | Visual effect for grenade explosion. |
| SPARKS | Generic spart visual effect. |
| SPLASH | Water splash visual effect. |
| BUBBLETRAIL | Bubble trail visual effect for object moving underwater. |
| SCREEN_SPARKS | Electrical sparks on screen or consoles visual effect. |
| SHIELD_SPARKS | Sparks generated when a shield absord damage. |
| BULLET_SPARKS | Sparks generated from bullet hitting metal surfaces. |
| LASER_SPARKS | Sparks generated from laser impact. |
| PARASITE_ATTACK | Visual effect for parasite monster attack. |
| ROCKET_EXPLOSION_WATER | Visual effect for rocket explosion in water. |
| GRENADE_EXPLOSION_WATER | Visual effect for grenade explosion in water. |
| MEDIC_CABLE_ATTACK | Visual effect for medic's attack with cables. |
| BFG_EXPLOSION | Visual effect for BFG explosion. |
| BFG_BIGEXPLOSION | Larger visual effect for BFG explosion. |
| BOSSTPORT | Teleportation effect for boss characters. |
| BFG_LASER | Laser effect from BFG weapon. |
| GRAPPLE_CABLE | Effect from grappling hook cable. |
| WELDING_SPARKS | Sparks from welding. |
| GREENBLOOD | Green blood effect. |
| 🍦BLUEHYPERBLASTER<br>✨&nbsp;🪽&nbsp;BLUEHYPERBLASTER_DUMMY | Blue hyperblaster effect. ✨🪽 left this for compatibility.  |
| PLASMA_EXPLOSION | Visual effect for plasma explosion. |
| TUNNEL_SPARKS | Sparks in tunnels. |
| BLASTER2 | Alternate blaster effect. |
| RAILTRAIL2 | Alternate railgun effect. |
| FLAME | Flame effect. |
| LIGHTNING | Lightning effect. |
| DEBUGTRAIL | Debug trail effect. |
| PLAIN_EXPLOSION | Simple explosion visual effect. |
| FLASHLIGHT | Flashlight beam visual effect. |
| FORCEWALL | Visual effect for force walls. |
| HEATBEAM | Visual effect for heat beam effect from weapon or device. |
| MONSTER_HEATBEAM | Visual effect from heat beam monster attack. |
| STEAM | Visual effect for steam from pipes or vents. |
| BUBBLETRAIL2 | Alternative bubble trail effect. |
| MOREBLOOD | Enhanced blood visual effects. |
| HEATBEAM_SPARKS | Visual effect for heat beam sparks. |
| HEATBEAM_STEAM | Visual effect for heat beam steam. |
| CHAINFIST_SMOKE | Visual effect for smoke from chainfirst weapon. |
| ELECTRIC_SPARKS | Visual effect for electric sparks. |
| TRACKER_EXPLOSION | Visual effect for tracker weapon. |
| TELEPORT_EFFECT | Visual effect for teleportation. |
| DBALL_GOAL | Special effect for ball-based gamemode. |
| WIDOWBEAMOUT | Visual effect for widow beam attack. |
| NUKEBLAST | Visual effect for nuclear blast. |
| WIDOWSPLASH | Visual effect for widow splash attack. |
| EXPLOSION1_BIG | Larger effect for standard explosion. |
| EXPLOSION1_NP | Non-physics based explosion. |
| FLECHETTE | Flechette projectile effect. |
| ✨🪽 BLUEHYPERBLASTER | Replaces the old `BLUEHYPERBLASTER`. |
| ✨🪽 BFG_ZAP | Laser when an entity has been zapped by a BFG explosion. |
| ✨🪽 BERSERK_SLAM | Large blue flash & particles at impact point towards a direction.  |
| ✨🪽 GRAPPLE_CABLE_2 | The grappling hook in Quake II 3.20 used a larger message that didn't allow the cable to render like other player-derived beams. |
| ✨🪽 POWER_SPLASH | Effect sent when a power shield evaporates. |
| ✨🪽 LIGHTNING_BEAM | A lightning bolt that originates from the player, like the heat beam. |
| ✨🪽 EXPLOSION1_NL | Visual effect for explosion that don't include dynamic light. |
| ✨🪽 EXPLOSION2_NL | Alternative effect for explosion that don't include dynamic light. |

## `splash_color_t`

Splash color used to categorize different types of splash effects. In 🍦 these flags are defined as constant values while in ✨🪽 it is an enum type.

| Member | Description |
| --- | --- |
| UNKNOWN | Default or unknown splash type; same as zero. |
| SPARKS | Creates sparks instead of liquid splash. |
| BLUE_WATER | Represents splash effect in blue water. |
| BROWN_WATER | Represents splash in muddy or dirty water. |
| SLIME | Represents splash effect of green or toxic slime. |
| LAVA | Represent splash of lava. |
| BLOOD | Represents splash of blood. |
| ✨🪽 ELECTRIC | Electric sparks that zaps. Used in N64. |

## `soundchan_t`

Sound channel used to assign autio to different channel. In 🍦 these flags are defined as constant values while in ✨🪽 it is an enum type.

| Member | Description |
| --- | --- |
| AUTO | Default channel; does not override any sound. |
| WEAPON | Used for weapon sounds. |
| VOICE | UIsed for player and monster sounds. |
| ITEM | Used for item interaction sounds. |
| BODY | Used for body-related sounds like footsteps, pain and grunts. |
| ✨🪽 AUX | |
| ✨🪽 FOOTSTEP | |
| ✨🪽 AUX3 | |
| NO_PHS_ADD | Bitwise modifier flag; tells the sound to be sent to all clients not just the one in potential hearing set. |
| RELIABLE | Bitwise modifier flag; tells the sound to be sent using a reliable message ensuring it will not be lost. |
| ✨🪽 FORCE_POS | Bitwise modifier flag; Forced the sound position in the packet to be used. |

## Sound Attenuation

Attenuation detemines how sound volume decrease with distance to the sound. These are defined as constant values.

| Member | Description |
| --- | --- |
| ✨🪽 LOOP_NONE | Full volume over entire level; only used for looping sounds. |
| NONE | Sound plays at full volume over the entire level. |
| NORM | Normal auttenuation; sound diminishes over distance. |
| IDLE | Higher attenuation; sound fades more quickly. |
| STATIC | Very rapid attenuation; sound fades quickly when moving away from the source. |

## ✨🪽 `game_style_t`

Enumeration describing different game styles.

| Member | Description |
| --- | --- |
| PVE | Player versus environment mode. |
| FFA | Free for all mode. |
| TDM | Team deathmatch mode. |

## Config Strings

Config strings are used to send game data from the server to all clients. These strings can store information such about map settings, models, sounds and player skins.

| Member | Description |
| --- | --- |
| NAME | Server name. |
| CDTRACK | Background music CD track to play. |
| SKY | Name of the skybox texture used in the map. |
| SKYAXIS | Rotation axis of the skybox. |
| SKYROTATE | Rotation speed of the skybox. |
| STATUSBAR | Status bar layout, used to display HUD elements. |
| AIRACCEL | Controls air acceleration. |
| MAXCLIENTS | Maximum number of players allowed on the server. |
| MODELS | Start index for the model filenames. |
| SOUNDS | Start index for the sound filenames. |
| IMAGES | Start index for the image filenames. |
| LIGHTS | Start index for the light defintions. |
| ✨🪽 SHADOWLIGHTS | Shadow light entries. |
| ITEMS | Start index for the item definitions. |
| PLAYERSKINS | Start index for the player skins. |
| GENERAL | Start index for general configuration strings. |
| ✨🪽 WHEEL_WEAPONS | Weapon entries for the weapon wheel. |
| ✨🪽 WHEEL_AMMO | Weapon ammo types entries for the weapon wheel. |
| ✨🪽 WHEEL_POWERUPS | Powerup entries for the powerup wheel. |
| ✨🪽 CD_LOOP_COUNT | Integer that defines how many times to loop the music before switching to ambient track. |
| ✨🪽 GAME_STYLE | [Game style; see game_style_t](Types#game_style_t). |
