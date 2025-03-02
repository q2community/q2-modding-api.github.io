This page goes through the most common Quake II-specific types you'll come across in Quake II modding, and explains their purpose & how to manipulate them. This will cover the mathematical & engine-shared types first, and only briefly go into detail on specific types that are used in the vanilla Quake II mod experience.

## `vec3_t`

A 3d vector; it might represent a position, a unit vector, a vector * magnitude, or even Euler angles. There are several global functions to manipulate them - most of them have `vec` in their name somewhere (✨🪽 and member functions of vec3_t, which is where most of the globals were moved to).

🍦 Be careful when passing vectors as parameters, as the type is a simple typedef to a C array, and these have weird semantics.

## time

Time in Quake II is represented as a number that is kept track of on the game side. The game [runs a level tick](Level-Lifecycle.md) which increases the current time of the level, and this value persists separately on levels (every new level always starts at a time of zero).

🍦 Times are represented as `float` **seconds** in most cases, and in a few other cases they are stored as `int` **frames**. A frame in Quake II is 100 milliseconds (10hz). The `FRAMETIME` macro contains the number of seconds in a frame (0.1) which can be used to convert to other units.

✨🪽 Times are represented as `int64` and stored in the `gtime_t` type. It contains several functions to create times from different units, as well as converting those times back into different components. (✨ You can also use the `_ms`, `_sec`, etc literal postfixes to easily create constants of units of time.)

## `entity_state_t`

The entity state stores the data that is used for transmission to the client. It basically describes everything the client needs to render a given entity, as well as some other data that the server uses for things like collision.

| Member | Description |
| --- | --- |
| number | The numeric index of this entity. This should never be touched. |
| origin | The entity's position in the world. [Changing this requires re-linking the entity.](../Entity-Lifecycle#linking) |
| angles | The entity's Euler angles. [Changing this may require re-linking the entity if it is a brush model.](../Entity-Lifecycle#linking) |
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
| mins, maxs | The entity's size in the world. This is an axis-aligned bounding box. [Changing this requires re-linking the entity.](../Entity-Lifecycle#linking) |
| absmin, absmax | The entity's absolute box in the world, expanded by 1 on each axis. This is the same as (origin + mins) - { 1, 1, 1} and (origins + maxs) + { 1, 1, 1 } respectively. This is read-only. |
| size | The entity's size (mins + maxs). This is read-only. |
| solid | The entity's solidity type. `TRIGGER` and `BBOX` are both usable by any entity, and simply link the entity into either trigger or solid area links, respectively. The `BSP` value can only be used on brush models (entities whose modelindex points to an inline BSP model, such as `*2`). [Changing this requires re-linking the entity.](../Entity-Lifecycle#linking) |
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
| ✨🪽 PROJECTILECLIP | Blocks movement from entities with [svflags_t::PROJECTILE](Types#svflags_t] server flags. | 
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

## `surfflags_t`

Surface flags are used to determine properties for materials and textures applied to brushes. These flags controls lighting, physics, rendering effects and texture behaviour. They are defined as bitflags meaning a surface can contain multiple flags. In 🍦 these flags are defined as constant values while in ✨🪽 it is an enum type.

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
| value | Material specific value. |
| ✨🪽 id | Used by the client to index footstep sounds. |
| ✨🪽 material | Material ID for this texinfo. |

## `trace_t`

Trace which is used for , hit detection, movement calculations and physics interactions.

| Member | Description |
| --- | --- |
| allsolid | If the trace is completely inside a solid object.  |
| startsolid | If the starting point of trace is insied a solid object. |
| fraction | How far the trace moved before hitting something. |
| endpos | The final position where the trace stopped. |
| plane | The surface normal at impact. |
| surface | The surface that was hit. ✨This value must never be null. |
| contents | The type of material  that was hit; [Content flags; see contents_t](Types#contents_t) |
| ent | The entity that was hit (if there was one). |
| ✨🪽 plane2| When a trace impacts multiple places at destination the collision system will now require both of them. The "second best" plane. |
| ✨🪽 surface2| The second best surface hit. Must be null if second surface was not hit. |
