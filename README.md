# Building Gazebo Harmonic + ROS 2 Integration on macOS (Apple Silicon)

This repository contains a **custom source-based build of Gazebo Harmonic** and related Ignition/GZ libraries as **submodules**, created to avoid issues caused by mismatched dependency versions in Homebrew—especially **Boost**.

---

## ⚠️ Preliminary Instructions

It is recommended to **first try the official Gazebo Harmonic macOS source installation guide**:

🔗 [Gazebo Harmonic macOS Installation](https://gazebosim.org/docs/harmonic/install_osx_src/)

- Follow the guide **up to and including the “Install Dependencies” section**.
- If the official build fails or causes issues (e.g., protobuf version conflicts on macOS/Apple Silicon), continue with this repository’s instructions **after installing the dependencies**.
- This ensures that your system has the required dependencies while letting this repository handle patched builds and ROS 2 integration.

## ⚠️ Why this repository exists

While building Gazebo Harmonic (e.g., `gz-sim8`, `gz-fuel-tools9`, `gz-msgs10`), macOS users often encounter **Boost-related compilation errors**, because the system-installed Boost version can differ from what the stack expects.

Gazebo Harmonic requires **Boost 1.89**, which is included in this repository under:

```
dependencies/boost-1.89
```

### ❗ Result  
Without using the provided Boost, builds of:

- `gz-sim8`
- `gz-msgs10`
- `gz-fuel-tools9`
- `ros_gz_bridge`

may fail.

---

## ✔️ Solution

Instead of using the Homebrew protobuf package, uninstall or unlink it:

```bash
    brew uninstall protobuf
    # or
    brew unlink protobuf
```

Instead of using the Homebrew Boost package, use the **provided Boost 1.89** in `dependencies/boost-1.89` when building the workspace. This ensures **all packages are built with a compatible Boost version**.

---

## 📦 Repository Structure

This repo includes:

- Gazebo Harmonic source submodules (`gz-math7`, `gz-msgs10`, `gz-fuel-tools9`, etc.)  
- Boost 1.89 included in `dependencies` for macOS builds  
- ROS 2 integration compatibility  
- Ability to build the entire stack from source without relying on Homebrew

---

## 🔧 Build Instructions

Clone the repository with all submodules:

```bash
git clone --recurse-submodules https://github.com/idesign0/gz-macOS.git -b harmonic
cd <this-repo>
```

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
Build rest of gazebo-harmonic packages:
```bash
colcon build \
    --packages-ignore protobuf \
    --executor parallel \
    --parallel-workers $(sysctl -n hw.ncpu) \
    --cmake-args \
        -DBUILD_TESTING=OFF \
        -DCMAKE_BUILD_TYPE=Release \
        -DBOOST_ROOT=$(pwd)/src/dependencies/boost-1.89 \
        -DCMAKE_MACOSX_RPATH=FALSE \
        -DCMAKE_INSTALL_NAME_DIR=$(pwd)/install/lib \
    --merge-install \
    --continue-on-error
```
---

## 🟢 Verification

After building:

```bash
source install/setup.zsh

# launch server in one terminal
gz sim -v 4 shapes.sdf -s

# launch GUI in a separate terminal
# remember to source the workspace setup script
gz sim -v 4 -g
```

---

## Result

<img width="1440" height="900" alt="Screenshot 2026-02-05 at 14 32 05" src="https://github.com/user-attachments/assets/b2ff42ca-2526-4312-a561-e76012f1ccba" />
