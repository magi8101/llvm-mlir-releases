# llvm-mlir-releases

Pre-built LLVM and MLIR shared libraries distributed via GitHub Releases.

## What This Provides

Two separate sets of pre-compiled shared libraries, built from the official
[llvm-project](https://github.com/llvm/llvm-project) source:

1. **LLVM shared library** (`libLLVM.dll` / `libLLVM.so` / `libLLVM.dylib`)
   - Unified LLVM dynamic library containing all components
   - LLVM headers and CMake config files for `find_package(LLVM)`

2. **MLIR C API shared library** (`MLIR-C.dll` / `libMLIR-C.so` / `libMLIR-C.dylib`)
   - The MLIR C API dynamic library
   - MLIR headers and CMake config files for `find_package(MLIR)`

## Supported Platforms

| Platform | Compiler | Architecture |
|----------|----------|-------------|
| Windows  | Clang-cl | x86_64      |
| Linux    | GCC      | x86_64      |
| macOS    | AppleClang | ARM64 (Apple Silicon) |

## Release Naming

LLVM releases are tagged as `llvm-v<version>` (e.g., `llvm-v18.1.8`).
MLIR releases are tagged as `mlir-v<version>` (e.g., `mlir-v18.1.8`).

Each release contains platform-specific archives:

```
llvm-18.1.8-windows-x64.zip
llvm-18.1.8-linux-x64.tar.gz
llvm-18.1.8-macos-arm64.tar.gz

mlir-18.1.8-windows-x64.zip
mlir-18.1.8-linux-x64.tar.gz
mlir-18.1.8-macos-arm64.tar.gz
```

## Archive Contents

Each archive contains an install tree with:

- `lib/` -- Shared libraries and import libraries
- `include/` -- Headers
- `lib/cmake/` -- CMake package config files

## Usage

Download the archive for your platform from the
[Releases](https://github.com/magi8101/llvm-mlir-releases/releases) page,
extract it, and point your CMake build at it:

```cmake
cmake -DLLVM_DIR=/path/to/llvm/lib/cmake/llvm ..
cmake -DMLIR_DIR=/path/to/mlir/lib/cmake/mlir ..
```

## Building Locally

These libraries are built using GitHub Actions. To reproduce locally:

### LLVM

```bash
git clone --depth 1 --branch llvmorg-18.1.8 https://github.com/llvm/llvm-project.git
cmake -G Ninja -S llvm-project/llvm -B build-llvm \
  -DCMAKE_BUILD_TYPE=Release \
  -DCMAKE_INSTALL_PREFIX=./install-llvm \
  -DCMAKE_C_COMPILER=clang-cl \
  -DCMAKE_CXX_COMPILER=clang-cl \
  -DLLVM_USE_LINKER=lld \
  -DLLVM_ENABLE_PROJECTS="mlir" \
  -DLLVM_BUILD_LLVM_DYLIB=ON \
  -DLLVM_LINK_LLVM_DYLIB=ON \
  -DLLVM_TARGETS_TO_BUILD="all" \
  -DLLVM_INCLUDE_TESTS=OFF \
  -DLLVM_INCLUDE_EXAMPLES=OFF \
  -DLLVM_INCLUDE_BENCHMARKS=OFF \
  -DLLVM_INCLUDE_DOCS=OFF
cmake --build build-llvm --target install
```

### MLIR C API

```bash
cmake -G Ninja -S llvm-project/llvm -B build-mlir \
  -DCMAKE_BUILD_TYPE=Release \
  -DCMAKE_INSTALL_PREFIX=./install-mlir \
  -DCMAKE_C_COMPILER=clang-cl \
  -DCMAKE_CXX_COMPILER=clang-cl \
  -DLLVM_USE_LINKER=lld \
  -DLLVM_ENABLE_PROJECTS="mlir" \
  -DLLVM_BUILD_LLVM_DYLIB=ON \
  -DLLVM_LINK_LLVM_DYLIB=ON \
  -DMLIR_BUILD_MLIR_C_DYLIB=ON \
  -DLLVM_TARGETS_TO_BUILD="all" \
  -DLLVM_INCLUDE_TESTS=OFF \
  -DLLVM_INCLUDE_EXAMPLES=OFF \
  -DLLVM_INCLUDE_BENCHMARKS=OFF \
  -DLLVM_INCLUDE_DOCS=OFF
cmake --build build-mlir --target install
```

## License

The LLVM and MLIR libraries are distributed under the
[Apache License v2.0 with LLVM Exceptions](https://llvm.org/LICENSE.txt).
