Lab 1

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
