# OfficeRadio

**ALWAYS follow these instructions first and fallback to additional search and context gathering only if the information in these instructions is incomplete or found to be in error.**

OfficeRadio is a C# Windows Forms desktop application for managing Chromecast-based office radio streaming. The application allows users to discover Chromecast devices on the local network and stream various radio stations to them.

## Working Effectively

### Prerequisites and Setup
- **CRITICAL**: This application requires .NET 9.0 SDK. The project targets `net9.0-windows` and cannot be built with earlier SDK versions.
- Install .NET 9.0 SDK using the official installer:
  ```bash
  wget -O /tmp/dotnet-install.sh https://dot.net/v1/dotnet-install.sh
  chmod +x /tmp/dotnet-install.sh
  /tmp/dotnet-install.sh --channel STS
  export PATH="/home/runner/.dotnet:$PATH"
  ```
- **Windows Development**: This application is designed for Windows and uses Windows Forms. While it can be built on Linux with special configuration, it can only run properly on Windows.

### Build Process
- **NEVER CANCEL**: All build operations complete in under 5 seconds. Set timeout to 60+ seconds for safety.
- **Restore packages**: 
  ```bash
  dotnet restore -p:EnableWindowsTargeting=true
  ```
  - Takes ~4 seconds. NEVER CANCEL.
- **Build the application**:
  ```bash
  dotnet build -p:EnableWindowsTargeting=true
  ```
  - Takes ~4-5 seconds. NEVER CANCEL.
- **Linux/macOS users**: MUST include `-p:EnableWindowsTargeting=true` for all dotnet commands due to Windows Forms dependency.

### Testing
- **No automated tests**: This repository does not contain unit tests or integration tests.
- **Manual testing required**: Testing must be performed on Windows with actual Chromecast devices.
- **Build validation**: Use `dotnet build -p:EnableWindowsTargeting=true` to verify code compiles without errors.

### Code Linting and Formatting
- **Code formatting**: 
  ```bash
  dotnet format --verbosity diagnostic
  ```
  - **FAILS on Linux/macOS**: dotnet format cannot load the workspace due to Windows targeting requirements. This is expected and cannot be worked around.
- **Code analysis**: Build warnings are enabled and should be addressed. Common warnings include nullability issues (CS8618, CS8622, CS8625).

## Validation Scenarios

### After Making Changes
1. **ALWAYS build and verify compilation**:
   ```bash
   dotnet build -p:EnableWindowsTargeting=true
   ```
2. **Check for new compiler warnings** - address nullability warnings for better code quality.
3. **Manual testing on Windows**: 
   - Run the application: `dotnet run --project Caster -p:EnableWindowsTargeting=true`
   - Verify Chromecast device discovery works
   - Test radio station streaming functionality
   - Validate volume control and playback controls

### Key Validation Points
- **Device Discovery**: The app should discover Chromecast devices on the local network within 30 seconds
- **Radio Streaming**: Test both KINK and Qmusic Non-Stop radio stations
- **UI Responsiveness**: Ensure async operations don't block the UI thread
- **Error Handling**: Verify proper error messages for network issues or missing devices

## Project Structure and Navigation

### Repository Layout
```
/
├── Caster.sln              # Visual Studio solution file
├── Caster/                 # Main application project
│   ├── Caster.csproj       # Project file targeting net9.0-windows
│   ├── Program.cs          # Application entry point
│   ├── Form1.cs            # Main Windows Form UI logic
│   ├── Form1.Designer.cs   # Auto-generated UI layout code
│   ├── Form1.resx          # Form resources
│   ├── CastingService.cs   # Chromecast device discovery and management
│   └── RadioCastingHelper.cs # Radio station definitions and media helpers
├── README.md               # Basic project description
└── .github/
    └── copilot-instructions.md # This file
```

### Key Files and Their Responsibilities
- **Form1.cs**: Main UI logic, device selection, playback controls
- **CastingService.cs**: Contains `ChromeCastDevice` class and `CastingService` for device discovery
- **RadioCastingHelper.cs**: Helper methods for creating radio station media objects
- **Program.cs**: Standard Windows Forms application entry point

### Important Code Patterns
- **Async/Await**: All network operations use async/await pattern to prevent UI blocking
- **Event Handling**: Device status changes trigger UI updates via event handlers
- **Dependency Injection**: Minimal DI pattern using constructor injection for services

## Dependencies and Libraries

### NuGet Packages
- **Sharpcaster (2.0.3)**: Primary library for Chromecast communication
- **Tmds.MDns**: Multicast DNS for device discovery
- **Google.Protobuf**: Protocol buffer support required by Sharpcaster
- **Microsoft.Extensions.DependencyInjection**: Lightweight DI container

### Target Framework
- **.NET 9.0-windows**: Requires Windows Forms support, cannot target standard .NET 9.0

## Common Development Tasks

### Adding New Radio Stations
1. **Edit RadioCastingHelper.cs**:
   - Add new static method following the pattern of `GetKinkMedia()` and `GetQMusicNonStopMedia()`
   - Use `GetLiveRadioMedia()` helper for consistent media object creation
2. **Update Form1.cs**: Add new button and event handler to load the new station

### Modifying UI Elements
1. **Use Visual Studio Designer**: Open Form1.cs in Visual Studio and use the designer for layout changes
2. **Manual code changes**: Edit Form1.Designer.cs for programmatic UI modifications
3. **Event handlers**: Add new event handlers in Form1.cs, following async/await pattern

### Debugging Connection Issues
1. **Check network connectivity**: Ensure Chromecast and development machine are on same network
2. **Verify device discovery**: Check CastingService.DiscoverReceiversAsync() method
3. **Review error handling**: Add try-catch blocks around network operations

## Build Output and Artifacts

### Build Output Location
- **Debug builds**: `Caster/bin/Debug/net9.0-windows/`
- **Main executable**: `Caster.exe` (on Windows)
- **Required DLLs**: Sharpcaster.dll, Google.Protobuf.dll, Tmds.MDns.dll, and Microsoft.Extensions.* libraries

### Common Build Issues
- **NETSDK1100 Error**: Missing EnableWindowsTargeting property - add `-p:EnableWindowsTargeting=true` to dotnet commands
- **Missing .NET 9.0**: Install .NET 9.0 SDK using the installation script provided above
- **Nullability warnings**: Expected due to nullable reference types being enabled - address for cleaner code

## Platform-Specific Notes

### Windows Development
- **Recommended**: Use Visual Studio 2022 with .NET 9.0 workload installed
- **Build command**: `dotnet build` (no special flags needed)
- **Run command**: `dotnet run --project Caster`

### Linux/macOS Development  
- **Limited support**: Can build but cannot run due to Windows Forms dependency and missing Microsoft.WindowsDesktop.App runtime
- **Required flag**: Must use `-p:EnableWindowsTargeting=true` for all dotnet commands
- **Testing limitation**: Cannot test actual functionality without Windows environment
- **Format tool limitation**: `dotnet format` does not work on Linux/macOS for this project

## Timing Expectations

- **Package restore**: 1-2 seconds. NEVER CANCEL builds - set 60+ second timeout.
- **Full build**: 1-2 seconds. NEVER CANCEL builds - set 60+ second timeout.
- **Device discovery**: 5-30 seconds depending on network
- **Radio stream connection**: 2-5 seconds for successful connection

**CRITICAL**: NEVER CANCEL build operations. They complete quickly but may appear to hang briefly during package resolution.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/wdko)
> Context snippets also available to append to your CLAUDE.md, GEMINI.md, and copilot-instructions.md — [download at TomeVault](https://tomevault.io/claim/wdko)
<!-- tomevault:4.0:agents_md:2026-04-09 -->
