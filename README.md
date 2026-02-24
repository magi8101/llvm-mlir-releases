# llvm-mlir-releases

Automated, pre-built LLVM, Clang, MLIR, and CIRCT libraries distributed via GitHub Releases.

## What This Provides

This repository builds three separate sets of pre-compiled libraries from the official LLVM ecosystem sources, using GitHub Actions.

1. **LLVM Release** (e.g., `llvm-18.1.8-<platform>`)
   - **Linux & macOS**: A unified `libLLVM.so` / `libLLVM.dylib` shared library containing all LLVM components.
   - **Windows**: Static LLVM libraries built with `clang-cl` (MSVC ABI does not support `LLVM_BUILD_LLVM_DYLIB`).
   - Includes **Clang**, **LLD** (linker), and **Polly** (polyhedral optimizer).
   - All LLVM/Clang/LLD/Polly headers and CMake config files.

2. **MLIR Release** (e.g., `mlir-18.1.8-<platform>`)
   - The standalone **MLIR C API shared library** (`MLIR-C.dll` / `libMLIR-C.so` / `libMLIR-C.dylib`).
   - MLIR C headers and CMake config files.

3. **CIRCT Release** (e.g., `circt-1.140.0-<platform>`)
   - CIRCT (Circuit IR Compilers and Tools) built as a unified LLVM+MLIR+CIRCT package.
   - **Linux & macOS**: Shared LLVM dylib with host-architecture targets.
   - **Windows**: Static libraries built with `clang-cl` (same MSVC ABI limitation as LLVM).
   - Versioned independently from LLVM using CIRCT's `firtool-X.Y.Z` release tags.

## Supported Platforms

### LLVM & MLIR

Compiled for **all LLVM target architectures** (x86, AArch64, WebAssembly, ARM, RISC-V, etc.):

| OS Platform | Compiler | Library Type | Archive |
|-------------|----------|-------------|---------|
| Ubuntu x64  | GCC | Shared (`libLLVM.so`) | `.tar.gz` |
| macOS ARM64 | AppleClang | Shared (`libLLVM.dylib`) | `.tar.gz` |
| Windows x64 | clang-cl (MSVC ABI) | Static | `.zip` |

### CIRCT

Compiled for **host architecture only** (CIRCT does hardware IR transforms, not software codegen):

| OS Platform | Compiler | Library Type | Archive |
|-------------|----------|-------------|---------|
| Ubuntu x64  | GCC | Shared (`libLLVM.so`) | `.tar.gz` |
| macOS ARM64 | AppleClang | Shared (`libLLVM.dylib`) | `.tar.gz` |
| Windows x64 | clang-cl (MSVC ABI) | Static | `.zip` |

## Release Naming & Triggers

Releases are triggered automatically when tags are pushed:
- **LLVM + MLIR**: Push `llvm-v18.1.8` → both workflows run, attach output to the same Release.
- **CIRCT**: Push `circt-v1.140.0` → CIRCT workflow runs with its own Release page.

All workflows also support `workflow_dispatch` for manual builds with a version input.

## How to Consume

Download the archive for your OS from the [Releases](https://github.com/magi8101/llvm-mlir-releases/releases) page and extract it. It contains a standard install tree (`bin/`, `lib/`, `include/`):

```cmake
cmake -DLLVM_DIR=/path/to/extracted/lib/cmake/llvm ..
cmake -DMLIR_DIR=/path/to/extracted/lib/cmake/mlir ..

find_package(LLVM REQUIRED CONFIG)
find_package(MLIR REQUIRED CONFIG)
```

## Local Build Reproduction

### LLVM (Linux / macOS — shared library)
```bash
cmake -G Ninja -S llvm-project/llvm -B build \
  -DCMAKE_BUILD_TYPE=Release \
  -DLLVM_ENABLE_PROJECTS="clang;lld;polly" \
  -DLLVM_BUILD_LLVM_DYLIB=ON \
  -DLLVM_LINK_LLVM_DYLIB=ON \
  -DLLVM_TARGETS_TO_BUILD="all" \
  -DLLVM_INCLUDE_TESTS=OFF -DLLVM_INCLUDE_EXAMPLES=OFF \
  -DLLVM_INCLUDE_BENCHMARKS=OFF -DLLVM_INCLUDE_DOCS=OFF
```

### LLVM (Windows — static, clang-cl)
```bash
cmake -G Ninja -S llvm-project/llvm -B build ^
  -DCMAKE_C_COMPILER=clang-cl -DCMAKE_CXX_COMPILER=clang-cl ^
  -DLLVM_USE_LINKER=lld -DCMAKE_BUILD_TYPE=Release ^
  -DLLVM_ENABLE_PROJECTS="clang;lld;polly" ^
  -DLLVM_TARGETS_TO_BUILD="all" ^
  -DLLVM_INCLUDE_TESTS=OFF -DLLVM_INCLUDE_EXAMPLES=OFF ^
  -DLLVM_INCLUDE_BENCHMARKS=OFF -DLLVM_INCLUDE_DOCS=OFF
```

### MLIR (all platforms)
Same as LLVM but with `-DLLVM_ENABLE_PROJECTS="mlir" -DMLIR_BUILD_MLIR_C_DYLIB=ON`.
Add `LLVM_BUILD_LLVM_DYLIB=ON` and `LLVM_LINK_LLVM_DYLIB=ON` on Linux/macOS only.

### CIRCT (Linux / macOS — shared library)
```bash
cmake -G Ninja -S circt/llvm/llvm -B build \
  -DCMAKE_BUILD_TYPE=Release \
  -DLLVM_ENABLE_PROJECTS="mlir" \
  -DLLVM_EXTERNAL_PROJECTS=circt \
  -DLLVM_EXTERNAL_CIRCT_SOURCE_DIR=/path/to/circt \
  -DLLVM_BUILD_LLVM_DYLIB=ON -DLLVM_LINK_LLVM_DYLIB=ON \
  -DLLVM_TARGETS_TO_BUILD="host" \
  -DLLVM_INCLUDE_TESTS=OFF -DLLVM_INCLUDE_EXAMPLES=OFF \
  -DLLVM_INCLUDE_BENCHMARKS=OFF -DLLVM_INCLUDE_DOCS=OFF
```

### CIRCT (Windows — static, clang-cl)
```bash
cmake -G Ninja -S circt/llvm/llvm -B build ^
  -DCMAKE_C_COMPILER=clang-cl -DCMAKE_CXX_COMPILER=clang-cl ^
  -DLLVM_USE_LINKER=lld -DCMAKE_BUILD_TYPE=Release ^
  -DLLVM_ENABLE_PROJECTS="mlir" ^
  -DLLVM_EXTERNAL_PROJECTS=circt ^
  -DLLVM_EXTERNAL_CIRCT_SOURCE_DIR=C:\path\to\circt ^
  -DLLVM_TARGETS_TO_BUILD="host" ^
  -DLLVM_INCLUDE_TESTS=OFF -DLLVM_INCLUDE_EXAMPLES=OFF ^
  -DLLVM_INCLUDE_BENCHMARKS=OFF -DLLVM_INCLUDE_DOCS=OFF
```

## License

The LLVM, MLIR, and CIRCT libraries are distributed under the [Apache License v2.0 with LLVM Exceptions](https://llvm.org/LICENSE.txt).
