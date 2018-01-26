# encoding: utf-8
# -*- mode: ruby -*-
# vi: set ft=ruby :

SSH_KEY='ext.domain@ssh'
DOMAIN='psu'

machines = [
	 { :name => 'chef',       :ssh_port => '3000', :subdomain => 'vmhost',    :ram => '512', :cpus => '1' },
	 { :name => 'isilon',     :ssh_port => '3001', :subdomain => 'vmhost',    :ram => '512', :cpus => '1' }
	#{ :name => 'mariaprod',  :ssh_port => '3002', :subdomain => 'libraries', :ram => '512', :cpus => '1' }
	#{ :name => 'ssprodweb',  :ssh_port => '3003', :subdomain => 'libraries', :ram => '512', :cpus => '1' }
	#{ :name => 'ssprodjobs', :ssh_port => '3004', :subdomain => 'libraries', :ram => '512', :cpus => '1' }
	#{ :name => 'ssprodrepo', :ssh_port => '3005', :subdomain => 'libraries', :ram => '512', :cpus => '1' }
	#{ :name => 'ssprodweb',  :ssh_port => '3006', :subdomain => 'libraries', :ram => '512', :cpus => '1' }
	#{ :name => 'ssprodjobs', :ssh_port => '3007', :subdomain => 'libraries', :ram => '512', :cpus => '1' }
	#{ :name => 'ssprodrepo', :ssh_port => '3008', :subdomain => 'libraries', :ram => '512', :cpus => '1' }
]

Vagrant.configure('2') do |config|
	config.landrush.enabled = true
	config.landrush.tld = 'psu.test'
	config.vbguest.auto_update = false
	config.vm.box_check_update = true
	config.vm.box = 'centos/7'
	config.ssh.private_key_path = [ "#{ENV['HOME']}/.ssh/#{SSH_KEY}.pem", "~/.vagrant.d/insecure_private_key" ]
	config.ssh.insert_key = false
	config.vm.provision "file", source: "#{ENV['HOME']}/.ssh/#{SSH_KEY}.pub", destination: "~/.ssh/authorized_keys"
	machines.each do |machine|
		config.vm.define machine[:name] do |guest|
			guest.vm.hostname = "#{machine[:name]}.#{machine[:subdomain]}.#{DOMAIN}.test"
			guest.vm.network :forwarded_port, :guest => 22, :host => 2222, :id => 'ssh', :disabled => true
			guest.vm.network :forwarded_port, :guest => 22, :host => machine[:ssh_port]
			guest.vm.provider :virtualbox do |vb|
				vb.name = machine[:name]
				vb.memory = machine[:ram]
				vb.cpus = machine[:cpus]
				vb.customize [ 'modifyvm', :id, '--vram', '33' ]
			end
		end
		config.vm.provision :chef_solo do |chef|
			if machine[:name] == 'chef'
				chef.run_list = [ 'recipe[chef-server]' ]
			else
				chef.run_list = [ 'recipe[chef-client]' ]
			end
		end
	end
end
