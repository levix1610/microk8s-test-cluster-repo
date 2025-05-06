my-deployment-files repo:  

This is my first repository for trying to learn Git and create yaml files for a kubernetes deployment.  

This is very early days and a lot of this is experimental and testing.  

I have currently deployed a 3-node kubernetes cluster using microk8s and ceph storage on ubuntu server 24.04 on my proxmox cluster.  

Working on trying to learn and implement standard practices with file sturctures and conventions.  

Currently these deployments are being created WITH the help of AI resources; so review still needs to be done as well as taking discretionary caution.  

Plans on starting a formal Class to better understand DevOps best practices I will be starting in the coming weeks. 

Had many failures with the process at creating the ceph cluster.  Found another way to do it but is FAR from production ready.  However, I think, is a good solid first attempt at a cluster.

Finally, I have a good structure in how to make a full deployment using a remote NFS share on my NAS.  And, how to provision such storage to be used by the kubernetes cluster without the host knowing about the NFS share.

This will make it the easiest and most repeatable ability to deploy a full app across future nodes.

Next will need to move onto figuring out how to work with Databases Within a POD.  As the homarr deployment, and the heimdall one, both seem to not work as they need a formal DB to work from, not just storage. 

Having issues with trying to deploy helm chart of homarr. Not understanding how to define the internal databases.  However, I am going to try to move to a better method; by deploying another POD for a PostgreSQL to use instead.  This is also a better practice to have stateFull and stateLess applications separate.  