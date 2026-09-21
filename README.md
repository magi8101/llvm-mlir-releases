# llvm-mlir-releases

Automated, pre-built LLVM, Clang, MLIR, CIRCT, and libuv libraries distributed via GitHub Releases.

## What This Provides

This repository builds pre-compiled libraries from official sources using GitHub Actions.

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

4. **libuv Release** (e.g., `libuv-1.52.0-<platform>`)
   - libuv shared library (`uv.dll` / `libuv.so` / `libuv.dylib`) on all three platforms.
   - Unlike LLVM, Windows produces a proper DLL — no static fallback needed.
   - Includes both the shared library (`uv`) and the static library (`uv_a`), plus all headers and CMake config files.

5. **MLIR Python Bindings Release** (e.g., `mlir-python-bindings-<llvm-ref>`, Linux x64 only)
   - A Linux x64 wheel built with `MLIR_ENABLE_BINDINGS_PYTHON=ON`, plus the **NVPTX** and **AMDGPU** GPU backends.
   - Triggered manually via `workflow_dispatch` only (not tied to an LLVM release tag) — pick any `llvm_ref` (tag, branch, or commit SHA).
   - Meant for compiling MLIR `gpu` dialect kernels to PTX from Python, e.g. in Google Colab on a T4 GPU. See [Using the MLIR Python Bindings Wheel](#using-the-mlir-python-bindings-wheel-google-colab) below.

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

### libuv

Shared library on all platforms — MSVC ABI supports libuv DLLs natively:

| OS Platform | Compiler | Library Type | Archive |
|-------------|----------|-------------|---------|
| Ubuntu x64  | GCC | Shared (`libuv.so`) | `.tar.gz` |
| macOS ARM64 | AppleClang | Shared (`libuv.dylib`) | `.tar.gz` |
| Windows x64 | MSVC (cl.exe) | Shared (`uv.dll` + `uv.lib`) | `.zip` |

## Release Naming & Triggers

Releases are triggered automatically when tags are pushed:
- **LLVM + MLIR**: Push `llvm-v18.1.8` → both workflows run, attach output to the same Release.
- **CIRCT**: Push `circt-v1.140.0` → CIRCT workflow runs with its own Release page.
- **libuv**: Push `libuv-v1.52.0` → libuv workflow runs with its own Release page.

All workflows also support `workflow_dispatch` for manual builds with a version input.

## How to Consume

Download the archive for your OS from the [Releases](https://github.com/magi8101/llvm-mlir-releases/releases) page and extract it. It contains a standard install tree (`bin/`, `lib/`, `include/`):

```cmake
cmake -DLLVM_DIR=/path/to/extracted/lib/cmake/llvm ..
cmake -DMLIR_DIR=/path/to/extracted/lib/cmake/mlir ..

find_package(LLVM REQUIRED CONFIG)
find_package(MLIR REQUIRED CONFIG)
```

For libuv:
```cmake
cmake -Dlibuv_DIR=/path/to/extracted/lib/cmake/libuv ..

find_package(libuv REQUIRED CONFIG)
target_link_libraries(my_target PRIVATE libuv::libuv)
```

## Using the MLIR Python Bindings Wheel (Google Colab)

Trigger the `Build MLIR Python Bindings` workflow manually from the Actions tab (see below), then download the wheel from the run's workflow artifact (or from the matching `mlir-python-bindings-*` Release if `publish_release` was enabled). Upload it to your Colab session and install it:

```bash
# The wheel's top-level importable package is `mlir`, which collides with the
# PyPI `mlir` package and with a previously installed `mlir-python-bindings` —
# remove both first to avoid a stale/mixed install.
!pip uninstall -y mlir mlir-python-bindings
!pip install /content/mlir_python_bindings-<version>-cp3XX-cp3XX-linux_x86_64.whl
```

**The wheel's Python version must match Colab's runtime exactly.** The compiled `.so` extensions are built against a specific CPython ABI (not the stable/limited ABI), so a `cp311` wheel will fail to import (or, with the platform-tagged wheel this workflow produces, fail to *install*) on a `cp310` or `cp312` runtime. Check Colab's version first with `!python -V` and set the workflow's `python_version` input to match before triggering a build.

**Expected build time:** unverified — this workflow has not had a real run yet. It builds only the `MLIRPythonModules` target (not the full `all` target), so it should be faster than the full LLVM/Clang/MLIR release builds above, but LLVM/MLIR core plus the NVPTX and AMDGPU codegen backends still have to compile from scratch on a cache-cold run; budget at least an hour on a GitHub-hosted runner until a real run gives an actual number.

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

### libuv (all platforms)
```bash
cmake -G Ninja -S libuv -B build \
  -DCMAKE_BUILD_TYPE=Release \
  -DLIBUV_BUILD_SHARED=ON \
  -DBUILD_TESTING=OFF
```

Windows: same command with `^` line continuations. MSVC produces `uv.dll` + `uv.lib` without any special flags.

## License

The LLVM, MLIR, and CIRCT libraries are distributed under the [Apache License v2.0 with LLVM Exceptions](https://llvm.org/LICENSE.txt).
libuv is distributed under the [MIT License](https://github.com/libuv/libuv/blob/v1.x/LICENSE).
