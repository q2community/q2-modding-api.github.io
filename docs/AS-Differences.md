# Differences from C/C++

Every language has its own quirks - AngelScript is certainly not without them. The codebase for the AngelScript implementation of Quake II's game module is different from the C/C++ codebase in a few areas, mainly because going over every single line of the code gave me the opporunity to make things better for the AngelScript framework.

This page will highlight a few of the differences and best practices that should be followed when working with Q2AS.

# Entity Handles / References

AngelScript supports the concept of handles *and* references; references (when dealing with by-ref types) are like non-null handles, similar-ish to how C++ references work. Handles, on the other hand, can be null.

Where appropriate, the AngelScript code now uses references to express this non-null contract. For instance, a `think` callback function will never be called on a null entity, so it is declared as a reference rather than a handle.

This might cause issues when porting old code; consider the following code from a theoretical C/C++ mod:

```cpp
void MyThink(edict_t *ent)
{
    ent->nextthink = level.time + 1_sec;

    ent = ent->owner;
    ent->count++;
}
```

This re-assigns the `ent` pointer to a different entity. In AngelScript, you cannot reassign a reference; it will attempt to perform *value assignment*, which is definitely not what you want here, and will cause weird things to happen.

To help catch these mis-uses, you will run into an exception when this occurs. Note that the exception may be misleading, since AngelScript does not currently support nested exceptions, but it should point you to the line where the assignment occurred and hopefully be obvious.

To do what the code above does in AngelScript, one would have to use a temp variable:

```cpp
void MyThink(ASEntity &ent)
{
    ent.nextthink = level.time + time_sec(1);

    ASEntity @owner = ent.owner;
    owner.count++;
}
```