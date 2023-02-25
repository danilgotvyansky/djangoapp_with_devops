Vagrant.configure("2") do |config|
    servers=[
        {
          :hostname => "control",
          :box => "bento/ubuntu-18.04",
          :ip => "192.168.6.2",
          :ssh_port => '21097'
        },
        {
          :hostname => "nodeapp",
          :box => "bento/ubuntu-18.04",
          :ip => "192.168.6.3",
          :ssh_port => '21098'
        },
        {
          :hostname => "nodedb",
          :box => "bento/ubuntu-18.04",
          :ip => "192.168.6.4",
          :ssh_port => '21099'
        }
      ]

    servers.each do |machine|
        config.vm.define machine[:hostname] do |node|
            node.vm.box = machine[:box]
            node.vm.hostname = machine[:hostname]
            node.vm.network :private_network, ip: machine[:ip]
            node.vm.network "forwarded_port", guest: 22, host: machine[:ssh_port], id: "ssh"
            node.vm.provider :virtualbox do |vb|
                vb.customize ["modifyvm", :id, "--memory", 512]
                vb.customize ["modifyvm", :id, "--cpus", 1]
            end
        end
    end
end