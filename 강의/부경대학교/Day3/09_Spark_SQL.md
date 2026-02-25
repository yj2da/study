# 09. Spark SQL

## 실습 목표

Spark SQL을 사용하여 **SQL 쿼리로 데이터를 분석**합니다.

---

## Spark SQL이란?

- Spark에서 SQL 쿼리를 사용할 수 있게 해주는 모듈
- DataFrame을 테이블처럼 다룰 수 있음
- SQL에 익숙한 사용자도 쉽게 Spark 사용 가능

---

## 실습 단계

### 1. DataFrame을 임시 테이블로 등록

```python
from pyspark.sql import SparkSession

spark = SparkSession.builder \
    .appName("SparkSQL_Example") \
    .master("spark://spark-master:7077") \
    .getOrCreate()

# 데이터 로드
df = spark.read.csv("/home/jovyan/work/data/users.csv", header=True, inferSchema=True)

# 임시 테이블로 등록
df.createOrReplaceTempView("users")
```

> 💡 **Temp View**: 
> - `createOrReplaceTempView()`는 현재 SparkSession에서만 유효한 임시 테이블을 생성합니다.
> - SQL 쿼리에서 테이블명으로 사용할 수 있습니다.

### 2. SQL 쿼리 실행

```python
# 전체 데이터 조회
spark.sql("SELECT * FROM users").show()

# 조건 필터링
spark.sql("SELECT * FROM users WHERE age >= 30").show()

# 집계
spark.sql("""
    SELECT city, COUNT(*) as count, AVG(salary) as avg_salary
    FROM users
    GROUP BY city
    ORDER BY avg_salary DESC
""").show()

# 복잡한 쿼리
spark.sql("""
    SELECT 
        city,
        COUNT(*) as total_people,
        AVG(age) as avg_age,
        MAX(salary) as max_salary,
        MIN(salary) as min_salary
    FROM users
    WHERE age >= 25
    GROUP BY city
    HAVING COUNT(*) >= 1
    ORDER BY total_people DESC
""").show()
```

### 3. SQL 결과를 DataFrame으로 저장

```python
# SQL 쿼리 결과를 DataFrame으로 받기
result = spark.sql("""
    SELECT city, AVG(salary) as avg_salary
    FROM users
    GROUP BY city
""")

# DataFrame 메서드 사용 가능
result.filter(result.avg_salary > 55000).show()

# 결과 저장 (HDFS에 저장 권장)
result.write.csv("hdfs://namenode:8020/user/data/output/city_salary", header=True, mode="overwrite")

# 저장 확인
spark.read.csv("hdfs://namenode:8020/user/data/output/city_salary", header=True).show()
```

> 💡 **로컬 파일 시스템 저장 시 주의**:
> - 로컬 경로(`/home/jovyan/work/data/`)에 저장하면 권한 문제 발생 가능
> - HDFS에 저장하는 것을 권장 (`hdfs://namenode:8020/...`)
> - 로컬 저장이 필요하면 호스트에서 권한 설정:
>   ```bash
>   chmod -R 777 ~/Desktop/data-engineering/day3/data
>   ```

### 4. HDFS 데이터로 SQL 실행

```python
# HDFS 데이터 로드
hdfs_df = spark.read.csv("hdfs://namenode:8020/user/data/sample_data.csv", header=True, inferSchema=True)
hdfs_df.createOrReplaceTempView("hdfs_users")

# SQL 쿼리
spark.sql("""
    SELECT city, COUNT(*) as count
    FROM hdfs_users
    GROUP BY city
""").show()
```

---

## DataFrame API vs Spark SQL 비교

같은 작업을 두 가지 방법으로 수행할 수 있습니다.

**DataFrame API:**
```python
df.groupBy("city").agg(F.avg("salary").alias("avg_salary")).show()
```

**Spark SQL:**
```python
spark.sql("SELECT city, AVG(salary) as avg_salary FROM users GROUP BY city").show()
```

> 💡 **어떤 것을 사용할까?**
> - **DataFrame API**: 프로그래밍 방식, 타입 안전성, IDE 자동완성
> - **Spark SQL**: SQL 문법에 익숙한 경우, 복잡한 쿼리 작성 시 가독성 좋음
> - 실무에서는 두 가지를 혼용하여 사용합니다.

---

## 체크포인트

**"Spark SQL로 30세 이상이면서 연봉이 60,000 이상인 사람을 찾아보세요"**

`spark.sql()`을 사용하여 다음 조건을 만족하는 쿼리를 작성하세요:
- 나이: 30세 이상
- 연봉: 60,000 이상
- 결과를 연봉 내림차순으로 정렬
- `name`, `age`, `city`, `salary` 컬럼 선택

<details>
<summary>힌트</summary>

`WHERE` 절에서 `AND`를 사용하고, `ORDER BY`로 정렬하세요.

```python
spark.sql("""
    SELECT ...
    FROM users
    WHERE ... AND ...
    ORDER BY ...
""").show()
```

</details>

<details>
<summary>정답 확인</summary>

```python
spark.sql("""
    SELECT name, age, city, salary
    FROM users
    WHERE age >= 30 AND salary >= 60000
    ORDER BY salary DESC
""").show()
```

예상 결과:
+-------+---+-------+------+
|   name|age|   city|salary|
+-------+---+-------+------+
|Charlie| 35|Incheon| 70000|
|    Eve| 32|  Busan| 65000|
|    Bob| 30|  Busan| 60000|
+-------+---+-------+------+

</details>

---

## 핵심 개념 정리

- **Spark SQL**: SQL 쿼리로 Spark 데이터 분석
- **Temp View**: `createOrReplaceTempView()`로 DataFrame을 SQL 테이블로 등록
- **SQL ↔ DataFrame**: SQL 결과는 DataFrame으로 받아 추가 처리 가능
- **실무 활용**: 복잡한 집계는 SQL로, 데이터 처리는 DataFrame API로 혼용

---

## 참고

- [10_실전_데이터분석.md](10_실전_데이터분석.md) — 다음 단계: 종합 실습.
- [Spark SQL Guide](https://spark.apache.org/docs/latest/sql-programming-guide.html)

---

## 그림 출처

본 문서에는 인용한 외부 그림이 없습니다.
