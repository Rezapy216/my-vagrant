require "fileutils"

# generate keys in my system before crating VMs
key_path = File.expand_path("keys/manager_key", __dir__)
FileUtils.mkdir_p(File.dirname(key_path))
unless File.exist?(key_path)
  system("ssh-keygen", "-t", "ed25519", "-N", "", "-f", key_path, "-C", "my-vagrant")
end
pub_key = File.read("#{key_path}.pub").strip

Vagrant.configure("2") do |config|

  config.vm.box = "ubuntu/jammy64"

  # manger
  config.vm.define "manager" do |mng|
    mng.vm.hostname = "manager"
    mng.vm.network "private_network", ip: "192.168.56.10" 
    mng.vm.provider "virtualbox" do |vb|
        vb.memory = 1024
        vb.cpus = 2
    end
    
    mng.vm.provision "shell", inline: <<-SHELL
      
      # create ssh directory and copy the keys that we generate earlier
      mkdir -p /home/vagrant/.ssh
      echo "#{File.read(key_path)}" > /home/vagrant/.ssh/id_ed25519
      echo "#{pub_key}" > /home/vagrant/.ssh/id_ed25519.pub

      # access assignment
      chmod 600 /home/vagrant/.ssh/id_ed25519
      chmod 644 /home/vagrant/.ssh/id_ed25519.pub
      chown -R vagrant:vagrant /home/vagrant/.ssh
      # install tools and ansible
      apt-get update -y
      apt-get install -y ansible sshpass
    SHELL
  end
      

  # worker1
  config.vm.define "worker1" do |w1|
    w1.vm.hostname = "worker1"
    w1.vm.network  "private_network", ip: "192.168.56.11" 
    w1.vm.provider "virtualbox" do |vb|
        vb.memory = 2048
        vb.cpus = 2
    end
    
    # add public key to worker1 
    w1.vm.provision "shell", privileged: true, inline: <<-SHELL
      install -d -o vagrant -g vagrant -m 700 /home/vagrant/.ssh
      touch /home/vagrant/.ssh/authorized_keys
      grep -qxF '#{pub_key}' /home/vagrant/.ssh/authorized_keys ||
        echo '#{pub_key}' >> /home/vagrant/.ssh/authorized_keys
      chown vagrant:vagrant /home/vagrant/.ssh/authorized_keys
      chmod 600 /home/vagrant/.ssh/authorized_keys
    SHELL
  end

     

  #worker2
  config.vm.define "worker2" do |w2|
    w2.vm.hostname = "worker2"
    w2.vm.network  "private_network", ip: "192.168.56.12" 
    w2.vm.provider "virtualbox" do |vb|
        vb.memory = 1024
        vb.cpus = 1 
    end

    # add public key to worker2
    w2.vm.provision "shell", privileged: true, inline: <<-SHELL
      install -d -o vagrant -g vagrant -m 700 /home/vagrant/.ssh
      touch /home/vagrant/.ssh/authorized_keys
      grep -qxF '#{pub_key}' /home/vagrant/.ssh/authorized_keys ||
        echo '#{pub_key}' >> /home/vagrant/.ssh/authorized_keys
      chown vagrant:vagrant /home/vagrant/.ssh/authorized_keys
      chmod 600 /home/vagrant/.ssh/authorized_keys
    SHELL
  end    


  # worker3 
    config.vm.define "worker3" do |w3|
    w3.vm.hostname = "worker3"
    w3.vm.network  "private_network", ip: "192.168.56.13" 
    w3.vm.provider "virtualbox" do |vb|
        vb.memory = 1024
        vb.cpus = 1 
    end

    # add public key to worker3 
    w3.vm.provision "shell", privileged: true, inline: <<-SHELL
      install -d -o vagrant -g vagrant -m 700 /home/vagrant/.ssh
      touch /home/vagrant/.ssh/authorized_keys
      grep -qxF '#{pub_key}' /home/vagrant/.ssh/authorized_keys ||
        echo '#{pub_key}' >> /home/vagrant/.ssh/authorized_keys
      chown vagrant:vagrant /home/vagrant/.ssh/authorized_keys
      chmod 600 /home/vagrant/.ssh/authorized_keys
    SHELL

    # ansible after running last worker
    w3.vm.provision "ansible" do |ans|
      ans.playbook = "playbooks/site.yaml"
      ans.inventory_path = "inventory/hosts.ini"
      ans.limit = "all"
    end
  end
end