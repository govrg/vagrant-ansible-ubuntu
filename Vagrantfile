$set_environment_variables = <<SCRIPT
tee "/etc/profile.d/myvars.sh" > "/dev/null" <<EOF
# Ansible environment variables.
export ansible_usr=#{ENV['ANSIBLE_USR']}
export ansible_ssh_pass=#{ENV['ANSIBLE_SSH_PASS']}

# User for ssh environment variables.
export dock_email=#{ENV['DOCKER_EMAIL']}
export dock_usr=#{ENV['DOCKER_USR']}
export dock_password=#{ENV['DOCKER_PASSW']}
EOF
SCRIPT

Vagrant.configure("2") do |config|

  #for the bento/ubuntu-18.04
  config.vm.network 'private_network', ip: '192.168.33.101'
  # Set video memory for the vagrant box
  config.vm.provider "virtualbox" do |v|
    v.customize ["modifyvm", :id, "--vram", "15"]

    v.memory = 2048
    v.cpus = 2
    v.customize ["modifyvm", :id, "--natdnshostresolver1", "on"]
    v.customize ["modifyvm", :id, "--ioapic", "on"]
  end
  config.vm.box = "bento/ubuntu-18.04" 
  config.vm.hostname = 'master1'

  config.vm.synced_folder ".", "/vagrant"
  config.vm.synced_folder "C:\\Users\\govrg\\.ssh", "/home/vagrant/ssh-host" 

  config.vm.provision "shell", inline: <<-SHELL
    sudo apt-get update -y
    cp /home/vagrant/ssh-host/* /home/vagrant/.ssh/.
  SHELL

  config.vm.provision "shell", inline: $set_environment_variables, run: "always"
  
  config.vm.provision "shell", inline: <<-SHELL
    sudo apt-get install -y software-properties-common
    sudo apt-add-repository ppa:ansible/ansible
    sudo apt-get update -y
    sudo apt-get install -y ansible
    cd /vagrant
    ansible-playbook local.yml
  SHELL
end
