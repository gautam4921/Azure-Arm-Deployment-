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

---
template_02.json was still designed for one VM.

{
    "$schema": "https://schema.management.azure.com/schemas/2019-04-01/deploymentTemplate.json#",
    "contentVersion": "1.0.0.0",

Below is a corrected full template_02.json that keeps your existing VNet/subnet/NSG structure and changes the VM, NIC, and Public IP resources to use the virtualMachines array.


{
    "$schema": "https://schema.management.azure.com/schemas/2019-04-01/deploymentTemplate.json#",
    "contentVersion": "1.0.0.0",

    "parameters": {

        "location": {
            "type": "string"
        },

        "subnetName": {
            "type": "string"
        },

        "virtualNetworkName": {
            "type": "string"
        },

        "publicIpAddressId": {
            "type": "string"
        },

        "pipDeleteOption": {
            "type": "string"
        },

        "publicIpAddressName": {
            "type": "string"
        },

        "publicIpAddressType": {
            "type": "string"
        },

        "publicIpAddressSku": {
            "type": "string"
        },

        "networkInterfaceName": {
            "type": "string"
        },

        "enableAcceleratedNetworking": {
            "type": "bool"
        },

        "networkSecurityGroupName": {
            "type": "string"
        },

        "networkSecurityGroupRules": {
            "type": "array"
        },

        "virtualMachineName": {
            "type": "string"
        },

        "virtualMachineComputerName": {
            "type": "string"
        },

        "virtualMachineRG": {
            "type": "string"
        },

        "virtualMachineSize": {
            "type": "string"
        },

        "osDiskType": {
            "type": "string"
        },

        "osDiskDeleteOption": {
            "type": "string"
        },

        "nicDeleteOption": {
            "type": "string"
        },

        "securityType": {
            "type": "string"
        },

        "adminUsername": {
            "type": "string"
        },

        "adminPassword": {
            "type": "secureString"
        },

        "enablePeriodicAssessment": {
            "type": "string"
        },

        "hibernationEnabled": {
            "type": "bool"
        },

        "vnetLocation": {
            "type": "string",
            "metadata": {
                "description": "Azure region for the deployment, resource group and resources."
            }
        },

        "vnetExtendedLocation": {
            "type": "object"
        },

        "vnetVirtualNetworkName": {
            "type": "string",
            "defaultValue": "myVnet",
            "metadata": {
                "description": "Name of the virtual network resource."
            }
        },

        "vnetTagsByResource": {
            "type": "object",
            "defaultValue": {},
            "metadata": {
                "description": "Optional tags for the resources."
            }
        },

        "vnetProperties": {
            "type": "object",
            "defaultValue": {},
            "metadata": {
                "description": "The properties of the virtual network"
            }
        },

        "vnetNatGatewaysWithNewPublicIpAddress": {
            "type": "array"
        },

        "vnetNatGatewaysWithoutNewPublicIpAddress": {
            "type": "array"
        },

        "vnetNatGatewayPublicIpAddressesNewNames": {
            "type": "array",
            "metadata": {
                "description": "Array of public IP addresses for NAT Gateways."
            }
        },

        "vnetNetworkSecurityGroupsNew": {
            "type": "array",
            "metadata": {
                "description": "Array of network security group objects."
            }
        },

        "vnetResourceGroupName": {
            "type": "string",
            "metadata": {
                "description": "Name of the VNet's containing resource group."
            }
        },

        "vnetDeploymentName": {
            "type": "string",
            "metadata": {
                "description": "Name of the VNet deployment."
            }
        },

        "virtualMachines": {
            "type": "array",
            "metadata": {
                "description": "List of virtual machines to deploy."
            }
        }
    },

    "variables": {

        "vnetId": "/subscriptions/cd6208f9-14cc-4d2d-a201-43ec9eeda060/resourceGroups/azvmsouthindia/providers/Microsoft.Network/virtualNetworks/az_vnet_south_india",

        "subnetRef": "[concat('/subscriptions/cd6208f9-14cc-4d2d-a201-43ec9eeda060/resourceGroups/azvmsouthindia/providers/Microsoft.Network/virtualNetworks/az_vnet_south_india/subnets/', parameters('subnetName'))]",

        "nsgId": "[resourceId(resourceGroup().name, 'Microsoft.Network/networkSecurityGroups', parameters('networkSecurityGroupName'))]",

        "standardSku": {
            "name": "Standard"
        },

        "vnetStaticAllocation": {
            "publicIPAllocationMethod": "Static"
        },

        "vnetPremiumTier": {
            "tier": "Premium"
        },

        "vnetPublicIpAddressesTags": "[if(contains(parameters('vnetTagsByResource'), 'Microsoft.Network/publicIpAddresses'), parameters('vnetTagsByResource')['Microsoft.Network/publicIpAddresses'], json('{}'))]",

        "vnetNatGatewayTags": "[if(contains(parameters('vnetTagsByResource'), 'Microsoft.Network/natGateways'), parameters('vnetTagsByResource')['Microsoft.Network/natGateways'], json('{}'))]"
    },

    "resources": [

        {
            "name": "[parameters('networkSecurityGroupName')]",
            "type": "Microsoft.Network/networkSecurityGroups",
            "apiVersion": "2020-05-01",
            "location": "[parameters('location')]",

            "properties": {
                "securityRules": "[parameters('networkSecurityGroupRules')]"
            }
        },

        {
            "name": "[concat(parameters('virtualMachines')[copyIndex()].name, '-pip')]",
            "type": "Microsoft.Network/publicIpAddresses",
            "apiVersion": "2023-06-01",
            "location": "[parameters('location')]",

            "sku": {
                "name": "[parameters('publicIpAddressSku')]"
            },

            "properties": {
                "publicIPAllocationMethod": "[parameters('publicIpAddressType')]"
            },

            "copy": {
                "name": "publicIpAddressCopy",
                "count": "[length(parameters('virtualMachines'))]"
            }
        },

        {
            "name": "[concat(parameters('virtualMachines')[copyIndex()].name, '-nic')]",
            "type": "Microsoft.Network/networkInterfaces",
            "apiVersion": "2022-11-01",
            "location": "[parameters('location')]",

            "properties": {

                "enableAcceleratedNetworking": "[parameters('enableAcceleratedNetworking')]",

                "ipConfigurations": [
                    {
                        "name": "ipconfig1",

                        "properties": {

                            "subnet": {
                                "id": "[variables('subnetRef')]"
                            },

                            "privateIPAllocationMethod": "Dynamic",

                            "primary": true,

                            "publicIPAddress": {
                                "id": "[resourceId('Microsoft.Network/publicIpAddresses', concat(parameters('virtualMachines')[copyIndex()].name, '-pip'))]",

                                "properties": {
                                    "deleteOption": "[parameters('pipDeleteOption')]"
                                }
                            }
                        }
                    }
                ],

                "networkSecurityGroup": {
                    "id": "[variables('nsgId')]"
                }
            },

            "dependsOn": [
                "[resourceId('Microsoft.Network/networkSecurityGroups', parameters('networkSecurityGroupName'))]",
                "[resourceId('Microsoft.Network/publicIpAddresses', concat(parameters('virtualMachines')[copyIndex()].name, '-pip'))]",
                "[parameters('vnetDeploymentName')]"
            ],

            "copy": {
                "name": "networkInterfaceCopy",
                "count": "[length(parameters('virtualMachines'))]"
            }
        },

        {
            "name": "[parameters('virtualMachines')[copyIndex()].name]",
            "type": "Microsoft.Compute/virtualMachines",
            "apiVersion": "2025-11-01",
            "location": "[parameters('location')]",

            "properties": {

                "hardwareProfile": {
                    "vmSize": "[parameters('virtualMachineSize')]"
                },

                "storageProfile": {

                    "osDisk": {
                        "createOption": "FromImage",

                        "managedDisk": {
                            "storageAccountType": "[parameters('osDiskType')]"
                        },

                        "deleteOption": "[parameters('osDiskDeleteOption')]"
                    },

                    "imageReference": {
                        "publisher": "canonical",
                        "offer": "ubuntu-24_04-lts",
                        "sku": "ubuntu-pro",
                        "version": "latest"
                    }
                },

                "networkProfile": {

                    "networkInterfaces": [
                        {
                            "id": "[resourceId('Microsoft.Network/networkInterfaces', concat(parameters('virtualMachines')[copyIndex()].name, '-nic'))]",

                            "properties": {
                                "deleteOption": "[parameters('nicDeleteOption')]"
                            }
                        }
                    ]
                },

                "securityProfile": {
                    "securityType": "[parameters('securityType')]"
                },

                "osProfile": {

                    "computerName": "[parameters('virtualMachines')[copyIndex()].computerName]",

                    "adminUsername": "[parameters('adminUsername')]",

                    "adminPassword": "[parameters('adminPassword')]",

                    "linuxConfiguration": {

                        "patchSettings": {
                            "assessmentMode": "[parameters('enablePeriodicAssessment')]",
                            "patchMode": "ImageDefault"
                        }
                    }
                },

                "diagnosticsProfile": {

                    "bootDiagnostics": {
                        "enabled": true
                    }
                },

                "additionalCapabilities": {

                    "hibernationEnabled": "[parameters('hibernationEnabled')]"
                }
            },

            "dependsOn": [
                "[resourceId('Microsoft.Network/networkInterfaces', concat(parameters('virtualMachines')[copyIndex()].name, '-nic'))]",
                "[parameters('vnetDeploymentName')]"
            ],

            "copy": {
                "name": "virtualMachineCopy",
                "count": "[length(parameters('virtualMachines'))]"
            }
        },

        {
            "type": "Microsoft.Resources/deployments",
            "apiVersion": "2021-04-01",
            "name": "[parameters('vnetDeploymentName')]",
            "resourceGroup": "[parameters('vnetResourceGroupName')]",

            "properties": {

                "mode": "Incremental",

                "template": {

                    "$schema": "https://schema.management.azure.com/schemas/2019-04-01/deploymentTemplate.json#",

                    "contentVersion": "1.0.0.0",

                    "resources": [

                        {
                            "type": "Microsoft.Network/virtualNetworks",
                            "apiVersion": "2024-01-01",
                            "name": "[parameters('vnetVirtualNetworkName')]",
                            "location": "[parameters('vnetLocation')]",

                            "extendedLocation": "[if(empty(parameters('vnetExtendedLocation')), json('null'), parameters('vnetExtendedLocation'))]",

                            "tags": "[if(empty(parameters('vnetTagsByResource')), json('{}'), parameters('vnetTagsByResource'))]",

                            "properties": "[parameters('vnetProperties')]"
                        }
                    ]
                }
            },

            "copy": {
                "name": "vnetDeploymentCopy",
                "count": 1
            },

            "dependsOn": [
                "natGatewaysWithNewPublicIpAddressCopy",
                "natGatewaysWithoutNewPublicIpAddressCopy",
                "networkSecurityGroupsCopy",
                "natGatewayPublicIpAddressesCopy"
            ]
        },

        {
            "condition": "[greater(length(parameters('vnetNatGatewaysWithoutNewPublicIpAddress')), 0)]",

            "apiVersion": "2020-11-01",

            "type": "Microsoft.Network/natGateways",

            "name": "[parameters('vnetNatGatewaysWithoutNewPublicIpAddress')[copyIndex()].name]",

            "location": "[parameters('vnetLocation')]",

            "tags": "[variables('vnetNatGatewayTags')]",

            "sku": "[variables('standardSku')]",

            "properties": "[parameters('vnetNatGatewaysWithoutNewPublicIpAddress')[copyIndex()].properties]",

            "copy": {
                "name": "natGatewaysWithoutNewPublicIpAddressCopy",

                "count": "[length(parameters('vnetNatGatewaysWithoutNewPublicIpAddress'))]"
            }
        },

        {
            "condition": "[greater(length(parameters('vnetNatGatewaysWithNewPublicIpAddress')), 0)]",

            "apiVersion": "2020-11-01",

            "type": "Microsoft.Network/natGateways",

            "name": "[parameters('vnetNatGatewaysWithNewPublicIpAddress')[copyIndex()].name]",

            "location": "[parameters('vnetLocation')]",

            "tags": "[variables('vnetNatGatewayTags')]",

            "sku": "[variables('standardSku')]",

            "properties": "[parameters('vnetNatGatewaysWithNewPublicIpAddress')[copyIndex()].properties]",

            "dependsOn": [
                "natGatewayPublicIpAddressesCopy"
            ],

            "copy": {
                "name": "natGatewaysWithNewPublicIpAddressCopy",

                "count": "[length(parameters('vnetNatGatewaysWithNewPublicIpAddress'))]"
            }
        },

        {
            "condition": "[greater(length(parameters('vnetNatGatewayPublicIpAddressesNewNames')), 0)]",

            "type": "Microsoft.Network/publicIpAddresses",

            "apiVersion": "2020-11-01",

            "name": "[parameters('vnetNatGatewayPublicIpAddressesNewNames')[copyIndex()].name]",

            "location": "[parameters('vnetLocation')]",

            "sku": "[variables('standardSku')]",

            "tags": "[variables('vnetPublicIpAddressesTags')]",

            "properties": "[variables('vnetStaticAllocation')]",

            "copy": {
                "name": "natGatewayPublicIpAddressesCopy",

                "count": "[length(parameters('vnetNatGatewayPublicIpAddressesNewNames'))]"
            }
        },

        {
            "condition": "[greater(length(parameters('vnetNetworkSecurityGroupsNew')), 0)]",

            "apiVersion": "2020-11-01",

            "type": "Microsoft.Network/networkSecurityGroups",

            "name": "[parameters('vnetNetworkSecurityGroupsNew')[copyIndex()].name]",

            "location": "[parameters('vnetLocation')]",

            "tags": "[if(contains(parameters('vnetTagsByResource'), 'Microsoft.Network/networkSecurityGroups'), parameters('vnetTagsByResource')['Microsoft.Network/networkSecurityGroups'], json('{}'))]",

            "properties": {},

            "copy": {
                "name": "networkSecurityGroupsCopy",

                "count": "[length(parameters('vnetNetworkSecurityGroupsNew'))]"
            }
        }
    ]
}
