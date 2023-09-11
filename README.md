Lab 1

Enable and Configure Ranger:
Go to the Ozone service, then select the Configuration tab.
Confirm that RANGER Service is enabled.
This integrates Ozone with Ranger security policies.



```console
kinit admin
``` 

```console
ozone sh volume create /testperms
``` 
INFO rpc.RpcClient: Creating Volume: testperms, with admin as owner and space quota set to -1 bytes, counts quota set to -1



>>> ozone sh volume create /testperms

INFO rpc.RpcClient: Creating Volume: testperms, with admin as owner and space quota set to -1 bytes, counts quota set to -1

At the command prompt, type ```kinit admin```


<span style="background-color:black"> Some *blue* text </span>


<div style="background-color:rgba(0, 0, 0, 0.0470588); text-align:center; vertical-align: middle; padding:40px 0; margin-top:30px">
VIEW THE BLOG
</div>

<div class="blue">
    This markdown cell is blue.
</div>

<style>
    .blue {
        background-color: #0074D9;
    }
</style>





%%html
<style>
    .blue {
        background-color: #0074D9;
    }
    .green {
        background-color: #2ECC40;
    }
</style>

<mark style="background-color: #FFFF00">Highlighted text</mark>

<span style="color:red">
Text content
</span>
