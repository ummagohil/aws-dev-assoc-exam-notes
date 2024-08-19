# EC2 Instance Storage

## EBS

- An EBS (Elastic Block Store) Volume is a network drive you can attach to your instances while they run
- It allows your instances to persist data, even after their termination
- They can only be mounted to one instance at a time (at the CCP level)
- They are bound to a specific availability zone
- Analogy: think of them as a "network USB stick"
- Free tier: 30 GB of free EBS storage of type General Purpose (SSD) or Magnetic per month

### EBS Volume

- It's a network drive (not a physical drive)
  - It uses the network to communicate the instance, which means there might be a bit of latency
  - It can be detached from an EC2 instance and attached to another one quickly
- It's locked to an availability zone (AZ)
  - An EBS volume in us-east-1a cannot be attached to us-east-1b
  - To move a volume across, you first need to snapshot it
- Have a provisioned capacity (size in GBs and IOPS)
  - You get billed for all the provisioned capacity
  - You can increase the capacity of the drive over time

### EBS - Delete on Termination Attribute

- Controls the EBS behaviour when an EC2 instance terminates
  - By default, the root EBS volume is deleted (attribute enabled)
  - By default, any other attached EBS volume i snot deleted (attribute disabled)
- This can be controlled by the AWS console/AWS CLI
- Use case: preserve root volume when instance is terminated

### EBS Snapshots

- Make a backup (snapshot) of your EBS volume at a point in time
- Not necessary to detach volume to do snapshot, but recommended
- Can copy snapshots across AZ or region
- EBS Snapshot Archive
  - Move a Snapshot to an "archive tier" that is 75% cheaper
  - Takes within 24 to 72 hours for restoring the archive
- Recycle Bin for EBS Snapshots
  - Setup rules to retain deleted snapshots so you can recover them after an accidental deletion
  - Specify retention (for 1 day to 1 year)
- Fast Snapshot Restore (FSR)
  - Force full initialisation of snapshot to have no latency on the first use (£££)

## AMI

- AMI = Amazon Machine Image
- AMI are a customisation of an EC2 instance
  - You add your own software, configuration, operating system, monitoring etc
  - Faster boot/configuration time because all your software is pre-packaged
- AMI are built for a specific region (and can be copied across regions)
- You can launch EC2 instances from:
  - A Public AMI (AWS provided)
  - Your own AMI (you make and maintain them yourself)
  - An AWS Marketplace AMI (an AMI someone else made and potentially sells)
- AMI Process (from an EC2 instance)
  - Start an EC2 instance and customise it
  - Stop the instance (For data integrity)
  - Build an AMI - this will also create EBS snapshots
  - Launch instances from other AMIs

## EC2 Instance Store

-EBS volumes are network drives with good but "limited" performance

- If you need a high-performance hardware disk, use EC2 Instance Store
- Better I/O performance
- EC2 Instance Store lose their storage if they're stopped (ephemeral)
- Good for buffer/cache/scratch data/temporary content
- Risk of data loss if hardware fails
- Backups and replication are your responsibility

## EBS Volume Types

- EBS volumes come in 6 types
  - gp2/gp3 (SSD): general purpose SSD volumes that balances price and performance for a wide variety of workloads
  - io1/io2 Block Express (SSD): Highest-performance SSD volume for mission-critical low-latency or high-throughput workloads
  - st1 (HDD): low cost HDD volume designed for frequently accessed, throughput-intensive workloads
  - sc1 (HDD): lowest cost HDD volume designed for less frequently accessed workloads
- EBS volumes are characterised in size | throughput | IOPS (I/O Ops Per Sec)
- When in doubt always consult the AWS documentation
- Only gp2/gp3 and io I/ oi 2 Block Express can be used as boot volumes
- Use cases:
  - General Purpose SSD
    - Cost effective storage, low-latency
    - System boot volumes, virtual desktops, development and test environments
    - 1 GiB = 16 TiB
    - gp3:
      - Baseline of 3,000 IOPS and throughput of 125 MiB/s
      - Can increase IOPS up to 16,000 and throughput up to 1000 MiB/s independently
    - gp2:
      - Small gp2 volumes can burst IOPS to 3,000
      - Size of the volume and IOPS are linked, max IOPS is 16,000
      - 3 IOPS per GB means 5,334 GB we are at the max IOPS
  - Provisioned IOPS (PIOPS) SSD:
    - Critical business applications with sustained IOPS performance
    - Or applications that need more than 16,000 IOPS
    - Great for databases workloads (sensitive to storage perf and consistency)
    - io 1 (4 GiB - 16 TiB):
      - Max PIOPS: 64,000 for Nitro EC2 instances & 32,000 for other
      - Can increase PIOPS independently from storage size
  - io2 Block Express (4 GiB - 64 TiB):
    - Sub-millisecond latency
    - Max PIOPS: 256,000 with an IOPS: GIB ration of 1,000:1
  - Supports EBS Multi-attach
  - Hard Disk Drives (HDD):
    - Cannot be a boot volume
    - 125 GiB to 16 TiB
    - Throughput optimised HDD (st 1):
      - Big data, data warehouses, log processing
      - Max throughput 500 MiB/s - max IOPS 500
    - Cold HDD (sc 1):
      - For data that is infrequently accessed
      - Scenarios where lowest cost is important
      - Max throughout 250 MiB/s - max IOPS 250

## EBS Multi-Attach

- Attach the same EBS volume to multiple EC2 instances in the same AZ
- Each instance has full read & write permissions to the high-performance volume
- Use cases:
  - Achieve higher application availability in clustered Linux applications (eg. Teradata)
  - Applications must manage concurrent write operations
- Up to 16 EC2 instances at a time
- Must use a file system that's cluster-aware (not XFS, EXT4, etc)

## Amazon EFS - Elastic File System

- Use cases: content management, web serving, data sharing, Wordpress
- Uses NFSv4.1 protocol
- Uses security group to control access to EFS
- Compatible with Linux based AMI (not Windows)
- Encryption at rest using KMS)
- POSIX file system (~Linux) that has a standard file API
- File system scales automatically, pay-per-user, no capacity planning
- Performance & Storage Classes:
  - EFS Scale
    - 1000s of concurrent NFS clients, 10 GB+/s throughput
    - Grow to petabyte-scale network file system, automatically
  - Performance Mode (set at EFS creation time)
    - General Purpose (default) - latency-sensitive use cases (web server, CMS etc.)
    - Max I/O - higher latency, throughput, highly parallel (big data, media processing)
  - Throughput mode
    - Bursting - 1 TB = 50 MiB/s + burst of up to 100MiB/s
    - Provisioned - set your throughput regardless of storage size, eg. 1 GiB/s for 1 TB storage
    - Elastic - automatically scales throughput up or down based on your workloads
      - Up to 3GiB/s for reads and 1GiB/s for writes
      - Used for unpredictable workloads
  - Storage classes
    - Storage Tiers (lifecycle management feature - move files are N days)
      - Standard: for frequently accessed files
      - Infrequent access (EGS-1A): cost to retrieve files, lower price to store
      - Archive: rarely accessed data (few times each year), 50% cheaper
      - Implement lifecycle policies to move files between storage tiers
    - Availability and durability
      - Standard: multi-az, great for prod
      - One zone: one az, great for dev, backup enabled by default, compatible with IA (EFS One Zone-1A)
    - Over 90% in cost savings

## EFS vs EBS

### EBS Volumes...

- One instance (except multi-attach io 1/ io2)
- Are locked at the availability zone (AZ) level
- gp2: IO increases if the disk size increases
- gp3 & io 1: can increase IO independently
- To migrate an EBS volume across AZ:
  - Take a snapshot
  - Restore the snapshot to another AZ
  - EBS backups use IO and you shouldn't run them while your application is handling a lot of traffic
- Root EBS Volumes of instances get terminated by default if the EC2 instance gets terminated (you can disable it)
- Mounting 100s of instances across AZ
- EFS share website files (WordPress)
- Only for Linux Instances (POSIX)
- EFS has a higher price point than EBS
- Can leverage Storage Tiers for cost savings
- Reminder: EFS vs EBS vs Instance Store
