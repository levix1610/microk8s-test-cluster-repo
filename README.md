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

Next I need to get back to trying the homarr config.  Luckily I got heimdall deployment working and can begin using that as a test Kubernetes environment. I might need to look into working with all my other deployments. Now that I got the NFS share set to map user as admin, my other files in the NFS location are still set to my kubernetes user on the cluster for ownership and group.  I might need to add the runAsGroup/User to other deployments in the future.

Well the deployment of the postgreSQL deployment and homarr did deploy with minimal fixes; they are not functioning together as intended.  I still get the 500 Internal Server Error well hitting the nodeport of homarr.  Will need to go back to drawing board as this issue is definetly beyond my current abilities.  Starting to reach a self ceiling and will need to reach out to more advance help.

I moved beyond just working this out on my own and finally signed up for boot.dev website to learn more backend deployment.  Though it focuses more on coding than kubernetes I am working through its course as well along with the beginning python course.  Its not the most comprehensive kuberentes course but I have picked up some things. 

I tried to work on the postgres Database along with homarr.  I deployed the postgres deployment again and ran some tests on the Pod to make sure it deployed correctly and the port was open and listening on the user created for homarr.  After the deployment went successfully I ran a temporary busybox image and ssh into the container and found that the port was open and lisening.  I continued to deploy the homarr deployment and it still failed with the 500 error. 

I did some more research and found the answer in the homarr deployment with a secret encryption key. Homarr required having a 64-character hex string added within the deployment to use to encrypt data from homarr itself; along with data a user might add to the dashboard. I created a new secrets yaml for the homarr deployment and referenced that within the main homarr deployment as is standard practice for kubernetes.  This worked and the deployment finally worked and homarr is up and running!

It looks like one aspect was overseen.  It turns out the volume mount in the homarr deployment was not what it expected.  So the data was not storing to the NFS share and the configuration did not survive deletion.  Turns out, base on homarr's documentation, it stores all its data at    /appdata.  After that change everything is 'officially' working and the homarr deployment keeps configuration even after deletion.  Also, after doing some more research on the deployment type I will in the future create another Database deployment for each application, as that is standard practice for isolation and easier upgradability in the future.  As well as, creating more namespaces for each one of these kind of deployments.  To better isolate the services from the rest of the cluster. Next, I am going to starting to look into trying to get Pihole to work along with the metallb serivce so I can properly loadbalance them and get a direct IP address for them.  

After tinkering around with k3s for about two weeks I learned some more thing and worked a little on streamlining some aspects of my code.  Recopying over to this repo with some updated and then modifications over to a new MicroK8s cluster.  Going to rebuild the whole cluster this time in my test network using the same TrueNas VM this time for the NFS target.  This way I can better set it up for a more true test environment.  Due to limitations of the ServiceLB loadbalancers in k3s I am going to give a MicroK8s cluster another go as it has MetalLB baked in as a service.  Which overall works better for bare metal on prem installs. 

On a first note I was able to get the metalLB services running correctly and working.  The few deployments I have now work with getting to them with a direct IP address with no addition of ports.  This will make DNS assigning much easier in the near future.  Going forward I am going to just use an external address to get to my services and cut out nodeports as they were good for beginning testing but not a end all solution.

My latest deployment was so far successful after some interesting problems that arose. My Pihole deployment is up and running and accessable. However, I ran into an issue where I could not exec into the pod due to a TLS issue.  Turns out installing Microk8s at the OS install level it put the original DHCP address into the 'acceptable' IP addresses for the tls connection into a pod.  After installation I assigned static IPs at the network level for all the nodes in the cluster; so it changed for the original DHCP address.  This caused a major issue as I could not get into the pod to change the login password for the pihole deployment. I found out I had to drain the node, remove it from the cluster, run a microk8s reset command, then rejoin it to the cluster.  Fixing all the cert issues the node was having; which allowed me to finally exec into the pods container.  Next, I am going to start using this pihole deployment as the DNS server for my test vlan and see how it handles it and make sure that DNS/53 port is working reliably.  
What I learned from this was to either make sure the server has the IP you want as install or hold off on installing microk8s until the server DOES have the desinated IP you want.  

I just had to make one minor adjustment to the mountPath for the volume on pihole.  As I followed the path pihole had in its documentations but that made the data get stored in the active Pod; not on the persistant NFS share.  Making it /etc/pihole works and all the data is now populated on the share and I gave the Pods a few deletes and everything stayed as configured. 