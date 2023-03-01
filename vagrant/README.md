# Setting up the Lab using Vagrant #
## Vagrantfile ##
1. Open the **Vagrantfile** in the **vagrant** directory using any IDE/text editor.
2. As you can see, this project offers you to use **Virtualbox** for setting up your lab. You can re-write this file to use **Docker** as your provisioner instead. 

To use **Docker** instead, you should find the docker-friendly box (*tknerr/baseimage-ubuntu-18.04*, for example), change the IP addresses to your **Docker Subnet** range and change the loop syntax to:
```
    servers.each do |machine|
        config.vm.define machine[:hostname] do |node|
            node.vm.provider "docker" do |d|
            d.image = machine[:image]
            node.vm.hostname = machine[:hostname]
            node.vm.network :private_network, ip: machine[:ip]
            node.vm.network "forwarded_port", guest: 22, host: machine[:ssh_port], id: "ssh"
            end
        end
    end
```
3. If VirtualBox is okay for you, before using the provided Vagrantfile, make sure to change the provided IP addresses to the one of your available **VirtualBox subnets** (*Tools > Network Manager*). 
4. Also, change the last 3 IP addresses in the **hosts** file. 
5. Open *Command prompt* and cd to the **vagrant** directory.
6. Run the **vagrant up** command to create your VMs. 

If there are no errors, you should see 3 new running Ubuntu machines in your VirtualBox.

*Note*: If you have used the Vagrant with other provisioner before, it may be required to run the **vagrant up --provider virtualbox** command instead.
## Setting up the SSH connection for Control machine ##

1. After VMs creation, run the **vagrant ssh control** command to connect to your control machine.
2. cd to the *root* directory and copy our hosts file to */etc/hosts*:
```
cd /
sudo cp /vagrant/hosts /etc/hosts
```
3. Add ssh keys to your nodes to make sure *Ansible* will be able to do configurations without prompting password.

* Generate key pair:
```
ssh-keygen
```
* Then push our keys to the nodes:
```
ssh-copy-id node1 && ssh-copy-id node1
```

Now we are ready to install **Ansible** on our control machine. Click [here](ansible/README.md) to open the next instruction. 


