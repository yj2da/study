# 04. PySpark 기본 문법 소개

## PySpark란?

- **PySpark**: Python에서 Spark를 사용할 수 있게 해주는 API.
- Spark의 Scala 코어 위에 **Py4J**로 연결.
- 데이터 사이언티스트·분석가에게 인기 (Python 생태계 활용).

---

## SparkSession

- **Spark 2.0+** 에서 DataFrame/SQL 작업의 진입점.
- SparkContext, SQLContext, HiveContext를 통합.

### SparkSession 생성

```python
from pyspark.sql import SparkSession

spark = SparkSession.builder \
    .appName("MyApp") \
    .getOrCreate()
```

- `pyspark` 셸에서는 **이미 `spark` 변수로 생성되어 있음**.

---

## DataFrame 기본 연산

### 데이터 읽기

```python
# CSV
df = spark.read.csv("/path/to/file.csv", header=True, inferSchema=True)

# JSON
df = spark.read.json("/path/to/file.json")

# Parquet
df = spark.read.parquet("/path/to/file.parquet")
```

### 데이터 조회

```python
df.show()           # 상위 20개 행 출력
df.show(5)          # 상위 5개 행
df.printSchema()    # 스키마(컬럼 타입) 출력
df.columns          # 컬럼 이름 리스트
df.count()          # 행 수
df.describe().show() # 기술 통계
```

### 컬럼 선택

```python
df.select("name", "age").show()
df.select(df.name, df.age).show()
```

### 필터링

```python
df.filter(df.age >= 30).show()
df.where(df.age >= 30).show()  # filter와 동일

# 여러 조건
df.filter((df.age >= 30) & (df.city == "Seoul")).show()
```

### 집계

```python
df.groupBy("city").count().show()
df.groupBy("city").avg("salary").show()

# 여러 집계 함수
from pyspark.sql import functions as F
df.groupBy("city").agg(
    F.count("id").alias("count"),
    F.avg("age").alias("avg_age"),
    F.max("salary").alias("max_salary")
).show()
```

### 정렬

```python
df.orderBy("age").show()                      # 오름차순
df.orderBy("age", ascending=False).show()     # 내림차순
df.orderBy(["city", "age"]).show()            # 여러 컬럼
```

---

## Pandas vs PySpark DataFrame

### 왜 Pandas와 비교하는가?

많은 데이터 분석가와 데이터 사이언티스트가 **Pandas**에 익숙합니다. PySpark DataFrame은 Pandas와 **API가 유사**하지만, 내부 동작 방식과 사용 목적이 다릅니다.

**핵심 질문**: "내가 익숙한 Pandas를 쓰면 되는데, 왜 PySpark를 배워야 하나?"

### 언제 Pandas를 쓰고, 언제 PySpark를 쓸까?

**Pandas를 사용하는 경우:**
- 데이터가 **메모리에 들어가는 크기** (보통 수 GB 이하)
- 단일 머신에서 빠르게 분석하고 싶을 때
- 탐색적 데이터 분석 (EDA), 프로토타이핑

**PySpark를 사용하는 경우:**
- 데이터가 **수십 GB ~ TB 이상**
- 여러 머신의 리소스를 활용해야 할 때
- 프로덕션 데이터 파이프라인 구축

### 비교표

| 항목 | Pandas | PySpark |
|------|--------|---------|
| 데이터 크기 | 메모리에 맞는 크기 (수 GB) | 분산 처리로 대용량 (TB+) |
| 실행 방식 | 즉시 실행 (Eager) | Lazy Evaluation |
| 병렬 처리 | 단일 머신 | 클러스터 분산 |
| API | pandas API | DataFrame API (유사) |
| 학습 곡선 | 쉬움 | 중간 (분산 개념 필요) |
| 속도 (소규모) | 빠름 | 오버헤드 있음 |
| 속도 (대규모) | 불가능 | 빠름 |

### 실무 팁

1. **작은 데이터**: Pandas로 시작하세요
2. **데이터가 커지면**: PySpark로 전환하세요
3. **API 유사성**: Pandas 경험이 있다면 PySpark 학습이 수월합니다
4. **함께 사용**: PySpark로 대용량 처리 → 결과를 Pandas로 변환하여 시각화
   ```python
   # PySpark DataFrame → Pandas DataFrame
   pandas_df = spark_df.toPandas()
   ```

---

## 핵심 개념 정리

- **SparkSession**: DataFrame/SQL 진입점. `spark.read`, `spark.sql` 등 사용.
- **DataFrame API**: `select`, `filter`, `groupBy`, `agg`, `orderBy` 등 SQL과 유사.
- **Lazy**: Transformation은 기록만, Action(`show`, `count`)이 호출되어야 실행.

---

## 참고

- [05_PySpark_환경설정.md](05_PySpark_환경설정.md) — 다음 단계: PySpark 환경 설정.
- [PySpark API 문서](https://spark.apache.org/docs/latest/api/python/)

---

## 그림 출처

본 문서에 사용한 그림이 있으면 `img/` 폴더에 두고 여기에 표로 적어 주세요.
