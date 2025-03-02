Entities are the main component of "things" in Quake II. They represent anything that can be seen, heard, or interacted with.

## Map entities

After the game is initialized and the server loads the BSP map file, the game is sent the **entity string**. This string is a simple structured array of key/value pairs. The server does not require that this string be in the `.map` entity format, but for interoperability with tools it's recommended that you always stick with this format.

The format is simple, and follows this basic grammar:

```as
{
    "key1" "value1"
    "key2" "value2"
}
{
    "key1" "value1"
    "key2" "value2"
}
```

This string is passed to the [[game import "SpawnEntities"|server-imports]], and the game is then responsible for actually creating the entities from these key/value pairs; this is done through `ED_ParseEdict`, which then parses the individual key/value pairs via `ED_ParseField` and places them into the correct memory location.

Once parsed, the entity is then passed off to `ED_CallSpawn` which attempts to initialize the entity based on its `classname` field:
* if an item exists in the `itemlist` with the classname, `SpawnItem` is called to handle the spawning.
* 🍦✨ the game will then check the `spawns` table to see if a matching classname exists, and if so, it calls that function.
* 🪽 the game checks if a global function with the name `SP_{classname}` exists, and if so, it calls that function.

Entities that don't end up spawning (either because they were inhibited from spawning or because they didn't have a spawn function) will get freed, and that slot will be re-used by another entity.

Inside of the actual spawn function, follow the rest of the next section (except the first paragraph, because the entity slot is already allocated for you). If you don't actually need the entity, be sure to free it via G_FreeEdict, otherwise it will continue to consume an entity slot!

## Game entities

Entities can be created programmatically via `G_Spawn`. This function finds a free entity slot and returns the entity pointer that can then be modified by the game. If you don't change any members, an entity will simply exist in the world and do nothing, using up a slot. You must be careful that your entity correctly gets freed when it is no longer required.

If you intend your entity to be "sensed" by either clients or the server (either by being visible, audible, or touchable), you **must** [[link|Entity-Lifecycle#linking]] the entity into the world. If you no longer need the entity to be accessible, you must unlink the entity. If you don't need the entity to be ever synced to the client, you should set the [[svflags_t\|Types#svflags_t]] `NOCLIENT`, which speeds up processing on the server side as it knows it doesn't need to try checking for [[PVS/PHS|PVS]] reachability.

You can set various members of the entities' [entity_state_t](../Types#entity_state_t) to make the entity visible or audible. For a minimally-visible entity, you'll need an `origin` and a `modelindex`, `effects` or `sound` at the very least, and [[link|Entity-Lifecycle#linking]] the entity.

To make the entity touchable by other objects, you must set [the 'solid' member](../Types#edict_t) to either `BBOX`, `TRIGGER` or `BSP`. When using `BSP`, or when using `TRIGGER` with a BSP model (eg a `trigger_multiple`), you must also set the `modelindex` field to a valid BSP entity **which should only be done [[through the game import 'setmodel'|server-imports]]**. In all other cases, you should set the `mins` and `maxs` fields to the two corner points on an axis-aligned bounding box. You may also have to set additional [[svflags_t\|Types#svflags_t]] to change how this entity should be treated when it comes to traces against them. Finally, [[link|Entity-Lifecycle#linking]] the entity.

To make the entity "do" things, see [[Thinking|Entity-Lifecycle#thinking]]. To allow an entity to interact with the world, see [[Physics|Entity-Lifecycle#physics]].

## Linking

For an entity to be visible to either the server or client, it must be linked. This is done [[through the game import 'linkentity'|server-imports]]. This function will link the entity into the correct world area with other entities, and sets up various members to their correct values (such as `absmin`, `absmax` and `size`) if they are solid. It also sets up their visibility state, which is used by the server to tell if an entity should be sent to a given client.

At any time, you can unlink entities from the world [[through the game import 'unlinkentity'|server-imports]]. This will make the entity 'invisible' to [['BoxEdicts' and 'trace'|server-imports]] and make them not appear to any clients. This does not free the entity, however, and it may still exist and perform tasks in the game code. The base game does not make use of this feature, though; the only thing the base game uses unlinking for is to hide body queue bodies, remove players, and temporarily hide an entity during teleportation so that you don't telefrag yourself.

## Physics

Quake II comes with some very basic physics routines, which are set in the form of a member field on an entity. The game calls this a `movetype`, and setting this on an entity will change the way it behaves in the world.

When a tangible movetype is set, the entities' `clipmask` is used to tell the physics function which contents the entity should collide with.

| `movetype_t` enum name | description |
| --- | --- |
| `NONE` | The entity can think, and does nothing else. |
| `NOCLIP` | The entity can think; if it does **not** think on a frame, it will then increase its `angles` and `origin` by `avelocity` (in degrees per second) and `velocity` (in units per second), respectively, and the entity is then re-linked. |
| `PUSH` and `STOP` | Brush models are a bit more complex than other entities. First, only team masters (the first entity in a given `team` chain) will do anything at all when running physics. Teams of one (just themselves) are valid, too. Each piece of the team will attempt to move; to do this, they calculate their next position + angle, grab all entities inside of their bounding box after they'd move into that position, and see if any entities are still touching that entity. If the movetype is `STOP`, the move will fail and simply pause if a non-rider would need to be pushed, otherwise it will attempt to push the entity by the same origin/angles and try to find a valid position for them to stand. If they're still impacting the mover, the move is failed and the entity is allowed to react to this via the `blocked` entity callback, otherwise it continues to do this for every part. If the entire move completes, the think function is run for each part. |
| `WALK` | A special movetype for players that does nothing. |
| `STEP` | Monsters and barrels exclusively use this movetype. It acts as `TOSS` when in the air, otherwise it can 'move' if given a velocity by sliding against the ground, with friction applied so that it eventually comes to a stop. Monsters are also given rotational friction on their angular velocity, although this feature is never used in vanilla. Finally, the think function is called. |
| `FLY`, `TOSS`, `FLYMISSILE`, `BOUNCE`, `WALLBOUNCE` | All five of these movetypes use the same basic function, but with minor differences. First, their think function is called. If they're still alive (and not a non-master of a team), some basic movement code will be run. If the entity has any upwards velocity (or their `groundentity` has gone away), their `groundentity` is unset. If they are still on ground and not affected by negative gravity, nothing more will happen. Velocity is then clamped by `sv_maxvelocity` (🍦 this is applied per-axis, leading to entities that hit the max velocity to clamp oddly), and for `TOSS` & `BOUNCE` will have gravity applied. Their angular velocity is applied, and then the entity performs a trace (✨🪽 `TOSS` will perform multiple traces, for less a less stuttery appearance upon collision) for movement until it either comes to a stop or has fully completed a move. `BOUNCE` and `WALLBOUNCE` type entities will rebound against their surfaces, whereas the others will simply slide against surfaces they contact with. |

If an entity ends up detecting a collision against something else, both entities will call their `touch` functions so they can each react to the impact.

## Thinking

Every entity (except the player) gets a chance to 'think'. Thinking is similar to what other engines might call ticking, although the entity controls when it wishes to receive a think by setting the `nextthink` member to the next time it wishes to have its `think` function called.