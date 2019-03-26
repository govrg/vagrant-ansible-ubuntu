$set_environment_variables = <<SCRIPT
tee "/etc/profile.d/myvars.sh" > "/dev/null" <<EOF
# Ansible environment variables.
export ansible_usr=#{ENV['ANSIBLE_USR']}
export ANSIBLE_HOST_KEY_CHECKING=False

# User for ssh environment variables.
export dock_usr=#{ENV['DOCKER_USR']}
EOF
SCRIPT

Vagrant.configure("2") do |config|

  #Vagrant machines so that they all use the same SSH key
  #config.ssh.insert_key = false

  #for the bento/ubuntu-18.04
   
  
  # Set video memory for the vagrant box
  config.vm.provider "virtualbox" do |v|
    v.customize ["modifyvm", :id, "--vram", "15"]
    v.memory = 2048
    v.cpus = 2
    v.customize ["modifyvm", :id, "--natdnshostresolver1", "on"]
    v.customize ["modifyvm", :id, "--ioapic", "on"]
  end

  config.vm.define "worker1" do |machine|
    machine.vm.box = "bento/ubuntu-18.04"
    machine.vm.hostname = 'worker1'
    machine.vm.network 'forwarded_port', host:2220, guest: 22, id: "ssh", auto_correct: true
    machine.vm.network 'private_network', ip: '192.168.33.102'

    machine.vm.synced_folder ".", "/vagrant"
    machine.vm.synced_folder "C:\\Users\\govrg\\.ssh", "/home/vagrant/ssh-host" 
  
    machine.vm.provision "shell", inline: <<-SHELL
      sudo apt-get update -y
      cp /home/vagrant/ssh-host/* /home/vagrant/.ssh/.
    SHELL
  
    machine.vm.provision "shell", inline: $set_environment_variables, run: "always"
    
    machine.vm.provision "shell", inline: <<-SHELL
      sudo apt-get install -y software-properties-common
      sudo apt-add-repository ppa:ansible/ansible
      sudo apt-get update -y
      sudo apt-get install -y ansible
    SHELL
  end

  config.vm.define "master1" do |machine|
    machine.vm.box = "bento/ubuntu-18.04"
    machine.vm.hostname = 'master1'
    machine.vm.network 'forwarded_port', host:2220, guest: 22, id: "ssh", auto_correct: true
    machine.vm.network 'private_network', ip: '192.168.33.101'

    machine.vm.synced_folder ".", "/vagrant"
    machine.vm.synced_folder "C:\\Users\\govrg\\.ssh", "/home/vagrant/ssh-host" 
  
    machine.vm.provision "shell", inline: <<-SHELL
      sudo apt-get update -y
      cp /home/vagrant/ssh-host/* /home/vagrant/.ssh/.
      mkdir /home/vagrant/vagrant-ansible-ubuntu/
      cp /vagrant/* /home/vagrant/vagrant-ansible-ubuntu/ -r
      sudo chown vagrant:vagrant -R /home/vagrant/vagrant-ansible-ubuntu/
    SHELL
  
    machine.vm.provision "shell", inline: $set_environment_variables, run: "always"
    
    machine.vm.provision "shell", inline: <<-SHELL
      sudo apt-get install -y software-properties-common
      sudo apt-add-repository ppa:ansible/ansible
      sudo apt-get update -y
      sudo apt-get install -y ansible
      cd /home/vagrant/vagrant-ansible-ubuntu/
      sudo mkdir /opt/ansible/
      sudo cp vault_password /opt/ansible/
      sudo chmod a-x /opt/ansible/vault_password
      #vagrant is using /etc/ansible/
      sudo cp ansible.cfg /etc/ansible/
      ansible-playbook -i ./hosts --vault-password-file /opt/ansible/vault_password local.yml 
    SHELL
  end
end
