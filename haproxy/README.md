# HAProxy Load Balancer #

We have such a logic here:
* HAProxy is launched inside the **Docker container**.
* The [configuration file](haproxy.cfg) defines a *frontend* section that listens to incoming HTTP traffic on port 80 (bind *:80) and checks if the hostname in the HTTP header is *local.burava.com* (acl is_app hdr(host) -i local.burava.com). 
* If the hostname matches, the traffic is routed to the *backend* section backendnodes using the *use_backend* keyword.
* The *backend* section **backendnodes** defines the application servers that should receive the traffic. 
* The *server* lines define the IP addresses and ports of the servers, and the *check* keyword ensures that HAProxy regularly checks if the servers are still available.

If the primary server *192.168.6.3:8000* is down, HAProxy will automatically route the traffic to the backup server *192.168.6.4:8000* because of the *backup* keyword.

### Installation ###

1. Save the contents of the [haproxy.cfg](haproxy.cfg) on the **haproxy** server:
```commandline
nano haproxy.cfg
```

NOTE: Make sure to edit all IP addresses with the ones corresponding to your servers used in a [Vagrantfile](..vagrant/Vagrantfile).

2. Launch HAProxy inside the Docker container with the following command:
```commandline
sudo docker run -d --privileged --user root -p 1935:1935 -p 80:80 -v $(pwd)/haproxy.cfg:/usr/local/etc/haproxy/haproxy.cfg -v /run/haproxy:/run/haproxy --name haproxy haproxy:latest
```

Let's review the options and parameters of the command above:
* `--privileged` flag: to run the container in privileged mode, which gives it elevated privileges.
* `--user root` flag: to run the container as the root user. Since the container needs root privileges to bind to privileged ports (like port 1935), this ensures that the container has the necessary permissions.
* `-v $(pwd)/haproxy.cfg:/usr/local/etc/haproxy/haproxy.cfg` flag: to mount the [haproxy.cfg](haproxy.cfg) file from the current directory ($(pwd)) into the container's file system at the specified path. This allows you to provide a custom HAProxy configuration file for the container.
* `-v /run/haproxy:/run/haproxy` flag: to mount the /run/haproxy directory from the host to the container. This ensures that HAProxy has access to the necessary runtime files and sockets.

3. Edit the **hosts** file on your Local PC with this line in the end:
```ini
192.168.6.5 local.burava.com
```

4. Check if you are able to open the application on *http://local.burava.com*.
