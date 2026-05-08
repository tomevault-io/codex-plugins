# Ariel LV2 Host - AI Coding Agent Instructions

## Project Overview
Ariel is a cross-platform LV2 plugin host built with GTK4 and lilv for real-time audio processing. The application provides a modern interface for loading and managing LV2 plugins with JACK audio backend support.

**Core Technologies**: C, GTK4, lilv (LV2), JACK Audio, Meson build system

## Architecture & Structure
```
/
├── src/
│   ├── main.c              # Application entry point & GTK setup
│   ├── ui/                 # User interface components
│   │   ├── window.c        # Main window & layout management
│   │   ├── plugin_list.c   # LV2 plugin browser with GtkListView
│   │   ├── mixer.c         # Mixer channels with volume/pan controls
│   │   └── transport.c     # Header bar & audio engine toggle
│   └── audio/              # Audio engine & plugin management
│       ├── engine.c        # JACK client lifecycle management
│       ├── jack_client.c   # JACK process callback & shutdown handling
│       └── plugin_manager.c # lilv integration & LV2 plugin discovery
├── include/ariel.h         # Main header with all structure definitions
├── data/                   # Desktop files & application metadata
└── meson.build            # Cross-platform build configuration
```

## Development Workflow
```bash
# Install dependencies (Ubuntu/Debian)
sudo apt install libgtk-4-dev liblilv-dev libjack-jackd2-dev meson

# Setup build
meson setup builddir

# Compile (after any changes)
meson compile -C builddir

# Run application
./builddir/ariel

# Clean rebuild
rm -rf builddir && meson setup builddir && meson compile -C builddir
```

## Key Architectural Patterns

### Main Data Structures (see `include/ariel.h`)
- **ArielApp**: GtkApplication subclass holding audio engine & plugin manager references
- **ArielWindow**: Main UI container with paned layout (plugin list | active plugins + mixer)
- **ArielAudioEngine**: JACK client wrapper with stereo I/O ports
- **ArielPluginManager**: lilv world container managing LV2 plugin discovery

### UI Component Creation Pattern
Each UI component follows the pattern: `ariel_create_*()` returns GtkWidget, takes ArielWindow parameter
- Components self-manage their callbacks and internal structure
- Use GTK4 modern widgets (GtkListView, GtkPaned, GtkScale)
- CSS classes for consistent styling (`suggested-action`, `destructive-action`, etc.)

### Audio Engine Integration
- JACK client lifecycle managed in `src/audio/engine.c`
- Real-time audio processing in `ariel_jack_process_callback()`
- Currently implements pass-through; plugin processing goes here
- Engine start/stop controlled via header bar toggle button

## Critical Code Patterns

### Plugin Discovery (lilv integration)
```c
// Initialize lilv world and load all plugins
manager->world = lilv_world_new();
lilv_world_load_all(manager->world);
manager->plugins = lilv_world_get_all_plugins(manager->world);
```

### GTK4 List View Setup Pattern
```c
// Factory for list item creation
factory = gtk_signal_list_item_factory_new();
g_signal_connect(factory, "setup", G_CALLBACK(setup_plugin_list_item), NULL);
g_signal_connect(factory, "bind", G_CALLBACK(bind_plugin_list_item), NULL);
```

### JACK Audio Processing Structure
- Register stereo input/output ports during engine start
- Process callback receives buffer pointers for each port
- Zero-copy audio processing - modify buffers in-place

## External Dependencies & Integration
- **GTK4**: Modern widget toolkit, requires >= 4.0
- **lilv**: LV2 plugin host library, handles plugin discovery/instantiation  
- **JACK**: Real-time audio server, fallback to PipeWire-JACK if needed
- **Meson**: Build system with automatic dependency detection

## Current Implementation Status
- ✅ Basic GTK4 application structure & main window
- ✅ JACK audio engine with stereo I/O
- ✅ LV2 plugin discovery via lilv
- ✅ UI layout with paned views and mixer placeholder
- 🚧 Plugin loading & parameter control (TODO)
- 🚧 Drag & drop functionality (TODO)
- 🚧 Transport controls implementation (TODO)

## Development Notes
- Use `g_print()` for debug output, visible in terminal
- Audio processing must be real-time safe (no malloc, no GTK calls in JACK callback)
- Plugin UI parameters need separate thread communication with audio thread
- Cross-platform: Test build system detects JACK vs PipeWire-JACK automatically

---
> Source: [djshaji/ariel](https://github.com/djshaji/ariel) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:agents_md:2026-05-08 -->
