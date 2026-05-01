LAB 2: MapReduce – Word Frequency (Python - Hadoop Streaming)
Commands
cd ~

nano file1.txt
data science is fun
big data is powerful
data is everywhere

nano map1.py
nano reduce1.py

chmod +x map1.py reduce1.py

hdfs dfs -mkdir /input1
hdfs dfs -put file1.txt /input1

hdfs dfs -rm -r /output1

hadoop jar /usr/lib/hadoop-mapreduce/hadoop-streaming.jar -files map1.py,reduce1.py -mapper map1.py -reducer reduce1.py -input /input1 -output /output1

hdfs dfs -cat /output1/part-00000

Mapper Code
#!/usr/bin/env python
import sys

target = "data"

for line in sys.stdin:
    words = line.lower().split()
    for word in words:
        if word == target:
            print "%s\t1" % word

Reducer Code
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

LAB 3: Matrix Multiplication using MapReduce
Input File
nano input.txt

A,0,0,1
A,0,1,2
A,1,0,3
A,1,1,4

B,0,0,5
B,0,1,6
B,1,0,7
B,1,1,8

Mapper Code
#!/usr/bin/env python
import sys

N = 2

for line in sys.stdin:
    parts = line.strip().split(',')
    matrix, i, j, val = parts[0], int(parts[1]), int(parts[2]), int(parts[3])

    if matrix == 'A':
        for k in range(N):
            print("%d,%d\tA,%d,%d" % (i, k, j, val))
    else:
        for k in range(N):
            print("%d,%d\tB,%d,%d" % (k, j, i, val))

Reducer Code
#!/usr/bin/env python
import sys

current_key = None
values = []

def process(key, values):
    A = {}
    B = {}

    for val in values:
        parts = val.split(',')
        if parts[0] == 'A':
            A[int(parts[1])] = int(parts[2])
        else:
            B[int(parts[1])] = int(parts[2])

    result = 0
    for k in A:
        if k in B:
            result += A[k] * B[k]

    print("%s\t%d" % (key, result))

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

Commands
chmod +x mapper.py reducer.py

hdfs dfs -mkdir /input
hdfs dfs -put input.txt /input

hdfs dfs -rm -r /output

hadoop jar /usr/lib/hadoop-mapreduce/hadoop-streaming.jar -input /input/input.txt -output /output -mapper mapper.py -reducer reducer.py

hdfs dfs -cat /output/part-00000

LAB 4: MapReduce – Student Grades
Input File
nano students.txt

101,Math,95
101,English,88
102,Math,72
102,Science,68
103,Math,55

Mapper Code
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

Reducer Code
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

Commands
chmod +x mapper.py reducer.py

hdfs dfs -mkdir /input
hdfs dfs -put students.txt /input

hadoop jar /usr/lib/hadoop-mapreduce/hadoop-streaming.jar -input /input/students.txt -output /output -mapper "python mapper.py" -reducer "python reducer.py"

hdfs dfs -cat /output/part-00000

