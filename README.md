# 📊 LAB 2: MapReduce – Word Frequency using Hadoop Streaming (Python)

## 📚 Theory

MapReduce is a programming model used for processing large datasets in a distributed environment.

It consists of two main phases:

* **Mapper Phase:** Processes input data and emits key-value pairs
* **Reducer Phase:** Aggregates values for each key

In this experiment:

* The **Mapper** filters and emits occurrences of the word `"data"`
* The **Reducer** counts the total occurrences

---

## 📂 Input Data

**file1.txt**

```
data science is fun
big data is powerful
data is everywhere
```

---

## 📜 Mapper Code (map1.py)

```python
#!/usr/bin/env python
import sys

target = "data"

for line in sys.stdin:
    words = line.lower().split()
    for word in words:
        if word == target:
            print "%s\t1" % word
```

---

## 📜 Reducer Code (reduce1.py)

```python
#!/usr/bin/env python
import sys

current_word = None
total = 0

for line in sys.stdin:
    parts = line.strip().split("\t")

    if len(parts) != 2:
        continue

    word, count = parts
    count = int(count)

    if word == current_word:
        total += count
    else:
        if current_word:
            print "%s\t%d" % (current_word, total)
        current_word = word
        total = count

if current_word:
    print "%s\t%d" % (current_word, total)
```

---

## ⚙️ Execution Steps

### 1️⃣ Navigate to Home Directory

```
cd ~
```

### 2️⃣ Create Input File

```
nano file1.txt
```

### 3️⃣ Create Mapper & Reducer Files

```
nano map1.py
nano reduce1.py
```

### 4️⃣ Make Scripts Executable

```
chmod +x map1.py reduce1.py
```

### 5️⃣ Create HDFS Directory

```
hdfs dfs -mkdir /input1
```

### 6️⃣ Upload File to HDFS

```
hdfs dfs -put file1.txt /input1
```

### 7️⃣ Remove Old Output Directory

```
hdfs dfs -rm -r /output1
```

### 8️⃣ Run Hadoop Streaming Job

```
hadoop jar /usr/lib/hadoop-mapreduce/hadoop-streaming.jar \
-files map1.py,reduce1.py \
-mapper map1.py \
-reducer reduce1.py \
-input /input1 \
-output /output1
```

### 9️⃣ View Output

```
hdfs dfs -cat /output1/part-00000
```

---

## ✅ Output

```
data    3
```

---

## 📌 Result

The MapReduce program successfully counted the occurrences of the word **"data"**, which appears **3 times** in the input file.

---

## 🚀 Future Enhancements

* Count frequency of all words instead of one
* Remove punctuation
* Convert all text to lowercase uniformly
* Add stopword filtering

---

## ❓ Viva Questions with Answers

### 1️⃣ What is Hadoop Streaming?

Hadoop Streaming is a utility that allows users to write MapReduce programs using any programming language (such as Python, C++, or Shell) that can read input from standard input (`stdin`) and write output to standard output (`stdout`). It eliminates the need to write MapReduce programs in Java.

---

### 2️⃣ What is the role of Mapper?

The Mapper processes the input data line by line and converts it into key-value pairs.

👉 In this experiment:

* It reads each line
* Splits it into words
* Filters the word **"data"**
* Outputs:

```
data    1
```

---

### 3️⃣ What is the role of Reducer?

The Reducer processes the output from the Mapper and aggregates values for each key.

👉 In this experiment:

* It receives all `(data, 1)` pairs
* Adds them together
* Produces the final count

---

### 4️⃣ Why is sorting required before Reducer?

Sorting and grouping are automatically performed by Hadoop between the Mapper and Reducer phases.

👉 Purpose:

* Groups identical keys together
* Ensures Reducer receives all values of a key at once

Example:

```
data 1
data 1
data 1
```

---

### 5️⃣ What is HDFS?

HDFS (Hadoop Distributed File System) is a distributed storage system used by Hadoop.

👉 Features:

* Stores large data across multiple machines
* Fault-tolerant using replication
* Provides high-speed data access

---

### 6️⃣ What is a key-value pair?

It is the intermediate data format used in MapReduce:

```
key    value
```

Example:

```
data    1
```

---

### 7️⃣ What is a Combiner?

A Combiner is an optional mini-reducer that runs after the Mapper to reduce data before sending it to the Reducer, improving performance.

---

### 8️⃣ Difference between Mapper and Reducer?

| Mapper                    | Reducer               |
| ------------------------- | --------------------- |
| Processes input data      | Aggregates results    |
| Generates key-value pairs | Produces final output |
| Runs first                | Runs after sorting    |

---

### 9️⃣ Why do we use tab (`\t`) in output?

Tab is used to separate key and value so Hadoop can correctly interpret the data.

---

### 🔟 What happens if the output directory already exists?

Hadoop throws an error because it does not overwrite existing output directories.

👉 Solution:

```
hdfs dfs -rm -r /output1
```

---

## 👨‍💻 Conclusion

This experiment demonstrates how Hadoop Streaming can be used with Python to process large datasets using the MapReduce model efficiently.
