## Lab 1

Enable and Configure Ranger:
Go to the Ozone service, then select the Configuration tab.
Confirm that RANGER Service is enabled.
This integrates Ozone with Ranger security policies.



```console
kinit admin
``` 

```console
ozone sh volume create /testperm
``` 
> `INFO rpc.RpcClient: Creating Volume: testperms, with admin as owner and space quota set to -1 bytes, counts quota set to -1`

```console
klist
```
> `
Ticket cache: FILE:/tmp/krb5cc_604400000_zNd5iw
Default principal: admin@WORKSHOP.COM
Valid starting       Expires              Service principal
09/11/2023 16:23:20  09/11/2023 17:23:20  krbtgt/WORKSHOP.COM@WORKSHOP.COM renew until 09/12/2023 16:23:20
`
```console
ozone fs -ls /
```

> `drwxr-xr-x   - hbase hbase               0 2023-09-09 13:12 /hbase`
> `drwxr-xr-x   - hdfs  supergroup          0 2023-09-09 13:08 /ranger`
> `drwxrwxr-x   - solr  solr                0 2023-09-09 13:09 /solr-infra`
> `drwxrwxrwt   - hdfs  supergroup          0 2023-09-09 13:19 /tmp`
> `drwxr-xr-x   - hdfs  supergroup          0 2023-09-09 13:17 /user`
> `drwxr-xr-x   - hdfs  supergroup          0 2023-09-09 13:10 /warehouse`
> `drwxr-xr-x   - hdfs  supergroup          0 2023-09-09 13:10 /yarn`

```console
ozone fs -ls ofs://ozone/
```
