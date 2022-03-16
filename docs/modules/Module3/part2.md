---
title: Deploy, Configure and test ACI with ALT
parent: Module 3 - Autoscaling with ACI
has_children: false
nav_order: 3
---


# Module 3 Lab Instructions

## Pre-requisites

* Completion of Module 1, which created AKS with a dedicated subnet, **AciSubnet"" required for ACI (virtual nodes)
* Completion of Module 2, which created the Azure Servie, queues, etc.

## Enable virtual node add on

* **virtual node** needs to be enabled on AKS cluster as an add-on to use it for bursting
* Execute the following az cli commands, replacing the `<aks-cluster-name>` and `<aci-subnet-name>` with the values from Module 1

```cli
az provider register --namespace Microsoft.ContainerInstance
az aks enable-addons \
    --name <aks-cluster-name> \
    --addons virtual-node \
    --subnet-name <aci-subnet-name>
```

## Deploying updated demo app

* In Module 2 **order processor app** was deployed on a VM nodepool
* In this module, [deploy-app.yaml](../../../tools/deploy/module3/deploy-app.yaml) file will be updated, so it gets deployed on virtual nodes, as shown below

```yaml
      nodeSelector:
        kubernetes.io/role: agent
        beta.kubernetes.io/os: linux
        type: virtual-kubelet
      tolerations:
      - key: virtual-kubelet.io/provider
        operator: Exists
      - key: azure.com/aci
        effect: NoSchedule
```

* Execute the following cli commands

```cli
cd <module3-folder>

kubectl apply -f deploy/deploy-app.yaml --namespace $demo_app_namespace

kubectl get pod -n $demo_app_namespace -o wide
```

### Scaling on Virtual Nodes with ALT

* In **Module2** ALT was set up to publish messages to ASB queue to scale the demo app.
* For this module, we just need to re-run the same test, and observe scaling of the pod on virtual nodes
* On Azure Load Testing portal page, in `Test` menu, select the test script that was created in Module 2, and click **run**
* At the bottom of Run Review dialog box, click **Run**
* on a bash shell, execute the following

```cli

watch kubectl get pod -n $demo_app_namespace -o wide

```

* In few seconds, you will notice the demo scaling up to process the messages generated by ALT script
* Notice that the pods are starting up separate virtual nodes
