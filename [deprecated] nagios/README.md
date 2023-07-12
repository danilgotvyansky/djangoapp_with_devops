# DEPRECATED* #

Please note that the Nagios is no longer used for this project. [Prometheus](/prometheus/README.md) is used instead.

# Nagios #
Nagios is used to monitor both servers' performance. Also, it will monitor our application demo and notify us when there is any issue.

### Installation ###
Follow the steps from the [Ansible tutorial](../vagrant/ansible/README.md) to install **Nagios Core** on the control server.

### Configure Monitoring ###
1. To run some specific checks on our *nodes* we should have the *Nagios plugins* installed on both remote servers:
```
sudo apt update
sudo apt-get install -y autoconf gcc libc6 libmcrypt-dev make libssl-dev wget bc gawk dc build-essential snmp libnet-snmp-perl gettext 
sudo apt install nagios-plugins
```
2. On the *control* server uncomment this line in *usr/local/nagios/etc/nagios.cfg* file:
```
cfg_dir=/usr/local/nagios/etc/servers
```
3. Create the */usr/local/nagios/etc/servers* directory.
4. Set your email (in our case - **monitor@burava.com**) in the **/usr/local/nagios/etc/objects/contacts.cfg** file >> **define contact { }**:
```
sudo nano /usr/local/nagios/etc/objects/contacts.cfg
```
```
define contact {

    contact_name            nagiosadmin             ; Short name of user
    use                     generic-contact         ; Inherit default values from generic-contact template (defined above)
    alias                   Nagios Admin            ; Full name of user
    email                   monitor@burava.com ;    <<***** CHANGE THIS TO YOUR EMAIL ADDRESS ******
}
```
Later, I will configure the **email client** on the *control* server to make the notifications work properly. 

5. Replace the contents of the **/usr/local/nagios/etc/objects/commands.cfg** file with the ones from this [file](commands.cfg).
6. Create the following files in the created **servers** directory and fill their contents accordingly:
   * [node1.cfg](node1.cfg)
   * [node2.cfg](node2.cfg)
   * [demoapp.cfg](demoapp.cfg)
7. Create the **SSH key** for the **nagios** user and push its id to the both nodes:

```
sudo su - nagios
chmod 700 ~/.ssh
chmod 600 ~/.ssh/*
ssh-keygen
```
Before copying the ssh id to the nodes make sure to connect via ssh to both nodes to add them to the known hosts and type **"yes"** when prompted.

```
ssh -p 21098 vagrant@node1 && ssh -p 21099 vagrant@node2
ssh-copy-id vagrant@node1 && ssh-copy-id vagrant@node2
```

Check if you are able to log in to the **nodes** via ssh using **nagios** user without prompting a password:

```
ssh vagrant@node1
```
8. Restart the **Nagios** service:
```
sudo systemctl restart nagios
```
9. Check your **Nagios Core GUI** on the *http://<your-control-ip>/nagios* link and check the monitoring services.

You should have the following services running:
* **demoapp**
  * Check App
  * PING
* **node1**
  * Check App
  * MariaDB Status
  * PING
* **node2**
  * Check App
  * MariaDB Status
  * PING
