# gen-clear-drive: CycleGAN + YOLO 기반 야간 이미지 개선 파이프라인

## 프로젝트 개요

이 프로젝트는 **야간 주행 이미지를 주간 이미지로 변환**하여 객체 탐지(YOLO) 성능을 개선하는 연구 코드베이스입니다.
- **CycleGAN**: 야간(Night) → 주간(Day) 이미지 도메인 변환 (B→A 방향)
- **YOLO11**: 변환된 이미지에서 객체 탐지 성능 평가
- **BDD100K 데이터셋**: 날씨(clear/adverse) × 시간대(day/night) 조합으로 구성

## 아키텍처: 3단계 파이프라인

### 1. 데이터 준비 (`tools/` 스크립트들)
- `bdd_time_weather_to_yolo.py`: BDD100K JSON → YOLO 형식(이미지/라벨 폴더) 변환
- `build_clear_day2night.py`: CycleGAN 학습용 `trainA`(day)/`trainB`(night) 폴더 구성
- 결과: `datasets/yolo_bdd100k/{clear_daytime,clear_night,clear_synth_*}/`

### 2. CycleGAN 학습 (`pytorch-CycleGAN-and-pix2pix/`)
- **서브모듈**: junyanz/pytorch-CycleGAN-and-pix2pix (별도 레포)
- 학습: `train.py --model cycle_gan --dataroot ./datasets/clear_day2night --name clear_d2n_256_e200_k10k`
- 핵심 설정: `--netG resnet_9blocks --norm instance --no_dropout --load_size 286 --crop_size 256`
- 체크포인트: `checkpoints/<name>/latest_net_G_B.pth` (B→A 생성기)

### 3. 평가 실행 (`run.py`, `realtime_pipeline.py`)
- `run.py`: **배치 실험 스크립트** - 랜덤 샘플링 → CycleGAN 변환 → YOLO 평가 → 보고서 생성
- `realtime_pipeline.py`: **실시간 추론** - 단일 이미지/비디오에서 야간 판단 → CycleGAN → YOLO

## 핵심 워크플로우

### CycleGAN B→A 변환 (야간→주간)
```python
# run.py에서 사용하는 패턴
run_cyclegan_b2a(
    input_dir=IN_NIGHT_IMG,
    results_root=CG_RESULTS_ROOT,
    ckpt_name="clear_d2n_256_e200_k10k",
    norm="instance", no_dropout=True, netG="resnet_9blocks"
)
# → 결과: results_root/<ckpt>/test_latest/images/*_fake_A.jpg
```

**중요**: `--model test --model_suffix _B` 사용 → `G_B` 생성기만 로드 (B→A 방향)

### 야간 감지 휴리스틱 (Night Detector)
```python
# HSV V채널 기반 단순 규칙
def is_night(img_bgr, v_thresh=55.0, dark_ratio_thresh=0.35):
    hsv = cv2.cvtColor(img_bgr, cv2.COLOR_BGR2HSV)
    V = hsv[..., 2]
    mean_v = V.mean()
    dark_ratio = (V < 40).sum() / V.size
    return (mean_v < v_thresh) and (dark_ratio > dark_ratio_thresh)
```

### YOLO 평가 (Ultralytics API)
```python
# run.py 내부: API 모드 사용 (CLI 대신)
from ultralytics import YOLO
model = YOLO("yolo11s.pt")
metrics = model.val(data="data.yaml", split="test", imgsz=1280, device="0")
# → metrics.box.map, map50, mp, mr 추출
```

## 디렉터리 구조 규칙

### 입력 데이터셋 레이아웃
```
datasets/yolo_bdd100k/
  clear_daytime/
    images/test/*.jpg
    labels/test/*.txt  # YOLO 형식: <cls> <cx> <cy> <w> <h>
    data.yaml          # names: [person, car, ...], nc: 10
  clear_night/        # 동일 구조
  clear_synth_night/  # CycleGAN로 생성된 합성 데이터
```

### 실험 결과 레이아웃 (`experiments/run_YYYYMMDD_HHMMSS/`)
```
experiments/run_20251014_032540/
  inputs/               # 샘플링된 원본
    day/images/, labels/
    night/images/, labels/
  outputs/
    cyclegan_results/<ckpt>/test_latest/images/*_fake_A.jpg
    fake_day_from_night/images/*.jpg  # 리네임된 변환 결과
    daylike_mixed/images/  # day원본 + fake_day 병합
  yolo_results/         # YOLO val 결과 (runs/detect 형식)
    subset_day/, subset_night/, subset_fake_day_from_night/
  report/
    summary.json        # night-detector + YOLO 메트릭 통합
    night_detector_confusion.csv
```

## 프로젝트별 특이사항

### 0. **CRITICAL: 실행 환경 규칙**
- **모든 Python 스크립트는 venv에서 실행할 것**
  - bat 파일 시작 부분에 `call ..\venv\Scripts\activate.bat` 필수
  - venv 없으면 에러 출력 후 종료
- **GPU ID 지정 금지**
  - `--gpu_ids` 인자 사용하지 말 것 (자동 감지됨)
  - train.py가 `init_ddp()`로 자동 처리
- **기존 bat 파일 참조**
  - 새 bat 파일 만들 때는 기존 `train_cyclegan_yolo.bat` 구조 그대로 복사
  - 이름과 설명만 변경, 인자는 동일하게 유지

### 1. Windows 경로 + PowerShell
- 모든 경로는 `Path(r"C:\Users\...")` 스타일 raw string 사용
- 터미널 명령: PowerShell 기준 (`;`로 다중 명령 연결)

### 2. CycleGAN과 YOLO의 전처리 차이
- **CycleGAN**: `286×286 resize → 256×256 center crop` (학습 시 적용)
- **YOLO**: 원본 비율 유지, 긴 변을 `imgsz=1280`에 맞춤
- `realtime_pipeline.py`에서는 변환 후 원본 크기로 다시 복원

### 3. CycleGAN 모델 로드 시 주의
```python
# pytorch-CycleGAN-and-pix2pix/models/networks.py 직접 import
from models.networks import ResnetGenerator, get_norm_layer
netG = ResnetGenerator(
    input_nc=3, output_nc=3, ngf=64,
    norm_layer=get_norm_layer('instance'),
    use_dropout=False,  # --no_dropout 플래그와 일치해야 함
    n_blocks=9
)
```

### 4. 데이터 클래스 매핑 (BDD100K → YOLO)
```python
NAMES = ["person","rider","car","bus","truck","bike","motor",
         "traffic light","traffic sign","train"]  # 10개 클래스
```

## 주요 명령어

### CycleGAN 학습 (pytorch-CycleGAN-and-pix2pix/ 내부)
```bash
python train.py --dataroot ./datasets/clear_day2night --name clear_d2n_256_e200_k10k \
  --model cycle_gan --netG resnet_9blocks --norm instance --no_dropout \
  --load_size 286 --crop_size 256 --n_epochs 100 --n_epochs_decay 100
```

### CycleGAN 테스트 (단방향 B→A)
```bash
# CRITICAL: preprocess must match training!
# If trained with default settings (resize_and_crop):
python test.py --dataroot <night_images_dir> --name clear_d2n_256_e200_k10k \
  --model test --model_suffix _B --netG resnet_9blocks --norm instance --no_dropout \
  --preprocess resize_and_crop --load_size 286 --crop_size 256

# If trained with scale_width or for better YOLO compatibility:
python test.py --dataroot <night_images_dir> --name clear_d2n_256_e200_k10k \
  --model test --model_suffix _B --netG resnet_9blocks --norm instance --no_dropout \
  --preprocess scale_width --load_size 256 --crop_size 256
```

### 전체 파이프라인 실행
```bash
# 기본: scale_width preprocess (aspect ratio 보존, YOLO에 유리)
python run.py --n_day 100 --n_night 100 --yolo_model yolo11s.pt --device 0 --imgsz 1280

# CycleGAN 학습 시 resize_and_crop 사용했다면
python run.py --n_day 100 --n_night 100 --use_crop --yolo_model yolo11s.pt --device 0
```

### 실시간 추론 (단일 이미지)
```bash
python realtime_pipeline.py --image path/to/night_image.jpg --yolo yolo11s.pt
```

## 의존성 관리

- `requirements.txt`: 루트 디렉터리 (torch, ultralytics, opencv-python, pyyaml 등)
- `pytorch-CycleGAN-and-pix2pix/environment.yml`: CycleGAN 전용 (conda 환경)
- **충돌 방지**: CycleGAN 학습은 별도 환경, 평가 시에는 통합 환경 사용 권장

## 디버깅 팁

### 중요: CycleGAN 변환 후 YOLO 성능 최적화

**문제**: 야간 이미지를 CycleGAN으로 주간 변환 후 YOLO 성능이 저하됨

**실험 결과 (mAP50 기준):**
```
원본 야간:                    0.307
변환 후 (preprocess=none):     0.112 (36% 저하)
변환 후 (resize_and_crop):     0.008 (97% 저하) ❌
변환 후 (scale_width):         0.135 (20% 향상) ✅
```

**원인 분석:**
1. **preprocess=none**: 학습-테스트 분포 불일치 (GAN 품질 저하)
2. **resize_and_crop**: 256×256 정사각형 크롭 → 원본 복원 시 5배 업스케일 (심한 블러)
3. **scale_width**: aspect ratio 보존 → 2.8배 업스케일 (품질 유지) ✅

**최적 설정:**
```bash
# scale_width 사용 (기본값, 권장)
python run.py --n_day 50 --n_night 50 --device 0 --imgsz 1280 --yolo_model yolo11s.pt

# resize_and_crop은 품질 저하로 비추천 (--use_crop 사용 안 함)
```

**scale_width의 장점:**
- Aspect ratio 보존 (1280×720 → 455×256, 비율 유지)
- 전체 이미지 활용 (가장자리 손실 없음)
- 낮은 업스케일 배율 (품질 저하 최소화)
- 학습-테스트 전처리 일치

**검증:**
- `experiments/*/report/summary.json`에서 `subset_fake_day_from_night` mAP 확인
- scale_width 사용 시 원본 대비 20% 향상 확인됨

### 일반 디버깅

1. **CycleGAN 결과가 이상함**: `--norm instance --no_dropout` 확인, 체크포인트 에폭 번호 확인
2. **YOLO mAP가 0**: `data.yaml`의 `test:` 경로가 `images` 폴더를 가리키는지, `labels` 폴더가 병렬 존재하는지 확인
3. **야간 감지 오류**: `--night_v_thresh`, `--night_dark_ratio` 튜닝 (report/night_detector_details.csv 참고)
4. **메모리 부족**: YOLO `--imgsz` 줄이기 (1280 → 640), CycleGAN `--batch_size 1`

## 코드 수정 시 체크리스트

- [ ] `PROJ = Path(r"...")` 절대 경로가 환경에 맞게 수정되었는가?
- [ ] CycleGAN 모델 설정(`norm`, `netG`, `no_dropout`)이 학습 시와 동일한가?
- [ ] YOLO `data.yaml`의 `names`, `nc` 필드가 올바르게 설정되었는가?
- [ ] 새로운 데이터셋 추가 시 `tools/bdd_time_weather_to_yolo.py`에서 클래스 필터링 확인
- [ ] Windows 경로 문자열은 반드시 raw string(`r"..."`) 또는 `Path` 객체 사용

## 참고 문서

- CycleGAN Tips: `pytorch-CycleGAN-and-pix2pix/docs/tips.md`
- CycleGAN FAQ: `pytorch-CycleGAN-and-pix2pix/docs/qa.md`
- Ultralytics YOLO Docs: https://docs.ultralytics.com/

---
> Source: [zerossin/gen-clear-drive](https://github.com/zerossin/gen-clear-drive) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:agents_md:2026-08-13 -->
