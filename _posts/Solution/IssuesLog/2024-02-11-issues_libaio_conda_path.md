---
title: "[Issues]  Conda 환경에서 `libaio.so`를 찾지 못하는 문제: ld: cannot find -laio: No such file or directory"
writer: chanho
date: 2025-04-11 11:20:00 +0900
categories: [Solution, IssuesLog]
tags: [issues, conda]
---

> 나는 Ubuntu환경의 Docker Container에서 작업하고 있는데 Conda 환경 안에서 `deepspeed` 라이브러리를 불러오는 과정에서 `libaio-dev`를 설치했는데도 불구하고, **Conda의 컴파일러(`ld`)가 시스템 경로를 인식하지 못해서**  아래와 같은 에러 로그를 확인하게 되었습니다. 이에 해당 에러로그를 해결하기 위해 시도한 방법들을 기록하고자 합니다..

```
ld: cannot find -laio: No such file or directory
collect2: error: ld returned 1 exit status
```

## 에러 상황
: apt-get install libaio-dev로 라이브러리를 설치했지만, Conda 환경의 ld는 시스템 디렉토리(/usr/lib/x86_64-linux-gnu)를 검색하지 않아 여전히 인식하지 못하는 문제가 발생했습니다.


## 문제 원인
: Conda 내부에서 사용하는 toolchain은 시스템 기본 경로인 /usr/lib/x86_64-linux-gnu/를 참조하지 않고, 자체적으로 설정된 sysroot 디렉토리를 우선적으로 검색합니다. 이로 인해 libaio.so가 실제로 /usr/lib/x86_64-linux-gnu에 존재하더라도, Conda 환경에서 설정된 ld에는 해당 경로가 포함되어 있지 않아 "cannot find -laio" 에러가 발생한 것입니다.

## 해결 방법

### 1. `libaio-dev` 설치
> 우선 `libaio`를 설치하지 않았다면 진행해야 합니다.

```bash
apt-get install -y libaio-dev
```

### 2. Conda `sysroot` 경로에 `libaio.so` 복사
> Conda가 사용하는 다음 경로에 `.so` 파일을 직접 복사해줘야 합니다.

```
$CONDA_PREFIX/x86_64-conda-linux-gnu/sysroot/usr/lib/
```

예:
```bash
mkdir -p $CONDA_PREFIX/x86_64-conda-linux-gnu/sysroot/usr/lib/
cp /usr/lib/x86_64-linux-gnu/libaio.so $CONDA_PREFIX/x86_64-conda-linux-gnu/sysroot/usr/lib/
```

해당 작업 후, `ld`가 `libaio.so`를 잘 찾게 됩니다.

### 3. `LDFLAGS` 설정 (Option)
추가로 컴파일·링크 과정에서 경로를 명시할 수도 있습니다.

```bash
export LDFLAGS="-L/usr/lib/x86_64-linux-gnu"
```

이 후 `make LDFLAGS="$LDFLAGS"` 또는 `cmake -DCMAKE_EXE_LINKER_FLAGS="$LDFLAGS" ..` 등으로 빌드하면 됩니다.


## 도커 업데이트 (기록)
> 아래와 같은 내용을 Dockerfile과 설치 스크립트에 반영하여, libaio 관련 에러 로그가 발생하지 않도록 조치하였습니다.

- `docker_inst.sh`에 추가 내용
```bash
apt-get install -y libaio-dev
mkdir -p $CONDA_PREFIX/x86_64-conda-linux-gnu/sysroot/usr/lib/
cp /usr/lib/x86_64-linux-gnu/libaio.so $CONDA_PREFIX/x86_64-conda-linux-gnu/sysroot/usr/lib/
echo 'export LDFLAGS="-L/usr/lib/x86_64-linux-gnu"' >> ~/.zshrc
```

- `Dockerfile`에 추가 내용
```dockerfile
ENV LIBRARY_PATH=/usr/lib:/usr/lib/x86_64-linux-gnu
ENV LDFLAGS="-L/usr/lib/x86_64-linux-gnu"
```

> - **Dockerfile**: 베이스 이미지 지정, 파일 복사(`COPY`), `docker_inst.sh` 실행  
> - **docker_inst.sh**: 실제 apt-get 패키지 설치, Miniconda 설정, `libaio` 처리, etc.
