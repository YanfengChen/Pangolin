# Pangolin Installation Guide

Complete installation guide for Pangolin on Ubuntu, following the official README.md instructions.

## System Information

- **OS**: Ubuntu 24.04.3 LTS (Noble Numbat)
- **CMake Version**: 3.28.3
- **Python Version**: 3.13.9 (Miniconda)
- **Installation Date**: December 10, 2025

---

## Prerequisites

Before installing Pangolin, ensure you have:
- Ubuntu 24.04 or compatible Linux distribution
- Git installed
- Internet connection for downloading dependencies
- Sudo privileges for installing system packages

---

## Step 1: Clone Repository and Initialize Submodules

```bash
cd ~/your_fav_code_directory
git clone --recursive https://github.com/stevenlovegrove/Pangolin.git
cd Pangolin
```

**Important**: The `--recursive` flag is required to clone the git submodules (pybind11 and vcpkg).

If you already cloned without `--recursive`, initialize submodules:
```bash
git submodule update --init --recursive
```

**Verification**:
```bash
git submodule status
```
You should see:
- `components/pango_python/pybind11`
- `scripts/vcpkg`

---

## Step 2: Install System Dependencies

Pangolin provides a script to automatically install prerequisites based on your package manager.

### Check what will be installed (dry-run):
```bash
./scripts/install_prerequisites.sh --dry-run recommended
```

### Install recommended dependencies:
```bash
./scripts/install_prerequisites.sh recommended
```

This will install:

**Required packages:**
- `libgl1-mesa-dev` - OpenGL development files
- `libwayland-dev` - Wayland display protocol
- `libxkbcommon-dev` - Keyboard handling
- `wayland-protocols` - Wayland protocol extensions
- `libegl1-mesa-dev` - EGL (OpenGL ES) support
- `libc++-dev` - C++ standard library
- `libepoxy-dev` - OpenGL function pointer management
- `libglew-dev` - OpenGL Extension Wrangler
- `libeigen3-dev` - Linear algebra library
- `cmake` - Build system
- `g++` - C++ compiler
- `ninja-build` - Fast build tool

**Recommended packages:**
- `libjpeg-dev` - JPEG image support
- `libpng-dev` - PNG image support
- `catch2` - Testing framework
- `libavcodec-dev` - Video codec support (FFmpeg)
- `libavutil-dev` - FFmpeg utilities
- `libavformat-dev` - Container format support
- `libswscale-dev` - Video scaling/conversion
- `libavdevice-dev` - Device input/output

**Optional packages** (use `all` instead of `recommended`):
- `libdc1394-dev` - FireWire camera support
- `libraw1394-dev` - Raw FireWire access
- `libopenni-dev` - Kinect/depth sensor support
- `python3-dev` - Python development headers

### Dependency Tiers:
- `required` - Minimum dependencies for basic build
- `recommended` - Required + commonly used features (default)
- `all` - Everything including optional camera/device drivers

---

## Step 3: Configure Build with CMake

Create a build directory and configure the project:

```bash
cmake -B build -GNinja
```

**Explanation:**
- `-B build` - Creates and uses `build/` directory for all build files (keeps source clean)
- `-GNinja` - Uses Ninja build system (faster than Make)

**Alternative without Ninja:**
```bash
cmake -B build
```

### What CMake Does:
1. Detects available dependencies on your system
2. Configures which Pangolin components to build
3. Finds Python interpreter and libraries
4. Generates build files

### Check the Output:

Look for these important messages:
```
-- Selected Python: '/home/yanfeng/miniconda/bin/python3'
-- libpng Found and Enabled
-- libjpeg Found and Enabled
-- Found Eigen: '/usr/include/eigen3'
-- Found OpenGL: /usr/lib/x86_64-linux-gnu/libOpenGL.so
-- V4L Found and Enabled
-- ffmpeg Found and Enabled
```

**Note**: If Python is not found or you want to use a specific Python version:
```bash
cmake -B build -GNinja -DPython3_EXECUTABLE=/path/to/python
```

Or for conda environments:
```bash
cmake -B build -GNinja -DPython3_EXECUTABLE=$(which python3)
```

---

## Step 4: Build Pangolin

Compile all components:

```bash
cmake --build build
```

This command:
- Compiles 233 C++ source files
- Builds the core Pangolin library (`libpangolin.so`)
- Compiles all example programs
- Compiles all tools (VideoViewer, ModelViewer, Plotter, etc.)
- Builds Python bindings if Python was detected

**Build Time**: Approximately 2-5 minutes depending on your CPU

### Alternative with Explicit Jobs:
```bash
cmake --build build -j$(nproc)
```

### Expected Output:
```
[233/233] Linking CXX shared module ...pypangolin.cpython-313-x86_64-linux-gnu.so
```

**Warnings about "may be used uninitialized"** in Eigen/pybind11 code are normal and harmless.

---

## Step 5: Install Python Bindings (Optional)

If you want to use Pangolin from Python, install the pypangolin module:

```bash
cmake --build build -t pypangolin_pip_install
```

This will:
1. Create a Python wheel (`.whl` file)
2. Install it via pip to your active Python environment

**Important**: The Python bindings are compiled specifically for the Python version detected by CMake (3.13.9 in this case).

### Verify Installation:
```bash
python -c 'import pypangolin; print("pypangolin imported successfully!")'
```

### Using Different Python Versions:

To build for a different Python version, you must reconfigure and rebuild:

```bash
# Activate target environment
conda activate my_other_env

# Reconfigure for this Python
cmake -B build-py312 -GNinja

# Build and install
cmake --build build-py312
cmake --build build-py312 -t pypangolin_pip_install
```

---

## Step 6: Verify Installation

### Check Built Components:

```bash
# List example programs
ls build/examples/*/

# List tools
ls build/tools/*/

# Check library
ls -lh build/src/libpangolin.so
```

### Installation Summary:

**Built Components:**
- ✓ Core Pangolin library (`libpangolin.so`)
- ✓ Python bindings (`pypangolin`) with 144 attributes
- ✓ 17 example programs
- ✓ 6 tools

**Example Programs:**
- `HelloPangolin` - Basic windowing example
- `HelloPangolinOffscreen` - Offscreen rendering
- `SimpleDisplay` - 3D graphics display
- `SimpleDisplayImage` - Image display
- `SimpleMultiDisplay` - Multiple viewports
- `SimplePlot` - Data plotting
- `SimpleRecord` - Record video
- `SimpleScene` - 3D scene rendering
- `SimpleVideo` - Video playback
- `VBODisplay` - Vertex Buffer Objects
- OpenGL tutorials (BasicOpenGL examples)

**Tools:**
- `VideoViewer` - View videos and camera streams
- `ModelViewer` - View 3D models
- `Plotter` - Data plotting tool
- `VideoConvert` - Convert video formats
- `VideoJsonPrint` - Print video metadata
- `VideoJsonTransform` - Transform video streams

---

## Running Examples

### Run C++ Examples:

```bash
# Basic windowing
./build/examples/HelloPangolin/HelloPangolin

# Simple 3D display
./build/examples/SimpleDisplay/SimpleDisplay

# Plot data
./build/examples/SimplePlot/SimplePlot

# Video viewer
./build/tools/VideoViewer/VideoViewer test://
```

### Run Python Examples:

```bash
cd examples/PythonExamples
python HelloPangolin.py
python SimpleDisplay.py
python SimplePlot.py
```

---

## Using Pangolin in Your Projects

### C++ Projects:

Add to your `CMakeLists.txt`:

```cmake
find_package(Pangolin REQUIRED)

add_executable(myapp main.cpp)
target_link_libraries(myapp PRIVATE pango_display pango_opengl)
```

Set Pangolin path when configuring:
```bash
cmake -B build -DCMAKE_PREFIX_PATH=/home/yanfeng/Pangolin/build
```

### Python Projects:

```python
import pypangolin as pango

# Create window
win = pango.CreateWindowAndBind("My Window", 640, 480)

# Use Pangolin features
# ... your code ...
```

---

## Troubleshooting

### Problem: CMake doesn't find Python

**Solution**: Explicitly specify Python executable:
```bash
cmake -B build -GNinja -DPython3_EXECUTABLE=$(which python3)
```

### Problem: "ModuleNotFoundError: No module named 'pypangolin'"

**Causes**:
1. Python bindings not installed: Run `cmake --build build -t pypangolin_pip_install`
2. Using different Python than build: Use the same Python that CMake detected
3. Wrong Python version: pypangolin must match Python version used during build

**Check which Python was used**:
```bash
grep "Selected Python" build/CMakeCache.txt
```

### Problem: Build fails with "Framebuffer with requested attributes not available"

This is a **runtime** error, not build error. Causes:
- No display available (SSH without X forwarding, Docker, VM)
- Old/missing graphics drivers
- Running in headless environment

**Solutions**:
- Use offscreen examples: `HelloPangolinOffscreen`
- Set window URI: `PANGOLIN_WINDOW_URI="headless://" ./myapp`
- Fix graphics drivers: `sudo ubuntu-drivers autoinstall`

### Problem: Compiler warnings about "may be used uninitialized"

These warnings in Eigen/pybind11 code are **normal and harmless**. The build succeeded.

### Problem: "catch2" package not available

On older Ubuntu versions, catch2 might not be in repos. Either:
- Skip tests: They're optional
- Manually install Catch2 from source
- Use `required` instead of `recommended` tier

---

## Build Options

### Custom Build Directory:

```bash
cmake -B mybuild -GNinja
cmake --build mybuild
```

### Build Type:

```bash
# Release build (default, optimized)
cmake -B build -GNinja -DCMAKE_BUILD_TYPE=Release

# Debug build (with debug symbols)
cmake -B build-debug -GNinja -DCMAKE_BUILD_TYPE=Debug
```

### Enable Tests:

```bash
cmake -B build -GNinja -DBUILD_TESTS=ON
cmake --build build
cd build
ctest
```

### Disable Optional Components:

```bash
# Disable Python bindings
cmake -B build -GNinja -DBUILD_PANGOLIN_PYTHON=OFF

# Disable video support
cmake -B build -GNinja -DBUILD_PANGOLIN_VIDEO=OFF
```

---

## Clean Build

To start fresh:

```bash
# Remove build directory
rm -rf build

# Reconfigure and rebuild
cmake -B build -GNinja
cmake --build build
```

---

## Environment Variables

### PANGOLIN_WINDOW_URI

Override default windowing options:

```bash
# Use X11 with specific options
PANGOLIN_WINDOW_URI="x11:[double_buffered=false]//" ./myapp

# Headless rendering
PANGOLIN_WINDOW_URI="headless://" ./myapp

# Custom font
PANGOLIN_WINDOW_URI="default:[default_font_size=20]//" ./myapp
```

---

## Additional Resources

- **Official Repository**: https://github.com/stevenlovegrove/Pangolin
- **Online Examples**: https://stevenlovegrove.github.io/Pangolin/examples
- **Issues**: https://github.com/stevenlovegrove/Pangolin/issues
- **Documentation**: See README.md and examples/ folder

---

## Installation Complete!

Your Pangolin installation is ready to use. You can now:
- Run example programs from `build/examples/`
- Use tools from `build/tools/`
- Import pypangolin in Python scripts
- Build your own projects using Pangolin

For questions or issues, refer to the [GitHub Issues](https://github.com/stevenlovegrove/Pangolin/issues) page.
