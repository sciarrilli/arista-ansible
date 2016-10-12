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
        device.vm.provision :shell , inline: "sudo chown vagrant:vagrant eos-ansible-quick-start/ -R"


  #        device.vm.provision :shell , inline: "sudo apt-get install software-properties-common git -y"
  #        device.vm.provision :shell , inline: "sudo apt-add-repository ppa:ansible/ansible -y"
  #        device.vm.provision :shell , inline: "sudo apt-get update"
  #        device.vm.provision :shell , inline: "sudo apt-get install ansible -qy"
  #        device.vm.provision :shell , inline: "git clone https://github.com/sciarrilli/cware-wbench.git"
  #        device.vm.provision :shell , inline: "ansible-playbook cware-wbench/wbench.yml --connection=local -i localhost,"


  #      device.vm.provision "ansible" do |ansible|
  #        ansible.playbook = "./wbench/wbench.yml"

      end


#spine01
  config.vm.define "spine01" do |device|
    device.vm.provider "virtualbox" do |v|
      v.name = "spine01"
    end
    device.vm.box = "vEOS4166M"
    device.vm.network "private_network", virtualbox__intnet: "s1l1", ip: "10.1.1.21", auto_config: false
    device.vm.network "private_network", virtualbox__intnet: "s1l2", ip: "10.1.2.21", auto_config: false

    device.vm.provision "shell", inline: <<-SHELL
      sleep 30
      FastCli -p 15 -c "configure
      hostname spine01
      interface Management1
        ip address 192.168.0.21/24 secondary
      interface Ethernet1
        no switchport
        ip address 10.1.1.21/24
      interface Ethernet2
        no switchport
        ip address 10.1.2.21/24"
    SHELL

    # remap nic1 for mgmt internal network
      config.trigger.after :up do
        info "Remapping nic1"
        run "VBoxManage controlvm spine01 nic1 intnet net_mgmt"
      end

    # clean up files on the host after the guest is destroyed
      config.trigger.after :destroy do
        run "rm -rf '/Users/Nick/VirtualBox VMs/spine01/'"
      end
  end

#spine02
  config.vm.define "spine02" do |device|
    device.vm.provider "virtualbox" do |v|
      v.name = "spine02"
    end
    device.vm.box = "vEOS4166M"
    device.vm.network "private_network", virtualbox__intnet: "s2l1", ip: "10.2.1.22", auto_config: false
    device.vm.network "private_network", virtualbox__intnet: "s2l2", ip: "10.2.2.22", auto_config: false

    device.vm.provision "shell", inline: <<-SHELL
      sleep 30
      FastCli -p 15 -c "configure
      hostname spine02
      interface Management1
        ip address 192.168.0.22/24 secondary
      interface Ethernet1
        no switchport
        ip address 10.2.1.22/24
      interface Ethernet2
        no switchport
        ip address 10.2.2.22/24"
    SHELL

    # remap nic1 for mgmt internal network
      config.trigger.after :up do
        info "Remapping nic1"
        run "VBoxManage controlvm spine02 nic1 intnet net_mgmt"
      end

    # clean up files on the host after the guest is destroyed
      config.trigger.after :destroy do
        run "rm -rf '/Users/Nick/VirtualBox VMs/spine02/'"
      end
  end



#leaf01
  config.vm.define "leaf01" do |device|
    device.vm.provider "virtualbox" do |v|
      v.name = "leaf01"
    end
    device.vm.box = "vEOS4166M"
    device.vm.network "private_network", virtualbox__intnet: "s1l1", ip: "10.1.1.11", auto_config: false
    device.vm.network "private_network", virtualbox__intnet: "s2l1", ip: "10.2.1.11", auto_config: false

    device.vm.provision "shell", inline: <<-SHELL
      sleep 30
      FastCli -p 15 -c "configure
      hostname leaf01
      interface Management1
        ip address 192.168.0.11/24 secondary
      interface Ethernet1
        no switchport
        ip address 10.1.1.11/24
      interface Ethernet2
        no switchport
        ip address 10.2.1.11/24"
    SHELL

    # remap nic1 for mgmt internal network
      config.trigger.after :up do
        info "Remapping nic1"
        run "VBoxManage controlvm leaf01 nic1 intnet net_mgmt"
      end

    # clean up files on the host after the guest is destroyed
      config.trigger.after :destroy do
        run "rm -rf '/Users/Nick/VirtualBox VMs/leaf01/'"
      end
   end


#leaf02
  config.vm.define "leaf02" do |device|
     device.vm.provider "virtualbox" do |v|
      v.name = "leaf02"
    end
    device.vm.box = "vEOS4166M"
    device.vm.network "private_network", virtualbox__intnet: "s1l2", ip: "10.1.2.12", auto_config: false
    device.vm.network "private_network", virtualbox__intnet: "s2l2", ip: "10.2.2.12", auto_config: false

    device.vm.provision "shell", inline: <<-SHELL
      sleep 30
      FastCli -p 15 -c "configure
      hostname leaf02
      interface Management1
      ip address 192.168.0.12/24 secondary
      interface Ethernet1
        no switchport
        ip address 10.1.2.12/24
      interface Ethernet2
        no switchport
        ip address 10.2.2.12/24"
    SHELL

   # remap nic1 for mgmt internal network
     config.trigger.after :up do
       info "Remapping nic1"
       run "VBoxManage controlvm leaf02 nic1 intnet net_mgmt"
      end

    # clean up files on the host after the guest is destroyed
     config.trigger.after :destroy do
        run "rm -rf '/Users/Nick/VirtualBox VMs/leaf02/'"
     end
  end





end
