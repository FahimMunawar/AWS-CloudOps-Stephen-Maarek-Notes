# EC2 Placement Groups

This note covers EC2 Placement Groups, which allow you to control how EC2 instances are placed within AWS infrastructure.
Placement groups are essential for optimizing performance, availability, and reliability for specific workloads.

## Overview

**What are Placement Groups?**
- A strategy to control the placement of EC2 instances relative to one another
- You do not have direct interaction with AWS hardware, but can specify placement preferences
- Three strategies available: Cluster, Spread, and Partition

## 1. Cluster Placement Group

### Characteristics
- All EC2 instances are placed in the **same availability zone**
- Instances are grouped together in a **low-latency hardware setup**
- Instances share the same physical rack or nearby hardware

### Networking Performance
- Up to **10 Gbps bandwidth** between instances (with enhanced networking enabled)
- **Low latency** and **high throughput** network communication
- Ideal for tightly coupled applications requiring fast inter-instance communication

### Advantages
- Excellent performance for computational jobs
- Fast networking for distributed applications
- High throughput for data transfer between instances

### Disadvantages
- **High risk**: If the AZ fails, all instances fail simultaneously
- Single point of failure at the AZ level
- Not suitable for applications requiring high availability

### Use Cases
- Big data jobs that need to complete very quickly
- High-performance computing (HPC)
- Applications requiring extremely low latency and high throughput between instances

## 2. Spread Placement Group

### Characteristics
- EC2 instances are spread across **different physical hardware**
- Instances can span **multiple availability zones**
- Each instance is isolated on separate hardware

### Advantages
- **Minimizes failure risk** through hardware diversity
- **Reduced risk of simultaneous failure** across instances
- Can span multiple AZ for geographic redundancy

### Limitations
- **Maximum 7 instances per AZ per placement group**
- Hard limit on scalability
- Not suitable for applications requiring many instances

### Failure Isolation
- If hardware 1 fails, hardware 2 is not affected
- Each instance failure is isolated from others
- Risk of simultaneous failure between instances is minimized

### Use Cases
- Critical applications requiring maximum availability
- Applications where instance failures must be isolated from one another
- Small to medium-sized deployments requiring high availability

## 3. Partition Placement Group

### Characteristics
- Instances are spread across **partitions** (which represent hardware racks)
- **Up to 7 partitions per AZ**
- Partitions can **span multiple availability zones**
- Many EC2 instances can be placed within each partition

### Partition Structure
- Each partition represents a separate hardware rack
- Instances within a partition share the same rack
- Instances in different partitions are on different racks

### Advantages
- **Up to hundreds of EC2 instances** can be deployed (vs. 7 limit in Spread)
- **Partition-level isolation**: Each partition is isolated from failure
- Allows scaling large distributed applications
- Good balance between performance and fault tolerance

### Failure Isolation
- If Partition 1 fails, Partition 2 is not affected
- Partition-level failure isolation (different from Spread's instance-level isolation)
- More resilient than Cluster but more scalable than Spread

### Partition Awareness
- Applications can access partition information via the **EC2 metadata service**
- Allows partition-aware applications to distribute data and servers accordingly

### Use Cases
- **Big data applications** that are partition-aware:
  - HDFS (Hadoop Distributed File System)
  - HBase
  - Apache Cassandra
  - Apache Kafka
- Distributed databases and message queues
- Any application designed to be partition-aware

## Comparison Table

| Feature | Cluster | Spread | Partition |
|---------|---------|--------|-----------|
| **Placement** | Same AZ, low-latency | Multiple AZ, different hardware | Multiple AZ, across partitions |
| **Instances per AZ** | All in one AZ | Max 7 per AZ | 7 Partitions per AZ, Up to 100s per group |
| **Failure Risk** | High (AZ-level) | Low (instance-level) | Medium (partition-level) |
| **Network Throughput** | 10 Gbps (enhanced) | Standard | Standard |
| **Latency** | Ultra-low | Standard | Standard |
| **Use Cases** | HPC, Big Data jobs | critical apps, HA | Big data (partition-aware) |

## 4. Practical Console Steps

### Finding Placement Groups
- In the AWS EC2 console, scroll down to **Network and Security**
- Select **Placement Groups**

### Creating a Placement Group
1. Click **Create Placement Group**
2. Provide a name (e.g., `ec2-cluster-pg`)
3. Select strategy: **Cluster**, **Spread**, or **Partition**
4. For **Partition** strategy: specify the **number of partitions** (1-7)
   - Instances will be distributed round-robin across partitions
   - If you have 3 partitions and launch 10 instances:
     - Partition 1: 4 instances
     - Partition 2: 3 instances
     - Partition 3: 3 instances
5. (Optional) Add tags for identification and billing
6. Click **Create Group**

### Launching Instances in a Placement Group
1. Click **Launch Instance**
2. Configure the instance (AMI, instance type, key pair, etc.)
3. In **Advanced Details** (scroll down), find the **Placement Group** dropdown
4. Select the placement group you created
5. Launch the instance

### Important: Do Not Close the Launch Window
- After launching, do not close the browser window immediately
- AWS needs time to allocate and place the instance in the correct hardware
- Premature window closure may result in launch failures or placement issues

## 5. EC2 User Data (Bootstrapping)

### What is User Data?
- Scripts that run automatically when an EC2 instance boots
- Used for bootstrapping and initialization tasks

### Common Use Cases
- Install software (Python, MySQL, Docker, etc.)
- Configure the instance
- Start services automatically
- Download files or applications

### Example
```bash
#!/bin/bash
apt-get update
apt-get install -y python3
```

### Location in Console
- Found in **Advanced Details** section when launching an instance
- Add your script before launching

## 6. Viewing Placement Group Information

### Enabling Partition Column in Console
- In the EC2 instances list, click the **gear icon** (column settings)
- Enable **Placement Group** and **Partition Number** columns
- These columns will display which placement group and partition each instance belongs to

### What You'll See
- **Spread instances**: No partition number (they are independent)
- **Partition instances**: Shows partition number (1, 2, 3, etc.)
  - Demonstrates round-robin distribution across partitions
- **Cluster instances**: Placement group name only

## 7. Rack and Hardware Concepts

### Racks in AWS
- A **rack** is a group of servers with independent power and network infrastructure
- Rack-level failure: if one rack fails, other racks are unaffected
- **Partition placement group**: distributes instances across different racks
- This ensures **partition-level isolation** from failures

### Why This Matters
- Partition placement group is effective for distributed applications
- Each partition (rack) can fail independently without affecting others
- Supports hundreds of instances across multiple partitions

## 8. Common Restrictions and Issues

### Cluster Placement Group
- May not be available in all regions or with all instance types
- IAM permissions and AWS account restrictions can prevent cluster placement
- Some instance types may not support cluster placement group requirements
- If you encounter errors, work with AWS support or change instance type

### All Placement Groups
- Cannot be changed after instance launch (must terminate and relaunch)
- Moving an instance to a different placement group requires stopping and restarting
- Some AWS actions (like certain maintenance) may move instances out of placement groups

## 9. Key Takeaways for SysOps Associate

- **Know the three placement group types** and their characteristics
- **Cluster**: Low latency, high performance, high risk of AZ failure
- **Spread**: High availability, fault isolation, limited to 7 instances per AZ
- **Partition**: Large-scale distributed applications, partition-aware (up to 100s of instances)
- Understand when to use each:
  - Cluster for HPC and tightly-coupled computational jobs
  - Spread for critical applications requiring HA
  - Partition for big data applications like Hadoop, Cassandra, Kafka
- Know how partition-aware applications use EC2 metadata service to determine partition
- Understand AZ-level failure implications for each type
- Be familiar with the limitations (e.g., 7 instances per AZ for Spread, 7 partitions per AZ for Partition)
