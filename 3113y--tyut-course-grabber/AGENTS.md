# Copilot Instructions for TYUT Course Grabber

## Overview
This project is a Python-based GUI tool designed to automate login to the TYUT teaching management platform and assist with course selection. It integrates AI-based CAPTCHA recognition using ONNX models and provides features like multi-server polling, configuration management, and a course-grabbing assistant.

### Key Components
- **`app.py`**: Entry point of the application.
- **`autolink_modules/`**: Core logic modules:
  - `main_window.py`: Main GUI logic.
  - `captcha_handler.py`: Handles CAPTCHA recognition using ONNX models.
  - `preprocess_helper.py`: Image preprocessing for CAPTCHA.
  - `config_manager.py`: Manages user configurations.
  - `js_scripts.py`: Injects JavaScript for DOM manipulation.
  - `course_grabber.py`: Course-grabbing module.
- **`models/`**: ONNX models for CAPTCHA recognition.
- **`scripts/config.json`**: User credentials and server configurations.

## Developer Workflows

### Python Version
1. Install dependencies:
   ```bash
   pip install -r requirements.txt
   ```
2. Run the application:
   ```bash
   python app.py
   ```
3. Package:
   ```bash
   python build_spec.py
   ```

### Testing
- Ensure `scripts/config.json` is properly configured before running tests.

## Architecture
- **GUI Framework**: PyQt5 with embedded QWebEngineView browser.
- **CAPTCHA Recognition**: ONNX Runtime for deep learning model inference.
- **Image Processing**: Pillow for image preprocessing and OpenCV for advanced operations.
- **Configuration**: JSON-based configuration stored in `scripts/config.json`.

## Project-Specific Conventions
- **Configuration Management**:
  - All user credentials and server URLs are stored in `scripts/config.json`.
  - Course grabber config is in `scripts/course_grabber_config.json`.
  - Use the GUI to update configurations, which automatically saves to the file.
- **CAPTCHA Recognition**:
  - Models are stored in `models/`.
  - Implementation in `autolink_modules/captcha_handler.py` and `preprocess_helper.py`.
- **Course-Grabbing**:
  - Implementation in `autolink_modules/course_grabber.py`.
  - JS automation scripts in `autolink_modules/js_scripts.py`.

## External Dependencies
- **ONNX Runtime**: For AI-based CAPTCHA recognition.
- **PyQt5**: For GUI and embedded browser.
- **Pillow**: For image preprocessing.
- **OpenCV**: For advanced image processing operations.

## Integration Points
- **VPN and Teaching Platform**:
  - Login logic interacts with `vpn.tyut.edu.cn` and `192.168.200.100`.
  - Multi-server polling is implemented in `main_window.py`.
- **JavaScript Injection**:
  - DOM manipulation scripts are defined in `js_scripts.py`.

## Notes for AI Agents
- **Python code** lives in `app.py` and `autolink_modules/` — maintain clean module separation.
- When adding new features, ensure they integrate properly with the PyQt5 GUI in `main_window.py`.
- Use `readme.md` and this document as primary references for understanding workflows.

---

For any unclear sections or additional guidance, please consult the project maintainers or refer to the `readme.md` file.

---
> Source: [3113y/Tyut-Course-Grabber](https://github.com/3113y/Tyut-Course-Grabber) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:agents_md:2026-09-11 -->
