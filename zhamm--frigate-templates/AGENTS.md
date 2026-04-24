# Frigate NVR Templates - Copilot Instructions

## Project Overview
This repository contains hardware-optimized Docker Compose templates for [Frigate NVR](https://frigate.video/) - a real-time AI object detection video surveillance system. Each template is tailored for specific hardware accelerators and includes pre-configured detection models.

## Architecture

### Template Structure (Per Hardware Profile)
Each hardware profile directory contains:
- `docker-compose.yaml` - Container orchestration with hardware-specific device mappings
- `config/config.yml` or `config.yml` - Frigate configuration (detectors, cameras, GenAI)
- `config/model_cache/` - Pre-downloaded ONNX/TFLite models (Jetson only)
- `restart.sh` - Quick restart helper (nvidia-onnx, openvino only)

### Hardware Templates
1. **coral-usb/** - Google Coral EdgeTPU USB accelerator
2. **jetson-jp6/** - NVIDIA Jetson with JetPack 6 (TensorRT runtime)
3. **nvidia-onnx/** - NVIDIA GPU with ONNX runtime
4. **openvino/** - Intel integrated GPU with OpenVINO
5. **rpi/** - Raspberry Pi with optional Coral USB

## Critical Patterns

### Device Mappings Are Hardware-Specific
- **Coral USB**: `/dev/bus/usb`, `/dev/apex_0`
- **Jetson**: `/dev/nvidia-uvm`, `/dev/nvidia0`, runtime: nvidia
- **Intel iGPU**: `/dev/dri/renderD128`, `/dev/dri`
- **NVIDIA GPU**: `/dev/nvidia*` devices + `gpus: "all"`

### Docker Image Variants
- Standard: `ghcr.io/blakeblackshear/frigate:0.16.3`
- TensorRT: `ghcr.io/blakeblackshear/frigate:0.16.3-tensorrt`
- Jetson JP6: `ghcr.io/blakeblackshear/frigate:0.16.3-tensorrt-jp6`

### Environment Variables
- **Intel iGPU Driver Selection**:
  - `LIBVA_DRIVER_NAME=iHD` - 8th gen+ (default)
  - `LIBVA_DRIVER_NAME=i965` - 1st-7th gen (legacy)
- **GPU Memory (OpenVINO/AMD)**: `GPU_FORCE_64BIT_PTR=1`, `GPU_MAX_ALLOC_PERCENT=100`, `GPU_SINGLE_ALLOC_PERCENT=100`, `GPU_MAX_HEAP_SIZE=100`
- **NVIDIA**: `NVIDIA_VISIBLE_DEVICES=all`, `NVIDIA_DRIVER_CAPABILITIES=compute,video,utility`
- **Secrets**: `FRIGATE_RTSP_PASSWORD`, `PLUS_API_KEY`, GenAI API keys

### Model Configuration Patterns
```yaml
# Non-Plus generic model (ONNX)
detectors:
  nvidia0:
    type: onnx
model:
  path: /config/model_cache/yolov9-t.onnx
  model_type: yolo-generic
  width: 320
  height: 320

# Frigate+ hosted model
model:
  path: plus://MODEL_UUID
```

### GenAI Integration
Two provider patterns:
1. **Cloud APIs**: OpenAI (gpt-4o), Google Gemini (gemini-2.5-flash-lite)
2. **Local**: `OPENAI_BASE_URL=http://server.local:8080/v1` (llama.cpp compatible)

GenAI prompts include security-focused threat assessment with "Threat Level: Low/Medium/High" ratings.

## Common Workflows

### Deploying a Template
```bash
cd coral-usb/  # or your hardware profile
# Edit config/config.yml - add cameras, update API keys
docker compose up -d
docker logs -f frigate
```

### Quick Restart (nvidia-onnx, openvino)
```bash
./restart.sh  # Runs: docker compose down && docker compose up -d && docker logs -f frigate
```

### Volume Mounts
- Config: `./config:/config` (persistent)
- Media: `./storage` or `./video:/media/frigate` (recordings)
- Cache: `/tmp/cache` (tmpfs, 1-2GB)

## Configuration Conventions

### Shared Memory Sizing
- Coral/Intel: `shm_size: "2048mb"`
- NVIDIA/Jetson: `shm_size: "1024mb"`
- RPi: `shm_size: "128mb"`

### Authentication
All configs use:
```yaml
auth:
  enabled: true
  failed_login_rate_limit: 1/second;5/minute;20/hour
  trusted_proxies:
    - 172.18.0.0/16  # Docker Compose network
```

### Port Mappings (Standard)
- `5000:5000` - Frigate web UI
- `8554:8554` - RTSP feeds
- `8555:8555/tcp` & `8555:8555/udp` - WebRTC

## Key Files to Reference

- [openvino/README-iGPU.txt](openvino/README-iGPU.txt) - Intel GPU driver selection guide
- [coral-usb/config/config.yml](coral-usb/config/config.yml) - Full example with GenAI prompts
- [jetson-jp6/config/model_cache/](jetson-jp6/config/model_cache/) - Pre-cached model structure

## When Modifying Templates

1. **Preserve device mappings** - They're hardware-critical
2. **Update all instances** - Frigate version bumps should be consistent
3. **Validate GenAI prompts** - Security focus with threat ratings
4. **Check tmpfs sizes** - Must match system memory availability
5. **Don't commit secrets** - Use placeholders like `YOUR_SECURE_PASSWORD`

---
> Source: [zhamm/frigate-templates](https://github.com/zhamm/frigate-templates) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:agents_md:2026-04-24 -->
