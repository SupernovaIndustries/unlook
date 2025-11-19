# UNLOOK 3D SCANNER

<p align="center">
  <a href="https://supernovaindustries.it">
    <img src="http://supernovaindustries.it/wp-content/uploads/2024/08/supernova-industries-logo.svg" alt="Supernova Industries Logo" width="200" />
  </a>
</p>

<p align="center">
  <img src="http://supernovaindustries.it/wp-content/uploads/2024/08/logo_full-white-unlook-1.svg" alt="Unlook Logo" width="300" />
</p>

<p align="center">
  <strong>Professional Modular Open-Source 3D Scanner</strong><br>
  <strong>0.1mm Precision • Industrial Grade • Completely Open Source</strong>
</p>

<p align="center">
  <a href="https://github.com/SupernovaIndustries/unlook/blob/main/LICENSE">
    <img src="https://img.shields.io/badge/License-MIT-blue.svg" alt="License: MIT">
  </a>
  <a href="https://www.raspberrypi.com/">
    <img src="https://img.shields.io/badge/Platform-Raspberry%20Pi%205-green.svg" alt="Platform">
  </a>
  <a href="https://isocpp.org/">
    <img src="https://img.shields.io/badge/C%2B%2B-17%2F20-blue.svg" alt="C++17/20">
  </a>
</p>

<p align="center">
  Made with ❤️ by <a href="https://supernovaindustries.it">Supernova Industries</a>
</p>

<p align="center">
  <a href="https://www.supernovaindustries.it/">Website</a> •
  <a href="https://github.com/SupernovaIndustries">GitHub</a> •
  <a href="#overview">Overview</a> •
  <a href="#features">Features</a> •
  <a href="#architecture">Architecture</a> •
  <a href="#installation">Installation</a> •
  <a href="#usage">Usage</a> •
  <a href="#roadmap">Roadmap</a>
</p>

---

## 🎯 Overview

**Unlook** is a professional-grade, modular, open-source 3D scanner designed for industrial applications requiring **0.1mm precision**. Built on Raspberry Pi 5 (CM5) hardware, it combines advanced stereo vision algorithms with VCSEL-based structured light for high-quality 3D reconstruction.

### **What Makes Unlook Different**

Unlike proprietary solutions, Unlook is built on open principles:

✅ **Modular & Expandable**: Designed for multiple scanning technologies  
✅ **Open-Source**: Complete software and hardware designs available  
✅ **Affordable**: Professional-grade precision at accessible costs  
✅ **Future-Proof**: Architecture ready for upcoming scanning technologies  
✅ **Industrial-Grade**: Professional C++ codebase, zero Python dependencies

### **Target Applications**
- 🏭 **Industrial Quality Control** and inspection  
- 🎓 **Educational 3D Scanning** projects and research
- ⚙️ **Professional Prototyping** and reverse engineering
- 🔬 **Research Applications** requiring high-precision measurements
- 🎨 **Cultural Heritage** documentation and preservation

### **Technical Specifications**
- 🎯 **Precision**: **0.1mm repeatability** for industrial metrology
- ⚡ **Real-time Performance**: ~10 seconds total processing (2s capture + 8s processing)
- 🏗️ **Architecture**: Modular with interchangeable scanning modules
- 📄 **License**: **MIT** - Completely open source
- 💻 **Language**: **C++17/20** exclusively (zero Python in production)
- 🚀 **Performance**: VGA stereo processing >20 FPS on CM5
- 🔧 **Deployment**: Self-contained system on Raspberry Pi CM5

---

## ⭐ Key Features

### **⚡ High-Performance Processing**
- **C++17/20 Exclusively** - Zero Python dependencies for maximum reliability
- **Thread-safe Architecture** with professional C++ patterns (RAII, smart pointers)
- **OpenMP Multi-threading**: 4-core parallelization for SGM-Census stereo
- **SGM-Census Stereo Matching**: 9×9 window, 8-path aggregation, subpixel precision
- **Advanced Algorithms**: OpenCV SGBM with future BoofCV integration
- **Memory-optimized** for embedded systems (works on 8GB, optimal on 16GB)

### **🖥️ Professional Touch Interface**
- **Industrial Touch Design** with Supernova-tech styling
- **Foolproof Operation** - 2-minute operator mastery
- **Real-time Controls** for all stereo parameters
- **Live Dual Camera Preview** with sync status indicators
- **Touch-optimized** with 48px minimum touch targets
- **ESC Key**: Toggle fullscreen/windowed mode anytime

### **🔬 Complete Development Stack**
- **Single Command Build** - `./build.sh` does everything
- **Cross-compilation Support** for ARM64/Raspberry Pi
- **Comprehensive Testing** framework with Google Test
- **Professional Documentation** with API examples
- **GitHub Integration** ready for CI/CD

### **🔧 Modular & Expandable Architecture**

**Current Scanning Module:**
- **Stereo Vision**: Dual synchronized global shutter cameras
- **VCSEL Structured Light**: Dot pattern projection for texture enhancement
- **Hardware Sync**: <1ms precision camera synchronization

**Coming Soon - Additional Scanning Modules:**
- 🔲 **MEMS Structured Light**: High-resolution line/pattern scanning
- 📡 **LiDAR Distance Measurement**: Long-range precision scanning
- 🌈 **Multi-wavelength VCSEL**: Enhanced pattern recognition
- ⚡ **Active Illumination Control**: Adaptive lighting systems

All modules use the same base platform and share the common API - just plug in different scanning modules for different applications.

---

## 🏗️ Software Architecture

### **Professional C++ Design**
The scanner operates in two modes using the **same** underlying API:
- **Standalone Mode**: Direct GUI operation on Raspberry Pi
- **Companion Mode**: External PC control via shared library API

```cpp
#include <unlook/unlook.h>

// Initialize scanner
unlook::api::UnlookScanner scanner;
scanner.initialize(unlook::core::ScannerMode::STANDALONE);

// Load calibration
scanner.loadCalibration("/unlook_calib/default.yaml");

// Capture and process
auto frames = scanner.captureFrames(10);
auto pointCloud = scanner.processToPointCloud(frames);

// Export result
scanner.exportPLY("output.ply", pointCloud);
```

### **Namespace Structure**
```cpp
namespace unlook {
    namespace core {          // Logging, configuration, exceptions
    namespace api {           // Public API (UnlookScanner, CameraSystem)
    namespace camera {        // Hardware sync, camera control, auto-exposure
    namespace stereo {        // SGM-Census stereo matching
    namespace calibration {   // Calibration loading and validation
    namespace hardware {      // I2C (AS1170), GPIO, LED control
    namespace pointcloud {    // Point cloud processing (Open3D)
    namespace mesh {          // Mesh generation and optimization
    namespace gui {           // Qt5 touch interface
}
```

### **Technology Stack**
- **Language**: C++17/20 exclusively
- **Build System**: CMake 3.16+ with cross-compilation support
- **Dependencies**:
  - OpenCV 4.6+ (stereo, calibration, image processing)
  - Qt5 5.15+ (GUI framework)
  - Open3D 0.16+ (point cloud and mesh processing)
  - libcamera-sync (custom hardware synchronization)
- **Optimization**: OpenMP 4.5+ for multi-core parallelization

---

## 🚀 Installation

### **Prerequisites**

```bash
# Install system dependencies (Debian/Ubuntu)
sudo apt-get update
sudo apt-get install -y \
    build-essential cmake pkg-config \
    libopencv-dev qt5-default \
    libomp-dev libopen3d-dev \
    libnlohmann-json3-dev
```

### **Build from Source**

```bash
# Clone repository
git clone https://github.com/SupernovaIndustries/unlook.git
cd unlook

# Build (automatic dependency detection)
./build.sh

# Cross-compile for Raspberry Pi 5
./build.sh --cross rpi5 -j 4
```

### **System Installation**

After building, install system-wide:

```bash
cd build
sudo make install

# Run from anywhere
unlook
```

---

## 💻 Usage

### **GUI Application**

```bash
# Run scanner GUI (system-installed)
unlook

# Or run from build directory
LD_LIBRARY_PATH=build/src:third-party/libcamera-sync-fix/build/src/libcamera:third-party/libcamera-sync-fix/build/src/libcamera/base:$LD_LIBRARY_PATH \
./build/src/gui/unlook_scanner

# Debug mode with verbose output
./build/bin/unlook_scanner --debug

# Mock mode (no hardware required)
./build/bin/unlook_scanner --mock
```

### **Interface Navigation**
1. **Main Menu**: Choose Camera Preview, Depth Test, or Options
2. **Camera Preview**: Live dual camera feeds with exposure/gain controls
3. **Depth Test**: Capture stereo pairs and process depth maps
4. **Options**: System status, calibration info, hardware diagnostics
5. **ESC Key**: Toggle fullscreen/windowed mode anytime

### **API Example - Basic Stereo Capture**

```cpp
#include <unlook/unlook.h>

int main() {
    // Initialize scanner in standalone mode
    unlook::api::UnlookScanner scanner;
    scanner.initialize(unlook::core::ScannerMode::STANDALONE);
    
    // Get camera system
    auto* camera = scanner.getCameraSystem();
    
    // Capture single stereo pair
    cv::Mat left_image, right_image;
    bool success = camera->captureSingleFrame(left_image, right_image);
    
    if (success) {
        cv::imwrite("stereo_left.png", left_image);
        cv::imwrite("stereo_right.png", right_image);
        std::cout << "Stereo pair captured successfully!" << std::endl;
    }
    
    return 0;
}
```

### **API Example - Depth Processing**

```cpp
#include <unlook/unlook.h>

int main() {
    // Initialize scanner
    unlook::api::UnlookScanner scanner;
    scanner.initialize(unlook::core::ScannerMode::STANDALONE);
    
    // Get depth processor
    auto* depth_processor = scanner.getDepthProcessor();
    
    // Load existing calibration
    auto* calib_manager = scanner.getCalibrationManager();
    calib_manager->loadCalibration("calibration/calib_boofcv_test3.yaml");
    
    // Process stereo pair
    cv::Mat left_image = cv::imread("stereo_left.png", cv::IMREAD_GRAYSCALE);
    cv::Mat right_image = cv::imread("stereo_right.png", cv::IMREAD_GRAYSCALE);
    cv::Mat depth_map;
    
    bool success = depth_processor->processFrames(left_image, right_image, depth_map);
    
    if (success) {
        cv::imwrite("depth_map.png", depth_map);
        std::cout << "Depth processing completed!" << std::endl;
    }
    
    return 0;
}
```

---

## 📊 Performance

### **Benchmarks (Raspberry Pi 5, 680×420 resolution)**

| Stage | Time | Optimization |
|-------|------|--------------|
| **Burst Capture** | ~2-3s | Continuous capture mode |
| **SGM-Census Stereo** | ~7-8s | OpenMP 4-core parallelization |
| **Point Cloud Generation** | ~1s | Border/statistical filtering |
| **Total** | **~10s** | 5-7× faster than baseline |

### **Optimization Techniques**

- ✅ **OpenMP Multi-threading**: Census, matching cost, 8-path SGM all parallelized
- ✅ **Burst Capture Mode**: Single continuous session (no start/stop overhead)
- ✅ **Memory Optimization**: CLAHE object reuse, reduced allocations
- 🔄 **Future**: NEON SIMD intrinsics, memory pooling, pipeline overlap

### **Target Performance Specifications**
- 🎯 **Precision**: 0.1mm repeatability at 100-400mm working distance
- ⚡ **Processing Speed**: VGA stereo >20 FPS, HD stereo >10 FPS
- 🔄 **Camera Sync**: <1ms synchronization precision
- 📱 **GUI Response**: <100ms for all touch interactions
- 🧠 **Memory Usage**: Optimized for 8GB, recommended 16GB
- 🔋 **Power Consumption**: <15W total system power

---

## 📁 Project Structure

```
unlook/
├── src/
│   ├── api/              # Public API implementation
│   ├── camera/           # Camera management and hardware sync
│   ├── stereo/           # SGM-Census stereo matching
│   ├── calibration/      # Calibration loading and validation
│   ├── hardware/         # I2C, GPIO, AS1170 LED driver
│   ├── pointcloud/       # Open3D point cloud processing
│   ├── mesh/             # Mesh generation and optimization
│   ├── gui/              # Qt5 graphical interface
│   └── core/             # Logging, exceptions, configuration
├── include/unlook/       # Public headers
├── third-party/
│   └── libcamera-sync-fix/  # Custom libcamera with hardware sync
├── calibration/          # Camera calibration files
├── tests/                # Comprehensive test suite
├── examples/             # Example applications
├── cmake/                # CMake modules and toolchains
├── CMakeLists.txt        # Main CMake configuration
├── build.sh              # Automated build script
└── README.md             # This file
```

---

## 🔧 Camera Calibration

Calibration files are stored in `/unlook_calib/` directory (created in home folder).

### **Calibration File Format**

```yaml
# Example: /unlook_calib/default.yaml
calibration_date: "2025-11-19T00:00:00"
calibration_method: "MATLAB_rectifyStereoImages_COMPLETE_EXPORT"
image_width: 1280
image_height: 720

camera_matrix_left: !!opencv-matrix
  rows: 3
  cols: 3
  dt: d
  data: [fx, 0, cx, 0, fy, cy, 0, 0, 1]

distortion_coeffs_left: !!opencv-matrix
  rows: 5
  cols: 1
  dt: d
  data: [k1, k2, p1, p2, k3]

# ... similar for right camera ...

rotation_matrix: !!opencv-matrix
  # 3×3 rotation between cameras

translation_vector: !!opencv-matrix
  # 3×1 translation (baseline)

rectification_transform_left: !!opencv-matrix
  # 3×3 rectification rotation

projection_matrix_left: !!opencv-matrix
  # 3×4 projection matrix

disparity_to_depth_matrix: !!opencv-matrix
  # 4×4 Q matrix for 3D reprojection
```

### **Supported Calibration Methods**

- **MATLAB Export**: `stereoParametersToOpenCV()` + `rectifyStereoImages()`
- **OpenCV**: `cv::stereoCalibrate()` + `cv::stereoRectify()`
- **ChArUco Boards**: Recommended for high precision

### **Calibration Targets**

Supported calibration patterns:

1. **Checkerboard**: 7×10 pattern, 24mm squares
2. **ChArUco Board #1**: 7×10, 24mm squares, 17mm ArUco markers, DICT_4X4_250
3. **ChArUco Board #2**: Alternative pattern with same specifications

---

## 🔬 Algorithm Details

### **Stereo Matching Pipeline**

1. **Image Acquisition**
   - Burst capture mode (continuous streaming)
   - Hardware-synchronized frames (<1ms sync)
   - Fixed exposure/gain from auto-calibration

2. **Preprocessing**
   - Epipolar rectification (cv::remap with precomputed maps)
   - Physical crop to 680×420 (69% memory/processing reduction)
   - CLAHE enhancement (clip=4.0 for VCSEL pattern visibility)

3. **SGM-Census Stereo Matching**
   - Census Transform: 9×9 window (81 bits binary descriptor)
   - Matching Cost: Hamming distance with ±2px vertical search
   - SGM Aggregation: 8-path dynamic programming (P1=8, P2=24)
   - WTA Selection: Winner-takes-all with subpixel interpolation

4. **Post-processing**
   - Border filter: Remove 40px margin (epipolar error zone)
   - Statistical outlier removal: k=12 neighbors, threshold=1.5σ
   - Multi-threaded filtering (OpenMP on 4 cores)

5. **3D Reconstruction**
   - Disparity-to-depth: cv::reprojectImageTo3D with Q matrix
   - Coordinate adjustment for crop offset
   - PLY export with RGB color (optional)

### **Technical Specifications**

- **Disparity Range**: 384 levels (0-383 pixels)
- **Subpixel Precision**: 1/16 pixel (parabola interpolation)
- **Depth Levels**: ~1500 distinct depth measurements
- **Valid Disparity Rate**: 60-85% (depending on scene texture)
- **Epipolar Error**: <2px after rectification (center crop region)

---

## 🛠️ Development

### **Build System Options**
```bash
# Development builds
./build.sh -t Debug --clean        # Debug build with clean
./build.sh --tests                 # Build with test suite
./build.sh --validate              # Validate dependencies only

# Production builds  
./build.sh -t Release -j 4          # Optimized release build
./build.sh --install               # System installation
./build.sh --package               # Create DEB/RPM packages
```

### **Testing and Validation**
```bash
# Run all tests
./build.sh --tests && make test -C build

# Hardware-specific tests (requires actual cameras)
./build/bin/test_camera_sync        # Camera synchronization test
./build/bin/test_sync_precision     # Timing precision validation
./test_synchronized_capture.sh     # Full system validation
```

### **Key Configuration**
Key CMake options:
```bash
# Enable specific features
cmake .. -DCMAKE_BUILD_TYPE=Release \
         -DUNLOOK_BUILD_GUI=ON \
         -DUNLOOK_BUILD_TESTS=ON \
         -DUNLOOK_BUILD_EXAMPLES=ON \
         -DUNLOOK_ENABLE_BOOFCV=ON
```

---

## 🗺️ Roadmap

### **Phase 1: Foundation** ✅ **(COMPLETED)**
- [x] Complete C++ API architecture
- [x] Hardware synchronization system (<1ms precision)
- [x] Qt5 fullscreen touch interface
- [x] OpenCV stereo processing pipeline
- [x] Build system with cross-compilation
- [x] Professional documentation and examples
- [x] SGM-Census stereo matching optimization
- [x] Point cloud generation and export (PLY)

### **Phase 2: Advanced Features** 🔄 **(IN PROGRESS)**
- [ ] Advanced calibration validation tools
- [ ] Performance optimization for CM5
- [ ] NEON SIMD optimizations for ARM64
- [ ] Memory pooling and pipeline overlap
- [ ] Enhanced point cloud filtering algorithms

### **Phase 3: Modular Scanning Technologies** 🔮 **(PLANNED)**
- [ ] **MEMS Structured Light Module**: High-resolution line scanning
- [ ] **LiDAR Distance Module**: Long-range precision measurement
- [ ] **Multi-wavelength VCSEL**: Enhanced pattern projection
- [ ] **Active Illumination Control**: Adaptive lighting systems
- [ ] Module hot-swap capability
- [ ] Unified API for all scanning modules

### **Phase 4: Production & Optimization** 🔮 **(FUTURE)**
- [ ] Industrial deployment packaging
- [ ] Advanced mesh generation algorithms
- [ ] Real-time preview optimization
- [ ] Extended calibration target support
- [ ] Multi-platform support (x86_64, ARM)

---

### **Coding Standards**
- **C++17/20 exclusively** - No Python in production code
- **Professional C++ patterns** - RAII, smart pointers, thread safety
- **Comprehensive documentation** - Every public API documented
- **Industrial quality** - Zero tolerance for incomplete implementations

---

## 📄 License

**MIT License** - See [LICENSE](LICENSE) file for details.

This project is completely open source and free for commercial use. We believe professional 3D scanning technology should be accessible to everyone.

---

## 🙏 Acknowledgments

- **Raspberry Pi Foundation** for excellent hardware platform
- **OpenCV Community** for robust computer vision libraries
- **Qt Project** for professional GUI framework
- **BoofCV Project** for high-precision calibration algorithms
- **libcamera Project** for professional camera control
- **Open3D** for point cloud and mesh processing

---

## 📞 Support & Contact

- 🌐 **Website**: [https://supernovaindustries.it](https://supernovaindustries.it)
- 📧 **Email**: contact@supernova-industries.com
- 🐙 **GitHub**: [https://github.com/SupernovaIndustries](https://github.com/SupernovaIndustries)
- 🐛 **Issues**: Use GitHub Issues for bug reports and feature requests

---

<p align="center">
  <strong>🚀 Professional 3D Scanning Made Accessible</strong><br>
  <em>Unlook 3D Scanner - Open Source Precision Engineering</em>
</p>

<p align="center">
  Made with ❤️ by <a href="https://supernovaindustries.it">Supernova Industries</a>
</p>
