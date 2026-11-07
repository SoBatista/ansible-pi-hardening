Vagrant.configure("2") do |config|
  config.vm.box      = "debian/trixie64"
  config.vm.hostname = "dev-pi"

  # Ansible reaches the VM over SSH, so we don't need the /vagrant shared folder.
  config.vm.synced_folder ".", "/vagrant", disabled: true

  config.vm.provider :libvirt do |lv|
    lv.memory = 1024
    lv.cpus   = 2
  end
end
