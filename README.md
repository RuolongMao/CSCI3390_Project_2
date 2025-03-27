# Large Scale Data Processing: Project 2

## Team Members
* Hoiting Mok
* Ruolong Mao

## 0. GCP Cluster Configuration
![image](https://github.com/user-attachments/assets/c909f40b-b8df-4b2e-8945-2927c036ee06)

## 1. Exact F2
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
#### Output:
![image](https://github.com/user-attachments/assets/f856467b-c549-4623-bae5-58fc3b39c196)

### GCP Execution
![image](https://github.com/user-attachments/assets/180e488e-f250-4285-8df5-f4980d10b073)

## 2. Tug-of-War
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
![image](https://github.com/user-attachments/assets/81b58d31-81f3-4a0a-bee4-623125ce7a4f)

When using a depth of 10 and width of 3, The F2 estimate computed by the Tug of War algorithm is quite close to the exact F2 value. Estimated value is **8776591360** and the exact value is **8567966130**, yielding a **2.4%** and **2.03%** error on the two trials. The Tug-of-War algorithm here is performing stably on the local machine using depth of 10 and width of 3. 

#### Command:
```sh
spark-submit --class project_2.main --master local[*] target/scala-2.12/project_2_2.12-1.0.jar "./2014to2017.csv" ToW 1 1
```
#### Output:
![image](https://github.com/user-attachments/assets/83917b8a-4e87-4daa-91df-e9ae7f5884f4)

The output with depth and width equal to 1 was calculated significantly faster than the exact F2 method but with much less accuracy. How close the estimated value is in comparison to exact F2 seems to fluctuate across different trials – mostly less than the value we are looking for. 

### GCP Execution
![image](https://github.com/user-attachments/assets/9ebcc29c-8838-4ad1-a1d7-1f0edd14f5a5)

## 3.1 Baseline: Exact F0
### Local Execution
#### Command:
```sh
spark-submit --class "project_2.main" --master "local[*]" target/scala-2.12/project_2_2.12-1.0.jar "./2014to2017.csv" exactF0
```
#### Output:
![image](https://github.com/user-attachments/assets/677a36c0-f631-4c3e-b4e9-2c56f988cefe)


## 3.2 BJKST
### Code
```
  def BJKST(x: RDD[String], width: Int, trials: Int): Double = {
    val estimates = (0 until trials).map { _ =>
        val hashFunc = new hash_function(Long.MaxValue)
        val initialSketch = new BJKSTSketch(Set.empty[(String, Int)], 0, width)
        
        val finalSketch = x.treeAggregate(initialSketch)(
            seqOp = (sketch, str) => {
                val hashVal = hashFunc.hash(str)
                val zeros = hashFunc.zeroes(hashVal)
                sketch.add_string(str, zeros)
            },
            combOp = (sketch1, sketch2) => sketch1 + sketch2
        )
        
        finalSketch.bucket.size * math.pow(2, finalSketch.z)
    }
    
    val sorted = estimates.sorted
    if (trials % 2 == 1) sorted(trials / 2)
    else (sorted(trials / 2 - 1) + sorted(trials / 2)) / 2
}
```

### Local Execution
#### Command:
```sh
spark-submit --class "project_2.main" --master "local[*]" target/scala-2.12/project_2_2.12-1.0.jar "./2014to2017.csv" BJKST 100 5
```
#### Output:
![image](https://github.com/user-attachments/assets/5236d103-b245-4117-8625-a27be261aef7)

The BJKST estimation differs from the exact F0 value by **11.5%**, which is the smallest width that we experimented to be within +/-20% of the desired value. We initially started with a large width that closely resembled the exact F0. The trial ran for more than 10 minutes and arrived at an estimate that was well outside of the +/-20% bound. As we slowly decremented the width, the algorithm was able to finish its estimate faster and faster as well as producing a good estimate.

### GCP Execution
![image](https://github.com/user-attachments/assets/9f8e6d9f-4ff5-4c39-a8cc-8d942229112a)

