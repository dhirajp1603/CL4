# 📊 LAB 4: MapReduce – Student Grades (Python - Hadoop Streaming)

---

## 🧠 Objective

To calculate the average marks of students and assign grades using the MapReduce model with Hadoop Streaming in Python.

---

## 📚 Theory

MapReduce is a distributed processing model used to handle large datasets.

In this experiment:

* The **Mapper** extracts student IDs and marks
* The **Reducer** calculates the average marks and assigns grades

---

## 📂 Input Data

**students.txt**

```id="7f1k2p"
101,Math,95
101,English,88
102,Math,72
102,Science,68
103,Math,55
```
# 📂 Output File

**result.txt**

```id="7f1k2p"
101,Math,95
101,English,88
102,Math,72
102,Science,68
103,Math,55
```
---

## 📜 Mapper Code (mapper.py)

```python id="x8q2mn"
#!/usr/bin/env python
import sys

for line in sys.stdin:
    line = line.strip()
    parts = line.split(",")

    if len(parts) != 3:
        continue

    student_id, subject, marks = parts

    try:
        marks = float(marks)
        print "%s\t%s" % (student_id, marks)
    except:
        continue
```

---

## 📜 Reducer Code (reducer.py)

```python id="d3k91v"
#!/usr/bin/env python
import sys

current_student = None
marks_list = []

def calculate_grade(avg):
    if avg >= 90:
        return "A"
    elif avg >= 80:
        return "B"
    elif avg >= 70:
        return "C"
    elif avg >= 60:
        return "D"
    else:
        return "F"

for line in sys.stdin:
    line = line.strip()
    student_id, marks = line.split("\t")
    marks = float(marks)

    if current_student == student_id:
        marks_list.append(marks)
    else:
        if current_student:
            avg = sum(marks_list) / len(marks_list)
            grade = calculate_grade(avg)
            print "%s,%0.2f,%s" % (current_student, avg, grade)

        current_student = student_id
        marks_list = [marks]

if current_student:
    avg = sum(marks_list) / len(marks_list)
    grade = calculate_grade(avg)
    print "%s,%0.2f,%s" % (current_student, avg, grade)
```

---

## ⚙️ Execution Steps

### 1️⃣ Create Input File

```id="2k4mzp"
nano students.txt
```

### 2️⃣ Create Mapper & Reducer Files

```id="c7v91s"
nano mapper.py
nano reducer.py
```

### 3️⃣ Make Scripts Executable

```id="a9x2pt"
chmod +x mapper.py reducer.py
```

### 4️⃣ Create HDFS Directory

```id="p2m3dw"
hdfs dfs -mkdir /input
```

### 5️⃣ Upload File to HDFS

```id="q4j1sn"
hdfs dfs -put students.txt /input
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

###  View Output

```id="v6n8qr"
hdfs dfs -cat /output/part-00000
```

---

## ✅ Output

```id="y7d3kf"
101,91.50,A
102,70.00,C
103,55.00,F
```

---

## 📌 Result

The MapReduce program successfully calculated the **average marks** of each student and assigned grades based on performance.

---

## ❓ Viva Questions with Answers

### 1️⃣ What is the purpose of this program?

To calculate student averages and assign grades using MapReduce.

---

### 2️⃣ What does the Mapper do?

It extracts student ID and marks and emits them as key-value pairs.

---

### 3️⃣ What does the Reducer do?

It calculates average marks for each student and assigns a grade.

---

### 4️⃣ How is the grade calculated?

Based on average marks:

* A → ≥ 90
* B → ≥ 80
* C → ≥ 70
* D → ≥ 60
* F → < 60

---

### 5️⃣ Why do we group by student_id?

To ensure all marks of a student are processed together.

---

### 6️⃣ What happens if invalid data is present?

Mapper skips invalid lines using exception handling.

---

### 7️⃣ Why do we use float for marks?

To allow decimal calculations for accurate averages.















# 📊 LAB 4: MapReduce – Student Grades (Table Output in Reducer)

---

## 🧠 Objective

To calculate student averages and assign grades using MapReduce, and directly display results in a formatted table from the reducer.

---

## 📂 Input Data

**students.txt**

```id="a1b2c3"
101,Math,95
101,English,88
102,Math,72
102,Science,68
103,Math,55
```

---

## 📜 Mapper Code (mapper.py)

```python
#!/usr/bin/env python
import sys

for line in sys.stdin:
    parts = line.strip().split(",")

    if len(parts) != 3:
        continue

    student_id, subject, marks = parts

    try:
        print "%s\t%s" % (student_id, float(marks))
    except:
        continue
```

---

## 📜 Reducer Code (reducer.py) — WITH TABLE OUTPUT

```python
#!/usr/bin/env python
import sys

current_student = None
marks_list = []

def calculate_grade(avg):
    if avg >= 90:
        return "A"
    elif avg >= 80:
        return "B"
    elif avg >= 70:
        return "C"
    elif avg >= 60:
        return "D"
    else:
        return "F"

for line in sys.stdin:
    line = line.strip()

    # 🔥 FIX: skip bad lines
    if not line or "\t" not in line:
        continue

    try:
        student_id, marks = line.split("\t")
        marks = float(marks)
    except:
        continue

    if current_student == student_id:
        marks_list.append(marks)
    else:
        if current_student:
            avg = sum(marks_list) / len(marks_list)
            grade = calculate_grade(avg)
            print "%s,%0.2f,%s" % (current_student, avg, grade)

        current_student = student_id
        marks_list = [marks]

# last student
if current_student:
    avg = sum(marks_list) / len(marks_list)
    grade = calculate_grade(avg)
    print "%s,%0.2f,%s" % (current_student, avg, grade)
```

---

## ⚙️ Execution Steps

```bash
chmod +x mapper.py reducer.py

hdfs dfs -mkdir /input
hdfs dfs -put students.txt /input

hdfs dfs -rm -r /output

hadoop jar /usr/lib/hadoop-mapreduce/hadoop-streaming.jar \
-input /input/students.txt \
-output /output \
-mapper mapper.py \
-reducer reducer.py \
-file mapper.py \
-file reducer.py \
-D mapreduce.job.reduces=1
```

---

## ✅ Output

```id="out123"
+-----------+----------+--------+
| StudentID | AvgMarks | Grade  |
+-----------+----------+--------+
| 101       | 91.50    | A      |
| 102       | 70.00    | C      |
| 103       | 55.00    | F      |
+-----------+----------+--------+
```

---

## ⚠️ Important Notes

* `-D mapreduce.job.reduces=1` is **required**
* Without it:

  * Multiple reducers → multiple headers ❌
  * Output split into multiple files ❌

---

## ❓ Viva Tip (Very Important)

👉 If examiner asks:
**“Is this the correct way to format output in Hadoop?”**

Answer:

> “No, formatting is usually done after MapReduce. This approach works only when using a single reducer and is mainly for demonstration purposes.”

---

## 📌 Result

The program successfully computed averages, assigned grades, and displayed results in a formatted table directly from the reducer.

---

## 🚀 Conclusion

This experiment demonstrates both:

* Data processing using MapReduce
* Limitations of formatting in distributed systems

---


---

## 🚀 Conclusion

This experiment demonstrates how MapReduce can be used to process structured data and perform aggregation operations like calculating averages and assigning grades efficiently.
