# 실무를 위한 Docker

> Docker 명령어와 Dockerfile 작성 요약. 출처는 문서 하단 참고.

---

## 목차

1. [Docker 명령어](#1-docker-명령어)
2. [Dockerfile 작성](#2-dockerfile-작성)
3. [BuildKit & Multistage Build](#3-buildkit--multistage-build)

---

## 1. Docker 명령어

### 이미지

| 용도     | 명령어 |
|----------|--------|
| 목록     | `docker images` (기본), `docker images -a` (전체) |
| dangling | `docker images -f dangling=true` |
| 삭제     | `docker rmi $(docker images -q -f dangling=true)` (dangling만), `docker rmi $(docker images -q)` (전체, 주의) |
| 상세     | `docker inspect [IMAGE-ID/NAME]` |

### 컨테이너 실행

```bash
# 인터랙티브 실행 (root)
docker run -it [image-id] /bin/bash

# 특정 사용자·작업 디렉터리
docker run -it -u zany -w ~ [image-id] /bin/bash

# 백그라운드 실행
docker run -itd [image-id] /bin/bash

# 포트 매핑 (호스트:컨테이너)
docker run --publish 28088:8088 superset:v1
```

- `-it`: interactive + TTY. 종료 없이 나가려면 **Ctrl+P, Q** (exit 시 컨테이너 종료).
- `-itd`: 백그라운드 실행. 다시 들어갈 때 `docker exec -it [container-id] /bin/bash`.
- `[image-id]`는 `repository:tag` 형태로 써도 됨.

### 컨테이너 조회·삭제

| 용도       | 명령어 |
|------------|--------|
| 목록       | `docker ps` (실행 중), `docker ps -a` (전체) |
| 종료       | `docker stop $(docker ps -q -f status=running)` (graceful, SIGTERM) |
| 강제 종료  | `docker kill $(docker ps -q -f status=running)` |
| 삭제       | `docker rm $(docker ps -q -f status=exited)` |

- **stop**: SIGTERM으로 정상 종료 시도. **kill**: 기본적으로 즉시 종료(시그널 지정 가능).

---

## 2. Dockerfile 작성

### ENV vs ARG

| 구분   | ENV | ARG |
|--------|-----|-----|
| 시점   | 빌드 + 런타임 | **빌드 시점만** |
| 오버라이드 | `docker run -e VAR=값` | `docker build --build-arg VAR=값` |
| 기본값 | `${VAR:-기본값}`, `${VAR:+값}` | 동일 문법 사용 가능 |

- **ARG로 ENV 초기값 주기**: `ARG NAME` → `ENV NAME=${NAME:-WonChul}`. build 시 `--build-arg`, run 시 `-e`로 덮어쓸 수 있음.

### CMD vs ENTRYPOINT

| 구분       | CMD | ENTRYPOINT |
|------------|-----|------------|
| 역할       | 컨테이너 시작 시 실행할 **명령(또는 인자)** | 컨테이너 시작 시 실행할 **실행 파일** |
| 개수       | 여러 개 적어도 **마지막 하나만** 적용 | 한 번만 (실질적으로 마지막만 유효) |
| 덮어쓰기   | `docker run [IMAGE] [COMMAND]` 시 CMD 무시 | `docker run --entrypoint="명령" [IMAGE] ...` 로 변경 가능 |

- **함께 쓸 때**: `ENTRYPOINT`가 실행 파일, `CMD`가 그 **인자** 역할.  
  예: `ENTRYPOINT ["echo"]` + `CMD ["hello"]` → `echo hello` 실행.

### ADD vs COPY

| 구분     | ADD | COPY |
|----------|-----|------|
| 복사     | 지원 | 지원 |
| 압축 해제 | **가능** (로컬 파일; URL은 해제만, OS 의존) | 없음 |
| 권한     | root:root, URL은 600 등 | **원본 그대로** |

- `.dockerignore`에 있는 경로는 둘 다 제외. **일반적인 파일 복사는 COPY 사용 권장.**

---

## 3. BuildKit & Multistage Build

### BuildKit

- Docker 18.09+ 에서 사용 가능. **성능·저장소 관리·기능·보안** 개선.
- 사용: `DOCKER_BUILDKIT=1 docker build .`
- 참고: [Docker Build enhancements](https://docs.docker.com/develop/develop-images/build_enhancements/)

### Multistage Build

- Docker 17.05+ (DockerCon 2017). 여러 `FROM`으로 **stage**를 나누고, 필요한 산출물만 다음 stage로 복사해 **최종 이미지 크기**를 줄임.
- 예전 “builder 패턴”(개발용/운영용 Dockerfile 분리)을 **한 Dockerfile**로 처리 가능.

**요점**

- `FROM` 한 번 = stage 하나. `COPY --from=stage이름` 또는 `COPY --from=이미지:태그` 로 복사.
- **특정 stage까지만 빌드**: `docker build --target builder -t myimg:latest .`
- **외부 이미지에서 복사**: `COPY --from=nginx:latest /etc/nginx/nginx.conf /nginx.conf`

**장점**: Dockerfile 하나로 여러 컴포넌트 버전·빌드 바이너리를 관리하면서, 최종 이미지는 런타임에 필요한 것만 포함시킬 수 있음.

- 참고: [Multistage build](https://docs.docker.com/develop/develop-images/multistage-build/), [outsider 블로그](https://blog.outsider.ne.kr/1300)

---

## 출처

- Docker 명령어: http://home.zany.kr:9003/board/bView.asp?bCode=13&aCode=14169
- Dockerfile (ENV/ARG, CMD/ENTRYPOINT, ADD/COPY): https://github.com/heowc/programming-study/issues/90
- Docker 공식 문서: BuildKit, Multistage Build 링크 위 참조
