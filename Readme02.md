How to deploy 3 vms at a time using the same arm template without disturbing variables?

You should not manually change the Variable Group every time. For deploying 3 VMs, a better production design is to make the VM configuration data-driven and use an ARM copy loop


Recommended design : Instead of putting:  

virtualMachineName
networkInterfaceName
publicIpAddressName

in the Variable Group, keep only common configuration there:

location = South India
virtualMachineSize = Standard_B2s
osDiskType = Premium_LRS
azureuser = azureuser 

Then define the 3 VM names in parameters.json.

json
{
  "$schema": "https://schema.management.azure.com/schemas/2019-04-01/deploymentParameters.json#",
  "contentVersion": "1.0.0.0",

  "parameters": {

    "location": {
      "value": "South India"
    },

    "virtualMachineSize": {
      "value": "Standard_B2s"
    },

    "osDiskType": {
      "value": "Premium_LRS"
    },

    "adminUsername": {
      "value": "azureuser"
    },

    "virtualMachines": {
      "value": [
        {
          "name": "azubuntu24si01",
          "computerName": "azubuntu24si01"
        },
        {
          "name": "azubuntu24si02",
          "computerName": "azubuntu24si02"
        },
        {
          "name": "azubuntu24si03",
          "computerName": "azubuntu24si03"
        }
      ]
    }
  }
}

Then your ARM template can automatically generate: using ARM copy loop 

The ARM template can automatically generate:
azubuntu24si01
azubuntu24si01-nic
azubuntu24si01-pip

azubuntu24si02
azubuntu24si02-nic
azubuntu24si02-pip

azubuntu24si03
azubuntu24si03-nic
azubuntu24si03-pip

So if tomorrow you want 5 VMs, you simply add:

{
  "name": "azubuntu24si04",
  "computerName": "azubuntu24si04"
},
{
  "name": "azubuntu24si05",
  "computerName": "azubuntu24si05"
}

No change to the pipeline is required.

