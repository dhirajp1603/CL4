# 📊 LAB 3: Matrix Multiplication using MapReduce (Python - Hadoop Streaming)

---

## 🧠 Objective

To perform matrix multiplication using the MapReduce programming model with Hadoop Streaming in Python.

---

## 📚 Theory

Matrix multiplication is a mathematical operation where two matrices **A** and **B** are multiplied to produce a result matrix **C**.

For matrices:

* A (m × n)
* B (n × p)

Result:

* C (m × p)

Each element:
C[i][j] = Σ (A[i][k] × B[k][j])



---




---

## ⚙️ Execution Steps

### 1️⃣ Create Input File

```id="rql2nm"
nano input.txt
```

---

## 📂 Input Data

**input.txt**

```id="x4m2qa"
A,0,0,1
A,0,1,2
A,1,0,3
A,1,1,4

B,0,0,5
B,0,1,6
B,1,0,7
B,1,1,8
```

👉 Matrix A:

```
1  2
3  4
```

👉 Matrix B:

```
5  6
7  8
```

### 2️⃣ Create Mapper & Reducer

```id="0a2k5z"
nano mapper.py
```

## 📜 Mapper Code (mapper.py)

```python id="0h39ml"
#!/usr/bin/env python
import sys

N = 2  # matrix size

for line in sys.stdin:
    line = line.strip()
    parts = line.split(",")

    if len(parts) != 4:
        continue

    matrix, i, j, val = parts
    i, j, val = int(i), int(j), float(val)

    if matrix == "A":
        for col in range(N):
            print "%d,%d\tA,%d,%f" % (i, col, j, val)

    elif matrix == "B":
        for row in range(N):
            print "%d,%d\tB,%d,%f" % (row, j, i, val)
```

### 2️⃣ Create Reducer

```id="0a2k4z"
nano reducer.py
```


---

## 📜 Reducer Code (reducer.py)

```python id="2t1x8c"
#!/usr/bin/env python
import sys

current_key = None
values = []

def process(key, values):
    A = {}
    B = {}

    for val in values:
        parts = val.split(',')
        if len(parts) != 3:
            continue

        matrix, k, v = parts
        k = int(k)
        v = float(v)

        if matrix == 'A':
            A[k] = v
        else:
            B[k] = v

    result = 0
    for k in A:
        if k in B:
            result += A[k] * B[k]

    print("%s\t%f" % (key, result))


for line in sys.stdin:
    key, val = line.strip().split('\t')

    if key == current_key:
        values.append(val)
    else:
        if current_key:
            process(current_key, values)

        current_key = key
        values = [val]

if current_key:
    process(current_key, values)
```

### 3️⃣ Make Files Executable

```id="8e6u5v"
chmod +x mapper.py reducer.py
```

### 4️⃣ Create HDFS Directory

```id="n1c7t0"
hdfs dfs -mkdir /input
```

### 5️⃣ Upload File

```id="sl4zdm"
hdfs dfs -put input.txt /input
```

### 6️⃣ Remove Old Output

```id="7og6sn"
hadoop fs -rm -r /output
```

### 7️⃣ Run Hadoop Job

```id="s2u7rz"
hadoop jar /usr/lib/hadoop-mapreduce/hadoop-streaming.jar \
-input /input/input.txt \
-output /output \
-mapper mapper.py \
-reducer reducer.py \
-file mapper.py \
-file reducer.py
```

### 8️⃣ View Output

```id="d9o1yq"
hdfs dfs -cat /output/part-00000
```

---

## ✅ Output

```id="f0kz2p"
0,0    19
0,1    22
1,0    43
1,1    50
```

---

## 📌 Result

The MapReduce program successfully computed the matrix multiplication.

👉 Result Matrix C:

```
19  22
43  50
```

---

## ❓ Viva Questions with Answers

### 1️⃣ What is matrix multiplication?

Matrix multiplication is the process of multiplying rows of one matrix with columns of another.

---

### 2️⃣ What is the role of Mapper in this program?

The Mapper distributes matrix elements and prepares key-value pairs for multiplication.

---

### 3️⃣ What is the role of Reducer?

The Reducer matches corresponding elements and computes the sum of products.

---

### 4️⃣ Why do we use key as (i,j)?

It represents the position of the result matrix element.

---

### 5️⃣ What is N in the mapper?

N is the number of columns in matrix A (or rows in matrix B).

---

### 6️⃣ What happens in shuffle and sort phase?

Hadoop groups all values with the same key before sending them to the reducer.

---

## 🚀 Conclusion

This experiment demonstrates how MapReduce can be used to perform matrix multiplication in a distributed manner using Hadoop Streaming and Python.

```id="d9o1yq"
#!/usr/bin/env python
import sys

N = 2  # number of columns in A / rows in B

for line in sys.stdin:
    line = line.strip()
    if not line:
        continue

    parts = line.split(",")
    if len(parts) != 4:
        continue

    matrix, i, j, val = parts
    i, j, val = int(i), int(j), float(val)

    if matrix == "A":
        for col in range(N):
            key = "%d,%d" % (i, col)
            value = "A,%d,%f" % (j, val)
            print("%s\t%s" % (key, value))

    else:  # matrix B
        for row in range(N):
            key = "%d,%d" % (row, j)
            value = "B,%d,%f" % (i, val)
            print("%s\t%s" % (key, value))
```
---
