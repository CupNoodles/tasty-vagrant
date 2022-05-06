# Vagrant developent setup for TastyIgniter Servers

[Vagrant](https://www.vagrantup.com/) is a tool for managing development envorinments. 

This tool utilizes the tasty-ansible ansible role in order to create a stand-alone development VM for TastyIgniter development.

## Who should use this repository?

This code is intended for developers who are interested in making custom modifications to TastyIgniter itself, submitting updates into the TastyIgniter codebase, or creating custom extentions and themes. If what you're looking for is to get started on TastyIgniter or to set up a website powered by TastyIgniter, please see the [Quick Start Guide](https://tastyigniter.com/support/articles/quick-start-guide).

## Testing/development environment for TastyIgniter

Vagrant can use multiple providers to create VMs and development environments. For this purpose, we'll be using [VirtualBox](https://www.virtualbox.org/) since it's free and widely supported, but usage with other providors is encouraged as necessary. 

The following has been tested using VirtualBox and Vagrant on a host machine running Ubuntu 20.04. While it should work on any *nix-based systems, YMMV for any alternate setups. 

Step 0.) Clone this repository

Pick an instll location and clone this repository there.

`git clone git@github.com:CupNoodles/tasty-vagrant.git /my/install/dir`

Step 1.) Install dependencies

```
# install virtualbox (version 6.1.32 as of writing)
sudo apt install virtualbox
# install vagrant (version 2.2.6 as of writing)
sudo apt install vagrant
# install ansible (version 2.9.6 as of writing))
sudo apt install ansible
# install the one ansible collection dependency needed - this should become obselete with the -r flag added to ansible-galaxy in 2.10
ansible-galaxy collection install community.crypto 
```

*Note: You may need to follow prompts and reboot your system when installing virtualbox due to UEFI Secure boot settings. If so, ensure that VirtualBox has been installed sucessfully with `sudo systemctl status virtualbox`.


Step 2.) Configure VirtualBox inventory file

Copy `provisioning/virtualbox-hosts.example.yml` to `provisioning/virtualbox-hosts.yml` and edit the new file to contain information for the new VM you'd like to spin up. It will look something like 

```
all:
  children:
    ti_vms:
      hosts:
        tastyigniter-example.vm:
          hostname: "tastyigniter-example.vm"
          ansible_host: 192.168.56.1
          http_port: 8080
          https_port: 4443
          ansible_port: 2220
          ssh_port: 2220
          disable_combiner: true
          ti_version_tag: '3.x'
          ansible_host: 127.0.0.1
```

You can create multiple VM configurations here by creating another `hosts:` entry as follows. Each host must have a unique name, IP address, and port numbers.

```
    hosts:
      pizza-shop.vm:
        hostname: "tastyigniter-example.vm"
        ansible_host: 192.168.56.1
        http_port: 8080
        https_port: 4443
        ansible_port: 2220
        disable_combiner: true
        ti_version_tag: '3.x'
        ansible_host: 127.0.0.1
      burger-joint.vm
        hostname: "tastyigniter-example.vm"
        ansible_host: 192.168.56.2
        http_port: 8081
        https_port: 4444
        ansible_port: 2221
        disable_combiner: false
        shared_folder_location: "/var/www/html"
        ansible_host: 127.0.0.1
```

Configuration variables for the ansible role can be set here in the hosts file - the whole list of available custom variables can be found at https://github.com/CupNoodles/tasty-ansible/blob/master/defaults/main.yml

Step 3.) Run vagrant up

From your installation directory, run `vagrant up`. 

If you've specified multiple hosts, you can manage individual boxes by name, ex. `vagrant up pizza-shop.vm` etc. 

## Notes on Shared Folders

If you're usually editing files with an IDE, it may be useful to set up shared folders with the `shared_folder_location` configuration option. As it is, this will create a two-way sync between the `shared/` folder in your project directory and a guest machine location of your choosing. Synced folders can be fiddly though, and your optimal solution will depend on your host/guest machine OSs and how much of your guest machine that needs to be shared. If it doesn't work out of the box, please refer to the [Vagrant documentation](https://www.vagrantup.com/docs/synced-folders) for other options.

## Reverse Proxy setup

You may notice that there are a few unnecessary networking and port configuration options, which you can use to set up a reverse proxy on your host machine. This is not at all necessary for working on the VM, but can be useful if, for instance, you need to remove the `:8080` from the URL, or if you're testing the website on a tablet device and thus need to share your VM on the local network.