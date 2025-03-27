# Large Scale Data Processing: Project 2

## Team Members
* Hoiting Mok
* Ruolong Mao

## 2. Exact F2
### Code
```
  def exact_F2(x: RDD[String]) : Long = {
    x.map(p => (p, 1))
      .reduceByKey(_ + _)
      .map{ case (_, count) => count.toLong * count.toLong }
      .reduce(_ + _)
  }
```

### Local Execution
#### Command:
```sh
spark-submit --class "project_2.main" --master "local[*]" target/scala-2.12/project_2_2.12-1.0.jar "./2014to2017.csv" exactF2
```

### GCP Execution


## 3. Tug-of-War
### Code
```
  def Tug_of_War(x: RDD[String], width: Int, depth: Int): Double = {
    val hashFuncs = Array.fill(depth)(
        Array.fill(width)(new four_universal_Radamacher_hash_function)
    )

    val estimates = for (d <- 0 until depth) yield {
        val rowEstimates = for (w <- 0 until width) yield {
            val hashFunc = hashFuncs(d)(w)
            val sketch = x.map(plate => hashFunc.hash(plate)).sum()
            sketch * sketch
        }
        rowEstimates.sum / width
    }

    val sortedEstimates = estimates.sorted
    if (depth % 2 == 1)
        sortedEstimates(depth / 2)
    else
        (sortedEstimates(depth / 2) + sortedEstimates(depth / 2 - 1)) / 2
}
```

### Local Execution
#### Command:
```sh
spark-submit --class project_2.main --master local[*] target/scala-2.12/project_2_2.12-1.0.jar "./2014to2017.csv" ToW 10 3
```
#### Output:
Using a depth of 10 and width of 3, the F2 estimate computed by the Tug-of-War algorithm is quite close to the exact F2 value. Estimated value: **8,776,591,360**  
Exact value: **8,567,966,130**  
Error: **2.4%**

#### Command:
```sh
spark-submit --class project_2.main --master local[*] target/scala-2.12/project_2_2.12-1.0.jar "./2014to2017.csv" ToW 1 1
```
#### Output:
With depth and width set to 1, execution was significantly faster than exact F2 but less accurate. The estimated value fluctuated across different trials, generally producing a lower estimate.

### GCP Execution
(To be updated with results from GCP)

## Baseline: Exact F0
### Local Execution
#### Command:
```sh
spark-submit --class "project_2.main" --master "local[*]" target/scala-2.12/project_2_2.12-1.0.jar "./2014to2017.csv" exactF0
```

## 4. BJKST
### Local Execution
#### Command:
```sh
spark-submit --class "project_2.main" --master "local[*]" target/scala-2.12/project_2_2.12-1.0.jar "./2014to2017.csv" BJKST 100 5
```
#### Output:
The BJKST estimation differed from the exact F0 value by **11.5%**, which was the smallest width we experimented with that stayed within the **±20%** error margin. A larger width closely approximated the exact F0, but execution took over **10 minutes** and resulted in an estimate outside the required range.

### GCP Execution
(To be updated with results from GCP)

## 5. Comparison of Algorithms
- **BJKST vs. Exact F0**: BJKST provides a reasonable approximation with reduced execution time, but accuracy varies with width and trial count.
- **Tug-of-War vs. Exact F2**: Tug-of-War achieves better efficiency but sacrifices accuracy, particularly with lower depth and width settings.

## Conclusion
This project demonstrated the trade-offs between exact and approximate algorithms in large-scale data processing. The experiments show that while exact methods provide higher accuracy, approximate methods significantly improve execution time when configured appropriately.

## Submission
This report is included in the project's repository. The repository link has been submitted via Canvas.
