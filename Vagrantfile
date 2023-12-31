# -*- mode: ruby -*-
# vi: set ft=ruby :
#
Vagrant.require_version ">= 1.6.0"

# All Vagrant configuration is done below. The "2" in Vagrant.configure
# configures the configuration version (we support older styles for
# backwards compatibility). Please don't change it unless you know what
# you're doing.


boxes = [
    {
        :name => "k8s-master",
        :eth1 => "192.168.205.10",
        :mem => "1024",
        :cpu => "1"
    },
    {
        :name => "k8s-worker1",
        :eth1 => "192.168.205.11",
        :mem => "1024",
        :cpu => "1"
    },
    # {
    #     :name => "k8s-worker2",
    #     :eth1 => "192.168.205.12",
    #     :mem => "1024",
    #     :cpu => "1"
    # }
]

Vagrant.configure(2) do |config|
  # The most common configuration options are documented and commented below.
  # For a complete reference, please see the online documentation at
  # https://docs.vagrantup.com.

  # Every Vagrant development environment requires a box. You can search for
  # boxes at https://atlas.hashicorp.com/search.
  config.vm.box = "ubuntu/focal64"

  boxes.each do |opts|
      config.vm.define opts[:name] do |config|
        config.vm.hostname = opts[:name]

        config.vm.provider "vmware_fusion" do |v|
          v.vmx["memsize"] = opts[:mem]
          v.vmx["numvcpus"] = opts[:cpu]
        end

        config.vm.provider "virtualbox" do |v|
          v.customize ["modifyvm", :id, "--memory", opts[:mem]]
          v.customize ["modifyvm", :id, "--cpus", opts[:cpu]]
        end
        # config.vm.boot_timeout = 1200
        config.vm.network :private_network, ip: opts[:eth1]
        # config.vm.network "public_network", bridge: "Hyper-V Virtual Ethernet Adapter #2"
        # config.vm.network "public_network", ip: opts[:eth1]
        config.vm.provision :shell, inline: <<-SHELL
        echo "KUBELET_EXTRA_ARGS='--node-ip #{opts[:eth1]}'" | sudo tee -a /etc/default/kubelet
        # echo "nameserver 10.96.0.10 8.8.8.8 2001:4860:48608888" | sudo tee -a /etc/resolv.conf
        sudo systemctl restart kubelet
      SHELL

      end
  end
  # Disable automatic box update checking. If you disable this, then
  # boxes will only be checked for updates when the user runs
  # `vagrant box outdated`. This is not recommended.
  # config.vm.box_check_update = false

  # Create a forwarded port mapping which allows access to a specific port
  # within the machine from a port on the host machine. In the example below,
  # accessing "localhost:8080" will access port 80 on the guest machine.
  # config.vm.network "forwarded_port", guest: 80, host: 8080

  # Create a private network, which allows host-only access to the machine
  # using a specific IP.
  # config.vm.network "private_network", ip: "192.168.33.10"

  # Create a public network, which generally matched to bridged network.
  # Bridged networks make the machine appear as another physical device on
  # your network.
  # config.vm.network "public_network"

  # Share an additional folder to the guest VM. The first argument is
  # the path on the host to the actual folder. The second argument is
  # the path on the guest to mount the folder. And the optional third
  # argument is a set of non-required options.
  # config.vm.synced_folder "../data", "/vagrant_data"

  # Provider-specific configuration so you can fine-tune various
  # backing providers for Vagrant. These expose provider-specific options.
  # Example for VirtualBox:
  #
  # config.vm.provider "virtualbox" do |vb|
  #   # Display the VirtualBox GUI when booting the machine
  #   vb.gui = true
  #
  #   # Customize the amount of memory on the VM:
  #   vb.memory = "1024"
  # end
  #
  # View the documentation for the provider you are using for more
  # information on available options.

  # Define a Vagrant Push strategy for pushing to Atlas. Other push strategies
  # such as FTP and Heroku are also available. See the documentation at
  # https://docs.vagrantup.com/v2/push/atlas.html for more information.
  # config.push.define "atlas" do |push|
  #   push.app = "YOUR_ATLAS_USERNAME/YOUR_APPLICATION_NAME"
  # end

  config.vm.provision "file", source: "weave_daemonset-k8s.yaml", destination: "~/weave_daemonset-k8s.yaml"
  config.vm.provision "shell", inline: "sudo mkdir -p /run/flannel"
  config.vm.provision "file", source: "subnet.env", destination: "~/subnet.env"
  config.vm.provision "file", source: "kube-flannel_change_interface.yml", destination: "~/kube-flannel_change_interface.yml"
  # Enable provisioning with a shell script. Additional provisioners such as
  # Puppet, Chef, Ansible, Salt, and Docker are also available. Please see the
  # documentation for more information about their specific syntax and use.
  config.vm.provision "shell", inline: <<-SHELL
    curl -s https://packages.cloud.google.com/apt/doc/apt-key.gpg | sudo apt-key add -
    echo "deb http://apt.kubernetes.io/ kubernetes-xenial main" >> ~/kubernetes.list
    sudo mv ~/kubernetes.list /etc/apt/sources.list.d
    sudo apt-get update
    # Install docker if you don't have it already.
    sudo apt-get install -y docker.io
    apt-get install -y kubelet kubeadm kubectl kubernetes-cni net-tools
  SHELL
  config.vm.provision "shell", inline: "sudo cp /home/vagrant/subnet.env /run/flannel/subnet.env"

end
