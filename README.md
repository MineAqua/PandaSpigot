# PandaSpigot [![Build](https://img.shields.io/github/actions/workflow/status/hpfxd/PandaSpigot/build.yml?branch=master&label=Build)](https://github.com/hpfxd/PandaSpigot/actions/workflows/build.yml) [![Discord](https://img.shields.io/discord/1048733138655924274?label=Discord)](https://discord.gg/m6vCCX6Hvr) [![Servers](https://img.shields.io/bstats/servers/15154?label=Servers)](https://bstats.org/plugin/bukkit/PandaSpigot/15154)
Fork of Paper for 1.8.8 focused on improved performance and stability.

## Highlights
- **Backported API enhancements from newer versions**
    - Native Adventure API
    - ServerTickStartEvent & ServerTickEndEvent
    - PlayerChunkLoadEvent & PlayerChunkUnloadEvent
    - PlayerHandshakeEvent
    - EntityMoveEvent

- **Greatly improved network performance**
    - **Updating to Netty 4.1** offers the ability to use newer Java versions with epoll on Linux.
    - **Improved flush handling** to massively improve entity tracker performance.
    - **Support for Unix domain sockets** to avoid the overhead of TCP when using a proxy on the same machine.

- **More configuration options**, such as:
    - Customizable knockback
    - World and player data saving

## Parallel world ticking
`parallelWorldTicking` is an opt-in setting in `pandaspigot.yml`. It assigns loaded worlds to a bounded number of stable worker lanes and waits for every lane before the server continues to connection and player processing. `parallelWorldTickingThreads` controls the lane limit (2–16); changing it reassigns worlds to new lanes.

This runs world and entity events from worker threads. Enable it only when every installed plugin supports concurrent world events; plugins must not block waiting for synchronous server-thread work during those events. It does not isolate regions or arenas within one world. Cross-world entity transfers requested during a parallel world tick are skipped, and world ticks run serially while Timings is enabled. The setting is off by default.

See a full list of patches [here](./patches/).

## Modern armor swapping

Clients running Java 1.19.4 or newer can right-click an armor piece in their
selected hotbar slot to swap it with the equipped piece. PandaSpigot captures the
original handshake before local protocol translation, without calling plugin APIs.
The translator (such as ViaVersion) must run on this backend: translation on a
proxy hides the original protocol. Protocol translation is still required for
modern clients to connect to a 1.8.8 server.

Clients on 1.8–1.19.3 retain the original behavior. Known release protocols from
1.19.4 through 26.3 are supported; unknown protocols and snapshots leave swapping
disabled. Extend the known protocol range when adding newer client releases.
Item-use cancellation through `PlayerInteractEvent` is respected.

## Using
You can download the latest pre-built server JAR by clicking the download button below.  
[![Download](https://custom-icon-badges.demolab.com/badge/-Download-blue?style=for-the-badge&logo=download&logoColor=white)](https://downloads.hpfxd.com/v2/projects/pandaspigot/versions/1.8.8/builds/latest/downloads/paperclip)

For support, please join our [Discord](https://discord.gg/m6vCCX6Hvr).

## API 
See our API patches [here](./patches/api/).  
[![Javadocs](https://repo.hpfxd.com/api/badge/latest/releases/com/hpfxd/pandaspigot/pandaspigot-api?name=Javadocs)](https://repo.hpfxd.com/javadoc/releases/com/hpfxd/pandaspigot/pandaspigot-api/1.8.8-R0.1-SNAPSHOT/raw/index.html)
<details>
<summary>Maven</summary>

```xml
<repositories>
    <repository>
        <id>hpfxd-repo</id>
        <url>https://repo.hpfxd.com/releases/</url>
    </repository>
</repositories>

<dependencies>
    <dependency>
        <groupId>com.hpfxd.pandaspigot</groupId>
        <artifactId>pandaspigot-api</artifactId>
        <version>1.8.8-R0.1-SNAPSHOT</version>
        <scope>provided</scope>
    </dependency>
</dependencies>
```
</details>

<details>
<summary>Gradle (kts)</summary>

```kotlin
repositories {
    mavenCentral()
    maven(url = "https://repo.hpfxd.com/releases/")
}

dependencies {
    compileOnly("com.hpfxd.pandaspigot:pandaspigot-api:1.8.8-R0.1-SNAPSHOT")
}
```
</details>

## Requirements
To work with PandaSpigot, you will need:
- JDK 21 or higher
- Git
- Bash

On JDK 25, compact object headers can be enabled with `-XX:+UseCompactObjectHeaders`. Measure this with your server's plugin set before using it in production.

### Panda Bash build script
The `panda` Bash script is the entry point for setup, patching, and builds.
Run it from a Bash-compatible shell on your platform. Run `./panda` without
arguments to print its command list.

Normal commands:

| Command | Description |
| --- | --- |
| `rb`, `rebuild` | Rebuild patches, can be called from anywhere. |
| `setup` | Remap, decompile, and patch Minecraft. Can be run from anywhere. |
| `p`, `patch` | Apply all patches to the project without building it. Can be run from anywhere. |
| `j`, `jar` | Build the project, `paperclip.jar` will be output. Can be run from anywhere. |
| `c`, `clean` | Removes build files under `PandaSpigot-API`, `PandaSpigot-Server`. |
| `dc`, `distclean` | Removes all generated files, `PandaSpigot-API`, `PandaSpigot-Server`, `base`, and `work`. |
| `con`, `continue` | Shortcut command for running `git am --continue` or `git rebase --continue`. |

Commands that require `. ./panda install` first:

| Command | Description |
| --- | --- |
| `r`, `root` | Change directory to the root of the project. |
| `a`, `api` | Move to the `PandaSpigot-API` directory. |
| `s`, `server` | Move to the `PandaSpigot-Server` directory. |
| `e`, `edit` | Use to edit a specific patch, give it the argument `server` or `api` respectively to edit the correct project. Use the argument `continue` after the changes have been made to finish and rebuild patches. Can be called from anywhere. |
| `install` | Add an alias to `$RCPATH` to allow full functionality of this script. |

Install the default `panda` alias with:

```bash
. ./panda install
```

You can also provide a custom alias name:

```bash
. ./panda install example
```

`PandaSpigot-API` and `PandaSpigot-Server` are generated work trees created by
the build scripts. The patch creation process follows Paper's contributing guide,
which is linked below; `./panda rb` is the PandaSpigot command that saves those
work tree commits back into `patches/api` and `patches/server`.

### Patch workflow
PandaSpigot uses a patch-based workflow. Changes are committed once inside the
generated work tree, then committed again from the repository root after the
patch files are rebuilt.

Use this order:

1. Make your changes in `PandaSpigot-API` or `PandaSpigot-Server`.
2. Commit those changes inside that generated work tree.
3. Return to the repository root and run `./panda rb`.
4. Review the regenerated files in `patches/api` or `patches/server`.
5. Commit the regenerated patch files from the repository root.

Only push commits from the repository root. Do not push or pull from
`PandaSpigot-API` or `PandaSpigot-Server`; their commits are only used to create
patch files. Commit work tree changes before rebuilding or reapplying patches,
because those commands can reset the generated work trees.

## Contributing
You can mostly follow [Paper's contributing guide](https://github.com/PaperMC/Paper-archive/blob/ver/1.16.5/CONTRIBUTING.md), just remember:
- Multi-line changes start with `// PandaSpigot start` and end with `// PandaSpigot end`
- If the change isn't obvious, add a small explanation like this: `// PandaSpigot start - reason`
- One-line changes should have `// PandaSpigot` at the end of the line.
- Follow Java code style (aka. Oracle style), with some exceptions:
  - If you are modifying upstream files, keep your diff size minimal. Going over 80 characters per line is fine to make this happen.
  - When in doubt or the code around your change is in a clearly different style, use the same style as the surrounding code.

When contributing, please think about the side effects of any changes you write.
Plugin compatibility is important, and we wish to minimize any breakage.

Please do not open pull requests for features that you cannot justify the existence of,
and the added maintenance costs of that come along with them. If you are thinking of
adding a feature that may be controversial, please open an issue first!

## Procedencia de los parches

La autoría original se conserva en las cabeceras de los archivos de `patches/`. WindSpigot se utilizó como referencia de integración para los siguientes parches; las adaptaciones y correcciones realizadas al incorporarlos son de **MineAqua**:

| Parche | Procedencia |
| --- | --- |
| `0163-Reduce-NBTTagCompound-memory-overhead` | Paper/Spottedleaf, mediante WindSpigot; adaptado por MineAqua para usar `HashMap` sin añadir fastutil. |
| `0164-Fix-entity-and-command-block-memory-leaks` | KigPaper-0039/0128, mediante NachoSpigot y WindSpigot; conserva la API de daño con referencias débiles. |
| `0165-Fix-EnchantmentManager-memory-leaks` | KigPaper-0129 y FlamePaper-0105/Minetick. |
| `0166-Fix-entity-target-and-crafting-view-memory-leaks` | KigPaper-0138/0161, con correcciones de null-safety de WindSpigot y MineAqua. |
| `0167-Fix-world-map-tracking-memory-leak` | FlamePaper-0105/Minetick. |
| `0168-Optimize-hopper-inventory-entity-lookup` | FlamePaper-0034; MineAqua corrige la comprobación para usar el contador de entidades con inventario y no perder minecarts. |
| `0169-Avoid-EnumDirection-array-allocations` | NachoSpigot-0004/0005, por Heath Logan Campbell. |
| `0170-Reuse-empty-int-arrays` | NachoSpigot, por Heath Logan Campbell. |

### Parches de MineAqua aplicados sobre PandaSpigot

El upstream PandaSpigot (`hpfxd/PandaSpigot`) termina en `0138` tanto en servidor como en API. Todos los parches aplicados a continuación sobre esa base son puertos y desarrollos propios de **MineAqua** (adaptados desde MASpigot, o escritos directamente para esta rama):

**Servidor** (`patches/server/`, `0139`–`0162`):

| Parche | Descripción |
| --- | --- |
| `0139` | MASpigot: corrección del estado de bloqueo con espada. |
| `0140` | MASpigot: rango de tracking para jugadores invisibles. |
| `0141` | MASpigot: multiplicador de daño de armadura configurable. |
| `0142` | MASpigot: exposición de los campos del paquete de multibloque. |
| `0143` | MASpigot: opción de tile entities en snapshots de chunk. |
| `0144` | MASpigot: toggles de física por mundo. |
| `0145` | MASpigot: toggles de ticks de servidor. |
| `0146` | MASpigot: toggles de experiencia de jugador. |
| `0147` | MASpigot: opción de tracking de jugadores más rápido. |
| `0148` | MASpigot: toggle del evento PlayerMoveEvent. |
| `0149` | MASpigot: opción para reducir el spam de sonidos de paso. |
| `0150` | MASpigot: opciones de física de pociones lanzadas. |
| `0151` | MASpigot: comando de prueba de snapshots de chunk. |
| `0152` | MASpigot: restauración del sistema de perfiles de knockback. |
| `0153` | MASpigot: knockback configurable de la caña de pescar. |
| `0154` | MASpigot: publicación de PandaSpigot-Server a Reposilite de MineAqua. |
| `0155` | MASpigot: compatibilidad con `com.destroystokyo.paper.Title`. |
| `0156` | MASpigot: eventos de efectos de poción. |
| `0157` | MASpigot: exposición de la clase `EntityTypes` al mapeo `EntityType`. |
| `0158` | MASpigot: implementación de `hideEntity`/`showEntity`. |
| `0159` | MASpigot: restauración de la extensión `MovementHandler`. |
| `0160` | Ticking paralelo de mundos opt-in (`parallelWorldTicking`). |
| `0161` | Compartición de cambios de multibloque pre-codificados. |
| `0162` | Reducción de memoria por chunk y misceláneos de Java 21. |

**API** (`patches/api/`, `0034`–`0040`):

| Parche | Descripción |
| --- | --- |
| `0034` | MASpigot: opción de tile entities para snapshots de chunk (API). |
| `0035` | MASpigot: publicación a Reposilite de MineAqua. |
| `0036` | MASpigot: API del sistema de perfiles de knockback. |
| `0037` | MASpigot: compatibilidad con `com.destroystokyo.paper.Title`. |
| `0038` | MASpigot: eventos de efectos de poción. |
| `0039` | MASpigot: API `hideEntity`/`showEntity`. |
| `0040` | MASpigot: extensión `MovementHandler`. |

Además de estos, los parches `0163`–`0170` de servidor (listados en la tabla de procedencia superior) fueron escritos por **MineAqua** como adaptaciones de optimizaciones de terceros. Los demás parches pertenecen a PandaSpigot, Paper, Spigot o sus respectivos autores, identificados en cada archivo de parche.
