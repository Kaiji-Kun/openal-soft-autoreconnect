OpenAL Soft with auto-reconnect
===============================

This is a fork of [OpenAL Soft](https://github.com/kcat/openal-soft) 1.24.3
that moves a game's audio to the current Windows default output when its
output device disappears (for example Bluetooth earbuds disconnecting) or
when the Windows default output changes. Games that don't handle audio device
changes themselves no longer go silent until restarted.

The auto-reconnect patch was written with Claude (Anthropic's AI) and tested by
[Kaiji-Kun](https://github.com/Kaiji-Kun). OpenAL Soft itself is by Chris Robinson
and contributors.

**Download:** prebuilt 64-bit and 32-bit `OpenAL32.dll` files are on the
[Releases](../../releases) page.

### How it behaves

- A game using the "default" device follows the Windows default output.
- A game using a named device keeps it until that device is lost, then
  follows the Windows default from then on.
- The switch happens about a second after Windows reports the change, with
  retries for ~10 seconds while a reconnecting Bluetooth device settles.
- The game's own audio settings (sample rate, HRTF etc.) are kept.
- Apps that register for device-change events through `ALC_SOFT_system_events`
  are assumed to handle switching themselves and are left alone.
- A sound playing at the moment a device is lost may stop rather than resume;
  new sounds play normally.

### Installing in a game

Pick the DLL by whether the **game** is 64-bit or 32-bit: `Win64` for 64-bit
games (most games from the last decade, e.g. X4: Foundations), `Win32` for
32-bit games.

The "32" in `OpenAL32.dll` is just the traditional file name and does **not**
mean 32-bit: 64-bit games use a file called `OpenAL32.dll` too.

1. In the game's folder, rename the game's own OpenAL DLL (usually
   `OpenAL32.dll`) to e.g. `OpenAL32.dll.bak`.
2. Copy in the DLL from the matching folder, with exactly the same name the
   game's original had (usually `OpenAL32.dll`).
3. To undo, delete it and rename the `.bak` back.

Not sure? In Task Manager a 32-bit game shows "(32 bit)" after its name. If
you pick the wrong one, the game won't start or will have no sound; swap in
the other.

Only for games that load OpenAL from a separate DLL. Avoid swapping DLLs in
games with anti-cheat. A game update or Steam "verify files" may restore the
original.

### Turning it off

Set the environment variable `ALSOFT_AUTO_REOPEN=0`, or add to `alsoft.ini`:

```ini
[general]
auto-reopen = false
```

For a debug log, set `ALSOFT_LOGLEVEL=3` and `ALSOFT_LOGFILE=C:\path\openal.log`
and look for `Auto-reopen` lines.

### The change

The patch is in `alc/alc.cpp` (worker that reopens devices via
`alcReopenDeviceSOFT`), `alc/events.cpp` (triggers on playback device
events), and `alc/device.h` (per-device state). The original OpenAL Soft
README follows.

---

OpenAL Soft
===========

`master` branch CI status : [![GitHub Actions Status](https://github.com/kcat/openal-soft/actions/workflows/ci.yml/badge.svg)](https://github.com/kcat/openal-soft/actions) [![Windows Build Status](https://ci.appveyor.com/api/projects/status/github/kcat/openal-soft?branch=master&svg=true)](https://ci.appveyor.com/api/projects/status/github/kcat/openal-soft?branch=master&svg=true)

OpenAL Soft is an LGPL-licensed, cross-platform, software implementation of the OpenAL 3D audio API. It's forked from the open-sourced Windows version available originally from openal.org's SVN repository (now defunct).
OpenAL provides capabilities for playing audio in a virtual 3D environment. Distance attenuation, doppler shift, and directional sound emitters are among the features handled by the API. More advanced effects, including air absorption, occlusion, and environmental reverb, are available through the EFX extension. It also facilitates streaming audio, multi-channel buffers, and audio capture.

More information is available on the [official website](http://openal-soft.org/).

Source Install
-------------
To install OpenAL Soft, use your favorite shell to go into the build/
directory, and run:

```bash
cmake ..
```

Alternatively, you can use any available CMake front-end, like cmake-gui,
ccmake, or your preferred IDE's CMake project parser.

Assuming configuration went well, you can then build it. The command
`cmake --build .` will instruct CMake to build the project with the toolchain
chosen during configuration (often GNU Make or NMake, although others are
possible).

Please Note: Double check that the appropriate backends were detected. Often,
complaints of no sound, crashing, and missing devices can be solved by making
sure the correct backends are being used. CMake's output will identify which
backends were enabled.

For most systems, you will likely want to make sure PipeWire, PulseAudio, and
ALSA were detected (if your target system uses them). For Windows, make sure
WASAPI was detected.


Building openal-soft - Using vcpkg
----------------------------------

You can download and install openal-soft using the [vcpkg](https://github.com/Microsoft/vcpkg) dependency manager:

    git clone https://github.com/Microsoft/vcpkg.git
    cd vcpkg
    ./bootstrap-vcpkg.sh
    ./vcpkg integrate install
    ./vcpkg install openal-soft

The openal-soft port in vcpkg is kept up to date by Microsoft team members and community contributors. If the version is out of date, please [create an issue or pull request](https://github.com/Microsoft/vcpkg) on the vcpkg repository.

Utilities
---------
The source package comes with an informational utility, openal-info, and is
built by default. It prints out information provided by the ALC and AL sub-
systems, including discovered devices, version information, and extensions.


Configuration
-------------

OpenAL Soft can be configured on a per-user and per-system basis. This allows
users and sysadmins to control information provided to applications, as well
as application-agnostic behavior of the library. See alsoftrc.sample for
available settings.


Language Bindings
-----------------

As a C API, OpenAL Soft can be used directly by any language that can use
functions with C linkage. For languages that can't directly use C-style
headers, bindings may be developed to allow code written in that language to
call into the library. Some bindings for some languages are listed here.

C# Bindings:
* [OpenTK](https://opentk.net/) includes low-level C# bindings for the OpenAL
API, including some extensions. It also includes utility libraries for math and
linear algebra, which can be useful for 3D calculations.

Java Bindings:
* [LWJGL](https://github.com/LWJGL/lwjgl3), the Lightweight Java Game Library,
includes Java bindings for the OpenAL API, usable with OpenAL Soft.
* [JOAL](https://jogamp.org/joal/www/), part of the JogAmp project, includes
Java bindings for the OpenAL API, usable with OpenAL Soft. It also includes a
higher level Sound3D Toolkit API and utility functions to make easier use of
OpenAL features and capabilities.

Kotlin Bindings:
* [Multiplatform OpenAL](https://git.karmakrafts.dev/kk/multiplatform-openal), developed for the Kleaver project,
includes Kotlin/Native bindings for the OpenAL API, based on OpenAL Soft with support
for Windows, Linux, macOS, iOS and Android.

Python Bindings:
* [PyOpenAL](https://pypi.org/project/PyOpenAL/). Also includes methods to play
wave files and, with PyOgg, also Vorbis, Opus, and FLAC.

FreePascal/Lazarus Bindings:
* [ALSound](https://github.com/Lulu04/ALSound). Also includes a higher level
API and libsndfile support to simplify loading and playing sounds.

Other bindings for these and other languages also exist. This list will grow as
more bindings are found.


Acknowledgements
----------------

Special thanks go to:

 - Creative Labs for the original source code this is based off of.
 - Christopher Fitzgerald for the current reverb effect implementation, and
helping with the low-pass and HRTF filters.
 - Christian Borss for the 3D panning code previous versions used as a base.
 - Ben Davis for the idea behind a previous version of the click-removal code.
 - Richard Furse for helping with my understanding of Ambisonics that is used by
the various parts of the library.
