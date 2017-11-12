# -*- mode: ruby -*-
# vi: set ft=ruby :

Vagrant.configure("2") do |config|
  # All Vagrant configuration is done here. The most common configuration
  # options are documented and commented below. For a complete reference,
  # please see the online documentation at vagrantup.com.

  config.vm.box = "ubuntu/xenial64"
  config.vm.hostname = "oracle"

  # share this project under /home/vagrant/vagrant-ubuntu-oracle-xe
  config.vm.synced_folder ".", "/home/vagrant/vagrant-ubuntu-oracle-xe", :mount_options => ["dmode=777","fmode=666"]

  # Forward Oracle port
  config.vm.network :forwarded_port, guest: 1521, host: 1521

  # Provider-specific configuration so you can fine-tune various backing
  # providers for Vagrant. These expose provider-specific options.
  config.vm.provider :virtualbox do |vb|
    # Use VBoxManage to customize the VM
    vb.customize ["modifyvm", :id,
                  "--name", "oracle",
                  # Oracle claims to need 512MB of memory available minimum
                  "--memory", "2048",
                  # Enable DNS behind NAT
                  "--natdnshostresolver1", "on"]
    end

  # Prevent TTY errors (copied from laravel/homestead: "homestead.rb" file)... By default this is "bash -l".
  # NOTE: This is an ugly hack...
  config.ssh.shell = "bash -c 'BASH_ENV=/etc/profile exec bash'"

  # Install puppet
  config.vm.provision :shell, :inline => "export DEBIAN_FRONTEND=noninteractive; apt-get update && apt-get install -y puppet"

  # This is just an example, adjust as needed
  config.vm.provision :shell, :inline => "echo \"Europe/Paris\" | sudo tee /etc/timezone && dpkg-reconfigure --frontend noninteractive tzdata"

  config.vbguest.auto_update = true

  $install_puppet_modules = <<SCRIPT
  if [ -f /home/vagrant/vagrant-ubuntu-oracle-xe/oracle-jdbc/ojdbc6.jar ]; then
    mkdir -p /etc/puppet/modules
    puppet module list | grep -q puppetlabs-java || puppet module install puppetlabs-java
    puppet module list | grep -q maestrodev-maven || puppet module install maestrodev-maven
  fi
SCRIPT

  config.vm.provision "shell", inline: $install_puppet_modules

  config.vm.provision :puppet do |puppet|
    puppet.manifests_path = "manifests"
    puppet.module_path = "modules"
    puppet.manifest_file = "base.pp"
    puppet.options = "--verbose --trace"
  end

  # Run the Maven goals for data-with-flyway
  config.vm.provision "shell", path: "flyway.sh"
end
