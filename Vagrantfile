# encoding: utf-8
# -*- mode: ruby -*-
# vi: set ft=ruby :

# CONSTANTS
SSH_KEY = 'edu.psu@pke3'
DOMAIN = 'psu'
USER_NAME = 'pke3'
FIRST_NAME = 'Kieran'
LAST_NAME = 'Etienne'
EMAIL = 'pke3@psu.edu'
PASSWORD = 'insecurepassword'
SHORT_NAME = 'psul'
LONG_NAME = 'Penn State University Libraries'

$chef_server_config = <<SCRIPT
chef-server-ctl user-create #{USER_NAME} #{FIRST_NAME} #{LAST_NAME} #{EMAIL} #{PASSWORD} -f /vagrant/#{USER_NAME}.pem
scp -i /home/vagrant/.ssh/#{SSH_KEY}.pem -o StrictHostKeyChecking=no /vagrant/#{USER_NAME}.pem #{ENV['USER']}@10.0.2.2:#{ENV['HOME']}/.chef/#{USER_NAME}.pem
chef-server-ctl org-create #{SHORT_NAME} #{LONG_NAME} --association_user #{USER_NAME} -f /vagrant/#{SHORT_NAME}-validator.pem
scp -i /home/vagrant/.ssh/#{SSH_KEY}.pem -o StrictHostKeyChecking=no /vagrant/#{SHORT_NAME}-validator.pem #{ENV['USER']}@10.0.2.2:#{ENV['HOME']}/.chef/#{SHORT_NAME}-validator.pem
SCRIPT

$chef_client_config = <<SCRIPT
/bin/echo 'log_level :info' >> /home/vagrant/client.rb
/bin/echo "log_location '/var/log/chef/client.log'" >> /home/vagrant/client.rb
/bin/echo "chef_server_url 'https://chef.vmhost.psu.test/organizations/#{SHORT_NAME}'" >> /home/vagrant/client.rb
/bin/echo "validation_client_name '#{SHORT_NAME}-validator'" >> /home/vagrant/client.rb
/bin/echo "validation_key '/etc/chef/#{SHORT_NAME}.pem'" >> /home/vagrant/client.rb
/bin/echo 'ssl_verify_mode :verify_none' >> /home/vagrant/client.rb
/bin/echo 'verify_api_cert false' >> /home/vagrant/client.rb
SCRIPT

machines = [
	 { name: 'chef',       subdomain: 'vmhost',    ram: '512', cpus: '1' },
	 { name: 'isilon',     subdomain: 'vmhost',    ram: '512', cpus: '1' }
	#{ name: 'mariaprod',  subdomain: 'vmhost',    ram: '512', cpus: '1' }
	#{ name: 'mariatest',  subdomain: 'vmhost',    ram: '512', cpus: '1' }
	#{ name: 'ssprodweb',  subdomain: 'libraries', ram: '512', cpus: '1' }
	#{ name: 'ssprodjobs', subdomain: 'libraries', ram: '512', cpus: '1' }
	#{ name: 'ssprodrepo', subdomain: 'libraries', ram: '512', cpus: '1' }
	#{ name: 'ssprodweb',  subdomain: 'libraries', ram: '512', cpus: '1' }
	#{ name: 'ssprodjobs', subdomain: 'libraries', ram: '512', cpus: '1' }
	#{ name: 'ssprodrepo', subdomain: 'libraries', ram: '512', cpus: '1' }
]

Vagrant.configure('2') do |config|
	config.vm.box_check_update = true
	config.vm.box = 'centos/7'
	config.vm.network :forwarded_port,
		guest: 22, host: 2222, id: 'ssh', disabled: true
	config.vm.network :forwarded_port,
		guest: 22, host: 2500, id: 'ssh', disabled: false, auto_correct: true
	config.vm.usable_port_range = (2501..2535)
	config.landrush.enabled = true
	config.landrush.tld = "#{DOMAIN}.test"
	config.ssh.private_key_path = [
		"#{ENV['HOME']}/.ssh/#{SSH_KEY}.pem",
		"#{ENV['HOME']}/.vagrant.d/insecure_private_key"
	]
	config.ssh.insert_key = false
	config.vm.provision :file,
		source: "#{ENV['HOME']}/.ssh/#{SSH_KEY}.pub",
		destination: "/home/vagrant/.ssh/authorized_keys"
	config.vm.provision :file,
		source: "#{ENV['HOME']}/.ssh/#{SSH_KEY}.pem",
		destination: "/home/vagrant/.ssh/#{SSH_KEY}.pem"
	config.vm.provision :file,
		source: "#{ENV['HOME']}/.ssh/#{SSH_KEY}.pub",
		destination: "/home/vagrant/.ssh/#{SSH_KEY}.pub"
	machines.each do |machine|
		config.vm.define machine[:name] do |guest|
			guest.vm.hostname = "#{machine[:name]}.#{machine[:subdomain]}.#{DOMAIN}.test"
			guest.vm.provider :virtualbox do |vb|
				vb.name = machine[:name]
				vb.memory = machine[:ram]
				vb.cpus = machine[:cpus]
				vb.customize [ 'modifyvm', :id, '--vram', '33' ]
			end
			guest.vm.provision :chef_solo do |chef|
				chef.synced_folder_type = 'rsync'
			end
			if machine[:name] == 'chef'
				guest.vm.provision :chef_solo do |chef|
					chef.synced_folder_type = 'rsync'
					chef.run_list = [ 'recipe[chef-server]' ]
				end
				guest.vm.provision :shell, inline: $chef_server_config
			else
				guest.vm.provision :chef_solo do |chef|
					chef.synced_folder_type = 'rsync'
					#chef.run_list = [ 'recipe[chef-client]' ]
				end
				
				# duplicates some of below
				guest.vm.provision :shell, inline: $chef_client_config
				
				guest.vm.provision :file, source: "#{ENV['HOME']}/.chef/#{SHORT_NAME}-validator.pem", destination: "/home/vagrant/#{SHORT_NAME}-validator.pem"
				guest.vm.provision :shell, inline: "mv /home/vagrant/#{SHORT_NAME}-validator.pem /etc/chef"
				guest.vm.provision :shell, inline: $script
				guest.vm.provision :shell, inline: "mv /home/vagrant/client.rb /etc/chef"
				guest.vm.provision :shell, inline: 'chef-client'
			end
		end
	end
end
