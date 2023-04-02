# -*- mode: ruby -*-
# vim: set ft=ruby :

MACHINES = {
  :auth => {
        :box_name => "/hdd/hdd1/packer/Centos/packer_centos7/centos7-minimal.box"        
  },
}

Vagrant.configure("2") do |config|

  MACHINES.each do |boxname, boxconfig|
      config.ssh.private_key_path = "/home/alex/.vagrant.d/insecure_private_key"
      config.ssh.forward_agent = true
  
      config.vm.define boxname do |box|

          box.vm.box = boxconfig[:box_name]
          box.vm.host_name = boxname.to_s

          #box.vm.network "forwarded_port", guest: 3260, host: 3260+offset

          box.vm.network "public_network" , bridge: "enp0s31f6"

          box.vm.provider :virtualbox do |vb|
            vb.customize ["modifyvm", :id, "--memory", "2048"]
            vb.customize ["modifyvm", :id, "--cpus", "2"] 
          end
   	          
	  end 	 
	end 
  
end 
