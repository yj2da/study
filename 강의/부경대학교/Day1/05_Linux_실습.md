# 05. Linux 실습

## 실습 목표

04에서 배운 **Linux 기본 명령어**를 직접 써 보며, 디렉토리 생성·파일 조작·검색까지 한 번에 따라 해 봅니다.  
데이터 파이프라인에서 자주 쓰는 **raw(원본) / processed(가공)** 폴더 구조를 만들어 보는 것이 목표입니다.

---

## 실습 단계

### 1. 터미널 열기 및 현재 위치 확인

```bash
pwd
```

- 터미널(또는 WSL, Git Bash)을 연 뒤 **현재 작업 디렉토리**를 확인합니다.
- `pwd` 출력 결과가 예상한 경로인지 확인해 두면, 이후 실습에서 혼란이 줄어듭니다.

---

### 2. 디렉토리 생성 및 이동

```bash
mkdir -p ~/Desktop/data-engineering/day1
cd ~/Desktop/data-engineering/day1
```

- `mkdir -p` 로 **상위 경로가 없어도 한 번에** `~/Desktop/data-engineering/day1` 를 만듭니다.
- `~` 는 홈 디렉토리입니다. 이 폴더를 오늘 실습의 **작업 디렉토리**로 사용합니다.

---

### 3. 샘플 파일 생성 및 조작

```bash
echo "Hello Data Engineering" > sample.txt
cat sample.txt
cp sample.txt sample_backup.txt
ls -l
ll
```

- `echo "..." > 파일` 로 **한 줄 텍스트를 파일로 저장**합니다. `>` 는 기존 내용을 덮어씁니다.
- `cat` 으로 내용을 확인하고, `cp` 로 백업 파일을 만든 뒤 `ls -l` 로 파일 목록(권한·크기·날짜)을 봅니다.
- `ls -l` 너무 길어서 `ll`으로 단축해서 사용하기도 합니다.

---

### 4. 디렉토리 구조 만들기

```bash
mkdir raw processed
mv sample.txt raw/
ls raw/
```

- **raw**(원본 데이터), **processed**(가공된 데이터) 폴더를 만들고, `sample.txt` 를 `raw/` 로 옮깁니다.
- 실제 파이프라인에서도 “원본은 raw, 처리 결과는 processed” 처럼 구분해 두는 경우가 많습니다.

---

### 5. 파일 검색 및 내용 확인

```bash
find . -name "*.txt"
grep "Data" raw/sample.txt
```

- `find . -name "*.txt"` 로 **현재 디렉토리 아래 모든 .txt 파일** 경로를 찾습니다.
- `grep "Data" raw/sample.txt` 로 해당 파일 안에 **"Data"** 가 들어 있는 줄만 출력합니다. 로그에서 키워드 찾을 때와 같은 방식입니다.

---

## 체크포인트

**확인 질문**: "여기까지 완료하셨나요? `sample_backup.txt` 파일이 보이나요?"

- **sample_backup.txt 가 안 보일 때**: 3단계에서 `cp sample.txt sample_backup.txt` 를 **`day1` 폴더 안에서** 실행했는지 확인. `ls -l` 로 현재 디렉토리 목록을 보면 됩니다.
- **경로가 다를 때**: `pwd` 로 현재 위치를 확인한 뒤, `cd ~/Desktop/data-engineering/day1` 로 다시 이동 후 실습을 이어가면 됩니다.
- **Permission denied**: WSL·Linux에서는 홈 아래 `~/Desktop/data-engineering` 는 보통 쓰기 가능. Windows Git Bash라면 `C:\Users\본인계정` 아래나 다른 드라이브 경로로 `mkdir -p` 경로를 바꿔서 시도해 볼 수 있습니다.

---

## 핵심 개념 정리

- **작업 디렉토리**: `pwd`, `cd` 로 위치를 확인·이동하고, 실습은 한 폴더(`~/Desktop/data-engineering/day1`)에서 진행.
- **파일 생성·복사·이동**: `echo > 파일`, `cp`, `mv` 로 raw/processed 같은 구조를 만들 수 있음.
- **검색**: `find`(파일 찾기), `grep`(텍스트 패턴) — 로그·설정 파일 다룰 때 자주 쓰는 조합.

---

## 참고

- [04_Linux_기본명령어.md](04_Linux_기본명령어.md) — 사용한 명령어의 옵션(`-p`, `-l`, `-name` 등) 설명.
- [06_Docker_환경구성.md](06_Docker_환경구성.md) — 다음 단계: Docker 환경 구성.

---

## 추가 실습 (5~10분)

아래 **한글 지시**를 보고, 직접 Linux 명령어를 찾아서 수행해 보세요.  
04에서 배운 명령어를 활용하면 됩니다. **Cursor, ChatGPT 등 LLM 도움을 받아도 괜찮습니다.**
다만, agent 보고 직접하라고 하면 안됩니다. 본인이 직접 명령어를 찾아서 치면서 익숙해지는게 중요합니다.

---

### 미션

```
1. 현재 위치를 확인하세요
2. 홈 디렉토리 아래에 mission 폴더를 만들고 그 안으로 이동하세요
3. 그 안에 backup, archive 두 폴더를 한 번에 만드세요
4. memo.txt 파일을 만들고 본인 이름을 저장하세요 (외부 text editor 사용 금지)
5. memo.txt를 backup 폴더로 복사하세요
6. 현재 폴더에서 .txt로 끝나는 파일을 모두 찾으세요
7. memo.txt에서 본인 이름이 있는 줄을 출력하세요
```

---

### 완료 후

다 했으면 아래 명령어를 실행해서 **결과를 강사에게 보여주세요**.

```bash
history | tail -15
```

---

## 그림 출처

본 문서에는 인용한 외부 그림이 없습니다.
