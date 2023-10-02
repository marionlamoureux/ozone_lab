The lab document is configured to run on the edge2ai 1 node setup.
reference:  https://github.com/asdaraujo/edge2ai-workshop 
Some checks needs to be done if you use the public repo:
it is recommended to take out knox from the deployment methodology
check hive metastore.warehouse.tenant.colocation=true parameter in h2s
hive_hs2_config_safety_valve
if hue is used double check the safety valve for the ldap configuration
you can use the 7.1.8 outside the paywall but you need a hotfix CHF 4 to deploy the ozone parcels reference: https://docs.cloudera.com/storage/latest/storage-options/topics/ozone-parcel2-requirements.html 
prior to deployment please update the stack.sh file where cm_services and other options needs to be updated accordingly

If you use the internal repo. please use the 7.1.8 paywall version which means you will need to use a cldr license key in order to deploy the runtime
