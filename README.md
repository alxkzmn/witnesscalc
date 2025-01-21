# About This Fork

This fork turns `wintesscalc` into a template for building witness generators for arbitrary Circom circuits into native binaries for various platforms. It is supposed to be used by external code that provides the [C++ sources](https://docs.circom.io/getting-started/computing-the-witness/#computing-the-witness-with-c) and finalizes the template (see [witnesscalc_adapter](https://github.com/zkmopro/witnesscalc_adapter/tree/main) for example).

## Changes

### Templating

- The makefile expects the circuit names to be provided as a semicolon-separated list in an environment variable [`CIRCUIT_NAMES`](https://github.com/0xPolygonID/witnesscalc/compare/main...zkmopro:witnesscalc:main#diff-1e7de1ae2d059d21e1dd75d5812d5a34b0222cef273b7c3a2af62eb747f9d20aR7), e.g.,

  ```sh
  export CIRCUIT_NAMES="circuit1;circuit2;circuit3"
  ```

- The circuit files are included into the build script by [template](https://github.com/0xPolygonID/witnesscalc/compare/main...zkmopro:witnesscalc:main#diff-148715d6ea0c0ea0a346af3f6bd610d010d490eca35ac6a9b408748f7ca9e3f4R68-R91).

- The main entry point into the resulting library is [templated](https://github.com/0xPolygonID/witnesscalc/compare/main...zkmopro:witnesscalc:main#diff-8a954268d3f6fa2d327cdabe4d886538703df67c25ec7d9fe05462a042610e57) and the circuit name is supposed to be replaced by the [external code](https://github.com/zkmopro/witnesscalc_adapter/blob/main/witnesscalc_adapter/src/lib.rs#L173-L194).

- Please see [this code](https://github.com/zkmopro/witnesscalc_adapter/blob/main/witnesscalc_adapter/src/lib.rs#L156-L195) for full circuit templating process illustration.

### Compilation

- [`arm64e` target was removed](https://github.com/0xPolygonID/witnesscalc/compare/main...zkmopro:witnesscalc:main#diff-64bbdb0755df515200264726477ed4756d2ae234d63781b61403d805239cf893L225-L259) as it lead to the linking errors when building the iOS artifacts from the Rust code.

- [Added iOS simulator support](https://github.com/0xPolygonID/witnesscalc/compare/main...zkmopro:witnesscalc:main#diff-8a20b3ecd12c0610eb193f9551df0e259f1cb04be83b6cdb4e45798cb0f3155cR36-R61) following [this other fork of wintesscalc](https://github.com/0xPolygonID/witnesscalc/compare/main...ulag:witnesscalc:main). Some important differences of our solution is setting the [`CMAKE_OSX_SYSROOT`](https://github.com/0xPolygonID/witnesscalc/compare/main...zkmopro:witnesscalc:main#diff-8a20b3ecd12c0610eb193f9551df0e259f1cb04be83b6cdb4e45798cb0f3155cR39) and [forcing the `CMAKE_OSX_ARCHITECTURES`](https://github.com/0xPolygonID/witnesscalc/compare/main...zkmopro:witnesscalc:main#diff-8a20b3ecd12c0610eb193f9551df0e259f1cb04be83b6cdb4e45798cb0f3155cR41) to stay the same throughout the build process (otherwise some of the dependencies would be incorrectly built for the host architecture).

### Installation

- All the targets have [the same CMAKE_INSTALL_PREFIX of `/package`](https://github.com/0xPolygonID/witnesscalc/compare/main...zkmopro:witnesscalc:main#diff-76ed074a9305c04054cdebb9e9aad2d818052b07091de1f20cad0bbac34ffb52R16) because this fork is meant to be used as a part of the Rust library build process, and the build code expects the same output paths for all the targets. The external code is responsible for later combining the outputs into the final library artifacts.
- The [`-GXcode`](https://github.com/0xPolygonID/witnesscalc/compare/main...zkmopro:witnesscalc:main#diff-76ed074a9305c04054cdebb9e9aad2d818052b07091de1f20cad0bbac34ffb52L26) flag was removed because the Rust code needs to link to the binary and the platform-specific artifacts are later generated from the Rust library.

# Original Readme

## Dependencies

You should have installed gcc and cmake

In ubuntu:

```sh
sudo apt install build-essential cmake m4
```

## Compilation

### Preparation

```sh
git submodule init
git submodule update
```

### Compile witnesscalc for x86_64 host machine

```sh
./build_gmp.sh host
make host
```

### Compile witnesscalc for arm64 host machine

```sh
./build_gmp.sh host
make arm64_host
```

### Compile witnesscalc for Android

Install Android NDK from https://developer.android.com/ndk or with help of "SDK Manager" in Android Studio.

Set the value of ANDROID_NDK environment variable to the absolute path of Android NDK root directory.

Examples:

```sh
export ANDROID_NDK=/home/test/Android/Sdk/ndk/23.1.7779620  # NDK is installed by "SDK Manager" in Android Studio.
export ANDROID_NDK=/home/test/android-ndk-r23b              # NDK is installed as a stand-alone package.
```

Compilation for arm64:

```sh
./build_gmp.sh android
make android
```

Compilation for x86_64:

```sh
./build_gmp.sh android_x86_64
make android_x86_64
```

### Compile witnesscalc for iOS

Requirements: Xcode.

1. Run:
   ```sh
   ./build_gmp.sh ios
   make ios
   ```
2. Open generated Xcode project.
3. Add compilation flag `-D_LONG_LONG_LIMB` to all build targets.
4. Add compilation flag `-DCIRCUIT_NAME=auth`, `-DCIRCUIT_NAME=sig` and `-DCIRCUIT_NAME=mtp` to the respective targets.
5. Compile witnesscalc.

## Updating circuits

1. Compile a circuit with compile-circuit.sh script in circuits repo as usual.
2. Replace existing <circuitname>.cpp and <circuitname>.dat files with generated ones (e.g. auth.cpp & auth.dat).
3. Enclose all the code inside <circuitname>.cpp file with `namespace CIRCUIT_NAME` (do not replace `CIRCUIT_NAME` with the real name, it will be replaced at compilation), like this:

   ```c++
   #include ...
   #include ...

   namespace CIRCUIT_NAME {

   // millions of code lines here

   } // namespace

   ```

4. Optional. Remove the `#include <assert.h>` line and replace all accurances of `assert(` with `check(` in .cpp file to switch from asserts to exceptions (more secure).

Alternatively you can patch the `.cpp` circuit file with a `patch_cpp.sh` script. Example how to run it:

```shell
./patch_cpp.sh ../circuits/build/linkedMultiQuery10/linkedMultiQuery10_cpp/linkedMultiQuery10.cpp > ./src/linkedMultiQuery10.cpp
```

## License

witnesscalc is part of the iden3 project copyright 2022 0KIMS association and published with GPL-3 license. Please check the COPYING file for more details.
