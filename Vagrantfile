# -*- mode: ruby -*-
# vi: set ft=ruby :

# All Vagrant configuration is done below. The "2" in Vagrant.configure
# configures the configuration version (we support older styles for
# backwards compatibility). Please don't change it unless you know what
# you're doing.
Vagrant.configure(2) do |config|
  config.vm.provider "virtualbox" do |v|
    v.gui=false
  end

  config.vm.define "workbench" do |device|
      device.vm.provider "virtualbox" do |v|
        v.name = "workbench"
        v.customize ["modifyvm", :id, '--audiocontroller', 'AC97', '--audio', 'Null']
        v.memory = 1024
      end
      device.vm.hostname = "workbench"
      device.vm.box = "boxcutter/ubuntu1404"

      device.vm.synced_folder ".", "/vagrant", disabled: true
      # UBUNTU DEVICES ONLY: Shorten Boot Process - remove \"Wait for Network
      device.vm.provision :shell , inline: "sudo sed -i 's/sleep [0-9]*/sleep 1/' /etc/init/failsafe.conf"

      device.vm.network "private_network", virtualbox__intnet: "net_mgmt", auto_config: false
      device.vm.provider "virtualbox" do |vbox|
        vbox.customize ['modifyvm', :id, '--nicpromisc2', 'allow-vms']
        vbox.customize ["modifyvm", :id, "--nictype1", "virtio"]
      end

        # Fixes "stdin: is not a tty" message --> https://github.com/mitchellh/vagrant/issues/1673
        device.vm.provision :shell , inline: "(grep -q -E '^mesg n$' /root/.profile && sed -i 's/^mesg n$/tty -s \\&\\& mesg n/g' /root/.profile && echo 'Ignore the previous error \"stdin: is not a tty\" -- fixing this now...') || exit 0;"

        # Run Any Extra Config
        device.vm.provision :shell , path: "./helper_scripts/config_oob_server.sh"

        device.vm.provision :shell , inline: "sudo apt-get update"
        device.vm.provision :shell , inline: "sudo apt-get install -y python-setuptools python-dev build-essential libssl-dev libffi-dev"
        device.vm.provision :shell , inline: "sudo easy_install pip"
        device.vm.provision :shell , inline: "sudo pip install --upgrade pip"
        device.vm.provision :shell , inline: "sudo pip install paramiko PyYAML Jinja2 httplib2 six"
        device.vm.provision :shell , inline: "sudo pip install ansible"
        device.vm.provision :shell , inline: "sudo apt-get install -y iputils-ping"
        device.vm.provision :shell , inline: "sudo apt-get install -y net-tools"
        device.vm.provision :shell , inline: "sudo apt-get install -y tcpdump"
        device.vm.provision :shell , inline: "sudo apt-get install -y vim git"
        device.vm.provision :shell , inline: "git clone https://github.com/arista-eosplus/eos-ansible-quick-start.git"


#        device.vm.provision :shell , inline: "sudo apt-get install software-properties-common git -y"
#        device.vm.provision :shell , inline: "sudo apt-add-repository ppa:ansible/ansible -y"
#        device.vm.provision :shell , inline: "sudo apt-get update"
#        device.vm.provision :shell , inline: "sudo apt-get install ansible -qy"
#        device.vm.provision :shell , inline: "git clone https://github.com/sciarrilli/cware-wbench.git"
#        device.vm.provision :shell , inline: "ansible-playbook cware-wbench/wbench.yml --connection=local -i localhost,"


#      device.vm.provision "ansible" do |ansible|
#        ansible.playbook = "./wbench/wbench.yml"

      end


#
# spine01
#
  config.vm.define "spine01" do |spine01|
    spine01.vm.provider "virtualbox" do |v|
      v.memory = "1536"
    end

    spine01.vm.box = "vEOS4166M"
    spine01.vm.network "private_network", virtualbox__intnet: "s1l1", ip: "10.0.0.2", auto_config: false
    spine01.vm.network "private_network", virtualbox__intnet: "s1l2", ip: "10.2.0.2", auto_config: false
    spine01.vm.provision "shell", inline: <<-SHELL
      sleep 30
      FastCli -p 15 -c "configure
      interface Management1
        ip address 192.168.0.21/24"
	  interface Ethernet1
        ip address 10.0.0.2/32"
	  interface Ethernet2
        ip address 10.2.0.2/32"
    SHELL
  end

#
# spine02
#
  config.vm.define "spine02" do |spine02|
    spine02.vm.provider "virtualbox" do |v|
      v.memory = "1536"
    end
    spine02.vm.box = "vEOS4166M"
    spine02.vm.network "private_network", virtualbox__intnet: "s2l1", ip: "10.1.0.3", auto_config: false
    spine02.vm.network "private_network", virtualbox__intnet: "s2l2", ip: "10.3.0.3", auto_config: false

    spine02.vm.provision "shell", inline: <<-SHELL
      sleep 30
      FastCli -p 15 -c "configure
      interface Management1
        ip address 10.132.0.6/24"
	  interface Ethernet1
        ip address 10.1.0.2/32"
	  interface Ethernet2
        ip address 10.3.0.2/32"
    SHELL
  end

#
# leaf01
#
  config.vm.define "leaf01" do |leaf01|
    leaf01.vm.provider "virtualbox" do |v|
      v.memory = "1536"
    end
    leaf01.vm.box = "vEOS4166M"
    leaf01.vm.network "private_network", virtualbox__intnet: "s1l1", ip: "10.0.0.3", auto_config: false
    leaf01.vm.network "private_network", virtualbox__intnet: "s2l1", ip: "10.1.0.3", auto_config: false

    leaf01.vm.provision "shell", inline: <<-SHELL
      sleep 30
      FastCli -p 15 -c "configure
      interface Management1
        ip address 10.132.0.9/24"
	  interface Ethernet1
        ip address 10.0.0.1/32"
	  interface Ethernet2
        ip address 10.1.0.1/32"
    SHELL
  end

#
# leaf02
#
  config.vm.define "leaf02" do |leaf02|
    leaf02.vm.provider "virtualbox" do |v|
      v.memory = "1536"
    end
    leaf02.vm.box = "vEOS4166M"
    leaf02.vm.network "private_network", virtualbox__intnet: "s1l2", ip: "10.2.0.3", auto_config: false
    leaf02.vm.network "private_network", virtualbox__intnet: "s2l2", ip: "10.3.0.3", auto_config: false

    leaf02.vm.provision "shell", inline: <<-SHELL
      sleep 30
      FastCli -p 15 -c "configure
      interface Management1
        ip address 10.132.0.7/24"
	  interface Ethernet1
        ip address 10.2.0.1/32"
	  interface Ethernet2
        ip address 10.3.0.1/32"
    SHELL
  end

end
