# -*- mode: ruby -*-
# vi: set ft=ruby :

Vagrant.configure('2') do |config|
  config.vm.define 'rockabox' do |c|
    c.vm.box = 'rockabox-vbox-linux'
    c.vm.box_url = 'http://cloud-images.ubuntu.com/vagrant/trusty/current/trusty-server-cloudimg-amd64-vagrant-disk1.box'
    c.vm.network :private_network, ip: '192.168.50.0'
    c.vm.hostname = 'rockabox.local'
    c.vm.provision 'ansible' do |ansible|
      ansible.playbook = 'test.yml'
      ansible.sudo = true
      ansible.inventory_path = 'vagrant-inventory'
      ansible.host_key_checking = false
    end
  end
end
