# 05. ETL 프로세스 이해

## ETL이란?

**Extract, Transform, Load**의 약자. 데이터를 **추출·변환·적재**하는 프로세스.

```
Extract (추출) → Transform (변환) → Load (적재)
```

---

## Extract (추출)

- **다양한 소스에서 데이터 추출**.
- 소스: DB (MySQL, PostgreSQL), 파일 (CSV, JSON), API, 로그.
- 도구: Sqoop (DB → HDFS), Kafka (스트리밍), Python 스크립트.

---

## Transform (변환)

데이터를 **분석·서비스에 맞게 변환·정제**.

### 주요 작업

- **필터링**: 불필요한 행 제거.
- **집계**: 그룹별 합계·평균 계산.
- **조인**: 여러 테이블/파일을 결합.
- **데이터 타입 변환**: 문자열 → 날짜, 숫자 등.
- **중복 제거**: 같은 키의 중복 행 제거.
- **결측치 처리**: Null 값을 기본값으로 채우거나 제거.

---

## Load (적재)

- 변환된 데이터를 **목적지에 저장**.
- 목적지: Data Warehouse (Redshift, BigQuery), HDFS, DB.
- 방식: **전체 적재** (Overwrite) vs **증분 적재** (Append).

---

## ELT vs ETL

![ELT vs ETL](img/ETLandELT.png)

**ETL**과 **ELT**는 데이터를 처리하는 **순서가 다른 두 가지 접근 방식**입니다. 어느 것이 더 좋다기보다는, **상황에 따라 적합한 방식을 선택**하면 됩니다.

| 항목      | ETL                        | ELT                               |
| --------- | -------------------------- | --------------------------------- |
| 순서      | Extract → Transform → Load | Extract → Load → Transform        |
| 변환 위치 | 중간 서버 (Spark 등)       | 목적지 (Data Warehouse)           |
| 장점      | 목적지 부하 적음           | 변환 유연성 높음                  |
| 단점      | 중간 서버 필요             | 목적지 컴퓨팅 리소스 필요         |
| 사용 사례 | 전통적 DW, 온프레미스      | 클라우드 DW (BigQuery, Snowflake) |

### 언제 어떤 것을 선택할까?

**ETL을 선택**:
- 온프레미스 환경, 목적지 Data Warehouse의 컴퓨팅 리소스가 제한적일 때.
- 데이터를 미리 정제·변환해서 목적지에 깨끗한 데이터만 적재하고 싶을 때.
- Spark, Hadoop 등 중간 처리 클러스터가 이미 있을 때.

**ELT를 선택**:
- 클라우드 Data Warehouse (BigQuery, Snowflake, Redshift 등)를 사용할 때.
- 목적지에서 강력한 SQL 엔진을 제공하고, 컴퓨팅 리소스를 유연하게 확장할 수 있을 때.
- 원시 데이터를 먼저 적재하고, 나중에 다양한 방식으로 변환·분석하고 싶을 때.

**이번 강의에서는 ETL 방식**(HDFS에서 읽기 → Spark로 변환 → HDFS에 저장)을 다룹니다.

---

## ETLT: 현대적 접근 방식

**ETLT**(Extract, **light** Transform, Load, Transform)는 **ETL과 ELT의 장점을 결합**한 최신 접근 방식으로, 현재 많은 기업에서 채택하고 있습니다.

### 정의

- 데이터 **수집 속도**를 높이는 동시에, **규정 준수·데이터 품질·보안**을 보장하는 방식.
- **순서**: Extract → **light Transform** → Load → **full Transform**

### 순서 상세

1. **Extract**: 데이터베이스·API·로그 등에서 원시 데이터 추출.
2. **light Transform (가벼운 변환)**: 스테이징 영역에서 **최소한의 변환**만 수행.
   - 민감한 데이터 제거·마스킹·암호화 (PII 보호)
   - 불필요한 필드 제거, 기본 포맷 정리
3. **Load**: 준비된 데이터를 Data Warehouse 또는 Data Lake에 적재.
4. **full Transform**: 적재 후, 목적지에서 **복잡한 변환·집계·조인** 수행.

### 특징 및 장점

- **데이터 보안·규정 준수**: 민감한 데이터를 적재 전에 마스킹/암호화 → 개인정보 보호.
- **수집 속도 향상**: 복잡한 변환을 뒤로 미뤄 초기 로딩 속도 개선.
- **유연성**: 적재 후 다양한 분석 요구에 맞게 변환 방식을 쉽게 변경 가능.
- **고객 신뢰**: 개인정보 침해 가능성 제한.

### 왜 ETLT가 대세인가?

- **클라우드 시대**: 원시 데이터를 저장할 공간(Data Lake)이 충분하고, 목적지(DW)의 컴퓨팅 파워가 강력해짐.
- **보안·규정 준수**: GDPR, CCPA 등 개인정보 보호 규정이 강화되면서, **적재 전 민감 데이터 처리**가 필수.
- **실시간·배치 혼합**: 실시간 데이터(Kafka 등)를 가볍게 변환 후 빠르게 적재, 이후 배치로 복잡한 변환.

---

## 핵심 개념 정리

- **ETL**: Extract → Transform → Load. 중간 서버에서 변환 후 적재. 온프레미스·보안 중시.
- **ELT**: Extract → Load → Transform. 목적지에서 변환. 클라우드 DW·유연성.
- **ETLT**: Extract → **light Transform** → Load → **full Transform**. **현대적 접근**. 보안·속도·유연성 모두 확보. **현재 많은 기업이 채택 중**.

---

## 참고

- [ETL vs ELT - AWS](https://aws.amazon.com/ko/compare/the-difference-between-etl-and-elt/)
- [Data Processing with EtLT - XenonStack](https://www.xenonstack.com/blog/data-processing-with-etlt)
- [04_Data_Warehouse_Data_Lake.md](04_Data_Warehouse_Data_Lake.md) — Data Warehouse와 Data Lake 개념 (이전 페이지)
- [06_파이프라인_자동화.md](06_파이프라인_자동화.md) — 다음 단계: 파이프라인 자동화.

---

## 그림 출처

- **img/ETLandELT.png**: ETL vs ELT 비교 다이어그램. 출처: [ETL과 ELT의 차이점 - AWS](https://aws.amazon.com/ko/compare/the-difference-between-etl-and-elt/).
