# StarRocks macOS Build System

This directory contains scripts and configurations for building StarRocks on macOS (especially Apple Silicon/ARM64).

## Quick Start

### Building Third-party Dependencies

```bash
cd build-mac
./build_thirdparty.sh
```

### Build Options

```bash
# Show help
./build_thirdparty.sh --help

# Build with clean start (removes all previous builds)
./build_thirdparty.sh --clean

# Build with specific number of parallel jobs
./build_thirdparty.sh --parallel 8

# Install only Homebrew dependencies
./build_thirdparty.sh --homebrew-only

# Build only source dependencies (skip Homebrew)
./build_thirdparty.sh --source-only
```

## Cleaning Build Artifacts

### Full Clean (All Third-party Libraries)

To clean **all** third-party build artifacts and start fresh:

```bash
./build_thirdparty.sh --clean
```

This removes:
- `../thirdparty/build/` - All build directories
- `../thirdparty/installed/` - All installed libraries

### Partial Clean (Specific Library)

If you need to rebuild only a specific library (e.g., brpc), you can manually remove its build directory:

#### Cleaning brpc

```bash
# Remove brpc build directory
rm -rf ../thirdparty/build/brpc

# Remove brpc installed files (optional, for complete clean)
rm -f ../thirdparty/installed/lib/libbrpc.a
rm -f ../thirdparty/installed/lib64/libbrpc.a
rm -rf ../thirdparty/installed/include/brpc
```

After cleaning, rebuild:

```bash
./build_thirdparty.sh
```

#### Cleaning Other Libraries

The same pattern applies to other libraries. Build directories are located at:
- `../thirdparty/build/<library-name>/`

For example:
- `../thirdparty/build/protobuf/`
- `../thirdparty/build/glog/`
- `../thirdparty/build/rocksdb/`

## Common Build Issues on macOS

### 1. Invalid Mach-O File Errors

If you see errors like:
```
file is not of required architecture
ld: warning: ignoring file libXXX.a, file was built for unsupported file format
```

**Solution**: Clean and rebuild the problematic library
```bash
# Example for brpc
rm -rf ../thirdparty/build/brpc
./build_thirdparty.sh
```

### 2. Linker Errors with Multiple Libraries

If multiple libraries show Mach-O errors, it's usually better to do a full clean:

```bash
./build_thirdparty.sh --clean
```

### 3. Version Conflicts

If you have libraries installed via Homebrew that conflict with the build:

```bash
# Rebuild only from source, skipping Homebrew
./build_thirdparty.sh --clean --source-only
```

## Directory Structure

```
build-mac/
├── build_thirdparty.sh    # Main build script for third-party libraries
├── build_be.sh            # StarRocks Backend build script  
├── env_macos.sh           # macOS environment configuration
├── CMakeLists.txt         # CMake configuration for macOS
├── patch/                 # Patches for third-party libraries
└── shims/                 # Compatibility shims

../thirdparty/
├── build/                 # Build directories (generated)
│   ├── brpc/             # brpc build artifacts
│   ├── protobuf/         # protobuf build artifacts
│   └── ...               # Other libraries
├── installed/            # Installed libraries (generated)
│   ├── lib/              # Installed libraries
│   ├── include/          # Installed headers
│   └── bin/              # Installed binaries
├── src/                  # Downloaded source archives (generated)
└── patches/              # Patches for third-party libraries
```

## Frequently Asked Questions

### Q: Does cleaning brpc mean deleting thirdparty/build/brpc directory?

**A: Yes.** To clean brpc specifically, delete the `../thirdparty/build/brpc` directory. The build script will automatically rebuild it on the next run.

For a complete brpc clean (including installed files):
```bash
rm -rf ../thirdparty/build/brpc
rm -f ../thirdparty/installed/lib/libbrpc.a
rm -f ../thirdparty/installed/lib64/libbrpc.a
rm -rf ../thirdparty/installed/include/brpc
```

### Q: What's the difference between build/ and installed/ directories?

- `build/` - Contains build artifacts, object files, and temporary files created during compilation
- `installed/` - Contains the final compiled libraries, headers, and binaries that are used by StarRocks

### Q: Can I delete the src/ directory?

The `src/` directory contains downloaded source archives. You can delete it, but the script will re-download them on the next run. It's generally safe to keep it.

### Q: How do I force a rebuild of a specific library?

1. Delete the library's build directory: `rm -rf ../thirdparty/build/<library-name>`
2. Optionally delete installed files: `rm -f ../thirdparty/installed/lib/lib<library-name>.a`
3. Run the build script: `./build_thirdparty.sh`

## Environment Variables

- `STARROCKS_THIRDPARTY` - Override third-party directory location
- `PARALLEL_JOBS` - Set via `--parallel N` flag

## See Also

- Main third-party build script: `../thirdparty/build-thirdparty.sh` (for Linux)
- Main README: `../README.md`
- Contributing guide: `../CONTRIBUTING.md`
