# [95_Spark](./95_Spark)

## 기본 정보

- 목적 : Spark의 구조, 장단점, 운영 상의 주의점 등을 알자
- 기간 : 2021/10/21 ~ 2021/11/04
- Ref : 
  - [아파치 > 스파크 > 닥스](https://spark.apache.org/docs/2.3.1/api/scala/index.html#org.apache.spark.sql.Dataset)

## 스파크 개념

### 스파크의 구조

- 하둡과 유사하게, 여러 컴퓨터를 엮어서 사용한다. 그래서 spark에서도 전체 리소스를 관리하는 master 와, 실제로 연산 작업을 하는 slave의 master-slave 구조가 나타난다. 또한 mapreduce와 같은 병렬 연산도 지원한다.

- 하둡과 다른 점은, 하둡은 디스크에 데이터를 저장하는 반면, 스파크는 메모리에 저장한다는 것이다. 데이터 IO의 속도는 `메모리(spark) << 다른 네트워크의 메모리(redis) << 내 서버의 디스크(hadoop)` 순이다.

  

### 스파크의 장단점

- 스파크는 데이터를 메모리에 저장하기 때문에, 속도가 매우 빠르다. 하지만 메모리는 서버가 죽으면 같이 죽기 때문에, 장기간 저장해야 하는 데이터에는 적합하지 않다.

  

### spark의 단위

- spark에는 `rdd, dataframe, dataset`이 있다. 셋다 개발자가 신경쓰지 않아도 알아서 병렬 처리를 해주는 똑똑한 아이들이다.

- 태초에는 rdd가 있었다.

- 그런데 데이터를 처리함에 있어서 데이터의 구조를 미리 알고 있다면, 더욱 빠르고 효율적인 처리가 가능할 것이다. 이러한 흐름에서 나온 것이 스키마를 갖고 있는 dataframe이다.

- dataset은 가장 최근에 나온 개념으로, dataframe의 행마저 쪼개어 연산이 더 용이하다.

  

## Spark 예시

### dataframe 사용 예시

```scala
// count
val itemDF = spark.read.parquet("hdfs://path/file.parquet")
itemDF.count

// groupby
// 방법1. sql
itemDF.createOrReplaceTempView("items")
val cnt_sql = spark.sql("SELECT COUNT(id) as cnt, mallNo FROM items GROUP BY mallNo ORDER BY cnt DESC LIMIT 100")
cnt_sql.show(100)

// 방법2. df api
val cnt_dfapi = itemDF.groupBy("mallNo").agg(count("id") as "cnt").orderBy(desc("cnt")).limit(100)
cnt_dfapi2.show()

// 연산
// df api를 활용한 간단한 연산
itemDF.withColumn("leafCategory", when($"categoryLevel" === "4", itemDF("category4Id")).otherwise(itemDF("category3Id"))).show()

// udf를 활용한 복잡한 연산
import org.apache.spark.sql.functions._

val sampleUdf = udf( (categoryLevel: String, category1Id: String, category2Id: String, category3Id: String, category4Id: String ) => {
    categoryLevel match {
        case "4" => category4Id
        case "3" => category3Id
        case "2" => category2Id
        case "1" => category1Id
        case _ => "error"
    }
} )

itemDF.withColumn("sample", sampleUdf($"categoryLevel", $"category1Id", $"category2Id", $"category3Id", $"category4Id"))
```



### 운영 팁

- 데이터를 분산 저장할 때에 크기를 고르게 하는 것이 중요하다. 잘못된 예로, 한 노드는 10MB를 다른 노드는 1KB를 처리하는 것보다는, 각자 5MB씩 처리하는 것이 더 빠르고 효율적이다. 그러므로 데이터를 처리할 때에 shuffle하여 partition하는 것이 필요하다
- parquet는 .csv와 같은 파일 형식 중에 하나이다. parquet는 컬럼별로 데이터를 저장한다. 컬럼은 데이터가 균일하므로 압축하기에 좋다. 또한 select age from people처럼 서비스에서 컬럼별로 읽는 경우가 많으므로 io에 좋다

