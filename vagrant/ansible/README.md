# Install Ansible #
1. To install **Ansible** on our control1 machine run this command:
```
sudo apt-get install ansible -y
```
2. Firstly, let's check if **Ansible** is able to do tasks locally and remotely:

Locally:
```
cd ../vagrant/ansible
ansible control -i myhosts -c local -m command -a hostname
```
Remotely:
```
ansible nodes -i myhosts -m command -a hostname
```

## Install Docker using Ansible ##
1. Run the [installation_docker.yml](installation_docker.yml) playbook remotely to install Docker on our nodes:
```
ansible-playbook -i myhosts -K installation_docker.yml
```

# Deprecated #

## Install Nagios using Ansible ##
This part will install **Nagios** on the control machine to monitor our future application project.
1. As we are connected to the **control** machine now, start the [installation_nagios.yml](installation_nagios.yml) playbook locally:
```
ansible-playbook installation_nagios.yml -i myhosts -c local
```
The installation process has been separated in two different playbooks to make possible the secure *Nagios user password* setting up. 

3. So now run this command to set your *Nagios admin user password*:
```
sudo htpasswd -c /usr/local/nagios/etc/htpasswd.users nagiosadmin
```
4. After that, we can run the second installation part [start_nagios.yml](installation_nagios.yml) that will actually start your Nagios:
```
ansible-playbook start_nagios.yml -i myhosts -c local
```

5. After that cd to the **/tmp/nagios-plugins-release-2.3.3** and run:
```
sudo make
sudo make install
```

**Note:** If you have Ansible installed on your local device, you can run both playbooks from it and that won't require the *-c local* parameter, however, it will be still required to connect to the control machine to set the *Nagios Admin user* password.

5. To test if **Nagios** is working, open *http://{{your control machine IP address}}/nagios* in Web browser.

If installation has been processed with errors, you can follow this guide to install it manually:
[https://www.linuxfordevices.com/tutorials/ubuntu/install-nagios-on-ubuntu](https://www.linuxfordevices.com/tutorials/ubuntu/install-nagios-on-ubuntu)
