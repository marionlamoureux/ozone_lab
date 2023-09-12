
This lab is designed to go through some Ozone fundamentals such as volume bucket and key creations. 

We will also explore some Cloudera security elements and how Ozone integrates broadly with the Cloudera ecosystem and beyond with the s3 gateway interaction.

# Lab 1 Security

Summary
- Enable Ranger from the Cloudera Manager UI
- Test Ranger privileges
- Review Ozone Security Settings from the command line

## 1.1 Enable and Configure Ranger
In Cloudera Manager, go to the Ozone service.


Select the Ozone Configuration tab and onfirm that RANGER Service is enabled.
![[Ozone Configuration - Ranger enabled.png]]
This integrates Ozone with Ranger security policies.

Access the Ranger Service and access the Ranger UI
![[Ranger Service - UI url.png]]

Select cm_ozone under the Ozone service.
![[cm_ozone in Ranger.png]]

In the top policy listed as **all - volume, bucket and key**, click the pencil icon on the right that shows Edit when you hover over it.

![[Edit policy for Ozone.png]]


Structure of the Ranger permissions for Ozone

Ozone Data Layout
![[Ozone Data Layout.png]]

Ranger - Allow and deny conditions
![[Ranger - Allow and Deny conditions.png]]

![[Ozone permission page.png]]


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
- Click on the + button under the "Allow Conditions"![[Add Allow Condition.png]]
- Under “Allow” condition add “read/list” privileges for the user Bob
![[Allow Ragner policy.png]]

- Under "Deny" condition, add "create" privileges for user Bob

![[Deny Ranger Policy.png]]

Add your username to the first “allow” condition (e.g. admin) to provide all privileges to your user.
![[Add user to Ozone permissions.png]]

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
Try the same command:
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


```console
ozone getconf -confKey ozone.om.kerberos.principal
ozone getconf -confKey ozone.om.http.auth.kerberos.principal
ozone getconf -confKey ozone.om.http.auth.kerberos.keytab
```

# Lab 2 Ozone protocol operations

### Ozone Protocols:

Ozone has multiple protocols to work with for a variety of operations. 
There is no ONE PROTOCOL THAT RULES THEM ALL yet.

![[Ozone protocoles.png]]

Ozone is a multi-protocol storage system with support for the following interfaces:
- ofs: Hadoop-compatible file system allows any application that expects an HDFS like interface to work against Ozone with no changes. Frameworks like Apache Spark, YARN, and Hive work against Ozone without the need for any change. 
- s3: Amazon’s Simple Storage Service (S3) protocol. You can use S3 clients and S3 SDK-based applications without any modifications to Ozone.  => try to avoid this protocol since all passes through the s3ateway 
- o3fs: A bucket-rooted Hadoop Compatible file system interface. 
- o3: An object store interface that can be used from the Ozone shell.

### Ozone CLIs
Ozone CLI is used to access Ozone. 
- ozone fs - Runs Hadoop filesystem compatible commands on FSO(File System Optimized) and LEGACY buckets. Compatible with ofs and o3fs interfaces. Supports trash implementation.
- ozone sh - Ozone command shell interface to access Ozone as a key-value store. Command format is: ozone sh object action url. Object can be volume/bucket/key. Compatible with o3 interface.

#### Ozone fs
Summary operations
- interact with HDFS
- interact with ozone
- create a volume
- create a bucket
- push data to a bucket
- delete data from a bucket



The Ozone client can access Ozone as a file system and as a key-value store. When Ozone is installed with the HDFS dependency, the Ozone client library support is built into the HDFS client commands, which will therefore be available for use with Ozone. **hdfs dfs** can also be used (if ozone is not the default fs, a URI path is needed.)


```console
ozone fs -ls /
```

expended output is the list of files stored in HDFS:
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
Volumes are at the highest level of the Ozone file system and are used to manage buckets that store keys. Quotas and user permissions can be applied to volumes for high-level file system management.
`drwxrwxrwx   - om                       0 2023-09-11 18:29 ofs://ozone/s3v`


Create a volume called vol1 and list the ozone file system to see volumes
```console
ozone fs -mkdir ofs://ozone/vol1
ozone fs -ls ofs://ozone/
```

Expended output:
`drwxrwxrwx   - om                       0 2023-09-11 18:29 ofs://ozone/s3v`
`drwxrwxrwx   - admin admins             0 2023-09-12 14:34 ofs://ozone/vol1`

Create a bucket in vol1 called bucket1. Buckets are used to store files.
```console
ozone fs -mkdir ofs://ozone/vol1/bucket1
ozone fs -ls ofs://ozone/vol1
```

```console
sudo cp /var/log/cloudera-scm-agent/cloudera-scm-agent.log /tmp/cloudera-scm-agent.log
ozone fs -put /tmp/cloudera-scm-agent.log ofs://ozone/vol1/bucket1
```

#### Ozone sh



# Lab 3 Bucket options FSO / OBS
Summary:
- 3 bucket layouts and why
- create a bucket FSO, obs


# Lab 4 data copy HDFS ⇔ Ozone

Summary
- configure Ranger policy rules
- download dataset, push it to hdfs
- distcp operations hdfs dataset to ozone
- crc checksum validation via a spark-submit job

# Lab 5 Hive & Spark on base

# Lab 6 Ozone S3 gateway

