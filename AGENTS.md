# AGENTS.md

JTube is a discontinued J2ME (CLDC 1.1 / MIDP 2.0) YouTube client using the Invidious API. It has no Gradle/Maven/Ant build, tests, lint, typecheck, or CI.

## Build and setup
- `lib/json`, `lib/midletintegration`, and `lib/j2mekeyboard` are git submodules listed in `.gitmodules`, but are empty in this checkout. Clone each repo into its `lib/` path before building. `javac` alone also fails because `javax.microedition.*` comes from a J2ME SDK.
- Build in Eclipse with MTJ 2.0.1 and a J2ME SDK (README: JDK 1.5, Symbian^3 and/or S40 SDK). `.classpath` is bound to "Nokia Symbian3 SDK v1.0".
- Package through `Application Descriptor` -> "Create package" -> check "Use deployment directory". Output goes under `deployed/`.
- `build.properties` defines per-platform source roots in `Symbian.includes` and `Blackberry.includes`. Add new source/resource roots to both it and `.classpath`.
- There is no automated verification command; verify by packaging in Eclipse and running the JAR/JAD in an emulator or device.

## Platform and code constraints
- Keep the existing Java 1.4-style subset: raw `Vector`/`Stack`, no generics, enhanced-for, diamond, `StringBuilder`, `java.nio`, `java.util.concurrent`, or other Java SE APIs not in CLDC/MIDP.
- `Application Descriptor` is the MIDlet manifest. `App.startApp()` throws unless `MIDlet-Name` is `JTube` and `MIDlet-Vendor` is `nnproject`. Entry point is `cc.nnproject.ytapp.App2`, which delegates to `jtube.App`.
- `res/` is a source/resource root included by MTJ; its files land at the JAR root. Code loads them by absolute classpath path (`/splash.png`, `/jtindex`, `/jtlng_de`).
- `res/jtlng_*` are binary locale files: repeated `short` `LocaleConstants` ID + `readUTF` string, terminated by `-1`. Missing IDs fall back to built-in English. Register new locales in `res/jtindex`; the separate `jtlngedit` tool builds these files.
- Settings persist as JSON in RMS record store `ytconfig`. When adding a setting, update both `Settings.loadConfig()` and `Settings.saveConfig()` and maintain the `"v"` schema value. Subscriptions, avatars, and thumbnail caches use separate RMS stores in `LocalStorage`.
- HTTP/Invidious access is centralized in `App.invApi`, `App.getImageBytes`, and `Util.get/getUtf/head`. These block. Use them instead of raw `Connector`. `Util` contains deliberate per-device hacks (Samsung data streams/delays, BlackBerry suffixes, charset fallback); do not remove them as redundant.
- Helper/proxy/update URLs and the default Invidious instance are hardcoded in `Constants.java` and `Settings.java`.

## Threading and UI
- Painting runs on the dedicated "UI Thread" (`App.run()` -> `AppUI.run()` -> `JTubeCanvas.updateScreen()`). Never run blocking network/image work there.
- Background loading normally goes through `ILoader.load()` queued with `Loader.add(...)` and executed by `LoaderThread`s. `App.inst.schedule(Runnable)` queues to the separate "Task Thread" (used for debounced actions and cache/preview follow-ups); `schedule(UIScreen)` hides that screen. Call `repaint()` after UI changes.
- `App.watch()` calls `Loader.stop()`, cancelling queued loaders; do not assume background loads survive video playback.
- Most UI is custom canvas code (`AppUI`, `JTubeCanvas`, `UIScreen`, `UIItem`, `ui/screens/`, `ui/items/`), not standard LCDUI. LCDUI is used for alerts and a few forms/lists.

## Layout
- `src/jtube/App.java` - MIDlet lifecycle, Invidious API calls, playback/download, launch arguments
- `src/jtube/ui/` - custom canvas UI, screens, items, Nokia text/font shims
- `src/jtube/models/` - Invidious models/loaders
- `src/jtube/Settings.java`, `src/jtube/LocalStorage.java` - RMS persistence
- `src/cc/nnproject/ytapp/App2.java` - MIDlet entry point
- `src/tube42/lib/imagelib/` - vendored image utilities
