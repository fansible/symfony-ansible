# Ansible provisioning for Symfony project using composer
This project is meant to make the provisioning of servers as easy as possible to run Symfony apps.

## How to use it

1) Add fansible/composer in your composer.json : `"fansible/composer": "dev-master",`

2) Add the file ansible.cfg in your root directory with

    [defaults]
    hostfile = app/config/ansible/hosts
    roles_path = vendor/fansible/composer/roles

3) Add your hosts configurations in `app/config/ansible/hosts` as for vagrant:

    [vagrant]
    vagrant ansible_ssh_host=127.0.0.1 ansible_ssh_port=2222

4) Add your group_vars file in `app/config/ansible/hosts/group_vars` as for vagrant:

    ---
    # File: app/config/ansible/hosts/group_vars/vagrant

    # Write here the vars that are specific to your host
    host_name: "vagrant-{{ name }}"
    web:
        server_name: "vagrant.project.com"
    iptables_allowed_tcp_ports: [22, 80, 443]

    mysql_users:
      - name: "{{ name }}"
        host: "%"
        pass: "{{ name }}"

You can then run your ansible provisioning: `ansible-playbook -i app/config/ansible/prod vendor/fansible/composer/playbook.yml -u root`.
Or for your vagrant: `vagrant provision`

## Vagrant

Here is a Vagrantfile you can use for your project:

    # -*- mode: ruby -*-
    # vi: set ft=ruby :

    # TODO: Change the name
    projectname = 'projectname'

    Vagrant.configure("2") do |config|
      config.vm.hostname = projectname
      config.vm.box = "ubuntu/trusty64"
    # TODO: Change the directory
      config.vm.network :private_network, ip: "10.0.0.7"

    # TODO: Change the directory
     config.vm.synced_folder "./", "/var/www/" + projectname + "/current", type: "nfs"

      config.vm.provider "virtualbox" do |v|
        v.customize ["modifyvm", :id, "--cpuexecutioncap", "100"]
        v.customize ["modifyvm", :id, "--memory", 2048]
        v.customize ["modifyvm", :id, "--cpus", 2]
      end

      config.ssh.forward_agent = true

      # Ansible see https://docs.vagrantup.com/v2/provisioning/ansible.html
      config.vm.provision "ansible" do |ansible|
        ansible.sudo = true
        ansible.playbook = "vendor/fansible/composer/playbook.yml"
        ansible.limit = 'vagrant'
        ansible.inventory_path = "app/config/ansible/hosts/vagrant"
        ansible.verbose = "v" #Use vvvv to get more log
      end
    end
