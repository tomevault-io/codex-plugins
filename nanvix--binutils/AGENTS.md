# Project Overview

This project is a fork of GNU Binutils specifically tailored for the Nanvix
operating system. It provides essential tools for binary manipulation, assembly,
linking, and debugging, enabling developers to create and manage binaries for
Nanvix.

## Target Architecture Support

This fork introduces the following new target architectures and operating system support:
- `i686-nanvix` (32-bit x86 for Nanvix)
- `x86_64-nanvix` (64-bit x86 for Nanvix)

## Key Components

This repository contains the source code of the following development utilities:
- `ar` - Archive Manager for creating and manipulating library archives
- `as` - GNU Assembler for converting assembly language to object code
- `gdb` - GNU Debugger for debugging applications
- `gprof` - Performance Profiler for analyzing program performance
- `ld` - GNU Linker for combining object files into executables
- `nm` - Symbol Table Viewer for examining symbol tables in object files
- `objcopy` - Object File Converter for copying and translating object files
- `objdump` - Object File Disassembler for displaying information about object files
- `readelf` - ELF File Analyzer for displaying information about ELF files
- `size` - Section Size Reporter for listing section sizes in object files
- `strings` - String Extractor for finding printable strings in files
- `strip` - Symbol Stripper for removing symbols from object files

## Building and Installation

This project uses a custom `z` utility script for streamlined building and installation.

### Quick Start

```bash
# See all available commands
./z help

# Complete build process
./z setup      # Install required system packages
./z configure  # Configure the build
./z build      # Compile the source code
./z install    # Install to default location
./z release    # Create a release archive
```

## Contributing Guidelines

When making changes:
- **Target Compatibility**: Ensure all changes maintain compatibility with the `i686-nanvix` and `x86_64-nanvix` targets
- **Testing**: Test changes thoroughly using the build and install process
- **Code Style**: Follow the established GNU coding standards.
- **Documentation**: Update relevant documentation and comments
- **Upstream Compatibility**: Consider impact on future upstream merges from GNU Binutils

### Code Style and Standards

This project follows GNU coding standards and conventions.

---
> Source: [nanvix/binutils](https://github.com/nanvix/binutils) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:agents_md:2026-06-07 -->
