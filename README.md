![cbb45f76-6fbd-4e90-9420-a7b195f22571](https://user-images.githubusercontent.com/95745669/234577584-41bf583e-0fa6-4d55-a1f2-decbe22a6348.jpg)


# AzureStack-AKS-Engine
AKS Engine is a command-line tool that automates the deployment of Kubernetes clusters on Azure Stack. With AKS Engine, you can customize the Kubernetes cluster deployment, including specifying the number of nodes, configuring network settings, and choosing the version of Kubernetes to deploy. This tool simplifies the process of deploying Kubernetes on Azure Stack and enables you to manage your Kubernetes clusters using familiar Azure tools and services.
**NOTE :This documentation is based on the AZ-Stack Version 2206**

In order to create a kubernetes cluster in Azure stack there are prerequisites that are divided between two entities :  

- The azure stack operator ( LDC in this case )     

- The tenant ( Our customer )

Lets go through the prerequisites on the operator part 
### Operator Prequisites
- Azure Stack Hub subscription 
  - we need to set a subscription for the tenant on the stack
- Azure Stack Hub 1910 or greater 
  - The AKS engine requires Azure Stack Hub 1910 or greater. 

- AKS Base Images
  - AKS Base Ubuntu and Windows Image for the cluster machines , can be downloaded from the marketplace of the stac
- Linux custom script extension
  - Custom Script for Linux 2.0 Version: 2.0.6 (or latest version) , can be downloaded from the marketplace of the stack
### Tenant's Prerequisites 1
- Application Registeration
  - we need to register an application on the tenant's subscription and assign it's service principal a contributer role 
  
  - login into the public url of azure's portal [Azure](portal.azure.com)
  
  - then go into app registeration
 
    ![b0f91bf9-b7ff-462a-bc33-bbf431da839b](https://user-images.githubusercontent.com/95745669/234536742-6485a38b-5b73-46e8-9de2-55b8d80adb1e.jpg)
  
   -  Register a new app

      ![0206b970-94eb-4725-aa42-aef5af738e96](https://user-images.githubusercontent.com/95745669/234537074-1112de53-48cf-492e-834c-06a028de7b52.jpg)

      ![8f411d0d-71c2-4280-af4c-48ab0daecf97](https://user-images.githubusercontent.com/95745669/234537155-1d6f41f5-3b61-4f89-93e5-c113ddfe66b3.jpg)

    - After creating the app ( example here : kubernetes) , we want to copy the Application (client) ID and the Directory(tenant) ID to use it on the AKS json file where we are creating the cluster

      ![1f376c2b-8599-46f0-b63d-cd6d1e6de294](https://user-images.githubusercontent.com/95745669/234544391-99b2e6d5-c1c3-464c-a1a7-1d66e8eab501.jpg)

    - We need to create a secret for the app to use on the Json file too , save the value after creating the secret as it will be hashed after couple of mins

      ![84b965d1-d920-4d17-9e85-4f3b40d9f8a4](https://user-images.githubusercontent.com/95745669/234548737-19a7ac14-24e2-4064-b8b4-7b69a48ffe51.jpg)

    - login to the azure stack portal url [Azure Stack](portal.eg2.linkdatacenter.net) , and go to subscriptions and choose your own 

      ![b3643bd3-5217-47ff-aea4-4c5f4932fb0a](https://user-images.githubusercontent.com/95745669/234553311-cdc4ddf3-aad6-4efe-919c-99bbd967defd.jpg)

    - Give role contributer for the app we created on azure public on the subscription

      ![8a6b04cc-5a30-4f56-8e2e-f15c1d4cff61](https://user-images.githubusercontent.com/95745669/234553749-246ff4bb-e5bf-4808-9b3d-f48718c94a67.jpg)


### Tenant prerequisites 2
- In order to deploy a kubernetes cluster we need to install the AKS engine on a virtual machine and use the engine to deploy the cluster , but we need to map the version of the engine with version of the stack  

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

[Official documentation of versions mappings](https://learn.microsoft.com/en-us/azure-stack/user/kubernetes-aks-engine-release-notes?view=azs-2206#aks-engine-and-azure-stack-version-mapping)

- The engine can be installed on a [windows VM](https://learn.microsoft.com/en-us/azure-stack/user/azure-stack-kubernetes-aks-engine-deploy-windows?view=azs-2206)  or a [Linux VM](https://learn.microsoft.com/en-us/azure-stack/user/azure-stack-kubernetes-aks-engine-deploy-linux?view=azs-2206)


-------------------
## Preparing The Apimodel Json file for deployment
--------------------
### We need to generate a private/public key pair in order to use it in our api-model file and to ssh into our nodes after deployment
**1**- Open a terminal or command prompt on your local machine / the client VM.

**2**- Type the following command: 
```
ssh-keygen -t rsa.
```
**3**- You will be prompted to enter a file name for the key pair. You can leave the default name or specify a custom name.

**4**- You will be prompted to enter a passphrase for the key pair. This is optional, but it is recommended for added security. If you choose to use a passphrase, make sure it is something you can remember, as you will need to enter it every time you use the key pair.

**5**- The tool will generate a public key and a private key. The public key will have the same file name as the private key, but with a .pub extension. The private key should be kept secret and not shared with anyone.

**6**- Copy the public key to the remote server where you want to use it. You can do this by using the ssh-copy-id command, or by manually copying the contents of the public key file and pasting it into the appropriate file on the remote server.

------------------------------

### After Installing the AKS engine on the client VM we need to create a json file normally called "apimodel" , this file is used by AKS Engine to define the desired state of a Kubernetes cluster that will be deployed on Azure Stack. It contains a set of fields that define various aspects of the cluster, such as its size, location, and configuration. 

Here's an overview of the fields that can be found in the apimodel.json file
```{
    "apiVersion": "vlabs",
    "location": "",
    "properties": {
        "orchestratorProfile": {
            "orchestratorType": "Kubernetes",
            "orchestratorRelease": "1.21",
            "orchestratorVersion": "1.21.10",
            "kubernetesConfig": {
                "cloudProviderBackoff": true,
                "cloudProviderBackoffRetries": 1,
                "cloudProviderBackoffDuration": 30,
                "etcdDiskSizeGB": "30",
                "cloudProviderRateLimit": true,
                "cloudProviderRateLimitQPS": 100,
                "cloudProviderRateLimitBucket": 150,
                "cloudProviderRateLimitQPSWrite": 25,
                "cloudProviderRateLimitBucketWrite": 30,
                "useInstanceMetadata": false,
                "networkPlugin": "kubenet",
                "kubeletConfig": {
                    "--node-status-update-frequency": "1m"
                },
                "controllerManagerConfig": {
                    "--node-monitor-grace-period": "5m",
                    "--pod-eviction-timeout": "5m",
                    "--route-reconciliation-period": "1m"
                }
            }
        },
        "customCloudProfile": {
            "portalURL": "https://portal.eg2.linkdatacenter.net",
            "identitySystem": ""
        },
        "featureFlags": {
            "enableTelemetry": true
        },
        "masterProfile": {
            "dnsPrefix": <UNIQUE_DOMAIN_NAME>,
            "distro": "aks-ubuntu-18.04",
            "count": 1,
            "vmSize": "Standard_F2s_v2"
        },
        "agentPoolProfiles": [
            {
                "name": "linuxpool",
                "count": 2,
                "vmSize": "Standard_DS2_v2",
                "distro": "aks-ubuntu-18.04",
                "availabilityProfile": "AvailabilitySet",
                "AcceleratedNetworkingEnabled": false
            }
        ],
        "linuxProfile": {
            "adminUsername": <VM-ADMIN-USERNAME>,
            "ssh": {
                "publicKeys": [
                    {
                        "keyData": <PUBLIC-SSH-KEY-DATA>
                    }
                ]
            }
        }
    }
}
```

#### We're going to explain the most important fields found in the file :  

properties : contains the main configuration settings for the Kubernetes cluster. The properties object has several sub-objects that define various aspects of the cluster, including:

- orchestratorProfile: An object that specifies the version and type of the Kubernetes orchestrator to use (e.g. Kubernetes 1.21.2) and a kubernetesConfig section that sepcifies additional configuration for the cluster like the etcdDiskSize in GB , the networking plgin (kubenet in this case) and other options.

- customCloudProfile: An object that specifies the cloud provider configuration for the cluster, as the url of azure stack portal . 

- masterProfile: An object that specifies the size and location of the Kubernetes master nodes.

- agentPoolProfiles: An array of objects that specify the size , numebrand location of the Kubernetes worker nodes and other options.

- linuxProfile: An object that specifies the settings for Linux nodes like the username and ssh-public key .


#### The values of this file can vary depending on the different requirements for the cluster , and this sample can provide u with a basic 1 master and workers setup for a kubernetes cluster to test things out and fine tune them if needed 

-------------------

## More information about the API model  
- For a complete reference of all the available options in the API model, refer to the [Cluster definitions](https://github.com/Azure/aks-engine-azurestack/blob/master/docs/topics/clusterdefinitions.md).
- For highlights on specific options for Azure Stack Hub, refer to the [Azure Stack Hub cluster definition specifics](https://github.com/Azure/aks-engine-azurestack/blob/master/docs/topics/azure-stack.md#cluster-definition-aka-api-model).

----------------------
## (OPTIONAL) Custom Subnet creation
#### If you want to deploy to a custom subnet we add a 'vnetSubnetId' field to both of the master and agent pool profiles like this (you can get the value by going into the desired subnet from the portal then copying the path starting from "/subscriptions/..." like the example
```
      "masterProfile": {
            "dnsPrefix": <UNIQUE_DOMAIN_NAME>,
            "distro": "aks-ubuntu-18.04",
            "count": 3,
            "vmSize": "Standard_F4s_v2",
            "vnetSubnetId": "/subscriptions/9e9b6865-4533-443b-b922-6606793dc2cc/resourceGroups/kubernetesPreProd_RGroup/providers/Microsoft.Network/virtualNetworks/PreProd_Vnet/subnets/PreProd_Subnet",
  "firstConsecutiveStaticIP": "10.141.144.100"
        },
        "agentPoolProfiles": [
            {
                "name": "linuxpool",
                "count": 11,
                "vmSize": "Standard_DS5_v2",
                "distro": "aks-ubuntu-18.04",
                "availabilityProfile": "AvailabilitySet",
                "AcceleratedNetworkingEnabled": false,
                "vnetSubnetId": "/subscriptions/9e9b6865-4533-443b-b922-6606793dc2cc/resourceGroups/kubernetesPreProd_RGroup/providers/Microsoft.Network/virtualNetworks/PreProd_Vnet/subnets/PreProd_Subnet"
            }
```

the firstConsecutiveStaticIP field ib the masterProfile sets an ip for the master nodes in the subnet provided ( these ips will be static for the nodes and must not be taken )

For Clusters provisiond by specifiying a custom subnet there are 2 steps to be done in order to get the cluster networking up and running :

#### Step 1 :
Associating the route table ( created by the aks engine for the inter-cluster networking ) to the subnet of the cluster , by heading into subnets in the portal and choosing the subnet then choosing the route table and associating to it 

![subnet-association](https://user-images.githubusercontent.com/95745669/229350516-6983e449-367e-4937-8b9e-03898a6b3583.png)

head to route tables in portal and choose the one created by the aks-engine and check the association 

![route tabe](https://user-images.githubusercontent.com/95745669/229350596-d67d69c3-a5f7-4c20-ba6b-2c1f7cb5ff45.png)

-----
#### Step 2 :
We need to create an inbound rule in the network security group of the cluster , and allow the source to be the cluster subnet ip ranges (e.g xxx.xxx.0.0/16) and the cluster pod subnet ip ranges ( can be found in the address prefix in the route table but the last 2 octets will be 0.0/16) 

<img width="595" alt="Screenshot_7" src="https://user-images.githubusercontent.com/95745669/229350854-1cd3b91c-2172-47c0-b0bf-43979e58361d.png">
 
and the destination of the rule to be also the pod-subnet-ip-ranges

![85381d2e-de8c-4298-a902-98f1411e13be](https://user-images.githubusercontent.com/95745669/234575423-9493de7e-8b60-47d9-a479-40e9e88f4525.jpg)


here the 10.244.0.0/16 is the pod-subnet-ip-ranges and the 192.167.0.0/16 is the cluster subnet ip ranges 


----------------------
## Deploying the cluster 
After creating the Json file there's one more step left to deploy the cluster which is done by running this command 
```
aks-engine deploy \
--azure-env AzureStackCloud \
--location <for asdk is local> \
--resource-group <name-of-resource-group> \
--api-model <path-of-apimodel-json-file> \
--output-directory <any-path-that-is-not-yet-created>\
--client-id xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx \
--client-secret xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx \
--subscription-id xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx \
```
- resource-group: Specifies the name of the Azure Resource Group where the Kubernetes cluster will be deployed. This is a required parameter.

- api-model: the path of the apimodel.json file for the cluster

- location: Specifies the Azure region where the Kubernetes cluster will be deployed.

- output-directory: Specifies the directory where the deployment artifacts will be stored. This directory will contain the apimodel.json file, the generated ARM templates, and the deployment logs.

- subscription-id: Specifies the ID of the Azure subscription to use for the deployment.

- client-id: Specifies the Azure Active Directory (AAD) client ID to use for the deployment.

- client-secret: Specifies the AAD client secret to use for the deployment.

- force-overwrite: Forces the command to overwrite any existing deployment artifacts in the output directory.

---------------------

### Verify Your cluster

**1** Get the public IP address of one of your control plane nodes using the Azure Stack Hub portal.

**2** connect via SSH into the new control plane node using a client such as PuTTY or just a regular bash shell.

- For the SSH username, you use the username procided in the apimodel.json file in the linux/windows profile sections and the private key file of the key pair you provided for the deployment of the cluster.

**3** Check that the cluster endpoints are running:
```
kubectl cluster-info
```
The output should look similar to the following:

```
Kubernetes master is running at https://democluster01.location.domain.com
CoreDNS is running at https://democluster01.location.domain.com/api/v1/namespaces/kube-system/services/kube-dns:dns/proxy
Metrics-server is running at https://democluster01.location.domain.com/api/v1/namespaces/kube-system/services/https:metrics-server:/proxy
```

Then, review node states:

```
kubectl get nodes
```

The output should be similar to the following:

```k8s-linuxpool-29969128-0   Ready      agent    9d    v1.15.5
k8s-linuxpool-29969128-1   Ready      agent    9d    v1.15.5
k8s-linuxpool-29969128-2   Ready      agent    9d    v1.15.5
k8s-master-29969128-0      Ready      master   9d    v1.15.5
k8s-master-29969128-1      Ready      master   9d    v1.15.5
k8s-master-29969128-2      Ready      master   9d    v1.15.5
```

Now the Cluster is up and running you can start deploying your workloads , Happy Orchestrating !
