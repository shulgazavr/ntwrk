# -*- mode: ruby -*-
# vim: set ft=ruby :
# -*- mode: ruby -*-
# vim: set ft=ruby :

MACHINES = {
# routers  
  :RI => {
        :box_name => "centos/7",
        :net => [
                  {ip: '192.168.255.1', adapter: 2, netmask: "255.255.255.252", virtualbox__intnet: "ri-net"}, # done
                ]
  },
  :RC => {
        :box_name => "centos/7",
        :net => [
                  {ip: '192.168.255.2', adapter: 2, netmask: "255.255.255.252", virtualbox__intnet: "ri-net"}, # done
                  {ip: '192.168.20.1', adapter: 3, netmask: "255.255.255.0", virtualbox__intnet: "rc-r1-net"}, # done
                  {ip: '192.168.10.1', adapter: 4, netmask: "255.255.255.0", virtualbox__intnet: "rc-r2-net"}, # done
                  {ip: '192.168.0.1', adapter: 5, netmask: "255.255.255.240", virtualbox__intnet: "rc-dir-net"}, # done
                  {ip: '192.168.0.31', adapter: 6, netmask: "255.255.255.240", virtualbox__intnet: "rc-hw-net"}, # done
                  {ip: '192.168.0.64', adapter: 7, netmask: "255.255.255.192", virtualbox__intnet: "rc-wf-net"}, # done
                ]
  },
  :R1 => {
        :box_name => "centos/7",
        :net => [  
                  {ip: '192.168.20.2', adapter: 2, netmask: "255.255.255.0", virtualbox__intnet: "rc-r1-net"}, # done
                  {ip: '192.168.2.1', adapter: 3, netmask: "255.255.255.192", virtualbox__intnet: "r1-dev-net"}, # done
                  {ip: '192.168.2.65', adapter: 4, netmask: "255.255.255.192", virtualbox__intnet: "r1-tst-net"}, # done
                  {ip: '192.168.2.129', adapter: 5, netmask: "255.255.255.192", virtualbox__intnet: "r1-mng-net"}, # done
                  {ip: '192.168.2.193', adapter: 6, netmask: "255.255.255.192", virtualbox__intnet: "r1-hw-net"}, # done
                ]
  },
  :R2 => {
        :box_name => "centos/7",
        :net => [
                  {ip: '192.168.10.2', adapter: 2, netmask: "255.255.255.0", virtualbox__intnet: "rc-r2-net"}, # done
                  {ip: '192.168.1.1', adapter: 3, netmask: "255.255.255.128", virtualbox__intnet: "r2-dev-net"}, # done
                  {ip: '192.168.1.129', adapter: 4, netmask: "255.255.255.192", virtualbox__intnet: "r2-tst-net"}, # done
                  {ip: '192.168.1.193', adapter: 5, netmask: "255.255.255.192", virtualbox__intnet: "r2-hw-net"}, # done
                ]
  },
# servers
  :SC => {
        :box_name => "centos/7",
        :net => [
                  {ip: '192.168.0.2', adapter: 2, netmask: "255.255.255.240", virtualbox__intnet: "rc-dir-net"}, # done
                ]
  },    
  :S1 => {
        :box_name => "centos/7",
        :net => [
                  {ip: '192.168.2.2', adapter: 2, netmask: "255.255.255.192", virtualbox__intnet: "r1-dev-net"},  # done
                ]
  },
  :S2 => {
        :box_name => "centos/7",
        :net => [
                  {ip: '192.168.1.2', adapter: 2, netmask: "255.255.255.192", virtualbox__intnet: "r2-dev-net"},  # done
                ]
  },
}

Vagrant.configure("2") do |config|
  MACHINES.each do |boxname, boxconfig|
        config.vm.define boxname do |box|
              box.vm.box = boxconfig[:box_name]    
              box.vm.host_name = boxname.to_s
              
              boxconfig[:net].each do |ipconf|
                box.vm.network "private_network", ipconf
              end
              
              if boxconfig.key?(:public)
                box.vm.network "public_network", boxconfig[:public]
              end

              box.vm.provision "shell", inline: <<-SHELL
                mkdir -p ~root/.ssh
                      cp ~vagrant/.ssh/auth* ~root/.ssh
              SHELL
              
              case boxname.to_s
                  when "RI"
                    box.vm.provision "shell", run: "always", inline: <<-SHELL
                      sysctl net.ipv4.conf.all.forwarding=1
                      iptables -t nat -A POSTROUTING ! -d 192.168.0.0/16 -o eth0 -j MASQUERADE
                      ip route add 192.168.0.0/16 via 192.168.255.2
                    SHELL
                  when "RC" 
                    box.vm.provision "shell", run: "always", inline: <<-SHELL
                      echo "net.ipv4.conf.all.forwarding=1" >> /etc/sysctl.conf
                      echo "DEFROUTE=no" >> /etc/sysconfig/network-scripts/ifcfg-eth0 
                      echo "GATEWAY=192.168.255.1" >> /etc/sysconfig/network-scripts/ifcfg-eth1
                      iptables --table nat --append POSTROUTING --out-interface eth0 -j MASQUERADE
                      iptables --append FORWARD --in-interface eth1 -j ACCEPT
                      iptables --append FORWARD --in-interface eth2 -j ACCEPT
                      iptables --append FORWARD --in-interface eth3 -j ACCEPT
                      iptables --append FORWARD --in-interface eth4 -j ACCEPT
                      iptables --append FORWARD --in-interface eth5 -j ACCEPT
                      iptables --append FORWARD --in-interface eth6 -j ACCEPT
                      ip ro add 192.168.2.0/24 via 192.168.20.2
                      ip ro add 192.168.1.0/24 via 192.168.10.2
                      sed -i '65s/PasswordAuthentication no/PasswordAuthentication yes/g' /etc/ssh/sshd_config
                      systemctl restart sshd
                      systemctl restart network
                    SHELL
                  when "R1"
                    box.vm.provision "shell", run: "always", inline: <<-SHELL
                      echo "net.ipv4.conf.all.forwarding=1" >> /etc/sysctl.conf
                      echo "DEFROUTE=no" >> /etc/sysconfig/network-scripts/ifcfg-eth0
                      echo "GATEWAY=192.168.20.1" >> /etc/sysconfig/network-scripts/ifcfg-eth1
                      iptables --append FORWARD --in-interface eth1 -j ACCEPT
                      iptables --append FORWARD --in-interface eth2 -j ACCEPT
                      iptables --append FORWARD --in-interface eth3 -j ACCEPT
                      iptables --append FORWARD --in-interface eth4 -j ACCEPT
                      iptables --append FORWARD --in-interface eth5 -j ACCEPT
                      ip ro add 192.168.1.0/24 via 192.168.20.1
                      ip ro add 192.168.0.0/24 via 192.168.20.1
                      sed -i '65s/PasswordAuthentication no/PasswordAuthentication yes/g' /etc/ssh/sshd_config
                      systemctl restart sshd        
                      systemctl restart network
                    SHELL
                  when "R2"
                    box.vm.provision "shell", run: "always", inline: <<-SHELL
                      echo "net.ipv4.conf.all.forwarding=1" >> /etc/sysctl.conf
                      echo "DEFROUTE=no" >> /etc/sysconfig/network-scripts/ifcfg-eth0
                      echo "GATEWAY=192.168.10.1" >> /etc/sysconfig/network-scripts/ifcfg-eth1
                      iptables --append FORWARD --in-interface eth1 -j ACCEPT
                      iptables --append FORWARD --in-interface eth2 -j ACCEPT
                      iptables --append FORWARD --in-interface eth3 -j ACCEPT
                      iptables --append FORWARD --in-interface eth4 -j ACCEPT
                      ip ro add 192.168.2.0/24 via 192.168.10.1
                      ip ro add 192.168.0.0/24 via 192.168.10.1
                      sed -i '65s/PasswordAuthentication no/PasswordAuthentication yes/g' /etc/ssh/sshd_config
                      systemctl restart sshd         
                      systemctl restart network
                    SHELL
                  when "SC"
                    box.vm.provision "shell", run: "always", inline: <<-SHELL
                      echo "DEFROUTE=no" >> /etc/sysconfig/network-scripts/ifcfg-eth0
                      echo "GATEWAY=192.168.0.1" >> /etc/sysconfig/network-scripts/ifcfg-eth1
                      sed -i '65s/PasswordAuthentication no/PasswordAuthentication yes/g' /etc/ssh/sshd_config
                      ip ro add 192.168.1.0/24 via 192.168.0.1
                      ip ro add 192.168.2.0/24 via 192.168.0.1
                      systemctl restart sshd
                      systemctl restart network
                    SHELL
                  when "S1"    
                    box.vm.provision "shell", run: "always", inline: <<-SHELL
                      echo "DEFROUTE=no" >> /etc/sysconfig/network-scripts/ifcfg-eth0 
                      echo "GATEWAY=192.168.2.1" >> /etc/sysconfig/network-scripts/ifcfg-eth1
                      ip ro add 192.168.0.0/24 via 192.168.2.1
                      ip ro add 192.168.1.0/24 via 192.168.2.1
                      sed -i '65s/PasswordAuthentication no/PasswordAuthentication yes/g' /etc/ssh/sshd_config
                      systemctl restart sshd
                      systemctl restart network
                    SHELL
                  when "S2"    
                    box.vm.provision "shell", run: "always", inline: <<-SHELL
                      echo "DEFROUTE=no" >> /etc/sysconfig/network-scripts/ifcfg-eth0 
                      echo "GATEWAY=192.168.1.1" >> /etc/sysconfig/network-scripts/ifcfg-eth1
                      ip ro add 192.168.0.0/24 via 192.168.1.1
                      ip ro add 192.168.2.0/24 via 192.168.1.1
                      sed -i '65s/PasswordAuthentication no/PasswordAuthentication yes/g' /etc/ssh/sshd_config
                      systemctl restart sshd
                      systemctl restart network
                    SHELL
              end
        end
  end
end