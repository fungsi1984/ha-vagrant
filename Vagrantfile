# -*- mode: ruby -*-
# vi: set ft=ruby :

# Vagrantfile API/syntax version. Don't touch unless you know what you're doing!
VAGRANTFILE_API_VERSION = "2"

Vagrant.configure(VAGRANTFILE_API_VERSION) do |config|
  config.vm.define "server01" do |server01|
    server01.vm.box = "ubuntu/jammy64"
    server01.vm.hostname = "server01"
    server01.vm.network :private_network, ip: "192.168.56.2", auto_config: true
    # server01.vm.provision :shell, path: "resources/provision.sh"
  end

  config.vm.define "server02" do |server02|
    server02.vm.box = "ubuntu/jammy64"
    server02.vm.hostname = "server02"
    server02.vm.network :private_network, ip: "192.168.56.3", auto_config: true
    # server02.vm.provision :shell, path: "resources/provision.sh"
  end

  config.vm.define "db01" do |db01|
    db01.vm.box = "ubuntu/jammy64"
    db01.vm.hostname = "db01"
    db01.vm.network :private_network, ip: "192.168.56.5", auto_config: true
    # server02.vm.provision :shell, path: "resources/provision.sh"
  end

  config.vm.define "db02" do |db02|
    db02.vm.box = "ubuntu/jammy64"
    db02.vm.hostname = "db02"
    db02.vm.network :private_network, ip: "192.168.56.6", auto_config: true
    # server02.vm.provision :shell, path: "resources/provision.sh"
  end

  config.vm.provider :virtualbox do |vb|
    vb.customize ["modifyvm", :id, "--cpus", "1"]
    # You can also keep your memory setting if needed
    vb.customize [ "modifyvm", :id, "--memory", "320" ]
  end
end
