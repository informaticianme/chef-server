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
		machines.each do |machine|
		config.vm.define machine[:name] do |guest|
			guest.vm.hostname = "#{machine[:name]}.#{machine[:subdomain]}.#{DOMAIN}.test"
			guest.vm.provider :virtualbox do |vb|
				vb.name = machine[:name]
				vb.memory = machine[:ram]
				vb.cpus = machine[:cpus]
				vb.customize [ 'modifyvm', :id, '--vram', '33' ]
			end
			if machine[:name] == 'chef'
				guest.vm.provision :chef_solo do |chef|
					chef.synced_folder_type = 'rsync'
					chef.run_list = [ 'recipe[chef-server]' ]
				end
				config.vm.provision :file,
					source: ".vagrant/machines/#{machine[:name]}/virtualbox/private_key_#{machine[:name]}",
					destination: "/home/vagrant/.ssh/private_key_#{machine[:name]}"
				config.vm.provision :shell,
					inline: "scp -i /home/vagrant/.ssh/private_key_#{machine[:name]} -o StrictHostKeyChecking=no vagrant@#{machine[:name]}.#{machine[:subdomain]}.#{DOMAIN}.test:/home/vagrant/.ssh/authorized_keys"
				config.vm.provision :shell,
					inline: "chef-server-ctl user-create #{USER_NAME} #{FIRST_NAME} #{LAST_NAME} #{EMAIL} #{PASSWORD} -f /vagrant/#{USER_NAME}.pem"
				config.vm.provision :shell,
					inline: "chef-server-ctl org-create #{SHORT_NAME} #{LONG_NAME} --association_user #{USER_NAME} -f /vagrant/#{SHORT_NAME}-validator.pem"
				config.vm.provision :shell,
					inline: "scp -i /home/vagrant/.ssh/private_key_chef -o StrictHostKeyChecking=no /vagrant/#{USER_NAME}.pem #{ENV['USER']}@10.0.2.2:#{ENV['HOME']}/.chef/#{USER_NAME}.pem"
				config.vm.provision :shell,
					inline: "scp -i /home/vagrant/.ssh/private_key_chef -o StrictHostKeyChecking=no /vagrant/#{SHORT_NAME}-validator.pem #{ENV['USER']}@10.0.2.2:#{ENV['HOME']}/.chef/#{SHORT_NAME}-validator.pem"
			else
				guest.vm.provision :chef_solo do |chef|
					chef.synced_folder_type = 'rsync'
					chef.run_list = [ 'recipe[chef-client]' ]
				end
				guest.vm.provision :file, source: "#{ENV['HOME']}/.chef/#{SHORT_NAME}-validator.pem", destination: "/home/vagrant/#{SHORT_NAME}-validator.pem"
				guest.vm.provision :shell, inline: "mv /home/vagrant/#{SHORT_NAME}-validator.pem /etc/chef"
				guest.vm.provision :shell, inline: $script
				guest.vm.provision :shell, inline: "mv /home/vagrant/client.rb /etc/chef"
				guest.vm.provision :shell, inline: 'chef-client'
			end
		end
	end
end
