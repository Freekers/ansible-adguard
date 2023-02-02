# Read the README.md before using this. 

Vagrant.configure("2") do |config|
  $script = <<-SCRIPT
    # Update and upgrade packages
    sudo dnf update -y
    sudo dnf install -y epel-release
    sudo dnf update --refresh -y
    sudo dnf install -y git curl wget ansible python3-pip

    # Create SSH key directory and add authorized keys file
    chmod 700 /home/vagrant/.ssh
    touch /home/vagrant/.ssh/authorized_keys && chmod 600 /home/vagrant/.ssh/authorized_keys

    # Configure keypair and add pubkey to authorized keys
    cd /vagrant && \
    cp tests/vagrant-keys/vagrant_key /home/vagrant/.ssh/id_ed25519
    cp tests/vagrant-keys/vagrant.pub /home/vagrant/.ssh/id_ed25519.pub
    cat /home/vagrant/.ssh/id_ed25519.pub >> /home/vagrant/.ssh/authorized_keys

    # Permissions for SSH keys
    chown vagrant:vagrant /home/vagrant/.ssh/id_ed25519 && chmod 600 /home/vagrant/.ssh/id_ed25519
    chown vagrant:vagrant /home/vagrant/.ssh/id_ed25519.pub && chmod 600 /home/vagrant/.ssh/id_ed25519.pub

    ssh-keyscan -H bitbucket.org >> ~/.ssh/known_hosts

    # install zsh
    # git clone https://github.com/bruvv/ansible-role-zsh.git /tmp/zsh
    # ansible-playbook -i "localhost," -c local -b /tmp/zsh/playbook.yml
    # ansible-playbook -i "localhost," -c local -b /tmp/zsh/playbook.yml --extra-vars="zsh_user=$(whoami)"

    cd /vagrant && ansible-galaxy install -r requirements/requirements.yml
  SCRIPT

  # Add IP's and hostnames to hosts files
  $host_script = <<-SCRIPT
    printf "172.23.18.20 test_server\n" >> /etc/hosts
  SCRIPT

  # Add commands to make the running of playbooks easier
  $playbook_script = <<-SCRIPT
    printf "alias test='cd /vagrant && ansible-playbook test.yml -i inventories/local-inventory.yml'\n" >> /home/vagrant/.bash_profile
  SCRIPT

  # Set the resources for every vagrant box we are going to describe
  config.vm.provider "virtualbox" do |v|
    v.memory = 2048
    v.cpus = 2
    v.gui = true
  end

  # Set synced folders. This way we don't have to install the roles, because the playbooks are in /vagrant and the roles in /vagrant/roles.
  # Ansible will always look for a 'roles' folder when executing a playbook from a directory. This allows for easy development.
  # vagrant vbguest --do install --no-cleanup
  config.vm.synced_folder ".", "/vagrant"
  config.ssh.insert_key = false

  # Actually 'calling' the scripts and run them either with elevated privileges or not.
  config.vm.provision "shell", inline: $script, privileged: false
  config.vm.provision "shell", inline: $host_script, privileged: true

  config.vm.define "test_server" do |ansible_host_01|
    ansible_host_01.vm.box = "geerlingguy/rockylinux8" # centos is dood, rocky linux is centos
    ansible_host_01.vbguest.installer_options = { allow_kernel_upgrade: true }
    # ansible_host_01.vbguest.installer_hooks[:before_install] = ["dnf update -y kernel-*", "sleep 2"]
    # ansible_host_01.vm.box = "ubuntu/jammy64" #ubuntu 22.04
    ansible_host_01.vm.provision "shell", inline: $playbook_script, privileged: false
    ansible_host_01.vm.provision :shell, inline: "hostnamectl set-hostname test_server"
    ansible_host_01.vm.network "private_network", ip: "172.23.18.20", nic_type: "virtio"
    ansible_host_01.vm.provision "shell", inline: "echo rebooting", privileged: true, reboot: true
  end

end
