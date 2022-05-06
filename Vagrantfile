require 'yaml'

current_dir     = File.dirname(File.expand_path(__FILE__))
configs         = YAML.load_file("#{current_dir}/provisioning/virtualbox-hosts.yml")
vm_hosts        = configs['all']['children']['ti_vms']['hosts']


VAGRANT_COMMAND = ARGV[0]
VAGRANTFILE_API_VERSION = "2"


['vagrant-vbguest', 'vagrant-hostmanager'].reject{|p| Vagrant.has_plugin? p}.each do |p|
  if system "vagrant plugin install #{p}"
    exec 'vagrant ' + (ARGV.join ' ')
  else
    abort %Q.Failed to install "#{p}" plugin :'(.
  end
end

Vagrant.configure(VAGRANTFILE_API_VERSION) do |config|

    config.vm.box = "ubuntu/focal64" 
    config.vm.box_check_update = false
    
    config.vm.provider :virtualbox do |vb|
        vb.customize ["modifyvm", :id, "--memory", "1024"]
        vb.customize ["modifyvm", :id, "--natdnsproxy1", "off"]
    
        # Set this to "off" if you experience DNS resolution issues.
        # Be warned that setting this to "off" will cause all network connections
        # to the VM to reset on host network status change.
        vb.customize ["modifyvm", :id, "--natdnshostresolver1", "on"]

        config.ssh.forward_agent = true

    end
end


Vagrant.configure("2") do |config|

    config.hostmanager.enabled = true
    config.hostmanager.manage_host = true
    config.hostmanager.aliases = %w(www.)
    config.hostmanager.ignore_private_ip = true
    
    vm_hosts.each do |hostname, host|



        config.vm.define hostname do |node|


            # comment out this line if you don't need it - useful if you need to edit files with an IDE
            node.vm.synced_folder "shared/" + hostname, host['shared_folder_location'], create: true, owner:'www-data', group: 'www-data'


            node.hostmanager.aliases = ['www.'+hostname]
            node.vm.hostname = hostname


            node.vm.network "private_network", ip: host['local_ip'], adapter:2
            node.vm.network "forwarded_port", guest: 80, host: host['http_port'], id: "http"
            node.vm.network "forwarded_port", guest: 443, host: host['https_port'], id: "https"
            node.vm.network "forwarded_port", guest: 22, host: host['ssh_port'], id: "ssh"

            

            node.vm.provision "ansible" do |ansible|\

                ansible.verbose = "v"
                ansible.inventory_path = 'provisioning/virtualbox-hosts.yml'
                requirements_file = 'provisioning/requirements.yml'
      
                if File.exist?(requirements_file) && !Psych.load_file(requirements_file).equal?(nil)
                  ansible.galaxy_role_file = requirements_file
                  ansible.galaxy_roles_path = 'roles/'
                  ansible.galaxy_command = 'ansible-galaxy install --role-file=%{role_file} --roles-path=%{roles_path} --force'
                end

                ansible.playbook = "provisioning/tastyigniter-virtualbox.yml"

            end
        end
    end
end
