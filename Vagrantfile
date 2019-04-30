# Vagrant base box to use
BOX_BASE = "geerlingguy/ubuntu1604"
# amount of RAM for Vagrant box
BOX_RAM_MB = "2048"
# number of CPUs for Vagrant box
BOX_CPU_COUNT = "1"

Vagrant.configure("2") do |config|

  # Define the base box and virtual box specific properties
  config.vm.box = BOX_BASE
  config.vm.provider :virtualbox do |virtualbox|
    virtualbox.customize ["modifyvm", :id, "--memory", BOX_RAM_MB]
    virtualbox.customize ["modifyvm", :id, "--cpus", BOX_CPU_COUNT]
  end

  config.ssh.insert_key = false
  config.ssh.private_key_path = ["~/.ssh/id_rsa.pub", "~/.vagrant.d/insecure_private_key"]
  config.vm.provision "file", source: "~/.ssh/id_rsa.pub", destination: "~/.ssh/id_rsa.pub"
  config.vm.provision "shell", inline: "cat ~vagrant/.ssh/id_rsa.pub >> ~vagrant/.ssh/authorized_keys"

  config.vm.define "staging" do |staging|
    staging.vm.hostname = "staging"
    staging.vm.network "private_network", ip: "192.168.0.3"
    staging.vm.network "forwarded_port", guest: 8080, host: 8081
  end

  config.vm.define "production" do |production|
    production.vm.hostname = "production"
    production.vm.network "private_network", ip: "192.168.0.4"
    production.vm.network "forwarded_port", guest: 8080, host: 8082
  end

  config.vm.define "build" do |build|
    build.vm.hostname = "build"
    build.vm.network "private_network", ip: "192.168.0.2"
    build.vm.network "forwarded_port", guest: 8080, host: 9000

    build.vm.provision :shell, :inline => 'echo "192.168.0.3 staging" >> /etc/hosts'
    build.vm.provision :shell, :inline => 'echo "192.168.0.4 production" >> /etc/hosts'

    build.vm.provision "file", source: "~/.ssh/id_rsa", destination: "~/.ssh/id_rsa"

    # Provisioning the boxes using Ansible.
    build.vm.provision :ansible do |ansible|
      # Configure groups in the autogenerated inventory file.
      ansible.groups = {
        "clickcount" => ["staging", "production"],
      }
      ansible.compatibility_mode = "2.0"
      ansible.limit = "all"
      ansible.ask_vault_pass = true
      ansible.playbook = "ansible/playbook.yml"
    end
  end
end