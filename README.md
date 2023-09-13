
This lab is designed to go through some Ozone fundamentals such as volume bucket and key creations. 

We will also explore some Cloudera security elements and how Ozone integrates broadly with the Cloudera ecosystem and beyond with the s3 gateway interaction.

# Lab 1 Security

Summary
- Enable Ranger from the Cloudera Manager UI
- Test Ranger privileges
- Review Ozone Security Settings from the command line

## 1.1 Enable and Configure Ranger
In Cloudera Manager, go to the Ozone service.
![[ClouderaManager-Ozoneservice.png]](./images/ClouderaManager-Ozoneservice.png)

Select the Ozone Configuration tab and onfirm that RANGER Service is enabled.
![OzoneConfiguration-Rangerenabled.png](./images/OzoneConfiguration-Rangerenabled.png)
This integrates Ozone with Ranger security policies.

Access the Ranger Service and access the Ranger UI
![RangerService-UIurl.png](./images/RangerService-UIurl.png)

Select cm_ozone under the Ozone service.
![cm_ozoneinRanger.png](./images/cm_ozoneinRanger.png)

In the top policy listed as **all - volume, bucket and key**, click the pencil icon on the right that shows Edit when you hover over it.

![EditpolicyforOzone.png](./images/EditpolicyforOzone.png)


Structure of the Ranger permissions for Ozone

Ozone Data Layout
![OzoneDataLayout.png](./images/OzoneDataLayout.png)

Ranger - Allow and deny conditions
![Ranger-AllowandDenyconditions.png](./images/Ranger-AllowandDenyconditions.png)

![Ozonepermissionpage.png](./images/Ozonepermissionpage.png)


**Test the following with users Bob and Alice**
Using the same Policy, set up to manage access over all Volume, buckets and keys:
Volume = \*
Bucket = \*
Key = \*

| Users  | Alice  | Bob  |
|--------|------|--------|
| Access |  All  |  Read/List file |
|  Deny  | None |  Create Volume |

Add users Admin and Alice to the existing "All" condition.

Bob's accesses:
- Click on the + button under the "Allow Conditions"![AddAllowCondition.png](./images/AddAllowCondition.png)
- Under “Allow” condition add “read/list” privileges for the user Bob
![AllowRangerpolicy.png](./images/AllowRangerpolicy.png)

- Under "Deny" condition, add "create" privileges for user Bob

![DenyRangerPolicy.png](./images/DenyRangerPolicy.png)

Add your username to the first “allow” condition (e.g. admin) to provide all privileges to your user.
![AddusertoOzonepermissions.png](./images/AddusertoOzonepermissions.png)

Scroll down to the bottom of the page and press Save.

## 1.2 Test Ranger privileges

Authenticate the user Bob with the Authentication Service of the KDC configured in /etc/krb5. conf
```console
kinit bob
``` 
Password: Supersecret1

Run the Ozone command to create a Volume
```console
ozone sh volume create /testperms
``` 

As Bob, user with restricted access, the expected response is:

`23/09/12 11:54:42 INFO rpc.RpcClient: Creating Volume: testperms, with bob as owner and space quota set to -1 bytes, counts quota set to -1
`PERMISSION_DENIED User bob@WORKSHOP.COM doesn't have CREATE permission to access volume Volume:testperms`

Switch to user Alice to create a volume and a file:
```console
kinit Alice
```
Password:Supersecret1

Try the same command as Alice:
```console
ozone sh volume create /testperms
```

As Alice, user with extended access, the expected response is:
`23/09/12 12:05:18 INFO rpc.RpcClient: Creating Volume: testperms1, with alice as owner and space quota set to -1 bytes, counts quota set to -1`

Create a test file and a bucket and save the file in the bucket using parameters:
Replication = ONE
Replication type = RATIS

```console
echo "Test file" > testfile
ozone sh bucket create /testperms/bucket1
ozone sh key put --replication=ONE --replication-type=RATIS o3://ozone/testperms/bucket1/alice_key1 testfile
```


## 1.3 Reviewing Ozone Security Settings

List the Kerberos principal and Kerberos tickets held in your credentials cache with klist. 
If necessary, obtain a Kerberos ticket-granting ticket using kinit.
Run the following ozone getconf commands to check some Ozone Manager properties:


```console
ozone getconf -confKey ozone.om.kerberos.principal
ozone getconf -confKey ozone.om.http.auth.kerberos.principal
ozone getconf -confKey ozone.om.http.auth.kerberos.keytab
```

# Lab 2 Ozone protocol operations

### Ozone Protocols:

Ozone has multiple protocols to work with for a variety of operations. 
There is no ONE PROTOCOL THAT RULES THEM ALL yet.

![Ozone protocoles.png](./images/Ozoneprotocoles.png)

Ozone is a multi-protocol storage system with support for the following interfaces:
- **ofs**: Hadoop-compatible file system allows any application that expects an HDFS like interface to work against Ozone with no changes. Frameworks like Apache Spark, YARN, and Hive work against Ozone without the need for any change. 
- **s3**: Amazon’s Simple Storage Service (S3) protocol. You can use S3 clients and S3 SDK-based applications without any modifications to Ozone.  => try to avoid this protocol since all passes through the s3ateway 
- **o3fs**: A bucket-rooted Hadoop Compatible file system interface. 
- **o3**: An object store interface that can be used from the Ozone shell.

### Ozone CLIs
Ozone CLI is used to access Ozone. 
- ozone **fs** - Runs Hadoop filesystem compatible commands on FSO(File System Optimized) and LEGACY buckets. Compatible with ofs and o3fs interfaces. Supports trash implementation.
- ozone **sh** - Ozone command shell interface to access Ozone as a key-value store. Command format is: ozone sh object action url. Object can be volume/bucket/key. Compatible with o3 interface.

#### Ozone fs
Summary operations
- interact with HDFS
- interact with ozone
- create a volume
- create a bucket
- push data to a bucket
- delete data from a bucket


The Ozone client can access Ozone as a file system and as a key-value store.
When Ozone is installed with the HDFS dependency, the Ozone client library support is built into the HDFS client commands, which will therefore be available for use with Ozone.
**hdfs dfs** can also be used (if ozone is not the default fs, a URI path is needed.)

Run the following command to list the files stored in HDFS.
```console
ozone fs -ls /
```

Expected output is the list of files stored in HDFS:
`drwxr-xr-x   - hbase hbase               0 2023-09-09 13:12 /hbase`
`drwxr-xr-x   - hdfs  supergroup          0 2023-09-09 13:08 /ranger`
`drwxrwxr-x   - solr  solr                0 2023-09-09 13:09 /solr-infra`
`drwxrwxrwt   - hdfs  supergroup          0 2023-09-09 13:19 /tmp`
`drwxr-xr-x   - hdfs  supergroup          0 2023-09-09 13:17 /user`
`drwxr-xr-x   - hdfs  supergroup          0 2023-09-09 13:10 /warehouse`
`drwxr-xr-x   - hdfs  supergroup          0 2023-09-09 13:10 /yarn`

 List the current ozone file system for Ozone sid ozone:
```console
ozone fs -ls ofs://ozone/
```
Volumes are at the highest level of the Ozone file system and are used to manage buckets that store keys.
Quotas and user permissions can be applied to volumes for high-level file system management.

Expected output for listing Ozone items at parent level:
`drwxrwxrwx   - om                       0 2023-09-11 18:29 ofs://ozone/s3v`


Create a volume called vol1 and list the ozone file system to see volumes
```console
ozone fs -mkdir ofs://ozone/vol1
ozone fs -ls ofs://ozone/
```

Expected output after volume creation and list command:
`drwxrwxrwx   - om                       0 2023-09-11 18:29 ofs://ozone/s3v`
`drwxrwxrwx   - admin admins             0 2023-09-12 14:34 ofs://ozone/vol1`

Create a bucket in vol1 called bucket1 and list all items under volume 1. Buckets are used to store files.
```console
ozone fs -mkdir ofs://ozone/vol1/bucket1
ozone fs -ls ofs://ozone/vol1
```
Expected output
`23/09/12 16:36:37 INFO rpc.RpcClient: Creating Bucket: vol1/bucket1, with the Bucket Layout FILE_SYSTEM_OPTIMIZED,
admin as owner, Versioning false, Storage Type set to DISK and Encryption set to false`
`drwxrwxrwx   - admin admins          0 2023-09-12 16:36 ofs://ozone/vol1/bucket1`

OFS mimics a traditional file system, the first two levels volume and bucket look like directories.
However, you cannot use the top level volume to store keys (files). When you add a key (file), it stores the contents of the file uploaded to Ozone under that key name. 
A key is a hybrid file name. It can be a file name stored at the root of the bucket or it can be a directory path from the bucket with a filename. 
Keys can be used to mimic a traditional file system.  It is important to note that volumes and buckets have naming restrictions and certain characters and cases are not allowed. 
Keys do not have this same limit. It is also important to note that you must have /volume/bucket for OFS.  
Files must have 2 directories at a minimum (/tmp is the only exception to be a hadoop compatible filesystem.) 
Some other notes is that EC and Encryption is at the bucket level.  Pathing from HDFS to Ozone may change due to these restrictions!!!!

Upload a file to bucket1:
```console
echo "Test file" > testfile
ozone fs -put testfile ofs://ozone/vol1/bucket1
ozone fs -ls ofs://ozone/vol1/bucket1
```
Expected output for listing content of bucket1:
-rw-rw-rw-   1 admin admin         10 2023-09-12 16:41 ofs://ozone/vol1/bucket1/testfile
![Ozone-Anatomyofawrite](./images/Ozone-Anatomyofawrite.png)

View content of the file:
```console
ozone fs -cat ofs://ozone/vol1/bucket1/testfile
```

##### Deletion
When you delete a file in Ozone using ozone fs, the file is not immediately removed from Ozone. Instead files are moved to a hidden .Trash dir (prefix dot Trash) that is user accessible under /user/<username>/.Trash/Current deleted directory. The full directory path of each user's deleted files will appear under this .Trash dir.

To bypass the trash to save disk space from keeping around files in the .Trash folder, set the -skipTrash flag to immediately delete the files bypassing the trash when you delete files.

```console
ozone fs -rm -r -skipTrash ofs://ozone/vol1/bucket1/testfile
```
Expected output
`Deleted ofs://ozone/vol1/bucket1/testfile`


#### Ozone sh
summary operations:
- create a volume
- create a bucket
- delete volume, buckets
- list operations
- get information from volume, bucket, key
- quota operations
- symlinks 
Other operations will be done in further section such as EC, replications, bucket layout type

Detailed operations:
Create a volume /vol2
```console
ozone sh volume create o3://ozone/vol2  ### or ozone sh volume create /vol2
```
Expected output
`ozone sh volume info /vol2
{
  "metadata" : { },
  "name" : "vol2",
  "admin" : "centos",
  "owner" : "centos",
  "quotaInBytes" : -1,
  "quotaInNamespace" : -1,
  "usedNamespace" : 0,
  "creationTime" : "2023-04-18T03:45:41.930Z",
  "modificationTime" : "2023-04-18T03:45:41.930Z",
  "acls" : [ {
    "type" : "USER",
    "name" : "centos",
    "aclScope" : "ACCESS",
    "aclList" : [ "ALL" ]
  }`

Create a bucket bucket1 under /vol2
```console
ozone sh bucket create /vol2/bucket1
```
Expected output
`ozone sh bucket info /vol2/bucket1
{
  "metadata" : { },
  "volumeName" : "vol2",
  "name" : "bucket1",
  "storageType" : "DISK",
  "versioning" : false,
  "usedBytes" : 0,
  "usedNamespace" : 0,
  "creationTime" : "2023-04-18T03:46:37.236Z",
  "modificationTime" : "2023-04-18T03:46:37.236Z",
  "quotaInBytes" : -1,
  "quotaInNamespace" : -1,
  "bucketLayout" : "LEGACY",
  "owner" : "centos",
  "link" : false
}`


Delete volume, buckets and create another volume and bucket associated to this exercise

```console
ozone sh volume delete o3://ozone/vol1
```
Expected output is an error message as the volume contains a bucket
```console
ozone sh bucket delete o3://ozone/vol1/bucket3
```
List operations
```cpnsole
ozone sh volume list --user=admin
ozone sh volume list --all o3://ozone 
# a variant which provides all the volumes for a dedicated user
ozone sh volume list --all o3://ozone | grep -A3 'metadata' | grep 'name\|owner\|admin'
ozone sh bucket list o3://ozone/vol1/
```

get information from volume, bucket, key
```console
ozone sh volume info o3://ozone/vol1
ozone sh bucket info o3://ozone/vol1/bucket1
```

Quota operations
```console
# set a quota 
## namespace-quota mean max number of buckets or keys
ozone sh volume setquota --namespace-quota=2 --space-quota 100MB o3://ozone/vol1
ozone sh bucket setquota --namespace-quota=10 --space-quota 100MB o3://ozone/vol1/bucket1 
#remove a quota
ozone sh volume clrquota --namespace-quota o3://ozone/vol1
ozone sh bucket clrquota --space-quota o3://ozone/vol1/bucket1
```

Symlinks
Symlinks are relevant when s3 operation required. you do not create a bucket within the volume srv but you symlink a bucket in it
```console
ozone sh bucket link o3://ozone/my-volume1/my-bucket1 o3://ozone/vol1/bucket3
```

Ozone bucket Erasure coding  ⇒ **won't fully work on a 1 node cluster**
Ozone supports RATIS and Erasure Coding Replication types. 
Default replication type is RATIS and the replication factor is 3. Copies of container replicas are maintained across the cluster. RATIS 3 replication has 200% storage overhead.
For cold and warm data with low I/O requirement EC storage is available. 50% replication overhead.

Create Erasure Coded(EC) buckets/keys
```console
ozone sh bucket create /vol1/ec5-bucket1 -t EC -r rs-3-2-1024k
ozone sh bucket info  /vol1/ec5-bucket1
```
Expected output
`{
  "metadata" : { },
  "volumeName" : "vol1",
  "name" : "ec5-bucket1",
  "storageType" : "DISK",
  "versioning" : false,
  "usedBytes" : 0,
  "usedNamespace" : 0,
  "creationTime" : "2023-04-19T17:41:02.340Z",
  "modificationTime" : "2023-04-19T17:41:02.340Z",
  "quotaInBytes" : -1,
  "quotaInNamespace" : -1,
  "bucketLayout" : "LEGACY",
  "owner" : "cdpuser1",
  "replicationConfig" : {
    "data" : 3,
    "parity" : 2,
    "ecChunkSize" : 1048576,
    "codec" : "RS",
    "replicationType" : "EC",
    "requiredNodes" : 5
  },
  "link" : false
}
`

```console
ozone sh bucket create /vol1/ec9-bucket1 -t EC -r rs-6-3-1024k
ozone sh bucket info  /vol1/ec9-bucket1
```
`{
  "metadata" : { },
  "volumeName" : "vol1",
  "name" : "ec9-bucket1",
  "storageType" : "DISK",
  "versioning" : false,
  "usedBytes" : 0,
  "usedNamespace" : 0,
  "creationTime" : "2023-04-19T17:42:03.273Z",
  "modificationTime" : "2023-04-19T17:42:03.273Z",
  "quotaInBytes" : -1,
  "quotaInNamespace" : -1,
  "bucketLayout" : "LEGACY",
  "owner" : "cdpuser1",
  "replicationConfig" : {
    "data" : 6,
    "parity" : 3,
    "ecChunkSize" : 1048576,
    "codec" : "RS",
    "replicationType" : "EC",
    "requiredNodes" : 9
  },
  "link" : false
}
`

For reference:  you can update replication config for existing buckets:
```console
ozone sh bucket create /vol1/bucket1
ozone sh bucket set-replication-config /vol1/bucket1 -t EC -r rs-3-2-1024k 
ozone sh bucket info /vol1/bucket1
```
`
{
  "metadata" : { },
  "volumeName" : "vol1",
  "name" : "bucket1",
  "storageType" : "DISK",
  "versioning" : false,
  "usedBytes" : 0,
  "usedNamespace" : 0,
  "creationTime" : "2023-04-19T17:42:27.327Z",
  "modificationTime" : "2023-04-19T17:42:32.026Z",
  "quotaInBytes" : -1,
  "quotaInNamespace" : -1,
  "bucketLayout" : "LEGACY",
  "replicationConfig" : {
    "data" : 3,
    "parity" : 2,
    "ecChunkSize" : 1048576,
    "codec" : "RS",
    "replicationType" : "EC",
    "requiredNodes" : 5
  },
  "link" : false
}
`


# Lab 3 Bucket options FSO / OBS
Summary:
- 3 bucket layouts and why
- create a bucket FSO, OBS

Ozone supports multiple bucket layouts
- FILE_SYSTEM_OPTIMIZED (FSO):
  - Hierarchical file system namespace with files and directories.
  - Atomic rename/delete operations supported.
  - Recommended to be used with Hadoop file system compatible interfaces rather than s3 interfaces.
  - Awesome for Hive / Impala
  - Trash implementation.
  
- OBJECT_STORE (OBS):
  - Flat key-value namespace like S3.
  - Recommended to be used with S3 interfaces.
- LEGACY:
  - Provides support for existing buckets created in older versions.
  - Default behavior is compatible with the Hadoop File system. 

Within Vol1 already created, create a bucket with the FSO layout and display the information about the bucket
```console
ozone sh bucket create /vol1/fso-bucket --layout FILE_SYSTEM_OPTIMIZED
ozone sh bucket info /vol1/fso-bucket
```

Expected output
`{
  "metadata" : { },
  "volumeName" : "vol1",
  "name" : "fso-bucket",
  "storageType" : "DISK",
  "versioning" : false,
  "usedBytes" : 0,
  "usedNamespace" : 0,
  "creationTime" : "2023-09-13T10:43:04.625Z",
  "modificationTime" : "2023-09-13T10:43:04.625Z",
  "quotaInBytes" : -1,
  "quotaInNamespace" : -1,
  "bucketLayout" : "FILE_SYSTEM_OPTIMIZED",
  "owner" : "admin",
  "link" : false
}`
Within Vol1 already created, create a bucket with the OBS layout and display the information about the bucket
```console
ozone sh bucket create /vol1/obs-bucket --layout OBJECT_STORE
ozone sh bucket info /vol1/obs-bucket
```

Expected Output
`
{
  "metadata" : { },
  "volumeName" : "vol1",
  "name" : "obs-bucket",
  "storageType" : "DISK",
  "versioning" : false,
  "usedBytes" : 0,
  "usedNamespace" : 0,
  "creationTime" : "2023-09-13T10:44:22.166Z",
  "modificationTime" : "2023-09-13T10:44:22.166Z",
  "quotaInBytes" : -1,
  "quotaInNamespace" : -1,
  "bucketLayout" : "OBJECT_STORE",
  "owner" : "admin",
  "link" : false
}`

# Lab 4 data copy HDFS ⇔ Ozone

Summary
- configure Ranger policy rules
- download dataset, push it to hdfs
- distcp operations hdfs dataset to ozone
- crc checksum validation via a spark-submit job

Prerequisites
In Ranger (log into Ranger UI using admin/Supersecret1).
Select cm_hdfs under the HDFS service.
Once you are on the cm_hdfs page, edit the first policy called all-path by clicking its number or the Edit (pencil) button on the right.
![cm_HDFS](./images/cm_HDFS.png)
Under Allow Conditions, add your admin user to the users with RWX permissions (e.g: admin)
![RangerforHDFS](./images/RangerforHDFS.png)
**Save** your changes to the Ranger policy

The below command creates a bucket by default
```console
ozone fs -mkdir -p ofs://ozone/hive/warehouse/cp/vehicles
```

Copy files to the bucket
```console
ozone fs -cp hdfs:///tmp/vehicles.csv ofs://ozone/hive/warehouse/cp/vehicles
```
*Note*
Files downloaded from https://www.fueleconomy.gov/feg/epadata/vehicles.csv were copied into your tmp folder. If they are missing, ssh as root (usually user is "centos") to the node and run the below command:
*sudo yum install -y wget
wget -qO - https://www.fueleconomy.gov/feg/epadata/vehicles.csv | hdfs dfs -copyFromLocal - /tmp/vehicles.csv*

Once copied over, list the files in the Ozone bucket:
```console
ozone fs -ls ofs://ozone/hive/warehouse/cp/vehicles
```
- Using the ozone fs -cp command is a very slow way to copy files, because only a single client shell on the gateway will download and upload the files between the systems. For greater scalability, you need to have the cluster move the files in parallel, directly from the source to the destination with multiple servers.
- Copy files using the hadoop distcp command. This will submit a MapReduce application to Yarn to run a map side job that will, by default, copy the files using multiple servers (4 containers in parallel). This will be much faster than using the ozone cp command for large files, as the hard work of copying all the files is done by the whole cluster, rather than a single machine with less bandwidth when using the ozone fs cp command. Distcp is a powerful tool for moving files in parallel. It offers many options for syncing and atomically copying data, so that no file is missed even if there is an error in communication. 

```console
ozone fs -mkdir -p ofs://ozone/hive/warehouse/distcp/vehicles
hadoop distcp -m 2 -skipcrccheck hdfs:///tmp/vehicles.csv ofs://ozone/hive/warehouse/distcp/vehicles
```
Another variant: authentication -- Establishes mutual authentication between the client and the server.
```console
hadoop distcp  -Ddfs.data.transfer.protection=authentication -m 2 -skipcrccheck \ hdfs://cdp.54.170.129.255.nip.io:8020/user/admin/* ofs://ozone/my-volume1/my-bucket1
```

List the Ozone files in /tmp
```console
ozone fs -ls  ofs://ozone/hive/warehouse/distcp/vehicles
```

# Lab 5 Hive & Spark on base
Summary:
- Configure ranger policy rules
- Check that hiveServer2 has the right colocation parameters in place
- Start a spark shell session and perform some operations
- Spark shell push data
- Hive via beeline
- Hive via hue

Prerequisites:
We need to change some Ranger policy rules so hive can access the Ozone layer, Ozone Manager also needs to access H2s.
We will also check the h2s parameter to allow managed db and table on both ozone and hdfs

Give your users all privileges on HadoopSQL repo in Ranger. Open Ranger UI, access the Hadoop SQL service, add your user to all - database, table, column policy and Save.
![RangerHadoopSQL](./images/RangerHadoopSQL.png)

In the same HadoopSQL tab, add your user to the "all - url" policy, this is needed for Spark
![RangerAllurls](./images/RangerAllurls.png)

Next, provide “hive” and “yarn” user all privileges on Ozone.

On Ranger UI, go to cm_ozone repo > Edit “all - volume, bucket, key” > Provide “hive” & “yarn” users all privileges.
![yarn_hiveoncm_ozone](./images/yarn_hiveoncm_ozone.png)

Check HiveServer 2 configuration:
 If we want hive managed databases and tables on both ozone and HDFS, we do need to add a parameter within hiveServer2 to make it happen.

within hive_hs2_config_safety_valve add  metastore.warehouse.tenant.colocation=true
![hiveontezconfig](./images/hiveontezconfig.png)

Detailed operations:

You need to add parameter to your spark shell in order to interact with ozone
Spark-shell
Let's push some data to ozone first.
```console
ozone fs -put /var/log/hadoop-ozone/ozone-recon.log ofs://ozone/vol1/bucket1/
```

Open the Spark Shell
```console
spark-shell --conf spark.yarn.access.hadoopFileSystems=ofs://ozone
```
within Spark-shell
```scala
val dfofs=spark.read.option("header", "true").option("inferSchema", "true").csv(s"ofs://ozone/vol1/bucket1/ozone-recon.log")
dfofs.collect()
```
To exit the spark shall
```scala
:quit
```

Another example
```console
hdfs dfs -mkdir -p ofs://ozone/data/vehicles
wget -qO - https://www.fueleconomy.gov/feg/epadata/vehicles.csv | hdfs dfs -copyFromLocal - ofs://ozone/data/vehicles/vehicles.csv 
spark-shell --conf "spark.debug.maxToStringFields=90" --conf spark.yarn.access.hadoopFileSystems="ofs://ozone/" << EOF
```

```scala
val df = spark.read.format("csv").option("header", "true").load("ofs://ozone/data/vehicles/vehicles.csv")
df.createOrReplaceTempView("tempvehicle")
spark.sql("create table vehicles stored as parquet location 'ofs://ozone/data/vehicles/vehicles' as select * from tempvehicle");
EOF
```

Hive Beeline
create an external table on ozone that will be used later in CDW
![Hivetable](./images/Hivetable.png)

Find out the the hostname for your machine
```console
hostname
```
and replace the hostname in the below command: 
`beeline -u "jdbc:hive2://`
**<hostnameX>**
`:10000/default;principal=hive/`
**<hostnameX>**
`@WORKSHOP.COM;ssl=true;sslTrustStore=/opt/cloudera/security/jks/truststore.jks"`

# Lab 6 Ozone S3 gateway

