  # Building Gazebo Jetty + ROS 2 Integration on macOS (Apple Silicon)

  This repository contains a custom source-based build of **Gazebo Jetty** and related Ignition/GZ libraries as **submodules**, created to avoid issues caused by mismatched dependency versions in Homebrew—especially **protobuf**.

  ---
  ## ⚠️ Preliminary Instructions
  
  It is recommended to **first try the official Gazebo Jetty macOS source installation guide**:
  
  🔗 [Gazebo Jetty macOS Installation](https://gazebosim.org/docs/jetty/install_osx_src/)
  
  - Follow the guide **up to and including the “Install Dependencies” section**.
  - If the official build fails or causes issues (e.g., protobuf version conflicts on macOS/Apple Silicon), continue with this repository’s instructions **after installing the dependencies**.
  - This ensures that your system has the required dependencies while letting this repository handle patched builds and ROS 2 integration.


  ## ⚠️ Why this repository exists

  While building `gz-fuel-tools11` (and therefore the Gazebo Jetty stack), the following error appears:

      error: "Protobuf C++ gencode is built with an incompatible version of"
      error: "Protobuf C++ headers/runtime. See"
      error: "https://protobuf.dev/support/cross-version-runtime-guarantee/#cpp"

  This originates from `gz-msgs12`:

      #include "google/protobuf/runtime_version.h"
      #if PROTOBUF_VERSION != 6032001
      #error "Protobuf C++ gencode is built with an incompatible version of"
      #error "Protobuf C++ headers/runtime. See"
      #error "https://protobuf.dev/support/cross-version-runtime-guarantee/#cpp"
      #endif

  Gazebo Jetty currently expects **Protobuf 32.0** (runtime version `6032001`), but Homebrew installs **Protobuf 33.1**, which is ABI-incompatible.

  ### ❗ Result  
  `gz-msgs12`, Gazebo Jetty, and `ros_gz_bridge` fail to build.

  ---

  ## ✔️ Solution

  Instead of using the Homebrew protobuf package, uninstall or unlink it:

      brew uninstall protobuf
      # or
      brew unlink protobuf

  Then build the correct protobuf version (**32.0**) from source, which is included in this repository under:

      dependencies/protobuf

  This ensures that:

  - `gz-msgs12`
  - Gazebo Jetty dependencies
  - `ros_gz_bridge`

  are all built using the **same protobuf runtime version**.

  ---

  ## 📦 Repository Structure

  This repo includes:

  - Gazebo Jetty source submodules (`gz-math`, `gz-msgs12`, `gz-fuel-tools11`, etc.)
  - Patches for macOS builds
  - ROS 2 integration compatibility
  - Ability to build the entire stack from source without relying on Homebrew

  ---

  ## 🔧 Build Instructions

  Clone the repository with all submodules:

      git clone --recurse-submodules <this-repo>
      cd <this-repo>

Build Protobuf First:
```bash
colcon build \
    --packages-select protobuf \
    --executor parallel \
    --parallel-workers $(sysctl -n hw.ncpu) \
    --cmake-args -DBUILD_TESTING=OFF \
            -DCMAKE_BUILD_TYPE=Release \
            -DBOOST_ROOT=$(pwd)/src/dependencies/boost-1.89 \
    --merge-install \
    --continue-on-error
```
Build rest of gazebo-ionic packages:
```bash
colcon build \
    --packages-ignore protobuf \
    --executor parallel \
    --parallel-workers $(sysctl -n hw.ncpu) \
    --cmake-args \
        -DBUILD_TESTING=OFF \
        -DCMAKE_BUILD_TYPE=Release \
        -DCMAKE_MACOSX_RPATH=FALSE \
        -DCMAKE_INSTALL_NAME_DIR=$(pwd)/install/lib \
        -DCMAKE_PREFIX_PATH="$(brew --prefix qt6)" \
    --merge-install \
    --continue-on-error
```
  ### Notes
  - make sure there no trace of `protobuf` from homwbrew
  - `--merge-install` simplifies linking for Gazebo + ROS 2.
  - Do NOT mix Homebrew protobuf with source-built protobuf.

  ---

  ## 🟢 Verification

  After building:

```bash
    source install/setup.zsh
    # launch server in one terminal
    gz sim -v 4 shapes.sdf -s

    # launch gui in a separate terminal
    # remember to source the workspace setup script
    gz sim -v 4 -g
  ```

  If these run without protobuf runtime errors, the environment is correctly configured.

  ---

  ## Result

<img width="1439" height="865" alt="Screenshot 2026-03-04 at 15 07 35" src="https://github.com/user-attachments/assets/378e2c18-f95e-419c-8fd9-172a952674e9" />

