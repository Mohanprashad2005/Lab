 **complete end-to-end documentation** for your environment (Ubuntu + Hadoop single node).
This will include: setup, running Hadoop daemons, preparing input, creating the Java program, compiling, packaging, running the job, and checking output.

---

# 🧾 **Complete Guide: Running a Hadoop WordCount Program on Ubuntu**

---

## 🧩 **1. Prerequisites**

Ensure the following are installed and configured:

| Component           | Required Version      | Example           |
| ------------------- | --------------------- | ----------------- |
| Java                | Java 11 (recommended) | `java -version`   |
| Hadoop              | 3.x or newer          | `hadoop version`  |
| HDFS + YARN running |                       | Confirm via `jps` |

Environment variables should be set (check):

```bash
echo $HADOOP_HOME
echo $PATH
```

Expected output:

```
/home/grace/hadoop
/home/grace/hive/bin:/home/grace/hadoop/bin:/home/grace/hadoop/sbin:...
```

---

## 🚀 **2. Start Hadoop Services**

Start both HDFS and YARN daemons.

```bash
start-dfs.sh
start-yarn.sh
```

Confirm that all services are running:

```bash
jps
```

Expected output (processes may have different PIDs):

```
NameNode
DataNode
SecondaryNameNode
ResourceManager
NodeManager
Jps
```

---

## 📁 **3. Prepare Input Files in HDFS**

Create a folder and upload your local input file to HDFS.

```bash
hdfs dfs -mkdir /input
hdfs dfs -put /home/grace/inputfile.txt /input
```

Verify:

```bash
hdfs dfs -ls /input
```

Expected output:

```
Found 1 items
-rw-r--r--   1 grace supergroup        6 2025-11-08 13:35 /input/inputfile.txt
```

---

## 💻 **4. Create the WordCount Java Program**

Create working directory:

```bash
mkdir -p ~/hadoop_programs/wordcount
cd ~/hadoop_programs/wordcount
```

Create and edit the file:

```bash
nano WordCount.java
```

Paste the code:

```java
import java.io.IOException;
import java.util.StringTokenizer;

import org.apache.hadoop.conf.Configuration;
import org.apache.hadoop.fs.Path;
import org.apache.hadoop.io.IntWritable;
import org.apache.hadoop.io.Text;
import org.apache.hadoop.mapreduce.Job;
import org.apache.hadoop.mapreduce.Mapper;
import org.apache.hadoop.mapreduce.Reducer;
import org.apache.hadoop.mapreduce.lib.input.FileInputFormat;
import org.apache.hadoop.mapreduce.lib.output.FileOutputFormat;

public class WordCount {

  public static class TokenizerMapper
       extends Mapper<Object, Text, Text, IntWritable>{

    private final static IntWritable one = new IntWritable(1);
    private Text word = new Text();

    public void map(Object key, Text value, Context context
                    ) throws IOException, InterruptedException {
      StringTokenizer itr = new StringTokenizer(value.toString());
      while (itr.hasMoreTokens()) {
        word.set(itr.nextToken());
        context.write(word, one);
      }
    }
  }

  public static class IntSumReducer
       extends Reducer<Text,IntWritable,Text,IntWritable> {
    private IntWritable result = new IntWritable();

    public void reduce(Text key, Iterable<IntWritable> values,
                       Context context
                       ) throws IOException, InterruptedException {
      int sum = 0;
      for (IntWritable val : values) {
        sum += val.get();
      }
      result.set(sum);
      context.write(key, result);
    }
  }

  public static void main(String[] args) throws Exception {
    Configuration conf = new Configuration();
    Job job = Job.getInstance(conf, "word count");
    job.setJarByClass(WordCount.class);
    job.setMapperClass(TokenizerMapper.class);
    job.setCombinerClass(IntSumReducer.class);
    job.setReducerClass(IntSumReducer.class);
    job.setOutputKeyClass(Text.class);
    job.setOutputValueClass(IntWritable.class);
    FileInputFormat.addInputPath(job, new Path(args[0]));
    FileOutputFormat.setOutputPath(job, new Path(args[1]));
    System.exit(job.waitForCompletion(true) ? 0 : 1);
  }
}
```

Save and exit (`Ctrl+O`, `Enter`, `Ctrl+X`).

---

## ⚙️ **5. Compile the Java Program**

Compile the source code using Hadoop’s libraries.

```bash
javac -source 11 -target 11 -classpath `hadoop classpath` -d . WordCount.java
```

Verify `.class` files:

```bash
ls
```

Expected:

```
WordCount.java
WordCount.class
WordCount$TokenizerMapper.class
WordCount$IntSumReducer.class
```

---

## 📦 **6. Create the JAR File**

Package the compiled classes into a JAR:

```bash
jar -cvf wordcount.jar WordCount*.class
```

Verify:

```bash
jar -tf wordcount.jar
```

Expected:

```
WordCount.class
WordCount$TokenizerMapper.class
WordCount$IntSumReducer.class
```

---

## 🧨 **7. Run the Hadoop Job**

Remove old output (if it exists):

```bash
hdfs dfs -rm -r /output
```

Run your MapReduce program:

```bash
hadoop jar ~/hadoop_programs/wordcount/wordcount.jar WordCount /input /output
```

Expected console messages:

* Job submission info
* Mapper/Reducer progress
* Job completion message:

  ```
  INFO mapreduce.Job: Job job_... completed successfully
  ```

---

## 📊 **8. View the Output**

Check that the output directory was created:

```bash
hdfs dfs -ls /output
```

View the results:

```bash
hdfs dfs -cat /output/part-r-00000
```

Example output:

```
Hadoop  1
Hello   3
World   2
```

---

## 🛑 **9. Stop Hadoop Services (optional)**

When done, you can stop all Hadoop daemons:

```bash
stop-yarn.sh
stop-dfs.sh
```

---

## 🧠 **10. Summary of All Commands**

```bash
# Start Hadoop
start-dfs.sh
start-yarn.sh

# Check services
jps

# Prepare input
hdfs dfs -mkdir /input
hdfs dfs -put /home/grace/inputfile.txt /input
hdfs dfs -ls /input

# Create program
mkdir -p ~/hadoop_programs/wordcount
cd ~/hadoop_programs/wordcount
nano WordCount.java

# Compile and build jar
javac -source 11 -target 11 -classpath `hadoop classpath` -d . WordCount.java
jar -cvf wordcount.jar WordCount*.class

# Run Hadoop job
hdfs dfs -rm -r /output
hadoop jar ~/hadoop_programs/wordcount/wordcount.jar WordCount /input /output

# Check output
hdfs dfs -ls /output
hdfs dfs -cat /output/part-r-00000

# Stop Hadoop (optional)
stop-yarn.sh
stop-dfs.sh
```

---

✅ **Result:**
You’ve built and executed a fully functional Hadoop MapReduce program in Java that counts word frequencies from text stored in HDFS.

---
