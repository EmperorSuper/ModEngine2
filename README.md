# Mod Engine 2

Mod Engine 2 is a ground up rewrite of Mod Engine, a runtime code patching and injection library used for adding modding functionality to the Souls games by FROM Software.

# Table of Contents
1. [Get started](#get-started-guide)
2. [Supported games](#supported-games)
2. [Differences](#what-are-the-differences)
3. [Features](#features)

## Get started guide

1. Download the latest [release](https://github.com/soulsmods/ModEngine2/releases).
2. Create a folder in the mod folder, I recommend the name of the mod for clarity.
3. Unpack the mod into the just created folder.
4. Edit the `eldenring_config.toml` (or your game of choice) to include the folder you just created.

```toml
mods = [
  { enabled = true, name = "default", path = "mod\\testmodName" },
  { enabled = false, name = "default", path = "mod\\disabledTestmodName" },
]
```

<details>
  <summary>Folder structure example (CLICK ME!)</summary>

  ![folder exmaple image](docs/media/mod-engine-mod-folder-example.png)

</details>

## Supported games

See the list below for information on games that currently have support in Mod Engine 2.

- [x] Dark Souls 3
- [x] Elden Ring
- [ ] Dark Souls 2: SOTF
- [ ] Dark Souls Remastered
- [ ] Sekiro: Shadows Die Twice
- [ ] Bloodborne


## What are the differences?

There are some big architecture differences between legacy Mod Engine and the new version, the bulk of which is in 1) the configuration format and 2) how we get ModEngine loaded into the game.
More details about these differences are listed below.

### New configuration format

The old `.ini` format is gone, replaced by a new structured configuration file based on [TOML](https://toml.io/en/).
This change is to suit the new extension model of Mod Engine by allowing plugins to specify their own configuration requirements.

This is used by the mod loader plugin to specify lists of mods that should be loaded and whether they should be enabled or not.
It may be extended in future to support a "Mod Manifest", which would contain additional information like website, version, authors, etc.

### Sideloading as `dinput8.dll` is optional

With the introduction of a launcher we no longer need to rely on games loading via `dinput8.dll` and we can instead launch the game pre-configured.
This frees up this module for anything else that relies on being sideloaded.

### Support for loading multiple mods

Multiple game roots can be specified in configuration, allowing users to run multiple mods concurrently without replacing files in their mod folder manually.
Note, however, that this is still restricted to mods that don't replace conflicting files.

### Focus on tooling for mod creators

The primary driver behind development of Mod Engine 2 is creating a tool that can be used to rapidly reverse the games that we are interested in.
To this end, we offer functionality to make the reversing process easier:

- Integration with Optick Profiler
- Runtime scripting and live code patching
- Crash dumps for all users

## Features

Mod Engine 2 covers all existing functionality from the original Mod Engine for Dark Souls but introduces some important new features that mod authors should be aware of. 

### Mod Launcher

We have created a launcher application that is designed to boot games with your mods pre-installed without the user having to do any manual file copying on their side.
Historically, mods would live in the same location as the game folder and Mod Engine would attempt to redirect requests for game files to mod file paths within subdirectories of this game folder.
This deployment model was inflexible and required a lot of manual tweaking if a user wanted to have multiple mods installed, switching between them as the choose.

To solve this problem `modengine2_launcher` was created.
This simple command-line application has 3 main objectives.

1. Find where a user has installed the game
2. Start the game with modengine2.dll already loaded
3. Pass information to modengine2.dll about the configuration the user launched the game with

This allows us to keep mods, modengine, and the game itself completely detached. 
As a result, running the game directly from Steam will always result in a vanilla instance being launched.

### Extension/plugin support

Mod Engine 2 introduces support for extensions and plugins which are designed to replace the old chainloading mechanism from legacy Mod Engine.
Extensions are able to use core functionality to register patches, code hooks, and interact with other extensions.
This will eventually be extended to support runtime Lua scripting in the future.

### Debugger support

ScyllaHide is included with Mod Engine 2 distributions and allows debugging games via native debuggers without worrying about evading anti-debug techniques using programs like Cheat Engine.
Instead, you can use your favourite debugger (WinDbg, x64dbg, or even Cheat Engine without relying on VEH support).

### Lua scripting

**WIP**










ModEngine2 DLL that works with Elden Mod Loader

https://github.com/techiew/EldenRingModLoader
Also AC6: https://modslab.net/en/armored_core_vi_fires_of_rubicon/mods/ReAaefZ9fY/
How to use?

Get Elden Mod Loader from https://www.nexusmods.com/eldenring/mods/117 and follow the install instructions.
Grab this ModLoader2 release, and put the modengine2/bin/lua.dll and modengine2/bin/modengine2.dll files into the ELDEN RING/Game/mods directory.

Set your elden ring launch option in steam like follows:
echo "%command%" | sed 's/start_protected_game/eldenring/' | WINEDLLOVERRIDES="dinput8.dll=n,b" sh
This skips the anti-cheat and tells wine to load the native version of dinput8.dll (Elden Mod Loader)

The modified modengine2 by default looks for config file in ELDEN RING/Game/modengine.toml, you can change this with MODENGINE_CONFIG environment variable if needed.

For AC6 see this issue: #19
How to use with seamless co-op

Copy the SeamlessCoop/elden_ring_seamless_coop.dll to the mods directory.
How to use with enemy / item randomizer?

Unpack the randomizer mod to game folder, you should have a subfolder called randomizer.
Run the randomizer exe in that subfolder and do your randomization.

Use the following modengine.toml config:

# Global mod engine configuration
[modengine]
# If set to true the debug console will appear while the game is running
debug = false

# Mod loader configuration
[extension.mod_loader]
enabled = true

# Not currently supported for Elden Ring
loose_params = false

# List of directories that contain modded files in order of prioritization. Inside each specified mod directory must have the game
# assets in Fromsoft's asset structure. I.e. if you mod parts/something.partsbnd.dcx, the modded version must be at mod/parts/something.partsbnd.dcx.
# Absolute paths to mods are supported but must use '\\' to separate path items. For example, if your mod is at E:\coolstuff\coolmod, you must enter
# the path in the config as "E:\\coolstuff\\coolmod".
# If there's no drive specifier (C:, D:, etc), the path is relative to where the launcher is located. For example, having the path as "mod" will tell
# modengine 2 to look for the directory mod inside the mod engine 2 directory with the launcher.
#
# Multiple mods must be separated with commas. For example if you have 3 mods, you will have something like the following:
# mods = [
#    { enabled = true, name = "coolmod", path = "mod1" },
#    { enabled = true, name = "nicemod", path = "mod2" },
#    { enabled = true, name = "sosomod", path = "mod3" }
# ]
# Note that modengine 2 currently has no way to resolve conflicting files including regulation.bin, and thus the mod with the highest priority
# will have the modded file be loaded in the case of conflict. Some support for merging of params and potentially other assets is considered for
# a future release.
mods = [
    { enabled = true, name = "default", path = "randomizer" }
]

# When enabled, scylly hide will be injected into the game. This allows for antidebug measures in the game to be bypassed so that you can attach
# debuggers such as Cheat Engine, x64dbg, windbg, etc to the game without as much trouble. If you're not reverse engineering the game, this option
# is probably not for you.
[extension.scylla_hide]
enabled = false

Can you use this with both co-op and enemy randomizer?

Yes

Example file tree of such setup:

ELDEN RING
└── Game
    ├── mods
    │   ├── lua.dll
    │   ├── modengine2.dll
    │   ├── elden_ring_seamless_coop.dll
    │   └── seamlesscoopsettings.ini
    ├── movie
    ├── randomizer
    │   ├── diste
    │   ├── event
    │   ├── map
    │   ├── msg
    │   ├── script
    │   ├── sfx
    │   ├── spoiler_logs
    │   ├── 12784.randomizeopt
    │   ├── EldenRingRandomizer.exe
    │   ├── README.txt
    │   ├── config_eldenringrandomizer.toml
    │   ├── locations.txt
    │   ├── oo2core_6_win64.dll
    │   └── regulation.bin
    ├── sd
    ├── amd_ags_x64.dll
    ├── bink2w64.dll
    ├── dinput8.dll
    ├── eldenring.exe
    ├── eossdk-win64-shipping.dll
    ├── eossdk-win64-shipping.so
    ├── mod_loader_config.ini
    ├── modengine.toml
    ├── oo2core_6_win64.dll
    ├── regulation.bin
    ├── start_protected_game.exe
    └── steam_api64.dll

The mods do not load, or load sometimes

Try changing the load_delay in mod_loader_config.ini, some people report success with value such as 2500
