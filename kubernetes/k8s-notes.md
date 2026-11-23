# Kubernetes

## Some of the biggest challenges facing enterprises

* handling traffic/load on their applications
* cybersecurity - and using AI to assist
* migration to the cloud
  * controlling cost, auditing
  * data governance
* how to use AI to speed up development, testing, deployment - but safely
* data - shadow AI? where is the data stored? how to secure it?
* may want to use Kubernetes
  * enough expertise?
  * need to be able to abstract complexity of Kubernetes away from developers

## Intro to Kubernetes

* Why is Kubernetes needed

  * manage containers, especially to make things more resilient + scalable

* Benefits of Kubernetes

  * orchestrate/schedule and manage containers at scale
  * open-source
  * can run practically anywhere
  * self healing
  * auto scaling
  * load balancing
  * rolling updates and rollbacks
  * define the defined state for the application
  * increased velocity, efficiency, scaling, reducing latency

* Success stories

  * Pokemon Go application

* Kubernetes architecture (include a diagram)


* The cluster setup
  * What is a cluster
    * At least one master node
    * must have at least worker node
    * single node setup is only for dev/testing env - can use Minikube

  * Azure Kubernetes Service (AKS) - how it works
    * Master node and worker nodes are kept separate
    * With AKS...
      * Azure takes of master node
      * Not charged for the master node
      * On VM will run each worker node (paid for each)
    * On AWS Elastic Kubernetes Service (EKS) or Google Kubernetes Engine (GKE)...
      * Charge for the master node (about 10p/hr)
    * Extra benefit: Kubernetes dashboard (GUI interface) on the cloud provider's console/portal

  * Minikube
    * Both the master and worker nodes run on a single VM
    * good for experimentation and learning or dev/testing

  * Master vs worker nodes

    * Master node:
      * Runs on K8s control plane (managing K8s, managing the worker nodes & what is run on them)
      * How many needed for prod env?
        * Min of 3 control plane/master nodes
        * If using cloud managed K8s, you don't need worry about it
    * Worker nodes
      * Often just called `nodes`
      * Where your containers will be deploy and run
      * Work on the data plane

  * Pros and cons of using managed service vs launching your own

    Managed service benefits:
    * Less experts required
    * security - less maintenance
    * save time not having to work out as issues with master nodes
    * focus on delivering value to customers

    Managed service disadvantage:
    * GKE and EKS - need to pay for master node

  * Control plane vs data plane
* Kubernetes objects
  * Research the most common ones, e.g. Deployments, ReplicaSets, Pods
  * What does it mean a pod is "ephemeral"
* How to mitigate security concerns with containers
* Maintained images
  * What are they
  * Pros and cons of using maintained images for your base container images