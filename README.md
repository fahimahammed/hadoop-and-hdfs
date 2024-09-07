Here's the complete documentation for working with Apache Hadoop 3.4.0 on Debian, including handling input files, running MapReduce jobs, and managing output directories.

---

# Apache Hadoop 3.4.0 Documentation on Debian

## Table of Contents
1. [Prerequisites](#prerequisites)
2. [Lab 23: Creating HDFS](#lab-23-creating-hdfs)
3. [Lab 24: Installing Hadoop Framework](#lab-24-installing-hadoop-framework)
4. [Lab 25: Query Processing in HDFS](#lab-25-query-processing-in-hdfs)
5. [Managing Files and Running Jobs](#managing-files-and-running-jobs)

---

## Prerequisites

1. **Java Installation**
   - Ensure Java 8 or higher is installed:
     ```bash
     java --version
     ```
   - Install Java if needed:
     ```bash
     sudo apt update
     sudo apt install openjdk-17-jdk -y
     ```

2. **SSH Configuration**
   - Install SSH:
     ```bash
     sudo apt install openssh-server openssh-client -y
     ```
   - Set up passwordless SSH:
     ```bash
     ssh-keygen -t rsa -P ""
     cat ~/.ssh/id_rsa.pub >> ~/.ssh/authorized_keys
     ```

---

## Lab 23: Creating HDFS

### Step 1: Download and Install Hadoop 3.4.0

1. **Download Hadoop**:
   ```bash
   wget https://downloads.apache.org/hadoop/common/hadoop-3.4.0/hadoop-3.4.0.tar.gz
   ```

2. **Extract Hadoop**:
   ```bash
   tar -xvzf hadoop-3.4.0.tar.gz
   sudo mv hadoop-3.4.0 /usr/local/hadoop
   ```

3. **Set Hadoop Environment Variables**:
   Edit `.bashrc`:
   ```bash
   nano ~/.bashrc
   ```
   Add:
   ```bash
   export HADOOP_HOME=/usr/local/hadoop
   export HADOOP_INSTALL=$HADOOP_HOME
   export HADOOP_MAPRED_HOME=$HADOOP_HOME
   export HADOOP_COMMON_HOME=$HADOOP_HOME
   export HADOOP_HDFS_HOME=$HADOOP_HOME
   export YARN_HOME=$HADOOP_HOME
   export PATH=$PATH:$HADOOP_HOME/bin:$HADOOP_HOME/sbin
   ```
   Apply changes:
   ```bash
   source ~/.bashrc
   ```

### Step 2: Configure Hadoop

1. **Set JAVA_HOME**:
   Find Java path:
   ```bash
   readlink -f $(which java)
   ```
   Update `hadoop-env.sh`:
   ```bash
   nano $HADOOP_HOME/etc/hadoop/hadoop-env.sh
   ```
   Add:
   ```bash
   export JAVA_HOME=/usr/lib/jvm/java-17-openjdk-amd64
   ```

2. **Configure HDFS**:
   - **Edit `core-site.xml`**:
     ```bash
     nano $HADOOP_HOME/etc/hadoop/core-site.xml
     ```
     Add:
     ```xml
     <configuration>
       <property>
         <name>fs.defaultFS</name>
         <value>hdfs://localhost:9000</value>
       </property>
     </configuration>
     ```

   - **Edit `hdfs-site.xml`**:
     ```bash
     nano $HADOOP_HOME/etc/hadoop/hdfs-site.xml
     ```
     Add:
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

   - **Create HDFS Directories**:
     ```bash
     sudo mkdir -p /usr/local/hadoop_data/hdfs/namenode
     sudo mkdir -p /usr/local/hadoop_data/hdfs/datanode
     sudo chown -R $USER:$USER /usr/local/hadoop_data
     ```

   - **Format the NameNode**:
     ```bash
     hdfs namenode -format
     ```

### Step 3: Start HDFS

1. **Start HDFS**:
   ```bash
   start-dfs.sh
   ```

2. **Verify Services**:
   Check that NameNode and DataNode are running:
   ```bash
   jps
   ```

3. **Access HDFS Web UI**:
   Navigate to:
   ```
   http://localhost:9870/
   ```

---

## Lab 24: Installing Hadoop Framework (YARN)

### Step 1: Configure YARN

1. **Edit `yarn-site.xml`**:
   ```bash
   nano $HADOOP_HOME/etc/hadoop/yarn-site.xml
   ```
   Add:
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

2. **Start YARN**:
   ```bash
   start-yarn.sh
   ```

3. **Verify YARN Services**:
   Check that ResourceManager and NodeManager are running:
   ```bash
   jps
   ```

---

## Lab 25: Query Processing in HDFS (Using MapReduce)

### Step 1: Write the MapReduce Program

1. **Create WordCount Java Program**:
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

### Step 2: Compile and Run the Program

1. **Compile the Program**:
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
   hdfs dfs -put input.txt /input/input.txt
   ```

4. **Run the MapReduce Job**:
   ```bash
   hadoop jar wordcount.jar WordCount /input/input.txt /output
   ```

5. **View the Output**:
   ```bash
   hdfs dfs -cat /output/part-r-00000
   ```

### Changing the Content of an Existing Input File in HDFS

1. **Remove the Existing Input File from HDFS**:
   ```bash
   hdfs dfs -rm /input/input.txt
   ```

2. **Create a New File with Updated Content Locally**:
   ```bash
   echo "New content for Hadoop processing" > new_input.txt
   ```

3. **Upload the New Input File to HDFS**:
   ```bash
   hdfs dfs -put new_input.txt /input/input.txt
   ```

4. **Run the Map

Reduce Job Again**:
   ```bash
   hadoop jar wordcount.jar WordCount /input/input.txt /output
   ```

5. **View the Output**:
   ```bash
   hdfs dfs -cat /output/part-r-00000
   ```

---

### Stopping Hadoop Services

1. **Stop HDFS**:
   ```bash
   stop-dfs.sh
   ```

2. **Stop YARN**:
   ```bash
   stop-yarn.sh
   ```

3. **Verify Services are Stopped**:
   ```bash
   jps
   ```

   No Hadoop-related processes should be listed.

---

This comprehensive guide should cover everything from setting up Hadoop to running MapReduce jobs and managing input/output files. If you need further assistance, feel free to ask!
