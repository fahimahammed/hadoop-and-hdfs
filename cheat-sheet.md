## **Hadoop Cheat Sheet**

### **HDFS Commands**

- **List Files and Directories**
  ```bash
  hdfs dfs -ls /path/to/directory
  ```

- **List Files and Directories with Detailed Information**
  ```bash
  hdfs dfs -ls -R /path/to/directory
  ```

- **Make a Directory**
  ```bash
  hdfs dfs -mkdir /path/to/directory
  ```

- **Make a Directory with Parent Directories**
  ```bash
  hdfs dfs -mkdir -p /path/to/directory
  ```

- **Remove a Directory**
  ```bash
  hdfs dfs -rmdir /path/to/directory
  ```

- **Remove a Directory Recursively**
  ```bash
  hdfs dfs -rm -r /path/to/directory
  ```

- **Delete a File**
  ```bash
  hdfs dfs -rm /path/to/file
  ```

- **Delete a File Recursively**
  ```bash
  hdfs dfs -rm -r /path/to/file
  ```

- **Copy Files from Local to HDFS**
  ```bash
  hdfs dfs -copyFromLocal /local/path/to/file /path/to/hdfs/directory
  ```

- **Copy Files from HDFS to Local**
  ```bash
  hdfs dfs -copyToLocal /path/to/hdfs/file /local/path/to/directory
  ```

- **Move Files from Local to HDFS**
  ```bash
  hdfs dfs -moveFromLocal /local/path/to/file /path/to/hdfs/directory
  ```

- **Move Files from HDFS to Local**
  ```bash
  hdfs dfs -moveToLocal /path/to/hdfs/file /local/path/to/directory
  ```

- **Display File Content**
  ```bash
  hdfs dfs -cat /path/to/hdfs/file
  ```

- **Display File Content with Pagination**
  ```bash
  hdfs dfs -cat /path/to/hdfs/file | less
  ```

- **Show Disk Usage**
  ```bash
  hdfs dfs -du -h /path/to/directory
  ```

- **Check File Status**
  ```bash
  hdfs dfs -stat /path/to/hdfs/file
  ```

- **Count Files and Directories**
  ```bash
  hdfs dfs -count /path/to/directory
  ```

- **Get File Status**
  ```bash
  hdfs dfs -stat %F /path/to/hdfs/file
  ```

### **YARN Commands**

- **List Running Applications**
  ```bash
  yarn application -list
  ```

- **Get Application Information**
  ```bash
  yarn application -status <applicationId>
  ```

- **Kill a Running Application**
  ```bash
  yarn application -kill <applicationId>
  ```

- **List Node Managers**
  ```bash
  yarn node -list
  ```

- **Check Node Manager Status**
  ```bash
  yarn node -status <nodeId>
  ```

- **Get Resource Manager Scheduler Status**
  ```bash
  yarn rmadmin -getAllSchedulers
  ```

- **Check Resource Manager Status**
  ```bash
  yarn rmadmin -getServiceState
  ```

- **List all Node Managers**
  ```bash
  yarn node -list
  ```

### **MapReduce Commands**

- **Run a MapReduce Job**
  ```bash
  hadoop jar /path/to/your.jar com.example.YourClass /input/path /output/path
  ```

- **Run a MapReduce Job with Configuration File**
  ```bash
  hadoop jar /path/to/your.jar com.example.YourClass -Dmapreduce.job.reduces=2 /input/path /output/path
  ```

- **Check Job Status**
  ```bash
  hadoop job -status <jobId>
  ```

- **List All Jobs**
  ```bash
  hadoop job -list
  ```

- **Kill a Running Job**
  ```bash
  hadoop job -kill <jobId>
  ```

- **Get Job Report**
  ```bash
  hadoop job -report <jobId>
  ```

### **General Commands**

- **Check Hadoop Version**
  ```bash
  hadoop version
  ```

- **Format the Namenode (Warning: This will delete all data in HDFS!)**
  ```bash
  hdfs namenode -format
  ```

- **Start Hadoop Daemons**
  ```bash
  start-all.sh
  ```

- **Stop Hadoop Daemons**
  ```bash
  stop-all.sh
  ```

- **Start HDFS**
  ```bash
  start-dfs.sh
  ```

- **Stop HDFS**
  ```bash
  stop-dfs.sh
  ```

- **Start YARN**
  ```bash
  start-yarn.sh
  ```

- **Stop YARN**
  ```bash
  stop-yarn.sh
  ```

- **Check Namenode Health**
  ```bash
  hdfs dfsadmin -report
  ```

- **Check Datanode Health**
  ```bash
  hdfs dfsadmin -report -datanode
  ```

- **Display HDFS Web UI URL**
  ```bash
  echo "http://<namenode-host>:50070"
  ```

- **Display Resource Manager Web UI URL**
  ```bash
  echo "http://<resourcemanager-host>:8088"
  ```

### **Additional Commands**

- **Create an Archive of Files in HDFS**
  ```bash
  hdfs dfs -cp /source/path /destination/path
  ```

- **Retrieve File Status from the File System**
  ```bash
  hdfs dfs -stat "%b %h %n" /path/to/hdfs/file
  ```

- **List all the applications using a given application ID**
  ```bash
  yarn application -list -appStates ALL
  ```

- **Display logs for a given application**
  ```bash
  yarn logs -applicationId <applicationId>
  ```

---

This cheat sheet provides a quick reference to common Hadoop commands and operations, which should help streamline your work with Hadoop. Adjust paths, filenames, and application IDs based on your specific setup and requirements.