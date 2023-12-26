# -*- mode: ruby -*-
# vim: set ft=ruby :

MACHINES = {
  :inetRouter => {
        :box_name => "centos/7",
        :vm_name => "inetRouter",
        :net => [   
                    #ip, adpter, netmask, virtualbox__intnet
                    ["192.168.255.1", 2, "255.255.255.252",  "router-net"], 
                    ["192.168.50.10", 8, "255.255.255.0",    "mgmt"],
                ]
  },

  :centralRouter => {
        :box_name => "centos/7",
        :vm_name => "centralRouter",
        :net => [
                   ["192.168.255.2",  2, "255.255.255.252",  "router-net"],
                   ["192.168.0.1",    3, "255.255.255.240",  "dir-net"],
                   ["192.168.0.33",   4, "255.255.255.240",  "hw-net"],
                   ["192.168.0.65",   5, "255.255.255.192",  "mgt-net"],
                   ["192.168.255.9",  6, "255.255.255.252",  "office1-central"],
                   ["192.168.255.5",  7, "255.255.255.252",  "office2-central"],
                   ["192.168.50.11",  8, "255.255.255.0",    "mgmt"],
                ]
  },
  
  :centralServer => {
        :box_name => "centos/7",
        :vm_name => "centralServer",
        :net => [
                   ["192.168.0.2",    2, "255.255.255.240",  "dir-net"],
                   ["192.168.50.12",  8, "255.255.255.0",    "mgmt"],
                ]
  },

  :office1Router => {
        :box_name => "centos/7",
        :vm_name => "office1Router",
        :net => [
                   ["192.168.255.10",  2,  "255.255.255.252",  "office1-central"],
                   ["192.168.2.1",     3,  "255.255.255.192",  "dev1-net"],
                   ["192.168.2.65",    4,  "255.255.255.192",  "test1-net"],
                   ["192.168.2.129",   5,  "255.255.255.192",  "managers-net"],
                   ["192.168.2.193",   6,  "255.255.255.192",  "office1-net"],
                   ["192.168.50.20",   8,  "255.255.255.0",    "mgmt"],
                ]
  },

  :office1Server => {
        :box_name => "centos/7",
        :vm_name => "office1Server",
        :net => [
                   ["192.168.2.130",  2,  "255.255.255.192",  "managers-net"],
                   ["192.168.50.21",  8,  "255.255.255.0",    "mgmt"],
                ]
  },

  :office2Router => {
       :box_name => "centos/7",
       :vm_name => "office2Router",
       :net => [
                   ["192.168.255.6",  2,  "255.255.255.252",  "office2-central"],
                   ["192.168.1.1",    3,  "255.255.255.128",  "dev2-net"],
                   ["192.168.1.129",  4,  "255.255.255.192",  "test2-net"],
                   ["192.168.1.193",  5,  "255.255.255.192",  "office2-net"],
                   ["192.168.50.30",  8, "255.255.255.0",     "mgmt"],
               ]
  },

  :office2Server => {
       :box_name => "centos/7",
       :vm_name => "office2Server",
       :net => [
                  ["192.168.1.2",    2,  "255.255.255.128",  "dev2-net"],
                  ["192.168.50.31",  8, "255.255.255.0", "mgmt"],
               ]
  }
}

Vagrant.configure("2") do |config|

    MACHINES.each do |boxname, boxconfig|
  
      config.vm.define boxname do |box|
  
          box.vm.box = boxconfig[:box_name]
          box.vm.host_name = boxname.to_s
  
          boxconfig[:net].each do |ipconf|
            box.vm.network("private_network", ip: ipconf[0], adapter: ipconf[1], netmask: ipconf[2], virtualbox__intnet: ipconf[3])
          end
          
          box.vm.provision "shell", inline: <<-SHELL
            mkdir -p ~root/.ssh
                  cp ~vagrant/.ssh/auth* ~root/.ssh
          SHELL
          
          case boxname.to_s
          when "inetRouter"
            box.vm.provision "shell", run: "always", inline: <<-SHELL
              sudo bash -c 'echo "net.ipv4.conf.all.forwarding=1" >> /etc/sysctl.conf'; sudo sysctl -p
              sudo yum install -y iptables-services; sudo systemctl enable iptables && sudo systemctl start iptables;
              #сброс всех цепочек с последующим удалением всех правил
              sudo iptables -F; sudo iptables -t nat -A POSTROUTING ! -d 192.168.0.0/16 -o eth0 -j MASQUERADE; sudo service iptables save
              #добавляем запись в цепочку POSTROUTING, настраиваем NAT
              sudo bash -c 'echo "192.168.0.0/16 via 192.168.255.2 dev eth1" > /etc/sysconfig/network-scripts/route-eth1'; sudo systemctl restart network
              sudo yum install -y traceroute
              sudo reboot
              SHELL
          when "centralRouter"
            box.vm.provision "shell", run: "always", inline: <<-SHELL
              sudo bash -c 'echo "net.ipv4.conf.all.forwarding=1" >> /etc/sysctl.conf'; sudo sysctl -p
              # отключаем дефолт на нат (eth0)
              echo "DEFROUTE=no" >> /etc/sysconfig/network-scripts/ifcfg-eth0 
              sudo nmcli connection modify "System eth1" +ipv4.addresses "192.168.254.1/30"; sudo nmcli connection modify "System eth1" +ipv4.addresses "192.168.253.1/30"
              sudo bash -c 'echo "192.168.2.0/24 via 192.168.254.2 dev eth1" > /etc/sysconfig/network-scripts/route-eth1'
              sudo bash -c 'echo "192.168.1.0/24 via 192.168.253.2 dev eth1" >> /etc/sysconfig/network-scripts/route-eth1'
              echo "GATEWAY=192.168.255.1" >> /etc/sysconfig/network-scripts/ifcfg-eth1
              sudo systemctl restart network
              sudo yum install -y traceroute
              sudo reboot
              SHELL
          when "centralServer"
            box.vm.provision "shell", run: "always", inline: <<-SHELL
              echo "DEFROUTE=no" >> /etc/sysconfig/network-scripts/ifcfg-eth0 
              echo "GATEWAY=192.168.0.1" >> /etc/sysconfig/network-scripts/ifcfg-eth1
              sudo systemctl restart network
              sudo yum install -y traceroute
              sudo reboot
              SHELL
          when "office1Router"
            box.vm.provision "shell", run: "always", inline: <<-SHELL
              sudo bash -c 'echo "net.ipv4.conf.all.forwarding=1" >> /etc/sysctl.conf'; sudo sysctl -p
              echo "DEFROUTE=no" >> /etc/sysconfig/network-scripts/ifcfg-eth0 
              echo "GATEWAY=192.168.254.1" >> /etc/sysconfig/network-scripts/ifcfg-eth1
              sudo systemctl restart network
              sudo yum install -y traceroute
              sudo reboot
              SHELL
          when "office1Server"
            box.vm.provision "shell", run: "always", inline: <<-SHELL
              echo "DEFROUTE=no" >> /etc/sysconfig/network-scripts/ifcfg-eth0 
              echo "GATEWAY=192.168.2.1" >> /etc/sysconfig/network-scripts/ifcfg-eth1
              sudo systemctl restart network
              sudo yum install -y traceroute
              sudo reboot
              SHELL
          when "office2Router"
            box.vm.provision "shell", run: "always", inline: <<-SHELL
              sudo bash -c 'echo "net.ipv4.conf.all.forwarding=1" >> /etc/sysctl.conf'; sudo sysctl -p
              echo "DEFROUTE=no" >> /etc/sysconfig/network-scripts/ifcfg-eth0 
              echo "GATEWAY=192.168.253.1" >> /etc/sysconfig/network-scripts/ifcfg-eth1
              sudo systemctl restart network
              sudo yum install -y traceroute
              sudo reboot
              SHELL
           when "office2Server"
             box.vm.provision "shell", run: "always", inline: <<-SHELL
               echo "DEFROUTE=no" >> /etc/sysconfig/network-scripts/ifcfg-eth0 
               echo "GATEWAY=192.168.1.1" >> /etc/sysconfig/network-scripts/ifcfg-eth1
               sudo systemctl restart network
               sudo yum install -y traceroute
               sudo reboot
               SHELL
          end
  
        end
  
    end
    
    
  end
