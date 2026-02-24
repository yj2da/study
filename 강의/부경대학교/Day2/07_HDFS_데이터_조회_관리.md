# 07. HDFS 데이터 조회 및 관리

## 실습 목표

HDFS에 올린 파일을 **읽고**, **복사·이동·다운로드·삭제**해 봅니다.

---

## 실습 단계

> NameNode 컨테이너 안에서 `hdfs dfs` 실행. `podman exec -it namenode bash` 또는 `docker exec -it namenode bash`로 접속

### 1. 파일 내용 확인

```bash
hdfs dfs -cat /user/data/sample_data.csv
hdfs dfs -tail /user/data/sample_data.csv
```

### 2. 파일 복사 및 이동

```bash
hdfs dfs -cp /user/data/sample_data.csv /user/data/backup.csv
hdfs dfs -mv /user/data/backup.csv /user/backup.csv
hdfs dfs -ls /user/
```

### 3. 파일 다운로드 (HDFS → 컨테이너 로컬)

```bash
hdfs dfs -get /user/data/sample_data.csv /tmp/downloaded.csv
cat /tmp/downloaded.csv
```

### 4. 파일 삭제

```bash
hdfs dfs -rm /user/backup.csv
hdfs dfs -ls /user/
```

---

## 체크포인트

**"파일을 다운로드해서 내용을 확인했나요?"**

---

## 핵심 개념 정리

Linux `cat`, `cp`, `mv`, `rm`과 비슷한 이름이지만 **HDFS 경로**를 씁니다.

- `-cat`, `-tail`: 내용 보기
- `-cp`, `-mv`: 복사·이동
- `-get`: HDFS → 로컬
- `-rm`: 삭제

---

## 참고

- [HDFS FileSystem Shell](https://hadoop.apache.org/docs/stable/hadoop-project-dist/hadoop-common/FileSystemShell.html)
- [08_분산저장_시나리오.md](08_분산저장_시나리오.md) — 로그 시나리오 실습.

---

## 그림 출처

본 문서에는 인용한 외부 그림이 없습니다.
