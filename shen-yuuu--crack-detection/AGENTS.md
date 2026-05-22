# AI Coding Agent Instructions for Crack Detection System

## Project Overview

This is a **road crack detection system** using deep learning (ConvNeXt + UPerNet) with a **Java + Python hybrid microservice architecture**. The project spans three major components:

- **Python AI Module** (`python-inference/`): Deep learning models, data processing, model training/inference
- **Java Backend** (`cloud-backend/`): 7 microservices (auth, gateway, dataset, inference, visual, report, common)
- **Vue.js Frontend** (`cloud-frontend/`): Modern web UI with Element Plus components

## Architecture Patterns

### Layered Architecture
```
User Layer (Vue 3) 
  ↓ HTTPS
Gateway Layer (Spring Cloud Gateway, port 8080)
  ↓ HTTP
Business Services (7 Java microservices, ports 8081-8085)
  ↓ HTTP/Feign
Python AI Service (FastAPI, port 8090)
  ↓
Data Storage (MySQL, Redis, MinIO, RabbitMQ)
```

### Key Design Decisions

1. **Java for Business Logic**: Spring Boot microservices handle user auth, data management, task orchestration, report generation
2. **Python for AI**: FastAPI service provides model inference, kept separate for independent scaling
3. **Message Queue Pattern**: RabbitMQ for async tasks (preprocessing, batch detection)
4. **Feature Extraction**: Multiple models - ConvNeXt Backbone, UPerNet Decoder, CBAM Attention, Strip Pooling for elongated cracks
5. **API Gateway**: Centralized auth (JWT), rate limiting, request routing

## Critical Files & Entry Points

### Python AI Module
- **Model Definition**: `python-inference/models/convnext_upernet.py` - ConvNeXt backbone + UPerNet decoder with edge detection branch
- **Data Loading**: `python-inference/dataset/data_loader.py` - COCO/VOC/YOLO format conversion, LMDB caching, 10x I/O speedup
- **Training**: `python-inference/training/trainer.py` - Mixed precision (AMP), EMA, SWA, early stopping
- **Inference**: `python-inference/inference/sliding_window.py` - Sliding window for arbitrary-sized images, TTA for 2-3% mIoU boost
- **API Server**: `python-inference/inference/api_server.py` - FastAPI endpoints for model inference
- **Config**: `python-inference/configs/train_config.yaml` - Training hyperparameters

**Key Performance**: mIoU 81.5%, F1 86.7%, 28 FPS on 512×512 images

### Java Backend Structure
```
cloud-backend/
├── cloud-common/          # Entities, DTOs, constants, utilities
├── cloud-auth/            # JWT token generation/validation (port 8081)
├── cloud-gateway/         # API routing, auth filter, rate limiting (port 8080)
├── cloud-dataset/         # Dataset/image CRUD, MinIO integration (port 8082)
├── cloud-inference/       # Feign client to Python service, job management (port 8083)
├── cloud-visual/          # Image overlay, heatmap generation (port 8084)
└── cloud-report/          # PDF/Excel generation via iText/EasyExcel (port 8085)
```

**Database**: MySQL with 6 core tables (users, datasets, images, detection_jobs, detection_results, reports)

### Frontend
- **Router**: `cloud-frontend/src/router/index.js` - Auth guards, role-based access control
- **API Integration**: `cloud-frontend/src/api/` - Axios wrapper with JWT auto-refresh
- **State Management**: `cloud-frontend/src/stores/user.js` - Pinia store
- **Pages**: 9 core views (login, dashboard, detection, dataset, history, report, profile, 404)

## Development Workflows

### Python Model Development
```bash
# Prepare multi-source datasets (COCO/VOC/YOLO → PNG masks)
cd python-inference
python scripts/prepare_datasets.py --source ../datasets --output ../data/processed

# Verify data quality and augmentations
python scripts/visualize_dataset.py --data-root ../data/processed --mode check

# Train with mixed precision, EMA, SWA
python train.py --config configs/train_config.yaml

# Export to ONNX for production
python export_onnx.py --model checkpoints/best.pth --output model.onnx

# Start inference API (used by Java services)
python inference/api_server.py
```

### Java Backend Development
```bash
# Start infrastructure (MySQL, Redis, MinIO, RabbitMQ, Nacos)
cd docker && docker-compose up -d

# Build all services
cd cloud-backend && mvn clean package

# Start individual services (they auto-register with Nacos)
java -jar cloud-gateway/target/cloud-gateway-1.0.0.jar
# Then other services discover gateway via Nacos registry

# Health checks
curl http://localhost:8080/api/v1/auth/health
curl http://localhost:8090/api/v1/health  # Python service
```

### Frontend Development
```bash
# Install dependencies
cd cloud-frontend && npm install

# Development server with hot reload
npm run dev

# Build for production
npm run build

# Test API endpoints (assumes backend running on localhost:8080)
```

## Code Conventions & Patterns

### Java Backend Conventions
- **Entity Naming**: Use `@TableName("table_name")` annotations (plural form: users, datasets, images)
- **DTO Naming**: Use suffixes like `Request`, `Response`, `VO`, `DTO`
- **API Endpoints**: Follow `/api/v1/{service}/{resource}` pattern (e.g., `/api/v1/dataset/images`)
- **Service Layer**: Each module has `Service` → `Mapper` (MyBatis Plus) → Database flow
- **Feign Clients**: Placed in `cloud-inference/` to call Python API (example: detect image via HTTP)
- **Error Handling**: Use `ApiException` with error codes for consistent error responses
- **Configuration**: Use `application-prod.yml` / `application-dev.yml` for environment-specific settings

**Example API Call Chain**:
```
Request → Gateway (8080) 
  → Auth Filter (JWT validation)
  → Route to Service (e.g., cloud-inference:8083)
  → Service → Feign Client → Python API (8090)
  → Python Model Inference
  → Response back through layers
```

### Python Model Conventions
- **Loss Functions**: `losses.py` contains composable loss components (Dice, Focal, BCE, Boundary, Edge)
- **Augmentation**: Use Albumentations for efficient data augmentation with mask preservation
- **Model Architecture**: 
  - Encoder: ConvNeXt (ImageNet-22K pretrained)
  - Decoder: UPerNet with PPM, FPN, CBAM, Strip Pooling
  - Auxiliary: Edge detection branch + deep supervision (3 heads)
- **Training Tricks**: Mixed precision (AMP), gradient accumulation, EMA, SWA, early stopping
- **Inference Modes**: Sliding window for large images, TTA for ensemble boosting, ONNX export for production

### Frontend Conventions
- **Components**: Vue 3 `<script setup>` composition API
- **State**: Pinia stores for global state (user, auth token, settings)
- **HTTP**: Axios instance with interceptors for JWT auto-refresh in `utils/request.js`
- **UI Library**: Element Plus for consistent component design
- **Routing**: Role-based access control via meta.public flag in route definitions

## Critical Integration Points

### Python ↔ Java Communication
1. **Synchronous**: Java Feign client calls Python `/api/v1/inference/detect` (blocking)
   - Used for single-image detection, requires immediate response
   - File: `cloud-inference/service/InferenceService.java`

2. **Asynchronous**: RabbitMQ for batch tasks (non-blocking)
   - Used for dataset preprocessing, batch detection
   - Producer: `cloud-inference/service/BatchTaskProducer.java`
   - Consumer: Python task worker (TBD implementation)

### Data Flow: Image Upload → Detection → Results
1. User uploads image via `POST /api/v1/dataset/{datasetId}/images`
2. `DatasetService` saves to MinIO, metadata to MySQL
3. User triggers detection via `POST /api/v1/inference/detect`
4. `InferenceService` calls Python API (Feign + HTTP)
5. Python model processes image (512×512 window), returns masks/confidence
6. `InferenceService` saves results to MySQL
7. User views results via `GET /api/v1/detection/result/{jobId}`
8. `VisualService` generates overlay image from original + mask

### Database Entity Mappings
```
users ↔ JWT claims (userId in token)
datasets ↔ data organization (multiple images per dataset)
images ↔ MinIO storage (fileUrl = s3://bucket/path/image.jpg)
detection_jobs ↔ async task tracking (status: processing/completed/failed)
detection_results ↔ model output persistence (masks, confidence scores)
reports ↔ PDF/Excel generation (aggregates results by dataset/job)
```

## Performance Optimization Notes

### Python Inference Optimization
- **ONNX Export**: 2-3x speedup over PyTorch FP32, enables cross-platform deployment
- **TensorRT**: 5-10x speedup with FP16/INT8 quantization (production-only)
- **Sliding Window**: Avoid OOM on large images, Gaussian blending for smooth seams
- **TTA**: 0.75x, 1.0x, 1.25x scales + flips = 2-3% mIoU improvement at inference cost

### Java Backend Optimization
- **Async Processing**: Use RabbitMQ for batch jobs, don't block API responses
- **Caching**: Redis for frequently accessed data (user info, model cache)
- **Connection Pooling**: HikariCP for database, reduce connection overhead
- **Rate Limiting**: Gateway-level limiting to prevent abuse

### Frontend Optimization
- **Lazy Loading**: Vue Router lazy-load components with `() => import(...)`
- **Image Optimization**: Lazy load images with `el-image`, cache results
- **State Caching**: Pinia auto-persists user token to localStorage

## Common Tasks & Examples

### Adding a New Detection Model
1. Create new model class in `python-inference/models/new_model.py` inheriting from `nn.Module`
2. Register in `inference/api_server.py` load function
3. Update `train.py` to support new model type via config
4. No Java changes needed (Python API is abstraction layer)

### Adding a New API Endpoint
1. Add controller method in relevant Java service (e.g., `DatasetController.java`)
2. Add corresponding Feign client method if calling Python
3. Create request/response DTOs in `cloud-common/entity/`
4. Update API documentation in JavaDoc or Knife4j swagger comments

### Debugging End-to-End Flow
```bash
# Check Python service is running
curl http://localhost:8090/api/v1/health

# Check Java gateway routing
curl -H "Authorization: Bearer TOKEN" http://localhost:8080/api/v1/auth/user/info

# Trace request through logs (service logs available in container)
docker logs crack-auth-service
```

## Repository Structure & Key Locations

| Path | Purpose |
|------|---------|
| `docs/系统设计方案.md` | Complete architecture + design details |
| `docs/项目总结.md` | Performance metrics, model comparison |
| `docs/毕业设计进展报告.md` | Thesis progress report |
| `docker/docker-compose.yml` | Infrastructure setup (MySQL, Redis, etc.) |
| `python-inference/requirements.txt` | Python dependencies |
| `cloud-backend/pom.xml` | Java dependency management |
| `.github/copilot-instructions.md` | This file |

## Testing & Validation

### Python
- **Data Validation**: `prepare_datasets.py` auto-checks image-mask alignment, artifact filtering
- **Model Validation**: Metrics computed on 6 public datasets (Crack500, CrackLS315, CFD, etc.)
- **Quick Validation**: `python quick_start.py` runs 5 quick tests

### Java
- Run tests via `mvn test` in each module
- Integration tests in `cloud-*/src/test/java/`
- Use Postman/curl for API testing (endpoints documented in README.md)

### Frontend
- Use browser DevTools network tab to trace HTTP requests
- Verify JWT token in localStorage under Application tab
- Check Console for Vue/Axios errors

## When Adding Features

1. **Always update database schema** first if adding new entity (SQL in `docker/init-db/01-init.sql`)
2. **Add corresponding entity class** in `cloud-common/entity/` with `@TableName` annotation
3. **Create Mapper interface** in relevant service module extending `BaseMapper<Entity>`
4. **Implement Service layer** with business logic
5. **Add Controller** with request/response DTOs
6. **Update Frontend** with API call in `cloud-frontend/src/api/` and page component
7. **Test end-to-end** through API gateway (port 8080)

## Dependencies & Version Notes

- **Python**: 3.10+, PyTorch 2.0+, CUDA 11.8+ for GPU acceleration
- **Java**: JDK 17+, Maven 3.8+
- **Frontend**: Node.js 16+, npm/yarn
- **Infrastructure**: Docker 20+, MySQL 8.0+, Redis 7.0+

## Questions?

Refer to:
- Architecture: `docs/系统设计方案.md` (pages 1-50 for overview)
- Implementation: README.md files in each module
- Performance: `docs/项目总结.md` (metrics section)
- API Details: Swagger at `http://localhost:8080/swagger-ui.html` (after services start)

---
> Source: [Shen-Yuuu/crack-detection](https://github.com/Shen-Yuuu/crack-detection) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:agents_md:2026-05-22 -->
