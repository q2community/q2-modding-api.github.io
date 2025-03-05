# Useful Functions

The base game provides several useful functions for performing tasks that make writing game functions easier. 

# High Level Mechanics

These document the functions used for performing higher level tasks, and provide common examples on how you might use them.

## Finding Entities

You'll often need to query the world to find entities. The game code provides several ways of iterating entities depending on your requirements.

### Fetch Entity By Number

Entities are consistently referred to by a particular number/index. This corresponds to their index in the entity list.

Although less common in the native codebases, you may run into a case where you have an entity number and need to fetch the entity it is associated to.

<!-- tabs:start -->

#### **🍦✨**

```cpp
// `i` here is the number of the entity
// you're wishing to fetch.
edict_t *e = &globals.edicts[i];
```

#### **🪽**

```cpp
// `i` here is the number of the entity
// you're wishing to fetch.
ASEntity @e = entities[i];
```

If you only need the `edict_t` handle, you can use the `G_EdictForNum` global function, which returns the handle of the given entity number:

```cpp
// `i` here is the number of the entity
// you're wishing to fetch.
edict_t @e = G_EdictForNum(i);
```

If you already have the `edict_t` handle and need the `ASEntity` instance, you can either grab it from the `entities` array or `cast` it out of `as_obj` member:

```cpp
// `e` here is the edict_t handle
ASEntity @ent = entities[e.number];

// or...
ASEntity @ent = cast<ASEntity>(e.as_obj);
```

<!-- tabs:end -->

### Iterate all entities

This is the lowest level method of entity iteration, and simply iterates all entities that are currently in the world. This allows you to perform whatever filtering you wish.

There's only two variables involved in this iteration: [`globals.edicts` (or `g_edicts`) and `globals.num_edicts`](Server-Exports#edicts) (🪽 `entities` and `num_edicts`). The former is the currently allocated entity list, and the latter is the current total number of entities that are allocated.

Keep in mind this list may include entities that have been freed; you should always make sure any entity you're trying to do anything with is still [`inuse`](Types#edict_t).

<details>
<summary>Usage Examples</summary>
<!-- tabs:start -->

#### **🍦✨**

```cpp
int i;

for (i = 0; i < globals.num_edicts; i++)
{
    edict_t *e = &globals.edicts[i];

    if (!e->inuse)
        continue;

    // perform task on `e` here
}
```

#### **🪽**

```cpp
for (int i = 0; i < num_edicts; i++)
{
    ASEntity @e = entities[i];

    if (!e.e.inuse)
        continue;

    // perform task on `e` here
}
```

<!-- tabs:end -->
</details>

> [!TIP]
> The start of your iteration can skip a few entries if you don't care about including the world entity (the world is always at position `0`), or players (players are always at position `1` to the value of the `maxclients` cvar). There's also a queue of bodies used by dead players that take up the first 8 entity slots. If you're only wanting to iterate through non-player entities & not the world & not the dead bodies, you can start iteration at `1 + game.maxclients + BODY_QUEUE_SIZE` (🪽 `1 + max_clients + BODY_QUEUE_SIZE`).

### Iterate nearby entities

You've got several options for iterating based on distance. The most customizable option is to use [Iterate all entities](#iterate-all-entities); this will let you control the exact conditions you need.

For convenience, there's a couple built-in functions you can use that handle the most common conditions.

#### findradius

This function is used in radius damage calculation, and is a quick way of iterating nearby entities that are `inuse` and `solid`. It uses a simple distance check between the two positions to calculate the distance.

> [!NOTE]
> Since this is calculated from a single point on every entity, the distance does not take into account the distance from the origin to the edge of the entities' bbox.

> [!TIP]
> This requires iterating every entity, and may be slow; [BoxEdicts](#BoxEdicts) might be a better choice if you need to scan for entities in a box around a position.

Function declaration:

<!-- tabs:start -->

#### **🍦**

```cpp
edict_t *findradius (edict_t *from, vec3_t org, float rad);
```

#### **✨**

```cpp
edict_t *findradius(edict_t *from, const vec3_t &org, float rad);
```

#### **🪽**

```cpp
ASEntity @findradius(ASEntity @from, const vec3_t &in org, float rad);
```

<!-- tabs:end -->

<details>
<summary>Usage example (selecting entities in a spherical radius of 256 units):</summary>
<!-- tabs:start -->

#### **🍦**

```cpp
// `org` is a position in the world that you want to
// get entities near, as vec3_t
edict_t *e = NULL;

while ((e = findradius(e, org, 256)) != NULL)
{
    // perform task on `e`
}
```

#### **✨**

```cpp
// `org` is a position in the world that you want to
// get entities near, as vec3_t
edict_t *e = nullptr;

while ((e = findradius(e, org, 256)) != nullptr)
{
    // perform task on `e`
}
```

#### **🪽**

```cpp
// `org` is a position in the world that you want to
// get entities near, as vec3_t
ASEntity @e = null;

while ((@e = findradius(e, org, 256)) !is null)
{
    // perform task on `e`
}
```

<!-- tabs:end -->
</details>

#### BoxEdicts

The [`BoxEdicts` server import](Server-Imports#boxedicts) can quickly find entities in a box. The server uses world links to speed up selection. It does not require iterating every entity, and instead only iterates world links that the box covers.

> [!NOTE]
> If the count overflows `max_count`, no more entities will be written to the array.

<details>
<summary>Usage example (selecting a 256^3 unit box of solid entities that can take damage):</summary>
<!-- tabs:start -->

#### **🍦**

```cpp
// `pos` is the origin of the search.
static edict_t *results[256];
vec3_t absmin, absmax;

VectorSet(absmin, -128, -128, -128);
VectorSet(absmax, 128, 128, 128);
VectorAdd(absmin, pos, absmin);
VectorAdd(absmax, pos, absmax);

int count = gi.BoxEdicts(absmin, absmax, results, sizeof(results) / sizeof(*results), AREA_SOLID);

int i = 0;

for (; i < count; i++)
{
    edict_t *e = results[i];

    if (!e->takedamage)
        continue;

    // perform task on `e`
}
```

#### **✨**

Using the vanilla-style approach:

```cpp
// `pos` is the origin of the search.
static edict_t *results[256];
vec3_t absmin = pos - vec3_t{-128, -128, -128}, absmax = pos + vec3_t{128, 128, 128};

size_t count = gi.BoxEdicts(absmin, absmax, results, q_countof(result), AREA_SOLID, nullptr, nullptr);

int i = 0;

for (; i < count; i++)
{
    edict_t *e = results[i];

    if (!e->takedamage)
        continue;

    // perform task on `e`
}
```

You can use a filter function to reduce the number of entities written to the result array:

```cpp
// outside of the function:
BoxEdictsResult_t FilterFunction(edict_t *ent, void *arg)
{
    if (ent->takedamage)
        return BoxEdictsResult_t::Keep;

    return BoxEdictsResult_t::Skip;
}

// ......

// `pos` is the origin of the search.
static edict_t *results[256];
vec3_t absmin = pos - vec3_t{-128, -128, -128}, absmax = pos + vec3_t{128, 128, 128};

size_t count = gi.BoxEdicts(absmin, absmax, results, q_countof(result), AREA_SOLID, FilterFunction, nullptr);

int i = 0;

for (; i < count; i++)
{
    edict_t *e = results[i];

    // perform task on `e`
}
```

You can also use a filter function as a lambda, and use the argument to pass a pointer to some kind of state that you can use. In this example, the state is used as a backing array, to dynamically add entries:

```cpp
// `pos` is the origin of the search.
std::vector<edict_t *> result;
vec3_t absmin = pos - vec3_t{-128, -128, -128}, absmax = pos + vec3_t{128, 128, 128};

gi.BoxEdicts(absmin, absmax, nullptr, 0, AREA_SOLID, [](edict_t *ent, void *arg) {
    auto &arg_result = *reinterpret_cast<decltype(result) *>(arg);

    if (ent->takedamage)
        arg_result.push_back(ent);

    return BoxEdictsResult_t::Skip;
}, nullptr);

for (auto e : result)
{
    // perform task on `e`
}
```

#### **🪽**

Classic-style iteration; note that `max_count` must still be set even though the array type in Angelscript is dynamic. If `max_count` is 0, it follows the original design of not writing to the entity list.

```cpp
// `pos` is the origin of the search.
array<edict_t @> results;
vec3_t absmin = pos - vec3_t(-128, -128, -128), absmax = pos + vec3_t(128, 128, 128);

gi_BoxEdicts(absmin, absmax, results, 256, solidity_area_t::SOLID, null, null, false);

foreach (edict_t @e_handle : results)
{
    ASEntity @e = cast<ASEntity>(e_handle.as_obj);

    if (!e.takedamage)
        continue;

    // perform task on `e`
}
```

Using a callback function to perform filtering:

```cpp
// outside of the function:
BoxEdictsResult_t FilterFunction(edict_t @ent_handle, any @const arg)
{
    ASEntity @ent = cast<ASEntity>(ent_handle.as_obj);

    if (ent.takedamage)
        return BoxEdictsResult_t::Keep;

    return BoxEdictsResult_t::Skip;
}

// ......

// `pos` is the origin of the search.
array<edict_t @> results;
vec3_t absmin = pos - vec3_t(-128, -128, -128), absmax = pos + vec3_t(128, 128, 128);

gi.BoxEdicts(absmin, absmax, results, 256, solidity_area_t::SOLID, FilterFunction, null, false);

foreach (edict_t @e_handle : results)
{
    ASEntity @e = cast<ASEntity>(e_handle.as_obj);

    // perform task on `e`
}
```

Using a lambda and the filter argument to simplify entity iteration (note that this method does have slightly more overhead due to how `any` works):

```cpp
// `pos` is the origin of the search.
array<ASEntity @> results;
vec3_t absmin = pos - vec3_t(-128, -128, -128), absmax = pos + vec3_t(128, 128, 128);

any arg;
arg.store(@results);

gi_BoxEdicts(absmin, absmax, null, 0, solidity_area_t::SOLID, function(ent_handle, arg) {
    ASEntity @ent = cast<ASEntity>(ent_handle.as_obj);

    if (ent.takedamage)
    {
        array<ASEntity @> @results;
        arg.retrieve(@results);
    }

    return BoxEdictsResult_t::Skip;
}, @arg, false);

foreach (ASEntity @e : results)
{
    // perform task on `e`
}
```

<!-- tabs:end -->
</details>

### Finding entities by member/field

Another common operation is to find entities that match a given condition based on a member, such as their `classname`. In the majority of cases, this is done by string; the samples here will only focus on string-based searches.

Related functions:

<!-- tabs:start -->

#### **🍦**

```cpp
// find entities that match the given string in the supplied field offset.
edict_t *G_Find (edict_t *from, int fieldofs, char *match);

// pick a random entity (up to MAXCHOICES, which is 8) from the map that has the given targetname
edict_t *G_PickTarget (char *targetname);
```

#### **✨**

```cpp
// find entities that pass the given matcher function. this is provided in case you wish to make custom matchers.
edict_t *G_Find(edict_t *from, std::function<bool(edict_t *e)> matcher);

// find entities that match the given member of `edict_t`. The template paramter must be a pointer to a member of `edict_t`, and must be a `const char *`.
template<auto M>
edict_t *G_FindByString(edict_t *from, const std::string_view &value);

// pick a random entity (up to MAXCHOICES, which is 8) from the map that has the given targetname
edict_t *G_PickTarget(const char *targetname);
```

#### **🪽**

```cpp
// match entities that match the given string value in `member`. `T` must be a type that is castable from the `as_obj` member of `edict_t` (in the default framework, this should always be `ASEntity`)
T @+find_by_str<T>(T @+from, const string &in member, const string &in value) nodiscard

// pick a random entity from the map with the given targetname.
ASEntity @G_PickTarget(const string &in targetname);
```

<!-- tabs:end -->

Usage example (iterating every entity in the map with the classname "monster_soldier"):

<!-- tabs:start -->

#### **🍦**

```cpp
edict_t *e = NULL;

while ((e = G_Find(e, FOFS(classname), "monster_soldier")) != NULL)
{
    // perform task on `e`
}
```

#### **✨**

```cpp
edict_t *e = nullptr;

while ((e = G_FindByString<&edict_t::classname>(e, "monster_soldier")) != nullptr)
{
    // perform task on `e`
}
```

#### **🪽**

```cpp
ASEntity @e = null;

while ((@e = find_by_str<ASEntity>(e, "classname", "monster_soldier")) !is null)
{
    // perform task on `e`
}
```

<!-- tabs:end -->