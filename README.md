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

Turns out, as stated on line 17, more configuration was needed to properly use a NFS share.  Defining the PV and PVC is not enough for most deployments it seems.  Environmental variables need to be defined so the container in the POD understand where to store the data to have true proper peristance.  So, if a pod needs to be resheduled to another node, its true configuration and data persist.  

I was able to get jellyfin working by seeing I can mount other NFS volumes within the deployment config file.  This mapped the share perfectly within the Container of the Pod, allowing us to use it as a media folder. 

Had some issues with heimdalls deployment as well as it did not work at all.  Turns out two issues were present.  First it was listening on port 80 well i was trying to use 8099.  I needed to set it to 80/80 for it to work right under http.  Along with that, the mount path needed to be /config as its just how the container is defined to use it.  /app/data was not a path is was expecting.  

InfluxDB also was causing deployment issues.  The config environmental variables needed to be defined. But, since I have the NFS mount set as map users to admin the container was failing to deploy in the pod.  Based on permissions the container was trying to run it as some other user well the NFS share is just trying to map it to admin.  In order to fix I needed to either change the NFS share to no mapping or configure the POD to work as admin user.  Since changing the sqaush mapping broke my other deployments I reverted that change and tried the users settings.  By adding securityContext:runAsUser: 1024  runAsGroup: 1024 this fixed the permission issues the deployment was having. 

Next I need to get back to trying the homarr config.  Luckily I got heimdall deployment working and can begin using that as a test Kubernetes environment. 