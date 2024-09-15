machines = {
  "master" => {"memory" => "1024", "cpu" => "1", "image" => "bento/ubuntu-22.04", "ip" => "192.168.50.11"},
  "node01" => {"memory" => "1024", "cpu" => "1", "image" => "bento/ubuntu-22.04", "ip" => "192.168.50.12"},
  "node02" => {"memory" => "1024", "cpu" => "1", "image" => "bento/ubuntu-22.04", "ip" => "192.168.50.13"},
  "node03" => {"memory" => "1024", "cpu" => "1", "image" => "bento/ubuntu-22.04", "ip" => "192.168.50.14"}
}

Vagrant.configure("2") do |config|

  machines.each do |name, conf|
    config.vm.define "#{name}" do |machine|
      machine.vm.box = "#{conf["image"]}"
      machine.vm.hostname = "#{name}"
      machine.vm.network "private_network", ip: conf["ip"]

      machine.vm.provider "virtualbox" do |vb|
        vb.name = "#{name}"
        vb.memory = conf["memory"]
        vb.cpus = conf["cpu"]
      end

      machine.vm.provision "shell", inline: <<-SHELL
        apt-get update
        apt-get install -y docker.io
        usermod -aG docker vagrant
      SHELL

      if name == "master"
        machine.vm.provision "shell", inline: <<-SHELL
          docker swarm init --advertise-addr #{conf["ip"]}
          docker swarm join-token worker > /vagrant/worker_join.sh
        SHELL
      else
        machine.vm.provision "shell", inline: <<-SHELL
          bash /vagrant/worker_join.sh
        SHELL
      end

      machine.vm.provision "docker" do |d|
        d.run "apache",
          image: "httpd:latest",
          restart: "always",
          detach: true,
          args: "-p 8080:80"
      end
    end
  end

 
  config.vm.define "master" do |master|
    master.vm.provision "shell", inline: <<-SHELL
      docker service create --name apache-service --replicas 2 -p 80:80 httpd:latest
    SHELL
  end

end