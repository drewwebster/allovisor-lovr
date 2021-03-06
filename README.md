# Alloverse visor (Lua edition)

Allovisor is the "user interface" into the Alloverse. Use this Lovr app to connect to an Alloverse Place' and interact with the apps available in that place.

We [first tried to build the Alloverse visor](https://github.com/alloverse/allovisor)
in Unity, but have decided to switch to Lovr because it's easier and faster to develop
with, and easier to extend with low level functionality.

## Developing Allovisor

### Mac

1. Install CMake 3.13.0 or newer
2. `mkdir build && cd build && cmake ..` to prepare to build
3. In build, `make alloverse-dist` to build `Alloverse.app`.
4. You could now just double-click Alloverse.app, but then you'd need to recompile
   for each change. Instead, you can start it from the command line together with
   lodr to auto-reload whenever you change any lua source file. From `build`:

`./Alloverse.app/Contents/MacOS/lovr ../deps/lodr ../lua`

You could also `make allonet` and use the regular Lovr visor like below. (It's not
recommended, as you wouldn't get any Alloverse-specific Lovr app patches, and things
might not work as expected.) cpath is set up to find
`liballonet.so` in `build` next to `lua`. Run it from the project root like so:

`/Applications/LÖVR.app/Contents/MacOS/lovr deps/lodr lua`

### Windows

1. Install Visual Studio 2019, including "C++ CMake tools for Windows" which comes with "Desktop development with C++".
2. Open the project folder in VS2019
3. In the Solution Explorer, right-click CMakeLists.txt and select "Generate CMake cache for allovisor"
4. Switch to the [targets view](https://docs.microsoft.com/en-us/cpp/build/cmake-projects-in-visual-studio?view=vs-2019#ide-integration)
5. Build the target `alloverse-dist`.

To iterate on the lua stuff, from `out/build/x64-debug/Alloverse`, you can run:

`Alloverse.exe ..\..\..\..\deps\lodr ..\..\..\..\lua`

If you want to run from the VS2019 debugger to debug C stuff:

1. choose lovr.exe as the startup item
2. copy `allonet.dll` from `out/build/x64-Debug/deps/allonet` to `out/build/x64-Debug/deps/lovr/liballonet.dll`
3. Run in Visual Studio. Copy liballonet again if you make changes.

Not great but _shrug_.

### Oculus Quest, with hard-linked allonet, from a Mac

1. Install CMake version 3.15.4 exactly
2. Install Android Studio if you haven't already.
3. [Enable developer mode on your Quest](https://developer.oculus.com/documentation/quest/latest/concepts/mobile-device-setup-quest/).
4. Connect it to your computer, and ensure it shows up when you run `adb devices` in your terminal.
5. Configure to build the alloverse.apk: `mkdir android-build; cd android-build; cmake -DCMAKE_TOOLCHAIN_FILE=~/Library/Android/sdk/ndk-bundle/build/cmake/android.toolchain.cmake -DANDROID_ABI=arm64-v8a ..`
6. Actually build it: `make alloverse-dist`
7. Upload to headset: `adb install alloverse-debug.apk`

If you are iterating on the native code parts, you can re-build and upload the api with this handy one-liner
from the `build` directory:

`cmake -DCMAKE_TOOLCHAIN_FILE=~/Library/Android/sdk/ndk-bundle/build/cmake/android.toolchain.cmake -DANDROID_ABI=arm64-v8a ..; and rm -rf *.apk; and make alloverse-dist; and adb uninstall com.alloverse.visor; and adb install alloverse-release.apk`

Note that this command deletes apks on disk (because the cmake integration is iffy and it doesn't know
to rebuild unless the apk is missing), and deletes from device (because I haven't worked out keychain yet
so each build gets a new signature).

If you are iterating on the lua code parts, it would be nice to upload just the lua files and
lodr could override the bundled sources to give the changes to you immediately, without even
having to restart the app on your Quest. If that had worked, you'd sync your source files like so:

`adb push --sync lua /sdcard/Android/data/com.alloverse.visor/files/.lodr`

... but that's waiting [for a card on clubhouse](https://app.clubhouse.io/alloverse/story/168/get-lodr-to-work-on-android-for-custom-alloverse-debug-apk)
to finish before it's possible.

## Building Allovisor for distribution

_Note that builds are available on Azure Pipelines CI and you shouldn't need to make distribution builds from your machine. But if you do..._

This will compile Lovr, Allonet and build a nice little package to be distributed
for Mac, Windows or Android.

1. Install CMake 3.13.0 or newer
2. `mkdir build && cd build && cmake ..`

### Mac

- `make alloverse-dist`
- You now have an `Alloverse.app` in the build folder.

### Windows

1. Install Visual Studio 2019, including "C++ CMake tools for Windows" which comes with "Desktop development with C++".
2. Open the project folder in VS2019
3. In the Solution Explorer, right-click CMakeLists.txt and select "Generate CMake cache for allovisor"
4. Switch to the [targets view](https://docs.microsoft.com/en-us/cpp/build/cmake-projects-in-visual-studio?view=vs-2019#ide-integration)
5. Build the target `alloverse-dist`.

Tada. You now have an `Alloverse` folder and zip file in `out/build/x64-Debug`.

## Quest/Android (from a Mac)

1. `mkdir build && cd build && cmake -DCMAKE_TOOLCHAIN_FILE=~/Library/Android/sdk/ndk-bundle/build/cmake/android.toolchain.cmake -DANDROID_ABI=arm64-v8a ..`
2. `make alloverse-dist`
3. You now have an `alloverse-debug.apk` and `alloverse-release.apk` in your build folder
