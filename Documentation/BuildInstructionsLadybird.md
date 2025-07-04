# Ladybird browser build instructions

## Build Prerequisites

Qt6 development packages, nasm, additional build tools, and a C++23 capable compiler are required.

We currently use gcc-14 and clang-20 in our CI pipeline. If these versions are not available on your system, see
[`Meta/find_compiler.py`](../Meta/find_compiler.py) for the minimum compatible version.

CMake 3.25 or newer must be available in $PATH.

> [!NOTE]
> In all of the below lists of packages, the Qt6 multimedia package is not needed if your Linux system supports PulseAudio.

---

### Debian/Ubuntu:

<!-- Note: If you change something here, please also change it in the `devcontainer/devcontainer.json` file. -->
```bash
sudo apt install autoconf autoconf-archive automake build-essential ccache cmake curl fonts-liberation2 git libgl1-mesa-dev nasm ninja-build pkg-config qt6-base-dev qt6-tools-dev-tools qt6-wayland tar unzip zip
```

#### CMake 3.25 or newer:

- Recommendation: Install `CMake 3.25` or newer from [Kitware's apt repository](https://apt.kitware.com/):

> [!NOTE]
> This repository is Ubuntu-only

```bash
# Add Kitware GPG signing key
wget -O - https://apt.kitware.com/keys/kitware-archive-latest.asc 2>/dev/null | gpg --dearmor - | sudo tee /usr/share/keyrings/kitware-archive-keyring.gpg >/dev/null

# Optional: Verify the GPG key manually

# Use the key to authorize an entry for apt.kitware.com in apt sources list
echo "deb [signed-by=/usr/share/keyrings/kitware-archive-keyring.gpg] https://apt.kitware.com/ubuntu/ $(lsb_release -sc) main" | sudo tee /etc/apt/sources.list.d/kitware.list

# Update apt package list and install cmake
sudo apt update -y && sudo apt install cmake -y
```

#### C++23-capable compiler:

- Recommendation: Install clang from [LLVM's apt repository](https://apt.llvm.org/):

```bash
# Add LLVM GPG signing key
sudo wget -O /usr/share/keyrings/llvm-snapshot.gpg.key https://apt.llvm.org/llvm-snapshot.gpg.key

# Optional: Verify the GPG key manually

# Use the key to authorize an entry for apt.llvm.org in apt sources list
echo "deb [signed-by=/usr/share/keyrings/llvm-snapshot.gpg.key] https://apt.llvm.org/$(lsb_release -sc)/ llvm-toolchain-$(lsb_release -sc)-20 main" | sudo tee -a /etc/apt/sources.list.d/llvm.list

# Update apt package list and install clang and associated packages
sudo apt update -y && sudo apt install clang-20 clangd-20 clang-tools-20 clang-format-20 clang-tidy-20 lld-20 -y
```

- Alternative: Install gcc from [Ubuntu Toolchain PPA](https://launchpad.net/~ubuntu-toolchain-r/+archive/ubuntu/test):

```bash
sudo add-apt-repository ppa:ubuntu-toolchain-r/test
sudo apt update && sudo apt install g++-14 libstdc++-14-dev
```

#### Audio support:

- Recommendation: Install PulseAudio development package:

```bash
sudo apt install libpulse-dev
```

- Alternative: Install Qt6's multimedia package:

```bash
sudo apt install qt6-multimedia-dev
```

### Arch Linux/Manjaro:

```
sudo pacman -S --needed autoconf-archive automake base-devel ccache cmake curl libgl nasm ninja qt6-base qt6-multimedia qt6-tools qt6-wayland ttf-liberation tar unzip zip
```

### Fedora or derivatives:

```
sudo dnf install autoconf-archive automake ccache cmake curl liberation-sans-fonts libglvnd-devel nasm ninja-build patchelf perl-FindBin perl-IPC-Cmd perl-lib qt6-qtbase-devel qt6-qtmultimedia-devel qt6-qttools-devel qt6-qtwayland-devel tar unzip zip zlib-ng-compat-static
```

### openSUSE:

```
sudo zypper install autoconf-archive automake ccache cmake curl gcc14 gcc14-c++ liberation-fonts libglvnd-devel nasm ninja qt6-base-devel qt6-multimedia-devel qt6-tools-devel qt6-wayland-devel tar unzip zip
```
The build process requires at least python3.7; openSUSE Leap only features Python 3.6 as default, so it is recommendable to install package python311 and create a virtual environment (venv) in this case.

### Void Linux:

```
sudo xbps-install -Su # (optional) ensure packages are up to date to avoid "Transaction aborted due to unresolved dependencies."
sudo xbps-install -S git bash gcc python3 curl cmake zip unzip linux-headers make pkg-config autoconf automake autoconf-archive nasm MesaLib-devel ninja qt6-base-devel qt6-multimedia-devel qt6-tools-devel qt6-wayland-devel
```

### NixOS or with Nix:

A Nix development shell is maintained [here](https://github.com/nix-community/nix-environments/tree/master/envs/ladybird),
in the [nix-environments](https://github.com/nix-community/nix-environments/) repository. If you encounter any problems
building with Nix, please create an issue there.

### macOS:

Xcode 15 or clang from homebrew is required to successfully build ladybird.

```
xcode-select --install
brew install autoconf autoconf-archive automake ccache cmake nasm ninja pkg-config flex bison
```

If you wish to use clang from homebrew instead:
```
brew install llvm@20
```

If you also plan to use the Qt UI on macOS:
```
brew install qt
```

The following environment variables are important to ensure all tools and libraries are correctly found and used during compilation. Defining these helps avoid build errors due to mismatched or missing toolchain components, especially when using Homebrew-installed LLVM, Flex, and Bison:

```bash
export AR="/opt/homebrew/opt/llvm/bin/llvm-ar"
export CC="/opt/homebrew/opt/llvm/bin/clang"
export CXX="/opt/homebrew/opt/llvm/bin/clang++"

export CPPFLAGS="-I/opt/homebrew/opt/flex/include \
                 -I/opt/homebrew/opt/bison/include \
                 -I/opt/homebrew/opt/llvm/include"

export LDFLAGS="-L/opt/homebrew/opt/flex/lib \
                -L/opt/homebrew/opt/bison/lib \
                -L/opt/homebrew/opt/llvm/lib \
                -L/opt/homebrew/opt/llvm/lib/c++ \
                -Wl,-rpath,/opt/homebrew/opt/llvm/lib/c++"

export PATH="/opt/homebrew/opt/ccache/libexec:$PATH"
export PATH="/opt/homebrew/opt/llvm/bin:$PATH"
export PATH="/opt/homebrew/opt/bison/bin:$PATH"
export PATH="/opt/homebrew/opt/flex/bin:$PATH"
export PATH="/opt/homebrew/bin/ninja:$PATH"
```

> [!NOTE]
> It is recommended to add your terminal application (i.e. Terminal.app or iTerm.app) to the system list of developer tools.
> Doing so will reduce slow startup time of freshly compiled binaries, due to macOS validating the binary on its first run.
> This can be done in the "Developer Tools" section of the "Privacy & Security" system settings.

### Windows:

WSL2 is the supported way to build Ladybird on Windows. An experimental native build is being setup but does not fully
build.

#### WSL2
- Create a WSL2 environment using one of the Linux distros listed above. Ubuntu or Fedora is recommended.

- Install the required packages for the selected Linux distro in the WSL2 environment.

WSL1 is known to have issues. If you run into problems, please use WSL2.

MinGW/MSYS2 are not supported.

##### Clang-CL (experimental)

> [!NOTE]
> This only gets the cmake to configure. There is still a lot of work to do in terms of getting it to build.

In order to get pkg-config available for the vcpkg install, you can use Chocolatey to install it.
To install Chocolatey, see `https://chocolatey.org/install`.

Then Install pkg-config using chocolatey.
```
choco install pkgconfiglite -y
```

### Android:

> [!NOTE]
> The Android port is currently [broken](https://github.com/LadybirdBrowser/ladybird/issues/2774) and won't build.

On a Unix-like platform, install the prerequisites for that platform and then see the [Android Studio guide](EditorConfiguration/AndroidStudioConfiguration.md).
Or, download a version of Gradle >= 8.0.0, and run the ``gradlew`` program in ``UI/Android``

## Build steps

### Using ladybird.py

The simplest way to build and run ladybird is via the ladybird.py script:

```bash
# From /path/to/ladybird
./Meta/ladybird.py run
```

On macOS, to build using clang from homebrew:
```bash
CC=$(brew --prefix llvm)/bin/clang CXX=$(brew --prefix llvm)/bin/clang++ ./Meta/ladybird.py run
```

You may also choose to start it in `gdb` using:
```bash
./Meta/ladybird.py gdb ladybird
```

The above commands will build a Release version of Ladybird. To instead build a Debug version, run the
`Meta/ladybird.py` script with the value of the `BUILD_PRESET` environment variable set to `Debug`, like this:

```bash
BUILD_PRESET=Debug ./Meta/ladybird.py run
```

Note that debug symbols are available in both Release and Debug builds.

If you want to run other applications, such as the the JS REPL or the WebAssembly REPL, specify an executable with
`./Meta/ladybird.py run <executable_name>`.

### The User Interfaces

Ladybird will be built with one of the following browser frontends, depending on the platform:
* [AppKit](https://developer.apple.com/documentation/appkit?language=objc) - The native UI on macOS.
* [Qt](https://doc.qt.io/qt-6/) - The UI used on all other platforms.
* [Android UI](https://developer.android.com/develop/ui) - The native UI on Android.

The Qt UI is available on platforms where it is not the default as well (except on Android). To build the
Qt UI, install the Qt dependencies for your platform, and enable the Qt UI via CMake:

```bash
# From /path/to/ladybird
cmake --preset default -DENABLE_QT=ON
```

To re-disable the Qt UI, run the above command with `-DENABLE_QT=OFF`.

### Build error messages you may encounter

The section lists out some particular error messages you may run into, and explains how to deal with them.

#### Unable to find a build program corresponding to "Ninja"

This error message is a red herring. We use vcpkg to manage our third-party dependencies, and this error is logged when
something went wrong building those dependencies. The output in your terminal will vary depending on what exactly went
wrong, but it should look something like:

```