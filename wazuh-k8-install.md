# 👺 Deploy Wazuh in Kubernetes: A Step-by-Step Guide 👺

![alt text](images/image.png)

## Introduction

Looking to enhance your security monitoring and incident response with Wazuh? This comprehensive guide will walk you through the process of deploying Wazuh in a Kubernetes cluster, ensuring you can leverage its powerful security features in a scalable and containerized environment.

By the end of this guide, you'll have Wazuh deployed in your Kubernetes cluster, ready to provide comprehensive security monitoring and incident response capabilities. Let's get started!

## Prerequisites 🐼:-

+ **Kubernetes Cluster:** A running Kubernetes cluster. 

+ **Kubectl Installed:** Kubectl should be installed and configured to interact with your Kubernetes cluster.

+ **Persistent Storage:** Configure persistent storage for Wazuh components, as they require persistent volumes for data storage.

## Steps 🍄:-

**Step 1** — Clone the repository

```
git clone https://github.com/wazuh/wazuh-kubernetes.git -b v4.8.0 --depth=1
cd wazuh-kubernetes
```

**Step 2** — Generate SSL Certificates

Generate self-signed certificates for the Wazuh indexer cluster using the script at wazuh/certs/indexer_cluster/generate_certs.sh, or provide your own.

```
./wazuh/certs/indexer_cluster/generate_certs.sh
```

Generate self-signed certificates for the Wazuh dashboard cluster using the script at wazuh/certs/dashboard_http/generate_certs.sh, or provide your own.

```
./wazuh/certs/dashboard_http/generate_certs.sh
```

**Step 3** — Setup Storage Class

The Storage Class provisioner may vary based on your cluster type. Check your provisioner by running:

```
$ kubectl get sc
```

Result: 
```
NAME                          PROVISIONER            RECLAIMPOLICY   VOLUMEBINDINGMODE   ALLOWVOLUMEEXPANSION   AGE
elk-gp2                       microk8s.io/hostpath   Delete          Immediate           false                  67d
microk8s-hostpath (default)   microk8s.io/hostpath   Delete          Immediate           false                  54d
```

If the provisioner column shows microk8s.io/hostpath, edit the file envs/local-env/storage-class.yaml and configure this provisioner.

**Step 3** — Apply All Manifests Using Kustomize

Deploy the cluster with a single command using the customization file:

```
$ kubectl apply -k envs/local-env/
```

**Step 4** — Verifying the deployment

**4A** — Namespace

```
$ kubectl get namespaces | grep wazuh
```

Output:

```
wazuh         Active    12m
```

**4B** — Services

```
$ kubectl get services -n wazuh
```

Output:

```
NAME            TYPE        CLUSTER-IP       EXTERNAL-IP   PORT(S)                          AGE
dashboard       NodePort    10.108.185.154   <none>        443:32763/TCP                    127m
indexer         NodePort    10.102.251.255   <none>        9200:32122/TCP                   127m
wazuh           NodePort    10.108.94.169    <none>        1515:30755/TCP,55000:31047/TCP   127m
wazuh-cluster   ClusterIP   None             <none>        1516/TCP                         127m
wazuh-indexer   ClusterIP   None             <none>        9300/TCP                         127m
wazuh-workers   NodePort    10.104.69.166    <none>        1514:31773/TCP                   127m
```

**4C** — Deployments

```
$ kubectl get deployment -n wazuh
```

Output:

```
NAME              READY   UP-TO-DATE   AVAILABLE   AGE
wazuh-dashboard   1/1     1            1           128m
```

**4D** — Statefulset

```
$ kubectl get statefulsets -n wazuh
```

Output:

```
NAME                   READY   AGE
wazuh-indexer          1/1     129m
wazuh-manager-master   1/1     129m
wazuh-manager-worker   1/1     129m
```

**4E** — Pods

```
NAME                               READY   STATUS    RESTARTS        AGE
wazuh-dashboard-76bb664bdc-gr527   1/1     Running   0               129m
wazuh-indexer-0                    1/1     Running   0               129m
wazuh-manager-master-0             1/1     Running   0               20m
wazuh-manager-worker-0             1/1     Running   0               20m
```

**Step 4** — Access Wazuh Dashboard

If you created domain names for the services, access the dashboard using the proposed domain name: https://wazuh.your-domain.com. Check the services:

```
$ kubectl get services -o wide -n wazuh
```

Output:

```
NAME            TYPE        CLUSTER-IP       EXTERNAL-IP   PORT(S)                          AGE    SELECTOR
dashboard       NodePort    10.108.185.154   <none>        443:32763/TCP                    138m   app=wazuh-dashboard
```

The Wazuh dashboard is accessible at **https://<IP_ADDRESS>:32763**. The default credentials are **admin:SecretPassword**.

## Final Note

If you find this repository useful for learning, please give it a star on GitHub. Thank you!

**Authored by:** [ELemenoppee](https://github.com/ELemenoppee)