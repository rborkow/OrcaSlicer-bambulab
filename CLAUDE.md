# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Overview

OrcaSlicer is an open-source 3D slicer application forked from Bambu Studio, built using C++ with wxWidgets for the GUI and CMake as the build system. The project uses a modular architecture with separate libraries for core slicing functionality, GUI components, and platform-specific code.

## Build Commands

### Building on Windows
**Always use this command to build the project when testing build issues on Windows.**
```bash
cmake --build . --config %build_type% --target ALL_BUILD -- -m
```

### Building on macOS
**Always use this command to build the project when testing build issues on macOS.**
```bash
cmake --build build/arm64 --config RelWithDebInfo --target all --
```

### Building on Linux
 **Always use this command to build the project when testing build issues on Linux.**
```bash
cmake --build build/arm64 --config RelWithDebInfo --target all --

```
### Build test:

**Always use this command to build the project when testing build issues on Windows.**
```bash
cmake --build . --config %build_type% --target ALL_BUILD -- -m
```

### Building on macOS
**Always use this command to build the project when testing build issues on macOS.**
```bash
cmake --build build/arm64 --config RelWithDebInfo --target all --
```

### Building on Linux
 **Always use this command to build the project when testing build issues on Linux.**
```bash
cmake --build build --config RelWithDebInfo --target all --

```


### Build System
- Uses CMake with minimum version 3.13 (maximum 3.31.x on Windows)
- Primary build directory: `build/`
- Dependencies are built in `deps/build/`
- The build process is split into dependency building and main application building
- Windows builds use Visual Studio generators
- macOS builds use Xcode by default, Ninja with -x flag
- Linux builds use Ninja generator

### Testing
Tests are located in the `tests/` directory and use the Catch2 testing framework. Test structure:
- `tests/libslic3r/` - Core library tests (21 test files)
  - Geometry processing, algorithms, file formats (STL, 3MF, AMF)
  - Polygon operations, clipper utilities, Voronoi diagrams
- `tests/fff_print/` - Fused Filament Fabrication tests (12 test files)
  - Slicing algorithms, G-code generation, print mechanics
  - Fill patterns, extrusion, support material
- `tests/sla_print/` - Stereolithography tests (4 test files)
  - SLA-specific printing algorithms, support generation
- `tests/libnest2d/` - 2D nesting algorithm tests
- `tests/slic3rutils/` - Utility function tests
- `tests/sandboxes/` - Experimental/sandbox test code

Run all tests after building:
```bash
cd build && ctest
```

Run tests with verbose output:
```bash
cd build && ctest --output-on-failure
```

Run individual test suites:
```bash
# From build directory
ctest --test-dir ./tests/libslic3r/libslic3r_tests
ctest --test-dir ./tests/fff_print/fff_print_tests
ctest --test-dir ./tests/sla_print/sla_print_tests
# and so on
```

## Architecture

### Core Libraries
- **libslic3r/**: Core slicing engine and algorithms (platform-independent)
  - Main slicing logic, geometry processing, G-code generation
  - Key classes: Print, PrintObject, Layer, GCode, Config
  - Modular design with specialized subdirectories:
    - `GCode/` - G-code generation, cooling, pressure equalization, thumbnails
    - `Fill/` - Infill pattern implementations (gyroid, honeycomb, lightning, etc.)
    - `Support/` - Tree supports and traditional support generation
    - `Geometry/` - Advanced geometry operations, Voronoi diagrams, medial axis
    - `Format/` - File I/O for 3MF, AMF, STL, OBJ, STEP formats
    - `SLA/` - SLA-specific print processing and support generation
    - `Arachne/` - Advanced wall generation using skeletal trapezoidation

- **src/slic3r/**: Main application framework and GUI
  - GUI application built with wxWidgets
  - Integration between libslic3r core and user interface
  - Located in `src/slic3r/GUI/` (not shown in this directory but exists)

### Key Algorithmic Components
- **Arachne Wall Generation**: Variable-width perimeter generation using skeletal trapezoidation
- **Tree Supports**: Organic support generation algorithm  
- **Lightning Infill**: Sparse infill optimization for internal structures
- **Adaptive Slicing**: Variable layer height based on geometry
- **Multi-material**: Multi-extruder and soluble support processing
- **G-code Post-processing**: Cooling, fan control, pressure advance, conflict checking

### File Format Support
- **3MF/BBS_3MF**: Native format with extensions for multi-material and metadata
- **STL**: Standard tessellation language for 3D models
- **AMF**: Additive Manufacturing Format with color/material support  
- **OBJ**: Wavefront OBJ with material definitions
- **STEP**: CAD format support for precise geometry
- **G-code**: Output format with extensive post-processing capabilities

### External Dependencies
- **Clipper2**: Advanced 2D polygon clipping and offsetting
- **libigl**: Computational geometry library for mesh operations
- **TBB**: Intel Threading Building Blocks for parallelization
- **wxWidgets**: Cross-platform GUI framework
- **OpenGL**: 3D graphics rendering and visualization
- **CGAL**: Computational Geometry Algorithms Library (selective use)
- **OpenVDB**: Volumetric data structures for advanced operations
- **Eigen**: Linear algebra library for mathematical operations

## File Organization

### Resources and Configuration
- `resources/profiles/` - Printer and material profiles organized by manufacturer
- `resources/printers/` - Printer-specific configurations and G-code templates  
- `resources/images/` - UI icons, logos, calibration images
- `resources/calib/` - Calibration test patterns and data
- `resources/handy_models/` - Built-in test models (benchy, calibration cubes)

### Internationalization and Localization  
- `localization/i18n/` - Source translation files (.pot, .po)
- `resources/i18n/` - Runtime language resources
- Translation managed via `scripts/run_gettext.sh` / `scripts/run_gettext.bat`

### Platform-Specific Code
- `src/libslic3r/Platform.cpp` - Platform abstractions and utilities
- `src/libslic3r/MacUtils.mm` - macOS-specific utilities (Objective-C++)
- Windows-specific build scripts and configurations
- Linux distribution support scripts in `scripts/linux.d/`

### Build and Development Tools
- `cmake/modules/` - Custom CMake find modules and utilities
- `scripts/` - Python utilities for profile generation and validation  
- `tools/` - Windows build tools (gettext utilities)
- `deps/` - External dependency build configurations

## Development Workflow

### Code Style and Standards
- **C++17 standard** with selective C++20 features
- **Naming conventions**: PascalCase for classes, snake_case for functions/variables
- **Header guards**: Use `#pragma once` 
- **Memory management**: Prefer smart pointers, RAII patterns
- **Thread safety**: Use TBB for parallelization, be mindful of shared state

### Common Development Tasks

#### Adding New Print Settings
1. Define setting in `PrintConfig.cpp` with proper bounds and defaults
2. Add UI controls in appropriate GUI components  
3. Update serialization in config save/load
4. Add tooltips and help text for user guidance
5. Test with different printer profiles

#### Modifying Slicing Algorithms  
1. Core algorithms live in `libslic3r/` subdirectories
2. Performance-critical code should be profiled and optimized
3. Consider multi-threading implications (TBB integration)
4. Validate changes don't break existing profiles
5. Add regression tests where appropriate

#### GUI Development
1. GUI code resides in `src/slic3r/GUI/` (not visible in current tree)
2. Use existing wxWidgets patterns and custom controls
3. Support both light and dark themes
4. Consider DPI scaling on high-resolution displays
5. Maintain cross-platform compatibility

#### Adding Printer Support
1. Create JSON profile in `resources/profiles/[manufacturer].json`
2. Add printer-specific start/end G-code templates
3. Configure build volume, capabilities, and material compatibility
4. Test thoroughly with actual hardware when possible
5. Follow existing profile structure and naming conventions

### Dependencies and Build System
- **CMake-based** with separate dependency building phase
- **Dependencies** built once in `deps/build/`, then linked to main application  
- **Cross-platform** considerations important for all changes
- **Resource files** embedded at build time, platform-specific handling

### Performance Considerations
- **Slicing algorithms** are CPU-intensive, profile before optimizing
- **Memory usage** can be substantial with complex models
- **Multi-threading** extensively used via TBB
- **File I/O** optimized for large 3MF files with embedded textures
- **Real-time preview** requires efficient mesh processing

## Important Development Notes

### Codebase Navigation
- Use search tools extensively - codebase has 500k+ lines
- Key entry points: `src/OrcaSlicer.cpp` for application startup
- Core slicing: `libslic3r/Print.cpp` orchestrates the slicing pipeline
- Configuration: `PrintConfig.cpp` defines all print/printer/material settings

### Compatibility and Stability
- **Backward compatibility** maintained for project files and profiles
- **Cross-platform** support essential (Windows/macOS/Linux)  
- **File format** changes require careful version handling
- **Profile migrations** needed when settings change significantly

### Quality and Testing
- **Regression testing** important due to algorithm complexity
- **Performance benchmarks** help catch performance regressions
- **Memory leak** detection important for long-running GUI application
- **Cross-platform** testing required before releases

## Fork Purpose: Bambu Lab Connectivity

This fork (`OrcaSlicer-bambulab`) preserves the original Bambu Lab network stack
that upstream OrcaSlicer removed/gated. The Bambu sources are compiled
unconditionally and exposed alongside the Orca cloud agent through a factory so
users can pick which provider to use at runtime via `AppConfig`
(`use_orca_cloud`).

### Component map

- **Plugin loader** — `src/slic3r/Utils/BBLNetworkPlugin.{hpp,cpp}`
  Loads the proprietary `BambuNetwork_<ver>` DLL/dylib/so at runtime via
  `GetProcAddress` / `dlsym`. Entry point: `GUI_App.cpp` ≈ L3780–3818
  (`NetworkAgent::initialize_network_module`), gated by the
  `installed_networking` config flag (≈ L1512, L3725).
- **Cloud agent** — `src/slic3r/Utils/BBLCloudServiceAgent.{cpp,hpp}`
  Wraps the plugin's auth/session/cert/log calls and supplies the OAuth login
  URL. Bambu endpoints (`api.bambulab.com` / `api.bambulab.cn`) are referenced
  in `GUI_App.cpp` ≈ L1145, L1160.
- **Printer agent** — `src/slic3r/Utils/BBLPrinterAgent.{cpp,hpp}`
  Routes print jobs through plugin entry points (`start_print`,
  `start_local_print`, `start_sdcard_print`).
- **Tunnel/protocol** — `src/slic3r/GUI/Printer/BambuTunnel.h` plus
  `PrinterFileSystem.{h,cpp}` (inherits `BambuLib`). Binary tunnel handles
  MQTT-style control, file ops, and camera/video streams.
- **Factory** — `src/slic3r/Utils/NetworkAgentFactory.{hpp,cpp}`
  `enum CloudAgentProvider { Orca, BBL }`. `create_cloud_agent()` returns BBL
  if the plugin loaded, else falls back to Orca.
  `create_agent_from_config()` reads `use_orca_cloud`; WSL2 + Linux bridge
  forces BBL.
- **WSL2 / Linux bridge forwarder** —
  `src/slic3r/Utils/PJarczakLinuxBridge/PJarczakBambuNetworkForwarderExports.cpp`
  Lets Windows builds reach the Linux-only plugin through a helper process;
  fallback host `https://bambulab.com`.
- **Build wiring** — `src/slic3r/CMakeLists.txt` ≈ L41–47, L618–623 compiles
  the BBL sources unconditionally (no `#ifdef DISABLE_BBL`-style gates).

### What "re-enabling" actually means here

There is no protocol rewrite. The fork:
1. Keeps the BBL source files compiled.
2. Keeps the `BambuNetwork` plugin loader + entry-point bindings.
3. Adds a factory layer so BBL and Orca agents can coexist and be selected at
   runtime.
4. Adds the `PJarczakLinuxBridge` forwarder so Windows builds reach the
   Linux plugin via WSL2.

The proprietary `BambuNetwork_*` binary itself is **not** in this repo — it is
loaded from `plugins/` at runtime. Authentication/MQTT/FTP logic lives inside
that DLL; this codebase only binds to its exported symbols.

### Git history caveat

The repo currently has only two commits (`Initial release` + a CI fix), so
there is no incremental diff against upstream to inspect. Treat the "Component
map" above as the inventory of files that an agent should diff against
upstream `SoftFever/OrcaSlicer` when asked "what was changed?".

## Agent Ergonomics

Tips for agents (and humans) working in this repo:

- **Develop on `claude/orcaslicer-bambu-connection-30m4d`.** All pushes go to
  that branch; do not push to `main` without explicit permission.
- **GitHub access is restricted to `rborkow/orcaslicer-bambulab`.** Use the
  `mcp__github__*` tools — there is no `gh` CLI in this environment.
- **Don't run full builds speculatively.** A full CMake build of this tree is
  multi-hour and downloads dependencies in `deps/build/`. Only build when
  diagnosing an actual build issue, and follow the per-platform commands
  above.
- **For codebase exploration, prefer the `Explore` agent.** The tree is
  ~500k LOC; a thorough `Explore` run is faster and uses less main-context
  than ad-hoc `grep`/`find` loops.
- **Bambu connection investigations start at the Component map above.** Begin
  at `BBLNetworkPlugin`, then `NetworkAgentFactory`, then the specific agent
  (`BBLCloudServiceAgent` for auth/cloud, `BBLPrinterAgent` for jobs,
  `PrinterFileSystem`/`BambuTunnel` for device I/O).
- **Don't try to run or test connectivity end-to-end** — it requires the
  proprietary `BambuNetwork_*` plugin binary which is not in this repo, plus
  real Bambu credentials and (on Windows) a WSL2 Linux bridge.
- **CMake quirks**: minimum 3.13, capped at 3.31.x on Windows; Linux uses
  Ninja; macOS defaults to Xcode (Ninja with `-x`). Build artifacts land in
  `build/` (or `build/arm64/` on macOS/Linux per the commands above).
- **Test discovery**: tests are Catch2 under `tests/`; run with `ctest` from
  the build dir. Don't add new test directories without wiring them into
  `tests/CMakeLists.txt`.
- **Avoid touching `resources/profiles/`** unless the task is explicitly
  about printer/material profiles — accidental edits there break
  user-visible printer support.
- **The `installed_networking` AppConfig flag** is the master switch for
  loading the BBL plugin; if connectivity is silently disabled in a session,
  check that flag and the presence of the plugin binary under `plugins/`
  before chasing code paths.
