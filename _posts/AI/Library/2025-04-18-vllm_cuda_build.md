---
title: "[vLLM] CUDA 12.4 및 11.8 환경에서 vLLM 빌드"
writer: chanho
date: 2025-04-18 10:00:00 +0900
categories: [AI, Library]
tags: [llm, vllm, troubleshooting]
---

> vLLM을 LLM 서빙 라이브러리로 사용하고자 하였으나, 현재 주로 사용하는 서버의 GPU가 오랜된 연식의 GPU여서 몇몇의 난관이 있었다. 이를 해결한 과정과 함께 최종 빌드에 성공한 스크립트를 기록하고자 한다.

- Docker Image: https://hub.docker.com/r/nvidia/cuda
- Pytorch: https://pytorch.org/get-started/locally/

## 1. CUDA 12.4 -Pre-built wheels

### 가.  환경 구성

- GPU: `Nvidia - RTX 4090`
- CUDA Version: `12.4.1`
- OS: `Ubuntu 22.04`
- Python: `3.12`
- PyTorch: `2.6.0`

### 나. Script (CUDA 12.4)

```bash
# - CUDA 12.4 환경에 맞는 PyTorch 2.6.0 설치
$ pip install torch==2.6.0 torchvision==0.21.0 torchaudio==2.6.0 \\
  --index-url <https://download.pytorch.org/whl/cu124>

# (Option) CMake 최신 버전 설치 및 PATH 설정
# - vLLM은 CMake 3.26 이상 필요
$ pip install cmake --upgrade

# vLLM 설치
$ pip install vllm

# --- 빌드 확인 ---
$ python -c "import vllm; print(vllm.__version__)"
```

---

## 2. CUDA 11.8 - Source Build

### 가. 환경 구성

- GPU:  `Nvidia RTX 4090 or Nvidia RTX Titan`
- CUDA Version: `11.8.0`
- OS: `Ubuntu 22.04`
- Python: `3.10`
- Pythorch: `2.6.0+cu118`

### 나. Script (CUDA 11.8)

```bash
# 1. PyTorch + CUDA 11.8 설치
$ pip install torch torchvision torchaudio --index-url <https://download.pytorch.org/whl/cu118>

# 2. (Option) CMake 최신 버전 설치
$ pip install cmake --upgrade

# 3. vllm 디렉토리에서 진행: PyTorch 의존성을 vLLM 설치 과정에서 제외시키기 위해 사용
$ python use_existing_torch.py
$ pip *install -r requirements/build.txt*

# 4. CUDA 11.8 환경 변수 설정 및 소스 빌드
$ export VLLM_VERSION=0.6.1.post1
$ export PYTHON_VERSION=310

$ export CUDA_HOME=/usr/local/cuda-11.8
$ export PATH="$CUDA_HOME/bin:$PATH"
$ export LD_LIBRARY_PATH="$CUDA_HOME/lib64:$LD_LIBRARY_PATH"
$ export MAX_JOBS=1 # 메모리 부족(OOM) 방지를 위해 빌드 병렬 작업 수 제한 (시스템 사양에 따라 조절)

$ pip install -e . --no-build-isolation

# 5. 빌드 확인
python -c "import vllm._C; print('CUDA 11.8 빌드된 vLLM 모듈 로드 성공')"

# (Option) vLLM 실행 시 xformers가 필요할 수 있음
pip install xformers==0.0.29.post2 --index-url <https://download.pytorch.org/whl/cu118>
```

---

## 3. Troubleshooting Details

### 문제 1: CUDA 버전 관련 문제

- **(환경: CUDA 12.4 + 구형 GPU)** `RuntimeError: ... Error 804: forward compatibility was attempted on non supported HW`
    - **원인**: 설치된 PyTorch 버전이 시스템의 CUDA 드라이버/GPU 조합과 호환되지 않음 (주로 PyTorch가 너무 최신이거나 구형일 때 발생). TITAN RTX(Compute Capability 7.5)는 CUDA 12.4를 지원하지만, 특정 PyTorch 버전과 미묘한 호환성 문제가 있을 수 있음.
    - **시도**: CUDA 11.8, 12.4용 PyTorch 버전을 바꿔가며 설치.
    - **해결**: vLLM 최신 코드가 요구하는 PyTorch 2.6.0 (cu124) 버전을 명시적으로 설치하여 해결. 이는 vLLM과의 호환성 문제였을 가능성이 높음.
- **(환경: CUDA 11.8)** `ImportError: libcudart.so.12: cannot open shared object file`
    - **원인**: CUDA 11.8 환경에서 CUDA 12.x용으로 사전 컴파일된 vLLM 바이너리(`VLLM_USE_PRECOMPILED=1` 사용 시)를 설치했기 때문. 시스템에는 `libcudart.so.11`만 존재함.
    - **시도**: `VLLM_USE_PRECOMPILED=1` 옵션으로 설치 시도.
    - **해결**: `VLLM_USE_PRECOMPILED=1` 옵션을 **제거**하고, `CUDA_HOME` 등 환경 변수를 CUDA 11.8에 맞게 설정한 뒤 **소스 코드에서 직접 빌드** (`pip install -e . --no-build-isolation`).

### 문제 2: CMake 버전 부족

- **에러**: `CMake Error at CMakeLists.txt:1 (cmake_minimum_required): CMake 3.26 or higher is required. You are running version 3.22.1`
    - **원인**: 시스템 또는 가상환경에 설치된 CMake 버전이 vLLM이 요구하는 최소 버전(3.26)보다 낮음.
    - **시도/해결**: `pip install cmake --upgrade` 명령으로 최신 CMake를 설치하고, 필요 시 `export PATH=...` 명령으로 시스템이 새로 설치된 CMake를 인식하도록 경로를 설정.

### 문제 3: PyTorch 버전/API 불일치

- **에러**: `error: ‘needs_fixed_stride_order’ is not a member of ‘at::Tag’`
    - **원인**: 빌드하려는 vLLM 버전(특히 최신 main 브랜치)이 내부적으로 사용하는 PyTorch API(`at::Tag::needs_fixed_stride_order`)가 설치된 PyTorch 버전(예: < 2.6.0)에는 존재하지 않음.
    - **시도**: 기존 PyTorch 제거 후 다른 버전 설치.
    - **해결**: vLLM 코드와 호환되는 PyTorch 버전 (이 경우, PyTorch 2.6.0)을 설치. 해당 CUDA 버전에 맞는 wheel 파일을 명시하여 설치 (`-index-url <https://download.pytorch.org/whl/cu124`>).

### 문제 4: NumPy 2.x 호환성

- **에러**: `A module that was compiled using NumPy 1.x cannot be run in NumPy 2.x ... downgrade to 'numpy<2'`
    - **원인**: vLLM 또는 그 의존성 중 일부가 NumPy 1.x 버전으로 컴파일되었는데, 현재 환경에는 NumPy 2.0 이상이 설치되어 ABI 호환성 문제가 발생.
    - **시도/해결**: `pip install "numpy<2"` 명령으로 NumPy 버전을 1.x대로 다운그레이드.

### 문제 5: xformers 관련 문제

- **(환경: CUDA 12.4)** 초기 빌드 시도 중 xformers와 attention backend 관련 충돌이 언급되었으나, 근본 원인은 PyTorch 버전 불일치로 파악되어 PyTorch 버전 해결 후 문제가 사라짐.
- **(환경: CUDA 11.8)** 소스 빌드 성공 후 vLLM 실행 시 `ModuleNotFoundError: No module named 'xformers'` 발생.
    - **원인**: vLLM이 특정 Attention 메커니즘을 사용하기 위해 xformers 라이브러리를 요구하는데, 빌드 과정에서 자동으로 설치되지 않았거나, `requirements.txt`에 포함되지 않았을 수 있음.
    - **시도/해결**: 빌드 완료 후, 현재 환경(CUDA 11.8, PyTorch 버전)에 맞는 xformers 버전을 찾아서 수동으로 설치 (`pip install xformers==0.0.29.post2 --index-url <https://download.pytorch.org/whl/cu118`>).

### 문제 6: Git 'dubious ownership'

- **에러**: `fatal: detected dubious ownership in repository at '/app/vllm/.deps/cutlass-src'`
    - **원인**: Docker 컨테이너 등에서 파일을 마운트하거나 사용자 권한이 다른 경우, Git이 저장소 소유권을 신뢰할 수 없다고 판단하여 발생. vLLM 빌드 중 `cutlass` 같은 외부 의존성을 클론할 때 나타날 수 있음.
    - **시도/해결**: `git config --global --add safe.directory /app/vllm/.deps/cutlass-src` 또는 더 광범위하게 `git config --global safe.directory '*'` 명령으로 해당 디렉토리를 안전한 것으로 등록.

### 문제 7: 빌드 캐시 및 잔여 파일 문제

- **에러**: `ERROR: Failed building editable for vllm` 또는 기타 예측 불가능한 빌드 오류.
    - **원인**: 이전 빌드 시도에서 생성된 `build/`, `dist/`, `.egg-info`, `.deps` 디렉토리나 pip 캐시가 남아있어 새로운 빌드 과정과 충돌.
    - **시도/해결**: 빌드를 재시도하기 전에 관련 캐시와 디렉토리를 깨끗하게 삭제 (`pip cache purge`, `rm -rf build/ dist/ *.egg-info .deps vllm.egg-info/`, `/tmp`의 관련 파일 삭제 등).

---

> vLLM을 로컬 환경에서 빌드하는 작업은 시스템 환경(OS, CUDA 버전, GPU 종류), Python 패키지 버전(PyTorch, CMake, NumPy 등)에 매우 민감한 것 같다. 특히 CUDA 버전과 PyTorch 버전, 그리고 빌드하려는 vLLM 코드 버전 간의 호환성을 맞추는 것이 핵심이다.
