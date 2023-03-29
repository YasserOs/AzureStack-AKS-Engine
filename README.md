# AzureStack-AKS-Engine
**NOTE :This documentation is based on the AZ-Stack Version 2206**

In order to create a kubernetes cluster in Azure stack there are prerequisites that are divided between two entities :  

- The azure stack operator ( LDC in this case )     

- The tenant ( Our customer )

Lets go through the prerequisites on the operator part 
## Operator Prequisites
- Azure Stack Hub subscription 
  - we need to set a subscription for the tenant on the stack
- Azure Stack Hub 1910 or greater 
  - The AKS engine requires Azure Stack Hub 1910 or greater. 

- AKS Base Images
  - AKS Base Ubuntu and Windows Image for the cluster machines , can be downloaded from the marketplace of the stack
- Linux custom script extension
  - Custom Script for Linux 2.0 Version: 2.0.6 (or latest version) , can be downloaded from the marketplace of the stack

- Application Registeration
  - we need to register an application on the tenant's subscription and assign it's service principal a contributer role 

## Tenant prerequisites
- In order to deploy a kubernetes cluster we need to install the AKS engine on a virtual machine and use the engine to deploy the cluster , but we need to match the version of the engine with version of the stack  

| Azure Stack Hub Version   | AKS Engine Version  | 
:------------- | :----------------- |
| 2206 | 	0.70.0, 0.71.0, 0.73.0, 0.75.3* |


**NOTE : starting from version 0.75.3 and above all ```aks-engine``` commands should be replaced with ```aks-engine-azurestack```**

- AKS engine and corresponding AKS base images mappings (the images needed in the marketplace for the kubernetes cluster VMs ) and Kubernetes versions

| AKS Engine | AKS Base Images for VMS of the cluster     | Installed Kubernetes Version                        |
| :-------- | :------- | :-------------------------------- |
| 0.70.0 |AKS Base Ubuntu 18.04-LTS Image Distro, 2022 Q2 (2022.04.07), AKS Base Windows Image (17763.2565.220408) |1.21.10*, 1.22.7* | 
| 0.71.0 |AKS Base Ubuntu 18.04-LTS Image Distro, 2022 Q3 (2022.08.12), AKS Base Windows Image (17763.3232.220805) |1.22.7*, 1.23.6* | 
| 0.73.0 |AKS Base Ubuntu 18.04-LTS Image Distro, 2022 Q4 (2022.11.02), AKS Base Windows Image (17763.3532.221102) |1.22.15*, 1.23.13* | 
| 0.75.3 |AKS Base Ubuntu 20.04-LTS Image Distro (2023.032.2), AKS Base Windows Server 2019 Image Docker (17763.3887.20230332), AKS Base Windows Server 2019 Image Containerd (17763.3887.20230332) |1.23.15*, 1.24.9** | 

**NOTE: * Starting from Kubernetes v1.21, only Cloud Provider for Azure is supported on Azure Stack Hub**

**NOTE: ** Starting from Kubernetes v1.24, ONLY the containerd container runtime is supported**
- The engine can be installed on a [windows VM](https://learn.microsoft.com/en-us/azure-stack/user/azure-stack-kubernetes-aks-engine-deploy-windows?view=azs-2206)  or a [Linux VM](https://learn.microsoft.com/en-us/azure-stack/user/azure-stack-kubernetes-aks-engine-deploy-linux?view=azs-2206)
