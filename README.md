# Unlook 3D Scanner

**Professional industrial-grade 3D scanning system** powered by stereo vision and VCSEL structured light technology.

[![License: MIT](https://img.shields.io/badge/License-MIT-blue.svg)](LICENSE)
[![Platform](https://img.shields.io/badge/Platform-Raspberry%20Pi%205-green.svg)](https://www.raspberrypi.com/)
[![C++](https://img.shields.io/badge/C%2B%2B-17%2F20-blue.svg)](https://isocpp.org/)

## Overview

Unlook is an open-source, modular 3D scanner designed for industrial applications requiring **0.005mm precision**. Built on Raspberry Pi 5 (CM5) hardware, it combines advanced stereo vision algorithms with VCSEL-based structured light for high-quality 3D reconstruction.

### Key Features

- **High Precision**: 0.005mm repeatability for industrial metrology
- **Real-time Performance**: ~10 seconds total processing (2s capture + 8s processing)
- **Hardware Synchronized Stereo**: Custom libcamera-sync for <1ms camera synchronization
- **VCSEL Structured Light**: AS1170 dual VCSEL driver for dot pattern projection
- **Advanced Algorithms**: SGM-Census stereo matching with OpenMP parallelization
- **Professional GUI**: Qt5-based touch interface for handheld scanning
- **Complete Pipeline**: Camera → Calibration → Stereo Matching → Point Cloud → PLY Export

## Hardware Requirements

### Required Components

- **Compute Module**: Raspberry Pi Compute Module 5 (CM5) or Raspberry Pi 5
- **Cameras**: 2× Global Shutter Camera (Raspberry Pi IMX296, 1456×1088, SBGGR10)
  - Hardware sync via XVS/XHS connections
  - Baseline: ~70mm (calibrated)
- **Illumination**:
  - LED1: ams OSRAM BELAGO1.1 VCSEL dot projector
  - LED2: Flood illuminator for calibration
- **LED Driver**: AS1170 LED controller (I2C bus 1, address 0x30)
- **Optics**: 6mm focal length lenses

### Camera Configuration

```
Camera Mapping (Scanner POV):
- Camera 0 (/base/soc/i2c0mux/i2c@0/imx296@1a): RIGHT camera
- Camera 1 (/base/soc/i2c0mux/i2c@1/imx296@1a): LEFT camera (MASTER)

Hardware Synchronization:
- Camera 1 (LEFT) = MASTER
- Camera 0 (RIGHT) = SLAVE
- Sync signals: XVS, XHS, GND connected
- Precision: <1ms synchronization
```

## Software Architecture

### Core Components

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

### Technology Stack

- **Language**: C++17/20 exclusively
- **Build System**: CMake 3.16+ with cross-compilation support
- **Dependencies**:
  - OpenCV 4.6+ (stereo, calibration, image processing)
  - Qt5 5.15+ (GUI framework)
  - Open3D 0.16+ (point cloud and mesh processing)
  - libcamera-sync (custom hardware synchronization)
- **Optimization**: OpenMP 4.5+ for multi-core parallelization (4 cores)

## Installation

### Prerequisites

```bash
# Install system dependencies (Debian/Ubuntu)
sudo apt-get update
sudo apt-get install -y \
    build-essential cmake pkg-config \
    libopencv-dev qt5-default \
    libomp-dev libopen3d-dev \
    libnlohmann-json3-dev
```

### Build from Source

```bash
# Clone repository
git clone https://github.com/SupernovaIndustries/unlook.git
cd unlook

# Build (automatic dependency detection)
./build.sh

# Cross-compile for Raspberry Pi 5
./build.sh --cross rpi5 -j 4
```

### System Installation

After building, install system-wide:

```bash
cd build
sudo make install

# Run from anywhere
unlook
```

## Camera Calibration

Calibration files are stored in `/unlook_calib/` directory (created in home folder).

### Calibration File Format

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

### Supported Calibration Methods

- **MATLAB Export**: `stereoParametersToOpenCV()` + `rectifyStereoImages()`
- **OpenCV**: `cv::stereoCalibrate()` + `cv::stereoRectify()`
- **ChArUco Boards**: Recommended for high precision

## Usage

### GUI Application

```bash
# Run scanner GUI (system-installed)
unlook

# Or run from build directory
LD_LIBRARY_PATH=build/src:third-party/libcamera-sync-fix/build/src/libcamera:third-party/libcamera-sync-fix/build/src/libcamera/base:$LD_LIBRARY_PATH \
./build/src/gui/unlook_scanner
```

### API Example

```cpp
#include <unlook/unlook.h>

int main() {
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

    return 0;
}
```

## Performance

### Benchmarks (Raspberry Pi 5, 680×420 resolution)

| Stage | Time | Optimization |
|-------|------|--------------|
| **Burst Capture** | ~2-3s | Continuous capture mode |
| **SGM-Census Stereo** | ~7-8s | OpenMP 4-core parallelization |
| **Point Cloud Generation** | ~1s | Border/statistical filtering |
| **Total** | **~10s** | 5-7× faster than baseline |

### Optimization Techniques

- ✅ **OpenMP Multi-threading**: Census, matching cost, 8-path SGM all parallelized
- ✅ **Burst Capture Mode**: Single continuous session (no start/stop overhead)
- ✅ **Memory Optimization**: CLAHE object reuse, reduced allocations
- 🔄 **Future**: NEON SIMD intrinsics, memory pooling, pipeline overlap

## Project Structure

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
├── CMakeLists.txt        # Main CMake configuration
├── build.sh              # Automated build script
└── README.md             # This file
```

## Algorithm Details

### Stereo Matching Pipeline

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

### Technical Specifications

- **Disparity Range**: 384 levels (0-383 pixels)
- **Subpixel Precision**: 1/16 pixel (parabola interpolation)
- **Depth Levels**: ~1500 distinct depth measurements
- **Valid Disparity Rate**: 60-85% (depending on scene texture)
- **Epipolar Error**: <2px after rectification (center crop region)

## Calibration Targets

Supported calibration patterns:

1. **Checkerboard**: 7×10 pattern, 24mm squares
2. **ChArUco Board #1**: 7×10, 24mm squares, 17mm ArUco markers, DICT_4X4_250
3. **ChArUco Board #2**: Alternative pattern with same specifications

## License

This project is licensed under the MIT License - see the [LICENSE](LICENSE) file for details.

## Contributing

Contributions are welcome! Please read our [Development Guidelines](https://github.com/SupernovaIndustries/unlook-standalone) for details on:

- Code standards (C++17/20, professional OOP)
- Build system requirements
- Testing procedures
- Pull request process

## Support

For industrial applications, custom development, or commercial support:
- **GitHub Issues**: [Report bugs or request features](https://github.com/SupernovaIndustries/unlook/issues)
- **Email**: contact@supernova-industries.com

## Acknowledgments

- **OpenCV**: Stereo vision and calibration algorithms
- **Open3D**: Point cloud and mesh processing
- **Qt Project**: Cross-platform GUI framework
- **Raspberry Pi Foundation**: Hardware platform
- **ams OSRAM**: VCSEL structured light technology

---

**Built with precision. Designed for professionals.**

*Unlook 3D Scanner - Industrial metrology meets open-source innovation.*
