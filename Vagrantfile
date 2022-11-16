Vagrant.configure("2") do |config|
  config.vm.box = "ubuntu/jammy64"
  config.vm.provider "virtualbox" do |vb|
    vb.memory = "1024"
    vb.cpus = 1
  end

  # config.vm.define "ansible_host" do |anshost|
  #   anshost.vm.network "private_network", ip: "192.168.50.100", virtualbox__intnet: "mynetwork"
  #   anshost.vm.hostname = "ansiblehost"
  #   anshost.vm.provider "virtualbox" do |vm|
  #     vm.name = "ansiblehost"
  #   end
  # end

  (1..3).each do |i|
    config.vm.define "machine_#{i}" do |m|
      m.vm.network "private_network", ip: "192.168.50.#{i}0", virtualbox__intnet: "mynetwork"
      m.vm.hostname = "machine#{i}"
      m.vm.provider "virtualbox" do |vm|
        vm.name = "machine#{i}"
      end
      m.vm.provision "shell", inline: <<-SHELL
        apt update -y
        apt upgrade -y
        apt install -y nginx
        apt install -y php-fpm
      SHELL
    end
  end
end

# Access to the ansible_host from my Windows machine:
# ssh -i vagrant\.vagrant\machines\ansible_host\virtualbox/private_key -l vagrant -p 2222 localhost
# The port will be displayed in the terminal during the installation.
