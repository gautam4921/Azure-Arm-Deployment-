# Azure-Arm-Deployment-
Azure Arm Deployment  using azure Ado 
First create a service connection from azure devops to azure portal for deploying your azure arm template deployment 
Go to project settings ---go to pipelines ---Navigate to Service connection ---Create service connection -----Select Azure Resource Manager 

Service Connection Name : Azure-devops-svc-connection to Azure -----Grant Permission to all Pipelines 

<img width="858" height="1398" alt="image" src="https://github.com/user-attachments/assets/e6081ac5-5854-4974-97f0-93912c8ef67b" />

<img width="2856" height="1610" alt="image" src="https://github.com/user-attachments/assets/ed09986e-7a7d-4721-bd0b-c85e65ed6952" />

<img width="2868" height="1600" alt="image" src="https://github.com/user-attachments/assets/101d6a17-4bfc-4124-a829-32991b526044" />

Also ---go to ogrganisation settings and disable these settings 

<img width="2854" height="1596" alt="image" src="https://github.com/user-attachments/assets/e17eec70-964c-4e38-b446-4b5090411b50" />


The Complete Yaml should look like this 

trigger:
- main

pool:
  vmImage: 'ubuntu-latest'

variables:
- group: 'Variable Group for AzUbuntu24'

stages:

- stage: Deploy
  displayName: 'Deploy ARM Template'

  jobs:

  - job: Deploy
    displayName: 'Deploy ARM Template'

    steps:

    - checkout: self

    # Verify variables and ARM files
    - bash: |
        echo "Checking ARM template files..."
        ls -la "$(Build.SourcesDirectory)"

        echo "Location variable:"
        echo "$(location)"

        echo "VM name:"
        echo "$(virtualMachineName)"

        echo "VM size:"
        echo "$(virtualMachineSize)"

        test -f "$(Build.SourcesDirectory)/template.json"
        test -f "$(Build.SourcesDirectory)/parameters.json"

      displayName: 'Validate Pipeline Variables and ARM Files'

    - task: AzureResourceManagerTemplateDeployment@3
      displayName: 'Deploy ARM Template'

      inputs:
        deploymentScope: 'Resource Group'

        azureResourceManagerConnection: 'Azure-devops-svc-connection to Azure'

        subscriptionId: 'cd6208f9-14cc-4d2d-a201-43ec9eeda060'

        action: 'Create Or Update Resource Group'

        resourceGroupName: 'azvmsouthindia'

        location: 'South India'

        templateLocation: 'Linked artifact'

        csmFile: '$(Build.SourcesDirectory)/template.json'

        csmParametersFile: '$(Build.SourcesDirectory)/parameters.json'

        overrideParameters: >
          -location "$(location)"
          -virtualMachineName "$(virtualMachineName)"
          -virtualMachineSize "$(virtualMachineSize)"
          -networkInterfaceName "$(networkInterfaceName)"
          -osDiskType "$(osDiskType)"
          -publicIpAddressName "$(publicIpAddressName)"
          -virtualMachineComputerName "$(virtualMachineComputerName)"
          -adminUsername "$(azureuser)"
          -adminPassword "$(adminPassword)"

        deploymentMode: 'Incremental'

        deploymentName: 'AzArm-CI-Azubu2402'

  -------------------------------------------------------------

  Variable Group has the variable
Azure DevOps → Pipelines → Library → Variable groups
Open -----------Variable Group for AzUbuntu24

The spelling must be exactly: as below 

location
virtualMachineName
virtualMachineSize
networkInterfaceName
osDiskType
publicIpAddressName
virtualMachineComputerName
azureuser
adminPassword

Example:
location                   = South India
virtualMachineName         = azubuntu24si01
virtualMachineSize         = Standard_B2s
networkInterfaceName      = azubuntu24sinic01
osDiskType                 = Premium_LRS
publicIpAddressName        = azubuntu24sinic01-ip
virtualMachineComputerName = azubuntu24si01
azureuser                  = azureuser
adminPassword              = ********


@@@@@@@@@@@ Most important: Pipeline permissions

Inside the Variable Group, check: Pipeline permissions 
Your pipeline must be authorized to use: Variable Group for AzUbuntu24
If you see an option like: Authorize for use in all pipelines

Also check your YAML
The top of your YAML must be: 

trigger:
- main

pool:
  vmImage: 'ubuntu-latest'

variables:
- group: 'Variable Group for AzUbuntu24'

stages:
- stage: Deploy


#### Add this diagnostic step

Put this before AzureResourceManagerTemplateDeployment@3:

- bash: |
    echo "========== PIPELINE VARIABLE CHECK =========="

    echo "location:"
    echo "$(location)"

    echo "virtualMachineName:"
    echo "$(virtualMachineName)"

    echo "virtualMachineSize:"
    echo "$(virtualMachineSize)"

    echo "networkInterfaceName:"
    echo "$(networkInterfaceName)"

    echo "osDiskType:"
    echo "$(osDiskType)"

    echo "publicIpAddressName:"
    echo "$(publicIpAddressName)"

    echo "virtualMachineComputerName:"
    echo "$(virtualMachineComputerName)"

    echo "============================================"

displayName: 'Verify Variable Group Variables'
You should see something like:
========== PIPELINE VARIABLE CHECK ==========

location:
South India

virtualMachineName:
azubuntu24si01

virtualMachineSize:
Standard_B2s

networkInterfaceName:
azubuntu24sinic01

osDiskType:
Premium_LRS

publicIpAddressName:
azubuntu24sinic01-ip

virtualMachineComputerName:
azubuntu24si01

============================================  


  
