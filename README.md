# WebGL/WebGPU Texture Upload Performance Benchmark Suite

A comprehensive benchmarking suite designed to measure and compare texture upload performance across WebGL and WebGPU APIs on various hardware platforms and browsers.

## 🎯 Purpose & Motivation

Modern web graphics applications require efficient texture streaming and upload capabilities. This benchmark suite was created to:

- **Compare WebGL vs WebGPU** texture upload performance across different hardware
- **Identify optimal texture upload strategies** for different texture sizes and scenarios
- **Analyze driver-level optimizations** and their impact on upload performance
- **Provide cross-platform performance data** to guide graphics optimization decisions
- **Test both mutable and immutable texture approaches** in WebGL for comprehensive analysis

![WebGL/WebGPU Texture Upload Test Screenshot](test-image.png)
*Example of the benchmark suite running with checkerboard texture patterns and real-time performance metrics*

**🚀 [Try the Live Demo](https://mvaligursky.github.io/webgl-webgpu-texture-upload/)**

## 🧪 Test Categories

### WebGL Tests

The WebGL benchmark (`index-webgl.html`) includes **dual-path testing** that compares both mutable (WebGL1-style) and immutable (WebGL2-style) texture approaches:

#### Core Test Types:
1. **Basic Test** - Baseline texture reuse with minimal GPU load (simple shader)
2. **GPU-Stressed Test** - Heavy fragment shader load with NxN sampling
3. **Realloc Test** - Fresh texture allocation each frame + NxN sampling
4. **Buffer-Orphan Test** - Texture orphaning with null data + NxN sampling
5. **Multi-buffer Tests** - Double/triple buffering approaches + NxN sampling
6. **PBO Tests** - Pixel Buffer Object optimizations + NxN sampling
7. **Pack-Align Tests** - Memory alignment optimizations + NxN sampling
8. **Memory Tests** - SharedArrayBuffer and aligned memory + NxN sampling
9. **GPU Sync Tests** - Different GPU synchronization approaches + NxN sampling

**Note**: Only the Basic Test uses a lightweight shader. All other tests use the expensive NxN sampling shader to stress both texture upload and GPU rendering performance.

#### Mutable vs Immutable Comparison:
- **WebGL1 Mode (Mutable)**: Uses `texImage2D` for allocation and `texSubImage2D` for updates
- **WebGL2 Mode (Immutable)**: Uses `texStorage2D` for allocation and `texSubImage2D` for updates

### WebGPU Tests

The WebGPU benchmark (`index-webgpu.html`) focuses on modern GPU API approaches:

#### Core Test Types:
1. **Basic Test** - Standard texture creation and upload (simple shader)
2. **GPU-Stressed Test** - Heavy compute load with NxN sampling  
3. **Realloc Test** - Fresh texture creation each frame + NxN sampling
4. **Multi-buffer Tests** - Multiple texture ping-ponging + NxN sampling
5. **Buffer-Copy Test** - Staging buffer to texture copy + NxN sampling
6. **Queue-Submit Test** - Command queue submission timing + NxN sampling
7. **Memory-Aligned Test** - Memory alignment optimizations + NxN sampling
8. **Storage Buffer Test** - Compute shader with storage buffer bypass + NxN sampling
9. **Tiled Upload Test** - Chunked texture upload approach + NxN sampling

**Note**: Only the Basic Test uses a simple shader. All other tests use the expensive NxN sampling shader to stress both texture upload and GPU rendering performance.

## 🔧 Technical Implementation

### Texture Format
- **Format**: `R32_UINT` (single-channel 32-bit unsigned integer)
- **Data Pattern**: High-contrast checkerboard for optimal blur visibility
- **Size Range**: 256×256 to 4096×4096 pixels
- **Adaptive Check Size**: Larger textures get proportionally larger checkerboard squares

### GPU Load Testing
- **Sampling Pattern**: Configurable NxN grid sampling (default: 40×40 = 1600 samples/fragment)
- **Visual Validation**: Sampling blur effects should be visible and vary with texture size
- **Performance Impact**: Creates realistic GPU load to stress texture bandwidth

### Measurement Methodology
- **Iterations**: 200 loops (full test) or 50 loops (quick test)
- **Timing**: High-precision `performance.now()` measurements
- **Metrics**: Upload time, render time, throughput (MB/s), frame skipping
- **Warmup**: Initial iterations excluded from final averages

## 🚀 How to Run

### Prerequisites
- Modern browser with WebGL2 and/or WebGPU support
- Hardware accelerated graphics

### Running Tests
1. **WebGL Tests**: Open `index-webgl.html`
2. **WebGPU Tests**: Open `index-webgpu.html`
3. **Test Options**:
   - **Run All Tests (200 loops)**: Comprehensive benchmark
   - **Run Quick Tests (50 loops)**: Faster validation
   - **GPU Load**: Adjust NxN sampling (1-50, default: 40)

### Expected Visual Behavior
- **Basic Tests**: Sharp checkerboard pattern
- **GPU-Stressed Tests**: Blurred checkerboard (sampling effect)
- **Size Variation**: Blur intensity should vary with texture size
- **Consistent APIs**: WebGL and WebGPU should show similar visual effects

## 📊 Performance Results by Platform

### Apple M4 Max (MacBook Pro, macOS, Chrome 138.0.0.0)

#### WebGL Results:

**WebGL1 (Mutable Textures):**
```
Test Method  | 256×256  | 512×512  | 1024×1024 | 2048×2048  | 4096×4096 
-------------|----------|----------|-----------|------------|----------
Basic        | 0.03ms   | 0.06ms   | 0.41ms    | 1.64ms     | 6.30ms
GPU-Stress   | 0.24ms   | 1.84ms   | 1.20ms    | 7.82ms     | 14.73ms
Realloc      | 1.66ms   | 2.41ms   | 1.95ms    | 0.93ms ⭐  | 4.64ms ⭐
Buf-Orphan   | 38.13ms  | 1.78ms   | 1.22ms    | 7.79ms     | 14.69ms
Pack-Aln1    | 0.37ms ⭐ | 1.78ms   | 1.11ms ⭐ | 7.89ms     | 14.71ms
Pack-Aln8    | 1.32ms   | 1.71ms   | 1.16ms    | 7.87ms     | 14.69ms
Mem-Share    | 0.99ms   | 1.32ms ⭐ | 1.23ms    | 7.80ms     | 14.56ms
```

**WebGL2 (Immutable Textures):**
```
Test Method  | 256×256  | 512×512  | 1024×1024 | 2048×2048  | 4096×4096 
-------------|----------|----------|-----------|------------|----------
Basic        | 0.87ms   | 0.05ms   | 0.35ms    | 1.84ms     | 6.87ms
GPU-Stress   | 0.47ms   | 1.57ms   | 1.07ms    | 7.83ms     | 14.59ms
Realloc      | 1.48ms   | 2.30ms   | 1.78ms    | 7.74ms ⭐  | 14.54ms ⭐
Buf-Orphan   | 1.32ms   | 1.58ms   | 1.12ms ⭐ | 7.89ms     | 14.62ms
Pack-Aln1    | 0.43ms ⭐ | 1.56ms ⭐ | 1.12ms ⭐ | 7.81ms     | 14.54ms ⭐
Pack-Aln8    | 1.28ms   | 2.33ms   | 1.80ms    | 8.44ms     | 14.73ms
Mem-Share    | 0.86ms   | 3.11ms   | 2.19ms    | 8.80ms     | 15.17ms
```

**WebGPU Results:**
```
Test Method  | 256×256  | 512×512  | 1024×1024 | 2048×2048  | 4096×4096 
-------------|----------|----------|-----------|------------|----------
Basic        | 0.02ms   | 0.05ms   | 0.13ms    | 2.75ms     | 10.56ms
GPU-Stress   | 0.02ms ⭐ | 0.06ms ⭐ | 0.18ms    | 2.72ms     | 10.58ms
Storage Buf  | 0.04ms   | 0.06ms ⭐ | 0.12ms    | 2.69ms ⭐  | 10.74ms
Realloc      | 0.01ms ⭐ | 0.08ms   | 0.19ms    | 3.06ms     | 10.46ms
Multi-Buf    | 0.02ms ⭐ | 0.06ms ⭐ | 0.10ms ⭐ | 2.74ms     | 10.40ms
Buf-Copy     | 2.51ms   | 2.99ms   | 3.78ms    | 4.91ms     | 10.21ms ⭐
Tiled        | 0.29ms   | 0.53ms   | 1.56ms    | 5.55ms     | 76.87ms
```

**Platform**: Apple M4 Max, macOS, Chrome 138.0.0.0  
**Driver**: ANGLE Metal Renderer

> **⭐ Star Highlighting**: Stars indicate the best performing upload strategy (excluding Basic test) for each texture size. This focuses on upload performance when the GPU is under realistic load, as Basic tests use minimal shader complexity and don't represent real-world usage patterns.

---

### Apple M4 Max (MacBook Pro, macOS, Safari Tech Preview 223)

#### WebGL Results:

**WebGL1 (Mutable Textures):**
```
Test Method  | 256×256  | 512×512  | 1024×1024 | 2048×2048  | 4096×4096 
-------------|----------|----------|-----------|------------|----------
Basic        | 0.01ms   | 0.03ms   | 0.24ms    | 0.94ms     | 3.58ms
GPU-Stress   | 0.01ms ⭐ | 0.01ms ⭐ | 0.23ms    | 0.94ms     | 3.63ms ⭐
Realloc      | 0.01ms ⭐ | 0.01ms ⭐ | 0.25ms    | 0.90ms ⭐  | 3.72ms
Buf-Orphan   | 0.00ms ⭐ | 0.03ms   | 0.25ms    | 0.92ms     | 3.81ms
Double-Buf   | 0.00ms ⭐ | 0.03ms   | 0.24ms    | 0.93ms     | 3.81ms
Triple-Buf   | 0.01ms ⭐ | 0.01ms ⭐ | 0.22ms ⭐ | 1.15ms     | 3.96ms
Quad-Buf     | 0.00ms ⭐ | 0.01ms ⭐ | 0.36ms    | 1.18ms     | 3.92ms
Penta-Buf    | 0.00ms ⭐ | 0.03ms   | 0.23ms    | 1.12ms     | 3.99ms
PBO-Single   | 0.01ms ⭐ | 0.03ms   | 0.27ms    | 1.30ms     | 4.17ms
PBO-Double   | 0.02ms   | 0.02ms   | 0.27ms    | 1.32ms     | 4.18ms
Pack-Aln1    | 0.02ms   | 0.01ms ⭐ | 0.26ms    | 0.99ms     | 3.91ms
Pack-Aln8    | 0.01ms ⭐ | 0.02ms   | 0.26ms    | 1.01ms     | 3.92ms
Sync-Flush   | 0.01ms ⭐ | 0.02ms   | 0.26ms    | 0.99ms     | 3.94ms
Sync-Fin     | 0.01ms ⭐ | 0.03ms   | 0.23ms    | 1.03ms     | 3.91ms
Sync-None    | 0.01ms ⭐ | 0.01ms ⭐ | 0.25ms    | 1.00ms     | 3.95ms
Mem-Align    | 0.05ms   | 0.08ms   | 0.48ms    | 1.81ms     | 6.42ms
Mem-Share    | 0.00ms ⭐ | 0.02ms   | 0.29ms    | 1.04ms     | 4.00ms
```

**WebGL2 (Immutable Textures):**
```
Test Method  | 256×256  | 512×512  | 1024×1024 | 2048×2048  | 4096×4096 
-------------|----------|----------|-----------|------------|----------
Basic        | 0.02ms   | 0.03ms   | 0.28ms    | 1.02ms     | 3.94ms
GPU-Stress   | 0.01ms   | 0.01ms ⭐ | 0.27ms    | 1.02ms     | 4.02ms
Realloc      | 0.01ms   | 0.01ms ⭐ | 0.27ms    | 1.00ms     | 3.97ms
Buf-Orphan   | 0.00ms ⭐ | 0.03ms   | 0.24ms    | 1.00ms     | 3.94ms
Double-Buf   | 0.00ms ⭐ | 0.01ms ⭐ | 0.25ms    | 1.21ms     | 3.84ms
Triple-Buf   | 0.01ms   | 0.03ms   | 0.28ms    | 1.18ms     | 3.93ms
Quad-Buf     | 0.03ms   | 0.06ms   | 0.35ms    | 1.19ms     | 3.90ms
Penta-Buf    | 0.01ms   | 0.02ms   | 0.24ms    | 1.00ms     | 3.94ms
PBO-Single   | 0.00ms ⭐ | 0.02ms   | 0.31ms    | 1.32ms     | 4.19ms
PBO-Double   | 0.00ms ⭐ | 0.03ms   | 0.30ms    | 1.32ms     | 4.18ms
Pack-Aln1    | 0.00ms ⭐ | 0.02ms   | 0.28ms    | 0.97ms ⭐  | 3.86ms
Pack-Aln8    | 0.00ms ⭐ | 0.01ms ⭐ | 0.25ms    | 0.99ms     | 3.89ms
Sync-Flush   | 0.00ms ⭐ | 0.02ms   | 0.24ms    | 1.00ms     | 3.89ms
Sync-Fin     | 0.01ms   | 0.02ms   | 0.26ms    | 0.98ms     | 3.97ms
Sync-None    | 0.00ms ⭐ | 0.02ms   | 0.23ms ⭐ | 0.98ms     | 3.90ms
Mem-Align    | 0.03ms   | 0.09ms   | 0.46ms    | 1.67ms     | 6.20ms
Mem-Share    | 0.01ms   | 0.01ms ⭐ | 0.28ms    | 0.99ms     | 3.79ms ⭐
```

**WebGPU Results:**
```
Test Method  | 256×256  | 512×512  | 1024×1024 | 2048×2048  | 4096×4096 
-------------|----------|----------|-----------|------------|----------
Basic        | 0.12ms   | 0.31ms   | 1.19ms    | 4.86ms     | 3.74ms
GPU-Stress   | 0.08ms   | 0.30ms   | 1.20ms    | 4.75ms ⭐  | 3.76ms ⭐
Realloc      | 0.07ms   | 0.27ms ⭐ | 1.21ms    | 4.84ms     | 3.88ms
Double-Buf   | 0.06ms   | 0.28ms   | 1.22ms    | 4.98ms     | 3.91ms
Triple-Buf   | 0.05ms   | 0.31ms   | 1.28ms    | 5.14ms     | 3.93ms
Quad-Buf     | 0.04ms ⭐ | 0.31ms   | 1.30ms    | 5.19ms     | 4.10ms
Penta-Buf    | 0.08ms   | 0.32ms   | 1.30ms    | 5.22ms     | 4.07ms
Buf-Copy     | 0.59ms   | 1.27ms   | 5.28ms    | 19.10ms    | 43.40ms
Que-Submit   | 0.28ms   | 0.76ms   | 2.83ms    | 11.40ms    | 15.57ms
Mem-Align    | 0.07ms   | 0.39ms   | 1.45ms    | 5.76ms     | 6.37ms
Storage      | 0.10ms   | 0.42ms   | 1.30ms    | 5.25ms     | 4.22ms
Tiled        | 0.55ms   | 2.00ms   | 7.97ms    | 33.30ms    | 157.12ms
GPU-Realloc  | 0.10ms   | 0.28ms   | 1.29ms    | 5.06ms     | 4.04ms
Fresh-Buf    | 0.53ms   | 1.25ms   | 5.28ms    | 19.42ms    | 45.10ms
Map-Buf      | 0.55ms   | 1.26ms   | 5.16ms    | 19.11ms    | 45.01ms
```

**Platform**: Apple M4 Max, macOS, Safari Tech Preview 223  
**Driver**: WebKit WebGPU Implementation

## 🤖 Development Credits

This repository was built using **Windsurf with Claude Sonnet 3.5** - an AI-powered development environment that enabled rapid prototyping, comprehensive testing, and detailed performance analysis across WebGL and WebGPU APIs.

## 📝 License

MIT License - Feel free to use, modify, and distribute.

## 🔗 References

- [WebGL Specification](https://www.khronos.org/webgl/)
- [WebGPU Specification](https://gpuweb.github.io/gpuweb/)
- [Texture Upload Optimization Techniques](https://developer.nvidia.com/content/constant-buffers-without-constant-pain-0)
