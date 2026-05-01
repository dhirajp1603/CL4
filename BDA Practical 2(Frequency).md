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

## ❓ Viva Questions

1. What is Hadoop Streaming?
2. What is the role of Mapper?
3. What is the role of Reducer?
4. Why is sorting required before Reducer?
5. What is HDFS?

---

## 👨‍💻 Conclusion

This experiment demonstrates how Hadoop Streaming can be used with Python to process large datasets using the MapReduce model efficiently.
