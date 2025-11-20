# Distributed Hadoop Cluster for Road Safety Analytics

This repository documents the implementation of a three-node Hadoop cluster used to analyse 2018 UK road safety casualty data. The work was carried out as part of the **Distributed Database Engineering** lab and focuses on setting up the cluster, configuring Hadoop components, and running a Java MapReduce job to aggregate accident occurrences by casualty severity.

> 📄 Full technical report with screenshots, configuration snippets, and discussion is in `docs/DDE_Lab_Report_Road_Safety_Hadoop.pdf`.

---

## Project Overview

The goal of the project was to:

- Install and configure **Java**, **Hadoop (2.9.0)**, and **Apache Spark** across three virtual machines.
- Set up a **multi-node Hadoop cluster** with:
  - `machine01` – NameNode + ResourceManager
  - `machine02`, `machine03` – DataNodes + NodeManagers
- Store the **2018 road safety casualties dataset** in HDFS.
- Implement and run a **Java MapReduce** program to count accident occurrences grouped by **casualty severity**.
- Validate the cluster and job execution using the **Hadoop web UIs** (NameNode and ResourceManager).

Although Spark was installed as part of the environment, the main analytics in this lab were performed using **MapReduce**.

---

## Dataset

- **Source:** UK Department for Transport – 2018 road safety casualties data  
- **Format:** CSV (`dftRoadSafetyData_Casualties_2018.csv`)  
- **Size:** ~130K records  
- **Key field used:** `Casualty Severity`

The MapReduce job parses the CSV, skips the header row, extracts the casualty severity column, and aggregates counts per severity level.

c

## Cluster Architecture

The cluster consists of **three virtual machines**:

- `machine01`
  - NameNode (HDFS)
  - ResourceManager (YARN)
- `machine02`
  - DataNode
  - NodeManager
- `machine03`
  - DataNode
  - NodeManager

Key Hadoop components:

- **HDFS** – distributed storage with a replication factor of `3`
- **YARN** – resource management and job scheduling
- **MapReduce** – batch processing framework for the aggregation task

Spark was installed and configured but used mainly to align with modern industry practice; the core lab task centres around the MapReduce pipeline.

---

## Implementation Summary

The full step-by-step implementation (with commands, screenshots, and code snippets) is documented in the lab report. At a high level, the work included:

### 1. Java Installation & Configuration

- Installed **JDK 1.8.0_251**.
- Set `JAVA_HOME` and updated `PATH` in `~/.bashrc`.
- Verified with `java -version`.

### 2. Hadoop Installation & Configuration

- Extracted **Hadoop 2.9.0** into `/usr/local/` and created a symbolic link at `/usr/local/hadoop`.
- Set environment variables in `~/.bashrc`, including:
  - `HADOOP_HOME`
  - `HADOOP_CONF_DIR`
- Configured key XML files:
  - `core-site.xml` – `fs.defaultFS = hdfs://machine01:9000`
  - `hdfs-site.xml` – `dfs.replication = 3`, `dfs.namenode.name.dir = /abc/name`, `dfs.datanode.data.dir = /abc/dataX`
  - `mapred-site.xml` – `mapreduce.framework.name = yarn`
  - `yarn-site.xml` – `yarn.resourcemanager.hostname = machine01`

### 3. Spark Installation (Environment Only)

- Extracted Spark into `/usr/local/spark`.
- Set `SPARK_HOME` and updated `PATH`.
- Verified with `spark-shell --version`.

> Spark installation was completed, but the main analysis in this lab remained MapReduce-based.

### 4. Multi-Node Cluster Setup

- Cloned `machine01` to create `machine02` and `machine03`.
- Updated `/etc/hostname` and `/etc/hosts` on all nodes.
- Configured `hdfs-site.xml` on each node so that:
  - `machine02` used `/abc/data2`
  - `machine03` used `/abc/data3`
- Set up passwordless SSH:
  - Generated SSH keys and distributed them to `machine02` and `machine03`.
- Updated the `slaves` file on `machine01` with `machine02` and `machine03`.
- Formatted HDFS and started Hadoop services (`start-all.sh`).
- Verified running daemons using `jps` and the **Hadoop web UIs**.

### 5. MapReduce Job (Road Safety Severity Counts)

The MapReduce application (implemented in Java) consisted of:

- **Mapper** – reads each CSV line, skips the header, extracts casualty severity, and emits key–value pairs:
  - `<severity, 1>`
- **Reducer** – sums the counts per severity key.
- **Driver** – configures the job, sets input and output paths, and wires the Mapper and Reducer classes.

Deployment steps (as described in the report):

1. Exported the project as a JAR (e.g. `RoadSafetyCount.jar`) from Eclipse.
2. Uploaded the CSV file into HDFS:
   ```bash
   hadoop fs -mkdir -p /user/<username>/RoadSafety
   hadoop fs -put dftRoadSafetyData_Casualties_2018.csv /user/<username>/RoadSafety
3. Executing the MapReduce Job

After compiling the Java MapReduce program into a JAR file (e.g., `RoadSafetyCount.jar`), the job was executed on the Hadoop cluster.

```bash
hadoop jar /home/<username>/RoadSafetyCount.jar \
  /user/<username>/RoadSafety/dftRoadSafetyData_Casualties_2018.csv \
  /user/<username>/RoadSafety/Result1
```
The output file lists casualty severity levels and their corresponding counts.

4. Monitored the job via the ResourceManager UI (```http://machine01:8088```).
5. Verified the output using:
   ```hadoop fs -cat /user/<username>/RoadSafety/Result1/part-r-00000```
The output file lists casualty severity levels and their corresponding counts.

---

## Results & Observations

The Hadoop cluster successfully processed the 2018 UK road safety dataset using MapReduce.
The severity distribution observed was:

- Severity 3 (least severe) → highest count

- Severity 2 → second-highest

- Severity 1 (most severe) → lowest count

This trend is expected: severe and fatal accidents are statistically rare compared to minor incidents.

---

## Configuration Challenges

During setup, the following issues required careful attention:

- Establishing passwordless SSH between cluster nodes

- Correctly configuring Hadoop’s XML files to avoid hostname, directory, and path errors

Resolving these helped solidify understanding of distributed system configuration and Hadoop internals.

---

## Future Improvements (as identified in the report)

Several enhancements could evolve this project into a larger production-grade environment:

- Automation: Use Ansible or Puppet to automate multi-node cluster setup

- Performance Tuning: Adjust block sizes, replication factors, and YARN container memory for larger datasets

- Monitoring: Integrate monitoring tools such as Ambari, Grafana, or Cloudera Manager

- Spark-Based Analytics: Re-implement the workload using Spark for faster, in-memory processing

- Containerisation / Cloud Deployment: Recreate the cluster using Docker or deploy on AWS/Azure for scalability

These are suggested improvements, not part of the original lab implementation.

---

## Repository Contents

- README.md — High-level overview of the project and cluster setup

- docs/DDE_Lab_Report_Road_Safety_Hadoop.pdf — Full lab report including:
  - Step-by-step implementation
  - Screenshots of Hadoop UIs and commands
  - MapReduce code (Mapper, Reducer, Driver)
  - Configuration files and analysis
  - Discussion, critique, and references

