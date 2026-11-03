# JTube
<h3>Discontinued!</h3>

YouTube client based on the Invidious API for Java devices with MIDP 2.0 support.

<a href="FAQ.md">FAQ</a><br>

## Prebuilt package

A patched `2.90.3` build is included in `dist/`:

- `JTube-2.90.3-t455.jar`
- `JTube-2.90.3-t455.jad`

This build targets the Tecno T455 and other MIDP 2.0 / CLDC 1.1 devices. Install both files in the same folder and open the JAD, or install the JAR directly. The Tecno T455 build is maintained by Alfred Njobvu.

Changes in this build:

- Default Invidious instance is `http://s60tube.io.vn/`
- API proxy is disabled by default
- `301`, `302`, `303`, `307`, and `308` redirects are followed
- Settings migrate to schema `v11`, resetting old dead-instance and proxy defaults
- Tecno T455 forces keypad-only navigation and ignores touch input
- Menus highlight the selected row for keypad navigation

The API endpoints were verified over HTTP, but video playback still depends on the device's network speed and codec support.

## Running JTube from other MIDlets
<a href="https://github.com/shinovon/MIDletIntegrationLibrary">MIDletIntegration library</a><br>

Example code:<br>
```
try {
	if(MIDletIntegration.startApp(this, "JTube", "nnproject", "0xAFCE0816", 1260, "url=" + Util.encodeURL("https://youtube.com/watch?v=somevideo"))) {
		notifyDestroyed();
	}
} catch (MIDletNotFoundException e) {
	e.printStackTrace();
} catch (ProtocolNotSupportedException e) {
	e.printStackTrace();
} catch (IOException e) {
	e.printStackTrace();
}
```

## Building

### Requirements

- Any J2ME SDK (for example WTK, S40 5th, or Symbian^3)
- CLDC 1.1 and MIDP 2.0 API libraries
- JSR-75 API library
- `com.nokia.mid.ui` stubs or a Nokia SDK
- Eclipse SDK with MTJ 2.0.1, or ECJ 4.6.1 for a command-line build
- JDK capable of running ECJ

Clone the repository and initialize the submodules:

```
git submodule update --init --recursive
```

If the submodules are empty, clone them into `lib/json`, `lib/midletintegration`, and `lib/j2mekeyboard`.

### Eclipse / MTJ

Import the project from the repository directory in Eclipse.

Open `Application Descriptor` in the Package Explorer, click **Create package**, check **Use deployment directory**, and press **Finish**.

To use the **Obfuscate the code** option, configure ProGuard in Eclipse preferences first.

The built JAR and JAD files appear under `<project path>/deployed/S40_5th_Edition_SDK/`.

### Command line

`javac` alone is not enough because the project uses CLDC/MIDP APIs and optional Nokia UI classes. The released package was built with ECJ 4.6.1 using `-source 1.3 -target cldc1.1`.

1. Compile local `com.nokia.mid.ui` stubs (`TextEditor`, `S60TextEditor`, `TextEditorListener`, and `DirectUtils`) into a classes directory.
2. Compile the app and libraries with the CLDC, MIDP, JSR-75, and stub classes on the bootclasspath.
3. Package the compiled classes plus the contents of `res/` and `lib/j2mekeyboard/res/` at the JAR root.
4. Add the manifest attributes from `Application Descriptor` and generate a matching JAD with the correct `MIDlet-Jar-URL` and `MIDlet-Jar-Size`.

The final artifacts are placed in `dist/`.
