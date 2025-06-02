---
title: "[Docker] Window에서 Docker Desktop 없이 사용하기"
writer: chanho
date: 2025-06-02 09:10:00 +0900
categories: [Backend, Infra]
tags: [backend, docker]
---
도커 데스크탑(Docker Desktop)은 기업에서 사용할 때 규모나 사용 방식에 따라 라이센스 비용이 발생될 수 있다. 이에 따라 Docker Desktop을 제거하고, 라이센스 문제가 없는 오픈소스 무료 버전인 Docker Engine만 단독으로 설치해서 사용하고자 한다.

## 시작 전 체크리스트(Window 기준)

- ✅ WSL2가 설치된 Windows 환경 확인
- ✅ Docker Desktop 완전 제거 확인
- ✅ 최신 WSL2 Ubuntu (예: Ubuntu 22.04 LTS) 설치 확인
- ✅ WSL systemd 활성화 (최근 버전 디폴트 활성화됨, 확인 필요)

---

## 1단계: WSL Ubuntu 내에서 Docker Engine 설치하기

```bash
# 기존의 오래된 Docker 관련 항목이 있다면 삭제
sudo apt remove docker docker-engine docker.io containerd runc || true

# 설치 필수 패키지 설치
sudo apt update
sudo apt install -y apt-transport-https ca-certificates curl software-properties-common gnupg lsb-release

# 공식 Docker GPG 키 추가
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /usr/share/keyrings/docker-archive-keyring.gpg

# Docker 저장소 주소를 시스템 패키지에 추가하는 과정
echo "deb [arch=amd64 signed-by=/usr/share/keyrings/docker-archive-keyring.gpg] https://download.docker.com/linux/ubuntu $(lsb_release -cs) stable" | sudo tee /etc/apt/sources.list.d/docker.list > /dev/null

# Docker Engine 설치 (Docker CE)
sudo apt install -y docker-ce docker-ce-cli containerd.io

# Docker 서비스 시작
sudo service docker start

# Docker 시스템 시작 시 자동 실행 활성화
sudo systemctl enable docker.service
sudo systemctl enable containerd.service
```

## 2단계(Option): Docker를 sudo 없이 사용하도록 설정

```bash
sudo usermod -aG docker $USER

# 그룹 변경 즉시 반영
newgrp docker
```

> 만약 위 명령을 실행 후 docker 명령이 동작하지 않으면, WSL 재 실행 한다. 이러한 과정을 통해 PC에 Docker Desktop을 설치하지 않고도 도커를 활용할 수 있다.
> 

---

### 참조( 자주 사용하는 명령어)

### 1. `docker ps` – 실행 중인 컨테이너 목록 확인

- 기본 사용법: `docker ps`
- 옵션:
    - `a` : 실행 중이지 않은 컨테이너까지 모두 표시
    - `l` : 마지막으로 실행된 컨테이너만 표시
    - `q` : 컨테이너의 ID만 표시
    - `f` : 특정 조건의 컨테이너 필터링 예시) `docker ps -f status=exited`

사용 예시:

```bash
docker ps -a
```

### 2. `docker images` – 로컬에 있는 도커 이미지 목록 확인

- 기본 사용법: `docker images`
- 옵션:
    - `a` : 모든 이미지(중간 이미지 포함) 표시
    - `q` : 이미지 ID만 표시
    - `f` : 이미지 필터링

사용 예시:

```bash
docker images -a
docker images -q
```

### 3. `docker rm` – 컨테이너 삭제

- 기본 사용법: `docker rm [컨테이너 ID 또는 이름]`
- 옵션:
    - `f` : 실행 중인 컨테이너 강제 삭제
    - `v` : 컨테이너가 사용한 볼륨도 함께 삭제

사용 예시:

```bash
docker rm 컨테이너ID
docker rm -f 컨테이너ID
```

### 4. `docker rmi` – 이미지 삭제

- 기본 사용법: `docker rmi [이미지 ID 또는 이름:태그]`
- 옵션:
    - `f` : 이미지 강제 삭제 (컨테이너가 해당 이미지를 사용하는 경우에도 강제 진행 가능)

사용 예시:

```bash
docker rmi 이미지ID
docker rmi -f 이미지ID
```

### 5. `docker build` – Dockerfile을 바탕으로 이미지 생성

- 기본 사용법: `docker build [옵션] [경로 또는 URL]`
- 옵션:
    - `t` : 이미지 이름과 태그 지정
    예시) `docker build -t myimage:latest .`
    - `-no-cache` : 캐시 사용하지 않고 이미지 생성
    - `f` : 특정 Dockerfile 지정

사용 예시:

```bash
docker build -t 이미지이름:태그 .
docker build --no-cache -t 이미지이름:태그 .
```

### 6. `docker run` – 새로운 컨테이너 실행

- 기본 사용법: `docker run [옵션] 이미지 이름 [명령어]`
- 옵션 (자주 사용하는 옵션들):
    - `it` : 인터랙티브 모드로 컨테이너 접속 터미널 연결
    - `d` : 백그라운드 모드로 컨테이너 실행(detached mode)
    - `p` : 컨테이너와 호스트 간 포트 연결
    - `v` : 호스트 디렉토리와 컨테이너 디렉토리를 볼륨으로 연결
    - `-name` : 컨테이너 이름 지정
    - `-rm` : 컨테이너 종료 시 자동 삭제
    - `-gpus all` : 모든 GPU 리소스를 컨테이너에 연결 (NVIDIA Docker 설치 필요)
    - `-env` 또는 `e` : 컨테이너 내 환경변수 설정

사용 예시:

```bash
docker run -it -p 8080:80 --name 웹컨테이너 nginx
docker run -d --rm --name 임시컨테이너 ubuntu
docker run -it --gpus all nvidia/cuda:latest
docker run -v ./host_dir:/container_dir 이미지이름
```

### 7. `docker stop` – 실행 중인 컨테이너 정지

- 기본 사용법: `docker stop [컨테이너 ID 또는 이름]`

사용 예시:

```bash
docker stop 컨테이너ID
```

### 8. 컨테이너 재시작 및 재실행

- 정지된 컨테이너 재시작:

```bash
docker start 컨테이너ID
```

- 실행 중인 컨테이너 재시작:

```bash
docker restart 컨테이너ID
```

- 기존 컨테이너의 셸(shell)에 진입하여 명령어 수행:

```bash
docker exec -it 컨테이너ID /bin/bash
```

### 9. `docker logs` – 컨테이너 로그 확인

- 사용 예시:

```bash
docker logs 컨테이너ID
docker logs -f 컨테이너ID  # -f 옵션 시 실시간 로그 확인 가능
```

### 10. `docker pull` – 이미지 원격 저장소에서 다운로드

- 사용 예시:

```bash
docker pull 이미지이름:태그
docker pull nginx:latest
```

### 11. `docker push` – 로컬의 이미지를 원격 저장소에 업로드

- 사용 예시:

```bash
docker push 내 DockerHub 계정/이미지이름:태그
```

### 12. Docker 시스템 관리 명령어

- 사용하지 않는 컨테이너 정리:

```bash
docker container prune
```

- 사용하지 않는 이미지 정리:

```bash
docker image prune
```

- 사용하지 않는 모든 자원(volume, 네트워크 등) 정리:

```bash
docker system prune -a
```
