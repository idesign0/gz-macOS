  # Building Gazebo Ionic + ROS 2 Integration on macOS (Apple Silicon)

  This repository contains a custom source-based build of **Gazebo Ionic** and related Ignition/GZ libraries as **submodules**, created to avoid issues caused by mismatched dependency versions in Homebrew—especially **protobuf**.

  ---

  ## ⚠️ Why this repository exists

  While building `gz-fuel-tools10` (and therefore the Gazebo Ionic stack), the following error appears:

      error: "Protobuf C++ gencode is built with an incompatible version of"
      error: "Protobuf C++ headers/runtime. See"
      error: "https://protobuf.dev/support/cross-version-runtime-guarantee/#cpp"

  This originates from `gz-msgs11`:

      #include "google/protobuf/runtime_version.h"
      #if PROTOBUF_VERSION != 6032001
      #error "Protobuf C++ gencode is built with an incompatible version of"
      #error "Protobuf C++ headers/runtime. See"
      #error "https://protobuf.dev/support/cross-version-runtime-guarantee/#cpp"
      #endif

  Gazebo Ionic currently expects **Protobuf 32.1** (runtime version `6032001`), but Homebrew installs **Protobuf 33.1**, which is ABI-incompatible.

  ### ❗ Result  
  `gz-msgs11`, Gazebo Ionic, and `ros_gz_bridge` fail to build.

  ---

  ## ✔️ Solution

  Instead of using the Homebrew protobuf package, uninstall or unlink it:

      brew uninstall protobuf
      # or
      brew unlink protobuf

  Then build the correct protobuf version (**32.1**) from source, which is included in this repository under:

      dependencies/protobuf

  This ensures that:

  - `gz-msgs11`
  - Gazebo Ionic dependencies
  - `ros_gz_bridge`

  are all built using the **same protobuf runtime version**.

  ---

  ## 📦 Repository Structure

  This repo includes:

  - Gazebo Ionic source submodules (`gz-math`, `gz-msgs11`, `gz-fuel-tools10`, etc.)
  - Patches for macOS builds
  - ROS 2 integration compatibility
  - Ability to build the entire stack from source without relying on Homebrew

  ---

  ## 🔧 Build Instructions

  Clone the repository with all submodules:

      git clone --recurse-submodules <this-repo>
      cd <this-repo>

  Build using **colcon**:

      colcon build --merge-install \
        ---executor parallel \
        --parallel-workers $(sysctl -n hw.ncpu) \
        --cmake-args \
          -DBUILD_TESTING=OFF \
          -DCMAKE_MACOSX_RPATH=FALSE \
          -DCMAKE_INSTALL_NAME_DIR=$(pwd)/install/lib \
          -DCMAKE_BUILD_TYPE=Release

  ### Notes

  - `-DCMAKE_MACOSX_RPATH=FALSE` avoids incorrect RPATH rewriting on macOS.
  - `--merge-install` simplifies linking for Gazebo + ROS 2.
  - Do NOT mix Homebrew protobuf with source-built protobuf.

  ---

  ## 🟢 Verification

  After building:

      source install/setup.zsh
      gz fuel --help
      gz topic --help
      ros2 run ros_gz_bridge parameter_bridge

  If these run without protobuf runtime errors, the environment is correctly configured.

  ---

  ## Result

  <img width="1440" height="900" alt="Screenshot 2025-12-08 at 18 27 10" src="https://github.com/user-attachments/assets/86688ca6-475f-4d57-a2dc-e895e1385ca2" />
