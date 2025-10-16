# -*- mode: ruby -*-
# vi: set ft=ruby :

# This file will set up two VMs through 'libvirt', along with the SSH keys
# used to connect to them via 'SSH'

REQUIRED_PLUGINS = %w(vagrant-libvirt)
exit unless REQUIRED_PLUGINS.all? do |plugin|
  Vagrant.has_plugin?(plugin) || (
    puts "The #{plugin} plugin is required. Please install it with:"
    puts "$ vagrant plugin install #{plugin}"
    false
  )
end

system("
    if [ #{ARGV[0]} = 'up' ]; then
        echo 'Creating SSH deploy keys...'
        [ -f deploy_key ] || ssh-keygen -t ed25519 -N \"\" -f deploy_key -C 'deploy key'
    elif [ #{ARGV[0]} = 'destroy' ]; then
        echo 'Removing SSH deploy keys...'
        rm --verbose --force deploy_key deploy_key.pub
    fi
")

# All Vagrant configuration is done below. The "2" in Vagrant.configure
# configures the configuration version (we support older styles for
# backwards compatibility). Please don't change it unless you know what
# you're doing.
Vagrant.configure("2") do |config|
  # The most common configuration options are documented and commented below.
  # For a complete reference, please see the online documentation at
  # https://docs.vagrantup.com.

  config.vm.provider :libvirt do |libvirt|
    libvirt.graphics_type = "none"
  end

  # Every Vagrant development environment requires a box. You can search for
  # boxes at https://vagrantcloud.com/search.
  config.vm.box = "cloud-image/almalinux-10"

  # Disable automatic box update checking. If you disable this, then
  # boxes will only be checked for updates when the user runs
  # `vagrant box outdated`. This is not recommended.
  # config.vm.box_check_update = false

  if File.exist?("deploy_key.pub")
    public_key = File.read("deploy_key.pub")
  end

  $script = <<-SHELL
      getent passwd deploy || useradd --comment "deploy" --home-dir /home/deploy --create-home deploy
      printf "%deploy ALL=(ALL) NOPASSWD: ALL\n" > /etc/sudoers.d/deploy
      umask 077
      mkdir --parents /home/deploy/.ssh
      printf "#{public_key}\n" > /home/deploy/.ssh/authorized_keys
      chown --recursive deploy:deploy ~deploy
      ip -c a s eth0
    SHELL

  config.vm.define "webserver" do |webserver|
    webserver.vm.hostname = "webserver"
    webserver.vm.provision "shell", inline: $script
    webserver.vm.post_up_message = "To access this VM, do 'ssh -l deploy -i deploy_key.pub [hostname/IP]"
    webserver.vm.synced_folder ".", "/vagrant",
      type: "nfs",
      nfs_version: 4,
      nfs_udp: false
  end

  config.vm.define "dbserver" do |dbserver|
    dbserver.vm.hostname = "dbserver"
    dbserver.vm.provision "shell", inline: $script
    dbserver.vm.post_up_message = "To access this VM, do 'ssh -l deploy -i deploy_key.pub [hostname/IP]"
    dbserver.vm.synced_folder ".", "/vagrant",
      type: "nfs",
      nfs_version: 4,
      nfs_udp: false
  end
end
