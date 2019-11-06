VAGRANTFILE_API_VERSION = "2"

$test = <<-SCRIPT
  nod_nam=$1
  nod_ip=$2
  is_first_node=$3

  echo $nod_nam
  echo $nod_ip
  echo $is_first_node

  #echo "Substitution with sed..."
  #sed -i 's/127.0.1.1/#127.0.1.1/g' /etc/hosts
  cat /vagrant/json/define_replicaset.json | mongo --host mongodb-configsvr1 --port 27017
  echo $?
SCRIPT

$install_keepalived_haproxy = <<-SCRIPT
  node=$1
  echo ""
  echo "add service and run haproxy and keepalived on srv"$node
  echo ""
  sudo apt install keepalived haproxy -y

  rm -fv /etc/haproxy/haproxy.conf
  rm -fv /etc/keepalived/keepalived.conf

  cp -pv /vagrant/srv${node}/haproxy.cfg /etc/haproxy/
  cp -pv /vagrant/srv${node}/keepalived.conf /etc/keepalived/
  chmod a-x /etc/keepalived/keepalived.conf

  #allow binding to virtual IP of keepalived and apply this adjustment
  echo "net.ipv4.ip_nonlocal_bind=1" >> /etc/sysctl.conf
  sysctl -p

  systemctl daemon-reload
  systemctl start keepalived
  systemctl enable keepalived
  systemctl restart haproxy
  systemctl enable haproxy

SCRIPT

$install_misc = <<-SCRIPT
   apt update
   apt install mc -y
   echo -e "192.168.1.11       srv1\n192.168.1.12       srv2\n172.16.94.11       mongodb-node1\n172.16.94.12       mongodb-node2\n172.16.94.13       mongodb-node3\n172.16.94.14       mongodb-node4\n172.16.94.15       mongodb-node5\n172.16.94.16       mongodb-node6\n172.16.94.17       mongodb-configsvr1\n172.16.94.18       mongodb-configsvr2\n172.16.94.19       mongodb-router1\n172.16.94.20       mongodb-router2\n192.168.1.100       mongodb-router">> /etc/hosts
   sed -i 's/127.0.1.1/#127.0.1.1/g' /etc/hosts
SCRIPT

$install_mongodb_cluster = <<-SCRIPT
   nod_nam=$1

   apt-key adv --keyserver hkp://keyserver.ubuntu.com:80 --recv 9DA31620334BD75D9DCB49F368818C72E52529D4
   echo "deb [ arch=amd64 ] https://repo.mongodb.org/apt/ubuntu bionic/mongodb-org/4.0 multiverse" | sudo tee /etc/apt/sources.list.d/mongodb-org.list
   apt update
   apt install mongodb-org -y

   case $nod_nam in
     mongodb-configsvr2)
          echo "add service and run mongodb on"$nod_nam
          rm -fv /etc/mongod.conf
          cp -v /vagrant/mongodb-configsvr2/mongod.conf /etc/
          systemctl daemon-reload
          systemctl start mongod
          ;;
     mongodb-configsvr1)
          echo "add service and run mongodb on"$nod_nam
          rm -fv /etc/mongod.conf
          cp -v /vagrant/mongodb-configsvr1/mongod.conf /etc/
          systemctl daemon-reload
          systemctl start mongod
          sleep 2
          cat /vagrant/json/define_replicaset.json | mongo --host mongodb-configsvr1 --port 27017
          ;;
     mongodb-node2)
          echo "add service and run mongodb on"$nod_nam
          rm -fv /etc/mongod.conf
          cp -v /vagrant/mongodb-node2/mongod.conf /etc/
          systemctl daemon-reload
          systemctl start mongod
          ;;
     mongodb-node1)
          echo "add service and run mongodb on"$nod_nam
          rm -fv /etc/mongod.conf
          cp -v /vagrant/mongodb-node1/mongod.conf /etc/
          systemctl daemon-reload
          systemctl start mongod
          sleep 2
          cat /vagrant/json/sharedreplica01.json | mongo --host mongodb-node1 --port 27017
          ;;
     mongodb-node4)
          echo "add service and run mongodb on"$nod_nam
          rm -fv /etc/mongod.conf
          cp -v /vagrant/mongodb-node4/mongod.conf /etc/
          systemctl daemon-reload
          systemctl start mongod
          ;;
     mongodb-node3)
          echo "add service and run mongodb on"$nod_nam
          rm -fv /etc/mongod.conf
          cp -v /vagrant/mongodb-node3/mongod.conf /etc/
          systemctl daemon-reload
          systemctl start mongod
          sleep 2
          cat /vagrant/json/sharedreplica02.json | mongo --host mongodb-node3 --port 27017
          ;;
     mongodb-node6)
          echo "add service and run mongodb on"$nod_nam
          rm -fv /etc/mongod.conf
          cp -v /vagrant/mongodb-node6/mongod.conf /etc/
          systemctl daemon-reload
          systemctl start mongod
          ;;
     mongodb-node5)
          echo "add service and run mongodb on"$nod_nam
          rm -fv /etc/mongod.conf
          cp -v /vagrant/mongodb-node5/mongod.conf /etc/
          systemctl daemon-reload
          systemctl start mongod
          sleep 2
          cat /vagrant/json/sharedreplica03.json | mongo --host mongodb-node5 --port 27017
          ;;
     mongodb-router1)
          echo "add service and run mongodb on"$nod_nam
          rm -fv /etc/mongod.conf
          cp -v /vagrant/mongodb-router1/mongos.service /lib/systemd/system/
          systemctl daemon-reload
          systemctl start mongos
          sleep 5
          echo 'sh.addShard( "shardreplica01/mongodb-node1:27017")' | mongo --host mongodb-router1 --port 27017
          sleep 1
          echo 'sh.addShard( "shardreplica01/mongodb-node2:27017")' | mongo --host mongodb-router1 --port 27017
          sleep 1
          echo 'sh.addShard( "shardreplica02/mongodb-node3:27017")' | mongo --host mongodb-router1 --port 27017
          sleep 1
          echo 'sh.addShard( "shardreplica02/mongodb-node4:27017")' | mongo --host mongodb-router1 --port 27017
          sleep 1
          echo 'sh.addShard( "shardreplica03/mongodb-node5:27017")' | mongo --host mongodb-router1 --port 27017
          sleep 1
          echo 'sh.addShard( "shardreplica03/mongodb-node6:27017")' | mongo --host mongodb-router1 --port 27017
          ;;
     mongodb-router2)
          echo "add service and run mongodb on"$nod_nam
          rm -fv /etc/mongod.conf
          cp -v /vagrant/mongodb-router2/mongos.service /lib/systemd/system/
          systemctl daemon-reload
          systemctl start mongos
          ;;
   esac

SCRIPT


Vagrant.configure(VAGRANTFILE_API_VERSION) do |config|

  config.vm.define "srv1" do |srv1|
    srv1.vm.box = "bento/ubuntu-18.04"
    srv1.vm.hostname = "srv1"
    srv1.vm.network :private_network, ip: "192.168.1.11"
    config.vm.provision "script1", type: "shell" do |shell|
       shell.inline = $install_misc
    end
    config.vm.provision "script2", type: "shell" do |shell|
       shell.inline = $install_keepalived_haproxy
       shell.args = ["1"]
    end
  end

  config.vm.define "srv2" do |srv2|
    srv2.vm.box = "bento/ubuntu-18.04"
    srv2.vm.hostname = "srv2"
    srv2.vm.network :private_network, ip: "192.168.1.12"
    config.vm.provision "script1", type: "shell" do |shell|
       shell.inline = $install_misc
    end
    config.vm.provision "script2", type: "shell" do |shell|
       shell.inline = $install_keepalived_haproxy
       shell.args = ["2"]
    end
  end

  config.vm.define "configsvr2" do |configsvr2|
    configsvr2.vm.box = "bento/ubuntu-18.04"
    configsvr2.vm.hostname = "mongodb-configsvr2"
    configsvr2.vm.network :private_network, ip: "172.16.94.18"
    config.vm.provision "script1", type: "shell" do |shell|
       shell.inline = $install_misc
    end
    config.vm.provision "script2", type: "shell" do |shell|
        shell.inline = $install_mongodb_cluster
        shell.args = ["mongodb-configsvr2"]
    end
   end

  config.vm.define "configsvr1" do |configsvr1|
    configsvr1.vm.box = "bento/ubuntu-18.04"
    configsvr1.vm.hostname = "mongodb-configsvr1"
    configsvr1.vm.network :private_network, ip: "172.16.94.17"
    config.vm.provision "script1", type: "shell" do |shell|
       shell.inline = $install_misc
    end
    config.vm.provision "script2", type: "shell" do |shell|
       shell.inline = $install_mongodb_cluster
       shell.args = ["mongodb-configsvr1"]
    end
   end

  config.vm.define "mongodb2" do |mongodb2|
    mongodb2.vm.box = "bento/ubuntu-18.04"
    mongodb2.vm.hostname = "mongodb-node2"
    mongodb2.vm.network :private_network, ip: "172.16.94.12"
    config.vm.provision "script1", type: "shell" do |shell|
       shell.inline = $install_misc
    end
    config.vm.provision "script2", type: "shell" do |shell|
        shell.inline = $install_mongodb_cluster
        shell.args = ["mongodb-node2"]
    end
  end

  config.vm.define "mongodb1" do |mongodb1|
    mongodb1.vm.box = "bento/ubuntu-18.04"
    mongodb1.vm.hostname = "mongodb-node1"
    mongodb1.vm.network :private_network, ip: "172.16.94.11"
    config.vm.provision "script1", type: "shell" do |shell|
       shell.inline = $install_misc
    end
    config.vm.provision "script2", type: "shell" do |shell|
        shell.inline = $install_mongodb_cluster
        shell.args = ["mongodb-node1"]
    end
  end

  config.vm.define "mongodb3" do |mongodb3|
    mongodb3.vm.box = "bento/ubuntu-18.04"
    mongodb3.vm.hostname = "mongodb-node3"
    mongodb3.vm.network :private_network, ip: "172.16.94.13"
    config.vm.provision "script1", type: "shell" do |shell|
       shell.inline = $install_misc
    end
    config.vm.provision "script2", type: "shell" do |shell|
        shell.inline = $install_mongodb_cluster
        shell.args = ["mongodb-node3"]
    end
   end

  config.vm.define "mongodb4" do |mongodb4|
    mongodb4.vm.box = "bento/ubuntu-18.04"
    mongodb4.vm.hostname = "mongodb-node4"
    mongodb4.vm.network :private_network, ip: "172.16.94.14"
    config.vm.provision "script1", type: "shell" do |shell|
       shell.inline = $install_misc
    end
    config.vm.provision "script2", type: "shell" do |shell|
        shell.inline = $install_mongodb_cluster
        shell.args = ["mongodb-node4"]
    end
   end

  config.vm.define "mongodb6" do |mongodb6|
    mongodb6.vm.box = "bento/ubuntu-18.04"
    mongodb6.vm.hostname = "mongodb-node6"
    mongodb6.vm.network :private_network, ip: "172.16.94.16"
    config.vm.provision "script1", type: "shell" do |shell|
       shell.inline = $install_misc
    end
    config.vm.provision "script2", type: "shell" do |shell|
        shell.inline = $install_mongodb_cluster
        shell.args = ["mongodb-node6"]
    end
   end


  config.vm.define "mongodb5" do |mongodb5|
    mongodb5.vm.box = "bento/ubuntu-18.04"
    mongodb5.vm.hostname = "mongodb-node5"
    mongodb5.vm.network :private_network, ip: "172.16.94.15"
    config.vm.provision "script1", type: "shell" do |shell|
       shell.inline = $install_misc
    end
    config.vm.provision "script2", type: "shell" do |shell|
        shell.inline = $install_mongodb_cluster
        shell.args = ["mongodb-node5"]
    end
   end

  config.vm.define "mongos1" do |mongos1|
    mongos1.vm.box = "bento/ubuntu-18.04"
    mongos1.vm.hostname = "mongodb-router1"
    mongos1.vm.network :private_network, ip: "172.16.94.19"
    config.vm.provision "script1", type: "shell" do |shell|
       shell.inline = $install_misc
    end
    config.vm.provision "script2", type: "shell" do |shell|
        shell.inline = $install_mongodb_cluster
        shell.args = ["mongodb-router1"]
    end
   end

  config.vm.define "mongos2" do |mongos2|
    mongos2.vm.box = "bento/ubuntu-18.04"
    mongos2.vm.hostname = "mongodb-router2"
    mongos2.vm.network :private_network, ip: "172.16.94.20"
    config.vm.provision "script1", type: "shell" do |shell|
       shell.inline = $install_misc
    end
    config.vm.provision "script2", type: "shell" do |shell|
        shell.inline = $install_mongodb_cluster
        shell.args = ["mongodb-router2"]
    end
   end


  config.vm.provider :virtualbox do |vb|
    vb.customize ["modifyvm", :id, "--memory", "320"]
    vb.customize ["modifyvm", :id, "--cpus", "1"]
  end
end
