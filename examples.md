# azurerm Python library programming examples

Here are some simple Python programming examples using the **azurerm** package. In each case, before using these you will need to register your application with Azure, create a Service Principal (app user) and set its access permissions. After these steps you will have a tenant id, an application id and an application secret. See <a href="https://azure.microsoft.com/en-us/documentation/articles/resource-group-authenticate-service-principal/">Authenticating a Service Principal with Azure Resource Manager</a> and are covered <a href="https://msftstack.wordpress.com/2016/01/05/azure-resource-manager-authentication-with-python/">Azure Resource Manager REST calls from Python</a>.

### List Azure subscriptions
```
import azurerm

tenant_id = 'your_tenant_id'
app_id = 'your_application_id'
app_secret = 'your_app_secret'

access_token = azurerm.get_access_token(tenant_id, app_id, app_secret)

# list subscriptions
subscriptions = azurerm.list_subscriptions(access_token)
for sub in subscriptions["value"]:
    print(sub["displayName"] + ': ' + sub["subscriptionId"])
```
	
### List Azure locations and coordinates
```
tenant_id = 'your_tenant_id'
app_id = 'your_application_id'
app_secret = 'your_app_secret'
subscription_id = 'your_sub_id'

access_token = azurerm.get_access_token(tenant_id, app_id, app_secret)

# list locations
locations = azurerm.list_locations(access_token, subscription_id)

for location in locations['value']:
    print(location['name']
          + ', Display Name: ' + location['displayName']
          + ', Coords: ' + location['latitude']
          + ', ' + location['longitude'])
```

### List the resource groups in a subscription
```
import azurerm

tenant_id = 'your_tenant_id'
app_id = 'your_application_id'
app_secret = 'your_app_secret'
subscription_id = 'your_sub_id'

access_token = azurerm.get_access_token(tenant_id, app_id, app_secret)

# list resource groups
resource_groups = azurerm.list_resource_groups(access_token, subscription_id)
for rg in resource_groups["value"]:
    print(rg["name"] + ', ' + rg["location"] + ', ' + rg["properties"]["provisioningState"])
```

### List the storage accounts in a subscription and print storage account quota
```
import azurerm

tenant_id = 'your_tenant_id'
app_id = 'your_application_id'
app_secret = 'your_app_secret'
subscription_id = 'your_sub_id'

# pull the resource group name from the id string
def rgfromid(idstr):
    rgidx = idstr.find('resourceGroups/')
    providx = idstr.find('/providers/')
    return idstr[rgidx + 15:providx]

access_token = azurerm.get_access_token(tenant_id, app_id, app_secret)

# list storage accounts per sub
sa_list = azurerm.list_storage_accounts_sub(access_token, subscription_id)
# print(sa_list)
for sa in sa_list['value']:
    print(sa['name'] + ', ' + sa['properties']['primaryLocation'] + ', ' + rgfromid(sa['id']))

# get storage account quota
quota_info = azurerm.get_storage_usage(access_token, subscription_id)
used = quota_info['value'][0]['currentValue']
limit = quota_info["value"][0]["limit"]
print('\nUsing ' + str(used) + ' accounts out of ' + str(limit) + '.')
```

### Create a storage account
```
import azurerm

tenant_id = 'your_tenant_id'
app_id = 'your_application_id'
app_secret = 'your_app_secret'
subscription_id = 'your_sub_id'

access_token = azurerm.get_access_token(tenant_id, app_id, app_secret)

# create a storage account 
print('Which resource group do you want to create a storage account in?')
resource_group = input()
print('Enter storage account name to create.')
saname = input()
location = 'southeastasia'
sareturn = azurerm.create_storage_account(access_token, subscription_id, resource_group, saname, location)
print(sareturn)
```

### Delete a storage account
```
import azurerm
import sys

def usage():
    sys.exit('Usage: python ' + sys.argv[0] + ' rg_name')

# check for single command argument    
if len(sys.argv) != 2:
    usage()

rgname = sys.argv[1]

tenant_id = 'your_tenant_id'
app_id = 'your_application_id'
app_secret = 'your_app_secret'
subscription_id = 'your_sub_id'

access_token = azurerm.get_access_token(
    tenant_id,
    application_id,
    application_secret
)

# delete a resource groups
rgreturn = azurerm.delete_resource_group(access_token, subscription_id, rgname)
print(rgreturn)
```
### List the virtual machines in a subscription and print the properties
```
import azurerm

tenant_id = 'your_tenant_id'
app_id = 'your_application_id'
app_secret = 'your_app_secret'
subscription_id = 'your_sub_id'

access_token = azurerm.get_access_token(tenant_id, app_id, app_secret)

# loop through resource groups
resource_groups = azurerm.list_resource_groups(access_token, subscription_id)
for rg in resource_groups["value"]:
    rgname = rg["name"] 
    vmlist = azurerm.list_vms(access_token, subscription_id, rgname)
    for vm in vmlist['value']:
        name = vm['name']
        location = vm['location']
        offer = vm['properties']['storageProfile']['imageReference']['offer']
        sku = vm['properties']['storageProfile']['imageReference']['sku']
        print(''.join(['Name: ', name,
                       ', RG: ', rgname,
                       ', location: ', location,
                       ', OS: ', offer, ' ', sku]))
```

### List the VM Scale Sets in a subscription and print basic properties
```
import azurerm

tenant_id = 'your_tenant_id'
app_id = 'your_application_id'
app_secret = 'your_app_secret'
subscription_id = 'your_sub_id'

access_token = azurerm.get_access_token(tenant_id, app_id, app_secret)

# loop through resource groups
resource_groups = azurerm.list_resource_groups(access_token, subscription_id)
for rg in resource_groups["value"]:
    rgname = rg["name"] 
    vmsslist = azurerm.list_vm_scale_sets(access_token, subscription_id, rgname)
    for vmss in vmsslist['value']:
        name = vmss['name']
        location = vmss['location']
        capacity = vmss['sku']['capacity']
        offer = vmss['properties']['virtualMachineProfile']['storageProfile']['imageReference']['offer']
        sku = vmss['properties']['virtualMachineProfile']['storageProfile']['imageReference']['sku']
        print(''.join(['Name: ', name,
                       ', RG: ', rgname,
                       ', location: ', location,
                       ', Capacity: ', str(capacity),
                       ', OS: ', offer, ' ', sku]))
```

### Print VM Scale Set VM properties for all the VMs in all the VM Scale Sets in a subscription
```
import azurerm

tenant_id = 'your_tenant_id'
app_id = 'your_application_id'
app_secret = 'your_app_secret'
subscription_id = 'your_sub_id'

access_token = azurerm.get_access_token(tenant_id, app_id, app_secret)

# loop through resource groups
resource_groups = azurerm.list_resource_groups(access_token, subscription_id)
for rg in resource_groups["value"]:
    rgname = rg["name"] 
    vmsslist = azurerm.list_vm_scale_sets(access_token, subscription_id, rgname)
    for vmss in vmsslist['value']:
        name = vmss['name']
        location = vmss['location']
        capacity = vmss['sku']['capacity']
        offer = vmss['properties']['virtualMachineProfile']['storageProfile']['imageReference']['offer']
        sku = vmss['properties']['virtualMachineProfile']['storageProfile']['imageReference']['sku']
        print(''.join(['Name: ', name,
                       ', RG: ', rgname,
                       ', location: ', location,
                       ', Capacity: ', str(capacity),
                       ', OS: ', offer, ' ', sku]))
        print('Virtual machines...')
        vms = azurerm.list_vmss_vms(access_token, subscription_id, rgname, name)
        for vm in vms['value']:
            print(vm['instanceId'] + ', ' + vm['name'] + '\n')
```

### Change the number of VMs in a VM Scale Set to 5
```
import azurerm

tenant_id = 'your_tenant_id'
app_id = 'your_application_id'
app_secret = 'your_app_secret'
subscription_id = 'your_sub_id'

access_token = azurerm.get_access_token(tenant_id, app_id, app_secret)

scaleoutput = azurerm.scale_vmss(access_token, subscription_id, 'myresourcegroup', 'myvmss', 'Standard_A1', 'Standard', 5)
print(scaleoutput)
```

### Start all the VMs in a VM Scale Set
```
import azurerm

tenant_id = 'your_tenant_id'
app_id = 'your_application_id'
app_secret = 'your_app_secret'
subscription_id = 'your_sub_id'

access_token = azurerm.get_access_token(tenant_id, app_id, app_secret)

startoutput = azurerm.start_vmss(access_token, subscription_id, 'myresourcegroup', 'myvmss')
print(startoutput)
```

### List image publishers available in South East Asia region
```
tenant_id = 'your_tenant_id'
app_id = 'your_application_id'
app_secret = 'your_app_secret'
subscription_id = 'your_sub_id'

access_token = azurerm.get_access_token(tenant_id, app_id, app_secret)

pubs = azurerm.list_publishers(access_token, subscription_id, 'southeastasia')

for pub in pubs:
    print(pub['name'])
```

### List offers from Canonical, skus for UbuntuServer, versions for 15.10 sku
```
tenant_id = 'your_tenant_id'
app_id = 'your_application_id'
app_secret = 'your_app_secret'
subscription_id = 'your_sub_id'

access_token = azurerm.get_access_token(tenant_id, app_id, app_secret)

offers = azurerm.list_offers(access_token, subscription_id, 'southeastasia', 'Canonical')
for offer in offers:
    print(offer['name'])

skus = azurerm.list_skus(access_token, subscription_id, 'southeastasia', 'Canonical', 'UbuntuServer')
for sku in skus:
    print(sku['name'])

versions = azurerm.list_sku_versions(access_token, subscription_id, 'southeastasia', 'Canonical', 'UbuntuServer', '15.10')
for version in versions:
    print(version['name'])
```

### Deploy a VM Scale Set in Southeast Asia from a template, getting the parameters interactively
```
tenant_id = 'your_tenant_id'
app_id = 'your_application_id'
app_secret = 'your_app_secret'
subscription_id = 'your_sub_id'

location = 'Southeast Asia'

access_token = azurerm.get_access_token(tenant_id, app_id, app_secret)

template_uri = 'https://raw.githubusercontent.com/Azure/azure-quickstart-templates/master/201-vmss-lapstack-autoscale/azuredeploy.json'

raw_params = '{     "resourceLocation": {       "value": "LOCATION"     },     "vmSku": {       "value": "Standard_A1"     },     "ubuntuOSVersion": {       "value": "15.04"     },     "vmssName": {       "value": "VMSSNAME"     },     "instanceCount": {       "value": INSTANCECOUNT     },     "adminUsername": {       "value": "ADMINUSER"     },     "adminPassword": {       "value": "ADMINPASSWORD"     }   }'

print('Enter new resource group name')
rgname = input()

# create resource group
rgreturn = azurerm.create_resource_group(access_token, subscription_id, rgname, location)
print(rgreturn)

print('Enter VMSS name')
vmss_name = input()

print('Enter instance count')
instance_count = input()

print('Enter user name')
user_name = input()

print('Enter password')
password = input()

params = raw_params.replace('VMSSNAME', vmss_name)
params = params.replace('INSTANCECOUNT', instance_count)
params = params.replace('LOCATION', location)
params = params.replace('ADMINUSER', user_name)
params = params.replace('ADMINPASSWORD', password)

deploy_return = azurerm.deploy_template_uri(access_token, subscription_id, rgname, 'mydep3', template_uri, params)
print(deploy_return)
```