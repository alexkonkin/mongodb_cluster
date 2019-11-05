VAGRANTFILE_API_VERSION = "2"

$install_mysql_cluster1 = <<-SCRIPT
  nod_nam=$1
  nod_ip=$2
  is_first_node=$3

  echo $nod_nam
  echo $nod_ip
  echo $is_first_node

  echo "Substitution with sed..."
  sed -i 's/bind-address\t\t= 127.0.0.1/#bind-address\t\t= 127.0.0.1\\nskip-bind-address\\nskip-networking=0\\n/g' /etc/mysql/my.cnf
  echo $?
SCRIPT

$install_keepalived_haproxy = <<-SCRIPT
  state=$1
  priority=$2
  this_ip=$3
  peer_ip=$4

  echo $nod_nam
  echo $nod_ip
  echo $is_first_node
  sudo apt install keepalived haproxy mc -y
  echo -e "192.168.1.11       srv1\n192.168.1.12       srv2\n172.16.94.11       mongodb-node1\n172.16.94.12       mongodb-node2\n172.16.94.13       mongodb-node3\n172.16.94.14       mongodb-node4\n172.16.94.15       mongodb-node5\n172.16.94.16       mongodb-node6\n172.16.94.17       mongodb-configsvr1\n172.16.94.18       mongodb-configsvr2\n172.16.94.19       mongodb-router1\n172.16.94.20       mongodb-router2\n192.168.1.100       mongodb-router">> /etc/hosts

cat > /etc/haproxy/haproxy.cfg << "END"
global
    log 127.0.0.1 local0 notice
    user haproxy
    group haproxy


defaults
    log     global
    option  dontlognull
    retries 3
    option redispatch
    timeout connect  5000
    timeout client  50000
    timeout server  50000
    stats enable
    stats uri /haproxy/stats
    stats auth admin:admin


frontend http
    bind *:80
    mode http
    option httplog
    default_backend webservers

backend webservers
    mode http
    stats enable
    stats uri /haproxy/stats
    stats auth admin:admin
    stats hide-version
    balance roundrobin
    option httpclose
    option forwardfor
    cookie SRVNAME insert
    server srv1 192.168.1.11:80 check cookie srv1
    server srv2 192.168.1.12:80 check cookie srv2

frontend mongos
    bind 192.168.1.100:27017
    mode tcp
    option tcplog
    default_backend mongos_servers

backend mongos_servers
    mode tcp
    balance roundrobin
    option tcplog
    server mongodb-router1 172.16.94.19:27017 check
    server mongodb-router2 172.16.94.20:27017 check
    redispatch
END

cat > /etc/keepalived/keepalived.conf << "END"
vrrp_script chk_haproxy {
    script "killall -0 haproxy"     # cheaper than pidof
    interval 2                      # check every 2 seconds
}

vrrp_instance VI_1 {
    state STATE
    interface eth1
    virtual_router_id 51
    priority PRIORITY
    vrrp_unicast_bind THIS_IP   # Internal IP of this machine
    vrrp_unicast_peer PEER_IP   # Internal IP of peer
    virtual_ipaddress {
        192.168.1.100 dev eth1 label eth1:vip1
    }
    track_script {
        chk_haproxy weight 2
    }
}
END

#allow binding to virtual IP of keepalived and apply this adjustment
echo "net.ipv4.ip_nonlocal_bind=1" >> /etc/sysctl.conf
sysctl -p

#configue master/slave keepalived instances
sed -i 's/STATE/'$state'/g' /etc/keepalived/keepalived.conf
sed -i 's/PRIORITY/'$priority'/g' /etc/keepalived/keepalived.conf
sed -i 's/THIS_IP/'$this_ip'/g' /etc/keepalived/keepalived.conf
sed -i 's/PEER_IP/'$peer_ip'/g' /etc/keepalived/keepalived.conf

systemctl start keepalived
systemctl restart haproxy

SCRIPT

$install_misc_packages = <<-SCRIPT
   apt update
   apt install mc -y
SCRIPT

$install_mongodb_cluster = <<-SCRIPT
   nod_nam=$1
   nod_ip=$2
   is_first_node=$3

   echo -e "192.168.1.11       srv1\n192.168.1.12       srv2\n172.16.94.11       mongodb-node1\n172.16.94.12       mongodb-node2\n172.16.94.13       mongodb-node3\n172.16.94.14       mongodb-node4\n172.16.94.15       mongodb-node5\n172.16.94.16       mongodb-node6\n172.16.94.17       mongodb-configsvr1\n172.16.94.18       mongodb-configsvr2\n172.16.94.19       mongodb-router1\n172.16.94.20       mongodb-router2\n192.168.1.100       mongodb-router">> /etc/hosts
   apt-key adv --keyserver hkp://keyserver.ubuntu.com:80 --recv 9DA31620334BD75D9DCB49F368818C72E52529D4
   echo "deb [ arch=amd64 ] https://repo.mongodb.org/apt/ubuntu bionic/mongodb-org/4.0 multiverse" | sudo tee /etc/apt/sources.list.d/mongodb-org.list
   apt update
   apt install mongodb-org -y

SCRIPT


Vagrant.configure(VAGRANTFILE_API_VERSION) do |config|

  config.vm.define "srv1" do |srv1|
    srv1.vm.box = "bento/ubuntu-18.04"
    srv1.vm.hostname = "srv1"
    srv1.vm.network :private_network, ip: "192.168.1.11"
    config.vm.provision "script1", type: "shell" do |shell|
       shell.inline = $install_keepalived_haproxy
       shell.args = ["MASTER","101","192.168.1.11","192.168.1.12"]
    end
  end


  config.vm.define "srv2" do |srv2|
    srv2.vm.box = "bento/ubuntu-18.04"
    srv2.vm.hostname = "srv2"
    srv2.vm.network :private_network, ip: "192.168.1.12"
    config.vm.provision "script1", type: "shell" do |shell|
       shell.inline = $install_keepalived_haproxy
       shell.args = ["BACKUP","100","192.168.1.12","192.168.1.11"]
    end
  end

  config.vm.define "mongodb1" do |mongodb1|
    mongodb1.vm.box = "bento/ubuntu-18.04"
    mongodb1.vm.hostname = "mongodb-node1"
    mongodb1.vm.network :private_network, ip: "172.16.94.11"
       config.vm.provision "script1", type: "shell" do |shell|
       shell.inline = $install_misc_packages
    end
    config.vm.provision "script2", type: "shell" do |shell|
        shell.inline = $install_mongodb_cluster
        shell.args = ["mongodb-node1","172.16.94.11","true"]
    end
  end

  config.vm.define "mongodb2" do |mongodb2|
    mongodb2.vm.box = "bento/ubuntu-18.04"
    mongodb2.vm.hostname = "mongodb-node2"
    mongodb2.vm.network :private_network, ip: "172.16.94.12"
       config.vm.provision "script1", type: "shell" do |shell|
       shell.inline = $install_misc_packages
    end
    config.vm.provision "script2", type: "shell" do |shell|
        shell.inline = $install_mongodb_cluster
        shell.args = ["mongodb-node2","172.16.94.12","false"]
    end
  end

  config.vm.define "mongodb3" do |mongodb3|
    mongodb3.vm.box = "bento/ubuntu-18.04"
    mongodb3.vm.hostname = "mongodb-node3"
    mongodb3.vm.network :private_network, ip: "172.16.94.13"
       config.vm.provision "script1", type: "shell" do |shell|
       shell.inline = $install_misc_packages
    end
    config.vm.provision "script2", type: "shell" do |shell|
        shell.inline = $install_mongodb_cluster
        shell.args = ["mongodb-node3","172.16.94.13","false"]
    end
   end

  config.vm.define "mongodb4" do |mongodb4|
    mongodb4.vm.box = "bento/ubuntu-18.04"
    mongodb4.vm.hostname = "mongodb-node4"
    mongodb4.vm.network :private_network, ip: "172.16.94.14"
       config.vm.provision "script1", type: "shell" do |shell|
       shell.inline = $install_misc_packages
    end
    config.vm.provision "script2", type: "shell" do |shell|
        shell.inline = $install_mongodb_cluster
        shell.args = ["mongodb-node4","172.16.94.14","false"]
    end	
   end

  config.vm.define "mongodb5" do |mongodb5|
    mongodb5.vm.box = "bento/ubuntu-18.04"
    mongodb5.vm.hostname = "mongodb-node5"
    mongodb5.vm.network :private_network, ip: "172.16.94.15"
       config.vm.provision "script1", type: "shell" do |shell|
       shell.inline = $install_misc_packages
    end
    config.vm.provision "script2", type: "shell" do |shell|
        shell.inline = $install_mongodb_cluster
        shell.args = ["mongodb-node5","172.16.94.15","false"]
    end	
   end

  config.vm.define "mongodb6" do |mongodb6|
    mongodb6.vm.box = "bento/ubuntu-18.04"
    mongodb6.vm.hostname = "mongodb-node6"
    mongodb6.vm.network :private_network, ip: "172.16.94.16"
       config.vm.provision "script1", type: "shell" do |shell|
       shell.inline = $install_misc_packages
    end
    config.vm.provision "script2", type: "shell" do |shell|
        shell.inline = $install_mongodb_cluster
        shell.args = ["mongodb-node6","172.16.94.15","false"]
    end
   end

  config.vm.define "configsvr1" do |configsvr1|
    configsvr1.vm.box = "bento/ubuntu-18.04"
    configsvr1.vm.hostname = "mongodb-configsvr1"
    configsvr1.vm.network :private_network, ip: "172.16.94.17"
       config.vm.provision "script1", type: "shell" do |shell|
       shell.inline = $install_misc_packages
    end
    config.vm.provision "script2", type: "shell" do |shell|
       shell.inline = $install_mongodb_cluster
       shell.args = ["mongodb-configsvr1","172.16.94.17","false"]
    end
   end

  config.vm.define "configsvr2" do |configsvr2|
    configsvr2.vm.box = "bento/ubuntu-18.04"
    configsvr2.vm.hostname = "mongodb-configsvr2"
    configsvr2.vm.network :private_network, ip: "172.16.94.18"
       config.vm.provision "script1", type: "shell" do |shell|
       shell.inline = $install_misc_packages
    end
    config.vm.provision "script2", type: "shell" do |shell|
        shell.inline = $install_mongodb_cluster
        shell.args = ["mongodb-configsvr2","172.16.94.18","false"]
    end
   end

  config.vm.define "mongos1" do |mongos1|
    mongos1.vm.box = "bento/ubuntu-18.04"
    mongos1.vm.hostname = "mongodb-router1"
    mongos1.vm.network :private_network, ip: "172.16.94.19"
       config.vm.provision "script1", type: "shell" do |shell|
       shell.inline = $install_misc_packages
    end
    config.vm.provision "script2", type: "shell" do |shell|
        shell.inline = $install_mongodb_cluster
        shell.args = ["mongodb-router1","172.16.94.19","false"]
    end
   end

  config.vm.define "mongos2" do |mongos2|
    mongos2.vm.box = "bento/ubuntu-18.04"
    mongos2.vm.hostname = "mongodb-router2"
    mongos2.vm.network :private_network, ip: "172.16.94.20"
       config.vm.provision "script1", type: "shell" do |shell|
       shell.inline = $install_misc_packages
    end
    config.vm.provision "script2", type: "shell" do |shell|
        shell.inline = $install_mongodb_cluster
        shell.args = ["mongodb-router2","172.16.94.20","false"]
    end
   end


  config.vm.provider :virtualbox do |vb|
    vb.customize ["modifyvm", :id, "--memory", "320"]
    vb.customize ["modifyvm", :id, "--cpus", "1"]
  end
end
