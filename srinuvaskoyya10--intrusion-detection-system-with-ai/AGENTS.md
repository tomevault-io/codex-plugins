# Copilot Instructions - Network Intrusion Detection System (IDS)

## Project Overview

**Purpose**: Production-ready ML-based Intrusion Detection System with Flask REST API backend, web dashboard, and automated email alerting.

**Key Metrics**: 97% detection rate using Random Forest classifier on KDD Cup 99 dataset (41 features after one-hot encoding).

**Architecture**:
- **Backend** (`backend/server.py`): Flask API + scikit-learn Random Forest model
- **Frontend** (`frontend/index.html`): Real-time dashboard with attack monitoring
- **ML Pipeline** (`extract_model.py`, `intrusion-detection-system-with-ml-dl.ipynb`): Data preprocessing, model training, serialization
- **Testing** (`backend/attack_simulator.py`): Synthetic attack generation for validation

---

## Critical Context for AI Agents

### Data Flow: Request → Prediction → Alert
1. **Input**: 41-dimensional feature vector (normalized numeric + one-hot encoded categorical)
2. **Processing**: `scaler.pkl` normalizes raw features → `rf_model.pkl` predicts (normal=0, attack=1)
3. **Output**: Confidence score; if >70% and attack, triggers email alert via SMTP
4. **Logging**: All predictions logged; attack patterns tracked for threshold anomalies

**Key Files**:
- Model serialization: `models/rf_model.pkl`, `models/scaler.pkl`, `models/feature_columns.pkl`
- Feature schema: 41 total (mixed numerical, categorical), hardcoded in `backend/server.py` line 87-130
- Production data: `KDDTest+.arff` (test dataset reference)

### Feature Engineering Pattern
The dataset uses **41 features after preprocessing** (in `extract_model.py`):
- **Raw features** (22): duration, protocol_type, service, flag, bytes counts, error counts, etc.
- **Encoded features**: One-hot encoding applied to categorical columns: `protocol_type`, `service`, `flag`
- **Result**: Each category expands to binary columns (e.g., protocol_type: tcp/udp/icmp → 3 binary features)

**Always validate**: Input feature count must be exactly 41 or API returns 400 error (see `backend/server.py` line 107).

### Email Alert System
Located in `backend/server.py` lines 44-70 (`send_alert_email()`):
- **Config required**: SMTP credentials in `EMAIL_CONFIG` dict (gmail app password recommended)
- **Trigger**: `confidence > 0.7 AND prediction == 'ATTACK'`
- **Content**: ISO timestamp, confidence %, full attack feature vector, action instructions
- **Error handling**: Logs failures; does NOT block predictions if email fails

**Critical**: Update `EMAIL_CONFIG` before deployment or alerts silently fail.

### Attack Types (from KDD-99 Dataset)
Four classes defined in `backend/attack_simulator.py`:
- **DoS** (Denial of Service): High connection/error rates, repeated service hits
- **Probe** (Reconnaissance): Port scanning, diverse destination hosts/ports
- **R2L** (Remote-to-Local): Failed logins, unusual patterns, external attacks
- **U2R** (User-to-Root): Local privilege escalation, file creation, shell execution

**Note**: Model trained on binary classification (normal vs. any attack); doesn't distinguish attack subtypes.

---

## Development Workflows

### Setup & Deployment
```bash
# 1. Extract model from notebook (one-time)
python extract_model.py                    # Creates models/*.pkl files

# 2. Start backend (Terminal 1)
cd backend && python server.py             # Listens on http://0.0.0.0:5000

# 3. Open dashboard (Terminal 2)
# Open frontend/index.html in browser (Dashboard auto-connects to API)

# 4. Test with attack simulation (Terminal 3)
cd backend && python attack_simulator.py
```

### Model Retraining
1. Update training data in `KDDTest+.arff` or source notebook
2. Run `intrusion-detection-system-with-ml-dl.ipynb` (cells execute model training + evaluation)
3. Execute `python extract_model.py` to save updated `models/*.pkl`
4. Restart backend to load new model

### Health Checks
- **API health**: `curl http://localhost:5000/health` → returns `{"status":"online","model_loaded":true}`
- **Dashboard**: Check status cards in frontend for attack count, uptime, model accuracy
- **Logs**: Backend logs in stdout; check for "Model loaded successfully" at startup

---

## Project-Specific Patterns & Conventions

### Pickle-Based Model Persistence
Unlike modern ML workflows (ONNX, SavedModel), this project uses **pickle for serialization**:
- All 3 artifacts stored in `models/` directory as `.pkl` files
- **Advantage**: Simple, preserves scikit-learn objects exactly
- **Limitation**: Version-dependent; requires same sklearn version for load/predict

When modifying model loading (`backend/server.py` lines 26-33):
- Always wrap in try-except; log failures explicitly
- Validate `feature_count` matches expected 41 or raise error immediately

### Dashboard Real-Time Updates
Frontend (`frontend/index.html`) uses **polling** (not WebSockets):
- AJAX calls to `/health` and `/predict` endpoints every 2-5 seconds
- Updates status cards: attack count, detection rate %, model accuracy
- **Pattern**: Stateless HTTP calls; no persistent connections required

### Feature Vector Validation
The API endpoint (`backend/server.py` line 87-130) enforces:
```python
if len(data['features']) != 41:
    return jsonify({'error': 'Expected 41 features'}), 400
```
When generating test data, always produce exactly 41 normalized floats (see `attack_simulator.py` for examples).

### Confidence Threshold as Risk Control
- Model outputs probability (0.0-1.0)
- Emails triggered only if confidence > 0.7 **AND** prediction == attack
- **Design intent**: Reduce false alarm fatigue while catching 95%+ of true attacks

---

## When Debugging or Extending

### Common Issues & Resolutions
| Issue | Root Cause | Fix |
|-------|-----------|-----|
| `FileNotFoundError: models/rf_model.pkl` | Model not extracted | Run `python extract_model.py` |
| 400 Bad Request on `/predict` | Wrong feature count or format | Verify 41 float values sent as JSON |
| Email alerts not sent | SMTP config missing/invalid | Update `EMAIL_CONFIG` in `server.py` lines 37-42 |
| Dashboard shows "API offline" | Backend not running | `cd backend && python server.py` |
| Low accuracy (< 90%) | Model degradation or new data distribution | Retrain using notebook with fresh data |

### Extending Attack Types
To add new attack simulations:
1. Add function to `backend/attack_simulator.py` (e.g., `generate_new_attack()`)
2. Follow pattern: return 41-length list with appropriate feature values
3. Optionally update ATTACK_TYPES dict and update REST API in `server.py` if adding /attack/<type> endpoint

### Adding Endpoints
Keep pattern consistent:
- Accept POST with JSON body containing 'features' array
- Validate feature count (41)
- Scale features using loaded scaler
- Predict using model
- Log result with timestamp
- Return JSON with 'prediction', 'confidence', 'timestamp'

---

## File Reference

| File | Purpose | Key Functions/Classes |
|------|---------|----------------------|
| `backend/server.py` | Flask API server | `health()`, `predict()`, `send_alert_email()` |
| `backend/attack_simulator.py` | Test data generator | `generate_*_attack()` functions (DoS, R2L, U2R, Probe) |
| `extract_model.py` | Model extraction from notebook | Preprocessing, Random Forest training, serialization |
| `intrusion-detection-system-with-ml-dl.ipynb` | ML training notebook | EDA, model comparison (RF vs XGBoost vs TensorFlow), evaluation plots |
| `frontend/index.html` | Dashboard UI | Real-time stats display, attack log, API polling logic |
| `models/*.pkl` | Serialized artifacts | `rf_model.pkl`, `scaler.pkl`, `feature_columns.pkl` |
| `AGENT_INSTRUCTIONS.md` | Operational guide (for humans) | Deployment, monitoring, alert thresholds, recovery procedures |

---
> Source: [srinuvaskoyya10/Intrusion_detection_system_with_AI](https://github.com/srinuvaskoyya10/Intrusion_detection_system_with_AI) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:agents_md:2026-08-26 -->
