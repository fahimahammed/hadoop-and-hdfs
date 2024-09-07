# Apache Hadoop 3.4.0 Setup on Debian

---
### Lab 23: Creating HDFS

### Step 1: Installing Prerequisites

#### 1. **Install Java (OpenJDK 8 or higher)**:
Hadoop 3.4.0 requires Java to run. Verify the installation:
```bash
java -version
```
If Java is not installed, install it:
```bash
sudo apt update
sudo apt install openjdk-8-jdk -y
```

#### 2. **Install SSH (required for Hadoop)**:
Hadoop requires passwordless SSH access between nodes:
```bash
sudo apt install ssh
```
Generate an SSH key if you don’t already have one:
```bash
ssh-keygen -t rsa -P ""
cat ~/.ssh/id_rsa.pub >> ~/.ssh/authorized_keys
```

---

### Step 2: Download and Install Hadoop 3.4.0

1. **Download Apache Hadoop 3.4.0**:
   ```bash
   wget https://downloads.apache.org/hadoop/common/hadoop-3.4.0/hadoop-3.4.0.tar.gz
   ```

2. **Extract Hadoop**:
   ```bash
   tar -xvzf hadoop-3.4.0.tar.gz
   sudo mv hadoop-3.4.0 /usr/local/hadoop
   ```

3. **Set Hadoop Environment Variables**:
   Add Hadoop to the environment variables by editing the `.bashrc` file:
   ```bash
   nano ~/.bashrc
   ```

   Add the following lines at the end:
   ```bash
   export HADOOP_HOME=/usr/local/hadoop
   export HADOOP_INSTALL=$HADOOP_HOME
   export HADOOP_MAPRED_HOME=$HADOOP_HOME
   export HADOOP_COMMON_HOME=$HADOOP_HOME
   export HADOOP_HDFS_HOME=$HADOOP_HOME
   export YARN_HOME=$HADOOP_HOME
   export PATH=$PATH:$HADOOP_HOME/bin:$HADOOP_HOME/sbin
   ```

4. **Apply changes**:
   ```bash
   source ~/.bashrc
   ```

---

### Step 3: Configuring Hadoop for HDFS

#### 1. **Set JAVA_HOME in Hadoop**:
   ```bash
   nano $HADOOP_HOME/etc/hadoop/hadoop-env.sh
   ```
   Modify the `JAVA_HOME` variable to the path of Java:
   ```bash
   export JAVA_HOME=/usr/lib/jvm/java-8-openjdk-amd64
   ```

#### 2. **Edit `core-site.xml`**:
   Configure Hadoop to point to your local filesystem and HDFS:
   ```bash
   nano $HADOOP_HOME/etc/hadoop/core-site.xml
   ```
   Add the following properties:
   ```xml
   <configuration>
     <property>
       <name>fs.defaultFS</name>
       <value>hdfs://localhost:9000</value>
     </property>
   </configuration>
   ```

#### 3. **Edit `hdfs-site.xml`**:
   Configure HDFS by specifying NameNode and DataNode directories:
   ```bash
   nano $HADOOP_HOME/etc/hadoop/hdfs-site.xml
   ```
   Add the following configuration:
   ```xml
   <configuration>
     <property>
       <name>dfs.replication</name>
       <value>1</value>
     </property>
     <property>
       <name>dfs.namenode.name.dir</name>
       <value>file:///usr/local/hadoop_data/hdfs/namenode</value>
     </property>
     <property>
       <name>dfs.datanode.data.dir</name>
       <value>file:///usr/local/hadoop_data/hdfs/datanode</value>
     </property>
   </configuration>
   ```

#### 4. **Create HDFS Directories**:
   Create the necessary directories for HDFS data:
   ```bash
   sudo mkdir -p /usr/local/hadoop_data/hdfs/namenode
   sudo mkdir -p /usr/local/hadoop_data/hdfs/datanode
   sudo chown -R $USER:$USER /usr/local/hadoop_data
   ```

#### 5. **Format the NameNode**:
   Initialize HDFS by formatting the NameNode:
   ```bash
   hdfs namenode -format
   ```

---

### Step 4: Start HDFS (Hadoop File System)

1. **Start the Hadoop DFS**:
   ```bash
   start-dfs.sh
   ```

2. **Check Hadoop services**:
   Use `jps` to ensure services like NameNode and DataNode are running:
   ```bash
   jps
   ```
   Services you should see:
   - NameNode
   - DataNode
   - SecondaryNameNode

3. **Access HDFS Web UI**:
   Navigate to HDFS Web UI to verify HDFS is up and running:
   ```
   http://localhost:9870/
   ```

---

### Lab 24: Installing the Hadoop Framework (YARN)

#### 1. **Configure `yarn-site.xml`**:
   Set up YARN by editing its configuration file:
   ```bash
   nano $HADOOP_HOME/etc/hadoop/yarn-site.xml
   ```
   Add the following configuration:
   ```xml
   <configuration>
     <property>
       <name>yarn.resourcemanager.hostname</name>
       <value>localhost</value>
     </property>
     <property>
       <name>yarn.nodemanager.aux-services</name>
       <value>mapreduce_shuffle</value>
     </property>
   </configuration>
   ```

#### 2. **Start YARN**:
   Run the following command to start YARN:
   ```bash
   start-yarn.sh
   ```

#### 3. **Check Running Services**:
   Use `jps` to verify if ResourceManager and NodeManager are running:
   ```bash
   jps
   ```
   Services to expect:
   - ResourceManager
   - NodeManager
   - DataNode
   - NameNode

---

### Lab 25: Query Processing in HDFS (Using MapReduce)

HDFS stores the data, while MapReduce processes it. For this, we'll run a simple word count program.

#### Step 1: Write the MapReduce Program

Create a simple WordCount Java program:
```bash
nano WordCount.java
```

Insert the following code:
```java
import java.io.IOException;
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
    public static class TokenizerMapper extends Mapper<Object, Text, Text, IntWritable> {
        private final static IntWritable one = new IntWritable(1);
        private Text word = new Text();

        public void map(Object key, Text value, Context context) throws IOException, InterruptedException {
            String[] words = value.toString().split("\\s+");
            for (String w : words) {
                word.set(w);
                context.write(word, one);
            }
        }
    }

    public static class IntSumReducer extends Reducer<Text, IntWritable, Text, IntWritable> {
        private IntWritable result = new IntWritable();

        public void reduce(Text key, Iterable<IntWritable> values, Context context) throws IOException, InterruptedException {
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

#### Step 2: Compile and Run the Program

1. **Compile the WordCount program**:
   ```bash
   hadoop com.sun.tools.javac.Main WordCount.java
   jar cf wordcount.jar WordCount*.class
   ```

2. **Create an Input File**:
   ```bash
   echo "Hello Hadoop Hello HDFS" > input.txt
   ```

3. **Upload File to HDFS**:
   ```bash
   hdfs dfs -mkdir /input
   hdfs dfs -put input.txt /input/
   ```

4. **Run the MapReduce Job**:
   ```bash
   hadoop jar wordcount.jar WordCount /input /output
   ```

5. **View Output**:
   After the job is complete, check the output:
   ```bash
   hdfs dfs -cat /output/part-r-00000
   ``

`

   You should see word counts displayed in the terminal.

---

This updated guide for Hadoop 3.4.0 covers HDFS setup, installing Hadoop, and performing basic query processing using MapReduce.