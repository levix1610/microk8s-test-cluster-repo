### Keepalived

Keepalived is a virtual router service allowing you to communicate directly with the cluster under one unified IP address.  Granted this is NOT a great way to have a Kubernetes cluster work I am for the time going to implement this for the time being until i get a better grasp on how to load balance and proxy a Kubernetes cluster properly.  

First, start with installing the keepalived package on all node in the cluster.

```
sudo apt install keepalived -y
```

Next, you will need to go to this config file on each node and add the virtual instance on each node.  See below for the config for the respective host:

```
sudo nano /etc/keepalived/keepalived.conf
```

Config Files info needed, Config for PRIMARY HOST.    
Make sure the 'ens'192' under interface matches the name of your hosts interface by doing an 'ip addr.' Additionally, the auth_pass can be whatever you want.  Just make sure it is consistent across the configs. Make sure to double check the IP addresses are correct as your adding to the config.  virtual_ipaddress at the bottom is the IP that will work with the whole cluster once configured. 

```
vrrp_instance VI_1 {

    state MASTER

    interface ens192

    virtual_router_id 51

    priority 120

    advert_int 1

    authentication {

        auth_type PASS

        auth_pass Letmein1$

    }

    unicast_peer {

    10.0.100.42

    10.0.100.43

    }

    virtual_ipaddress {

        10.0.100.40

    }

}
```

Config for SECONDARY HOST.  
make sure to change interface and to drop the priority a little from the primary host.  So it always knows which one is the overall primary node at the time:

```
vrrp_instance VI_1 {
    state BACKUP

    interface ens192

    virtual_router_id 51

    priority 110

    advert_int 1

    authentication {

        auth_type PASS

        auth_pass Letmein1$

    }

    unicast_peer {

    10.0.100.41

    10.0.100.43

    }

    virtual_ipaddress {

        10.0.100.40

    }

}
```

Config for THIRD HOST.  
Same as before now in the last two configs:

```
vrrp_instance VI_1 {
    state BACKUP

    interface ens192

    virtual_router_id 51

    priority 100

    advert_int 1

    authentication {

        auth_type PASS

        auth_pass Letmein1$

    }

    unicast_peer {

    10.0.100.41

    10.0.100.42

    }

    virtual_ipaddress {

        10.0.100.40

    }

}
```

Next will need to run commands to make sure the keepalived services starts at boot.

run following command on all 3 nodes; will start and enable the service to start at boot:

```
sudo systemctl start keepalived
```

```
sudo systemctl enable keepalived
```

then check the status of keepalived:

```
sudo systemctl status keepalived
```

Try pinging the virtual IP chosen for the cluster and see if it responds.

DEPRECIATED SINCE METALLB IS WORKING.