# -*- mode: ruby -*-
# vi: set ft=ruby :

Vagrant.configure(2) do |config|

  config.vm.define "puppetmaster" do |puppetmaster|
  puppetmaster.vm.box = "ubuntu/trusty64"
  puppetmaster.vm.network "forwarded_port", guest: 22, host: 3331
  puppetmaster.vm.network "private_network", ip: "192.168.1.11"
  #puppetmaster.vm.network "public_network", bridge: 'en0: Ethernet' 
  puppetmaster.vm.hostname = "puppetm.test.com"
  puppetmaster.vm.provision "shell", inline: <<-SHELL
    wget https://apt.puppetlabs.com/puppetlabs-release-trusty.deb
    sudo dpkg -i puppetlabs-release-trusty.deb
    sudo apt-get update
    sudo apt-get install -y puppetmaster
    sed -i '/templatedir/ d' /etc/puppet/puppet.conf
    sudo echo "haprox.test.com" >> /etc/puppet/autosign.conf
    sudo echo "apache.test.com" >> /etc/puppet/autosign.conf
    sudo echo "192.168.1.11 puppetm.test.com" >> /etc/hosts
    sudo echo "192.168.1.12 haprox.test.com" >> /etc/hosts
    sudo echo "192.168.1.13 apache.test.com" >> /etc/hosts
    sudo echo '[main]' > /etc/puppet/puppetdb.conf
    sudo echo 'server = puppetm.test.com' >> /etc/puppet/puppetdb.conf
    sudo echo 'port = 8081' >> /etc/puppet/puppetdb.conf
    sudo echo "reports = store,puppetdb" >> /etc/puppet/puppet.conf
    sudo echo "storeconfigs = true" >> /etc/puppet/puppet.conf
    sudo echo "storeconfigs_backend = puppetdb" >> /etc/puppet/puppet.conf

    sudo echo '--- 
               master: 
               facts: 
               terminus: puppetdb 
               cache: yaml' > /etc/puppet/routes.yaml

    sudo echo 'node default {
		service { "puppet":
		ensure => running,
		enable => true,
		}
	}

	node "puppetm.test.com" {
		require puppetdb
	}

	node "haprox.test.com" {

  	 class { "haproxy": } 

  	  haproxy::frontend { "wwwfrontend":
    	  ipaddress => "192.168.1.12",
	  ports => "80",
	  bind_options  => "transparent",
  	  options       => {
    		"default_backend" => "wwwbackend",
    		"timeout client"  => "30",
    		"option"          => [
      		"tcplog",
      		"accept-invalid-http-request",
    		]
  		}
  	  }

	  haproxy::balancermember { "apache":
    		listening_service => "wwwbackend",
    		server_names      => "apache.test.com",
    		ipaddresses       => "192.168.1.13",
    		ports             => "80",
    		options           => "check",
  	  }

	  haproxy::backend { "wwwbackend":
  	  options => {
		mode => "tcp",
    		balance => "roundrobin",
	        },
	  }
	}

	node "apache.test.com" {
		require apache
        }	
      	  ' > /etc/puppet/manifests/site.pp

    sudo chown -R puppet:puppet `sudo puppet config print confdir`
    sudo puppet module install puppetlabs-puppetdb
    sudo puppet module install puppetlabs-haproxy
    sudo puppet module install puppetlabs-apache
    sudo puppet resource package puppetdb-terminus ensure=latest
    sudo service puppetmaster restart
    sudo puppet apply /etc/puppet/manifests/site.pp
    sudo puppet agent --enable
    sudo service puppet start
    #sudo puppet master --daemonize
    #sudo apt-get -y upgrade dist
    #wget https://apt.puppetlabs.com/puppetlabs-release-precise.deb
    #sudo dpkg -i puppetlabs-release-precise.deb
    #sudo apt-get update
    #sudo apt-get install -y puppetmaster
  SHELL
  end

  config.vm.define "haprox" do |haprox|
  haprox.vm.box = "ubuntu/trusty64"
  haprox.vm.network "forwarded_port", guest: 22, host: 3332
  haprox.vm.network "private_network", ip: "192.168.1.12"
  #haprox.vm.network "public_network", bridge: 'en0: Ethernet'
  haprox.vm.hostname = "haprox.test.com"
  haprox.vm.provision "shell", inline: <<-SHELL
    sudo apt-get update
    sudo echo "192.168.1.11 puppetm.test.com" >> /etc/hosts
    sudo echo "192.168.1.12 haprox.test.com" >> /etc/hosts
    sudo echo "192.168.1.13 apache.test.com" >> /etc/hosts
    sudo apt-get install -y puppet
    sudo sed -i '/[main]/a \
    server="puppetm.test.com"' /etc/puppet/puppet.conf
    #sudo apt-get -y upgrade dist
    sudo puppet agent --enable
    sudo service puppet restart
  SHELL
  end

  config.vm.define "apache" do |apache|
  apache.vm.box = "ubuntu/trusty64"
  apache.vm.network "forwarded_port", guest: 22, host: 3333
  apache.vm.network "private_network", ip: "192.168.1.13"
  #apache.vm.network "public_network", bridge: 'en0: Ethernet'
  apache.vm.hostname = "apache.test.com"
  apache.vm.provision "shell", inline: <<-SHELL
    sudo apt-get update
    sudo echo "192.168.1.11 puppetm.test.com" >> /etc/hosts
    sudo echo "192.168.1.12 haprox.test.com" >> /etc/hosts
    sudo echo "192.168.1.13 apache.test.com" >> /etc/hosts
    sudo apt-get install -y puppet
    sudo sed -i '/[main]/a \
    server="puppetm.test.com"' /etc/puppet/puppet.conf
    sudo puppet agent --enable
    sudo service puppet restart
  SHELL
  end

end
