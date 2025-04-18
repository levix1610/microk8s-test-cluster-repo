### MicroK8s Deployment Instructions:  

These instructions work to get you up and running with a simple 3 node microk8s Kubernetes cluster. 

Once you get 3 Ubuntu servers up and running, whether virtual or physical.  SSH into all 3 of the nodes.  

Install on all nodes, run the command below:

```
sudo snap install microk8s --classic
```

Check status of the microk8s installation:

```
microk8s status --wait-ready
```

It is good idea to add your user to the microk8s group so you do not have to run as sudo:

```
sudo usermod -a -G microk8s <username>
```

Back on node 1:

```
microk8s add-node
```

command will generate a unique token to join the node.  Run the command twice and run each on node 2 and 3 respectively.

to list the nodes and see if they are all joined to the cluster:

```
microk8s kubectl get nodes
```

Next, will need to install the microceph package. Run this command on all 3 nodes:

```
sudo snap install microceph --channel=latest/edge
```

Once the package is installed on all, run this command to start the cluster on Node 1:

```
microceph cluster bootstrap
```

Next, run this command on Node 1 for each nother node, (1, 2, 3, ...).  Make sure to add each unique name of the node each time you run the command:

```
microceph cluster add <name of other nodes>
```

Next run this command, with the unique token generated in the previous step.  On each corresponding node respectively.

```
microceph cluster join <unique token generated>
```

  
Now, will need to check to make sure we have a spare unused disc.  Make sure you can either add another virtual disk or physical one, depending on what your cluster is running on:

```
lsblk
```

next, run this command on all 3 nodes.  replace 'sdb' with whatever the storage target is on your system:

```
microceph disk add /dev/sdb --wipe
```

then run back on node 1 to initialize the cluster across the nodes:

```
microk8s enable rook-ceph
```

will say to run the command, which will complete the process. Run on node 1 as well:

```
sudo microk8s connect-external-ceph
```

will show the default storage name then after which will be something like 'ceph-rbd'  

From here you are ready to use your cluster.  Commands can be executed by using 'microk8s kubectl' <command>.

####   
Other Useful Commands:

Show Storage:

```
microk8s kubectl get sc
```

Show ALL:

```
microk8s kubectl get all --all-namespaces
```

Change Default Storage Path:

```
microk8s kubectl patch storageclass ceph-rbd -p '{"metadata": {"annotations":{"storageclass.kubernetes.io/is-default-class":"true"}}}'
```

Install Portainer:

```
microk8s kubectl apply -n portainer -f https://downloads.portainer.io/ee-lts/portainer.yaml
```

can then reach @ https://localhost:30779

get all services and namespaces:

```
microk8s kubectl get svc -A
```

  
to deploy a yaml (yml) files:

```
kubectl apply -f my-app-deployment.yaml
```

#### Deploying on Microk8s ex:

Well these commands can be used to quickly deploy pods and services.  It is strongly recommended not to do so in this manor.  And, to build it from yaml files.  However, the commands are listed below for examples.  

```
microk8s kubectl create deployment <deployment_name> --image=<image_name>:<tag>
```

ex:

```
microk8s kubectl create deployment my-nginx --image=nginx:latest
```

Create Cluster IP service

(Port and target-port can be the same #.  Check by viewing namespaces and copy over works fine)

```
microk8s kubectl expose deployment <deployment_name> --port=<port> --target-port=<target_port> --name=<service_name>
```

ex:

```
microk8s kubectl expose deployment my-nginx --port=80 --target-port=80 --name=my-nginx-service
```

Create external Port:

(this creates the port so you can access the pod/container via the clusters IPs)

```
microk8s kubectl expose deployment <deployment_name> --type=NodePort --port=<port> --target-port=<target_port> --name=<service_name>
```

ex:

```
microk8s kubectl expose deployment my-nginx --type=NodePort --port=80 --target-port=80 --name=nginx-nodeport-service
```

Scaling up a deployments Pods:

(If you want to replicate the number of Pods for a deployment.)

```
microk8s kubectl scale deployment <deployment_name> --replicas=<number_of_replicas>
```

ex:

```
microk8s kubectl scale deployment influxdb --replicas=3
```

Be careful on the replication of some deployments as not all scale well and do better off with a single pod deployment.  

Deleting Deployments and Services:

```
microk8s kubectl delete deployment <deployment-name>
```

```
microk8s kubectl delete service <service-name>
```