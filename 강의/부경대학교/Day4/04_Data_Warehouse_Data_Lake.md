# 04. Data Warehouse와 Data Lake

## 왜 이 개념이 중요한가?

대량의 데이터가 발생하면서 **'Data Lake'** 개념이 생겼습니다. 즉, **모든 데이터를 일단 저장한 후, 용도에 따라 가져다 쓰는** 논리입니다.

데이터 엔지니어는 **Data Warehouse, Data Lake, 머신러닝, 클라우드 마이그레이션** 등 프로젝트에 따라 **데이터 통합 접근 방식(ETL/ELT/ETLT)을 이해하고 선택**하는 것이 매우 중요합니다.

---

## 쉽게 이해하기: 호수 vs 창고

### Data Warehouse = 물류 창고 📦

- **쿠팡 물류 창고**를 생각하면 됩니다.
- **어디에 뭐가 있는지 다 정리**되어 있습니다. (A구역 1번 선반 = 전자제품)
- 필요한 물건을 **빠르게 찾아서** "로켓배송"으로 당일·다음날 배송할 수 있습니다.
- **하지만 창고에는 한계가 있습니다**:
  - 쿠팡에서 모든 제품이 로켓배송(당일·다음날 배송)이 되는 건 아닙니다.
  - 창고 공간이 제한적이라 **일부 제품만 정리해서 보관 가능**.
  - 미리 정해진 형식·규격에 맞는 것만 넣을 수 있음.

### Data Lake = 호수 🏞️

- **호수(Lake)** 는 **거대합니다**.
- **형식·규격에 상관없이** 물, 물고기, 수초, 돌멩이, 심지어 쓰레기까지 — **일단 다 물 속에 넣을 수 있습니다**.
- 나중에 **필요할 때 건져서** 쓰면 됩니다. (낚시, 수질 조사, 정화 등)
- **거대하고 유연하지만, 정리되지 않음**.

### 용어의 유래

- **Data Warehouse**: Bill Inmon이 1988년 IBM Systems Journal에서 "business data warehouse"라는 용어를 처음 사용. 1992년 "Building the Data Warehouse" 책 출간. **창고(Warehouse)** 처럼 데이터를 정리해서 보관한다는 의미.
- **Data Lake**: James Dixon (Pentaho CTO)이 2010년 10월 블로그에서 처음 사용. "Data Mart가 병에 담긴 생수(정제되고 포장됨)라면, Data Lake는 자연 상태의 거대한 물(호수)"이라고 비유. **호수(Lake)** 처럼 원시 데이터를 있는 그대로 담는다는 의미.

---

## Data Warehouse (데이터 웨어하우스) 📦

### 정의

- **구조화된 데이터**를 저장하고, **분석·리포트**에 최적화된 중앙 저장소.
- **스키마 온 라이트(Schema-on-Write)**: 데이터를 적재할 때 스키마를 정의하고, 그에 맞게 정제·변환 후 저장.
- **비유**: 쿠팡 물류 창고에 물건을 넣을 때, **어디에 넣을지 미리 정해서 정리**하는 것. (전자제품은 A구역, 식품은 B구역) 그래서 나중에 필요한 데이터를 **빠르게 찾을 수 있음**.

### 특징

- **정형 데이터**: 테이블(행·열) 형태. SQL 쿼리 최적화.
- **OLAP (Online Analytical Processing)**: 집계·분석 쿼리에 특화.
- **데이터 품질**: 적재 전 정제·검증 → 높은 품질 보장.

### 기술

- **전통적**: Oracle, Teradata, IBM Db2 Warehouse.
- **클라우드**: Amazon Redshift, Google BigQuery, Snowflake, Azure Synapse Analytics.

### 사용 사례

- 일일·월간 매출 리포트.
- BI 도구(Tableau, Looker)로 대시보드 생성.
- 경영진 의사결정용 집계 데이터.

---

## Data Lake (데이터 레이크) 🏞️

### 정의

- **원시 데이터**를 **있는 그대로** 저장하는 대규모 저장소.
- **스키마 온 리드(Schema-on-Read)**: 데이터를 읽을 때 스키마를 해석. 저장 시에는 스키마 강제 안 함.
- **비유**: 호수에 물건을 넣을 때는 **분류하지 않고 일단 다 던져 넣음**. 나중에 필요할 때 건져서 "이건 물고기, 이건 돌멩이"라고 분류·정제·분석.

### 특징

- **모든 형태의 데이터**: 정형(CSV, DB), 반정형(JSON, XML), 비정형(이미지, 동영상, 로그).
- **유연성**: 나중에 다양한 방식으로 분석·변환 가능.
- **저비용**: 클라우드 오브젝트 스토리지(S3, GCS) 활용 시 저렴.

### 기술

- **HDFS** (Hadoop Distributed File System)
- **AWS S3**, **Google Cloud Storage**, **Azure Blob Storage**
- **Delta Lake**, **Apache Iceberg** (Data Lake + ACID 트랜잭션)

### 사용 사례

- 머신러닝 학습 데이터 저장.
- 로그·센서 데이터 원본 보관.
- 탐색적 데이터 분석(EDA).

---

## Data Warehouse vs Data Lake

| 항목            | Data Warehouse 📦              | Data Lake 🏞️                  |
| --------------- | ------------------------------ | ----------------------------- |
| **비유**        | 쿠팡 물류 창고 (정리됨)        | 호수 (다양한 것 담김)         |
| **데이터 형태** | 정형 (테이블)                  | 정형·반정형·비정형            |
| **스키마**      | Schema-on-Write (적재 시 정의) | Schema-on-Read (읽을 때 해석) |
| **목적**        | 분석·리포트 (OLAP)             | 원시 데이터 보관, 탐색·ML     |
| **데이터 품질** | 높음 (적재 전 정제)            | 낮음 (원시 그대로)            |
| **검색 속도**   | 빠름 (정리되어 있음)           | 느림 (필요할 때 정제)         |
| **비용**        | 상대적으로 높음                | 저렴 (오브젝트 스토리지)      |
| **사용자**      | 비즈니스 분석가, 경영진        | 데이터 사이언티스트, 엔지니어 |
| **기술**        | Redshift, BigQuery, Snowflake  | HDFS, S3, Delta Lake          |

---

## Data Lakehouse (데이터 레이크하우스)

최근에는 **Data Warehouse**와 **Data Lake**의 장점을 결합한 **Data Lakehouse** 개념이 등장했습니다.

### 특징

- Data Lake의 **유연성·저비용** + Data Warehouse의 **ACID 트랜잭션·쿼리 성능**.
- 기술: **Delta Lake**, **Apache Iceberg**, **Apache Hudi**.

### 사용 사례

- 원시 데이터 저장 + SQL 쿼리 + 머신러닝을 **하나의 플랫폼**에서.
- Databricks, Snowflake 등이 Lakehouse 아키텍처 지원.

---

## 핵심 개념 정리

- **Data Warehouse 📦**: 쿠팡 물류 창고처럼 **정리된 창고**. 창고 공간 한계로 **일부만 보관 가능**. 정형 데이터, 빠른 검색, 높은 품질. Schema-on-Write.
- **Data Lake 🏞️**: 호수처럼 **거대하고 다양한 것을 담는 저장소**. 형식·규격 상관없이 **일단 다 넣고, 필요할 때 건짐**. 모든 형태 데이터, 원시 보관, 유연성·저비용. Schema-on-Read.
- **Data Lakehouse**: Warehouse + Lake 장점 결합. Delta Lake, Iceberg 등.
- **ETL/ELT/ETLT 선택**: 목적지(DW vs Lake)와 요구사항(보안, 속도, 유연성)에 따라 결정.

---

## 참고

- [05_ETL_프로세스.md](05_ETL_프로세스.md) — 다음 단계: ETL 프로세스.
- [Data Warehouse vs Data Lake - AWS](https://aws.amazon.com/ko/compare/the-difference-between-a-data-warehouse-data-lake-and-data-mart/)
- [Delta Lake](https://delta.io/)
- [James Dixon's Blog - Pentaho, Hadoop, and Data Lakes (2010)](https://jamesdixon.wordpress.com/2010/10/14/pentaho-hadoop-and-data-lakes/) — Data Lake 용어 최초 사용
- [A Brief History of Data Lakes - DATAVERSITY](https://dataversity.net/brief-history-data-lakes/)
- [Bill Inmon - Wikipedia](https://en.wikipedia.org/wiki/Bill_Inmon) — Data Warehouse의 아버지

---

## 그림 출처

본 문서에는 인용한 외부 그림이 없습니다.
