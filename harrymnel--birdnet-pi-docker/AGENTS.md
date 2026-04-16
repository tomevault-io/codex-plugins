# BirdNET-Pi AI Agent Instructions

## Project Overview
This is a **containerized BirdNET-Pi installation** for automated bird sound detection and identification. The system uses Docker to run the BirdNET AI model, processes RTSP audio streams, and provides a web interface for monitoring bird activity.

## Architecture & Data Flow
- **Core Container**: `ghcr.io/alexbelgium/birdnet-pi-amd64` runs BirdNET analysis engine
- **Audio Pipeline**: RTSP stream → BirdNET analysis → species detection → data storage
- **Data Storage**: `/config/BirdSongs/` (recordings organized by date, charts, processed files)
- **Configuration**: Centralized in `/config/birdnet.conf` with 270+ parameters
- **Web Interface**: Accessible on port 8001 with password protection

## Key Configuration Patterns

### Primary Config File: `config/birdnet.conf`
Critical sections to understand:
- **Geolocation**: `LATITUDE`/`LONGITUDE` (required for species accuracy)
- **Audio Source**: `RTSP_STREAM` (replaces microphone recording)
- **Detection Thresholds**: `CONFIDENCE` (0.7), `SENSITIVITY` (1.25), `SF_THRESH` (0.03)
- **Model Settings**: `MODEL=BirdNET_GLOBAL_6K_V2.4_Model_FP16`
- **Data Management**: `FULL_DISK=purge`, `PURGE_THRESHOLD=95`

### Directory Structure
```
config/
├── birdnet.conf          # Main configuration (270+ parameters)
├── BirdSongs/           # Audio data storage
│   ├── By_Date/         # Daily organized recordings  
│   └── Charts/          # Detection visualizations
├── TemporaryFiles/      # Real-time audio buffers (WAV files)
├── *_species_list.txt   # Include/exclude/confirmed species
└── birds.db             # SQLite detection database
```

## Development Workflows

### Configuration Changes
1. **Always edit `config/birdnet.conf`** - don't modify docker-compose environment vars for BirdNET settings
2. **Restart container** after config changes: `docker-compose restart`
3. **Test web interface** at `http://localhost:8001` after changes

### Common Operations
```bash
# View logs
docker-compose logs -f birdnet-pi

# Restart after config changes  
docker-compose restart

# Access container shell for debugging
docker-compose exec birdnet-pi /bin/bash
```

### Species Management
- **Include species**: Edit `include_species_list.txt` (one per line)
- **Exclude species**: Edit `exclude_species_list.txt` (reduces false positives)
- **Whitelist**: `whitelist_species_list.txt` (priority detection)
- **Confirmed**: `confirmed_species_list.txt` (manually verified)

## Integration Points

### RTSP Audio Source
- Configured via `RTSP_STREAM` parameter in birdnet.conf
- Format: `rtsp://IP:PORT/stream_path`
- Stream index selection: `RTSP_STREAM_TO_LIVESTREAM="0"`

### Notifications (Apprise)
- Configuration: `config/apprise.txt` (currently empty)
- Settings in birdnet.conf: `APPRISE_*` parameters
- Supports new species alerts, detection notifications

### External Services
- **BirdWeather**: Set `BIRDWEATHER_ID` for community sharing
- **Flickr Images**: Requires `FLICKR_API_KEY` for bird photos
- **Custom Images**: `CUSTOM_IMAGE` for overview page customization

## Project-Specific Conventions

### File Naming Patterns
- Audio files: `YYYY-MM-DD-birdnet-RTSP_N-HHMMSS.wav`
- Temporary files stored in `config/TemporaryFiles/`
- Processed recordings moved to designated directories

### Configuration Management
- **Single source of truth**: `config/birdnet.conf`
- **No inline editing**: Use proper text editors, maintain Unix line endings
- **Parameter validation**: Many settings have specific ranges (e.g., CONFIDENCE: 0.0-1.0)

### Data Lifecycle
1. **Recording**: RTSP stream → 15-second chunks (configurable)
2. **Analysis**: BirdNET model processes audio
3. **Detection**: Species above confidence threshold saved
4. **Storage**: Audio + metadata stored in date-organized folders
5. **Purging**: Automatic cleanup when disk usage > 95%

## Critical Files for AI Understanding
- `docker-compose.yml`: Container orchestration and port mapping
- `config/birdnet.conf`: Complete system configuration (review sections 1-279)
- `config/BirdSongs/`: Data output structure and file organization
- `config/*_species_list.txt`: Species filtering logic

When making changes, always consider the complete audio pipeline: RTSP → BirdNET → Storage → Web Interface.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/harrymnel) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:agents_md:2026-04-10 -->
