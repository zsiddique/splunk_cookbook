require 'berkshelf/vagrant'

if Vagrant::VERSION > "1.2.0"

  # Choose your OS, Override with Environment Variables.
  BOX_NAME = ENV['BOX_NAME'] || 'precise64'
  BOX_URI = ENV['BOX_URI']   || 'https://opscode-vm.s3.amazonaws.com/vagrant/boxes/opscode-ubuntu-12.04.box'

  # Set your box size, Override with Environment Variables.   
  BOX_MEM = ENV['BOX_MEM'] || '1024'
  BOX_CPU = ENV['BOX_CPU'] || '2' # higher number decrease compile times

  # Chef Run List
  chef_run_list = %w[
    recipe[splunk::server]    
  ]

# Chef JSON
chef_json = {
  splunk: {
    web_server_port: '8080'
  }
}



  Vagrant.configure("2") do |config|
    config.berkshelf.enabled = true
    config.vm.provider :virtualbox do |vb|
        vb.customize ["modifyvm", :id, "--memory", BOX_MEM] 
        vb.customize ["modifyvm", :id, "--cpus", BOX_CPU]
    end
    config.vm.provider :lxc do |lxc|
      lxc.customize 'cgroup.memory.limit_in_bytes', "#{BOX_MEM}M"
    end

    config.vm.network :forwarded_port, guest: 8080, host: 8080
    config.vm.network :forwarded_port, guest: 9997, host: 9997

    # Ensure latest Chef is installed for provisioning
    config.omnibus.chef_version = :latest

    config.vm.define :splunk do |config|
      config.vm.box = BOX_NAME
      config.vm.box_url = BOX_URI
      config.vm.hostname = 'splunk'
      config.vm.network :private_network, ip: "33.33.33.11"
      config.ssh.max_tries = 40
      config.ssh.timeout   = 120
      config.ssh.forward_agent = true

      config.vm.provision :chef_solo do |chef|
        chef.cookbooks_path    = [ '/tmp/splunk-cookbooks' ]
        chef.provisioning_path = '/etc/vagrant-chef'
        chef.log_level         = :debug
        chef.run_list          = chef_run_list
        chef.json              = chef_json
      end
    end
  end
end