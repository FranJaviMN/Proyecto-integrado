# -*- mode: ruby -*-
# vi: set ft=ruby :

Vagrant.configure("2") do |podman|
  podman.vm.box = "generic/debian10"
  podman.vm.hostname = "podman"
  podman.vm.synced_folder ".", "/vagrant", disabled: true
  podman.vm.provider :libvirt do |libvirt|
    libvirt.uri = 'qemu+unix:///system'
    libvirt.host = "podman"
    libvirt.cpus = 1
    libvirt.memory = 1024
  end
end