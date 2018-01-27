# encoding: utf-8
# -*- mode: ruby -*-
# vi: set ft=ruby :

machines = [
	 { name: 'chef',       subdomain: 'vmhost',    ram: '512', cpus: '1' }
]

Vagrant.configure('2') do |config|
	config.vm.box = 'centos/7'
	config.vbguest.auto_update = false
	machines.each do |machine|
		config.vm.define machine[:name] do |guest|
			guest.vm.hostname= 'blah1'
			guest.vm.provider :virtualbox do |vb|
				vb.name = 'blah1'
				vb.memory = '512'
				vb.cpus = '1'
				vb.customize [ 'modifyvm', :id, '--vram', '33' ]
			end
			guest.vm.provision :chef_solo do |chef|
				chef.run_list = [ 'recipe[chef-client]' ]
			end
		end
	end
end

