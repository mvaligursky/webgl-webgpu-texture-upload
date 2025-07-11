# WebGL/WebGPU Texture Upload Performance Benchmark Suite

A comprehensive benchmarking suite designed to measure and compare texture upload performance across WebGL and WebGPU APIs on various hardware platforms and browsers.

## 🎯 Purpose & Motivation

Modern web graphics applications require efficient texture streaming and upload capabilities. This benchmark suite was created to:

- **Compare WebGL vs WebGPU** texture upload performance across different hardware
- **Identify optimal texture upload strategies** for different texture sizes and scenarios
- **Analyze driver-level optimizations** and their impact on upload performance
- **Provide cross-platform performance data** to guide graphics optimization decisions
- **Test both mutable and immutable texture approaches** in WebGL for comprehensive analysis

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

### Apple M4 (MacBook Pro, macOS Sonoma, Chrome)

#### WebGL Results (R32U format):
```
Texture Size | Basic   | GPU-Stress | Realloc | Best Strategy
-------------|---------|------------|---------|----------------
256×256      | 0.02ms  | 0.32ms     | 1.79ms  | Basic
512×512      | 0.06ms  | 1.81ms     | 2.36ms  | Basic  
1024×1024    | 0.42ms  | 1.22ms     | 2.00ms  | Basic
2048×2048    | 1.63ms  | 7.80ms     | 0.96ms  | Realloc ⭐
4096×4096    | 6.64ms  | 14.58ms    | 3.85ms  | Realloc ⭐
```

#### WebGPU Results (R32U format):
```
Texture Size | Basic   | GPU-Stress | Storage Buffer | Best Strategy
-------------|---------|------------|----------------|----------------
256×256      | 0.02ms  | 0.03ms     | 0.03ms        | All equivalent
512×512      | 0.06ms  | 0.07ms     | 0.07ms        | All equivalent
1024×1024    | 0.17ms  | 0.20ms     | 0.11ms        | Storage Buffer ⭐
2048×2048    | 2.75ms  | 2.83ms     | 2.77ms        | All equivalent
4096×4096    | 10.81ms | 10.54ms    | 11.21ms       | GPU-Stress
```

#### Cross-API Comparison:
- **≤1024×1024**: WebGPU dominates (10-53x faster)
- **≥2048×2048**: WebGL Realloc wins (2.5-3x faster)  
- **Optimal Strategy**: WebGPU Storage Buffer (≤1024) → WebGL Realloc (≥2048)

---

### Apple M4 Max (MacBook Pro, macOS, Chrome 138.0.0.0)

#### WebGL Results:

**WebGL1 (Mutable Textures):**
```
Texture Size | Basic   | GPU-Stress| Realloc | Buf-Orphan| Pack-Aln1| Pack-Aln8 | Mem-Share | Notes
-------------|---------|-----------|---------|-----------|----------|----------|-----------|-------
256×256      | **0.03ms**| 0.24ms    | 1.66ms  | 38.13ms   | 0.37ms   | 1.32ms   | 0.99ms    | Basic best
512×512      | **0.06ms**| 1.84ms    | 2.41ms  | 1.78ms    | 1.78ms   | 1.71ms   | 1.32ms    | Basic best
1024×1024    | 0.41ms  | 1.20ms    | 1.95ms  | 1.22ms    | **1.11ms**| 1.16ms   | 1.23ms    | Pack-Aln1 best
2048×2048    | 1.64ms  | 7.82ms    | **0.93ms**| 7.79ms   | 7.89ms   | 7.87ms   | 7.80ms    | Realloc best
4096×4096    | 6.30ms  | 14.73ms   | **4.64ms**| 14.69ms  | 14.71ms  | 14.69ms  | 14.56ms   | Realloc best
```

**WebGL2 (Immutable Textures):**
```
Texture Size | Basic   | GPU-Stress| Realloc | Buf-Orphan| Pack-Aln1| Pack-Aln8 | Mem-Share | Notes
-------------|---------|-----------|---------|-----------|----------|----------|-----------|-------
256×256      | 0.87ms  | 0.47ms    | 1.48ms  | 1.32ms    | **0.43ms**| 1.28ms   | 0.86ms    | Pack-Aln1 best
512×512      | **0.05ms**| 1.57ms    | 2.30ms  | 1.58ms    | 1.56ms   | 2.33ms   | 3.11ms    | Basic best
1024×1024    | **0.35ms**| 1.07ms    | 1.78ms  | 1.12ms    | 1.12ms   | 1.80ms   | 2.19ms    | Basic best
2048×2048    | **1.84ms**| 7.83ms    | 7.74ms  | 7.89ms    | 7.81ms   | 8.44ms   | 8.80ms    | Basic best for WebGL2
4096×4096    | **6.87ms**| 14.59ms   | 14.54ms | 14.62ms   | 14.54ms  | 14.73ms  | 15.17ms   | Basic best for WebGL2
```

#### WebGPU Results:
```
Texture Size | Basic   | GPU-Stress| Storage Buf| Realloc | Multi-Buf | Buf-Copy | Tiled   | Notes
-------------|---------|-----------|------------|---------|-----------|----------|---------|-------
256×256      | 0.02ms  | 0.02ms    | 0.04ms     | **0.01ms**| 0.02ms    | 2.51ms   | 0.29ms  | Realloc best
512×512      | **0.05ms**| **0.06ms**| **0.06ms** | 0.08ms  | **0.06ms**| 2.99ms   | 0.53ms  | Basic/GPU-Stress/Storage/Multi best
1024×1024    | 0.13ms  | 0.18ms    | **0.12ms** | 0.19ms  | **0.10ms**| 3.78ms   | 1.56ms  | Multi-Buf best
2048×2048    | 2.75ms  | **2.72ms**| **2.69ms** | 3.06ms  | **2.74ms**| 4.91ms   | 5.55ms  | Storage/GPU-Stress/Multi best
4096×4096    | 10.56ms | 10.58ms   | 10.74ms    | **10.46ms**| **10.40ms**| **10.21ms**| 76.87ms | Multi-Buf/Buf-Copy best
```

**Platform**: Apple M4 Max, macOS, Chrome 138.0.0.0  
**Driver**: ANGLE Metal Renderer

---

### [Platform Name] - [Hardware] - [OS] - [Browser]

*Template for additional platform testing results*

#### WebGL Results:
```
Texture Size | Basic | GPU-Stress | Realloc | Notes
-------------|-------|------------|---------|-------
256×256      |       |            |         |
512×512      |       |            |         |
1024×1024    |       |            |         |
2048×2048    |       |            |         |
4096×4096    |       |            |         |
```

#### WebGPU Results:
```
Texture Size | Basic | GPU-Stress | Storage Buffer | Notes
-------------|-------|------------|----------------|-------
256×256      |       |            |                |
512×512      |       |            |                |
1024×1024    |       |            |                |
2048×2048    |       |            |                |
4096×4096    |       |            |                |
```

#### Platform-Specific Observations:
- Driver version:
- Key performance characteristics:
- Optimal strategies:
- Notable differences from Apple M4:

---

## 🔬 Analysis & Insights

### Key Findings
1. **Hardware-Dependent Optimization**: Different GPUs favor different strategies
2. **Size-Based Strategy Selection**: Optimal approach varies significantly with texture size
3. **API-Specific Advantages**: WebGL's realloc hack vs WebGPU's storage buffer approach
4. **Driver Maturity**: WebGL more mature, WebGPU rapidly improving

### Optimization Recommendations
- **Cross-platform apps**: Implement adaptive strategy selection based on texture size and API
- **WebGL optimization**: Use allocation-stressed approach for large textures (≥2048×2048)
- **WebGPU optimization**: Leverage storage buffers for small-medium textures (≤1024×1024)
- **Performance testing**: Always benchmark on target hardware before deployment

## 🤝 Contributing

To contribute platform-specific results:

1. Run both WebGL and WebGPU tests on your platform
2. Record the console output summary tables
3. Note your hardware, OS, and browser details
4. Add results using the template above
5. Submit via pull request or issue

**Platform Details Template:**
```
Platform: [CPU] + [GPU]
OS: [OS Version]
Browser: [Browser + Version]
Driver: [Graphics Driver Version]
Date: [Test Date]
```

## 📝 License

MIT License - Feel free to use, modify, and distribute.

## 🔗 References

- [WebGL Specification](https://www.khronos.org/webgl/)
- [WebGPU Specification](https://gpuweb.github.io/gpuweb/)
- [Texture Upload Optimization Techniques](https://developer.nvidia.com/content/constant-buffers-without-constant-pain-0)
