OpenAL for X4: Foundations with Bluetooth auto-reconnect
=========================================================

A replacement `OpenAL32.dll` for **X4: Foundations only**. With the original DLL,
when Bluetooth headphones disconnect and reconnect, or the Windows default
output changes, X4 goes silent until it's restarted. With this DLL, the audio
continues automatically through the new output device.

It is modeled on **Egosoft's custom OpenAL DLL** that ships with X4. Egosoft's
version is a modified OpenAL Soft 1.19.1 with its own extensions for how sounds
fade with distance, which X4 relies on. This build is
[OpenAL Soft](https://github.com/kcat/openal-soft) 1.24.3 with those Egosoft
extensions recreated, plus the auto-reconnect behaviour. It is not meant for
other games.

The patch was written with Claude (Anthropic's AI) and tested by
[Kaiji-Kun](https://github.com/Kaiji-Kun). OpenAL Soft itself is by Chris
Robinson and contributors; the original X4 audio extensions are Egosoft's.

**Download:** `X4-OpenAL-autoreconnect.zip` on the [Releases](../../releases)
page.

### Installing

1. In your X4 folder (e.g. `steamapps\common\X4 Foundations`), rename
   `OpenAL32.dll` to `OpenAL32.dll.orig`.
2. Copy `x64\OpenAL32.dll` from the zip into the X4 folder.
3. In X4's sound options, set the audio device to "system default" for the
   smoothest switching.

To undo, delete the new `OpenAL32.dll` and rename `OpenAL32.dll.orig` back. A
game update or Steam "verify files" may restore the original DLL, in which
case copy the new one in again.

### How it behaves

- When the output device disappears or the Windows default output changes,
  X4's audio moves to the current Windows default output about a second later.
- Sounds pause while there is no output and resume on the new device.
- For 30 seconds after a change it keeps checking that audio is actually
  playing on the current default, which covers Bluetooth devices that take a
  few seconds to become ready.
- X4's own audio settings are kept.

### Turning it off

Set the environment variable `ALSOFT_AUTO_REOPEN=0`, or create `alsoft.ini` in
the X4 folder containing:

```ini
[general]
auto-reopen = false
```

For a debug log, set `ALSOFT_LOGLEVEL=3` and `ALSOFT_LOGFILE=C:\path\openal.log`
and look for `Auto-reopen` lines.

### The changes

- **Egosoft compatibility** (`alc/alu.cpp`, `al/source.cpp`, `al/state.cpp`,
  `core/context.h`, `core/voice.h`, `al/source.h`): distance model `0xD007`,
  where each sound's volume comes from a 4-point distance curve (linear between
  points, held at the ends), and source property `0xD008` (index, distance,
  gain) to set the curve's points, matching Egosoft's DLL.
- **Auto-reconnect** (`alc/alc.cpp`, `alc/events.cpp`, `alc/device.h`): a
  worker that moves playback devices onto the Windows default via
  `alcReopenDeviceSOFT` after device events.

The original OpenAL Soft README follows.

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
