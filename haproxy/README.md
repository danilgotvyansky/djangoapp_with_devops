# HAProxy Load Balancer #

We have such a logic here:
* The [configuration file](haproxy.cfg) defines a *frontend* section that listens to incoming HTTP traffic on port 80 (bind 192.168.6.5:80) and checks if the hostname in the HTTP header is *local.burava.com* (acl is_app hdr(host) -i local.burava.com). 
* If the hostname matches, the traffic is routed to the *backend* section backendnodes using the *use_backend* keyword.
* The *backend* section **backendnodes** defines the application servers that should receive the traffic. 
* The *server* lines define the IP addresses and ports of the servers, and the *check* keyword ensures that HAProxy regularly checks if the servers are still available.

If the primary server *192.168.6.3:8000* is down, HAProxy will automatically route the traffic to the backup server *192.168.6.4:8000* because of the *backup* keyword.

### Installation ###

1. Run these commands on the **haproxy** server to install HAProxy:
```
sudo apt-get update
sudo apt-get install haproxy
```

2. Edit the **configuration file** with the value of [haproxy.cfg](haproxy.cfg) file:
```
sudo nano /etc/haproxy/haproxy.cfg
```

NOTE: Make sure to edit all IP addresses with the ones corresponding to your servers used in a [Vagrantfile](..vagrant/Vagrantfile).

3. Make sure to restart HAProxy after the changes:
```
sudo service haproxy restart
sudo service haproxy status
```

4. Edit the **hosts** file on your Local PC with this line in the end:
```
192.168.6.5 local.burava.com
```
5. Check if you are able to open the application on *http://local.burava.com*.
