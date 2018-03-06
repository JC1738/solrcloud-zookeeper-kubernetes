SolrCloud Zookeeper Kubernetes
==============================

This project aims to help developers and newbies that would try latest version of SolrCloud (and Zookeeper) in a Kubernetes environment.

### Prerequisite for Minikube installation

 * install Docker lastest version - https://docs.docker.com/engine/installation/
 * install Minikube latest version - https://kubernetes.io/docs/getting-started-guides/minikube/

### Quick start

If you want try a light configuration with 1 SolrCloud container and 1 Zookeeper container, start with:

    git clone https://github.com/freedev/solrcloud-zookeeper-kubernetes.git
    cd solrcloud-zookeeper-kubernetes

### Minikube quick start

The `minikube start` command can be used to start your cluster. 

    minikube start

This command creates and configures a virtual machine that runs a single-node Kubernetes cluster. This command also configures your kubectl installation to communicate with this cluster.

Once minikube is started you need to prepare your virtual machine to deploy Solr and Zookeeper. Because both Solr and Zookeeper need a PersistentVolume where store their data. 
Then run:

    ./prepare-minikube.sh

This will create inside the minikube virtual machine the directories:

    /data/zookeeper/datalog
    /data/zookeeper/data
    /data/zookeeper/logs
    /data/solr/data
    /data/solr/logs

Where Zookeeper and Solr will permanently store their data. 
After that you can start (create) your SolrCloud cluster with 1 Solr instance and 1 Zookeeper instance:

    ./start-minikube.sh

Then run the command `kubectl get pods` to ensure that the pods were created correctly: 

    $ kubectl get pods
    NAME             READY     STATUS    RESTARTS   AGE
    solr-ss-0        1/1       Running   0          12m
    zookeeper-ss-0   1/1       Running   0          12m

Then run the command `minikube service` to see where the services are (which port and ip address): 

    minikube service list

    |-------------|----------------------|---------------------------|
    |  NAMESPACE  |         NAME         |            URL            |
    |-------------|----------------------|---------------------------|
    | default     | kubernetes           | No node port              |
    | default     | solr-service         | http://192.168.64.5:32080 |
    | default     | zookeeper-service    | http://192.168.64.5:32181 |
    | kube-system | kube-dns             | No node port              |
    | kube-system | kubernetes-dashboard | http://192.168.64.5:30000 |
    |-------------|----------------------|---------------------------|

This is an example of the returned output, the ip address and the port for `solr-service` and `zookeeper-service`.
So you'll find the SorlCloud cluster at: http://192.168.64.5:32080

### Prerequisite for Google Cloud installation

<!---
## Quick start


    git clone https://github.com/freedev/solrcloud-zookeeper-kubernetes.git
    cd solrcloud-zookeeper-kubernetes
    ./start.sh
-->


### Introduction to Stateful application in Kubernetes

Before to deploy Solr or Zookeeper in Kubernetes, it is important understand what's the difference between Stateless 
and Stateful applications in Kubernetes.

> Stateless applications
>
> A stateless application does not preserve its state and saves no data to persistent storage — all user and session data stays with the client.
>
> Some examples of stateless applications include web frontends like Nginx, web servers like Apache Tomcat, and other web applications.
>
> You can create a Kubernetes Deployment to deploy a stateless application on your cluster. Pods created by Deployments are not unique and do not preserve their state, which makes scaling and updating stateless applications easier.
>
> Stateful applications
>
>A stateful application requires that its state be saved or persistent. Stateful applications use persistent storage, such as persistent volumes, to save data for use by the server or by other users.
>
>Examples of stateful applications include databases like MongoDB and message queues like Apache ZooKeeper.
>
>You can create a Kubernetes StatefulSet to deploy a stateful application. Pods created by StatefulSets have unique identifiers and can be updated in an ordered, safe way.

So a Solrcloud Cluster matches exactly the kind of Stateful application previously described.
And we have to create the environment following these steps:

1. create configmap where store the cluster configuration
2. create persistent volumes where Solr indexes and Zookeeper logs can be written
3. map solr and zookeeper as network services 
