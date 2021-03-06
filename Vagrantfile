# -*- mode: ruby -*-
# vi: set ft=ruby :

# require_relative './script/authorize_key'

domain          = "test.dev"
setup_complete  = false

# NOTE: currently using the same OS for all boxen
OS="centos" # "debian" || "centos"

# All Vagrant configuration is done below. The "2" in Vagrant.configure
# configures the configuration version (we support older styles for
# backwards compatibility). Please don't change it unless you know what
# you're doing.
Vagrant.configure(2) do |config|
  package=""
  if OS=="debian"
    config.vm.box = "debian/jessie64"
    package="_apt"
  elsif OS=="centos"
    config.vm.box = "centos/7"
    package="_yum"
  else
    puts "you must set the OS variable to a valid value before continuing"
    exit
  end

  {
    # 'solr'  => '10.11.12.103',
    # 'db'    => '10.11.12.102',
    'elasticsearch'   => '10.11.12.101'
  }.each do |short_name, ip|
    config.vm.define short_name do |host|
      host.vm.network 'private_network', ip: ip
      host.vm.hostname = "#{short_name}.#{domain}"
      # presumes installation of https://github.com/cogitatio/vagrant-hostsupdater on host
      host.hostsupdater.aliases = ["#{short_name}"]
      # avoinding "Authentication failure" issue
      host.ssh.insert_key = false
      host.vm.synced_folder ".", "/vagrant", disabled: true

      host.vm.provider "virtualbox" do |vb|
        vb.name = "#{short_name}.#{domain}"
        vb.memory = 1024
        vb.linked_clone = true
      end

      if short_name == "elasticsearch" # last in the list
        setup_complete = true
      end

      if setup_complete
        if OS=="centos"
          # workaround for https://github.com/mitchellh/vagrant/issues/8142
          host.vm.provision "shell",
            inline: "sudo service network restart"
        end

        host.vm.provision "ansible" do |ansible|
          ansible.galaxy_role_file = "requirements.yml"
          ansible.inventory_path = "inventory/vagrant"
          ansible.playbook = "setup.yml"
          ansible.limit = "all"
        end

        # NOTE: nfs synced_folder is broken in vagrant 1.9.1.
        # must downgrade to 1.9.0 for this to work
        # https://github.com/mitchellh/vagrant/issues/8138
        host.vm.synced_folder "project-code", "/opt/es-tutorial", type: 'nfs', mount_options: ['rw', 'vers=3', 'tcp', 'fsc' ,'actimeo=1'], map_uid: 0, map_gid: 0
      end
    end
  end
end
