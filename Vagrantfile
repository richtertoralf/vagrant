BOX_IMAGE = "ubuntu/jammy64"
NODE_COUNT = 3

Vagrant.configure("2") do |config|
  config.vm.box = BOX_IMAGE

  config.vm.provider "virtualbox" do |vb|
    vb.memory = "1024"
    vb.cpus = 1
  end

  config.vm.define "ansible_host" do |anshost|
    anshost.vm.network "private_network", ip: "192.168.50.100", virtualbox__intnet: "mynetwork"
    anshost.vm.hostname = "ansiblehost"
    anshost.vm.provider "virtualbox" do |vm|
      vm.name = "ansiblehost"
    end
    anshost.vm.provision "shell", inline: <<-SHELL
      apt update && apt upgrade -y
      apt install -y ansible
    SHELL
  end

  (1..NODE_COUNT).each do |i|
    config.vm.define "machine_#{i}" do |subconfig|
      subconfig.vm.network "private_network", ip: "192.168.50.#{i}0", virtualbox__intnet: "mynetwork"
      subconfig.vm.hostname = "machine#{i}"
      subconfig.vm.provider "virtualbox" do |vm|
        vm.name = "machine#{i}"
      end
      subconfig.vm.provision "shell", inline: <<-SHELL
        # apt update && apt upgrade -y
        # apt install -y nginx php-fpm
        sed -i 's/PasswordAuthentication no/PasswordAuthentication yes/g' /etc/ssh/sshd_config
        systemctl restart sshd
      SHELL
    end
  end
end

# Access to the ansible_host from my Windows machine:
# ssh -i vagrant-ansible\.vagrant\machines\ansible_host\virtualbox/private_key -l vagrant -p 2222 localhost
# The port will be displayed in the terminal during the installation.
