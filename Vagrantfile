# -*- mode: ruby -*-
# vi: set ft=ruby :

Vagrant.configure("2") do |config|
  #config.vm.synced_folder ".", "/mnt/vagrant", id: "vagrant", automount: true
  #config.vm.provision "shell", inline: "usermod -a -G vboxsf vagrant"
  #config.vm.provision "shell", inline: "ln -sfT /media/sf_vagrant /vagrant"

  #config.vm.provision "file", source: "mongod.conf", destination: "mongod.conf"
  #config.vm.provision "file", source: "dev.Dockerfile", destination: "dev.Dockerfile"
  #config.vm.provision "file", source: "docker-compose.yml", destination: "docker-compose.yml"

  config.vm.box = "ubuntu/focal64"
  config.vm.box_download_insecure=true
  config.disksize.size = '30GB'
  config.vm.network "forwarded_port", guest: 3000, host: 3000, host_ip: "127.0.0.1"
  config.vm.network "forwarded_port", guest: 3001, host: 3001, host_ip: "127.0.0.1"
  config.vm.network "forwarded_port", guest: 3002, host: 3002, host_ip: "127.0.0.1"
  config.vm.network "forwarded_port", guest: 6379, host: 6379, host_ip: "127.0.0.1"
  config.vm.network "forwarded_port", guest: 9222, host: 9222, host_ip: "127.0.0.1"
  config.vm.network "forwarded_port", guest: 27017, host: 27017, host_ip: "127.0.0.1"
  config.vm.network "forwarded_port", guest: 27017, host: 27018, host_ip: "127.0.0.1"
  config.vm.network "forwarded_port", guest: 22022, host: 22022, host_ip: "127.0.0.1"
  config.vm.network "forwarded_port", guest: 22023, host: 22023, host_ip: "127.0.0.1"
  config.vm.provider "virtualbox" do |vb|
    vb.memory = "4096"
    vb.cpus = 4
  end
  config.vm.provision "shell", inline: <<-SHELL
   # Install PerconaMongoDB client and tools
    wget https://repo.percona.com/apt/percona-release_latest.$(lsb_release -sc)_all.deb
    dpkg -i percona-release_latest.$(lsb_release -sc)_all.deb
    percona-release enable psmdb-50 release
    apt-get update
    apt-get install -y percona-server-mongodb-tools percona-server-mongodb-shell percona-server-mongodb-mongos
   # Install docker environment
    apt-get install -y docker.io docker-compose
    usermod -aG docker vagrant
    systemctl enable docker
   # Create swap file
    fallocate -l 4G /swap
    chmod 600 /swap
    mkswap /swap
    swapon /swap
    echo "/swap    none    swap    sw    0    0" >> /etc/fstab
   # Create MongoDB volumes
    docker volume create mongodb-pri
    docker volume create mongodb-sec
    docker volume create mongod.conf
    mkdir /var/lib/docker/volumes/mongodb-pri/data
    mkdir /var/lib/docker/volumes/mongodb-sec/data

    cp /vagrant/* /home/vagrant
    docker-compose up -d

  # Wait dockers to start
    sleep 10
    while [ "$(docker ps -q|wc -l)" -ne "4" ]; do echo "Waiting for dockers to start";sleep 1; done
  # Make the Replica
    mongo --quiet --eval 'rs.initiate({_id : "replica01", members : [{_id : 0, host : "10.30.20.10:27017",priority : 10},{_id : 1, host : "10.30.20.11:27017",priority : 5}]})'
  # Wait Mongo cluster to start
    sleep 10
    #while [ "$(mongo --quiet --eval "rs.status()"|grep "\"health\" : 1"|wc -l)" -ne "2" ]; do echo "Waiting for Mongo cluster to start";sleep 1; done
    #while [ "$(mongo --quiet --eval 'rs.isMaster().me')" != "10.30.20.10:27017" ]; do echo "Waiting for 10.30.20.10 to become Primary";sleep 1; done

  # Create the default database
    mongo --quiet --eval 'use default'

    apt-get -y upgrade
  SHELL
end
