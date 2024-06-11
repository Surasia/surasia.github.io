---

layout: post
author: Surasia
title: "Havokscript: Writing Custom Modes for Halo Infinite (Part 1)"
date:   2024-06-11
permalink: /havokscript-p1
categories: halo

---

![Havokscript Thumbnail](/assets/images/havokscript.jpg)
<span style="font-size:12px;color:#c9c9b5;">
  Image showcasing armor from Halo Infinite next to a block of Havokscript code.
</span>

<hr>
<br>

### Intro: Why Havokscript?

Halo has had a long history of utilizing a custom scripting language for game logic, spanning back to the original Halo: Combat Evolved which used a lisp-inspired `haloscript`. As the "Blam!" engine expanded and became hard to maintain, 343 Industries chose to switch to `HavokScript` for their next project, Halo 5: Guardians. Haloscript was a collection of simple bindings to C code, the switch to the new HavokScript retained a large part of engine functions.

<br>

Havokscript is a now-discontinued part of the Havok Game SDK, built upon Lua 5.1. It integrates structures, prototypes, a debugger, external hooks and additional type checking. There is no public documentation available for this language, however a [GDC Presentation](https://ubm-twvideo01.s3.amazonaws.com/o1/vault/gdc2011/slides/Malcolm_Tyrrell_Programming_EfficientLuaScripting.pdf) by Malcolm Tyrrell expands on the major changes made in Havokscript.

<br>

### Targeting Our Code

Before writing any code, it's important to define what will run on the server or the  client. In HavokScript, the server can interface with the client to run client-specific commands, such as `os.execute()`. This is done via the `RunClientScript()` function, which also allows for specific code to be executed for specific players.

<br>

The target for a section of our code is set using preprocessor directives, which are a custom extension to Havokscript added by 343 Industries.

- `--## CLIENT`: Client-side code
- `--## SERVER`: Server-side code
- `--## COMMON`: Code that is interpreted on both client and server.

<br>

```lua
  --## SERVER

  function HelloWorld():void -- optional type check: returns "void" type.
      print("Hello World!"); -- Prints "Hello World" to internal console.
  end

  --## CLIENT

  function remoteClient.HelloWorld():void 
  -- Every function needs to be defined under "remoteClient".
      print("Hello World!");
  end

  --## SERVER

  RunClientScript("HelloWorld"); -- Runs "Hello World" on client.

  --## COMMON

  function HelloWorld():void -- Code gets interpreted on both server and client.
      print("Hello World!");
  end
```

<br>

### The Parcel System

Parcels in HavokScript, in the simplest sense, are special classes that are initialized and run by the game mode at startup. These parcels are created using `Parcel.MakeParcel`, which creates an instance of a Parcel object which can then be initialized. They serve the main purpose of allowing the hooking of event handlers and registering callbacks, which in a practical sense means that the game can execute specific parts of the Parcel with arguments from the engine. For example, by hooking the `g_eventTypes.weaponPickUpEvent` event, we can run our custom code which will have access to variables such as `weaponPosition` locally.

<br>

To create a Parcel, we first need to create a global table with an instance name. Afterwards, we create a new instance of the parcel using the `New()` method, which will return a table of the newly created instance. For example, the code below takes in the arguments of a Slayer gamemode, which can be used globally in the parcel.

<br>

```lua
  --## SERVER

  global exampleParcel:table = Parcel.MakeParcel
  {
      instanceName = "exampleParcel",
  }

  function exampleParcel:New(initArgs:SlayerInitArgs):table
      local exampleParcelInstance = self:CreateParcelInstance();
      exampleParcelInstance.instanceName = initArgs.instanceName;
      return exampleParcelInstance;
  end
```

<br>

We can now define various methods for our Parcel, some of which will be run by the engine at startup:

- `Run()`: Runs at gamemode intro.
- `Initialize()`: Runs before gameplay.

<br>

```lua
  --## SERVER

  function exampleParcel:Initialize():void
      print("Hello from Init!");
  end

  function exampleParcel:Run():void
      print("Hello from Run!");
  end
```

<br>

The `Initialize()` function in particular allows us to register events, which will run a custom function that is called when an event is received. Gamemode settings can also be changed using this method.

```lua
  --## SERVER

  function exampleParcel:Initialize():void
      self:RegisterGlobalEventOnSelf(g_eventTypes.weaponPickupEvent, self.HandleWeaponPickedUp)
      -- Creates a global event callback to a method of our parcel.
  end

  function exampleParcel:HandleWeaponPickedUp():void
      print("This code runs when a player picks up a weapon!");
  end
```

<br>

### C++ to Lua API: Functions, Definitions
If you've been following on so far, you might've noticed that we are calling functions such as `RegisterGlobalEventOnSelf` which are seemingly not declared in any of our code. This is because HavokScript offers a set of bindings to functions in the actual engine, initialized at runtime.

<br>

The list of built-in functions and enum definitions is quite large, and is mostly undocumented. Most functions however are clearly named and can be used in parts of your code. Some examples include:
- `Variant_GetEngineName()`: Returns engine name string (ex: "Slayer").
- `Toggle_TaaEnabled()`: Toggles Temporal Anti Aliasing.
- `Player_GetXuid(Player)`: Gets Xbox User ID of player.

<br>

The full list of functions and enums (such as `g_eventTypes`) can be found in the [lua environment dump](/assets/env_alphabetical.json) created using Soupstream's debug mode. Unfortunately though, due to no documentation being available, most of these functions do not have known argument types and return types and requires guesswork to implement.

<br>
...
to be continued!