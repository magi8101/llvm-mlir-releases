# llvm-mlir-releases

Automated, pre-built LLVM, Clang, and MLIR shared libraries distributed directly via GitHub Releases.

## What This Provides

This repository builds two separate sets of pre-compiled libraries from the official [llvm-project](https://github.com/llvm/llvm-project) source, utilizing GitHub Actions.

1. **LLVM Release** (e.g., `llvm-18.1.8-<platform>.zip`)
   - **Linux & macOS**: A single, unified `libLLVM.so` / `libLLVM.dylib` containing all LLVM components.
   - **Windows**: Statically linked LLVM tools + `libclang.dll` C API (due to MSVC ABI limitations preventing a unified `libLLVM.dll`).
   - Includes all LLVM headers and CMake config files (`find_package(LLVM)`).

2. **MLIR Release** (e.g., `mlir-18.1.8-<platform>.zip`)
   - The standalone **MLIR C API shared library** (`MLIR-C.dll` / `libMLIR-C.so` / `libMLIR-C.dylib`).
   - Includes MLIR C headers and CMake config files (`find_package(MLIR)`).

## Supported Platforms & Targets

We compile for **ALL LLVM target architectures** (x86, AArch64, WebAssembly, ARM, RISC-V, etc.) across the following platforms:

| OS Platform | Compiler Used | Archive Format |
|-------------|---------------|----------------|
| Windows x64 | **Clang-cl** (MSVC ABI) | `.zip` |
| Ubuntu x64  | GCC | `.tar.gz` |
| macOS ARM64 | AppleClang (Apple Silicon) | `.tar.gz` |

## Release Naming & Triggers

Releases are triggered automatically when tags are pushed to this repository:
- Pushing a tag like `llvm-v18.1.8` triggers GitHub actions to download the `18.1.8` source from the official LLVM repo.
- Both LLVM and MLIR workflows run simultaneously and attach their output archives to the exact same GitHub Release page.

## How to Consume the Binaries

Download the appropriate archive for your operating system from the [Releases](https://github.com/magi8101/llvm-mlir-releases/releases) page.

Extract the archive. It contains a standard install tree (`bin/`, `lib/`, `include/`). Point your project's CMake configuration to the extracted directories:

```cmake
# In your project's CMake configuration step:
cmake -DLLVM_DIR=/path/to/extracted/llvm/lib/cmake/llvm ..
cmake -DMLIR_DIR=/path/to/extracted/mlir/lib/cmake/mlir ..

# Inside your CMakeLists.txt:
find_package(LLVM REQUIRED CONFIG)
find_package(MLIR REQUIRED CONFIG)

message(STATUS "Found LLVM ${LLVM_PACKAGE_VERSION}")
```

## Local Build Reproduction (CMake Flags)

These libraries are built fully autonomously by GitHub Actions. If you need to reproduce the build locally, here are the exact CMake configurations used.

### LLVM Build Configuration

**Linux / macOS (Unified Shared Library):**
```bash
cmake -G Ninja -S llvm-project/llvm -B build \
  -DCMAKE_BUILD_TYPE=Release \
  -DLLVM_ENABLE_PROJECTS="clang" \
  -DLLVM_BUILD_LLVM_DYLIB=ON \
  -DLLVM_LINK_LLVM_DYLIB=ON \
  -DLLVM_TARGETS_TO_BUILD="all" \
  -DLLVM_INCLUDE_TESTS=OFF \
  -DLLVM_INCLUDE_EXAMPLES=OFF \
  -DLLVM_INCLUDE_BENCHMARKS=OFF \
  -DLLVM_INCLUDE_DOCS=OFF
```

**Windows (Clang-cl without libLLVM DLL):**
```bash
cmake -G Ninja -S llvm-project/llvm -B build \
  -DCMAKE_C_COMPILER=clang-cl \
  -DCMAKE_CXX_COMPILER=clang-cl \
  -DLLVM_USE_LINKER=lld \
  -DCMAKE_BUILD_TYPE=Release \
  -DLLVM_ENABLE_PROJECTS="clang" \
  -DLLVM_TARGETS_TO_BUILD="all" \
  -DLLVM_INCLUDE_TESTS=OFF \
  -DLLVM_INCLUDE_EXAMPLES=OFF \
  -DLLVM_INCLUDE_BENCHMARKS=OFF \
  -DLLVM_INCLUDE_DOCS=OFF
```

### MLIR Build Configuration

**All Platforms:**
*(Windows additionally passes the `clang-cl` and `lld` flags shown above, and omits the `LLVM_BUILD_LLVM_DYLIB` lines)*
```bash
cmake -G Ninja -S llvm-project/llvm -B build \
  -DCMAKE_BUILD_TYPE=Release \
  -DLLVM_ENABLE_PROJECTS="mlir" \
  -DMLIR_BUILD_MLIR_C_DYLIB=ON \
  -DLLVM_BUILD_LLVM_DYLIB=ON \ # (Linux/macOS only)
  -DLLVM_LINK_LLVM_DYLIB=ON \  # (Linux/macOS only)
  -DLLVM_TARGETS_TO_BUILD="all" \
  -DLLVM_INCLUDE_TESTS=OFF \
  -DLLVM_INCLUDE_EXAMPLES=OFF \
  -DLLVM_INCLUDE_BENCHMARKS=OFF \
  -DLLVM_INCLUDE_DOCS=OFF
```

## License

The LLVM and MLIR libraries are distributed under the [Apache License v2.0 with LLVM Exceptions](https://llvm.org/LICENSE.txt).
