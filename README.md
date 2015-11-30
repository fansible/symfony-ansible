# /!\ Deprecated /!\
See https://github.com/fansible/tywin for the newer version.


# Ansible provisioning for Symfony project using composer
This project is meant to make the provisioning of servers running one Symfony app as easy and fast as possible.

## Requirements

You should have on your server installed:
* [Ansible](http://docs.ansible.com/intro_installation.html)
* [Composer](https://getcomposer.org/download/)

## How to use it

1) Require fansible/symfony-ansible in your composer.json: `composer require --dev "fansible/symfony-ansible"`

2) Add the file ansible.cfg in your root directory with

    [defaults]
    hostfile = app/config/ansible/hosts
    roles_path = vendor/fansible/symfony-ansible/roles

3) Add your hosts configurations. For vagrant, create a file called `vagrant` in `app/config/ansible/hosts`:
    
    [vagrant]
    vagrant ansible_ssh_host=127.0.0.1 ansible_ssh_port=2222 is_vagrant=true

4) Add the specific vars of your host. For vagrant, create a file called `vagrant` in `app/config/ansible/hosts/group_vars`:

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

5) If you have already installed Ansible, you can now run your provisioning.

For your vagrant: `vagrant provision`

For any hosts: `ansible-playbook -i app/config/ansible/hosts/HOSTNAME vendor/fansible/symfony-ansible/playbook.yml -u root`.

## Bonus step for Vagrant

1) You need to create Here is a Vagrantfile you can use for your project:

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
        ansible.playbook = "vendor/fansible/symfony-ansible/playbook.yml"
        ansible.limit = 'vagrant'
        ansible.inventory_path = "app/config/ansible/hosts/vagrant"
        ansible.verbose = "v" #Use vvvv to get more log
      end
    end

2) Change your web/app_dev.php to allow remote connection. You can copy/paste:

    <?php

    use Symfony\Component\HttpFoundation\Request;
    use Symfony\Component\Debug\Debug;

    $loader = require_once __DIR__.'/../app/bootstrap.php.cache';
    Debug::enable();

    require_once __DIR__.'/../app/AppKernel.php';

    $kernel = new AppKernel('dev', true);
    $kernel->loadClassCache();
    $request = Request::createFromGlobals();
    $response = $kernel->handle($request);
    $response->send();
    $kernel->terminate($request, $response);
