Vagrant.configure("2") do |config|

  config.vm.box = "ubuntu/bionic64"
  config.vm.network "forwarded_port", guest: 8080, host: 8080
  config.vm.network "forwarded_port", guest: 9000, host: 9000
  config.vm.network "private_network", type: "dhcp"

  config.vm.provider "virtualbox" do |vb|
      vb.memory = "4096"
  end

  config.vm.provision "shell", inline: <<-SHELL
    curl -fsSL https://get.docker.com  | sh
    usermod -aG docker vagrant
  SHELL
end
