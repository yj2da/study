# 07-2. 실전 ETLT 파이프라인 구현 (종합 실습)

## 실습 목표

지금까지 배운 내용을 활용하여 **ETLT 파이프라인**을 직접 구현합니다.

> 💡 **이번 실습은 여러분이 직접 해보는 시간입니다!**
> - 정답 코드는 제공하지 않습니다.
> - **힌트는 최대한 보지 말고**, 2번 이상 생각해도 정말 모르겠을 때만 확인하세요.
> - 처음이라 어렵겠지만, **실무에서는 스스로 생각하고 해결하는 능력이 중요**합니다.
> - 물론 실무에서는 LLM 도움을 받을 수 있지만, 이번은 **연습**이니 최대한 외부 도움 없이 스스로 해보세요.
> - 앞에서 배운 내용(06~07)을 참고하여 직접 작성해보세요.
> - 정말 막히면 강사에게 질문하세요!

---

## 시나리오: 전자상거래 주문 데이터 처리

온라인 쇼핑몰의 주문 데이터를 처리하는 ETLT 파이프라인을 구축합니다.

### 1. 데이터 준비

다음 샘플 데이터를 생성하고 **HDFS의 `/user/data/raw/orders/` 경로에 업로드**하세요.

**파일명**: `orders_raw.csv`

```csv
order_id,user_id,user_name,user_email,product,quantity,price,date
1,101,김철수,chulsoo@example.com,Laptop,1,1000,2024-02-01
2,102,이영희,younghee@example.com,Mouse,2,20,2024-02-01
3,101,김철수,chulsoo@example.com,Keyboard,1,50,2024-02-02
4,103,박민수,minsu@example.com,Monitor,1,300,2024-02-02
5,102,이영희,younghee@example.com,Laptop,1,1000,2024-02-03
6,104,최지은,jieun@example.com,Mouse,3,20,2024-02-03
7,103,박민수,minsu@example.com,Keyboard,2,50,2024-02-04
8,105,정수진,sujin@example.com,Monitor,1,300,2024-02-04
9,101,김철수,chulsoo@example.com,Mouse,5,20,2024-02-05
10,102,이영희,younghee@example.com,Keyboard,1,50,2024-02-05
```

> 💡 **힌트**: Day2에서 배운 HDFS 명령어를 활용하세요!

**데이터 설명:**
- `order_id`: 주문 번호
- `user_id`: 고객 ID
- `user_name`: 고객 이름 (**개인정보**)
- `user_email`: 고객 이메일 (**개인정보**)
- `product`: 제품명
- `quantity`: 수량
- `price`: 단가
- `date`: 주문 날짜

---

## 과제

### 과제 1: Extract + light Transform (ETLT의 Et)

**요구사항:**
1. HDFS에서 `orders_raw.csv` 데이터를 읽어오세요.
2. **개인정보 보호**를 위해 다음 변환을 수행하세요:
   - `user_name` 컬럼 삭제
   - `user_email` 컬럼을 마스킹 (예: `chulsoo@example.com` → `ch****@example.com`)
3. 변환된 데이터를 확인하세요.

<details>
<summary>힌트 1: 데이터 읽기</summary>

```python
df = spark.read.csv("hdfs://namenode:8020/user/data/raw/orders/orders_raw.csv", 
                    header=True, inferSchema=True)
```

</details>

<details>
<summary>힌트 2: 이메일 마스킹</summary>

`regexp_replace()` 함수 사용:
```python
from pyspark.sql import functions as F

# 이메일 앞 2글자만 남기고 나머지는 *로 마스킹
df_masked = df.withColumn("user_email", 
    F.regexp_replace("user_email", "^(.{2})(.*)(@.*)$", "$1****$3")
)
```

</details>

---

### 과제 2: Load (ETLT의 L)

**요구사항:**
1. light Transform된 데이터를 HDFS의 **스테이징 영역**에 저장하세요.
   - 경로: `hdfs://namenode:8020/user/data/staging/orders/`
   - 형식: Parquet
2. 저장된 데이터를 다시 읽어서 확인하세요.

<details>
<summary>힌트</summary>

```python
df_masked.write.parquet("hdfs://namenode:8020/user/data/staging/orders/", 
                        mode="overwrite")
```

</details>

---

### 과제 3: full Transform (ETLT의 T)

**요구사항:**
1. 스테이징 영역에서 데이터를 읽어오세요.
2. 다음 변환을 수행하세요:
   - 총 매출 계산: `total_amount = quantity * price`
   - 날짜 타입 변환: `date` 컬럼을 날짜 타입으로 변환
3. **제품별 통계**를 계산하세요:
   - 총 판매량 (`total_quantity`)
   - 총 매출 (`total_revenue`)
   - 평균 주문 수량 (`avg_quantity`)
   - 주문 건수 (`order_count`)
4. 매출이 높은 순서로 정렬하세요.

**예상 결과:**
```
+--------+--------------+-------------+------------+-----------+
|product |total_quantity|total_revenue|avg_quantity|order_count|
+--------+--------------+-------------+------------+-----------+
|Laptop  |2             |2000.0       |1.0         |2          |
|Monitor |2             |600.0        |1.0         |2          |
|Mouse   |10            |200.0        |3.33        |3          |
|Keyboard|4             |200.0        |1.33        |3          |
+--------+--------------+-------------+------------+-----------+
```

<details>
<summary>힌트 1: 총 매출 계산</summary>

```python
df_with_total = df.withColumn("total_amount", F.col("quantity") * F.col("price"))
```

</details>

<details>
<summary>힌트 2: 날짜 변환</summary>

```python
df_with_date = df.withColumn("date", F.to_date("date"))
```

</details>

<details>
<summary>힌트 3: 제품별 집계</summary>

```python
product_stats = df.groupBy("product").agg(
    F.sum("quantity").alias("total_quantity"),
    F.sum("total_amount").alias("total_revenue"),
    F.avg("quantity").alias("avg_quantity"),
    F.count("order_id").alias("order_count")
).orderBy(F.desc("total_revenue"))
```

</details>

---

### 과제 4: 최종 Load + 검증

**요구사항:**
1. 제품별 통계를 HDFS에 저장하세요:
   - 경로: `hdfs://namenode:8020/user/data/output/product_stats_{timestamp}`
   - 형식: Parquet
2. **데이터 품질 검증**을 수행하세요:
   - 총 제품 수 확인 (4개여야 함)
   - Null 값 체크
   - 음수 값 체크 (total_revenue, total_quantity가 0 이상이어야 함)
3. 검증 통과 시 성공 메시지 출력.

<details>
<summary>힌트: 검증 로직</summary>

```python
# Null 체크
null_count = product_stats.filter(
    F.col("total_revenue").isNull() | F.col("total_quantity").isNull()
).count()

# 음수 체크
invalid_count = product_stats.filter(
    (F.col("total_revenue") < 0) | (F.col("total_quantity") < 0)
).count()

if null_count == 0 and invalid_count == 0:
    print("✅ Validation passed")
else:
    print(f"❌ Validation failed: {null_count} nulls, {invalid_count} invalid values")
```

</details>

---

### 과제 5: 전체 파이프라인 함수화 (도전 과제)

**요구사항:**
1. 위의 모든 단계를 하나의 함수 `run_etlt_pipeline()`로 만드세요.
2. 함수는 다음을 수행해야 합니다:
   - Extract + light Transform (개인정보 마스킹)
   - Load to Staging
   - full Transform (집계·통계)
   - Load to Output
   - Validation
3. 에러 처리 (`try-except`)와 로깅을 추가하세요.

<details>
<summary>힌트: 함수 구조</summary>

```python
def run_etlt_pipeline(input_path, staging_path, output_base_path):
    try:
        # Extract + light Transform
        print("📥 [1/4] Extract + light Transform...")
        # ... 개인정보 마스킹 ...
        
        # Load to Staging
        print("💾 [2/4] Load to Staging...")
        # ... staging에 저장 ...
        
        # full Transform
        print("🔄 [3/4] full Transform...")
        # ... 집계·통계 ...
        
        # Load to Output + Validation
        print("💾 [4/4] Load to Output + Validation...")
        # ... 저장 및 검증 ...
        
        return output_path
    except Exception as e:
        print(f"❌ Pipeline failed: {e}")
        raise
```

</details>

---

## 체크포인트

**"ETLT 파이프라인이 성공적으로 실행되었나요?"**

다음을 확인하세요:
- [ ] 개인정보(user_name, user_email)가 마스킹되었나요?
- [ ] 스테이징 영역에 데이터가 저장되었나요?
- [ ] 제품별 통계가 올바르게 계산되었나요?
- [ ] 최종 결과가 HDFS에 저장되었나요?
- [ ] 데이터 품질 검증을 통과했나요?

---

## 핵심 개념 정리

- **ETLT**: Extract → **light Transform** (개인정보 보호) → Load → **full Transform** (집계·분석).
- **스테이징 영역**: 원시 데이터와 최종 데이터 사이의 중간 저장소. 민감 데이터 처리 후 임시 저장.
- **개인정보 보호**: GDPR, CCPA 등 규정 준수를 위해 적재 전 마스킹·암호화 필수.
- **데이터 품질 검증**: Null, 중복, 비즈니스 로직 검증으로 데이터 신뢰성 확보.

---

## 참고

- [05_ETL_프로세스.md](05_ETL_프로세스.md) — ETLT 개념 복습
- [06_파이프라인_자동화.md](06_파이프라인_자동화.md) — 파이프라인 구조 참고
- [07_파이프라인_검증.md](07_파이프라인_검증.md) — 검증 로직 참고
- [09_QA_마무리.md](09_QA_마무리.md) — 다음 단계: Q&A 및 마무리

---

## 그림 출처

본 문서에는 별도 이미지가 사용되지 않았습니다.
